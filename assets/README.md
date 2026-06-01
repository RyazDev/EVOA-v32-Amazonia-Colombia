# Recursos Gráficos y Visuales

Este directorio almacena los recursos gráficos, diagramas, banners y elementos visuales utilizados para la documentación, presentación y comunicación del proyecto. Su función es exclusivamente de soporte visual; no contiene datos científicos primarios ni productos de análisis geoespacial, los cuales se encuentran en `results/`.

---

## Recursos gráficos utilizados

### Diagramas de arquitectura

- **Pipeline de procesamiento** — Esquema visual del flujo completo desde la adquisición de imágenes satelitales hasta la exportación de productos científicos. Ilustra las etapas del pipeline en Google Earth Engine y su interacción con los notebooks de análisis.
- **Metodología** — Representación gráfica de las secciones metodológicas principales (diseño, áreas, datos, clases, puntos, modelos, validación, filtros, análisis temporal) y sus interdependencias.

### Banners y cabeceras

- **Banner principal del repositorio** — Imagen de ancho completo para el README principal del repositorio en GitHub. Incluye el título del proyecto, institución académica y palabras clave temáticas (teledetección, machine learning, Amazonia).
- **Banner para presentaciones** — Variante optimizada para presentaciones en formato 16:9, con fondo satelital de la región de estudio y paleta de colores del proyecto.

### Elementos visuales de contexto

- **Contexto amazónico** — Imagen ilustrativa de deforestación en la Amazonia colombiana (referenciada como Figura 1 del informe académico). Fuente: MAAP, Colombia 2020.
- **Contexto de teledetección** — Imagen ilustrativa de descarga de datos de teledetección (referenciada como Figura 2 del informe académico).
- **Mapa de áreas de estudio** — Mapa base de localización de las regiones de interés sobre relieve sombreado de Colombia (referenciado como Figura 3 del informe académico).
- **Esquema del pipeline** — Ilustración conceptual del flujo temporal y multirregional (referenciado como Figura 4 del informe académico).

### Paletas y leyendas

- **Paleta de colores oficial** — Paleta en formato vectorial con los colores asignados a cada clase de cobertura: Bosque, EVOA, Agua, Vegetación Secundaria. Utilizada en clasificaciones, figuras estadísticas y mapas de cambio.
- **Leyenda estandarizada** — Leyenda en alta resolución para inclusión en mapas impresos y presentaciones.

### Logos institucionales

- **Logo de la universidad** — Para documentación académica.
- **Logo de la facultad / programa** — Para portadas, encabezados y agradecimientos.

---

## Uso de los recursos

| Recurso | Ubicación de uso | Formato | Licencia de uso |
|---------|------------------|---------|-----------------|
| Diagramas de arquitectura | README principal, informe académico, presentaciones | PNG / SVG | Propiedad del proyecto |
| Banners | README principal, perfil de repositorio | PNG | Propiedad del proyecto |
| Contexto amazónico | Informe académico, README principal | JPG | Atribución MAAP / Mongabay |
| Mapa de áreas de estudio | Informe académico, README principal | PNG | Propiedad del proyecto |
| Paleta de colores | GeoTIFF embebida, figuras Python, mapas QGIS | SVG / PNG | Propiedad del proyecto |
| Leyenda estandarizada | Mapas impresos, presentaciones, posters | PNG | Propiedad del proyecto |
| Logos institucionales | Portadas, encabezados, agradecimientos | PNG | Universidad de la Amazonia |

---

## Relación con otros directorios

- **`results/`**: Contiene los productos científicos derivados (mapas, gráficos, tablas). Los recursos en `assets/` son elementos de diseño gráfico que facilitan la comunicación de esos resultados.
- **`informe/`**: El informe académico incorpora figuras de contexto y diagramas almacenados aquí.
- **`README.md` principal**: El banner y los diagramas de arquitectura se visualizan en la landing page del repositorio.

---

## Buenas prácticas de organización

1. **Formatos preferentes**: Se recomienda mantener los diagramas en formato vectorial para documentación web, y exportar versiones de alta resolución para impresión e inclusión en PDFs.
2. **Nomenclatura**: Los archivos siguen un patrón descriptivo (`tipo_descripcion`) para facilitar su identificación y uso automatizado en scripts de generación de documentación.
3. **Atribuciones**: Las imágenes de terceros deben conservar su referencia bibliográfica original cuando se utilicen fuera del contexto académico del informe.
4. **Control de versiones**: Los archivos binarios grandes deben gestionarse mediante Git LFS o almacenamiento externo vinculado, evitando el inflado del repositorio Git.

---

## Nota sobre propiedad intelectual

Los recursos originales creados específicamente para este proyecto (diagramas, banners, paleta, leyenda) son propiedad intelectual del autor del repositorio. Las imágenes de contexto amazónico provenientes de fuentes externas se utilizan con fines académicos y de divulgación científica, manteniendo la atribución correspondiente.