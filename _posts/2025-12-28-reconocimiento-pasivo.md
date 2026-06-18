---
title: "Reconocimiento Pasivo: El Arte de Investigar sin ser Visto"
date: 2025-12-28 12:34:00 -0500
categories: [Hacking, Reconocimiento]
tags: [osint, pasivo, footprinting, shodan, dns, tryhackme]
description: Guía estratégica para realizar una recolección de información (OSINT) efectiva y ética.
image:
  path: /assets/img/Primer-Blog/Passive.jpg
  alt: Reconocimiento Pasivo
toc: true
---

## ¿Qué es el Reconocimiento Pasivo?

El **reconocimiento pasivo** consiste en recolectar información pública disponible sobre un sistema u organización sin interactuar directamente con su infraestructura. Se basa en metodologías **OSINT** (Open Source Intelligence).

A diferencia del reconocimiento activo, aquí **no enviamos paquetes directos al servidor**, lo que nos permite operar sin levantar sospechas en los sistemas de detección (IDS).

### Objetivos Principales
1. **Infraestructura de Red:** Rangos de IP y Registros DNS.
2. **Identidad Digital:** Dominios, subdominios y registros WHOIS.
3. **Capital Humano:** Correos, redes sociales y organigramas.
4. **Stack Tecnológico:** CMS, frameworks y firewalls (WAF).

---

## Toolkit Esencial

### 🌐 Infraestructura y Dominios
* **[Whois](https://who.is/)**: Identificación de propietarios y fechas de registro de dominios.
* **[curl](https://curl.se/)**: Permite consultar servicios de registro modernos. Se puede auditar el protocolo RDAP de forma automatizada en la terminal devolviendo la respuesta en formato JSON:  
  `curl -s https://rdap.verisign.com/com/v1/domain/tryhackme.com | jq .`
* **[RDAP Lookup](https://lookup.icann.org/en/lookup)**: Búsqueda moderna de información de registro centrada en el protocolo RDAP.
* **[Whoxy](https://www.whoxy.com)**: Historial y capturas cronológicas de registros WHOIS (con uso gratuito limitado).
* **[DNSDumpster](https://dnsdumpster.com/)**: Mapeo visual de registros DNS y enumeración de subdominios.
* **[Crt.sh](https://crt.sh/)**: Búsqueda pasiva en registros de Certificados de Transparencia (CT Logs) para descubrir subdominios mediante SSL/TLS.
* **[OWASP Amass](https://github.com/owasp-amass/amass)**: Herramienta de alta potencia para el descubrimiento de activos. Soporta flujos de recolección tanto pasivos como activos.

### 🔍 Inteligencia de Servicios y Dispositivos
* **[Shodan](https://www.shodan.io)**: El motor de búsqueda por excelencia para dispositivos expuestos e infraestructura conectada (IoT).
* **[Censys](https://search.censys.io/)**: Excelente motor de búsqueda de hosts que sirve como complemento directo a Shodan.
* **[Wafw00f](https://github.com/EnableSecurity/wafw00f)**: Escáner automatizado para identificar la presencia y el tipo de Firewall de Aplicación Web (WAF).
* **[Wayback Machine](https://web.archive.org/)**: Archivo digital para acceder a versiones antiguas de un sitio web, útil para mapear endpoints o archivos indexados en el pasado.

### 📧 Inteligencia de Datos y Correos
* **[theHarvester](https://github.com/laramies/theHarvester)**: Herramienta clásica de consola para la extracción masiva de correos, nombres de empleados, subdominios y hosts virtuales utilizando múltiples fuentes OSINT de manera simultánea.
* **[Hunter.io](https://hunter.io)**: Plataforma para la identificación de correos corporativos y descubrimiento de patrones de nomenclatura organizacional.

### 🛠️ Análisis de Tecnologías
* **[Wappalyzer](https://www.wappalyzer.com/) / [BuiltWith](https://builtwith.com/) / [WhatWeb](https://whatweb.net/)**: Extensiones y utilidades que identifican el backend, CMS, servidores web y librerías de JavaScript en ejecución.
* **[Google Hacking Database](https://www.exploit-db.com/google-hacking-database)**: Repositorio de operadores avanzados (Google Dorks) para hallar fugas de información y archivos expuestos en índices públicos.

---

## ⚡ Categoría: Herramientas Adicionales y Especializadas
Para consultas profundas de bajo nivel y automatización técnica en la terminal, estas utilidades son indispensables:

* **[Dig](https://linux.die.net/man/1/dig)**: Utilidad de línea de comandos para realizar consultas exhaustivas y personalizadas a servidores DNS (Registros A, MX, NS, TXT).  
  **Sintaxis:** `dig [@SERVER] DOMAIN_NAME [TYPE]`
* **[DNS Recon](https://www.kali.org/tools/dnsrecon/)**: Script enfocado en la enumeración avanzada de DNS; realiza comprobaciones de registros SRV y detecta configuraciones erróneas que permitan transferencias de zona de forma pasiva.
* **[Sublist3r](https://github.com/aboul3la/Sublist3r)**: Framework optimizado en Python para enumerar subdominios consultando múltiples motores de búsqueda de manera paralela.
* **[HTTrack](https://github.com/xroche/httrack)**: Clonador de sitios web. Descarga la estructura HTML y multimedia a un entorno local para analizar código fuente y variables sin interactuar de forma continua con el backend del objetivo.
* **[Netcraft sitereport](https://sitereport.netcraft.com/)**: Proveedor de reportes detallados sobre el historial de direccionamiento IP, ASN, bloques de red y hosting de un dominio.
* **[Nslookup](https://www.nslookup.io/)**: Herramienta multiplataforma para interrogar servidores de nombres DNS especificando tipos de registros.  
  **Sintaxis:** `nslookup -type=TYPE DOMAIN_NAME [SERVER]`

### Tipos de Registros DNS Comunes

| Tipo de Consulta | Resultado Obtenido |
| :--- | :--- |
| **A** | Dirección o direcciones IPv4 vinculadas al dominio principal. |
| **AAAA** | Dirección o direcciones IPv6 asociadas al host. |
| **CNAME** | Nombre canónico: un alias que apunta un nombre de dominio hacia otro. |
| **MX** | Servidores de intercambio de correo encargados de gestionar la mensajería del dominio. |
| **SOA** | Inicio de autoridad: expone el servidor de nombres primario, correo del administrador y serial de zona. |
| **TXT** | Registros de texto arbitrario, empleados para configuraciones de seguridad como SPF, DKIM y DMARC. |

---

## El Proceso de Footprinting (Huella Digital)

El **Footprinting** es donde unimos todas las piezas. Al analizar un sitio web, hay dos archivos que nunca debes ignorar:
* **`robots.txt`**: Revela rutas que la empresa prefiere no indexar (ej: `/config`, `/tmp`).
* **`sitemap.xml`**: El mapa organizado de la web, ideal para descubrir páginas poco visibles.

> **Aviso de Seguridad:** Realiza tus investigaciones bajo un marco ético. El acceso no autorizado a sistemas restringidos es ilegal.
{: .prompt-warning }

---

### Casos Prácticos de Reconocimiento Pasivo

Para entender el alcance real del Footprinting, analicemos dos escenarios comunes utilizando herramientas de fuentes abiertas (OSINT) que simulan infraestructuras reales, basándonos en entornos controlados de entrenamiento:

#### 1. Descubrimiento de Subdominios Ocultos (DNSDumpster)
Las plataformas de mapas DNS como **DNSDumpster** permiten auditar registros históricos y subdominios que muchas veces quedan huérfanos o desconfigurados en las organizaciones.

Un caso típico ocurre al investigar un dominio objetivo; más allá de los registros estándar como `www` o `blog`, la enumeración pasiva puede revelar subdominios críticos como:
* **`remote.dominio.com`**: Muy codiciado en auditorías, ya que suele apuntar a accesos de teletrabajo, terminales de mantenimiento o redes de acceso VPN corporativo.
* **`cancel.dominio.com`**: Subdominios asociados a pasarelas de pago inactivas o flujos de suscripción que exponen arquitecturas en la nube (ej: instancias en AWS).

> **Nota de Pentester:** En escenarios de auditoría reales, los subdominios que apuntan a servicios de terceros dados de baja pero cuyos registros DNS siguen activos son candidatos perfectos para ataques de toma de control (*Subdomain Takeover*).
{: .prompt-info }

#### 2. Auditoría de Infraestructura con Shodan (Filtros Avanzados)
Cuando un servidor no responde a escaneos directos o se encuentra protegido detrás de proxies inversos globales, los motores de búsqueda de infraestructura como **Shodan** permiten interrogar pasivamente los banners de los servicios ya indexados.

Para optimizar la búsqueda y extraer métricas directas sin necesidad de procesar payloads masivos, podemos emplear filtros avanzados combinando operadores de host y producto directamente en el cajón de búsqueda:

```Plaintext
hostname:objetivo.com product:"nginx"
```
---

#### 3. Extracción de Métricas con Shodan Facets
Si necesitas identificar rápidamente vectores de ataque mediante el análisis de puertos más utilizados en una infraestructura específica, puedes forzar a la interfaz web a desglosar los Facets estadísticos directamente desde la URL de tu navegador:
```text
[https://www.shodan.io/search/facet?query=hostname%3Aobjetivo.com+product%3A%22nginx%22&facet=port](https://www.shodan.io/search/facet?query=hostname%3Aobjetivo.com+product%3A%22nginx%22&facet=port)
```

O automatizarlo desde la consola de Kali de manera profesional utilizando la Shodan CLI:

```bash
shodan stats --facets port hostname:objetivo.com
```

Este comando agrupa los resultados y devuelve una matriz limpia ordenada por frecuencia de uso, permitiéndote correlacionar de inmediato que puertos como el 443 (HTTPS) manejan certificados SSL específicos, o si existen servicios alternativos expuestos de forma insegura.

### Guía Rápida de Comandos para Reconocimiento Pasivo

Durante las primeras fases de recolección de información, interactuar con registros públicos y servidores DNS es fundamental. A continuación, se detalla una tabla de referencia rápida con las herramientas esenciales (`whois`, `nslookup`, `dig`) y su implementación práctica:

| Objetivo | Ejemplo de Línea de Comandos |
| :--- | :--- |
| **Consultar registro WHOIS** | `whois tryhackme.com` |
| **Buscar un registro DNS (Legado)** | `nslookup -type=A tryhackme.com` |
| **Consultar registros DNS MX en un servidor específico (Heredado)** | `nslookup -type=MX tryhackme.com 1.1.1.1` |
| **Buscar registros DNS TXT (Heredado)** | `nslookup -type=TXT tryhackme.com` |
| **Consultar los registros DNS A (Recomendado)** | `dig tryhackme.com A` |
| **Buscar registros DNS MX en un servidor específico (Recomendado)** | `dig @1.1.1.1 tryhackme.com MX` |
| **Consultar registros DNS TXT (Recomendado)** | `dig tryhackme.com TXT` |
| **Descubrimiento pasivo de subdominios (Navegador)** | Visita [crt.sh](https://crt.sh) y busca `%.tryhackme.com` |

> **Tip de Consola:** Aunque las herramientas heredadas como `nslookup` siguen operativas en la mayoría de distribuciones, se recomienda priorizar el uso de `dig` debido a su flexibilidad, claridad en la salida de datos y manejo nativo de consultas avanzadas en auditorías de seguridad.
{: .prompt-tip }

## Conclusión

El **reconocimiento pasivo** no es solo recolectar enlaces; es construir el rompecabezas de la superficie de ataque de un objetivo. La clave del éxito en esta fase es la **paciencia** y la **metodología**. Recuerda que cada dato, por pequeño que parezca (un correo, una versión de servidor o un subdominio olvidado), puede ser la puerta de entrada en fases posteriores.

## ¿Qué sigue?
Una vez que tenemos el mapa completo del objetivo sin haber interactuado con él, el siguiente paso es el **Reconocimiento Activo (Scanning)**. En la próxima entrega, veremos cómo usar herramientas como **Nmap** para validar qué servicios están realmente vivos y qué puertos están abiertos.

---

### 💬 ¡Participa!
¿Cuál de estas herramientas es tu favorita para empezar una investigación OSINT? Si conoces alguna otra herramienta imprescindible que no esté en la lista, ¡déjala en los comentarios!

> **Nota:** Este contenido tiene fines educativos. El conocimiento de estas herramientas debe usarse siempre dentro de marcos legales y éticos.