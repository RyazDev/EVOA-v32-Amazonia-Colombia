# Pipeline de Procesamiento en Google Earth Engine

Este directorio documenta el pipeline de procesamiento geoespacial, clasificación supervisada y exportación de productos desarrollado en **Google Earth Engine (GEE)**. El código implementa la metodología completa descrita en la Sección 2 del informe académico, desde la adquisición de imágenes satelitales hasta la generación de mapas de cambio multitemporales.

---

## Objetivo

Proporcionar un flujo de trabajo reproducible en la nube para:

1. Adquirir, filtrar y componer imágenes Sentinel-2 y Sentinel-1 sobre tres regiones de interés (ROI) en la Amazonia colombiana.
2. Ingenierizar 28 bandas de entrada combinando datos ópticos, radar, índices espectrales, texturas y variables auxiliares geoespaciales.
3. Entrenar y evaluar cuatro clasificadores supervisados (CART, Random Forest, GTB, SVM-RBF) sobre más de 350 puntos de entrenamiento digitalizados en Chocó (2022).
4. Seleccionar el modelo ganador mediante un score compuesto que prioriza la detección de EVOA.
5. Aplicar el modelo entrenado sin reentrenamiento a 21 escenarios (3 zonas × 7 años: 2019–2025).
6. Aplicar filtros post-clasificación morfológicos y temáticos para reducir falsos positivos.
7. Exportar productos científicos en formato GeoTIFF y tablas CSV para análisis posterior en Python.

---

## Estructura lógica del script

El pipeline se organiza en 11 bloques funcionales secuenciales:

### 1. Definición de regiones de interés (ROI)

Tres geometrías rectangulares en coordenadas WGS84:

- **Chocó (entrenamiento)**: 76.30°W – 77.20°W / 4.70°N – 5.80°N
- **Caquetá (validación externa norte)**: 73.00°W – 74.50°W / 0.50°N – 1.50°N
- **Puré (validación externa sur)**: 70.00°W – 71.50°W / 1.50°S – 0.50°S

### 2. Adquisición y preprocesamiento de Sentinel-2

- **Colección**: `COPERNICUS/S2_SR_HARMONIZED`
- **Filtrado espaciotemporal**: por ROI y año de análisis (2019–2025).
- **Máscara de nubes**: combinación de **Cloud Score+** (umbral `cs ≥ 0.60`) y **SCL** (Scene Classification Layer) para exclusión de nubes, sombras y cirros.
- **Composición anual**: mediana de los 5 mejores mosaicos anuales según calidad de nubosidad, garantizando representatividad del periodo seco y minimizando artefactos atmosféricos.
- **Bandas de entrada**: B2, B3, B4, B5, B6, B7, B8, B8A, B11, B12 (10 bandas).

### 3. Adquisición y preprocesamiento de Sentinel-1

- **Colección**: `COPERNICUS/S1_GRD`
- **Modo de adquisición**: IW (Interferometric Wide swath).
- **Polarizaciones**: VV y VH.
- **Composición**: selección de la órbita predominante (descendente o ascendente) por año y ROI, con conversión a escala lineal y filtrado de fronteras orbitales.
- **Propósito**: discriminación de coberturas bajo condiciones de alta nubosidad persistente en la Amazonia.

### 4. Ingeniería de variables espectrales

Cálculo de 8 índices espectrales a partir de las bandas Sentinel-2 compuestas:

| Índice | Fórmula (conceptual) | Propósito en el proyecto |
|--------|---------------------|--------------------------|
| NDVI | (NIR – Red) / (NIR + Red) | Discriminación bosque vs. suelo desnudo |
| BSI | Índice de Suelo Desnudo | Detección directa de suelos expuestos y sedimentos |
| MNDWI | (Green – SWIR1) / (Green + SWIR1) | Diferenciación agua / suelo |
| NDTI | (SWIR1 – SWIR2) / (SWIR1 + SWIR2) | Detección de turbidez y sedimentos en agua |
| EVI | 2.5 × (NIR – Red) / (NIR + 6×Red – 7.5×Blue + 1) | Índice de vegetación mejorado para dosel cerrado |
| NBR | (NIR – SWIR2) / (NIR + SWIR2) | Detección de quemas y perturbación severa |
| NDSSI | Índice de Sedimentos en Suspensión | Proxy de turbidez por minería fluvial |
| NSMI | Índice de Sedimentos Normalizado | Sensibilidad a materiales suspendidos |

### 5. Variables auxiliares geoespaciales

Integración de capas estáticas y derivadas para enriquecer el espacio de características:

- **Distancia a ríos**: derivada de **HydroSHEDS**. La minería aluvial opera invariablemente en zonas ribereñas; esta variable es clave para el filtro F1 y para el clasificador.
- **Pendiente**: derivada del **SRTM** (NASA). La EVOA opera en planicies; pendientes > 15° se excluyen post-clasificación (F3).
- **Texturas GLCM**: métricas de **contraste**, **ASM** (Second Moment Angular) y **entropía** calculadas sobre bandas de intensidad. Capturan la rugosidad superficial alterada por la minería (patrones de remoción mecánica vs. textura natural del dosel).
- **Filtros de bordes Sobel**: realce de alta frecuencia que resalta límites abruptos entre bosque y zonas intervenidas.

**Total de bandas de entrada para el clasificador: 28**.

### 6. Muestreo supervisado

- **Puntos de entrenamiento**: más de 350 geometrías digitalizadas manualmente sobre composiciones RGB de Sentinel-2 del año **2022** en la zona de Chocó.
- **Extracción temporal corregida**: las firmas espectrales se extraen exclusivamente del año 2022, evitando contaminación temporal (un punto de EVOA en 2022 podría ser bosque en 2019).
- **Balanceo por clase**: estratificación para evitar sesgo hacia la clase mayoritaria (Bosque).
- **Partición**: 70 % entrenamiento / 30 % prueba, con **semilla fija 42** para garantizar reproducibilidad completa.
- **Clases**: 0 = Bosque, 1 = EVOA, 2 = Agua, 3 = Vegetación Secundaria.

### 7. Entrenamiento y evaluación de clasificadores

Se entrenan cuatro algoritmos supervisados con los hiperparámetros documentados:

| Modelo | Hiperparámetros principales |
|--------|----------------------------|
| CART | Árbol con máximo 100 nodos, mínimo 5 muestras por hoja |
| Random Forest | 200 árboles, 4 variables por split, bag fraction 0.7 |
| GTB (Gradient Tree Boost) | 200 árboles, shrinkage 0.1, sampling rate 0.7 |
| SVM-RBF | Kernel RBF, γ = 0.5, C = 100, datos normalizados |

**Criterio de selección — Score compuesto**:

```
Score = 0.40 × F1-EVOA + 0.25 × OA + 0.15 × F1-Agua + 0.20 × F1-Sec
```

Este score ponderado favorece explícitamente la detección de la clase de interés (EVOA) sin descuidar el desempeño general ni las clases críticas de agua y vegetación secundaria.

### 8. Selección del modelo ganador

El **Gradient Tree Boost (GTB)** obtuvo el mayor score compuesto (0.5896) y el mayor F1-EVOA (0.9375), siendo seleccionado como clasificador único para la fase de producción multitemporal y multirregional.

### 9. Clasificación multitemporal y multirregional

El modelo GTB entrenado en Chocó (2022) se aplica sin reentrenamiento a:

- **7 años**: 2019, 2020, 2021, 2022, 2023, 2024, 2025
- **3 zonas**: Chocó, Caquetá, Puré

Generando **21 mapas de clasificación anual** con resolución nativa de 10 m (exportes definitivos) y 100 m (modo interactivo de exploración).

### 10. Filtros post-clasificación

Cinco filtros secuenciales aplicados sobre la clasificación cruda para reducir falsos positivos y alinear las detecciones con el conocimiento del dominio:

| Filtro | Regla | Justificación |
|--------|-------|---------------|
| **F1 — Proximidad a ríos** | Detecciones EVOA deben estar a < 500 m de un río HydroSHEDS | La minería aluvial es invariablemente ribereña |
| **F2 — Tamaño mínimo de parche** | Parcelas < 10 píxeles (~0.1 ha a 10 m) reclasificadas a Veg. Secundaria | Ruido espectral y artefactos de clasificación |
| **F3 — Restricción de pendiente** | Pendientes > 15° excluidas de EVOA | La minería aluvial opera en planicie |
| **F4 — Agua permanente** | Píxeles con ocurrencia de agua > 90 % (JRC GSW) no pueden ser EVOA | Evita confusión con lagunas naturales o embalses |
| **F5 — Áreas urbanas** | Coberturas construidas (GHSL 2018) excluidas del proxy | Diferenciación de suelo desnudo urbano vs. minero |

### 11. Exportación de productos

Los productos generados se exportan en los siguientes formatos:

- **GeoTIFF de clasificación visual** (`EVOA_v320_09v` a `14v`): paleta de colores embebida (Bosque `#228B22`, EVOA `#FF4500`, Agua `#1E90FF`, Veg. Sec. `#ADFF2F`), listos para visualización directa en QGIS sin configuración adicional.
- **GeoTIFF de composición RGB** (`EVOA_v320_15` a `20`): bandas B4-B3-B2 en color verdadero para validación visual cruzada.
- **GeoTIFF de cambio temporal** (`21_Cambio_Choco`, `22_Cambio_Caqueta`, `23_Cambio_Pure`): categorías 0 (sin cambio), 1 (nueva EVOA), 2 (EVOA persistente), 3 (otro cambio).
- **CSV de métricas** (`EVOA_v320_01_Metricas_Choco.csv`, etc.): exactitud global, Kappa, F1 por clase y score ponderado.
- **CSV de matrices de confusión** (`EVOA_v320_24_MatrizConfusion_4modelos.csv`): matrices normalizadas por fila para los 4 modelos.
- **CSV de K-Fold espacial** (`EVOA_v320_04_KFold_Choco.csv`): OA, Kappa y F1-EVOA por cuadrante.
- **CSV de deforestación anual** (`28_DeforestacionAnual_Choco.csv`, etc.): desglose de áreas (ha) por clase y año.

---

## Entradas utilizadas

| Capa | Colección GEE / Fuente | Variables | Resolución |
|------|------------------------|-----------|------------|
| Sentinel-2 SR | `COPERNICUS/S2_SR_HARMONIZED` | B2–B12 | 10–20 m |
| Sentinel-1 GRD | `COPERNICUS/S1_GRD` | VV, VH | 10 m |
| HydroSHEDS | `WWF/HydroSHEDS/15DIR` | Distancia a ríos derivada | ~90 m |
| SRTM | `USGS/SRTMGL1_003` | Elevación → Pendiente | 30 m |
| JRC GSW | `JRC/GSW1_4/GlobalSurfaceWater` | Ocurrencia anual de agua | 30 m |
| GHSL | `JRC/GHSL/P2016/SMOD_POP_GLOBE_V1` | Áreas urbanas construidas 2018 | 30 m |
| Cloud Score+ | `GOOGLE/CLOUD_SCORE_PLUS/V1/S2_HARMONIZED` | Probabilidad de nubes | 10 m |

---

## Salidas producidas

| Producto | Patrón de archivo | Formato | Resolución |
|----------|-------------------|---------|------------|
| Clasificación visual anual | `EVOA_v320_XXv_ClfVisual_[Zona]_[Año].tif` | GeoTIFF RGB paletizado | 10 m / 100 m |
| Composición RGB anual | `EVOA_v320_XX_RGB_[Zona]_[Año].tif` | GeoTIFF multibanda | 30 m |
| Mapa de cambio 2019→2025 | `XX_Cambio_[Zona].tif` | GeoTIFF monocapa | 10 m |
| Métricas de modelos | `EVOA_v320_01_Metricas_Choco.csv` | CSV | — |
| Matrices de confusión | `EVOA_v320_24_MatrizConfusion_4modelos.csv` | CSV | — |
| K-Fold espacial | `EVOA_v320_04_KFold_Choco.csv` | CSV | — |
| Áreas anuales | `XX_DeforestacionAnual_[Zona].csv` | CSV | — |

---

## Relación con el resto del proyecto

- Los **notebooks Python** (`code/python/`) consumen los CSVs exportados desde GEE para calcular métricas agregadas, generar visualizaciones estadísticas y producir el dashboard ejecutivo.
- Los **GeoTIFF** almacenados en `results/` y `results/mapas_cambio/` son la evidencia espacial primaria que respalda las conclusiones del informe académico.
- El **README principal** del repositorio sintetiza los hallazgos derivados de este pipeline para audiencias técnicas y científicas.

---

## Notas técnicas

- El pipeline está diseñado para ejecutarse principalmente en el **Code Editor de Google Earth Engine** (JavaScript) o mediante la **API Python de GEE** dentro de Google Colab.
- La autenticación con `earthengine-api` requiere una cuenta de Google con acceso a GEE habilitado.
- Las exportaciones de GeoTIFF grandes pueden requerir tareas de batch (`Export.image.toDrive` o `Export.image.toAsset`) debido a los límites de memoria del entorno interactivo.
- La reproducibilidad total depende de la disponibilidad de imágenes Sentinel en el catálogo de GEE; la composición por mediana de los 5 mejores mosaicos mitiga la variabilidad interanual de nubosidad.
