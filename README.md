## Módulo 2: OpenSky Network Engine (Realidad Operativa)

### Descripción

Este módulo constituye la **Etapa 2 del Pipeline**. Su función principal es capturar la **telemetría real de las aeronaves en operación**, proporcionando una visión objetiva y en tiempo casi real del comportamiento del tráfico aéreo.

A partir de la información recolectada por la red OpenSky (ADS-B), el módulo permite identificar **variaciones operativas**, tales como reducciones de velocidad, cambios de altitud o permanencia en tierra, que pueden estar asociadas a condiciones meteorológicas adversas.

Este módulo actúa como la **fuente de verdad operacional**, sobre la cual se evaluará el impacto del contexto climático suministrado por los módulos meteorológicos.

---

### Funcionalidad clave

- **Captura de Tráfico Aéreo en Tiempo Casi Real**  
  Obtiene el estado instantáneo de aeronaves comerciales equipadas con transpondedores ADS-B.

- **Monitoreo de Variables Operativas Críticas**  
  Extrae métricas de velocidad, altitud, posición y estado en tierra, esenciales para detectar anomalías operativas.

- **Base para Análisis de Impacto Climático**  
  Proporciona las variables dependientes que serán correlacionadas posteriormente con datos meteorológicos (viento, visibilidad, eventos extremos).

- **Normalización de Datos Operativos**  
  Convierte la estructura cruda de la API en un formato tabular estandarizado (CSV/DataFrame) compatible con el merge final del pipeline.

---

### Especificaciones técnicas

- **Fuente de Datos**: OpenSky Network API  
- **Endpoint**: `/api/states/all`  
- **Tipo de Consulta**: Estado global de aeronaves (snapshot dinámico)  
- **Frecuencia de Actualización**: Tiempo casi real (dependiente de la recepción ADS-B)  
- **Cobertura Geográfica**: Global, con filtrado espacial por bounding box (ej. Perú)  
- **Formato de Salida**: JSON → CSV normalizado  

**Unidades**
- Velocidad: metros por segundo (m/s)  
- Altitud: metros (m)  
- Coordenadas: grados decimales (WGS84)

---

### Estructura de Salida (Contrato de Datos)

El módulo genera un objeto de datos (DataFrame/CSV) con la siguiente estructura, que servirá como **insumo principal para el análisis de correlación** y como tabla base para el merge final por tiempo y ubicación.

| Columna | Tipo | Descripción | Uso en el Pipeline |
|------|------|------------|------------------|
| `icao24` | String | Identificador único de la aeronave. | Clave primaria del vuelo. |
| `callsign` | String | Código del vuelo (si está disponible). | Identificación operativa del vuelo. |
| `origin_country` | String | País de registro de la aeronave. | Segmentación geográfica del tráfico. |
| `time_position` | Integer | Timestamp UNIX de la posición reportada. | Clave temporal para correlación con clima. |
| `last_contact` | Integer | Último contacto registrado con la aeronave. | Control de frescura del dato. |
| `latitude` | Float | Latitud de la aeronave. | Asociación espacial con aeropuertos. |
| `longitude` | Float | Longitud de la aeronave. | Asociación espacial con aeropuertos. |
| `baro_altitude` | Float | Altitud barométrica (m). | Análisis de perfiles de vuelo. |
| `geo_altitude` | Float | Altitud geométrica (m). | Detección de cambios por turbulencia. |
| `velocity` | Float | Velocidad horizontal (m/s). | Variable dependiente clave del impacto climático. |
| `true_track` | Float | Rumbo real de la aeronave (°). | Análisis de trayectorias anómalas. |
| `vertical_rate` | Float | Velocidad vertical (m/s). | Detección de ascensos/descensos forzados. |
| `on_ground` | Boolean | Indica si la aeronave está en tierra. | Medición de retrasos y rodaje. |
| `squawk` | String | Código transponder. | Información secundaria operativa. |
| `spi` | Boolean | Indicador de identificación especial. | Control operativo. |
| `position_source` | Integer | Fuente de la posición reportada. | Evaluación de confiabilidad del dato. |

---

### Rol del módulo dentro del Pipeline

El módulo OpenSky representa la **realidad operacional observable** del sistema.  
Sus datos permiten cuantificar cómo el tráfico aéreo responde ante condiciones meteorológicas adversas, sirviendo como base para:

- Análisis descriptivo del comportamiento del tráfico aéreo.
- Modelos de regresión entre clima y variables operativas.
- Detección de anomalías asociadas a eventos extremos.
- Validación cruzada con información meteorológica externa.

---

### Salida final del módulo

- **Archivo generado**: `opensky_full_states.csv`  
- **Granularidad**: Instantánea por aeronave  
- **Uso final**: Integración con Visual Crossing y EcoWeather mediante unión por `timestamp` y `airport_id`
