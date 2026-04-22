---
title: "Recomendaciones eJPT"
date: 2025-12-29 00:45:00 -0500
categories: [Hacking, eJPT]
tags: [ejpt, hacking-etico]
description: Recomendaciones para la certificación de eJPT.
toc: true
---

## Recomendaciones
> Advertencia: Entrada en construcción.
{: .prompt-warning }

Analizar todos los equipos de la red y empezar por la primera dirección encontrada.
1. Cuál es la dirección del equipo que está corriendo WordPress.
    Se debe empezar viendo los servicios que están corriendo en los puerto 80 u 8080

2. Cuántos equipos en la DMZ están corriendo windows.
3. Cuando se encuentre varios puertos abiertos, empezar por los comunmente más explotables.
4. Analizar los nombres de los equipos en toda la red con crackmapexec
5. Revisar el código fuente del código en las páginas webs, ya que algunos servicios pueden apuntar a otras páginas o direcciónes, esto se analizar agregado la ip y dominio en /etc/hosts.
6. En la Evaluación solo se utiliza dirb para reconocimiento de directorios.
7. Al hacer un escaneo de puertos, si se encuentran los puertos:   
    139 o 445 ataque de ethernalblue
    3389 ataque de bluekepp
    139 o 445 ataque de SMBClient, SMBMap, enum4linux
   Todas estas herramientas permite ver recursos compartidos.
8. No se debe realizar fuerza bruta de usuarios y contraseas, primero buscar usuarios y luego un ataque a la contrasea a exepción de Windows porque los usuarios siempre son administrator.
9. En el exámen al hacer pivoting los puertos que nos van a interesar son el 80,445,139
10. Si se encuentran credenciales para 1 equipo, debemos probarlo en los demás.
