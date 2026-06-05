---
title: "PTES: El mapa de ruta real para auditorías de principio a fin"
date: 2026-06-05 02:04:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [ptes, pentesting, workflow, frameworks, tryhackme]
series: Metodologías de Pentesting de TryHackMe
---

Ya hemos analizado frameworks enfocados en métricas científicas matemáticas (**OSSTMM**), en la seguridad profunda de aplicaciones web (**OWASP WSTG**) y en los estrictos estándares gubernamentales (**NIST SP 800-115**). Pero aquí hay una pregunta clave: *Si te contrataran mañana para un pentesting estándar contra una red corporativa, ¿cuál framework refleja mejor lo que harás desde la primera llamada con el cliente hasta el reporte final?*

Para la gran mayoría de los pentesters profesionales, la respuesta es el **Estándar de Ejecución de Pruebas de Penetración (PTES)**. 

Desarrollado por un grupo de practicantes e ingenieros experimentados de la industria, PTES (*pentest-standard.org*) nació con un objetivo clarísimo: definir cómo se ve un pentesting real de extremo a extremo. No se enfoca solo en qué herramientas usar, sino en **cómo fluye el servicio en el mundo profesional**.

---

## Las 7 Fases del Ciclo de Vida de PTES

PTES organiza el compromiso en siete fases cronológicas. Para ver su enorme valor práctico, imaginemos que un proveedor de salud (*MedGuard Health*) contrata a tu equipo para auditar su red corporativa y su portal de expedientes médicos:

[1. Pre-Engagement] ➔ [2. Intel Gathering] ➔ [3. Threat Modeling] ➔ [4. Vuln Analysis]
⬇
[7. Reporting]      ⮜   [6. Post-Exploitation]   ⮜    [5. Exploitation]

### 1. Interacciones Previas al Compromiso (*Pre-Engagement*)
Es todo lo que ocurre antes de tocar el teclado. Se define el alcance técnico detallado (la LAN corporativa, el portal web y las redes Wi-Fi de la sede) y se documentan las **Reglas de Compromiso (RoE)**. 
* *El toque profesional:* Con *MedGuard* acuerdas que las pruebas se harán solo por las noches para no interrumpir operaciones médicas críticas, y consigues firmar la carta de autorización legal (el famoso *"Get out of jail free card"*). PTES es sumamente minucioso aquí porque **los alcances mal definidos son la principal causa de demandas legales en nuestra profesión**.

### 2. Recopilación de Información (*Intelligence Gathering*)
Consiste en recolectar inteligencia sobre el objetivo usando técnicas pasivas y activas. 
* *Pasiva:* Extraer correos de empleados en LinkedIn, revisar logs de certificados y analizar ofertas de empleo de la empresa (por ejemplo, buscar vacantes de *"Se busca administrador Oracle 19c"* te revela de inmediato qué bases de datos usan por detrás).
* *Activa:* Escaneos de red lógicos y enumeración DNS dentro del alcance permitido.

### 3. Modelado de Amenazas (*Threat Modeling*)
Con la información obtenida, dejas de lanzar escaneos aleatorios y empiezas a pensar como un atacante real. Identificas los activos más valiosos (la base de datos de pacientes) y diseñas las rutas de ataque más lógicas: infectar un equipo de la LAN interna mediante *phishing* para pivotar, o atacar directamente el portal web.

### 4. Análisis de Vulnerabilidades (*Vulnerability Analysis*)
Búsqueda sistemática de fallos que permitan abrir los caminos diseñados en el modelado de amenazas. Se combinan escaneos automatizados con **verificación manual** para eliminar por completo los falsos positivos. En tu auditoría con *MedGuard*, descubres que el portal web corre una versión desactualizada de Apache Tomcat vulnerable a deserialización.

### 5. Explotación (*Exploitation*)
El momento de lanzar el ataque. Utilizas la vulnerabilidad de Tomcat para conseguir una *shell* en el servidor web y envías un correo de *phishing* simulado (autorizado en el contrato) para comprometer una estación de trabajo interna. PTES recalca que la explotación debe tener un propósito claro: **demostrar un impacto al negocio, no solo "romper cosas" por diversión**.

### 6. Post-Explotación (*Post-Exploitation*)
Aquí es donde se mide el verdadero peligro. Una vez dentro, determinas el valor de lo comprometido. Pivotas desde el servidor web hacia la base de datos interna para confirmar acceso de lectura a miles de expedientes médicos, o extraes credenciales en caché de la estación de trabajo para moverte lateralmente hacia un servidor financiero. 
* *La clave de PTES:* Esta fase traduce los hallazgos técnicos en **riesgo de negocio**. Decir *"conseguimos acceso a 50,000 registros médicos bajo la ley HIPAA"* tiene mil veces más impacto para la directiva que decir *"ejecutamos un exploit de Tomcat"*.

### 7. Reporte (*Reporting*)
La entrega de resultados estructurada para dos audiencias radicalmente distintas:
* **Resumen Ejecutivo:** Dirigido a la mesa directiva. Explica el riesgo financiero, legal y reputacional en un lenguaje claro, sencillo y libre de tecnicismos.
* **Reporte Técnico:** Dirigido al equipo de TI. Detalla los pasos exactos de explotación, capturas de pantalla como evidencia, hosts afectados y las recomendaciones priorizadas para parchar cada fallo.

---

## Pros y Contras de PTES

### 🟢 Lo mejor
* **Estructura ultra práctica:** Funciona como un libro de jugadas de cómo se desenvuelve un contrato de pentesting real. Es ideal para analistas que están definiendo su flujo de trabajo comercial.
* **Enfoque legal y contractual:** Su fase de *Pre-engagement* te salva de cometer errores críticos que podrían meterte en problemas legales o romper la producción de un cliente.

### 🔴 Lo difícil
* **Secciones técnicas desactualizadas:** El estándar no ha recibido actualizaciones formales en su sitio web desde hace años. Aunque la metodología y las fases siguen siendo perfectamente válidas y vigentes, las herramientas específicas que menciona están obsoletas. Debes complementar PTES con herramientas modernas.
* **Subjetividad:** Carece de las métricas numéricas exactas que te da OSSTMM; los resultados e impactos dependen mucho del criterio, experiencia y habilidad del auditor.

---

## Conclusión de la Serie

Hemos recorrido los pilares de las metodologías de seguridad:
* **OSSTMM** te da la ciencia y la matemática.
* **OWASP WSTG** te da la profundidad en el código web.
* **NIST SP 800-115** te da el respaldo institucional y formal para auditorías estatales.
* **PTES** te da el plano de construcción de cómo trabajar y ejecutar un proyecto de pentesting de inicio a fin en el mercado real.

¡Tener estos cuatro frameworks documentados y claros es el primer paso para estructurar servicios de seguridad con un nivel completamente profesional!

---

## Contenido de la Serie (9 Artículos):

Aquí puedes navegar por cada una de las entregas detalladas de esta serie metodológica:

* **Vol. 0:** [Metodologías de Pentesting: ¿Por qué no basta con saber atacar?](/posts/1introduccion-metodologias)
* **Vol. 1:** [OSSTMM: El enfoque científico y matemático del Pentesting](/posts/2osstmm-framework/)
* **Vol. 2:** [OWASP WSTG: La guía definitiva para auditorías Web profesionales](/posts/3owasp-wstg/)
* **Vol. 3:** [NIST SP 800-115: El estándar institucional y gubernamental de evaluación](/posts/4nist-sp-800-115/)
* **Vol. 4:** [PTES: El mapa de ruta real para auditorías de principio a fin](/posts/5ptes-framework/) *(Este post)*
* **Vol. 5:** [ISSAF: Lecciones de una metodología clásica y el modelo de 9 pasos](/posts/6issaf-framework/)
* **Vol. 6:** [MITRE ATT&CK: El traductor universal del comportamiento adversario](/posts/7mitre-attack/)
* **Vol. 7:** [Frameworks Especializados: Cuando el contexto define las reglas del juego](/posts/8frameworks-especializados/)
* **Vol. 8:** [Criterios de Selección: ¿Qué framework aplicar en cada auditoría?](/posts/9criterios-seleccion/)