# ✈️ Pipeline de Análisis Aero-Meteorológico (Rama SENAMHI)
### Correlación entre Fenómenos Atmosféricos y Eficiencia en Rutas Aéreas Comerciales - Edición Perú

![Python](https://img.shields.io/badge/Python-3.8%2B-blue) ![Selenium](https://img.shields.io/badge/Selenium-Web%20Scraping-green) ![SENAMHI](https://img.shields.io/badge/Data-SENAMHI%20Perú-red) ![Status](https://img.shields.io/badge/Status-Operational-brightgreen)

## 📋 Descripción General
Esta rama (**`feature/senamhi-integration`**) implementa un módulo de validación meteorológica de alta precisión diseñado específicamente para la orografía compleja del espacio aéreo peruano.

Sustituye la capa de validación genérica (EcoWeather) por una **integración directa y forense con el SENAMHI** (Servicio Nacional de Meteorología e Hidrología del Perú). Esto permite distinguir matemáticamente entre **Lluvia**, **Nieve** y **Helada** en los Andes, utilizando datos en tiempo real de estaciones terrestres y satélites GOES-19.

---

## 🏗️ Arquitectura del Pipeline

El sistema fusiona tres fuentes de datos independientes para validar la causa raíz de los retrasos aéreos:

### 1. Tráfico Aéreo (OpenSky Network) 📡
* **Función:** Telemetría en vivo.
* **Detección:** Identifica patrones de espera (*holding patterns*), reducciones bruscas de velocidad y cambios de altitud no planificados.
* **Cobertura:** Bounding Box del territorio peruano.

### 2. Contexto General (Visual Crossing) ☁️
* **Función:** Datos METAR sinópticos.
* **Variables:** Viento (*Wind Speed/Gusts*), Visibilidad y condiciones generales para aeropuertos de origen/destino.

### 3. Validación Local (Módulo Custom SENAMHI) 🏔️
* **Función:** Capa de verificación de fenómenos extremos en tierra ("Ground Truth").
* **Técnica:** Web Scraping Forense (Network Sniffing) y Análisis Vectorial Geoespacial.

---

## 🔧 Implementación Técnica

### A. Minería de Datos Forense (Scraping)
A diferencia de los métodos tradicionales, este pipeline no lee el HTML visible, sino que intercepta el tráfico de datos:
* **Detector de API Oculta:** Utiliza `Selenium` para capturar las peticiones de red del mapa interactivo del SENAMHI.
* **Extracción Regex:** Decodifica las estructuras JSON ocultas (`var data = [...]`) dentro de la respuesta del servidor.
* **Resultado:** Generación automática de un **Maestro de Estaciones** con +1900 puntos de medición georreferenciados.

### B. Algoritmo de Discriminación "Nieve vs. Lluvia"
Para evitar falsos positivos en zonas andinas (donde una API global puede confundir lluvia fría con nieve), se aplica una lógica física:

```python
# Lógica implementada en analisis_clima.py
Si (Precipitación > 0 mm):
    Si (Temperatura <= 2.0°C):
        Estado = "❄️ NIEVE/HELADA" (Riesgo Alto: Cierre de Pista)
    Sino:
        Estado = "🌧️ LLUVIA LÍQUIDA" (Riesgo Moderado: Operación Estándar)
```

## 📂 Estructura del proyecto

```text
PROYECTO_AEREO_SENAMHI/
│
├── README.md                     # Documentación técnica       
│
├── src/                          # Código fuente
│   ├── detectar_api_oculta.py    # Sniffer de red
│   ├── analisis_clima.py         # Lógica de negocio
│   └── visualizador.py           # Dashboard
│
└── data/                         # Gestión de datos
    └── output/
        └── reporte_final.csv
```
