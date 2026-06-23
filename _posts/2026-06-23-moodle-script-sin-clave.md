---
title: "Script para automatizar la apertura y cierre de exámenes en Moodle sin clave"
date: 2026-06-22 23:15:00 -0500
categories: [Moodle, Automatización]
tags: [script, cron, python, mysql, moodle]
---

## Código Python para automatizar la apertura y cierre de exámenes en Moodle sin clave

En este blog, presento un script en Python que permite automatizar la apertura y cierre de exámenes en Moodle sin necesidad de gestionar claves para cada examen. Este script se puede ejecutar como un proceso cron para garantizar que los exámenes se abran y cierren automáticamente según el horario establecido.

```python
import mysql.connector
from datetime import datetime, timedelta
import pytz
import os

# ==========================================
# 1. CONFIGURACIÓN DE LA BASE DE DATOS
# ==========================================
db_config = {
    'host': 'localhost',
    'user': '****',
    'password': '****',
    'database': '****',
    'port': 3306
}

# Los IDs de los exámenes oficiales
ids_pruebas = [62072, 62073, 61502, 61516, 61517, 61518]

# Las 12 claves para la rotación 
claves = [
    'AMB363', 'KRU1835', 'ZTM7344', 'PE4R952', 'QY3L346', 'PO1C347', 
    'BTR9427', 'THM7533', 'SLF2289', 'DNX2585', 'FTR2301', 'E1CO387'
]

def calcular_ventana_y_clave_cron():
    """
    Calcula los tiempos y la clave de forma automatizada basándose en el reloj
    del sistema. Elimina la necesidad de pasar datos por la terminal.
    """
    tz_local = pytz.timezone('America/Guayaquil')
    ahora_local = datetime.now(tz_local)
    
    # Base del día de hoy para los cálculos
    hoy_08am = ahora_local.replace(hour=8, minute=0, second=0, microsecond=0)
    hoy_12pm = ahora_local.replace(hour=12, minute=0, second=0, microsecond=0)
    hoy_14pm = ahora_local.replace(hour=14, minute=0, second=0, microsecond=0)
    hoy_18pm = ahora_local.replace(hour=18, minute=0, second=0, microsecond=0)

    dia_semana = ahora_local.weekday()
    if dia_semana > 5:  # Si es domingo, salta a la lógica del lunes
        dia_semana = 0

    if ahora_local < hoy_12pm:
        # BLOQUE MAÑANA AUTOMÁTICO
        fecha_abrir = hoy_08am
        fecha_cerrar = hoy_12pm
        bloque = f"MAÑANA ({fecha_abrir.strftime('%A %d-%m')} | 08:00 - 12:00)"
        indice_clave = dia_semana * 2
        
    elif ahora_local >= hoy_12pm and ahora_local < hoy_18pm:
        # BLOQUE TARDE AUTOMÁTICO
        fecha_abrir = hoy_14pm
        fecha_cerrar = hoy_18pm
        bloque = f"TARDE ({fecha_abrir.strftime('%A %d-%m')} | 14:00 - 18:00)"
        indice_clave = (dia_semana * 2) + 1
        
    else:
        # PREPARACIÓN NOCTURNA: Si el cron se ejecuta en la noche, calcula el día siguiente
        manana_local = ahora_local + timedelta(days=1)
        dia_semana_manana = manana_local.weekday()
        if dia_semana_manana > 5: 
            dia_semana_manana = 0

        fecha_abrir = hoy_08am + timedelta(days=1)
        fecha_cerrar = hoy_12pm + timedelta(days=1)
        bloque = f"MAÑANA DEL DÍA SIGUIENTE ({fecha_abrir.strftime('%A %d-%m')} | 08:00 - 12:00)"
        indice_clave = dia_semana_manana * 2

    indice_clave = indice_clave % len(claves)
    nueva_clave = claves[indice_clave]

    timestamp_open = int(fecha_abrir.timestamp())
    timestamp_close = int(fecha_cerrar.timestamp())
    
    return timestamp_open, timestamp_close, bloque, nueva_clave

def ejecutar_proceso_cron():
    try:
        # Cálculo automático del bloque según la hora real de la ejecución
        ts_open, ts_close, bloque, nueva_clave = calcular_ventana_y_clave_cron()

        print(f"[{datetime.now()}] Iniciando automatización programada...")
        print(f" -> Bloque identificado: {bloque}")
        print(f" -> Clave rotada: '{nueva_clave}'")
        print(f" -> Cantidad de exámenes a actualizar: {len(ids_pruebas)}")

        # Conexión limpia
        conn = mysql.connector.connect(**db_config)
        cursor = conn.cursor()

        placeholders = ','.join(['%s'] * len(ids_pruebas))
        lista_ids = list(ids_pruebas)

        # ---------------------------------------------------------------------
        # INYECCIÓN DIRECTA DE REGLAS DE TIEMPO E INTENTOS ILIMITADOS
        # ---------------------------------------------------------------------
        sql_update_quiz = f"""
            UPDATE mdl_quiz q
            JOIN mdl_course_modules cm ON q.id = cm.instance
            SET q.password = %s, 
                q.timeopen = %s, 
                q.timeclose = %s,
                q.attempts = 0,
                q.overduehandling = 'autosubmit',
                q.graceperiod = 0
            WHERE cm.id IN ({placeholders})
        """
        parametros_update = [nueva_clave, ts_open, ts_close] + lista_ids
        cursor.execute(sql_update_quiz, parametros_update)
        examenes_actualizados = cursor.rowcount

        conn.commit()

        print(f" -> {examenes_actualizados} exámenes inyectados con éxito en la BD.")

        # ---------------------------------------------------------------------
        # PURGA DE CACHÉ DEL SISTEMA
        # ---------------------------------------------------------------------
        print(" -> Purgando cachés de Moodle de forma asíncrona...")
        os.system("sudo -u daemon php /opt/bitnami/moodle/admin/cli/purge_caches.php")

        print(f"[{datetime.now()}] !!! PROCESO AUTOMÁTICO EJECUTADO CON ÉXITO !!!\n")

    except Exception as e:
        print(f"[{datetime.now()}] ERROR CRÍTICO EN EL CRON: {e}\n")
    finally:
        if 'conn' in locals() and conn.is_connected():
            cursor.close()
            conn.close()

if __name__ == "__main__":
    ejecutar_proceso_cron()
```