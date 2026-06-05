---
title: "Criterios de Selección: ¿Qué framework aplicar en cada auditoría?"
date: 2026-06-05 02:00:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [pentesting, frameworks, compliance, consulting, tryhackme]
series: Metodologías de Pentesting de TryHackMe
---

A lo largo de esta serie hemos analizado diferentes metodologías y bases de conocimiento, cada una con su propia filosofía, estructura y fortalezas. Sin embargo, conocer qué hace cada framework no nos dice automáticamente cuál debemos elegir cuando estamos frente a un cliente real. 

En la práctica, la selección del framework es una de las primeras y más importantes decisiones que toma un pentester tras recibir el informe inicial (*briefing*). Es una decisión moldeada por múltiples factores que a menudo tiran en direcciones diferentes.

Imagina la caja de herramientas de un mecánico: él no utiliza todas las herramientas en todos los trabajos; las selecciona según el vehículo, el problema y las expectativas del cliente. En ciberseguridad operamos igual. El framework "correcto" depende de la industria, el tipo de sistemas, el entorno regulatorio y los objetivos del compromiso.

---

## Los 4 Factores que dominan la selección

En el mundo real, la elección se define principalmente a través de estos cuatro filtros esenciales:

1. **El alcance y tipo de objetivo:** Es el filtro más inmediato. Una auditoría web se alinea naturalmente con **OWASP WSTG**; una aplicación móvil exige **OWASP MASTG**; y una prueba de red completa se adapta a **PTES** o **OSSTMM**. Si el alcance incluye factores humanos (ingeniería social) y seguridad física, el modelo de 5 canales de OSSTMM toma la delantera.
2. **Requisitos regulatorios y de cumplimiento (Compliance):** Las leyes pueden anular por completo tus preferencias personales. Si el cliente procesa datos de tarjetas de crédito, las directrices de **PCI DSS** son obligatorias. Si es una entidad financiera del Reino Unido, se exigirá **CBEST**. Si es una agencia federal estadounidense o un contratista directo, **NIST SP 800-115** tendrá todo el peso. Cuando la regulación manda, el auditor se adapta.
3. **Necesidad de resultados cuantificables y repetibles:** Crucial cuando el cliente te dice: *"Queremos medir matemáticamente si nuestra seguridad ha mejorado respecto al año pasado"*. Los **RAV metrics de OSSTMM** están diseñados específicamente para esto, permitiendo comparar de forma objetiva los resultados en el tiempo, incluso si las pruebas fueron hechas por firmas de seguridad distintas.
4. **Experiencia del equipo y recursos disponibles:** Las limitaciones técnicas y de tiempo son fáciles de pasar por alto. Implementar OSSTMM a fondo exige dominar su complejo sistema de métricas. CBEST demanda capacidades avanzadas de inteligencia de amenazas. Para equipos pequeños que realizan evaluaciones corporativas estándar, **PTES** ofrece el flujo de trabajo más práctico y estructurado sin añadir una carga operativa excesiva.

---

## Casos Prácticos: La teoría puesta en marcha

Analicemos cuatro escenarios reales para entender cómo combinar estas herramientas estratégicamente:

### Escenario 1: Hospital Regional
* **El reto:** Evaluar un portal web de pacientes y la red interna a la que está conectado. Deben cumplir con la normativa de salud **HIPAA**, pero no procesan tarjetas de pago directamente (está subcontratado). Buscan un entregable claro para la junta directiva.
* **La recomendación:** **PTES** es la base ideal aquí. Proporciona la estructura completa de principio a fin, su fase de reporte genera tanto el resumen ejecutivo para la directiva como el reporte técnico para TI, y cubre tanto red como web. Puedes complementar la sección web con **OWASP WSTG** y mapear las vulnerabilidades a identificadores de **MITRE ATT&CK** para enriquecer el análisis con amenazas reales.

### Escenario 2: Banco Multinacional en Londres
* **El reto:** Una entidad financiera con sede en el Reino Unido necesita un pentesting que satisfaga sus estrictas obligaciones regulatorias locales. El compromiso debe iniciar obligatoriamente con una evaluación detallada de inteligencia de amenazas.
* **La recomendación:** **CBEST** es la única opción viable. Está diseñado específicamente por el Banco de Inglaterra para el sector financiero británico y obliga a ejecutar una fase inicial de *Threat Intelligence* a la medida. Ningún framework genérico serviría para cumplir esta ley.

### Escenario 3: Startup de Software (SaaS)
* **El reto:** Una plataforma 100% web quiere contratar a dos firmas de seguridad distintas de forma independiente durante los próximos dos años, y poder comparar los reportes directamente para medir su evolución.
* **La recomendación:** Los **RAVs de OSSTMM** son perfectos para habilitar comparaciones directas entre diferentes equipos y periodos de tiempo. Mientras que **OWASP WSTG** guiará los casos de prueba técnicos específicos del entorno web, OSSTMM aportará la capa de medición matemática para que la comparación año tras año tenga un sentido real y objetivo.

### Escenario 4: Compañía Fintech
* **El reto:** Evaluar las aplicaciones móviles de banca para Android e iOS, las cuales gestionan transacciones de tarjetas de crédito.
* **La recomendación:** Un enfoque combinado y simultáneo. **OWASP MASTG** nos dará los casos de prueba técnicos para auditar la seguridad de las plataformas móviles (almacenamiento, criptografía, etc.), mientras que las pautas de **PCI DSS v4.0** dictarán el alcance regulatorio y el formato del reporte debido al manejo de datos de tarjetas. 

---

## Conclusión de la Serie

Como demuestran estos escenarios, la selección de metodologías rara vez se reduce a una sola opción. Los compromisos profesionales suelen involucrar un **framework principal** que estructura todo el ciclo de vida del servicio (como PTES) y **frameworks suplementarios** que resuelven necesidades específicas del entorno, leyes o plataformas del cliente.

Tener la habilidad de reconocer qué frameworks aplican y cómo se complementan entre sí es la destreza clave que distingue a un verdadero consultor y auditor de ciberseguridad de alguien que simplemente se dedica a ejecutar herramientas automatizadas.

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
* **Vol. 7:** [Frameworks Especializados: Cuando el contexto define las reglas del juego](/posts/8frameworks-especializados/)
* **Vol. 8:** [Criterios de Selección: ¿Qué framework aplicar en cada auditoría?](/posts/9criterios-seleccion/) *(Este post)*