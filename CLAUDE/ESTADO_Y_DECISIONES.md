# ESTADO Y DECISIONES — Proyecto Obligatorio MLAD

Bitácora viva del avance. Refleja el estado **real** del notebook
`churn-obligatorio.ipynb`. Actualizar a medida que se avanza.
Ver `CONTEXTO.md` (hechos) e `INSTRUCCIONES.md` (cómo trabajar).

---

## Dónde estoy

**Tarea 1 (Preprocesamiento): estructuralmente completa.**
- Análisis de calidad de datos → hecho y documentado.
- Tratamiento de faltantes → hecho y documentado.
- Análisis de outliers → hecho y documentado.
- Drop de columnas basura → ejecutado.
- Estrategia de escalado → definida y documentada.
- Celda Markdown `[9]` corregida (se quitó la marca `[Tratamiento: pendiente]`).
- ⚠️ Pendiente de cierre: evaluar efecto del escalado sobre la regresión
  logística (se cierra en Tarea 3 comparando con/sin StandardScaler).

**Tarea 2 (Feature Engineering): estructuralmente completa.**
- Selección y descarte de derivadas redundantes → hecho.
- Encoding de categóricas → hecho.
- Creación de nuevas variables → hecho.
- Análisis de correlación con target → hecho.
- ⚠️ Pendiente de cierre: importancia de features desde modelo entrenado
  (se agrega en Tarea 3 con `.feature_importances_` del árbol o RF).

**Tarea 3 a 6: no iniciadas.**

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
| 1. Preprocesamiento | 🔵 Estructuralmente completa (cierra efecto escalado en T3) |
| 2. Feature Engineering | 🔵 Estructuralmente completa (cierra importancia features en T3) |
| 3. Modelos de clasificación | ⚪ Pendiente (árbol, logística, RF, boosting, MLP) |
| 4. Evaluación y selección | ⚪ Pendiente (Holdout/CV, hiperparámetros, tablas de métricas) |
| 5. Componente de investigación | ⚪ Pendiente (elegir SHAP / Stacking / t-SNE) |
| 6. Modelo final | ⚪ Pendiente |
| Kaggle | ⚪ Pendiente el primer envío (probabilidades, formato correcto) |
| Informe PDF (≤12 pág) | 🔵 En curso — texto de T1 y T2 redactado |

---

## Pendientes inmediatos (para la próxima sesión)

1. Verificar que los markdowns de cierre T1 y T2 estén pegados en el notebook.
3. Iniciar **Tarea 3 — Modelos de clasificación**:
   - Definir X e y, y el split Holdout (train/val).
   - Implementar en orden: Árbol de decisión → Regresión logística (con Pipeline
     + StandardScaler) → Random Forest → Boosting (AdaBoost y/o GradientBoosting)
     → MLP.
   - Al entrenar el primer árbol/RF: extraer `.feature_importances_` y graficar
     → cierra el "impacto de variables" de Tarea 2 y el efecto del escalado de
     Tarea 1.
   - Registrar por modelo: accuracy, precision, recall, F1, AUC-ROC, matriz de
     confusión.

## Ideas / decisiones a discutir más adelante

- **Componente de investigación (Tarea 5):** SHAP encaja bien para la defensa
  oral (permite explicar predicciones individuales). Stacking también es viable
  con lo visto en clase (Clase 10). Decidir según tiempo disponible.
- **Kaggle submission:** hacer un envío "tonto" (prob. 0.5 para todos) para
  validar el formato CSV antes de invertir en el modelo.
- **Validación:** Cross-Validation es más robusta que Holdout simple dado el
  tamaño del dataset (5634 filas). Evaluar usar StratifiedKFold para respetar
  el balance 73.5/26.5%.
