# Notebooks de Análisis Estadístico y Visualización

Este directorio alberga los notebooks de Python desarrollados para el análisis estadístico de métricas de clasificación, generación de visualizaciones científicas, interpretación de resultados multitemporales y construcción del dashboard ejecutivo integrado. Los notebooks operan sobre los archivos CSV y GeoTIFF producidos por el pipeline de Google Earth Engine.

---

## Objetivo

Transformar los productos numéricos y espaciales exportados desde GEE en evidencia analítica reproducible mediante:

1. Cálculo riguroso de métricas de desempeño clasificatorio (exactitud, Kappa, F1-score por clase, score ponderado compuesto).
2. Análisis comparativo de cuatro algoritmos de clasificación supervisada.
3. Evaluación de la estabilidad espacial del modelo mediante validación cruzada K-Fold con cuadrantes geográficos.
4. Caracterización de la dinámica temporal de EVOA y coberturas entre 2019 y 2025.
5. Generación de 9 visualizaciones científicas y un dashboard ejecutivo de nueve paneles.

---

## Procesamiento realizado

### 1. Ingesta y limpieza de datos

Los notebooks leen los archivos CSV generados en GEE:

- `EVOA_v320_01_Metricas_Choco.csv` — Métricas de desempeño de los 4 modelos sobre el conjunto de prueba de Chocó (2022).
- `EVOA_v320_24_MatrizConfusion_4modelos.csv` — Matrices de confusión normalizadas por fila (exactitud del productor por clase) para CART, Random Forest, GTB y SVM-RBF.
- `EVOA_v320_04_KFold_Choco.csv` — Resultados de validación cruzada espacial K-Fold (4 cuadrantes: NE, NO, SE, SO).
- `02_Metricas_Caqueta.csv` y `03_Metricas_Pure.csv` — Métricas de validación externa en zonas no vistas durante el entrenamiento.
- `28_DeforestacionAnual_Choco.csv`, `29_DeforestacionAnual_Caqueta.csv`, `30_DeforestacionAnual_Pure.csv` — Series temporales de áreas (ha) por clase y año (2019–2025).

### 2. Cálculo de métricas agregadas

A partir de los datos crudos se computan:

- **Exactitud Global (OA)** y **Coeficiente Kappa** por modelo.
- **F1-score** por clase: Bosque, EVOA, Agua, Vegetación Secundaria.
- **Score compuesto ponderado**:
  ```
  Score = 0.40 × F1-EVOA + 0.25 × OA + 0.15 × F1-Agua + 0.20 × F1-Sec
  ```
- **Desviación estándar inter-cuadrante** de OA, Kappa y F1-EVOA para cuantificar la estabilidad espacial del modelo ganador.
- **Tendencia lineal** de la serie temporal de área EVOA por zona (pendiente y coeficiente de determinación R²).
- **Variación interanual porcentual** de área EVOA respecto al año anterior.

### 3. Análisis de matrices de confusión

Normalización por fila para interpretar la **exactitud del productor** (sensibilidad por clase). Se identifican los principales errores de omisión y comisión:

- Confusión EVOA ↔ Vegetación Secundaria: justificada espectralmente por la presencia de suelo descubierto parcial y baja cobertura vegetal en ambas clases.
- Confusión Bosque ↔ Vegetación Secundaria: transición ecológica natural y post-intervención.
- Confusión EVOA ↔ Agua: marginalmente reducida por el filtro post-clasificación F4 (JRC GSW).

### 4. Validación cruzada espacial (K-Fold)

Análisis de los 4 folds geográficos sobre Chocó:

- Comparación de OA, Kappa y F1-EVOA por cuadrante (NE, NO, SE, SO).
- Gráficos de barras agrupadas y diagramas de violín para ilustrar la variabilidad inter-espacial.
- Criterio de aceptación: variación de OA esperada < 5 % entre cuadrantes, confirmando que el modelo captura patrones espectrales robustos y no está sobreajustado a un subsector específico del área de entrenamiento.

### 5. Análisis temporal multirregional

Procesamiento de las series de áreas anuales (2019–2025) para las tres zonas:

- Curvas de evolución de EVOA con línea de tendencia lineal.
- Gráficos de barras apiladas de composición anual de coberturas.
- Heatmaps de área EVOA por zona-año y variación interanual porcentual.
- Clasificación de cambio 2019→2025: sin cambio (0), nueva EVOA (1), EVOA persistente (2), otro cambio (3).

---

## Métricas calculadas

| Métrica | Definición operacional | Uso en el proyecto |
|---------|------------------------|--------------------|
| OA (Overall Accuracy) | Proporción de píxeles correctamente clasificados sobre el total | Desempeño general del modelo |
| Kappa | Coeficiente de concordancia corregido por azar | Robustez del acuerdo clasificatorio |
| F1-Score | Media armónica de precisión y sensibilidad por clase | Balance por clase, especialmente EVOA |
| F1-EVOA | F1-score específico de la clase de interés | Criterio prioritario del score compuesto |
| Score ponderado | 0.40×F1-EVOA + 0.25×OA + 0.15×F1-Agua + 0.20×F1-Sec | Criterio de selección del modelo ganador |
| Variación interanual | (Área_t – Área_t-1) / Área_t-1 × 100 | Detección de aceleración o desaceleración de perturbación |

**Resultados numéricos principales**:

| Modelo | OA (%) | Kappa | F1-EVOA | Score |
|--------|--------|-------|---------|-------|
| CART | 84.91 | 0.7969 | 0.8657 | 0.5586 |
| Random Forest | 87.74 | 0.8350 | 0.9231 | 0.5886 |
| **GTB** | **85.85** | **0.8100** | **0.9375** | **0.5896** |
| SVM-RBF | 87.74 | 0.8352 | 0.9231 | 0.5886 |

---

## Visualizaciones generadas

### Figuras individuales

1. **`01_comparativa_modelos.png`** — Gráfico de barras comparativo de OA, Kappa, F1-EVOA y Score para los 4 modelos. Las barras en rojo destacan el F1-EVOA como criterio prioritario. GTB se identifica como ganador por su score compuesto máximo.

2. **`02_matrices_confusion_4modelos.png`** — Cuatro matrices de confusión normalizadas por fila (una por modelo). La diagonal representa la exactitud del productor por clase. Dimensiones: clases × clases (Bosque, EVOA, Agua, Veg. Secundaria).

3. **`03_kfold_espacial.png`** — Resultados de validación cruzada espacial. Barras agrupadas por cuadrante (OA, Kappa, F1-EVOA) y gráfico de violín de la distribución inter-cuadrante. Confirma variabilidad < 5 % en OA.

4. **`04_evolucion_evoa_temporal.png`** — Serie temporal de área EVOA (ha) para Chocó, Caquetá y Puré (2019–2025). Incluye línea de tendencia lineal con ecuación y R². Una pendiente positiva indica expansión acumulada de perturbación.

5. **`05_composicion_cobertura_por_zona.png`** — Gráfico de barras apiladas anuales por zona. Cada barra representa la proporción de Bosque, EVOA, Agua y Vegetación Secundaria en un año determinado.

6. **`06_heatmap_variacion_evoa.png`** — Panel dual: (izquierda) heatmap de área EVOA por zona-año; (derecha) heatmap de variación interanual porcentual. Tonos rojos indican mayor perturbación o incremento; tonos azules/verdes indican reducción o estabilidad.

7. **`07_pie_coberturas_2019_2025.png`** — Tres pares de gráficos circulares (uno por zona) comparando la distribución porcentual de coberturas en los años extremos del análisis (2019 vs. 2025).

8. **`08_tasa_cambio_evoa_bosque.png`** — Gráfico combinado: barras de variación interanual de EVOA (eje izquierdo) y línea de evolución de área de Bosque (eje derecho) para las tres zonas. Permite visualizar la relación inversa perturbación–conservación.

9. **`09_dashboard_resumen_ejecutivo.png`** — Panel integrado de nueve sub-gráficos que consolidan métricas de modelos, K-Fold, evolución temporal multirregional, composición de coberturas y tasas de cambio en una única figura de síntesis ejecutiva.

### Mapas de cambio temporal

Los análisis de cambio 2019→2025 se visualizan a partir de los GeoTIFF exportados desde GEE (`21_Cambio_Choco.tif`, `22_Cambio_Caqueta.tif`, `23_Cambio_Pure.tif`), con la siguiente simbología:

- **Sin cambio relevante** (0): Gris neutro / transparente.
- **Nueva EVOA** (1): Rojo intenso. Frentes de expansión activa que requieren intervención preventiva urgente.
- **EVOA persistente** (2): Naranja. Zonas de explotación crónica o consolidada.
- **Otro cambio** (3): Azul. Regeneración, inundación temporal, apertura de vías u otros procesos de transición.

---

## Interpretación de resultados

### Selección del modelo

El **Gradient Tree Boost (GTB)** fue seleccionado por maximizar el F1-EVOA (0.9375), indicando alta capacidad para detectar la clase minoritaria y de mayor interés sin incurrir en exceso de falsos positivos. Aunque Random Forest y SVM-RBF alcanzan una OA ligeramente superior (87.74 %), su F1-EVOA (0.9231) y score compuesto (0.5886) son inferiores al de GTB. CART funciona como línea base interpretable pero con menor capacidad discriminativa.

### Validación espacial

La consistencia de métricas entre los 4 cuadrantes de Chocó (variación de OA < 5 %) confirma que el modelo aprende patrones espectrales y texturales inherentes a las coberturas, y no dependencias geográficas locales. Cuadrantes con mayor densidad de nubosidad o agua permanente presentan F1-EVOA ligeramente menores por la menor disponibilidad de muestras de entrenamiento locales, lo cual es esperado y no invalida la generalización del modelo.

### Transferibilidad espacial

La aplicación sin reentrenamiento a Caquetá y Puré demostró generalización genuina:

- **Caquetá**: Las detecciones de EVOA se concentran en márgenes ribereñas del río Caquetá, coherentes con los informes UNODC (2021, 2022) que reportan el mayor número de alertas EVOA en agua de los ríos amazónicos monitoreados. La coexistencia de múltiples motores de deforestación (minería fluvial, cultivos ilícitos, ganadería) produce una firma espectral de perturbación más heterogénea que en Chocó.
- **Puré**: La escasa extensión de EVOA en tierra (2.874 ha en 2024, 0.16 % del área) coincide con el carácter predominantemente fluvial de la minería documentado por MAAP (2025) y UNODC (2022). El dominio del bosque primario (1.666.990 ha) confirma que el PNN Río Puré conserva su cobertura en la mayor parte de su extensión.

### Dinámica temporal

Las series 2019–2025 revelan que las firmas espectrales de las coberturas de interés cambian poco a lo largo del tiempo, lo cual permite implementar un monitoreo continuo con un único modelo entrenado al inicio y reutilizable posteriormente. La distinción entre EVOA nueva y persistente tiene implicaciones directas para la gestión territorial:

- **Nueva EVOA** indica frentes de expansión activa que requieren intervención preventiva urgente.
- **EVOA persistente** puede corresponder a explotaciones formalizables en proceso o zonas de explotación crónica sin control estatal.

---

## Dependencias utilizadas

Las bibliotecas requeridas para ejecutar los notebooks están listadas en el archivo `requirements.txt` del directorio raíz. Las principales son:

- **pandas**: Manipulación y análisis de tablas de métricas y series temporales.
- **numpy**: Operaciones numéricas, álgebra lineal y cálculo de estadísticos.
- **matplotlib**: Generación de gráficos estáticos (barras, líneas, matrices, pies).
- **seaborn**: Visualizaciones estadísticas avanzadas (heatmaps, diagramas de violín, estilos científicos).
- **geopandas** (opcional para análisis espacial extendido): Manipulación de datos vectoriales y superposición con resultados raster.
- **rasterio** (opcional para análisis espacial extendido): Lectura y procesamiento de GeoTIFF de clasificación y cambio.
- **scikit-learn**: Replicación de métricas de clasificación (Kappa, F1, matriz de confusión) como validación independiente de los resultados de GEE.

---

## Flujo de trabajo recomendado

1. Ejecutar el pipeline de GEE (`code/gee/`) para generar los CSVs y GeoTIFFs exportados.
2. Descargar los productos a este directorio o vincularlos mediante rutas relativas (`../results/`).
3. Ejecutar los notebooks en el siguiente orden lógico:
   - `01_comparativa_modelos.ipynb` → Lectura de métricas y selección de ganador.
   - `02_matrices_confusion.ipynb` → Análisis de errores por clase.
   - `03_kfold_espacial.ipynb` → Evaluación de estabilidad geográfica.
   - `04_evolucion_temporal.ipynb` → Series de áreas y tendencias.
   - `05_dashboard_ejecutivo.ipynb` → Consolidación de visualizaciones en panel único.
4. Los notebooks generan automáticamente las figuras en alta resolución (300 dpi) listas para inclusión en informes técnicos o presentaciones científicas.

---

## Notas

- Los notebooks están optimizados para ejecución en **Google Colab** con acceso a Google Drive montado, o en entornos locales con JupyterLab.
- Las rutas de archivos CSV deben ajustarse según la estructura de directorios del entorno de ejecución.
- La reproducibilidad de las visualizaciones depende de la consistencia de los datos de entrada; los CSVs exportados desde GEE contienen los valores exactos reportados en el informe académico.
