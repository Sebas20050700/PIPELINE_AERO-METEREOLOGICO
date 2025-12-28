# Módulo 1: Visual Crossing Weather Engine (Ingesta Base)
------------------------------------------------------------------------------------------------------------------------------------------------------------
## Descripción
Este módulo constituye la Etapa 1 del Pipeline. Su función principal es actuar como el orquestador geográfico del sistema. A partir de códigos aeronáuticos (ICAO/IATA), el motor resuelve la ubicación exacta y proporciona el contexto meteorológico global necesario para disparar los procesos de rastreo aéreo y validación local.

## Funcionalidad clave

- Geocodificación Automática: Traduce identificadores de aeropuertos (ej. `SPJC`, `SPZO`) en coordenadas de precisión (_latitud_, _longitud_).
- Suministro de "Master Data": Genera el archivo maestro que define la ventana temporal y espacial para los módulos de OpenSky y SENAMHI.
- Contexto Sinóptico: Captura variables climáticas base (temperatura, velocidad del viento, visibilidad y ráfagas) bajo estándares internacionales (unidades métricas).

## Especificaciones técnicas

- Fuente de Datos: Visual Crossing Weather API (vía RapidAPI).
- Tipo de Consulta: `Forecast` (Pronóstico/Histórico por horas).
- Parámetros de Configuración:
  - `location`: Dinámico (Format: `ICAO, PE`).
  - `aggregateHours`: `1` (Granularidad horaria para correlación precisa con vuelos).
  - `unitGroup`: `metric` (Celsius, km/h).
  - `contentType`: `json`.
