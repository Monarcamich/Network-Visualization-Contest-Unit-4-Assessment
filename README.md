# Network Visualization Contest — Proyecto Final (UPY 2026)

**Materia:** Matemáticas Discretas
**Carrera:** Ingeniería en Redes y Análisis de Datos — 3.er cuatrimestre (RAAG)
**Institución:** Universidad Politécnica de Yucatán (UPY)
**Tipo de entrega:** Concurso de visualización de redes — lámina A4
**Notebook principal:** `Network Visualization Contest.ipynb`
**Dataset base:** MUSAE Facebook Page-Page Network (subgrafo filtrado)

---

## 1. Descripción general del proyecto

Este repositorio contiene el trabajo final de la unidad 4 de la materia de Matemáticas Discretas. El objetivo es analizar y visualizar un subgrafo del conjunto de datos MUSAE Facebook Page-Page Network, centrándose en tres ejes temáticos de la teoría de grafos:

- **Grado de los nodos** y su distribución (identificación de hubs).
- **Conectividad** mediante el estudio de componentes conexas.
- **Aristas puente (bridges)**, que representan enlaces críticos cuya remoción desconectaría el grafo.

El entregable principal es un notebook de Jupyter que produce figuras de alta calidad optimizadas para impresión en formato A4 (horizontal por defecto), listas para participar en el concurso universitario de visualización de redes.

---

## 2. Dataset

### 2.1 Origen
El conjunto de datos original es el **MUSAE Facebook Page-Page Network**, una red de páginas verificadas de Facebook en la que una arista `(u, v)` indica que existe "me gusta" mutuo entre ambas páginas. El dataset original contiene 22,470 nodos y 171,002 aristas, repartidos en cuatro tipos de página: `government`, `company`, `politician` y `tvshow`.

### 2.2 Filtrado aplicado
En lugar de utilizar un muestreo aleatorio simple —que destruiría la estructura de comunidades y dejaría aristas huérfanas—, se aplicó un **muestreo por bola de nieve estratificado (stratified snowball sampling)**. El detalle completo del procedimiento está documentado en `ficha_tecnica_filtrado.md`. En resumen:

1. Se eligieron 2 semillas aleatorias por cada uno de los 4 tipos de página (8 semillas en total).
2. Desde cada semilla se expandió por BFS alternando entre las 8 colas hasta acumular 290 nodos.
3. Se construyó el subgrafo inducido conservando todas las aristas originales entre los nodos muestreados.
4. El subgrafo inducido contenía 2 componentes conexas (de 275 y 15 nodos); se conservó únicamente la componente mayor para asegurar un grafo único, conexo y manejable.
5. Se fijó `random.seed(42)` para garantizar reproducibilidad.

### 2.3 Resultado del filtrado
- **Nodos:** 275
- **Aristas:** 1,995
- **Componentes conexas:** 1 (grafo completamente conexo, sin nodos aislados)
- **Puentes detectados:** 41
- **Distribución por tipo de página:** government 146, politician 79, company 25, tvshow 25. Las categorías `government` y `politician` quedan sobre-representadas respecto al dataset original porque sus semillas cayeron en zonas más densamente conectadas; este sesgo está documentado en la ficha técnica.

### 2.4 Archivos del dataset
- `facebook_nodes_filtrado.csv`: 275 filas con las columnas `id`, `facebook_id`, `page_name`, `page_type`.
- `facebook_edges_filtrado.csv`: 1,995 filas con las columnas `id_1` e `id_2`, donde ambos ids pertenecen al archivo de nodos.
- `ficha_tecnica_filtrado.md`: ficha técnica con el procedimiento de muestreo y la justificación metodológica.

---

## 3. Contenido del repositorio

| Archivo | Descripción |
| --- | --- |
| `Network Visualization Contest.ipynb` | Notebook principal con todo el análisis y la generación de figuras. |
| `facebook_nodes_filtrado.csv` | Tabla de nodos del subgrafo filtrado. |
| `facebook_edges_filtrado.csv` | Tabla de aristas del subgrafo filtrado. |
| `ficha_tecnica_filtrado.md` | Documento con la metodología de muestreo y los criterios de filtrado. |
| `README.md` | Este archivo. |

---

## 4. Estructura del notebook

El notebook está organizado en secciones modulares con el siguiente orden lógico:

1. **Carga de librerías y configuración global de estilo.** Se importan `networkx`, `pandas`, `matplotlib`, `seaborn`, `adjustText` (si está disponible) y se definen la paleta de colores, los parámetros tipográficos y el tamaño de figura A4 con DPI alto.
2. **Carga de datos.** Lectura de los CSV y verificación de existencia de los archivos con un mensaje de error claro.
3. **Construcción del grafo.** Funciones `build_graph` y `set_node_attributes_from_df` que crean un `networkx.Graph` no dirigido y le asignan los atributos `page_type` y `page_name` a cada nodo.
4. **Análisis.** Cálculo de grado, identificación de la componente conexa principal y enumeración de aristas puente con `nx.bridges(G)`.
5. **Visualización.** Funciones reutilizables `compute_layout` (spring layout con semilla reproducible) y `draw_network` (dibujo modular con destacado de puentes y etiquetado del Top-N con contornos).
6. **Análisis cuantitativo adicional.** Tabla y gráfica de barras del Top 15 de páginas por grado.
7. **Hallazgos y Conclusiones.** Celdas de markdown que resumen observaciones y recomendaciones para la lámina.

---

## 5. Metodología de análisis

### 5.1 Métricas calculadas
- **Grado de cada nodo:** `dict(G.degree())`, utilizado para dimensionar los nodos y ordenar el ranking.
- **Componentes conexas:** `nx.connected_components(G)`. En este subgrafo hay una única componente de 275 nodos, por lo que el grafo es íntegramente conexo.
- **Aristas puente:** `nx.bridges(G)`, devueltas como conjunto de tuplas ordenadas para comparaciones O(1). Se identificaron 41 puentes.

### 5.2 Layout
Se utiliza un *spring layout* de NetworkX con semilla fija (`seed=42`) y un parámetro `k` calculado heurísticamente a partir del número de nodos (`k = 1 / sqrt(n) * 0.75`). Esto garantiza que las figuras son reproducibles: dos ejecuciones del notebook producirán posiciones idénticas.

### 5.3 Codificación visual
- **Color por tipo de página:** se asigna un color por categoría a partir de la paleta `COLOR_MAP`. Los nodos sin tipo reconocido reciben un gris neutral de respaldo (`#7f7f7f`).
  - `government` — azul `#0082c3`
  - `politician` — rojo anaranjado `#d41c00`
  - `company` — verde `#00ae00`
  - `tvshow` — morado `#4f01a3`
- **Tamaño proporcional al grado:** `node_size = max(6, degree) * node_scale`, con un factor de escala configurable (por defecto 2).
- **Aristas:** las aristas no puente se dibujan en gris claro con baja opacidad para reducir el ruido visual; las aristas puente se destacan en rojo coral (`#CF6767`) con mayor grosor y opacidad.
- **Etiquetas:** se rotulan únicamente los Top-N nodos por grado (por defecto Top 12 en la figura principal, Top 15 en la tabla auxiliar). Cada etiqueta lleva un contorno blanco grueso (`path_effects.Stroke`) para mejorar el contraste sobre el grafo. Si la librería `adjustText` está disponible, se aplica un algoritmo de repulsión para minimizar superposiciones; en caso contrario, se aplica un pequeño desplazamiento alterno como mitigación.
- **Tipografía y fondo:** fuente `DejaVu Sans`, fondo blanco, títulos en negrita semibold, figuras con `dpi=200` por defecto y tamaño `11.69 x 8.27` pulgadas (A4 horizontal).

### 5.4 Reproducibilidad
- Semilla aleatoria fija (`seed=42`) en el layout.
- Sin muestreo aleatorio en el notebook: los datos ya vienen filtrados.
- Las funciones de visualización son deterministas dados los mismos parámetros de entrada.

---

## 6. Requisitos e instalación

El notebook está pensado para ejecutarse en un entorno Python 3 con Jupyter. Las dependencias principales son:

- `networkx`
- `pandas`
- `matplotlib`
- `seaborn`
- `adjustText` (opcional, mejora el anti-solapamiento de etiquetas; si no está instalada, el notebook la instala dinámicamente al importarla)

Instalación sugerida:

```bash
pip install networkx pandas matplotlib seaborn adjustText
```

Para abrir el notebook:

```bash
jupyter notebook "Network Visualization Contest.ipynb"
```

Los archivos CSV deben encontrarse en el mismo directorio que el notebook; de lo contrario, se lanzará un `FileNotFoundError` con un mensaje que indica cómo corregirlo.

---

## 7. Cómo ejecutar el análisis

1. Clonar o copiar el repositorio en una carpeta local.
2. Verificar que los cuatro archivos del proyecto estén en el mismo directorio: el notebook, los dos CSV y la ficha técnica.
3. Abrir el notebook en Jupyter.
4. Ejecutar las celdas en orden. La primera celda importa dependencias y valida que los CSV existan. Las celdas posteriores construyen el grafo, calculan las métricas y dibujan las figuras.
5. Para exportar la figura principal a un archivo de alta calidad para impresión:

   ```python
   plt.figure(figsize=(11.69, 8.27), dpi=300)
   # ... (cuerpo de draw_network)
   plt.savefig('lamina_a4.png', dpi=300, bbox_inches='tight')
   ```

---

## 8. Hallazgos principales

- **Heterogeneidad de grado marcada:** unos pocos nodos concentran la mayor parte de las conexiones. El nodo con mayor grado es la página de Hans-Joachim Fuchtel con 63 enlaces, seguido por EU in South Africa (54) y Maryland Department of Public Safety and Correctional Services (53).
- **Estructura bipartita funcional:** los nodos de tipo `government` y `politician` dominan el grafo (225 de 275 nodos), formando comunidades institucionales densamente conectadas. Las páginas `company` y `tvshow` aparecen como satélites más pequeños.
- **41 aristas puente:** representan los enlaces críticos cuya eliminación fragmentaría la red. Estos puentes conectan principalmente las comunidades institucionales europeas con las estadounidenses, y los hubs políticos con el resto de la red.
- **Red íntegramente conexa:** la presencia de una sola componente indica que, pese a los clusters observables, no hay islas aisladas en el subgrafo.

---

## 9. Conclusiones

1. La combinación de tamaño de nodo (grado), color por tipo y destaque de puentes permite identificar visualmente los nodos centrales y las conexiones críticas en una sola lámina.
2. La paleta utilizada es equilibrada para impresión: saturación moderada, alto contraste contra fondo blanco, y suficiente diferenciación entre las cuatro categorías de página.
3. Para la entrega del concurso se recomienda usar la figura principal generada por `draw_network` complementada con la tabla `Top 15` y, opcionalmente, una versión con DPI 300 (exportable vía `plt.savefig`) para impresión de máxima calidad.
4. El muestreo por bola de nieve estratificado, aunque introduce cierto sesgo en la distribución por tipo respecto al dataset original, preserva la estructura local de comunidades, lo que resulta más útil para fines de visualización y análisis estructural.

---

## 10. Trabajo futuro

- Incorporar métricas adicionales de centralidad (betweenness, closeness, eigenvector) para complementar el análisis de hubs.
- Aplicar detección de comunidades (por ejemplo Louvain) y colorear el grafo por comunidad en lugar de por tipo de página.
- Generar versiones alternativas: retrato A4, blanco y negro para impresión económica, y una versión interactiva con `pyvis` o `plotly` para anexar al repositorio digital.
- Comparar el subgrafo filtrado con el grafo completo MUSAE para cuantificar el impacto del sesgo de muestreo sobre métricas globales.

---

## 11. Autor y contexto académico

- **Materia:** Matemáticas Discretas — Unidad 4 (Grafos y redes)
- **Carrera:** Ingeniería en Redes y Análisis de Datos
- **Cuatrimestre:** 3.° (RAAG)
- **Universidad:** Universidad Politécnica de Yucatán
- **Año:** 2026

El proyecto se entrega como pieza del concurso universitario de visualización de redes y como evaluación final de la unidad 4 de la materia.
