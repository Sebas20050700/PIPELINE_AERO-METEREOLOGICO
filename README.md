## Módulo 3: SENAMHI — Validación Forense Metereológica  
## *Ground Truth Local para Operaciones Aéreas en Perú*

![Python](https://img.shields.io/badge/Python-3.12-blue)
![Status](https://img.shields.io/badge/Status-Operational-brightgreen)
![Role](https://img.shields.io/badge/Role-Ground%20Truth-red)

---

## 🎯 PROPÓSITO DEL MÓDULO

El **Módulo SENAMHI** constituye la capa de verificación física y final del pipeline. Su función principal es actuar como un **validador de verdad de terreno (Ground Truth)**, contrastando las lecturas satelitales globales y la telemetría aérea con la red nacional de estaciones meteorológicas terrestres. Este módulo es el encargado de reducir la incertidumbre climática provocada por la compleja geografía peruana, confirmando si un riesgo detectado remotamente tiene una base física real en la superficie antes de emitir una alerta crítica.

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
├── README.md                       # La documentación
├── requirements.txt                # Librerías requeridas
│
├── src/
│   └── scraping/
│       └── Scraping_SENAMHI.ipynb  # El notebook principal
│
├── data/
│   ├── raw/                        # Datos crudos (Inputs)
│   │   ├── MAESTRO_ESTACIONES_SENAMHI_GEO.csv
│   │   └── datos_crudos_senamhi.txt
│   │
│   └── processed/                  # Datos procesados (Outputs)
│       ├── SENAMHI_ESTACIONES_FINAL.csv        <-- (El resultado del scraping)
│       ├── reporte_final_clasificado.csv
│       ├── reporte_nieve.csv
│       └── senamhi_clima_indicadores.csv
│
└── results/                        # Resultados visuales
    ├── maps/
    │   └── MAPA_VALIDACION_RESULTADOS.html
    └── figures/
        └── GRAFICO_IMPACTO_CLIMATICO.png
