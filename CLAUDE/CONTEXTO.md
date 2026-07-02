# CONTEXTO — Proyecto Obligatorio MLAD (predicción de churn)

Todo lo factual del trabajo: consigna, entorno, dataset y contenidos del curso.
Para cómo trabajar ver `INSTRUCCIONES.md`; para el avance ver
`ESTADO_Y_DECISIONES.md`.

---

## 1. Qué es el trabajo

Obligatorio de **Machine Learning para Análisis de Datos** — Licenciatura en
Sistemas, Universidad ORT. **Problema de clasificación binaria**: predecir el
abandono de clientes (`Churn`, sí/no) de una empresa de telecomunicaciones.

El énfasis está en el **análisis y la justificación de decisiones**, no solo en
la predicción. La consigna aclara que la performance del modelo no es el único
aspecto relevante: pesan la calidad de la documentación, la justificación de las
decisiones y las reflexiones de los experimentos.

### Fechas y entrega
- **Entrega:** 08/07/2026, hasta 21:00, vía gestion.ort.edu.uy.
- **Defensa oral (obligatoria y ELIMINATORIA):** 09/07/2026, en forma de
  pregunta en el segundo parcial.
- **Formato:** un único **zip/rar** (máx. 40 MB) con el **informe PDF** (máx. 12
  páginas, sin contar carátula/índice/anexos) + **todo el código** (notebook)
  **con las celdas ya ejecutadas**.
- **Grupos:** hasta 2 personas del mismo dictado.
- Puntaje: 0 a 30.

### Competencia Kaggle
- Competencia **privada**: `mlad-primer-semestre-2026`.
- Métrica de evaluación: **AUC (Área bajo la curva ROC)**. Hay que subir
  **probabilidades** de churn (no clases 0/1), porque AUC mide el ordenamiento.
- Se exige **al menos un envío**. La **posición en el ranking NO condiciona la
  nota**: el foco es el proceso.
- **Formato del submission:** CSV con **1409 filas + header** (un cliente por
  fila, todos los del test). Columnas: `customerID` + predicción de `Churn`.
- Envío por web ("Submit Prediction") o por CLI:
  `kaggle competitions submit -c mlad-primer-semestre-2026 -f submission.csv -m "mensaje"`
- Se pueden hacer muchos envíos; al final **cuentan 5 seleccionados** (o los 5
  mejores automáticamente). Límite típico de 5/día (confirmar en pestaña Rules).

## 2. Las 6 tareas exigidas (todas van en el informe)

1. **Preprocesamiento** — análisis de calidad de datos (inconsistencias,
   faltantes, atípicos y su impacto); tratamiento de datos (limpieza/
   transformación justificada); transformación de variables.
2. **Feature Engineering** — manejo de categóricas (codificación); creación de
   nuevas variables (fundamentadas); selección de características (justificar
   inclusión/exclusión); análisis del impacto de variables.
3. **Modelos de Clasificación** — implementar **TODOS** los modelos de
   clasificación vistos en el curso y comparar su desempeño.
4. **Evaluación y selección** — validación (Holdout / Repeated Holdout /
   Cross-Validation) + búsqueda de hiperparámetros. Registrar por experimento:
   **accuracy, precision, recall, F1-score, AUC-ROC y matriz de confusión**.
   Resumir en **tablas** comparativas.
5. **Componente de investigación (elegir UNA)** — con justificación técnica
   citando fuentes:
   - **Explicabilidad**: SHAP o LIME (importancia global + explicaciones locales).
   - **Ensemble (Stacking)**: combinar ≥3 algoritmos de distinta naturaleza
     (lineal + árbol + red neuronal) y comparar formalmente vs. modelos base.
   - **Manifold learning**: t-SNE o UMAP; analizar agrupamientos y si la
     separación visual explica el rendimiento.
6. **Modelo final** — elegirlo justificando por métricas, robustez y
   observaciones; describir hiperparámetros elegidos y su impacto.

## 3. Entorno técnico

- **SO:** Windows. **Python 3.12.4** en un entorno virtual (`venv`).
- **Editor:** VS Code con extensiones Python + Jupyter.
- **Notebook:** `churn-obligatorio.ipynb` en `C:\dev\ML\obli`.
- **Librerías instaladas:** pandas, numpy, scikit-learn, matplotlib, seaborn,
  jupyter, ipykernel. (Para el componente de investigación pueden faltar: shap,
  o umap-learn — instalar si se elige esa vía.)
- **Datos:** `train.csv` y `test.csv` en la misma carpeta del notebook.
- **Activación del venv (Windows/PowerShell):** `venv\Scripts\Activate.ps1`
  (NO usar `source venv/bin/activate`, eso es Mac/Linux).
- **GitHub (opcional, backup/versionado):** `.gitignore` debe excluir `venv/` y
  `*.csv`. Repo privado recomendado. Esto NO reemplaza la entrega a Gestión.
- **Empaque de entrega:** zip con `informe.pdf` + `churn-obligatorio.ipynb`
  ejecutado (+ csv opcional). **NUNCA** incluir `venv/` en el zip.

## 4. El dataset (verificado)

- **train:** 5634 filas × 40 columnas (incluye `Churn`).
- **test:** 1409 filas × 39 columnas (sin `Churn`; es lo que se predice).
- `Churn` viene ya como **0/1** (numérica). Balance: **73.5% no churn / 26.5%
  churn** → moderadamente desbalanceado (relevante: por eso AUC/F1 importan más
  que accuracy sola).
- **`TotalCharges` ya viene como float64** → NO tiene la trampa clásica de venir
  como texto con espacios. Un problema menos.

### El dataset fue MODIFICADO por el host (no es el telecom clásico de ~21 cols)
Hay tres familias de columnas:

**(a) Originales del dataset telecom:**
`Contract, MonthlyCharges, InternetService, tenure, TotalCharges, gender,
Partner, Dependents, PhoneService, PaperlessBilling, MultipleLines,
SeniorCitizen, PaymentMethod`, y los servicios `OnlineBackup, OnlineSecurity,
DeviceProtection, TechSupport, StreamingTV, StreamingMovies`.

**(b) Derivadas que el host ya construyó (evaluar en Feature Engineering):**
- Redundantes con originales (mismo dato, otro formato): `SeniorCitizenLabel,
  PartnerLabel, DependentsLabel, PhoneServiceLabel, PaperlessLabel`.
- Agregaciones de servicios: `ServiceFamily, PaymentFamily, support_depth,
  stream_depth, service_count`.
- Binneadas: `tenure_band, monthly_charge_band`.
- Ratios/scores: `bill_per_month, bill_pressure, household_score`.

**(c) Sospechosas / sin valor predictivo (auditar y probablemente descartar):**
`profile_code, row_variant, dup_sig, col_order_key`. (Ver decisiones tomadas en
`ESTADO_Y_DECISIONES.md`.)

### Tipos (según pandas)
- **Numéricas:** MonthlyCharges, TotalCharges, tenure, SeniorCitizen,
  support_depth, stream_depth, service_count, bill_per_month, bill_pressure,
  household_score, col_order_key (+ Churn).
- **Categóricas (object):** customerID, Contract, OnlineBackup, InternetService,
  StreamingTV, DeviceProtection, MultipleLines, Dependents, gender,
  PaperlessBilling, PhoneService, TechSupport, Partner, PaymentMethod,
  StreamingMovies, OnlineSecurity, SeniorCitizenLabel, PartnerLabel,
  DependentsLabel, PhoneServiceLabel, PaperlessLabel, ServiceFamily,
  PaymentFamily, tenure_band, monthly_charge_band, profile_code, row_variant,
  dup_sig.

## 5. Contenidos del curso (para fundamentar decisiones)

Lo que se vio (usar como marco de justificación y citas en el informe):

- **Clase 2 — Resumir una distribución:** media/desvío, mediana/cuantiles,
  **boxplot**. **Correlación:** scatterplot, covarianza, correlación.
- **Clase 3 — Aprendizaje estadístico** (ingredientes, algoritmo de ML, sesgo
  inductivo). **Árboles de decisión (clasificación)**: impureza de **Gini**.
- **Clase 4 — Modelos lineales: regresión lineal** (con funciones base,
  polinomial). *(Regresión, no entra como clasificador.)*
- **Clase 5 — Árboles de regresión** (MSE, criterio de varianza). *(Regresión.)*
  **Regresión logística:** logits, **sigmoidea**, modelo logístico, **Binary
  Cross Entropy (BCE)**, con funciones base.
- **Clase 6 — Regularización:** definición general, Cost Complexity Pruning,
  **Ridge y LASSO**.
- **Clase 7 — Validación y selección:** costo empírico vs. verdadero, margen de
  generalización, **Holdout**, **Cross-Validation**. **Métricas:** accuracy,
  precision, recall, F1, dependencia del **umbral**, **curva ROC y AUC**; MSE,
  MAE, r². **Sesgo y varianza:** descomposición del error, learning curve.
- **Clase 10 — Bagging y Random Forest** (ensembles, bagging, RF). **Boosting:**
  **AdaBoost**, **Gradient Boosting**.
- **Clase 11 — Deep Learning:** perceptrón, **MLP**, regularización.
- **Clase 12 — NLP:** bag of words, TF-IDF, word embeddings. *(No aplica al
  problema.)*

### Modelos de clasificación a implementar (Tarea 3 = "todos los vistos")
1. **Árbol de decisión** (clasificación, Gini).
2. **Regresión logística.**
3. **Random Forest.**
4. **Boosting** (AdaBoost y/o Gradient Boosting).
5. **MLP** (red neuronal, Clase 11).

*Regresión lineal y árboles de regresión NO entran (son para regresión).*

### Notas de terminología (importante)
- **NO** se dictaron MCAR/MAR/MNAR ni "data leakage" como términos formales →
  usar lenguaje llano ("patrón sistemático de ausencia", "validación honesta").
- Conceptos fuertes disponibles para justificar: correlación, AUC/ROC y umbral,
  sesgo-varianza, regularización Ridge/LASSO, Holdout/CV.
- La **regresión logística requiere escalar** las variables numéricas; los
  modelos de árboles (árbol, RF, boosting) **no** lo requieren. Tenerlo en
  cuenta al armar los pipelines (transformaciones dependientes del modelo).
