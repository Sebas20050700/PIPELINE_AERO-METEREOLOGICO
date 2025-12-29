# 🛫 Pipeline de Análisis de Correlación Aero-Meteorológica
### Impacto de Fenómenos Atmosféricos en la Eficiencia de Rutas Comerciales en Perú

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![OpenSky](https://img.shields.io/badge/Telemetry-OpenSky-green) ![VisualCrossing](https://img.shields.io/badge/Context-VisualCrossing-orange) ![SENAMHI](https://img.shields.io/badge/Validation-SENAMHI%20Peru-red)

## 🎯 Objetivo del Proyecto
Automatizar la detección de condiciones climáticas adversas y medir su impacto real en el flujo de vuelos comerciales. El sistema identifica qué aeropuertos o rutas son vulnerables a retrasos operacionales, distinguiendo entre variaciones rutinarias y eventos climáticos de fuerza mayor.

---

## 🏗️ Arquitectura de Datos: La "Integración Triple"

El sistema se basa en la triangulación de tres fuentes de verdad para evitar falsos positivos:

### 1. Ingesta Geográfica y Contexto (Visual Crossing Weather)
**Función:** Actúa como el Orquestador del Pipeline. Provee la ubicación base y el estado general de la atmósfera.
* **Variables Clave:**
    * `lat/lon`: Coordenadas maestras.
    * `visibility`: Factor crítico IFR/VFR.
    * `wspd`: Wind Speed.
    * `wgust`: Ráfagas.

### 2. Realidad Operativa (OpenSky Network)
**Función:** Proporcionar la telemetría real de las aeronaves dentro del radio definido por la Etapa 1.
* **Variables Clave:**
    * `velocity`: Detección de patrones de espera.
    * `geo_altitude`: Cambios por turbulencia.
    * `icao24`: ID único.
  
### 3. Validación de Eventos Críticos (SENAMHI)
**Función:** Capa de verificación forense local mediante estaciones terrestres y satélites GOES-19.
* **Variables Clave:**
    * **Discriminación Nieve/Lluvia:** Algoritmo basado en temperatura (`T < 2°C`) y precipitación (`PP > 0`) para validar cierres de pista en zonas de altura.
    * **Evidencia Satelital:** Descarga de imágenes infrarrojas para confirmar topes nubosos (tormentas convectivas).
    * **Alertas Oficiales:** Extracción de avisos meteorológicos vigentes (Niveles Naranja/Rojo).


## 🔄 Flujo del Pipeline (Hitos de Automatización)

### Etapa 1: Ingesta Maestra y Geocalización (Módulo Visual Crossing)
* **Acción:** Resolución de códigos de aeropuertos y obtención del clima sinóptico.
* **Objetivo:** Establecer el **Ancla Geográfica**. Define las coordenadas exactas que sirven de entrada para que OpenSky y SENAMHI sepan dónde buscar información.

### Etapa 2: Captura de Telemetría Aérea (Módulo OpenSky)
* **Acción:** Escaneo de vectores de estado de aeronaves en el "Bounding Box" generado en la Etapa 1.
* **Objetivo:** Medir el comportamiento del tráfico. ¿Quién está volando y quién está frenando en las zonas de aproximación?

### Etapa 3: Validación Forense (Módulo SENAMHI)
* **Acción:** Cruce de datos globales con la red de estaciones locales y sensores remotos.
* **Objetivo:** Actuar como **filtro de veracidad**.
    * *Ejemplo:* Visual Crossing reporta "Lluvia" en Cusco. El módulo SENAMHI analiza la temperatura (-1°C) y reclasifica el evento como "NIEVE", validando un retraso masivo.

### Etapa 4: Normalización y Salida (El "Join")
* **Acción:** Unificación de las tres fuentes mediante `aeropuerto_id` y ventanas temporales.
* **Resultado:** Dataset consolidado para análisis de toma de decisiones.
    > **Ejemplo de Salida:** "El vuelo LA2023 (OpenSky) redujo velocidad un 20% en aproximación. Visual Crossing reportó visibilidad reducida, y SENAMHI confirmó alerta de nevada severa en la estación Granja Kcayra".

## 🗂️ Estructura del Repositorio

El proyecto se organiza de forma modular para permitir el escalamiento independiente de cada motor de extracción:

```text
PIPELINE_AERO-METEREOLOGICO/
├── dashboard/          # Visualización de métricas y estados operativos
├── data/               # Almacenamiento de datasets
│   ├── processed/      # Datos finales normalizados (CSV)
│   └── raw/            # Ingesta bruta de APIs y estaciones
├── images/             # Documentación de procesos y diagramas BPMN
├── maps/               # Mapas interactivos de validación de vuelo (HTML)
├── notebooks/          # Notebooks de desarrollo y pruebas (Colab)
├── src/                # Scripts fuente del proceso ETL
├── README.md           # Documentación maestra
└── requirements.txt    # Librerías y dependencias del sistema
```
