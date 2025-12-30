---
title: "Reconocimiento Activo: El Arte de Investigar"
date: 2025-12-29 00:45:00 -0500
categories: [Hacking, Reconocimiento]
tags: [osint, activo, hacking-etico]
description: Guía estratégica para realizar una recolección de información (OSINT) efectiva y ética.
---

## ¿Qué es el Reconocimiento Activo?


# Comandos útiles para realizar el reconocimeinto
En linux existe un gran número de comandos que permiten escaner direcciones IP y servicios que tiene un servidor.

**Arp-Scan** Con este comando se puede analizar todas las direcciones Ip y máscaras de red que poseen los equipos que están conectados dentro de mi red local (--localnet).

```shell
sudo arp-scan --localnet
```

**NMAP** Es una herramienta de línea de comandos de Linux de código abierto que se utiliza para escanear direcciones IP y puertos en una red y para detectar aplicaciones instaladas.

```shell
sudo nmap -sn (subred)/máscara
#Ejemplo
sudo nmap -sn 192.168.100.0/24
```

* **-sn** Permite enumerar todos los host

* **-iL** Permite analizar todas las IPs que se encuentren guardadas en un archivo

* **-p-** Permite buscar todos los puertos de un host (1 - 65535)

* **-T** Permite cambiar las velocidades de escaneo, el intervalo es 0 a 5.

* **-O** Permite identificar el Sistema Operativo

* **--min-rate** Tasa mínima de paquetes que va a enviar nmap en su escaneo. Lo recomendado entre 3000 a 5000

* **-sV** Enumerar versiones de servicios que corren en cada puerto. Es un comando muy intrusivo.

* **-Pn** Permite identificar si un equipo está activo en la red SIN NECESIDAD DE HACER PING.

* **-sS** HAce conexión incompleta de lo que se conoce como Tree HAnk Shake para que el escaneo sea rápido.

* **-n** No hace una relsolución DNS, evita que se pueda hacer una redirección a dominios públicos.

* **-sC** Utiliza los scripts por defecto de nmap para enumerar más información. Los script se pueden encontrar en la ruta __**/usr/share/nmap/scripts**__. Los scripts que tengan la categoría **default** Son los que aplica al escaneo.

**--script** Aquí se puede detallar los scripts por serparado. Ejemplo --script "vuln,exploit".
* smb-protocols
* smb-security-mode
* smb-enum-sessions
* smb-enum-shares
* smb-enum-users
* smb-enum-groups
* smb-enum-domains
* smb-enum-services
* smb-enum-shares,smb-ls  #Listar contenido de los recursos compartidos.

**--script-args** Funciona con el comando anterior de scripts y permite pasar argumentos
```shell
sudo nmap -p445 --script smb-enum-users --script-args smbusername=vagrant,smbpassword=vagrant 192.168.100.26
sudo nmap -p445 --script smb-shares --script-args smbusername=vagrant,smbpassword=vagrant 192.168.100.26
```
**smbclient** Conectarse a recursos compartidos encontrados

**evil-winrm** Permite conectarse a cuando se encuentra un servicio winrm abierto y genera una shell super cómoda e interactiva.

```shell
evil-winrm -i 192.168.100.26 -u username -p password 
```

**-F** Escaneo de los puertos más comúnes

**-top-ports** Escaneo de los 100 puertos más comunes

**-v** Verbosito para saber el historial de lo que va haciendo el escaneo.

**-o** Permite descargar en varios formatos el resultado.
* -oX Permite descargar el resultado a formato xml
* -oN Permite descargar el resultado a formato normal
* -oG Permite descargar el resultado a formato grefeable.

**-p U:1-200** Escanea los puertos del 1 al 200 pero que sean UDP
```shell
sudo nmap -p- -T5 -iL hosts.txt
sudo nmap -p- -O --min-rate 5000 -iL hosts.txt
sudo nmap -Pn -p80,21,22,139,3389 192.168.100.126 --open #Permite saber si el puerto está abierto sin necesidad de un ping.
sudo nmap -p21,22,80,139,445,8080,8282 -sS -n --min-rate 3000 -sV-sC 192.168.100.26 -oN HostDiscovery.txt #Es recomendable aplicar todas estas opciones solo a los puertos escaneados para que el tiempo no sea muy alto al escanear todos los puertos

```
En redes locales, no hay problema si se hace mucho ruido con los parámetros de velocidad o envío de paquetes, al subir mucho la tasa, se puede alterar los resultados y nos puede arrojar falsos positivos o resultados erroneos.

### Formas de conocer si un equipo es windows o linux.
Para ello se puede realizar un ping y analizar el valor del TTL

```shell
└─$ ping -c 1 192.168.100.18
PING 192.168.100.18 (192.168.100.18) 56(84) bytes of data.
64 bytes from 192.168.100.18: icmp_seq=1 ttl=64 time=6.17 ms

--- 192.168.100.18 ping statistics ---
1 packets transmitted, 1 received, 0% packet loss, time 0ms
rtt min/avg/max/mdev = 6.170/6.170/6.170/0.000 ms
```

Con ese resultado se puede intuir los siguientes TTL
* **TTL=64** -> Corresponde a un equipo Linux
* **TTL=128** -> Corresponde a un equipo Windows
* **TTL=255** -> Corresponde a equipos de red como los routers ya que necesitan la máxima conectividad.

## Uso de NMAP con Metasploit
Permite importar los resultados de nmap a metasploit siguiendo los siguientes pasos
```shell
sudo nmap -sS --script "vuln,exploit" -p21,22,80,8585,139,445,8080 --min-rate 5000 192.168.100.26 -oX test.xml
service postgresql start  #Iniciar servicio de Postgress Sql
sudo msfdb init           #Iniciar la base de datos de msfconsole
msfconsole -q             #Arrancamos metasploit
db_status                 #Verificamos que la base este conectada
help workspace            #Manual de Workspace
workspace                 #Para consultar los espacios de trabajo que hay
workspace -d nombre       #Permite eliminar un workspace creado previamente
workspace -a nombre       #Permite crear un nuevo workspace. Es recomendable 1 por máquina
workspace nombre          #Permite seleccionar un espacio para trabajar
db_nmap -p21,22,80,8585,139,445 --script smb-vuln* -sV -O -T5 192.168.100.26
hosts                     #Permite revisar los equipos escaneados
services                  #Muestra los servicios que ya se han escaneado
vulns                     #Muestra las vulenerabilidades
db_import /home/kali/Desktop/test.xml #Importa un resultado hecho con nmap y se puede seguir importando varios archivos.
exit
```

Cuando se escaneen puertos, se debe empezar por explotar los más comunes. Entre los más comunes tenemos los siguientes:
1. **21 FTP** Permite conectarse sin credenciales o con autenticación anónima. (anonymous-anounymous)
2. **80 HTTP** Enumeración del servicio Web, (CSM - Gestor de contenidos, Windows IIS, Apache(Wordpress, Drupal, etc)) - Encontrar vulnerabilidades de SQLi, rutas con archivos secretos.
2. **139,445 SMB** Samba - Recursos compartidos (SMBClient)
3. **3306 MYSQL** Servidor de base de datos (Post Explotación)
4. **22 SSH** 
