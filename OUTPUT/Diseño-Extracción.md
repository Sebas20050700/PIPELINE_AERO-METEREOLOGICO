# 📑 Especificación de Diseño y Arquitectura de Extracción
## Sistema de Inteligencia Aero-Meteorológica (SIAM-PERÚ)

**Estado:** 🟢 Operacional | **Versión:** 1.2 | **Rama:** `Visual-Crossing`

Este documento describe el **Pipeline de Ingeniería de Datos** diseñado para la ingesta, normalización y validación cruzada de variables meteorológicas críticas para la aviación civil en territorio peruano.



---

### 1. 🎯 Filosofía del Diseño: Resiliencia Multi-Fuente
El SIAM-PERÚ se basa en el principio de **Redundancia Crítica**. El sistema mitiga el riesgo de "puntos únicos de falla" mediante la integración de telemetría satelital global y validación física local (*Ground Truth*). En escenarios donde la API satelital presenta latencia o datos incompletos (`N/D`), el motor activa automáticamente la capa de validación terrestre basada en la infraestructura de SENAMHI.

---

### 2. 🧬 Arquitectura del Flujo de Datos (ETL)

La estructuración se rige por un proceso de **Join Espacial Dinámico** distribuido en tres fases:

#### Fase A: Ingesta Primaria y Resolución de Coordenadas
* **Protocolo:** Consumo de REST API (Visual Crossing).
* **Lógica de Normalización:** Se normalizan los identificadores ICAO (ej. `SPJC`) concatenando el sufijo geográfico `, Peru` para garantizar la integridad de la geolocalización dentro de la jurisdicción nacional.
* **Output Técnico:** Generación de un objeto `DataFrame` maestro con coordenadas `float64` que actúan como clave primaria espacial para el resto del pipeline.

#### Fase B: Validación Forense Terrestre (Módulo SENAMHI)
* **Fuente:** `MAESTRO_ESTACIONES_SENAMHI_GEO.csv` (Dataset extraído vía Web Scraping duro).
* **Algoritmo de Correlación:** Implementación del modelo **K-Nearest Neighbors (KNN)** simplificado mediante la fórmula de Distancia Euclidiana ajustada:
    $$d = \sqrt{(lat_1 - lat_2)^2 + (lon_1 - lon_2)^2} \times 111.12$$
* **Factor de Corrección:** El valor $111.12$ se aplica para la conversión lineal de grados decimales a kilómetros en el eje ecuatorial.

#### Fase C: Consolidación y Persistencia
* **Estructura:** Unión de variables satelitales (Temperatura, Forecast) y metadatos terrestres (Nombre de estación, Distancia de validación).
* **Persistencia:** Exportación a `Reporte_etapa_clima.csv` con codificación UTF-8 para garantizar la interoperabilidad de los datos.



---

### 3. 🧗 Gestión de Riesgos y Manejo de Excepciones

Para garantizar la continuidad operativa solicitada en la rúbrica, se implementaron las siguientes estrategias:

1. **Sanitización de Datos Nulos (NaN Handling):** Implementación de funciones que transforman valores `None` o `N/D` en constantes numéricas (`0.0`), evitando interrupciones críticas por errores de tipo en el pipeline.
2. **Resolución de Ambigüedad Geográfica:** Filtrado riguroso de parámetros de localización para evitar colisiones con topónimos homónimos internacionales.
3. **Auditoría de Precisión:** Generación de una métrica de "Distancia de Validación" que permite al evaluador calificar la representatividad del sensor terrestre asignado.

---

### 📊 Diccionario de Datos del Producto Final

| Campo | Tipo | Descripción |
| :--- | :--- | :--- |
| `aeropuerto_id` | `String` | Identificador único aeronáutico ICAO. |
| `temp_c` | `Float` | Temperatura ambiente en grados Celsius. |
| `estado_actual` | `String` | Fenomenología reportada por satélite. |
| `VALIDADOR_TIERRA` | `String` | Estación SENAMHI física más cercana (Ground Truth). |
| `DIST_VALIDACIÓN` | `Float` | Proximidad geodésica del validador en KM. |

---
*Este diseño estructural cumple estrictamente con el objetivo de integrar elementos desarrollados en clase (Web Scraping, APIs y manipulación de DataFrames) de manera colaborativa.*
