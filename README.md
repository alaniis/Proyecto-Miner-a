## Contexto y motivación

En la UNAM, los fenómenos de rezago, irregularidad y abandono escolar varían significativamente por plantel, área y cohorte. Este proyecto busca extraer conocimiento útil de los datos académicos, administrativos y socioeconómicos para comprender el cómo y por qué del abandono, y con ello diseñar palancas de intervención y modelos de predicción.

##  Objetivo

Desarrollar un sistema analítico que identifique segmentos de riesgo, descubra reglas y patrones secuenciales asociados al abandono, y priorice intervenciones mediante un score calibrado y explicable.

##  Estructura general del proyecto

## Parte A — Frente analítico “duro”: Segmentación, perfilado y score calibrado

* **Modelado de segmentos:**
  Clustering con `k-means` (grid corto en *k*) y `HDBSCAN` (`min_cluster_size`, `min_samples`).
  Se evaluará **estabilidad** (Jaccard por bootstrap) y **métricas internas** (Silhouette, Davies-Bouldin).
* **Perfilado:**
  Tablas comparativas por cluster: medianas, prevalencia de abandono, top features (V de Cramer, ANOVA, KW).
  Interpretación en lenguaje de negocio (*“Riesgo por rezago acumulado con racha de 2+ reprobaciones”*).
* **Score auxiliar:**
  Regresión logística (L1/L2) con **calibración Platt/Isotónica**, AUC-PR y Brier por cohorte.
  Se incluirá curva de ganancia y umbral óptimo según cupo real de intervención.
  
* Mapa de segmentos con narrativa clara y riesgos asociados.
* Score calibrado con guía de umbrales según recursos disponibles.
* Apéndice metodológico documentando hiperparámetros probados y justificación.

## Parte B — Frente de descubrimiento “explicable”: Reglas, patrones y anomalías

* **Reglas de asociación:**
  Discretización curada (quantiles / MDL) y minado con `Apriori` y `FP-Growth`.
  Filtros: soporte ≥ 5–10% por cohorte, lift > 1.2, leverage > 0.01, conviction > 1.2.
  Poda de reglas triviales (redundantes o con consecuencia incluida en antecedente).
* **Patrones secuenciales:**
  Eventos longitudinales por semestre (Aprob/Reprob, Baja de carga, Reinscripción tardía, Extraordinarios).
  Uso de `PrefixSpan` con ventana ≤ 2 semestres y top-k por *gain*.
* **Anomalías:**
  `IsolationForest` y `LOF` para trayectorias inusuales, evaluando si son errores o casos críticos.

* Catálogo de reglas priorizadas y top-patrones secuenciales.
* Informe de interesabilidad (qué reglas se descartan y por qué).
* Láminas listas para comité o presentación ejecutiva.

## Parte C — Frente de datos, calidad, documentación y tablero “ligero”

* **Ingesta y limpieza:**
  Pipeline `C1_ingesta_limpieza.ipynb` que integra fuentes académicas, LMS y administrativas.
  Imputación (mediana/moda), codificación categórica y diccionario de datos documentado (origen, dominio, reglas).
* **Discretización reproducible:**
  `C2_discretizacion.ipynb` con bins por quantiles/MDL guardados en YAML/JSON (usados por todos los frentes).
* **Calidad y sesgos:**
  `C3_calidad_tablero.ipynb` con tasas de nulos, cobertura por variable/cohorte y chequeos de sesgo.
* **Dashboard mínimo:**
  (Streamlit o notebook interactivo) mostrando segmentos, top reglas y patrones.
  Exportación a CSV/PNG.
* **Ética y gobernanza:**
  Checklist de privacidad, minimización y comunicación de limitaciones.

* Dataset maestro documentado.
* Bins versionados.
* Tablero interactivo mínimo.
* Manual de datos y checklist ético.


## Fuentes

| Fuente                                                        | Descripción                                  | Enlace                                                                                                                               |
| :------------------------------------------------------------ | :------------------------------------------- | :----------------------------------------------------------------------------------------------------------------------------------- |
| INEGI – Tasa de abandono por entidad federativa               | Datos agregados por nivel educativo y año    | [🔗 INEGI Educación 11](https://www.inegi.org.mx/app/tabulados/interactivos/?px=Educacion_11&bd=Educacion)                           |
| Encuesta Nacional de Ingresos y Gastos de los Hogares (ENIGH) | Proxy socioeconómico y desigualdad regional  | [🔗 ENIGH 2024](https://www.inegi.org.mx/programas/enigh/nc/2024/#datos_abiertos)                                                    |
| Población en situación de pobreza                             | Contexto territorial para riesgo de abandono | [🔗 INEGI Pobreza](https://www.inegi.org.mx/app/tabulados/interactivos/?pxq=Hogares_Hogares_15_d495789b-8be5-42a9-9189-511f3953702a) |



##  Metodología CRISP-DM

1. **Comprensión del negocio**
   Definición de criterios de intervención: tutorías, becas, ajustes de carga.
   Una regla o patrón es valioso si puede mapearse a una acción disponible.
2. **Comprensión y preparación de datos**
   Auditoría de cobertura, manejo de desbalance, creación de dataset longitudinal.
   Discretización (quantiles/MDL) y estandarización reproducible.
3. **Minería de datos (descubrimiento)**
   Clustering, reglas de asociación, patrones secuenciales, detección de anomalías.
4. **Modelado supervisado (auxiliar)**
   Score de riesgo calibrado (logística L1/L2 o árboles interpretables).
   Validación temporal y curvas costo-beneficio.
5. **Evaluación y entrega**
   Métricas cuantitativas y validación cualitativa con expertos de plantel/tutorías.
   Resultados presentados en tablero y narrativa clara.

##  Estructura del repositorio

```
 mineria-educativa-unam/
├── README.md
├── data/
│   ├── raw/
│   ├── processed/
│   └── docs/
├── notebooks/
│   ├── A1_segmentation.ipynb
│   ├── A2_cluster_profiling.ipynb
│   ├── A3_logit_calibration.ipynb
│   ├── B1_rules_sequences.ipynb
│   ├── C1_ingesta_limpieza.ipynb
│   ├── C2_discretizacion.ipynb
│   └── C3_calidad_tablero.ipynb
├── src/
│   ├── utils/
│   ├── mining/
│   └── dashboards/
└── reports/
    ├── resultados/
    ├── catalogos/
    └── presentaciones/
```
