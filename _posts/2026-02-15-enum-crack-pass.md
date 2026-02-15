---
title: "Enumeración - Cracking de contraseñas"
date: 2026-02-15 16:45:00 -0500
categories: [Hacking, Cracking]
tags: [cracking, enumeración, hacking-etico]
description: Cómo enumerar usuarios y crackear contraseñas de servicios vulnerables.
---

## Reconocimiento de Windows - NMAP Descubrimiento de Hosts
Lo primero que se debe realizar para un reconocimiento de un equipo, es probar la conectividad con el dispositivo.
```
ping dominio.com
```
En caso de **NO TENER PING**, no significa que el dispositivo no sea accesible, puede significar que ICMP se encuentre desactivado o esté en medio de un FireWall.
```
nmap -Pn dominio.com
```
Con este comando se puede determinar si el dominio está o no activo, sin necesidad de realizar un ping.

## Enumeración de servicios FTP
FTP (File PRotocol Transfer). Protocolo que permite transferir archivos entre varios host dentro de una red.

Para este proceso se va a analizar toda la información que posea FTP. Para ello iniciamos
```shell
#Revisión de conectividad con el dominio
ping dominio.com
nmap -p21 dominio.com

#Inicia el servicio de la base de datos de msfconsole
service postgresql start 
msfdb init

msfconsole -q #Se salta el panel de Login de Msfconsole

#Creación de Workspace
Workspace #Permite observar los workspace existentes
Workspace -a FTP_ENUM

#Búsqueda de puertos
search portscan
use scanner/portscan/tcp
set rhosts dominio.com
run

#Módulos para enumerar FTP
search type:auxiliary name:ftp #Permite hacer una búsqueda más exacta

use scanner/ftp/ftp_version
setg rhosts dominio.com #SetG permite configurar una variable global, lo que significa que todos los RHOSTS van a estar configurados.
show options #Permite ver las configuraciones del módulo
run

search type:auxiliary name:ftp #Permite hacer una búsqueda más exacta
use scanner/ftp/ftp_login
show options

set USER_FILE /usr/share/metasploit-framework/data/wordlist/unix_users.txt #Se configura un diccionario para que haga la fuerza bruta de usuarios
set USER_PASSFILE /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt #Se configura un diccionario de contraseñas
run

search type:auxiliary name:ftp
use scanner/ftp/anonymous
show options
run
```

Ahora se procederá a analizar ftp mediante la herramiento **Hydra**
```shell
hydra -L /usr/share/metasploit-framework/data/wordlist/unix_users.txt -P /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt ftp://dominio.com
```

Otra opción es mediante **MEDUSA**
```shell
medusa -h dominio.com -U /usr/share/metasploit-framework/data/wordlist/unix_users.txt -P /usr/share/metasploit-framework/data/wordlist/unix_passwords.txt ftp://dominio.com -M ftp -t 8 -f
#En donde:
#   -M es el módulo FTP que utiliza.
#   -t son los hilos
#   -f muestra la información de las pruebas en pantalla
```

Una vez que se haya conseguido las credenciales, se procede a realizar la conexión:
```shell
ftp dominio.com
> get archivo.txt
> exit
cat archivo.txt
```

Buscar vulnerabilidades con versiones exactas
```shell
searchsploit ProFTPD 1.3.5a #Si no encuentra resultado, buscar con una versión menos.
searchsploit ProFTPD 1.3.5
```

También se puede realizar una consulta en GOOGLE buscando: *ProFTPD 1.3.5a exploit*

### Material de apoyo
Se puede buscar más comandos en cheatsheet de apoyo como el que les comparto a continuación:
 **[CheatSheet](https://www.stationx.net/nmap-cheat-sheet/)**, **[Hausec](https://hausec.com/pentesting-cheatsheet/#_Toc475368978)**

## Reconocimiento Básico SAMBA (SMB)

#Video 1:10:00

## FootPrinting y Escaneo
