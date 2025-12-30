## Módulo 1: Visual Crossing Weather Engine (Ingesta Base)
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
 
## Estructura de Salida (Contrato de Datos)

El módulo genera un objeto de datos (DataFrame/CSV) con la siguiente estructura, sirviendo como llave primaria (`aeropuerto_id`) para el _Merge_ final:

| Columna | Tipo | Descripción | Uso en el Pipeline |
| :--- | :--- | :--- | :--- |
| **aeropuerto_id** | `String` | Código ICAO del aeropuerto (Ej: SPJC). | **Primary Key** para la unión de tablas. |
| **lat / lon** | `Float` | Coordenadas decimales del aeródromo. | Insumo geográfico para **OpenSky** y **SENAMHI**. |
| **temp_celsius** | `Float` | Temperatura ambiente actual. | Validación cruzada vs. datos locales de SENAMHI. |
| **viento_kmh** | `Float` | Velocidad del viento en km/h. | Análisis de impacto en la velocidad de las aeronaves. |
| **condicion_global** | `String` | Descripción general del clima (Cloudy, Rain, etc). | Etiquetado y categorización del reporte final. |

------------------------------------------------------------------------------------------------------------------------------------------------------------

## 📂 Estructura del Proyecto

Archivos generados y gestionados en esta rama:

```text
ETAPA-1-VISUAL-CROSSING/
│
├── README.md                       # 1. DOCUMENTACIÓN PRINCIPAL
│                                   # Resumen ejecutivo, flujo del pipeline y 
│                                   # guía rápida de uso.
│
├── Requerimientos/                 # 2. ESPECIFICACIONES TÉCNICAS
│   ├── REQUERIMIENTOS.md           # Detalle de hardware, red y API Keys.
│   └── requirements.txt            # Librerías (pip install) necesarias.
│
├── OUTPUT/                         # 3. RESULTADOS DEL PROCESAMIENTO
│   └── data_maestra_clima.csv      # El "Master Data" que activa el resto 
│                                   # del sistema (OpenSky y SENAMHI).
│
└── API_VISUAL_CROSSING.ipynb       # 4. MOTOR LÓGICO (CORE)
                                    # Notebook con la arquitectura de 
                                    # geocodificación y consumo de API.
```

------------------------------------------------------------------------------------------------------------------------------------------------------------

## Instalación de dependencias

| `pip install -r Requerimientos.txt` |
