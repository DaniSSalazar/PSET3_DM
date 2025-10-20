Proyecto: Ingesta y modelado analítico del dataset NYC TLC Trips (2015–2025)
Tecnología: Spark (Jupyter) + Snowflake
Modelo: One Big Table (OBT) para consumo analítico

1. Arquitectura del Proyecto

Este proyecto implementa un flujo completo de datos (ELT) desde archivos Parquet de taxis de Nueva York, hasta una tabla analítica desnormalizada en Snowflake.

Parquet (NYC TLC)
    │
    ▼
RAW (Snowflake)
    - Tablas espejo por servicio (YELLOW / GREEN)
    - Metadatos: run_id, source_year, source_month
    │
    ▼
SILVER (Snowflake)
    - Integración Taxi Zones
    - Unificación YELLOW + GREEN
    - Normalización de catálogos
    │
    ▼
ANALYTICS.OBT_TRIPS
    - One Big Table (1 fila = 1 viaje)
    - Derivadas: trip_duration, avg_speed_mph, tip_pct
    - Consultas directas (JOIN-free)

2. Esquemas Snowflake
Esquema	Propósito
RAW	Aterrizaje espejo del Parquet con metadatos de ingesta
SILVER	Integración y limpieza (Taxi Zones, catálogos unificados)
ANALYTICS	OBT_TRIPS: Tabla final desnormalizada para BI/Analytics

3. Variables de Entorno (.env.example)
El proyecto utiliza exclusivamente variables de entorno para seguridad y reproducibilidad.
SF_ACCOUNT=your_account_name
SF_USER=your_username
SF_PASSWORD=your_password
SF_WAREHOUSE=COMPUTE_WH
SF_DATABASE=NYC_TAXI
SF_ROLE=ACCOUNTADMIN

4. Notebooks del Proyecto
01_ingesta_raw.ipynb: Carga Parquet → RAW (Spark)
02_silver_unificacion.ipynb: Taxi Zones + Unificación
03_obt_construccion.ipynb: Creación analytics.obt_trips
04_validaciones.ipynb: Calidad, nulos, rangos, fechas
05_data_analysis.ipynb: Análisis de negocio

5. Calidad & Auditoría
Reglas aplicadas:
Cohesión de fechas pickup <= dropof
Rangos lógicos: trip_distance > 0, total_amount > 0
Nulos críticos: location_id, vendor_id, rate_code
Registro en DATA_QUALITY_LOG
Idempotencia: Se verificó reinserción del mes 2017-08 sin duplicados (HASH_KEY natural)

6. Conclusiones
Este proyecto permitió implementar un flujo completo de ingeniería y análisis de datos a escala real, utilizando Spark y Snowflake sobre un poco más de 700 millones de viajes de taxi (Yellow & Green, 2015–2025).
Entre los principales aprendizajes:
  1. Arquitectura reproducible y escalable
    La separación en esquemas RAW → SILVER → ANALYTICS demostró ser efectiva para mantener:
    Trazabilidad completa (lineage) 
    Idempotencia garantizada por HASH_KEY
    Capacidad de reconstrucción por partición (mes/año)

  2. One Big Table (OBT) como acelerador analítico
    El modelo ANALYTICS.OBT_TRIPS simplificó las consultas, eliminando JOINs y acelerando la exploración. Fue clave para responder rápidamente preguntas de negocio como:
    Zonas con mayor actividad
    Comportamiento del tip_pct
    Comparación Yellow vs Green
  3. Calidad de datos: retos reales
  Se identificaron desafíos reales de calidad:
    Más de 18M de registros con passenger_count = 0 → posible omisión del conductor
    Casos con pickup > dropoff y distancias 0 → filtros necesarios para analítica avanzada
    Esto evidencia la importancia de validaciones automáticas.
    4. Insights urbanos de movilidad
      El análisis reveló patrones socioeconómicos y de comportamiento urbano:
      Manhattan concentra mayor volumen y tarifas      
      Green Taxis dominan zonas periféricas (Brooklyn, Queens)
      Congestion_surcharge afecta horarios pico (17–20h)
      Propinas más altas con pago en tarjeta vs efectivo
