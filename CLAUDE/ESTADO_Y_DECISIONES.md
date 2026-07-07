# ESTADO Y DECISIONES — Proyecto Obligatorio MLAD

Bitácora viva del avance. Refleja el estado **real** del notebook
`churn-obligatorio.ipynb`. Actualizar a medida que se avanza.
Ver `CONTEXTO.md` (hechos) e `INSTRUCCIONES.md` (cómo trabajar).

---

## Dónde estoy

**Tarea 1 (Preprocesamiento): completa.**
- Análisis de calidad de datos → hecho y documentado.
- Tratamiento de faltantes → hecho y documentado.
- Análisis de outliers → hecho y documentado.
- Drop de columnas basura → ejecutado.
- Estrategia de escalado → definida y documentada (Pipeline con StandardScaler).

**Tarea 2 (Feature Engineering): completa.**
- Selección y descarte de derivadas redundantes → hecho.
- Encoding de categóricas → hecho.
- Creación de nuevas variables → hecho.
- Análisis de correlación con target → hecho.

**Tarea 3 (Modelos de Clasificación): completa.**
- 6 modelos implementados: Árbol, Logística, RF, GB, AdaBoost, MLP.
- Todos evaluados sobre el mismo holdout 80/20 estratificado.
- Markdowns actualizados con tablas de resultados base.

**Tarea 4 (Evaluación y Selección): completa.**
- Holdout estratificado 80/20 (4507/1127, tasa churn 0.265 en ambos).
- GridSearchCV 5-fold sobre GB (27 combinaciones × 5 = 135 modelos).
- Métricas completas: accuracy, precision, recall, F1, AUC-ROC, matrices de confusión.
- Modelo seleccionado: GB Tuneado (lr=0.05, max_depth=2, n_estimators=200, subsample=0.8).
- AUC-ROC holdout: 0.8404.
- Submission Kaggle generada (submission.csv, 1409 filas, probabilidades).

**Tarea 5 (Componente de Investigación): completa — SHAP.**
- Técnica elegida: SHAP (Lundberg & Lee, 2017) con `TreeExplainer` sobre GB Tuneado.
- Global (bar chart): hecho. Top variables: Contract_Month-to-month, tenure, InternetService_Fiber optic, OnlineSecurity_No, PaymentMethod_Electronic check.
- Global (beeswarm): hecho — confirma dirección del impacto.
- Local (waterfall): hecho — churner de mayor riesgo (idx=1042, P=0.891) y cliente de menor riesgo (idx=968, P=0.015). Los mismos factores actúan en direcciones opuestas (simetría).
- Markdown Tarea 5: redactado y pegado en el notebook.

**Tarea 6 (Modelo Final): completa.**
- Justificación por métricas (mejor AUC-ROC), robustez (CV interno 0.8437 vs holdout 0.8404, diff 0.0033) y observaciones (Árbol solo 0.6641 vs ensembles; Logística competitiva sugiere señal ~lineal).
- Evidencia cuantitativa del impacto de `max_depth`: a mayor profundidad, baja el AUC-ROC promedio y sube el desvío entre folds (2→0.8378/0.0065, 3→0.8294/0.0116, 4→0.8256/0.0119).
- **Experimento probado y descartado**: reentrenar el modelo final con el 100% de los datos (train completo) antes de predecir sobre test. Bajó el score público de Kaggle → se revirtió, se mantiene `gs.best_estimator_` (entrenado solo sobre X_train) como el submission final. Vale como reflexión honesta para el informe.

---

## Estado del notebook (~27 celdas ejecutadas)

- `[0]` imports + carga train/test + `head()`.
- `[1]` `train.info()`.
- `[2]` faltantes por columna.
- `[3]` `list(train.columns)`.
- `[4]` confirma que los faltantes caen juntos (98 servicios / 73 pago+support).
- `[5]` inspección de columnas sospechosas (nunique + ejemplos).
- `[6]` base vs shadow + head de 10 filas.
- `[7]` duplicados ignorando IDs (crea columna auxiliar `es_dup`).
- `[8]` tasa de churn por variant + existencia de shadow en test.
- `[9]` **(Markdown)** "Análisis de calidad de datos" — ⚠️ todavía dice
  `[Tratamiento: pendiente]`, quitar esa línea.
- `[10]` valores de servicios + faltantes en test.
- `[11]` imputación de categóricas con "Unknown" (train y test).
- `[12]` inspección de `support_depth`.
- `[13]` recálculo de `support_depth` (train y test). Resultado: 0 NaN.
- `[14]` **(Markdown)** "Tratamiento de valores faltantes".
- `[15]` boxplot + cuantiles de MonthlyCharges, TotalCharges, tenure.
- `[16]` **(Markdown)** "Análisis de valores atípicos". Decisión: no se tratan.
- `[17]` drop de columnas basura (customerID, col_order_key, dup_sig,
  profile_code, row_variant, es_dup). Resultado: train 35 cols / test 34 cols.
- `[18]` tabla de rangos numéricos (`cols_numericas.agg`). Justifica escalado.
- `[19]` **(Markdown)** "Cierre Tarea 1" — drop de columnas + estrategia de
  escalado + inconsistencia SeniorCitizen.
- `[20]` auditoría de columnas `*Label` (pares orig ↔ label).
- `[21]` drop de `*Label` + drop de `ServiceFamily` y `PaymentFamily`.
  Resultado: train 28 cols / test 27 cols.
- `[22]` auditoría de derivadas restantes (tenure_band, monthly_charge_band,
  bill_per_month, bill_pressure, stream_depth, service_count, household_score).
- `[23]` drop de derivadas redundantes (stream_depth, household_score,
  tenure_band, monthly_charge_band, bill_per_month, bill_pressure).
  Resultado: train 22 cols / test 21 cols.
- `[24]` valores únicos de cada categórica (para definir estrategia de encoding).
- `[25]` encoding: mapeo 0/1 para binarias + one-hot para multi-valor +
  reindex test. Resultado: train 50 cols / test 49 cols.
- `[26]` creación de `charge_per_service` e `is_new_customer`.
  Resultado: train 50 cols / test 49 cols → **50 y 49 columnas finales**.
- `[27]` correlación de numéricas con Churn + gráfico de barras horizontales.

---

## Decisiones tomadas

### Tarea 1 — Preprocesamiento

**Faltantes — patrón sistemático**
- 98 filas pierden los 4 servicios de internet juntos. 73 pierden PaymentMethod +
  support_depth juntos. El test replica el mismo patrón (25 por columna).
- Decisión: imputar categóricas con categoría propia **"Unknown"** (no la moda).
  El patrón es sistemático → la moda sería incorrecto; el modelo puede aprender
  si la ausencia se asocia al churn. Eliminar filas descartado: test exige
  predecir 1409 clientes.

**`support_depth`**
- Confirmado: conteo de servicios de soporte en "Yes" (OnlineSecurity,
  OnlineBackup, DeviceProtection, TechSupport). Se recalculó desde las 4 columnas
  tras imputar (no imputar a ciegas). Los ex-NaN quedan en 0 (correcto: "Unknown"
  no es "Yes").

**Columnas descartadas en Tarea 1** (no son atributos del cliente):
- `customerID`, `col_order_key`, `dup_sig`, `profile_code`, `row_variant`,
  `es_dup`.

**Inconsistencia SeniorCitizen**: detectada durante auditoría de Labels (Tarea 2).
400 filas contradicen SeniorCitizen (0/1) con SeniorCitizenLabel. Se conserva
SeniorCitizen (original, ya numérica) y se descarta la Label.

**Outliers**: TotalCharges muestra cola derecha marcada (máx ≈ 11011). No son
errores: son clientes con cuota alta y larga permanencia. No se tratan. La
sensibilidad de la logística se resuelve con escalado, no eliminando datos.

**Escalado**: `StandardScaler` dentro del `Pipeline` de logística y MLP
(ajustado solo en train de cada fold). Árboles, RF y boosting no lo requieren.
Efecto sobre la logística se evalúa en Tarea 3.

---

### Tarea 2 — Feature Engineering

**Columnas descartadas en Tarea 2** (derivadas redundantes del host):

| Columna | Motivo |
|---|---|
| `SeniorCitizenLabel`, `PartnerLabel`, `DependentsLabel`, `PhoneServiceLabel`, `PaperlessLabel` | Duplican sus originales (mapeo 1-a-1 verificado con datos) |
| `ServiceFamily` | Concatenación exacta de InternetService + Contract |
| `PaymentFamily` | Concatenación de PaymentMethod + PaperlessBilling (mismatch solo en filas imputadas) |
| `stream_depth` | Reconstruible exacto: StreamingTV + StreamingMovies en "Yes" |
| `household_score` | Reconstruible exacto: Partner + Dependents en "Yes" |
| `tenure_band`, `monthly_charge_band` | Bins con rangos solapados; versión continua es mejor |
| `bill_per_month` | Corr 0.97 con TotalCharges/tenure |
| `bill_pressure` | Corr 0.96 con MonthlyCharges/tenure |

**Columna conservada**: `service_count` (1818 filas no coinciden con nuestra
reconstrucción desde las columnas de servicios → captura info adicional).

**Encoding**:
- Binarias puras (Dependents, PaperlessBilling, PhoneService, Partner, gender):
  mapeo directo 0/1.
- Multi-valor (Contract, OnlineBackup, InternetService, StreamingTV,
  DeviceProtection, MultipleLines, TechSupport, PaymentMethod, StreamingMovies,
  OnlineSecurity): one-hot encoding, todas las categorías, dtype=int.
- Consistencia train/test: encode en train → reindex test.

**Nuevas variables creadas**:
- `charge_per_service` = MonthlyCharges / (service_count + 1). Corr con
  Churn: +0.243 (más que MonthlyCharges solo, +0.183).
- `is_new_customer` = 1 si tenure ≤ 12. Churn: 47.4% nuevos vs 17.3% resto.
  Segunda variable más correlacionada con Churn (+0.315).

**Correlaciones numéricas con Churn** (análisis pre-modelo):
- tenure: -0.354 (más fuerte)
- is_new_customer: +0.315
- charge_per_service: +0.243
- MonthlyCharges: +0.183
- support_depth: -0.164
- SeniorCitizen: +0.113
- TotalCharges: -0.200
- service_count: -0.023

📊 Gráfico de barras horizontales generado en celda [27] → **incluir en el
informe PDF** (sección "Análisis del impacto de variables").

**Dataset final de modelado**: 49 variables predictoras + Churn.

---

## Roadmap general

| Tarea | Estado |
|---|---|
| 1. Preprocesamiento | ✅ Completa |
| 2. Feature Engineering | ✅ Completa |
| 3. Modelos de clasificación | ✅ Completa (6 modelos: Árbol, Logística, RF, GB, AdaBoost, MLP) |
| 4. Evaluación y selección | ✅ Completa (Holdout + GridSearchCV + tablas + matrices) |
| 5. Componente de investigación | ✅ Completa — SHAP (global bar + beeswarm + waterfall churner/no-churner) |
| 6. Modelo final | ✅ Completa (justificación + hiperparámetros + evidencia) |
| Kaggle | ✅ Subido — Score público: **0.86247 AUC-ROC** |
| Informe PDF (≤12 pág) | ⚪ Pendiente redacción formal |
| Uso de IA citado | ⚠️ Pendiente — no está redactado en ningún lado todavía |

---

## Pendientes inmediatos

1. **Informe PDF**: redacción formal de las 6 tareas (≤12 páginas sin carátula/índice/anexos). Notebook ya tiene todo el contenido y las 6 secciones Markdown — el informe es una adaptación de eso, no una redacción desde cero.
2. **Uso de IA**: falta redactar la cita obligatoria (herramienta + contexto de uso) para el informe.
3. **Empaque final**: zip/rar ≤40MB con informe.pdf + notebook ejecutado, sin `venv/`.
4. **Kaggle**: score público 0.86247 (suficiente, ya cumple "al menos un envío" — la letra aclara que el ranking no condiciona la nota).

## Decisiones tomadas — Tareas 3 y 4

**Modelo ganador**: GB Tuneado (AUC-ROC 0.8404)
- Hiperparámetros: lr=0.05, max_depth=2, n_estimators=200, subsample=0.8
- GridSearchCV 5-fold, optimizando AUC-ROC, 135 modelos entrenados
- AUC-ROC CV interno: 0.843 (consistente con holdout → sin overfitting)

**Ranking final por AUC-ROC**:
1. GB Tuneado: 0.8404
2. AdaBoost: 0.8398
3. Gradient Boosting: 0.8380
4. Regresión Logística: 0.8352
5. MLP: 0.8296
6. Random Forest: 0.8128
7. Árbol de Decisión: 0.6641

**AdaBoost**: segundo mejor AUC-ROC (0.8398), muy cerca del GB base.
Destacable porque AdaBoost es un ensemble diferente al GB (secuencial, sin 
subsample, minimiza error de clasificación vs. pérdida diferenciable).

**Submission Kaggle**: generada con `gs.best_estimator_.predict_proba(test)[:,1]`
— probabilidades entre 0 y 1, formato customerID + Churn, 1409 filas.
