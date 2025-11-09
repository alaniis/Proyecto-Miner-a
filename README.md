# Laboratorio 9: AdaBoost y contenedores

En este repositorio se explica el propósito de la práctica, desarrolla con rigor la teoría fundamental de AdaBoost, describe el diseño de la implementación propia y detalla el protocolo experimental de comparación contra scikit-learn. Además, especifica el contrato de la API de inferencia, los criterios de validación de entradas, y los lineamientos de empaquetado y despliegue con Docker y Makefile. La intención es que el lector pueda comprender, reproducir y auditar cada decisión sin consultar fuentes externas.

## 1. Motivación y objetivos

Boosting es una metodología para transformar clasificadores débiles en un clasificador fuerte mediante la combinación aditiva de modelos entrenados secuencialmente. En particular, AdaBoost es uno de los algoritmos más influyentes por su simplicidad y por las garantías teóricas que lo acompañan. En esta práctica se pide: implementar AdaBoost desde cero utilizando *decision stumps* como clasificadores base; comparar su desempeño contra AdaBoostClassifier de scikit-learn bajo un protocolo experimental controlado; exponer el pipeline de inferencia mediante una API REST que aplica el mismo preprocesamiento que el entrenamiento; y entregar un entorno reproducible vía Docker y Makefile. El énfasis está en el rigor: trazabilidad del preprocesamiento, control de semillas, reporte de métricas en conjuntos separados y descripción explícita de supuestos. La práctica se estructura en tres elementos principales, cada uno con sus propios objetivos específicos:

### Elemento 1: Implementación desde cero de AdaBoost
El objetivo es comprender y programar el principio del algoritmo AdaBoost (Adaptive Boosting), utilizando clasificadores débiles secuenciales para construir un modelo fuerte.\
El enfoque principal de este elemento es aprender cómo el modelo ajusta los pesos de las observaciones en función de los errores previos, logrando una combinación ponderada de predictores débiles que reduce el sesgo del sistema.

### Elemento 2: Comparativa con sklearn
El objetivo es evaluar la implementación propia de AdaBoost frente al modelo de referencia \texttt{sklearn.ensemble.AdaBoostClassifier}.\
Esto implica analizar la precisión, estabilidad y convergencia del ensamble. El propósito de esta comparación es validar que la versión implementada capture correctamente el comportamiento adaptativo del algoritmo y comprender los efectos del número de clasificadores y la distribución de pesos.

### Elemento 3: API, contenedor Docker y automatización con Makefile
El objetivo es exponer el modelo de AdaBoost entrenado mediante una API REST simple.\
Adicionalmente, se debe empaquetar la aplicación en un contenedor Docker ejecutable en local y automatizar las tareas principales (construir, ejecutar, detener y empaquetar; es decir, \textit{build}, \textit{run}, \textit{stop}, \textit{package}) utilizando un Makefile. La meta de este elemento es que el docente pueda evaluar cada solución de forma reproducible, ejecutando unos cuantos comandos \texttt{make} y probando la API.

## 2. Fundamentos teóricos de AdaBoost

### 2.1. De la idea de boosting a la formulación de AdaBoost

El punto de partida es suponer que contamos con un clasificador débil capaz de tener un error ligeramente menor que el azar sobre la distribución de entrenamiento. El boosting entrena una sucesión de clasificadores débiles $h_1, \ldots, h_T$ sobre reponderaciones adaptativas de los ejemplos y los combina por una regla de voto ponderado. Sea $\left(x_i, y_i\right)_{i=1}^n$ el conjunto de entrenamiento con etiquetas binarias $y_i \in\{-1,+1\}$. En la iteración $t$ se mantiene una distribución de pesos $D_t$ sobre los índices $i$, se entrena $h_t$ minimizando el error ponderado $\epsilon_t=\sum_i D_t(i) \mathbf{1}\left[h_t\left(x_i\right) \neq y_i\right]$ y se asigna un peso al clasificador $\alpha_t=\frac{1}{2} \log \left(\frac{1-\epsilon_t}{\epsilon_t}\right)$. A continuación se actualiza la distribución:

$$
D_{t+1}(i)=\frac{D_t(i)\,\exp(-\alpha_t\, y_i\, h_t(x_i))}{Z_t}
$$

donde \(Z_t\) es un factor de normalización. El clasificador final es el signo de la suma ponderada:

$$
F_T(x)=\sum_{t=1}^T \alpha_t\, h_t(x), \qquad \hat{y}(x)=\operatorname{sign}(F_T(x))
$$


### 2.2. Conexión con minimización de pérdida exponencial

AdaBoost puede verse como un procedimiento de minimización de la pérdida exponencial

$$
\mathcal{L}(F)=\sum_{i=1}^n \exp \left(-y_i F\left(x_i\right)\right),
$$

mediante forward stagewise additive modeling. en cada paso se agrega un término $\alpha h$ eligiendo $h$ en una clase base $\mathcal{H}$ y un peso $\alpha$ que minimizan la pérdida en dirección greedy. Con etiquetas en $\{-1,+1\}$, el margen en $x_i$ es $y_i F\left(x_i\right)$, y la pérdida exponencial penaliza márgenes negativos de forma muy severa, lo que induce a concentrar masa sobre ejemplos mal clasificados. La fórmula cerrada para $\alpha_t$ es consecuencia de minimizar $\sum_i D_t(i) \exp \left(-\alpha y_i h_t\left(x_i\right)\right)$ respecto a $\alpha$, con $D_t(i)$ ya incorporando la historia previa.

### 2.3. Márgenes, error de entrenamiento y capacidad de generalización

Un resultado clásico muestra que el error de entrenamiento de AdaBoost está acotado por $\prod_{t=1}^T Z_t$, y que $Z_t=2 \sqrt{\epsilon_t\left(1-\epsilon_t\right)}$ en el caso binario. Por tanto, si cada $\epsilon_t<1 / 2$, el error de entrenamiento decrece exponencialmente con $T$. Más interesante aún, a pesar de que la pérdida exponencial puede seguir decreciendo, el error de generalización puede estabilizarse o incluso mejorar si el algoritmo continúa incrementando márgenes, un fenómeno frecuentemente observado en práctica. La teoría de márgenes sugiere que la distribución de márgenes (no solo su promedio) es crítica para explicar el buen comportamiento de AdaBoost frente a sobreajuste en muchos conjuntos.

### 2.4. Sensibilidad a ruido, valores atípicos y extensiones multiclase (SAMME y SAMME.R)

La pérdida exponencial es más agresiva que la logística frente a ejemplos dificiles. En presencia de ruido de etiqueta, AdaBoost puede asignar pesos desproporcionados a observaciones imposibles de clasificar, causando inestabilidad en $\alpha_t$ y degradación de desempeño. Por ello, es recomendable introducir mecanismos de regularización: shrinkage (tasa de aprendizaje), tope en $\alpha_t$, parada temprana o control de la complejidad del débil (p. ej., mantener stumps de profundidad 1 o 2 , o realizar poda). Para $K>2$ clases, scikit-learn utiliza SAMME (Stagewise Additive Modeling using a Multiclass Exponential loss) y su variante real SAMME.R. En SAMME, la actualización de $\alpha_t$ incorpora un término $\log (K-1)$, y los clasificadores débiles devuelven etiquetas discretas; en SAMME.R, los débiles devuelven estimaciones de probabilidad y las actualizaciones se hacen con funciones reales, típicamente mejorando la eficiencia. En esta práctica nos centramos en el caso binario, pero la implementación puede generalizarse si el débil provee $\hat{p}(y \mid x)$.

### 2.5. Relación con logística y *gradient boosting*

AdaBoost es forward stagewise con pérdida exponencial, mientras que el gradient boosting con pérdida logística desciende en la dirección del gradiente negativo en el espacio de funciones, ajustando regresores a los residuos de la pérdida. En datasets ruidosos, la logística suele ser más robusta. Teóricamente, con aprendizaje suficientemente pequeño y gran número de iteraciones, ambos métodos se acercan a soluciones con buenos márgenes, aunque con perfiles de regularización distintos.

## 3. Especificación del algoritmo implementado

La implementación propia ( SimpleAdaBoost ) emplea como débil un árbol de decisión con max_depth=1 . Se espera lo siguiente:
1. Etiquetas en $\{-1,+1\}$ durante el fit. Si el conjunto original usa $\{0,1\}$, se mapea $\{0 \mapsto-1,1 \mapsto+1\}$ de forma consistente y se documenta en el preprocesamiento.
2. Inicialización uniforme $D_1(i)=1 / n$.
3. Entrenamiento del stump con sample_weight=D_t.
4. Cálculo del error ponderado $\epsilon_t$, con recorte numérico $\epsilon_t \leftarrow \min \left(\max \left(\epsilon_t, 10^{-12}\right), 1-10^{-12}\right)$.
5. Peso del clasificador $\alpha_t=\frac{1}{2} \log \left(\frac{1-\epsilon_t}{\epsilon_t}\right)$.
6. Actualización y normalización de $D_{t+1}$ -
7. Predicción como signo de la suma ponderada.

Se registran por iteración $\epsilon_t, \alpha_t \mathrm{y}$, si se desea para diagnóstico, estadísticas agregadas de $D_t$ (mínimo, máximo, entropía). La clase expone learners_, alphas_, errors_ y predict . La estabilidad numérica se cuida evitando $\epsilon_t \in\{0,1\}$ exactos y controlando funciones exponenciales.

## 4. Preprocesamiento, datos y consistencia de la API

El preprocesamiento define imputación de faltantes, codificación de categóricas (p. ej., one-hot) y estandarización de numéricas. Se serializa el pipeline como preprocessor.pk1 . El modelo entrenado -implementación propia o AdaBoostClassifier - se serializa como model.pkl. La API de inferencia debe aplicar el mismo preprocessor.transform antes de invocar model.predict. El contrato de entrada se define sobre nombres de columnas "crudas" legibles. La coherencia entre entrenamiento y despliegue exige que las columnas del JSON coincidan exactamente con las esperadas por el preprocesador. Si el objetivo del entrenamiento se representó como $\{-1,+1\}$, la API devuelve por convenio $\{0,1\}$ o $\{-1,+1\}$, pero ese convenio debe quedar explícito y ser estable.

## 5. Protocolo experimental, regularización, *tuning* y consideraciones prácticas

El protocolo mínimo considera una división estratificada entrenamiento/validación con semilla fija. Las métricas a reportar incluyen exactitud, precisión, recall y F1. Además de comparar la implementación propia contra AdaBoostClassifier manteniendo el mismo stump y el mismo número de estimadores, se estudia la sensibilidad a n_estimators y, cuando sea aplicable, la evolución del error por iteración $\epsilon_t$. Los resultados deben presentarse con tablas o texto explicativo, no solo con figuras, y deben incluir una interpretación: concordancias, discrepancias, causas probables (ruido, atípicos, colinealidad), y una discusión del compromiso sesgo-varianza al incrementar $T$. El control de sobreajuste en AdaBoost se consigue mediante varias palancas. La primera es la complejidad del débil: stumps tienden a ser más estables que árboles profundos. La segunda es el número de iteraciones $T$ : puede fijarse por validación cruzada o emplearse parada temprana cuando la métrica en validación deje de mejorar. La tercera es la tasa de aprendizaje $\nu \in(0,1]$, aplicando una reducción $\alpha_t \leftarrow \nu \alpha_t$ (shrinkage), lo que típicamente requiere más iteraciones pero suaviza el ajuste. Otras variantes incluyen ponderar clases desbalanceadas mediante sample_weight inicial no uniforme, o incorporar penalizaciones que limiten la influencia de ejemplos con pérdidas extremas. El tuning debe realizarse en un conjunto de validación o por CV para evitar sesgo optimista.

## 6. Complejidad computacional

Con stumps entrenados por particiones binarias, el costo por iteración es del orden de $O(n d \log n)$ si se clasifican umbrales por característica, o $O(n d)$ con estrategias lineales y pocas categorías; multiplicado por $T$, la complejidad total es $O(T \cdot$ coste_stump $)$. En la práctica, la implementación de scikit-learn es altamente optimizada; nuestra versión docente prioriza claridad sobre micro-optimizaciones. El consumo de memoria crece linealmente con el número de débiles almacenados. Aunque AdaBoost produce puntajes $F_T(x)$ que correlacionan con confianza, no son probabilidades calibradas. Si el uso requiere umbrales distintos o probabilidades bien calibradas, conviene aplicar Platt scaling o isotonic regression sobre los puntajes en un conjunto de calibración separado. La API podría exponer, opcionalmente, un endpoint de calibración o devolver además del rótulo un puntaje continuo para decisiones con umbral.

## 7. Diseño de la API de inferencia

La API se implementa típicamente con FastAPI. Debe cargar preprocessor.pk1 y model.pkl al iniciar. Se recomiendan tres rutas:
- /health devuelve un diagnóstico mínimo para orquestación.
- /info expone metadatos del modelo, del pipeline y del equipo, de modo que un evaluador pueda confirmar versión, número de estimadores y columnas esperadas.
- /predict recibe un único registro en un objeto JSON con la clave "features" y un diccionario de columna: valor. La función construye un DataFrame de una fila, aplica preprocessor.transform y llama a model.predict. Los errores de validación (faltantes, categorías inválidas, tipos incompatibles) devuelven 400/422 con un mensaje detallado que enumere los campos problemáticos.

El contrato debe quedar inmutable durante la evaluación. Si se necesitan cambios, deben versionarse explícitamente (por ejemplo, /v2/predict ) para no romper clientes.

## 8. Seguridad, robustez y auditoría de entradas

Aunque se trata de un entorno académico, es importante contemplar ataques triviales: entradas gigantes, tipos incorrectos, JSON malformados. La aplicación debe imponer límites de tamaño al cuerpo, validar tipos y valores permitidos, y registrar de manera controlada los errores sin volcar datos sensibles. Los logs deben incluir timestamp, ruta, código de estado y, si procede, un identificador de solicitud. No deben imprimirse tracebacks completos al cliente. Toda la experimentación debe ser reproducible con una semilla global definida. El repositorio debe fijar versiones mínimas en app/requirements.txt para la ejecución de la API y, si se desea, disponer de un requirements-dev.txt con dependencias de análisis (Jupyter, gráficos) que no se instalan en la imagen. Los artefactos que afectan la inferencia ( preprocessor.pkl y model.pkl ) se versionan mediante nombres estables y se incluyen en la imagen. El cuaderno de comparación notebooks/ contiene las celdas que generan las métricas reportadas y puede regenerar figuras y tablas.

## 9. Estructura del repositorio




## 10. Docker y Makefile: Protocolo de ejecución y comprobación

La imagen de Docker se basa en python:3.10-slim, copia el repositorio, instala dependencias y levanta uvicorn . El puerto de exposición es 8000 . Un Makefile con reglas build, run, status, stop, clean y package automatiza el ciclo. La regla package genera un tarball equipo_<NOMBRE>.tar.gz con todo lo necesario para evaluar. Se recomienda comprobar localmente que el flujo funciona en limpio: extraer el tarball en un directorio nuevo, construir la imagen, iniciar el contenedor, consultar /health $y$ /predict con un ejemplo mínimo. Para construir y ejecutar, primero se crea la imagen. Luego se ejecuta el contenedor con mapeo del puerto 8000 al host. Se verifica la salud con una petición GET a /health. Se inspecciona /info para comprobar que el modelo y el preprocesamiento se cargaron como se esperaba. Finalmente, se envía un registro de prueba a /predict. La salida debe ser determinista dada la semilla y el pipeline. Si el preprocesamiento espera columnas categóricas específicas, el ejemplo de prueba debe respetar exactamente esos dominios para evitar respuestas de error.

## 11. Resultados esperados y limitaciones

En conjuntos moderadamente limpios, la implementación propia con stumps debería aproximarse a AdaBoostClassifier cuando se igualan n_estimators y la semilla del débil. Las diferencias residuales suelen provenir de detalles internos de partición y manejo de empates en el árbol base. Es importante reportar no solo un número agregado de exactitud sino una discusión del comportamiento frente a cambios en $T$, y, si es viable, una inspección de la curva de $\epsilon_{\ell}$. Un patrón típico muestra $\epsilon_{\ell}<0.5$ de forma consistente y una disminución de la pérdida exponencial incluso cuando la exactitud valida se estabiliza, lo que merece explicación en términos de márgenes y de la sensibilidad a ejemplos difíciles. AdaBoost es potente y, a la vez, frágil frente a etiquetas ruidosas. En presencia de ruido sustancial se recomiendan estrategias como shrinkage y parada temprana. Para clasificación multiclase real, SAMME.R suele ser preferible si el débil produce probabilidades. Más allá de esta práctica, se pueden explorar pérdidas alternativas (logística) y conectarlas con gradient boosting, así como el uso de regularización explícita en los débiles o reglas de recorte de $\alpha_t$ ante $\epsilon_t$ extremos. Otra extensión útil es la calibración posterior de puntajes para producir probabilidades bien calibradas cuando la aplicación lo requiera.

