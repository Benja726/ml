# LETRA DEL OBLIGATORIO — MLAD (Machine Learning para Análisis de Datos)

Resumen de la consigna oficial (Universidad ORT, Licenciatura en Sistemas,
1er semestre 2026). Es la fuente de verdad de **qué se pide**; para el estado del
trabajo ver `ESTADO_Y_DECISIONES.md` y para cómo trabajar `INSTRUCCIONES.md`.

**Tema:** Análisis y Predicción de Abandono de Clientes (*Churn*) en una empresa
de telecomunicaciones. Problema de **clasificación binaria**.

---

## 1. Datos administrativos

- **Puntaje:** de 0 a 30 puntos.
- **Entrega:** 08/07/2026 hasta las 21:00, vía gestion.ort.edu.uy.
- **Defensa:** 09/07/2026, **oral, obligatoria y ELIMINATORIA**. Es en forma de
  pregunta dentro del segundo parcial. Presencial (salvo que se habilite remota).
- **Grupos:** hasta 2 personas del mismo dictado.
- **Inscripción:** hay que inscribirse antes de entregar.

## 2. Formato de entrega

- Un **único archivo zip o rar**, máximo **40 MB**, subido a Gestión.
- Contenido del zip/rar:
  - **Informe en PDF**, máximo **12 páginas** (sin contar carátula, índice ni
    anexos).
  - **Todo el código desarrollado** (notebooks o scripts de preprocesamiento,
    entrenamiento, evaluación, comparación y generación de predicciones), **con
    las celdas ya ejecutadas**.
- Los documentos de texto deben ir en PDF, dentro del zip/rar.
- Ante problemas de inscripción o técnicos: contactar a Coordinación antes de
  las 20:00h del día de entrega (adjuntos_ei@ort.edu.uy).

## 3. Competencia Kaggle

- Competencia **privada** en Kaggle. Predecir `Churn` sobre un test cuyo valor
  real se desconoce.
- **Al menos un envío** con las predicciones del modelo entrenado.
- Métrica de evaluación: **AUC (Área Bajo la Curva ROC)**. Objetivo: AUC lo más
  alto posible.
- **La posición en el ranking NO condiciona la nota.** El foco está en el
  proceso, la justificación de las decisiones y el aprendizaje.

## 4. Criterio de evaluación (leer con atención)

> La performance del modelo **no es el único aspecto relevante**.

Pesan igual o más que el número:
- Calidad de la documentación.
- **Justificación de cada decisión tomada.**
- Reflexiones obtenidas a partir de los experimentos.

---

## 5. Las 6 tareas exigidas (todas van en el informe)

### Tarea 1 — Preprocesamiento
Definir y justificar el tratamiento inicial de los datos, **incluyendo el
análisis de su impacto en el modelo**. Se espera:
- **Análisis de calidad de datos:** identificar inconsistencias, valores
  faltantes o registros atípicos, y **discutir su posible impacto**.
- **Tratamiento de datos:** definir estrategias de limpieza o transformación,
  **justificando cada decisión**.
- **Transformación de variables:** aplicar transformaciones cuando sean
  necesarias, **evaluando su efecto sobre el comportamiento del modelo**.

> Nota: la consigna pide explícitamente el **impacto/efecto en el modelo**. Ese
> componente exige, idealmente, mostrar evidencia (comparar con/sin la
> transformación) → parte de esta tarea se cierra recién con modelos entrenados.

### Tarea 2 — Feature Engineering
Proceso de creación, modificación y selección de variables que mejore el
rendimiento. Se espera al menos:
- **Manejo de categóricas:** transformarlas a un formato adecuado para el
  modelado (codificación).
- **Creación de nuevas características:** generar variables con valor predictivo,
  **fundamentando su construcción**.
- **Selección de características:** analizar críticamente las variables,
  **justificando inclusión o exclusión**.
- **Análisis del impacto de variables:** evaluar cómo distintas variables
  afectan el desempeño del modelo.

### Tarea 3 — Modelos de Clasificación
Implementar **TODOS los modelos de clasificación vistos en el curso** y
**comparar su desempeño** en la predicción.

### Tarea 4 — Evaluación y selección de modelos
- Detallar el proceso de **validación** y selección, **incluyendo búsqueda de
  hiperparámetros**. Usar **alguna** técnica vista en el curso: **Holdout,
  Repeated Holdout o Cross-Validation**.
- Para **cada experimento**, registrar: **accuracy, precision, recall, F1-score,
  AUC-ROC y matriz de confusión**.
- Resumir los resultados en **tablas comparativas** por modelo y sus métricas.

### Tarea 5 — Componente de Investigación y Valor Agregado
Elegir e implementar **UNA** de las tres técnicas, con **justificación técnica
(citando fuentes)**, metodología aplicada y análisis del valor que aporta:
- **Explicabilidad (SHAP o LIME):** desglosar la "caja negra" del modelo final;
  presentar e interpretar el impacto de variables a nivel **global**
  (importancia general) y **local** (predicciones individuales).
- **Ensemble (Stacking Classifier):** combinar **≥3 algoritmos de distinta
  naturaleza** (ej. lineal + árbol + red neuronal); comparativa formal vs.
  modelos base para demostrar si mejora.
- **Manifold Learning (t-SNE o UMAP):** reducción de dimensionalidad no lineal;
  analizar agrupamientos naturales de clientes y discutir si la separación
  visual explica el rendimiento de los clasificadores.

### Tarea 6 — Modelo final
Seleccionar un modelo final, justificando la elección por **métricas, robustez y
observaciones** de la experimentación. El informe debe incluir:
- Explicación clara de por qué ese modelo es el más adecuado.
- Breve descripción de los **hiperparámetros seleccionados y su impacto**.

---

## 6. Uso de IA (obligatorio citarlo)

La consigna permite usar IA generativa **como apoyo**, no como sustituto del
razonamiento crítico ni de la elaboración personal. Requisitos:
- **Citar** las herramientas utilizadas y el **contexto de uso** (ej. generación
  de ideas, redacción inicial, análisis de datos, corrección de estilo).
- Todo contenido de IA debe ser **revisado y verificado**; los errores son
  responsabilidad del estudiante.
- El trabajo debe reflejar la **propia comprensión** del estudiante (clave para
  la defensa oral eliminatoria).

---

## 7. Checklist rápido de cumplimiento

- [ ] Inscripción realizada + grupo formado (≤2 personas).
- [ ] Informe PDF ≤ 12 páginas (sin carátula/índice/anexos).
- [ ] Las 6 tareas presentes y justificadas en el informe.
- [ ] Tarea 3: **todos** los modelos de clasificación del curso.
- [ ] Tarea 4: métricas completas (accuracy, precision, recall, F1, AUC-ROC,
      matriz de confusión) + tablas comparativas.
- [ ] Tarea 5: una técnica de investigación, con fuentes citadas.
- [ ] Código entregado con **celdas ejecutadas**.
- [ ] Al menos **un envío** a Kaggle (probabilidades de churn; formato correcto).
- [ ] Uso de IA citado (herramientas + contexto).
- [ ] Empaque final: un solo zip/rar ≤ 40 MB, con PDF + código (sin `venv/`).
