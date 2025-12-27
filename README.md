# 🛫 Pipeline de Análisis de Correlación Aero-Meteorológica
### Impacto de Fenómenos Atmosféricos en la Eficiencia de Rutas Comerciales

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![OpenSky](https://img.shields.io/badge/Telemetry-OpenSky-green) ![VisualCrossing](https://img.shields.io/badge/Context-VisualCrossing-orange) ![SENAMHI](https://img.shields.io/badge/Validation-SENAMHI%20Peru-red)

## 🎯 Objetivo del Proyecto
Automatizar la detección de condiciones climáticas adversas y medir su impacto real en el flujo de vuelos comerciales. El sistema identifica qué aeropuertos o rutas son vulnerables a retrasos operacionales, distinguiendo entre variaciones rutinarias y eventos de fuerza mayor.

---

## 🏗️ Arquitectura de Datos: La "Integración Triple"

El sistema se basa en la triangulación de tres fuentes de verdad para evitar falsos positivos:

### 1. La Realidad Operativa (OpenSky Network)
**Función:** Proporcionar la telemetría real de los aviones en el aire.
* **Variables Clave:**
    * `icao24`: Identificador único de la aeronave.
    * `velocity`: Detección de reducciones de velocidad o patrones de espera (*holding patterns*).
    * `geo_altitude`: Cambios de altitud de crucero por turbulencia.
    * `on_ground`: Medición de tiempos de rodaje y retrasos en despegue.

### 2. El Contexto Climático (Visual Crossing Weather)
**Función:** Proporcionar el estado general de la atmósfera (METAR) en los nodos de origen y destino.
* **Variables Clave:**
    * `wspd` (Wind Speed): Detección de vientos de cola o cruzados.
    * `visibility`: Factor crítico para operaciones de aterrizaje (IFR vs VFR).
    * `conditions`: Etiquetas generales (Thunderstorm, Fog).
    * `wgust` (Wind Gust): Ráfagas que causan cancelaciones inmediatas.

### 3. Validación de Eventos Críticos (SENAMHI)
**Función:** Capa de verificación forense para eventos extremos en los Andes. **Reemplaza a EcoWeather** para brindar precisión local mediante estaciones terrestres y satélites GOES-19.
* **Variables Clave:**
    * **Discriminación Nieve/Lluvia:** Algoritmo basado en temperatura (`T < 2°C`) y precipitación (`PP > 0`) para validar cierres de pista en zonas de altura.
    * **Evidencia Satelital:** Descarga de imágenes infrarrojas para confirmar topes nubosos (tormentas convectivas).
    * **Alertas Oficiales:** Extracción de avisos meteorológicos vigentes (Niveles Naranja/Rojo).

---

## 🔄 Flujo del Pipeline (Etapas)

### Etapa 1: Captura de Tráfico Real
* **Acción:** Escaneo de vectores de estado de aeronaves en un radio específico de los aeropuertos objetivo (ej. SPJC, SPZO, SPQU).
* **Objetivo:** Establecer la "Línea Base" operativa. ¿Quién está volando y quién está frenando?

### Etapa 2: Contextualización Ambiental
* **Acción:** Consulta del clima general en Visual Crossing para los vuelos detectados.
* **Objetivo:** Responder al "por qué" inicial. Proveer el contexto estándar que justifica variaciones menores.

### Etapa 3: Validación Forense (Módulo SENAMHI)
* **Acción:** Cruce de datos globales con la red de estaciones locales del SENAMHI.
* **Objetivo:** Actuar como **filtro de veracidad**.
    * *Ejemplo:* Visual Crossing reporta "Lluvia" en Cusco. El módulo SENAMHI analiza la temperatura (-1°C) y reclasifica el evento como **"NIEVE"**, validando un retraso masivo que una simple lluvia no justificaría.

### Etapa 4: Normalización y Salida (El "Join")
* **Acción:** Unificación de las tres fuentes mediante `Timestamp` y `Airport_ID`.
* **Resultado:** Dataset estructurado o Dashboard de correlación.
    > **Ejemplo de Salida:** "El vuelo LA2023 (OpenSky) redujo velocidad un 20% en aproximación. Visual Crossing reportó visibilidad reducida, y SENAMHI confirmó alerta de nevada severa en la estación Granja Kcayra".

---

## 💻 Instalación y Uso

### Requisitos
```bash
pip install pandas requests selenium webdriver-manager beautifulsoup4 geopy
```

### Estructura

PROYECTO_AEREO_SENAMHI/
│
├── README.md                     <-- Tu documentación técnica
├── requirements.txt              <-- (pandas, selenium, matplotlib, requests)
│
├── src/                          <-- CÓDIGO FUENTE
│   ├── detectar_api_oculta.py
│   ├── extractor_maestro.py
│   ├── analisis_clima.py
│   ├── descarga_satelite.py
│   └── visualizador_resultados.py
│
├── data/                         <-- DATOS
│   ├── input/
│   │   └── datos_crudos_senamhi.txt
│   └── output/
│       ├── MAESTRO_ESTACIONES_SENAMHI_GEO.csv
│       ├── reporte_final_clasificado.csv
│       └── reporte_nieve.csv
│
└── evidence/                     <-- IMÁGENES
    ├── EVIDENCIA_SATELITE_ACTUAL.jpg
    └── GRAFICO_IMPACTO_CLIMATICO.png
