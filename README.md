# **Poke-Laboratorio 7: Ensambles de Votación: ¡Atrapa el mejor modelo!**

## **Introducción y Objetivos**

Este proyecto investiga si la combinación de clasificadores heterogéneos mejora la capacidad para distinguir Pokémon "fuertes" y "no fuertes" cuando sólo se observan variables simples y estables. Partimos de la PokéAPI para construir un conjunto de datos reproducible; definimos la etiqueta de "fuerza" a partir del percentil 75 del puntaje total (suma de las seis estadísticas base); y, para lograr estimaciones más estables y comparables entre subpoblaciones, extraemos una muestra estratificada jerárquica de tamaño 500 según la intersección type_main $x$ strong. Sobre esa muestra entrenamos cuatro modelos base (KNN, Regresión Logística, Árbol de Decisión y SVM) utilizando únicamente variables visibles ( height , weight , base_experience, type_main codificado). Finalmente integramos tres esquemas de votación (dura, suave y ponderada) y seleccionamos el ensamble con mejor desempeño validado para generar el archivo de predicciones con el formato requerido para evaluación externa. En conjunto, el flujo garantiza trazabilidad (desde la API hasta el CSV final), control de sesgo por composición (vía estratificación), y una comparación justa entre clasificadores y ensambles.

## **Integrantes:**

| Participante | Rol en el proyecto | Aportes  |
| :--- | :--- | :--- |
| Abril Minerva Estrada Montaño | Adquisición y curación de datos | Descarga desde PokéAPI, construcción del CSV canónico, cálculo de power_score y strong |
| Erick José Fabián Sandoval | Muestreo y preparación | Diseño del muestreo estratificado type_main $\times$ strong, codificación de type_mai |
| Alanis González Sebastian | Modelado base | Entrenamiento y sintonía de KNN, Regresión Logística, Árbol y SVM |
| Melisa Ashareth Arano Bejarano | Ensambles y reporte | Implementación de votación dura/suave/ponderada, selección del ensamble final|

# **Parte 1. Construcción del dataset: descarga, etiqueta y muestra estratificada**

El proceso inicia con la descarga sistemática del universo de Pokémon a través de la ruta pública https://pokeapi.co/api/v2/pokemon/\{id\} . No se toma una muestra temprana ni un subconjunto adhoc. Se recorre la secuencia de identificadores y, para cada registro válido, se extraen de forma homogénea los campos necesarios. El CSV canónico resultante reúne, por fila, las cuatro características visibles ( height , weight , base_experience, type_main) y las seis estadísticas base (hp, attack, defense, special_attack, special_defense, speed) que se usan únicamente para construir la etiqueta. A partir de esas seis se define el power_score como suma simple y la etiqueta binaria strong como indicador de si el power_score cae por encima del percentil 75 del universo. Esa decisión evita fugas de información porque las estadísticas que definen la etiqueta no entran en el vector de entrada; también hace que la noción de "fuerte" sea objetiva y reproducible. Para la trazabilidad, el CSV final conserva al menos trece columnas en orden lógico: identificador/nombre, las cuatro visibles, las seis estadísticas, power_score y strong Además, el pipeline tolera alias razonables de nombres (por ejemplo, "altura" por height, "tipo" por type_main) normalizando encabezados y detectando equivalentes comunes, de modo que no se rompa si alguien exporta columnas con variantes simples.

Con el universo calculado, no entrenamos directamente sobre él. Para estabilizar las métricas y hacer comparables los resultados por subpoblaciones, fijamos una muestra de trabajo de tamaño 500 mediante muestreo estratificado jerárquico en el producto type_main $\times$ strong . El orden es deliberado: primero se calculan power_score y strong en el universo; después se forman los estratos combinando el tipo elemental con la clase fuerte/no fuerte; por último, se asignan cuotas para sumar exactamente 500 observaciones, ajustando estratos muy pequeños cuando es necesario para no perder cobertura. El resultado se materializa en un segundo CSV por ejemplo, pokemon_samples_500.csv y es la base de modelado. Este diseño reduce varianza muestral, evita dominancias de tipos mayoritarios y mejora la lectura por subpoblaciones, porque tanto la clase strong $=1$ como strong $=0$ quedan representadas dentro de cada tipo elemental en la medida en que lo permite el universo.

# **Parte 2. Modelos individuales (KNN, Regresión Logística, Árbol y SVM)**

Con la muestra estratificada establecida, definimos el vector de entrada con las cuatro características visibles ( height , weight , base_experience y la codificación explicita de type_main ) y fijamos strong como la variable objetivo. Las estadísticas que fabrican la etiqueta y el propio power_score no entran en el entrenamiento, para preservar el principio de no fugas. Trabajamos con un esquema de partición de entrenamiento y validación dentro de la muestra de 500, dejando el test externo del profesor como un conjunto completamente ciego hasta la fase final.

Entrenamos cuatro familias de clasificadores que ofrecen sesgos inductivos complementarios. El KNN captura estructuras locales en el espacio de características, por lo que requiere escalamiento cuidadoso $y$ una elección de $k$ que balancee sesgovarianza. La Regresión Logística plantea una frontera lineal, entrega probabilidades y con frecuencia provee una base sólida para la combinación suave. El Árbol de Decisión modela interacciones no lineales mediante particiones jerárquicas; su poder expresivo se controla con límites de profundidad u otros parámetros para evitar sobreajuste. La SVM busca maximizar el margen; con kernel lineal o RBF, reclama estandarización y una sintonía cuidadosa de c (y de v si se usa RBF). De cada modelo documentamos desempeño en validación y una lectura cualitativa que explica cuándo y por qué funciona mejor o peor (por ejemplo, sensibilidad de KNN a escalas, estabilidad de la Logística, trade-off de profundidad en Árbol, efecto de márgenes en SVM). El objetivo de esta parte no es encontrar "el mejor modelo absoluto", sino preparar una base diversa que tenga sentido combinar.

Todos los modelos se empacan en pipelines de scikit-learn que aplican imputación mediana a variables numéricas, escalamiento estándar cuando corresponde y codificación one-hot de type_main. Esto asegura que el mismo preprocesamiento se replique idéntico en validación y, más tarde, en el test externo. En consola registramos una EDA mínima: balance de clases, algunas correlaciones entre numéricas y la etiqueta, y tasas de strong por tipo elemental, útiles para interpretar resultados. Para estimar el rendimiento individual, realizamos validación cruzada estratificada con tres particiones y reportamos media y desviación estándar de accuracy por modelo. Además, mantenemos una verificación de prohibidos en tiempo de ejecución que revisa el nombre de las clases y aborta si alguien intenta colar AdaBoost, Gradient Boosting, Random Forest, XGBoost o equivalentes.

# **Parte 3. Ensambles de votación: dura, suave y ponderada**

Los métodos de ensamble de votación combinan las predicciones de varios clasificadores base para obtener una decisión final más robusta. Sea un conjunto de $M$ modelos $\left\{h_1, h_2, \ldots, h_M\right\}$ que predicen una etiqueta binaria $y \in\{0,1\}$ a partir de una entrada $x$.

**Votación dura** (majority voting): cada clasificador emite una votación $h_j(x) \in \{0,1\}$, y la clase final se determina por mayoría:

$$
\hat{y}= \begin{cases}1, & \text { si } \sum_{j=1}^M h_j(x)>\frac{M}{2} \\ 0, & \text { en otro caso. }\end{cases}
$$

Este método asume que todos los clasificadores tienen el mismo peso y que los errores individuales son independientes. La votación dura toma las etiquetas binarias de todos los modelos y decide por mayoría simple; en términos de cómputo, se apilan las predicciones en una matriz de tamaño $M \times n$ (M modelos por n ejemplos) y se predice 1 si el conteo de unos supera M/2. Es robusta y muy interpretable, pero al ignorar la intensidad de la evidencia puede quedarse corta cuando algunos clasificadores son mucho más confiables que otros.

**Votación suave** (soft voting): en lugar de usar decisiones binarias, cada modelo produce una probabilidad $p_j(y=1 \mid x)$. La predicción final se obtiene al promediar las probabilidades:

$$
\hat{y}= \begin{cases}1, & \text { si } \frac{1}{M} \sum_{j=1}^M p_j(y=1 \mid x) \geq 0.5 \\ 0, & \text { en otro caso. }\end{cases}
$$

Este método suele ser más estable porque aprovecha la confianza de cada modelo. La votación suave utiliza probabilidades $P(y=1 \mid x)$ de cada modelo que las expone. Promedia las probabilidades y decide por umbral 0.5. Esta estrategia suele estabilizar el desempeño cuando al menos una parte de los modelos emite probabilidades razonablemente calibradas (la Regresión Logística y LDA suelen ayudar). En el pipeline solo se añaden al "panel suave" los modelos con predict_proba disponible, para no introducir aproximaciones frágiles.

**Votación ponderada** (weighted voting): asigna un peso $w_j$ a cada clasificador en función de su desempeño (por ejemplo, su exactitud en validación):

$$
\hat{y}= \begin{cases}1, & \text { si } \frac{\sum_{j=1}^M w_j p_j(y=1 \mid x)}{\sum_{j=1}^{M 1} w_j} \geq 0.5 \\ 0, & \text { en otro caso. }\end{cases}
$$

Los pesos permiten dar mayor influencia a los modelos más confiables. La votación ponderada es una extensión de la suave que aprende pesos para cada modelo a partir del desempeño validado. En cada fold se reentrenan los modelos con el conjunto de entrenamiento del fold, se estima su accuracy en el conjunto de validación del mismo fold y esas exactitudes se usan como pesos, normalizados para sumar uno. La probabilidad final del ensamble se calcula como promedio ponderado de las probabilidades individuales. Este mecanismo permite que el ensamble confie más en los clasificadores que mejor funcionan en esa partición, sin necesidad de introducir modelos prohibidos ni optimizaciones meta-complejas. Si por cualquier motivo los pesos de un fold degeneran (caso límite), se recurre a un promedio uniforme como red de seguridad.

El principio fundamental es que, si los clasificadores cometen errores de manera no correlacionada, la combinación por votación tiende a reducir la varianza del sistema y mejorar la capacidad de generalización. Para comparar objetivamente los tres votadores, utilizamos validación cruzada estratificada con la misma partición y preprocesamiento. En cada fold se reentrenan los modelos base, se instancia el votador correspondiente (duro, suave o ponderado) y se mide la accuracy en el subconjunto de validación del fold. Al final se reportan media y desviación estándar por votador. Ese mismo loop nos da una lectura clara de estabilidad: si un votador fluctúa mucho entre folds, se documenta y se prefiere el que mantenga rendimiento alto con menor dispersión. Con esa evidencia se elige el ensamble final: el que combina mayor exactitud validada con mejor estabilidad.






