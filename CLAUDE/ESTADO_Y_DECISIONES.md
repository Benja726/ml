# ESTADO Y DECISIONES — Proyecto Obligatorio MLAD

Bitácora viva del avance. Refleja el estado **real** del notebook
`churn-obligatorio.ipynb`. Actualizar a medida que se avanza.
Ver `CONTEXTO.md` (hechos) e `INSTRUCCIONES.md` (cómo trabajar).

---

## Dónde estoy

**Tarea 1 (Preprocesamiento): casi cerrada.**
- Análisis de calidad de datos → **hecho y documentado**.
- Tratamiento de faltantes → **hecho y documentado**.
- Análisis de outliers → **hecho y documentado** (celdas [15] y [16]).
- Falta cerrar Tarea 1: **drop de columnas basura** + **transformación de
  variables** (escalado dependiente del modelo). Ambos van juntos al inicio
  de Tarea 2 / armado del pipeline.

Tareas 2 a 6: **no iniciadas.**

## Estado del notebook (17 celdas)

Orden actual, todas ejecutadas:
- `[0]` imports + carga train/test + `head()`.
- `[1]` `train.info()`.
- `[2]` faltantes por columna.
- `[3]` `list(train.columns)`.
- `[4]` confirma que los faltantes caen juntos (98 servicios / 73 pago+support).
- `[5]` inspección de columnas sospechosas (nunique + ejemplos).
- `[6]` base vs shadow + head de 10 filas.
- `[7]` duplicados ignorando IDs.
- `[8]` tasa de churn por variant + existencia de shadow en test.
- `[9]` **(Markdown)** "Análisis de calidad de datos" — ⚠️ dice
  `[Tratamiento: pendiente]`, quitar esa marca.
- `[10]` valores de servicios + faltantes en test.
- `[11]` imputación de categóricas con "Unknown" (train y test).
- `[12]` inspección de `support_depth`.
- `[13]` recálculo de `support_depth` (train y test).
- `[14]` **(Markdown)** "Tratamiento de valores faltantes".
- `[15]` boxplot + cuantiles de MonthlyCharges, TotalCharges, tenure.
- `[16]` **(Markdown)** "Análisis de valores atípicos" — decisión: no se
  tratan; TotalCharges tiene cola larga esperada (producto MonthlyCharges ×
  tenure); IQR sobre-marca distribuciones asimétricas. Escala se trata con
  el modelo (logística escala, árboles no).

## Decisiones tomadas (con su porqué)

### Faltantes — patrón sistemático
- 98 filas pierden los 4 servicios de internet **juntos** (OnlineBackup,
  DeviceProtection, TechSupport, OnlineSecurity). 73 pierden PaymentMethod +
  support_depth juntos. **El test tiene los mismos faltantes** (25 por columna).
- **Decisión:** imputar categóricas (los 4 servicios + PaymentMethod) con
  categoría propia **"Unknown"** (no la moda). Porqué: el patrón es sistemático
  → asumir "son como la mayoría" (moda) sería incorrecto; no se inventan datos;
  el modelo puede aprender si la ausencia se asocia al churn; y es coherente con
  el dataset, que ya usa estados explícitos ("No internet service"). Eliminar
  filas quedó descartado porque en test hay que predecir las 1409. **Aplicado
  (celda 11). Dataset sin NaN en esas columnas.**

### `support_depth` (derivada)
- Confirmado que es el **conteo de servicios de soporte en "Yes"**
  (OnlineSecurity, OnlineBackup, DeviceProtection, TechSupport).
- **Decisión:** **recalcularla** desde las 4 columnas tras imputar (no imputarla
  a ciegas). **Aplicado (celda 13).** 0 faltantes en train y test. La
  distribución cambió levemente (los ex-NaN pasan a contar 0, porque "Unknown"
  no es "Yes") → comportamiento correcto y esperado.

### Columnas a descartar (auditadas)
- `col_order_key`: ID único por fila (5634 únicos) → ruido; un árbol podría
  memorizarlo. **Descartar.**
- `dup_sig`: ID derivado (customerID + row_variant), 5599 únicos. **Descartar.**
- `profile_code`: concatenación de tenure_band + Contract + PaymentMethod (58
  únicos), redundante. **Descartar.**
- ⚠️ **Pendiente de ejecución:** la decisión está tomada pero el `drop` de estas
  columnas **todavía no se hizo** en el notebook. Hacerlo al armar el dataset de
  modelado (o antes), y documentarlo.

### Filas "shadow" (`row_variant`)
- 300 shadow / 5334 base. **No son duplicados** (solo 24 dups reales, todos
  base). Tasa de churn similar (base 26.8% / shadow 22.0%). **El test también
  tiene shadow** en proporción equivalente (78/1409 ≈ 5.5%).
- **Decisión:** **conservar las filas** shadow (el test las incluye; entrenar
  sin ellas degradaría la predicción sobre ese subconjunto). **`row_variant` no
  se usa como variable predictiva** (es marca de procedencia, no un atributo del
  cliente).

## Pendientes inmediatos (próximos pasos)

1. ⚠️ Quitar la marca `[Tratamiento: pendiente]` de la celda Markdown `[9]`.
2. **Drop de columnas basura**: ejecutar y documentar el descarte de
   `col_order_key`, `dup_sig`, `profile_code` (y de `row_variant`, `customerID`
   que tampoco son predictores). Hacerlo como primera celda de Tarea 2.
3. **Tarea 2 — Feature Engineering:**
   - Descartar columnas `*Label` (SeniorCitizenLabel, PartnerLabel,
     DependentsLabel, PhoneServiceLabel, PaperlessLabel): duplican las
     originales, ya numéricas o binarias.
   - Evaluar si `ServiceFamily`, `PaymentFamily`, `tenure_band`,
     `monthly_charge_band` aportan sobre las originales.
   - Encoding de categóricas: one-hot para nominales (Contract, InternetService,
     PaymentMethod, MultipleLines, etc.); ordinal si los bands tienen orden.
   - Escalado de numéricas: `StandardScaler` solo para el pipeline de logística
     (no para árboles).
   - Análisis de importancia de variables (post-modelo).

## Roadmap general

| Tarea | Estado |
|---|---|
| 1. Preprocesamiento | 🔵 Casi cerrada (calidad + faltantes + outliers hechos; falta drop de basura y escalado) |
| 2. Feature Engineering | ⚪ Pendiente (categóricas → encoding; evaluar las derivadas del host; selección) |
| 3. Modelos de clasificación | ⚪ Pendiente (árbol, logística, RF, boosting, MLP) |
| 4. Evaluación y selección | ⚪ Pendiente (Holdout/CV, hiperparámetros, tablas de métricas) |
| 5. Componente de investigación | ⚪ Pendiente (elegir SHAP / Stacking / t-SNE) |
| 6. Modelo final | ⚪ Pendiente |
| Kaggle | ⚪ Pendiente el primer envío (validar pipeline de submission end-to-end) |
| Informe PDF (≤12 pág) | ⚪ Se redacta al final desde las celdas Markdown |

## Ideas / decisiones a discutir más adelante

- **Encoding de categóricas:** one-hot para nominales; ordinal para las band
  (`tenure_band`, `monthly_charge_band`) si tienen orden natural.
- **Redundancia:** las `*Label` probablemente se descartan (duplican columnas
  originales) — confirmar con datos antes de tirarlas.
- **Componente de investigación:** el **Stacking** encaja natural con lo visto
  en ensembles (Clase 10) y permite combinar logística + RF/boosting + MLP.
  Alternativa fuerte: **SHAP** para explicar el modelo final (bueno para la
  defensa). Decidir según tiempo.
- **Primer envío Kaggle "tonto"** (prob. 0.5 para todos) para validar formato
  del submission antes de invertir en el modelo — opcional, no gasta nada útil.
