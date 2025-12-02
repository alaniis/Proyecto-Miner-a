# Indicadores Municipales de Vulnerabilidad Asociada a la Deserción Escolar en México  
## (Construcción de un Índice Compuesto Mediante Minería de Datos)

Este repositorio documenta la construcción de un índice compuesto de vulnerabilidad estructural asociada a la deserción escolar a nivel municipal en México.  
El proyecto combina minería de datos, análisis multicriterio y visualización geoespacial para producir un instrumento territorial que puede utilizarse en investigación, docencia y discusión de política pública.

---

## Descripción general

El punto de partida es una idea sencilla. La deserción escolar no se distribuye de manera uniforme en el territorio, sino que se asocia con contextos de pobreza, vulnerabilidad y carencias en acceso a servicios básicos.  
El objetivo del proyecto es sintetizar estas condiciones estructurales en un índice municipal de vulnerabilidad asociada a la deserción escolar, que permita:

- identificar municipios con mayor riesgo estructural  
- clasificar el territorio en niveles de vulnerabilidad (muy bajo, bajo, medio, alto, muy alto)  
- servir como insumo cuantitativo para análisis y toma de decisiones

El índice no mide directamente la deserción, sino el contexto socioeconómico y de carencias que favorece el abandono escolar.

---

## Metodología (resumen CRISP DM)

### Comprensión del negocio

Se define el problema como la necesidad de contar con un instrumento territorial que apoye la focalización de políticas educativas y sociales.  
El índice se concibe como una herramienta para áreas de planeación educativa, instancias de política social, investigadores y organizaciones que trabajan en derecho a la educación.

### Comprensión de los datos

Se trabaja a nivel municipal con datos agregados provenientes de fuentes oficiales.  
Las variables centrales son:

- contexto económico (ingreso y gasto municipal)  
- pobreza y vulnerabilidad por carencias  
- carencias en salud y seguridad social  

Se verifica compatibilidad geográfica (clave de municipio), cobertura nacional y ausencia de valores faltantes en las variables seleccionadas.

### Preparación de los datos

Las bases de datos se integran mediante la clave de municipio y se selecciona un conjunto compacto de variables conceptualmente relevantes.  
Se construye un archivo consolidado que contiene:

- identificadores territoriales  
- siete variables numéricas principales  
- registros completos únicamente

Como primer paso de homogeneización de escalas se aplica una normalización tipo min max sobre todas las variables numéricas, utilizando un script dedicado.

### Modelado

En lugar de un modelo supervisado clásico, se construye un índice multicriterio. El proceso incluye:

- estimación de pesos para cada variable mediante SuperDecisions (enfoque tipo AHP)  
- aplicación de funciones de valor no lineales, crecientes o decrecientes según el sentido de la variable  
- agregación ponderada de las variables transformadas para obtener un índice continuo  
- aplicación de una transformación inspirada en la Ley de Weber Fechner sobre la suma agregada para mejorar la diferenciación entre municipios

A partir del índice continuo se generan también versiones categóricas con niveles de vulnerabilidad.

### Evaluación

Se comparan tres configuraciones del índice:

1. normalización min max más suma ponderada  
2. funciones de valor sin transformación adicional  
3. funciones de valor combinadas con transformación Weber Fechner

Los criterios de comparación incluyen distribución del índice, capacidad de diferenciación dentro de cada entidad federativa y comportamiento espacial en mapas coropléticos.  
El modelo final seleccionado corresponde a la tercera configuración, que ofrece una mejor separación de municipios y patrones espaciales más interpretabes.

### Despliegue

El resultado de la pipeline se despliega en varios formatos:

- archivos CSV con variables originales, variables transformadas e índices continuos y categóricos  
- notebooks que documentan paso a paso el proceso de integración, normalización, modelado y generación de mapas  
- capas geográficas (shapefiles y geojson) que permiten visualizar el índice en sistemas de información geográfica y aplicaciones web

Además se desarrolló una página web sencilla que presenta el índice, su motivación y algunos mapas coropléticos para consulta rápida.

---

## Datos y variables

Las principales fuentes de información son:

- Instituto Nacional de Estadística y Geografía (INEGI)  
- Consejo Nacional de Evaluación de la Política de Desarrollo Social (CONEVAL)  
- otras bases institucionales y académicas públicas

En el conjunto consolidado se trabajan siete variables numéricas clave (todas a nivel municipal), que se almacenan con nombres tipo:

- `INGRESO_HOGAR_TRI` (ingreso trimestral por hogar)  
- `GASTO_MUN_TRI` (gasto municipal trimestral)  
- `POBR_CAR_PROM` (pobreza por carencias promedio)  
- `VUL_CAR_PROM` (vulnerabilidad por carencias promedio)  
- `VUL_ING_POR` (porcentaje de población vulnerable por ingreso)  
- `CAR_SALUD_PROM` (carencias en acceso a salud)  
- `CAR_SEG_SOC_PROM` (carencias en acceso a seguridad social)

A partir de estas columnas se generan versiones normalizadas y transformadas mediante funciones de valor, que se emplean en la construcción del índice.

---

## Índice resultante

El índice final se presenta en dos formas complementarias.

1. Índice continuo  
   Valores numéricos derivados de la suma ponderada de funciones de valor, con ajuste Weber Fechner. Permite análisis más finos de distribución y comparaciones detalladas entre municipios.

2. Índice categórico  
   Clasificación de los municipios en niveles de vulnerabilidad (muy bajo, bajo, medio, alto, muy alto). Esta versión es más adecuada para comunicación y toma de decisiones, y se utiliza en los mapas coropléticos.

Ambas versiones se entregan en archivos procesados, listos para ser utilizados en otros análisis o integrados en sistemas externos.

---

## Estructura básica del repositorio

La estructura general del proyecto es la siguiente:

```text
data/           datos crudos, intermedios y procesados
notebooks/      exploración, modelado, construcción de índices y mapas
scripts/        utilidades para normalización y procesamiento
geo/            insumos geográficos (shapefiles, geojson)
README.md       descripción del proyecto
