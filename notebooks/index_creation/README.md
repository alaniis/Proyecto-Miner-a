# 📉 Generación y Ponderación de Índices Sintéticos (MIN-MAX y Weber-Fechner)

![Python](https://img.shields.io/badge/Python-3.11-blue?style=for-the-badge&logo=python&logoColor=white)
![Pandas](https://img.shields.io/badge/Pandas-Data_Analysis-150458?style=for-the-badge&logo=pandas&logoColor=white)
![Geopandas](https://img.shields.io/badge/Geopandas-Geospatial-139C5A?style=for-the-badge&logo=python&logoColor=white)
![Matplotlib](https://img.shields.io/badge/Matplotlib-Visualization-005F9E?style=for-the-badge&logo=python&logoColor=white)

Este módulo constituye la fase analítica y de modelado del proyecto. Su propósito es transformar las variables socioeconómicas crudas en **Índices Sintéticos de Vulnerabilidad y Oportunidad**. Se implementan metodologías de normalización estadística (Min-Max), transformación mediante **Funciones de Valor (Utilidad)** y clasificación basada en la **Ley de Weber-Fechner**, culminando con una visualización geoespacial a nivel municipal.

---

## 🧾 Descripción General

El notebook procesa el dataset maestro generado previamente para construir métricas comparables entre municipios. A diferencia de una suma lineal simple, este análisis incorpora la **subjetividad paramétrica** mediante funciones de valor (cóncavas y convexas) que modelan cómo cada variable impacta realmente en la calidad de vida o en la deserción escolar (e.g., la utilidad marginal del ingreso es decreciente, mientras que el impacto de la pobreza es exponencialmente negativo). Finalmente, se generan mapas coropléticos para identificar clusters geográficos de riesgo.

---

## 🧩 Arquitectura del Modelo de Índices

```mermaid
flowchart TD
    Input[(data_final_proyecto_final.csv)] --> Normalizacion[Normalización Min-Max]
    Input --> Funciones[Funciones de Valor\n(Cóncavas/Convexas)]
    
    Normalizacion --> Ponderacion1[Ponderación Lineal]
    Funciones --> Ponderacion2[Ponderación No-Lineal]
    
    Ponderacion1 --> Suma1[Índice SUMA Simple]
    Ponderacion2 --> Suma2[Índice SUMA Valorado]
    
    Suma1 --> Clasificacion[Clasificación por Quantiles & Weber-Fechner]
    Suma2 --> Clasificacion
    
    SHP[(Shapefile Municipios)] --> JoinGeo{Join Espacial}
    Clasificacion --> JoinGeo
    
    JoinGeo --> Mapas[Visualización Geoespacial]
```
# 📂 Documentación Técnica del Notebook (Indices.ipynb)
1. 📏 Normalización Lineal (Min-Max)
## Objetivo
Estandarizar todas las variables numéricas (ingresos, gastos, carencias) en una escala común de [0, 1] para permitir operaciones aritméticas entre unidades heterogéneas.

## Procedimiento

1. Identificación de variables de interés (INGRESO_HOGAR_TRI, POBR_CAR_PROM, etc.).
2. Aplicación de la fórmula de normalización Min-Max:$$x_{norm} = \frac{x - \min(x)}{\max(x) - \min(x)}$$
3. Manejo de valores atípicos y nulos durante la transformación.

## Bases de datos utilizadas
data_final_proyecto_final.csv (Dataset consolidado).

## Creación/Eliminación de columnas
Creación: Se generan columnas con prefijo norm_ (e.g., norm_INGRESO_HOGAR_TRI) que representan el valor relativo de cada municipio.

2. 🧮 Transformación mediante Funciones de Valor
### Objetivo
Modelar el comportamiento no lineal de las variables socioeconómicas. Se asume que el impacto de una variable no siempre es proporcional a su magnitud (e.g., un aumento de ingreso en zonas pobres tiene más impacto que en zonas ricas).

### Procedimiento
Definición de funciones matemáticas de utilidad (tomadas de teoría de decisión multicriterio):
Cóncava Creciente: Para variables con rendimientos decrecientes (Ingreso, Gasto).
Convexa Decreciente: Para variables de impacto negativo acelerado (Pobreza, Vulnerabilidad).
Aplicación de parámetros de forma (gamma, alpha) para ajustar la curvatura de la función.
Ponderación de las variables transformadas según su importancia relativa en el modelo.

### Snippet de Código (Funciones de Transferencia):

```python
    """
    Transformación para variables donde el impacto negativo crece
    exponencialmente (ej. Pobreza Extrema).
    """
    # ... lógica de normalización ...
    num = np.exp(gamma * (1.0 - x_norm)) - 1.0
    den = np.exp(gamma) - 1.0
    return num / den
```
### Creación/Eliminación de columnas

Creación: Columnas con sufijo _val (e.g., POBR_CAR_PROM_val) que representan la "utilidad" o "desutilidad" de la variable.

Creación: SUMA_val (Índice sintético final basado en funciones de valor).

# 3. 📊 Clasificación Weber-Fechner y Estratificación
## Objetivo 
Categorizar los índices continuos obtenidos en niveles discretos de riesgo ("Muy Bajo" a "Muy Alto") utilizando una escala logarítmica que simula la percepción humana de las magnitudes (Ley de Weber-Fechner).

## Procedimiento

1. Implementación de algoritmo de binning basado en progresión geométrica/logarítmica.

2. Comparación con cortes tradicionales (lineales).

3. Asignación de etiquetas categóricas a cada municipio.

## Resultados obtenidos

Variables categóricas INDICE (Lineal) e indice_wf (Weber-Fechner) añadidas al dataframe.

# 4. 🗺️ Visualización Geoespacial
## Objetivo 
Representar espacialmente la distribución de los índices generados para identificar patrones regionales de vulnerabilidad educativa y económica.

## Procedimiento
1. Carga de la geometría municipal de México (2024_1_00_MUN.shp).

2. Estandarización de claves geográficas (CVEGEO) para realizar el cruce con el dataframe de índices.

3. Unión (Merge) espacial tipo Left Join.

4. Generación de un panel de 4 mapas comparativos utilizando matplotlib y geopandas, contrastando los índices lineales vs. valorados y las clasificaciones simples vs. Weber-Fechner.
## Bases de datos utilizadas

2024_1_00_MUN.shp (Marco Geoestadístico Nacional - INEGI).

## Resultados obtenidos

Visualización gráfica de alta resolución que muestra la disparidad regional, destacando zonas críticas (e.g., Sierra de Oaxaca, zonas rurales de Chiapas) frente a centros económicos.

🧱 Estructura del Entregable

.
├── data/
│   ├── data_final_proyecto_final.csv  # Entrada principal
│   └── shp/
│       └── 2024_1_00_MUN.shp          # Geometría municipal
├── notebooks/
│   └── Indices.ipynb                  # Notebook de modelado y visualización
└── README.md

# 👥 Equipo de Desarrollo
El diseño de las funciones de valor y el análisis geoespacial fue realizado por:

**Alanís González Sebástian**
**Fonseca González Bruno**
**Minerva Estrada Montaño Abril**

# 📜 Licencia y Uso
Este código es parte de una investigación académica sobre deserción escolar. Las funciones de transformación matemática pueden ser reutilizadas citando a los autores. Los mapas generados utilizan datos públicos del INEGI.