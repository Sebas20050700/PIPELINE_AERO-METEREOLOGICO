# Módulo 2: SENAMHI (Captura en paralelo)
### Correlación Forense entre Fenómenos Atmosféricos y Eficiencia en Rutas Aéreas - Edición Perú

![Python](https://img.shields.io/badge/Python-3.12-blue) ![Selenium](https://img.shields.io/badge/Selenium-Web%20Scraping-green) ![Folium](https://img.shields.io/badge/Folium-Geospatial_Viz-orange) ![Meteostat](https://img.shields.io/badge/Data-Meteostat_Historical-purple) ![Status](https://img.shields.io/badge/Status-Operational-brightgreen)

## 📋 Descripción Técnica
Esta rama (**`feature/senamhi-integration`**) constituye el núcleo de validación local del pipeline. A diferencia de las APIs globales que interpolan datos, este módulo implementa un enfoque de **"Ground Truth"** (Verdad en Tierra) específico para la orografía peruana.

Realiza una **extracción forense** de datos ocultos del SENAMHI, audita la cobertura de estaciones y valida si las condiciones reportadas por satélites globales (Visual Crossing) coinciden con la realidad local (Neblina, Nieve, Helada).

---

## 🔄 Flujo de Ejecución (Arquitectura Geocéntrica)

El pipeline opera bajo una lógica secuencial de 3 etapas, donde la **Ubicación Geográfica** actúa como la llave maestra que conecta los módulos:

### 📍 ETAPA 1: Contexto Global (Visual Crossing)
* **Input:** Nombre de Ciudad/Aeropuerto (ej. "Lima", "Cusco").
* **Proceso:** Geolocalización y consulta de condiciones sinópticas.
* **Output:** Coordenadas Maestras (`Lat: -12.02`, `Lon: -77.11`) y Viento General (`15 km/h`).
* *Función:* Define el punto cero del análisis.

### ✈️ ETAPA 2: Realidad Operativa (OpenSky Network)
* **Input:** Coordenadas Maestras de Etapa 1 (`-12.02, -77.11`).
* **Proceso:** Escaneo de tráfico aéreo en un radio dinámico sobre ese punto.
* **Output:** Telemetría de aeronaves (ID, Velocidad Vertical, Patrones de Espera).
* *Función:* Detectar si la atmósfera está afectando realmente a los vuelos en esa zona.

### 🏔️ ETAPA 3: Validación Local (Módulo SENAMHI)
* **Input:** Coordenadas Maestras de Etapa 1 (`-12.02, -77.11`).
* **Proceso:**
    1.  Búsqueda de la estación SENAMHI real más cercana (Cálculo Haversine).
    2.  Extracción de datos de sensores locales (no satelitales).
* **Output Final:** Confirmación de Fenómeno Crítico (ej. **"¿Hay Neblina densa?"**, **"¿Es Nieve o Lluvia?"**).
* *Función:* Juez final que confirma o descarta la causa meteorológica.

---

## 🔧 Implementación Técnica del Módulo SENAMHI

### 1. Ingesta Forense (Web Scraping & Regex) 🕵️‍♂️
El sistema extrae datos que no son públicos vía API, atacando directamente el código fuente del mapa oficial.
* **Técnica:** Escaneo de patrones dentro del HTML renderizado usando Expresiones Regulares (`Regex`).
* **Patrón de Extracción:**
    ```python
    regex = r'"nom"\s*:\s*"(.*?)".*?"lat"\s*:\s*(-?\d+\.?\d*).*?"lon"\s*:\s*(-?\d+\.?\d*)'
    ```

### 2. Algoritmo de Discriminación "Nieve vs. Lluvia" ❄️
Para resolver el problema de la "Nieve Fantasma" en los Andes, se aplica una lógica física sobre los datos:
* Si `Precipitación > 0` y `Temperatura <= 1.0°C` ➡️ **❄️ NIEVE (Riesgo Alto)**.
* Si `Precipitación > 0` y `Temperatura > 3.0°C` ➡️ **🌧️ LLUVIA (Riesgo Medio)**.

### 3. Visualización de Cobertura (Folium) 🗺️
Genera mapas interactivos para validar la confianza del dato.
* **Elementos:** Marcador de Referencia (Rojo) + Estación SENAMHI (Verde) + Radio de Confianza (5km).

---

## 📂 Estructura del Proyecto

Archivos generados y gestionados en esta rama:

```text
PROYECTO-AEREO-SENAMHI/
│
├── README.md                       # Tu documentación
├── requirements.txt                # Tus librerías
│
├── src/
│   └── scraping/
│       └── Scraping_SENAMHI.ipynb  # Tu notebook principal
│
├── data/
│   ├── raw/                        # Datos crudos (Inputs)
│   │   ├── MAESTRO_ESTACIONES_SENAMHI_GEO.csv  <-- (CORREGIDO: Estaba aquí realmente)
│   │   └── datos_crudos_senamhi.txt
│   │
│   └── processed/                  # Datos procesados (Outputs)
│       ├── SENAMHI_ESTACIONES_FINAL.csv        <-- (El resultado de tu scraping)
│       ├── reporte_final_clasificado.csv
│       ├── reporte_nieve.csv
│       └── senamhi_clima_indicadores.csv
│
└── results/                        # RESULTADOS VISUALES
    ├── maps/
    │   └── MAPA_VALIDACION_RESULTADOS.html
    └── figures/
        └── GRAFICO_IMPACTO_CLIMATICO.png

└── 📂 results/                      # RESULTADOS VISUALES (Evidencias)
    ├── MAPA_VALIDACION_RESULTADOS.html
    ├── debug_mapa.html
    └── GRAFICO_IMPACTO_CLIMATICO.png
