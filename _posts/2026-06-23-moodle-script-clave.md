---
title: "Automatización de moodle con clave en evaluaciones"
date: 2026-06-21 22:15:00 -0500
categories: [Moodle, Automatización]
tags: [script, cron, python, mysql, moodle]
---

## Código Python para automatizar la apertura y cierre de exámenes en Moodle con clave

En este blog, presento un script en Python que permite automatizar la apertura y cierre de exámenes en Moodle, incluyendo la gestión de claves para cada examen. Este script se puede ejecutar como un proceso cron para garantizar que los exámenes se abran y cierren automáticamente según el horario establecido.


```python
import mysql.connector
from datetime import datetime, timedelta
import pytz
import os  # <-- CORREGIDO: Importación necesaria para ejecutar comandos de consola

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

# ==========================================
# 2. CONFIGURACIÓN DE EXÁMENES (SIN CLAVE)
# ==========================================
# Los 24 IDs de tus pruebas sin contraseña oficiales
ids_pruebas_sin_clave = [61957, 61236, 61937]

def calcular_ventana_tiempo():
    """
    Calcula las fechas de apertura/cierre en formato Unix timestamp
    e identifica el bloque horario actual de manera totalmente autónoma.
    """
    tz_local = pytz.timezone('America/Guayaquil')
    ahora_local = datetime.now(tz_local)
        
    hoy_08am = ahora_local.replace(hour=8, minute=0, second=0, microsecond=0)
    hoy_12pm = ahora_local.replace(hour=12, minute=0, second=0, microsecond=0)
    hoy_14pm = ahora_local.replace(hour=14, minute=0, second=0, microsecond=0)
    hoy_18pm = ahora_local.replace(hour=18, minute=0, second=0, microsecond=0)
    
    if ahora_local < hoy_12pm:
        fecha_abrir = hoy_08am
        fecha_cerrar = hoy_12pm
        bloque = "MAÑANA (08:00 - 12:00)"

    elif ahora_local >= hoy_12pm and ahora_local < hoy_18pm:
        fecha_abrir = hoy_14pm
        fecha_cerrar = hoy_18pm
        bloque = "TARDE (14:00 - 18:00)"

    else:
        fecha_abrir = hoy_08am + timedelta(days=1)
        fecha_cerrar = hoy_12pm + timedelta(days=1)
        bloque = "MAÑANA DEL DÍA SIGUIENTE (08:00 - 12:00)"

    timestamp_open = int(fecha_abrir.timestamp())
    timestamp_close = int(fecha_cerrar.timestamp())
    
    return timestamp_open, timestamp_close, bloque

def ejecutar_proceso_sin_clave():
    try:
        # 1. Obtener tiempos y bloques horarias
        ts_open, ts_close, bloque = calcular_ventana_tiempo()

        print(f"[{datetime.now()}] Iniciando actualización automática (SIN CLAVE) para el bloque: {bloque}")
        
        # Conectar a la base de datos
        conn = mysql.connector.connect(**db_config)
        cursor = conn.cursor()

        placeholders = ','.join(['%s'] * len(ids_pruebas_sin_clave))
        lista_ids = list(ids_pruebas_sin_clave)

        # ---------------------------------------------------------------------
        # ACTUALIZACIÓN DE HORARIOS E INTENTOS ILIMITADOS SIN CONTRASEÑA
        # ---------------------------------------------------------------------
        sql_update_quiz = """
            UPDATE mdl_quiz q
            JOIN mdl_course_modules cm ON q.id = cm.instance
            SET q.password = '', 
                q.timeopen = %s, 
                q.timeclose = %s,
                q.attempts = 0,
                q.overduehandling = 'autosubmit',
                q.graceperiod = 0
            WHERE cm.id IN ({})
        """.format(placeholders)

        parametros_update = [ts_open, ts_close] + lista_ids
        cursor.execute(sql_update_quiz, parametros_update)
        examenes_actualizados = cursor.rowcount

        # Confirmación de la transacción en la BD
        conn.commit()
        print(f" -> {examenes_actualizados} exámenes inyectados correctamente sin clave.")

        # ---------------------------------------------------------------------
        # CORREGIDO: PURGAR CACHÉ INTEGRAL (Elimina el retraso visual)
        # ---------------------------------------------------------------------
        print(" -> Purgando las cachés globales de Moodle...")
        os.system("sudo -u daemon php /opt/bitnami/moodle/admin/cli/purge_caches.php")
        
        print(f"\n[{datetime.now()}] !!! PROCESO COMPLETADO CON ÉXITO (SIN CLAVE) !!!")
        print(f" -> Intentos establecidos como: ILIMITADOS (Historial protegido).")

    except Exception as e:
        print(f"\n[{datetime.now()}] ERROR CRÍTICO: No se pudieron aplicar los cambios: {e}")
    finally:
        if 'conn' in locals() and conn.is_connected():
            cursor.close()
            conn.close()

if __name__ == "__main__":
    ejecutar_proceso_sin_clave()
    
```