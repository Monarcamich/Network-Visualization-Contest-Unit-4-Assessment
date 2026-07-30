# Ficha técnica — Filtrado del grafo MUSAE Facebook

## Datos originales
- `musae_facebook_target.csv`: 22,470 nodos (páginas de Facebook)
- `musae_facebook_edges.csv`: 171,002 aristas (mutual likes entre páginas)
- Tipos de página: government (30.6%), company (28.9%), politician (25.7%), tvshow (14.8%)
- El grafo completo es una sola componente conexa (no hay nodos aislados)

## Criterio de filtrado/muestreo
Se usó **muestreo por bola de nieve estratificado (stratified snowball sampling)** en lugar de un muestreo aleatorio simple de nodos, porque el muestreo puramente aleatorio destruye la estructura de comunidades y deja demasiadas aristas huérfanas (nodos sin vecinos en la muestra).

Pasos:
1. **Semillas**: se eligieron 2 nodos aleatorios por cada uno de los 4 tipos de página (8 semillas en total), para garantizar representación de todas las categorías desde el inicio.
2. **Expansión (snowball/BFS)**: desde cada semilla se exploraron vecinos mediante BFS, alternando entre las 8 colas, hasta acumular 290 nodos. Esto preserva la conectividad local real del grafo original.
3. **Subgrafo inducido**: se tomaron todas las aristas del grafo original que conectan pares de nodos dentro de la muestra.
4. **Depuración final**: el subgrafo resultante tenía 2 componentes conexas (275 y 15 nodos); se conservó solo la componente conexa más grande para entregar un grafo único, conexo y manejable de analizar/visualizar.
5. Semilla aleatoria fija (`random.seed(42)`) para reproducibilidad.

## Resultado
- **Nodos finales**: 275
- **Aristas finales**: 1,995
- **Una sola componente conexa** (sin nodos aislados)
- Distribución por tipo en el subgrafo: government 146, politician 79, company 25, tvshow 25
  (nota: government y politician quedan sobre-representados respecto al dataset original porque las semillas de esos tipos cayeron en zonas del grafo más densamente conectadas; se documenta como sesgo conocido del muestreo)

## Archivos entregados
- `facebook_nodes_filtrado.csv`: id, facebook_id, page_name, page_type (275 filas)
- `facebook_edges_filtrado.csv`: id_1, id_2 (1,995 filas), referencian únicamente ids presentes en el archivo de nodos
