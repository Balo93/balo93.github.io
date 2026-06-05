---
title: "Frameworks Especializados: Cuando el contexto define las reglas del juego"
date: 2026-06-05 02:10:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [mastg, pci-dss, csa, compliance, frameworks, tryhackme]
series: Metodologías de Pentesting de TryHackMe
---

Hasta ahora hemos analizado los frameworks core que todo tester junior o analista va a encontrar en su camino corporativo. Sin embargo, el panorama de la ciberseguridad no se detiene ahí. Dependiendo de la industria, la plataforma técnica o el entorno regulatorio en el que opere tu cliente, las metodologías generales pueden quedarse cortas. 

En esta entrega de la serie, haremos un recorrido por **cinco frameworks especializados**. El objetivo aquí no es dominarlos a fondo de inmediato, sino desarrollar **consciencia del entorno**: saber que existen, entender su nicho exacto y reconocer cuándo un contrato exige que uses uno de ellos.

---

## El Mapa de los Frameworks Especializados

Como se aprecia en la matriz de la imagen `image_b61703.png`, cada una de estas herramientas atiende un dominio y una necesidad completamente única:

| Framework | Dominio Principal | Tipo | ¿Activo? | ¿Cuándo se utiliza? |
| :--- | :--- | :--- | :--- | :--- |
| **WASC Threat Classification** | Aplicaciones Web | Taxonomía de amenazas | No *(Superado por OWASP)* | Referencias heredadas y contexto histórico. |
| **CSA Cloud Controls Matrix** | Entornos Cloud | Controles de Gobierno/Cumplimiento | Sí | Evaluaciones de la postura de seguridad en la nube. |
| **OWASP MASTG** | Apps Móviles (Android/iOS) | Guía de pruebas técnicas | Sí | Pentesting de aplicaciones móviles. |
| **PCI DSS Guidelines** | Entornos de Tarjetas de Pago | Mandato regulatorio obligatorio | Sí *(PCI DSS v4.0)* | Cualquier auditoría que involucre datos de tarjetas. |
| **CBEST Framework** | Sector Financiero del Reino Unido | Pentesting guiado por Inteligencia | Sí | Contratos con instituciones financieras en el Reino Unido. |

---

## Zoom a cada Metodología: ¿Cuál es su nicho?

### 1. WASC Threat Classification
Creado por el *Web Application Security Consortium*, nació como una taxonomía para categorizar vulnerabilidades y tipos de ataque en la web (dividiéndolos en áreas como fuga de información o abuso de funcionalidad). Aunque fue muy influyente a mediados de los 2000 para estandarizar el lenguaje de la industria, hoy en día ha sido reemplazado casi por completo por el OWASP Top 10 y el WSTG. Lo encontrarás únicamente en documentación antigua o marcos de cumplimiento heredados.

### 2. CSA Cloud Controls Matrix (CCM)
Publicado por la *Cloud Security Alliance*, este marco de controles está diseñado específicamente para la computación en la nube. Mapea controles estructurados en 17 dominios (como seguridad de datos, gestión de identidades y accesos, y seguridad de infraestructura) alineándolos con estándares globales como ISO 27001 o PCI DSS. 
> **Nota clave:** El CCM **no es una metodología de pentesting**. Es una herramienta de gobernanza que te servirá cuando un cliente te pida evaluar su *postura de seguridad en la nube* en lugar de ejecutar una intrusión tradicional.

### 3. OWASP MASTG (Mobile Application Security Testing Guide)
Es la contraparte móvil absoluta del WSTG web. Mantenida activamente por OWASP, provee casos de prueba técnicos y minuciosos para la seguridad en aplicaciones Android e iOS, evaluando el almacenamiento local de datos, implementaciones criptográficas, comunicaciones de red y la calidad del código. Si el alcance de tu proyecto incluye una app de banca móvil o un portal médico en un teléfono inteligente, este es tu manual de cabecera obligatorio, el cual trabaja en conjunto con el estándar de requisitos **MASVS**.

### 4. PCI DSS Penetration Testing Guidelines
A diferencia de las guías de buenas prácticas, aquí estamos ante un **mandato regulatorio por ley** (específicamente bajo el Requisito 11.4 de PCI DSS v4.0). Cualquier empresa que procese, almacene o transmita datos de tarjetas de crédito debe realizar un pentesting que cumpla estrictamente con estos criterios. La norma exige evaluar obligatoriamente el perímetro externo y la red interna al menos una vez al año (o tras cambios mayores en la infraestructura), validando además que los controles de segmentación de red funcionen de verdad. Si tu cliente vende en línea o procesa pagos, este estándar moldeará tanto el alcance de tus pruebas como el formato de tu reporte.

### 5. CBEST Framework
Desarrollado por el Banco de Inglaterra, es un enfoque de pruebas de penetración **guiado por inteligencia de amenazas (Threat-Intel-Led)**. Está hecho a la medida del sector financiero británico (bancos, aseguradoras y proveedores de infraestructura de mercado). Una auditoría CBEST empieza obligatoriamente con una fase de inteligencia que identifica exactamente qué actores de amenazas reales apuntan a esa institución y qué tácticas usan; posteriormente, el equipo ofensivo simula exactamente esos escenarios reales. Destaca por su rigurosa integración de la inteligencia de amenazas con el hacking técnico.

---

## En pocas palabras

La selección de un framework nunca debe ser una decisión al azar. Está guiada por el **contexto**: la industria del cliente, sus obligaciones legales, su infraestructura tecnológica y su jurisdicción geográfica definirán qué reglas aplicar. 

* Si trabajas con un banco del Reino Unido, tu norte es **CBEST**.
* Si evalúas una app móvil de salud, tu biblia es **OWASP MASTG**.
* Si trabajas con una pasarela de pagos, no hay forma de esquivar **PCI DSS**.

Conocer el mapa completo es lo que te permite identificar cuál de ellos demanda una auditoría específica. En la próxima y última entrega de la serie, unificaremos todos estos conocimientos para aprender a elegir la combinación de herramientas ideal en escenarios del mundo real.

---

## Contenido de la Serie (9 Artículos):

Aquí puedes navegar por cada una de las entregas detalladas de esta serie metodológica:

* **Vol. 0:** [Metodologías de Pentesting: ¿Por qué no basta con saber atacar?](/posts/1introduccion-metodologias)
* **Vol. 1:** [OSSTMM: El enfoque científico y matemático del Pentesting](/posts/2osstmm-framework/)
* **Vol. 2:** [OWASP WSTG: La guía definitiva para auditorías Web profesionales](/posts/3owasp-wstg/)
* **Vol. 3:** [NIST SP 800-115: El estándar institucional y gubernamental de evaluación](/posts/4nist-sp-800-115/)
* **Vol. 4:** [PTES: El mapa de ruta real para auditorías de principio a fin](/posts/5ptes-framework/)
* **Vol. 5:** [ISSAF: Lecciones de una metodología clásica y el modelo de 9 pasos](/posts/6issaf-framework/)
* **Vol. 6:** [MITRE ATT&CK: El traductor universal del comportamiento adversario](/posts/7mitre-attack/)
* **Vol. 7:** [Frameworks Especializados: Cuando el contexto define las reglas del juego](/posts/8frameworks-especializados/) *(Este post)*
* **Vol. 8:** [Criterios de Selección: ¿Qué framework aplicar en cada auditoría?](/posts/9criterios-seleccion/)