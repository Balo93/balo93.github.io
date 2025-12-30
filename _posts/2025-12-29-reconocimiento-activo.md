---
title: "Reconocimiento Activo: El Arte de Investigar"
date: 2025-12-29 00:45:00 -0500
categories: [Hacking, Reconocimiento]
tags: [osint, activo, hacking-etico]
description: Guía estratégica para realizar una recolección de información (OSINT) efectiva y ética.
---

## Reconocimiento Activo: El Arte de Investigar

El reconocimiento activo es la fase donde interactuamos directamente con el sistema objetivo para recolectar datos sobre puertos abiertos, servicios, versiones y posibles vulnerabilidades.

---

## Comandos Útiles para el Reconocimiento

En Linux existe un gran número de comandos que permiten escanear direcciones IP y servicios que tiene un servidor.

### 1. Arp-Scan
Con este comando se puede analizar todas las direcciones IP y máscaras de red que poseen los equipos que están conectados dentro de mi red local.

```shell
sudo arp-scan --localnet
```

### 2. NMAP (Network Mapper)
Es una herramienta de línea de comandos de código abierto que se utiliza para escanear direcciones IP y puertos en una red y para detectar aplicaciones instaladas.

```shell
sudo nmap -sn (subred)/máscara
#Ejemplo
sudo nmap -sn 192.168.100.0/24
```
## Parámetros Principales

* **-sn** Permite enumerar todos los host.

* **-iL** Permite analizar todas las IPs que se encuentren guardadas en un archivo.

* **-p-** Permite buscar todos los puertos de un host (1 - 65535).

* **-T** Permite cambiar las velocidades de escaneo (intervalo de 0 a 5).

* **-O** Permite identificar el Sistema Operativo.

* **--min-rate** Tasa mínima de paquetes (recomendado entre 3000 a 5000).

* **-sV** Enumerar versiones de servicios. Es un comando muy intrusivo.

* **-Pn** Identifica si un equipo está activo sin necesidad de hacer **PING**.

* **-sS** Hace una conexión incompleta (TCP SYN) para que el escaneo sea rápido.

* **-n** No hace resolución DNS, evita redirecciones a dominios públicos.

* **-sC** Utiliza los scripts por defecto de nmap (__/usr/share/nmap/scripts__).


* **-F** Escaneo de los puertos más comúnes.

* **-top-ports** Escaneo de los 100 puertos más comunes.

* **-v** Modo "verboso" para ver el progreso en tiempo real.

* **-p U:1-200** Escaneo de los puertos del 1 al 200 pero que sean UDP

## Gestión de Salidas (Output)

* **-oX**: Exporta el resultado a formato XML.

* **-oN**: Exporta el resultado a formato normal.

* **-oG**: Exporta el resultado a formato "grepeable".

## Uso de Scripts Avanzados

Se pueden detallar scripts por separado mediante --script "vuln,exploit". Algunos útiles para SMB:

* smb-protocols, smb-security-mode, smb-enum-sessions, smb-enum-services.

* smb-enum-shares, smb-enum-users, smb-enum-domains, smb-enum-groups.

* smb-ls: Listar contenido de recursos compartidos.

### Ejemplo con argumentos:
```Shell
sudo nmap -p445 --script smb-enum-users --script-args smbusername=vagrant,smbpassword=vagrant 192.168.100.26
```
## Herramientas de Conexión y Acceso
* **smbclient**: Para conectarse a recursos compartidos encontrados.

* **evil-winrm**: Permite conectarse cuando se encuentra un servicio WinRM abierto, generando una shell interactiva.
```shell
evil-winrm -i 192.168.100.26 -u username -p password 
```




**--script-args** Funciona con el comando anterior de scripts y permite pasar argumentos
```shell
sudo nmap -p445 --script smb-enum-users --script-args smbusername=vagrant,smbpassword=vagrant 192.168.100.26
sudo nmap -p445 --script smb-shares --script-args smbusername=vagrant,smbpassword=vagrant 192.168.100.26
```

## Identificación de SO mediante TTL (Time To Live)
Podemos intuir el Sistema Operativo realizando un ping y analizando el valor del TTL:
* **TTL=64** Equipo Linux
* **TTL=128** Equipo Windows
* **TTL=255** Equipos de red (Routers/Switches)
Ejemplo:
```Shell
64 bytes from 192.168.100.18: icmp_seq=1 ttl=64 time=6.17 ms # Es Linux
```

## Integración de NMAP con Metasploit
Permite importar los resultados de nmap a metasploit para una gestión centralizada:

```Shell
# 1. Escaneo y exportación
sudo nmap -sS --script "vuln,exploit" -p21,22,80,139,445 -oX test.xml 192.168.100.26

# 2. Configuración en msfconsole
service postgresql start  # Iniciar base de datos
sudo msfdb init           # Inicializar MSF
msfconsole -q             # Arrancar Metasploit

# 3. Comandos de base de datos
db_status                 # Verificar conexión
workspace -a nombre       # Crear espacio de trabajo (1 por máquina)
workspace -d nombre       # Elimina un espacio de trabajo
db_import /path/test.xml  # Importar resultados
hosts                     # Revisar equipos
services                  # Revisar servicios
```


## Estrategia de Explotación: Puertos Críticos
Al analizar puertos, se recomienda empezar por los más comunes:

1. **21 FTP**: Buscar autenticación anónima (anonymous).

2. **80 HTTP**: Enumeración web (CMS como Wordpress, SQLi, archivos secretos).

3. **139, 445 SMB**: Análisis de recursos compartidos con smbclient.

4. **3306 MYSQL**: Servidor de base de datos (Post-Explotación).

5. **22 SSH**: Acceso remoto.

> Advertencia: En redes locales, subir mucho la tasa de paquetes (--min-rate) puede generar falsos positivos o resultados erróneos.
{: .prompt-warning }

## Ejemplos de Escaneos Combinados
```shell
# Escaneo masivo desde archivo con alta velocidad
sudo nmap -p- -T5 -iL hosts.txt

# Detección de OS en red local
sudo nmap -p- -O --min-rate 5000 -iL hosts.txt

# Comprobar puertos específicos sin PING
sudo nmap -Pn -p80,21,22,139,3389 192.168.100.126 --open

# Escaneo completo recomendado para reporte (enfocado en puertos abiertos)
sudo nmap -p21,22,80,139,445,8080,8282 -sS -n --min-rate 3000 -sV -sC 192.168.100.26 -oN HostDiscovery.txt
```