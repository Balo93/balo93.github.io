---
title: "Metodologías de Pentesting: ¿Por qué no basta con saber atacar?"
date: 2026-06-05 02:39:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [pentesting, frameworks, compliance, tryhackme]
series: Metodologías de Pentesting de TryHackMe
img_path: /assets/img/posts/
---

Cuando empezamos en el mundo del hacking, es común pensar que todo se reduce a lanzar exploits, encontrar vulnerabilidades y conseguir acceso (obtener nuestra ansiada *shell*). Sin embargo, en el ámbito profesional y corporativo, **un pentester no es un atacante caótico, es un auditor.**

Imagina a un inspector de edificios: él no camina por los pasillos esperando "tener la suerte" de ver una grieta. Sigue una lista de control estricta para evaluar la estructura, la electricidad y los sistemas contra incendios bajo un estándar conocido. En ciberseguridad, los **Frameworks de Pentesting** son exactamente eso: nuestra guía de control.

---

## ¿Por qué es vital usar una metodología estructurada?

Confiar en un proceso estandarizado nos aporta ventajas críticas que diferencian a un aficionado de un profesional:

* **Exhaustividad:** Garantiza que no pases por alto áreas críticas del sistema por centrarte solo en lo que te llama la atención.
* **Consistencia:** Permite que si dos testers diferentes evalúan el mismo entorno, obtengan resultados comparables y replicables.
* **Cumplimiento (Compliance):** Alinea la auditoría con los requisitos legales y regulatorios que la empresa está obligada a cumplir (como proteger datos médicos o bancarios).
* **Confianza y Comunicación:** Los clientes, auditores y directivos entenderán y confiarán en tu reporte porque está respaldado por un estándar internacional reconocido.

---

## El Mapa de los Frameworks de Pentesting

A continuación, se presenta un mapa general de las principales metodologías que dominan la industria actual, clasificadas según su filosofía y casos de uso ideales:

### 1. Metodologías de Propósito General y Ejecución
* **PTES (Penetration Testing Execution Standard):** Es un estándar muy práctico impulsado por fases que refleja con exactitud cómo se lleva a cabo un compromiso en el mundo real (desde la interacción inicial con el cliente hasta el reporte y la post-explotación).
* **NIST SP 800-115:** La guía técnica del gobierno de EE. UU. para pruebas y evaluaciones de seguridad. Es el estándar de oro en entornos federales y corporativos rígidos.
* **OSSTMM (Open Source Security Testing Methodology Manual):** Un enfoque científico y métrico. No solo busca fallos, sino que mide operativamente la seguridad mediante análisis cuantificables.
* **ISSAF:** Una metodología con un gran peso histórico basada en un modelo de evaluación detallado de 9 pasos.

### 2. Guías Específicas por Entorno (Aplicaciones y Datos)
* **OWASP WSTG (Web Security Testing Guide):** El marco de referencia absoluto e imprescindible para auditorías de aplicaciones web.
* **OWASP MASTG (Mobile Application Security Testing Guide):** La guía especializada para el análisis e ingeniería inversa de aplicaciones móviles (Android/iOS).
* **WASC Threat Classification:** Una taxonomía detallada para clasificar debilidades y amenazas en sitios web.
* **PCI DSS Penetration Testing Guidelines:** Directrices obligatorias para empresas que procesan, almacenan o transmiten datos de tarjetas de crédito.
* **CSA Cloud Controls Matrix:** Diseñado específicamente para evaluar los riesgos en la infraestructura de la nube.

### 3. Marcos Complementarios de Inteligencia de Amenazas
* **MITRE ATT&CK:** No es un framework de ejecución de pentesting tradicional, sino una base de conocimiento viva que mapea las tácticas, técnicas y procedimientos (TTPs) reales que utilizan los cibercriminales en el mundo real. Es el complemento perfecto para simular adversarios reales durante una prueba.
* **CBEST Framework:** Framework enfocado en la resiliencia cibernética mediante simulaciones de ataques guiados por inteligencia (Threat Intelligence-Led Assessments), muy usado en el sector financiero.

---

## Conclusión

El hacking técnico es la herramienta, pero **la metodología es el mapa**. Saber qué framework aplicar en cada escenario (por ejemplo, usar *WSTG* para una tienda en línea o *PCI DSS* si esa tienda procesa pagos) es lo que convierte un reporte de vulnerabilidades en un activo de valor estratégico para cualquier cliente.

---

## Contenido de la Serie (9 Artículos):

Aquí puedes navegar por cada una de las entregas detalladas de esta serie metodológica:

* **Vol. 0:** [Metodologías de Pentesting: ¿Por qué no basta con saber atacar?](/posts/1introduccion-metodologias) *(Este post)*
* **Vol. 1:** [OSSTMM: El enfoque científico y matemático del Pentesting](/posts/2osstmm-framework/)
* **Vol. 2:** [OWASP WSTG: La guía definitiva para auditorías Web profesionales](/posts/3owasp-wstg/)
* **Vol. 3:** [NIST SP 800-115: El estándar institucional y gubernamental de evaluación](/posts/4nist-sp-800-115/)
* **Vol. 4:** [PTES: El mapa de ruta real para auditorías de principio a fin](/posts/5ptes-framework/)
* **Vol. 5:** [ISSAF: Lecciones de una metodología clásica y el modelo de 9 pasos](/posts/6issaf-framework/)
* **Vol. 6:** [MITRE ATT&CK: El traductor universal del comportamiento adversario](/posts/7mitre-attack/)
* **Vol. 7:** [Frameworks Especializados: Cuando el contexto define las reglas del juego](/posts/8frameworks-especializados/)
* **Vol. 8:** [Criterios de Selección: ¿Qué framework aplicar en cada auditoría?](/posts/9criterios-seleccion/)