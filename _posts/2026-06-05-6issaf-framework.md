---
title: "ISSAF: Lecciones de una metodología clásica y el modelo de 9 pasos"
date: 2026-06-05 02:15:00 -0500
categories: [Ciberseguridad, Metodologías]
tags: [issaf, pentesting, kill-chain, frameworks, tryhackme]
series: Metodologías de Pentesting de TryHackMe
---

¿Alguna vez has estudiado un libro técnico antiguo y has descubierto que, a pesar de los años, su lógica central sigue siendo totalmente válida? En ciberseguridad, los exploits y las herramientas tienen una fecha de caducidad muy corta, pero las metodologías bien diseñadas pueden sobrevivir a las tecnologías para las que fueron creadas. El **Information Systems Security Assessment Framework (ISSAF)** es el ejemplo perfecto de esto.

Desarrollado por el *Open Information Systems Security Group (OISSG)*, ISSAF es un framework de código abierto diseñado para evaluar la seguridad de redes, sistemas y aplicaciones. Su última versión (`v0.2.1`) se publicó en **2006** y actualmente ya no recibe mantenimiento ni tiene un sitio oficial de descarga. 

Entonces, **¿por qué deberías estudiar un framework obsoleto?** Porque el modelo de evaluación de **9 pasos** de ISSAF es una de las representaciones más claras y didácticas de cómo un atacante real progresa dentro de un entorno corporativo, imitando a la perfección la lógica de una Amenaza Persistente Avanzada (APT) o el concepto del *Cyber Kill Chain*.

---

## Las 3 Fases de ISSAF

ISSAF divide un compromiso de seguridad en tres grandes fases. Para entenderlo, utilizaremos un caso práctico: auditar a **TechBridge Solutions**, una empresa de desarrollo de software con 200 empleados, un servidor Git interno y un portal de gestión de proyectos para clientes.

### Fase 1: Planificación y Preparación (*Planning and Preparation*)
Aquí se definen los límites del compromiso con la directiva de la empresa: el alcance exacto (red corporativa, servidor Git y portal de proyectos), los protocolos de escalación, los contactos de emergencia y las restricciones técnicas críticas (por ejemplo, *el servidor Git de producción no se puede interrumpir en horario laboral*).

### Fase 2: Evaluación (*Assessment* y el Modelo de 9 Pasos)
Este es el núcleo de ISSAF. Cada paso construye los cimientos del siguiente, simulando el avance profundo de un adversario real:

[Recon] 1. Info Gathering ➔ 2. Network Mapping ➔ 3. Vuln Identification
⬇
[Attack] 6. Further Enum ⮜ 5. Privilege Escalation ⮜ 4. Penetration
⬇
[Stealth] 7. Lateral Movement ➔ 8. Maintaining Access ➔ 9. Covering Tracks

#### 🛡️ Bloque A: Reconocimiento y Análisis (Pasos 1-3)
1. **Information gathering (Recopilación de información):** Recolectar datos públicos de la empresa (registros DNS, perfiles de LinkedIn de empleados o tecnologías mencionadas en ofertas de empleo como *"experiencia en Jenkins y GitLab"*).
2. **Network mapping (Mapeo de red):** Diseñar la topología de la red viva. Identificar IPs externas, pasarelas VPN, servidores de correo y, a nivel interno, ubicar el servidor Git y las estaciones de trabajo de los desarrolladores.
3. **Vulnerability identification (Identificación de vulnerabilidades):** Escanear los activos mapeados. Descubres que el portal web usa un CMS antiguo con un bypass de autenticación conocido, y que el servidor de Jenkins tiene su consola de administración expuesta sin contraseña.

#### ⚔️ Bloque B: Compromiso Activo (Pasos 4-7)
4. **Penetration (Penetración):** Intentar la explotación inicial. Utilizas la consola expuesta de Jenkins para ejecutar comandos del sistema en el servidor de compilación.
5. **Gaining access and privilege escalation (Acceso y escalada de privilegios):** Pasar del acceso inicial a privilegios más altos. Desde el servidor Jenkins, logras recuperar credenciales almacenadas de una cuenta de servicio con derechos de administrador en el servidor Git.
6. **Enumerating further (Enumeración profunda):** Con el nuevo acceso elevado, investigas qué más puedes alcanzar. Al ingresar al servidor Git, descubres repositorios que contienen llaves API hardcodeadas, credenciales de bases de datos y código fuente de clientes.
7. **Lateral movement (Movimiento lateral):** Usar las credenciales cosechadas para saltar hacia otros sistemas críticos, como las estaciones de trabajo de los desarrolladores y el servidor de correo interno.

#### 👤 Bloque C: Persistencia y Sigilo (Pasos 8-9)
8. **Maintaining access (Mantener el acceso):** Diseñar y documentar cómo un atacante real plantaría un *backdoor* o persistencia en el pipeline de CI/CD para no perder el control incluso si los servidores se reinician.
9. **Covering tracks (Cubrir huellas):** Analizar qué logs del sistema registraron tu actividad técnica e identificar los fallos o huecos en la estrategia de monitoreo de la empresa que permitirían a un criminal operar sin ser detectado.

### Fase 3: Reporte y Limpieza (*Reporting and Cleanup*)
Se compilan los hallazgos priorizados por el impacto real que generan al negocio (el fallo de Jenkins se marca como *Crítico* por abrir la puerta al código fuente). El paso final es la **Limpieza**: eliminar cualquier artefacto de prueba, revertir cuentas temporales creadas y confirmar con el cliente que no quedó ningún residuo del test en el entorno.

---

## Pros y Contras de ISSAF

### 🟢 Lo mejor: Su legado educativo
El modelo de 9 pasos es una obra de arte para entrenar la mentalidad de un pentester. Te enseña de forma lineal y ordenada cómo se encadenan los fallos (fase de reconocimiento ➔ intrusión ➔ escalada ➔ persistencia).

### 🔴 Lo difícil: Totalmente desactualizado
Su documentación técnica hace referencia a versiones de software y herramientas de hace casi dos décadas. **No debes usar ISSAF para buscar comandos o procedimientos técnicos actuales.**

---

## Conclusión

Estudiar **ISSAF** te aporta la estructura de pensamiento de un atacante avanzado. Usa este framework clásico para entrenar tu **lógica de ataque y análisis de impacto**, pero compleméntalo siempre con las guías técnicas modernas de **PTES** u **OWASP WSTG** para la ejecución de tus herramientas actuales.

---

## Contenido de la Serie (9 Artículos):

Aquí puedes navegar por cada una de las entregas detalladas de esta serie metodológica:

* **Vol. 0:** [Metodologías de Pentesting: ¿Por qué no basta con saber atacar?](/posts/1introduccion-metodologias)
* **Vol. 1:** [OSSTMM: El enfoque científico y matemático del Pentesting](/posts/2osstmm-framework/)
* **Vol. 2:** [OWASP WSTG: La guía definitiva para auditorías Web profesionales](/posts/3owasp-wstg/)
* **Vol. 3:** [NIST SP 800-115: El estándar institucional y gubernamental de evaluación](/posts/4nist-sp-800-115/)
* **Vol. 4:** [PTES: El mapa de ruta real para auditorías de principio a fin](/posts/5ptes-framework/)
* **Vol. 5:** [ISSAF: Lecciones de una metodología clásica y el modelo de 9 pasos](/posts/6issaf-framework/) *(Este post)*
* **Vol. 6:** [MITRE ATT&CK: El traductor universal del comportamiento adversario](/posts/7mitre-attack/)
* **Vol. 7:** [Frameworks Especializados: Cuando el contexto define las reglas del juego](/posts/8frameworks-especializados/)
* **Vol. 8:** [Criterios de Selección: ¿Qué framework aplicar en cada auditoría?](/posts/9criterios-seleccion/)