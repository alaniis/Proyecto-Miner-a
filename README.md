## Contexto

En la UNAM, los fenómenos de rezago, irregularidad y abandono escolar varían significativamente por plantel, área y cohorte. Este proyecto busca extraer conocimiento útil de los datos académicos, administrativos y socioeconómicos para comprender el cómo y por qué del abandono, y con ello diseñar palancas de intervención y modelos de predicción.

##  Objetivo

Desarrollar un sistema analítico que identifique segmentos de riesgo, descubra reglas y patrones secuenciales asociados al abandono, y priorice intervenciones mediante un score calibrado y explicable.


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

