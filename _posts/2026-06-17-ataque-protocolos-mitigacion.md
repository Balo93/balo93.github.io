---
title: "Ataque a protocolos de red y mitigación"
date: 2026-06-17 22:15:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [pentesting, frameworks, compliance, consulting, tryhackme]
image:
  path: /assets/img/Primer-Blog/ataque.png
  alt: "Protocolos y Servicios"
series: Metodologías de Pentesting de TryHackMe
---

# 🕵️‍♂️ Análisis Profundo: Sniffing, MITM y Ataques de Contraseña

En esta sección de nuestra preparación, profundizaremos en la mecánica de los tres vectores de ataque más comunes a nivel de red, sus escenarios de riesgo en el ecosistema moderno y las contramedidas criptográficas esenciales para neutralizarlos.

---

## 1. Sniffing Attacks (Intercepción de Tráfico)

El **Sniffing** consiste en el uso de herramientas de captura de paquetes para recolectar información en tránsito. Cuando un protocolo opera en texto claro (cleartext), cualquier tercero con acceso al segmento de red puede leer los datos confidenciales.

### Escenarios de Riesgo Actuales
A pesar de la adopción masiva de TLS, el sniffing sigue siendo crítico en:
* **Redes corporativas internas:** Donde el tráfico este-oeste frecuentemente carece de cifrado.
* **Sistemas Legacy e Industriales:** Servidores de correo antiguos, sistemas de control industrial (ICS) y dispositivos embebidos.
* **Dispositivos IoT:** Que suelen priorizar el rendimiento sobre el cifrado de sus comunicaciones.
* **Ataques de Degradación:** Escenarios post-MITM donde se remueve la capa segura.

### Herramientas de Captura Esenciales
* **Tcpdump:** Herramienta CLI ligera, por defecto en sistemas Linux. Requiere privilegios de `root`.
* **Wireshark:** Analizador de protocolos con interfaz gráfica (GUI), ideal para filtrado avanzado y disección profunda.
* **Tshark:** La alternativa por línea de comandos de Wireshark, optimizada para automatización y scripting.

### Filtros Prácticos de Tcpdump para Auditorías
```bash
# Capturar tráfico POP3 en formato ASCII
sudo tcpdump port 110 -A

# Filtrar tráfico por un host específico
sudo tcpdump host 10.20.30.148 -A

# Guardar la captura en un archivo pcap para análisis posterior
sudo tcpdump -w captura.pcap

# Leer un archivo pcap guardado
tcpdump -r captura.pcap -A
```

## 2. Ataques Man-in-the-Middle (MITM)

Un ataque MITM ocurre cuando un atacante se interpone activamente entre la víctima y el destino legítimo, alterando o interceptando el flujo de datos sin que ninguna de las partes lo note.

### Técnicas de Redirección de Tráfico

| Técnica | Descripción / Mecanismo |
| :--- | :--- |
| **ARP Spoofing** | Envenenamiento de las tablas ARP locales para asociar la dirección MAC del atacante con la IP de la puerta de enlace (Gateway). |
| **DNS Spoofing** | Inyección de respuestas falsas de DNS para redirigir el tráfico a servidores controlados por el atacante. |
| **Rogue Access Points** | Puntos de acceso Wi-Fi falsos con nombres legítimos (ej. "Free_Airport_Wi-Fi"). |
| **BGP Hijacking** | Alteración de rutas a nivel de enrutamiento de Internet (AS), usualmente dirigido a organizaciones específicas. |

### Mecanismos de Ataque sobre Tráfico Cifrado

* **SSL Stripping:** Degradación forzada de conexiones HTTPS a HTTP. El atacante mantiene el enlace HTTPS con el servidor pero le entrega texto plano a la víctima.
* **Certificados Falsos:** Presentación de certificados inválidos esperando que el usuario ignore la advertencia del navegador, o mediante la vulneración de una Autoridad de Certificación (CA).

### El Arsenal MITM

| Herramienta | Propósito Principal |
| :--- | :--- |
| **Bettercap** | El framework moderno, modular y mantenido para auditorías de red y ataques MITM en entornos LAN. |
| **mitmproxy** | Proxy interactivo especializado en la interceptación y modificación de tráfico HTTP/HTTPS. |
| **Responder** | Herramienta crítica en entornos Windows/Active Directory que envenena protocolos de resolución de nombres locales (LLMNR, NBT-NS) para capturar hashes. |

---

## 3. Evolución del Cifrado: El Ecosistema TLS

Para mitigar el sniffing y los ataques MITM en la capa de presentación, la implementación estricta de Transport Layer Security (TLS) es obligatoria.

### Cronología de Protocolos

| Protocolo | Estado | Observación de Seguridad |
| :--- | :---: | :--- |
| **SSL 2.0 / 3.0 & TLS 1.0 / 1.1** | 🚫 Obsoleto | Completamente obsoletos y vulnerables. Deben deshabilitarse por cumplimiento (*compliance*). |
| **TLS 1.2** | ✅ Seguro | Ampliamente utilizado, seguro si se configura exclusivamente con *cipher suites* modernas. |
| **TLS 1.3** | 🚀 Estándar | Reduce la latencia del protocolo (1-RTT), elimina algoritmos débiles y fuerza *Forward Secrecy* por defecto. |

### Implicit TLS vs. STARTTLS

* **Implicit TLS:** El cifrado se negocia de forma inmediata en un puerto dedicado (ej. HTTPS en el 443, IMAPS en el 993). Es la opción más segura.
* **STARTTLS:** La conexión inicia en texto claro en el puerto estándar y se eleva a TLS mediante un comando del cliente. Vulnerable a ataques de degradación (*strip attacks*) si no se fuerza su obligatoriedad.

---

## 4. Ataques contra Mecanismos de Contraseña

Los ataques contra el factor de autenticación "algo que sabes" han evolucionado drásticamente gracias a la masificación de bases de datos de credenciales filtradas.

### Tipos Modernos de Ataque

| Tipo de Ataque | Descripción |
| :--- | :--- |
| **Credential Stuffing** | Automatización del uso de pares usuario/contraseña filtrados en brechas previas contra múltiples servicios, explotando la reutilización. |
| **Password Spraying** | Intento de usar 1 o 2 contraseñas muy comunes (ej. `Welcome2026!`) contra miles de cuentas simultáneamente, evadiendo bloqueos (*lockout*). |
| **Diccionario y Híbridos** | Uso de listas optimizadas (`SecLists`, `RockYou`) combinadas con reglas de sustitución basadas en patrones de comportamiento humano. |

### Sintaxis Avanzada en Hydra para Servicios Comunes

```bash
# Ataque de fuerza bruta contra FTP usando el diccionario RockYou
hydra -l mark -P /usr/share/wordlists/rockyou.txt 10.66.164.238 ftp

# Ataque de fuerza bruta contra SSH para un usuario específico
hydra -l frank -P /usr/share/wordlists/rockyou.txt 10.66.164.238 ssh

# Ataque de Credential Stuffing (Lista de usuarios + Lista de contraseñas)
hydra -L users.txt -P passwords.txt 10.66.164.238 ssh
```

### 🛠️ Opciones Útiles de Hydra (Useful Hydra Options)

Para realizar auditorías de contraseñas y simular ataques de fuerza bruta o diccionario, `hydra` es una de las herramientas más robustas. A continuación, se detallan sus opciones principales:

| Opción | Descripción |
| :--- | :--- |
| `-l username` | Atacar un único nombre de usuario. |
| `-L users.txt` | Archivo que contiene una lista de nombres de usuario (wordlist). |
| `-p password` | Probar una única contraseña. |
| `-P wordlist.txt`| Archivo que contiene una lista de contraseñas (wordlist). |
| `-s PORT` | Especificar un puerto que no sea el predeterminado del servicio. |
| `-V` o `-vV` | Salida detallada (Verbose) que muestra los intentos en tiempo real. |
| `-t n` | Número de conexiones paralelas simultáneas (hilos/threads). |
| `-d` | Modo de depuración (Debug) para la resolución de problemas. |
| `-f` | Detener el ataque inmediatamente después de encontrar la primera contraseña válida. |
| `-w n` | Tiempo de espera (Wait time) en segundos entre conexiones. |

### Contramedidas Modernas y Defensa en Profundidad
La seguridad robusta exige la combinación de controles técnicos en múltiples capas:

- [x] **HSTS (HTTP Strict Transport Security):** Cabecera web que instruye al navegador a comunicarse estrictamente vía HTTPS, anulando los ataques de SSL Stripping.
- [x] **Certificate Transparency (CT):** Auditoría pública obligatoria de certificados que previene la emisión no detectada de certificados fraudulentos.
- [x] **Autenticación Basada en Llaves (SSH):** Inhabilitar por completo `PasswordAuthentication` en el servidor SSH y exigir llaves criptográficas robustas (ej. Ed25519 o RSA de mínimo 4096 bits).
- [x] **Arquitectura Zero Trust y Segmentación:** Tratar las redes internas como hostiles, forzando cifrado de extremo a extremo e implementando autenticación basada en puertos (802.1X).
- [x] **Autenticación Passwordless:** Transición hacia estándares como FIDO2 / WebAuthn (Passkeys) o llaves físicas de seguridad para eliminar por completo el vector de ataque basado en contraseñas tradicionales.

---

💬 ¿Qué opinas? ¿Cuál es el error de configuración de protocolos que más te encuentras durante tus auditorías o laboratorios? ¡Déjalo en la caja de comentarios!