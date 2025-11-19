# 🪙 Gold Recovery Prediction — Flotation Plant
Predicción de la recuperación de oro en etapas críticas del proceso.
<p align="center"> <img src="https://img.shields.io/badge/Status-Completed-brightgreen" /> <img src="https://img.shields.io/badge/Jupyter-Notebook-orange" /> <img src="https://img.shields.io/badge/Python-3.10-blue" /> <img src="https://img.shields.io/badge/Metric-sMAPE-purple" /> <img src="https://img.shields.io/badge/Model-RandomForestRegressor-green" /> </p>

### 📑 Tabla de Contenidos

- 📌 Descripción del Proyecto
- 🎯 Objetivo
- 📂 Datos
- 🧠 Enfoque de la Solución
- 🛠️ Tecnologías Utilizadas
- 📊 Análisis y Hallazgos Clave
- 🤖 Modelado y Métrica sMAPE
- 🧾 Conclusiones


## 📌 Descripción del Proyecto

Este proyecto se centra en una planta de procesamiento de oro que utiliza flotación y varias etapas de limpieza (cleaning stages) para enriquecer el mineral. La meta es construir un modelo capaz de predecir la recuperación de oro en etapas críticas del proceso:

- rougher.output.recovery
- final.output.recovery

La evaluación se realiza con la métrica symmetric Mean Absolute Percentage Error (sMAPE), definida así:

<p align="center">
$sMAPE = \frac{1}{N}\sum_{i=1}^N \frac{|y_i - \hat{y}_i|}{(|y_i| + |\hat{y}_i|)/2} \times 100\%$
</p>
El resultado final combina ambas etapas:

<p align="center"> $
sMAPE_{final} = 0.25 \cdot sMAPE_{rougher} + 0.75 \cdot sMAPE_{final}
$ </p>


## 🎯 Objetivo

Desarrollar un modelo de machine learning que:

✔️ prediga correctamente la recuperación rougher y final

✔️ minimice el sMAPE combinado, requerido por negocio

✔️ sea robusto ante diferencias entre train y test

✔️ utilice solo características disponibles en el entorno real (dataset test)

## 📂 Datos

El proyecto utiliza tres archivos:

- gold_recovery_train.csv → dataset de entrenamiento
- gold_recovery_test.csv → dataset de prueba (sin objetivos)
- gold_recovery_full.csv → dataset maestro con todos los datos

Características principales:

- Índice temporal (date)
- Variables físico-químicas del proceso
- Caudales, reactivos, concentraciones metálicas (Au, Ag, Pb)
- Salidas de los procesos rougher y cleaner

## 🧠 Enfoque de la Solución
### 1️⃣ Preparación de Datos

🔍 Exploración inicial

- Revisión de tipos de datos, valores nulos y estructura temporal.
- Identificación de columnas presentes en train pero ausentes en test (principalmente columnas objetivo y valores tardíos).

🧪 Validación de la fórmula de recuperación

- Se recalculó rougher.output.recovery.
- Se comparó contra la columna original → MAE ≈ 7.88e–15, confirmando consistencia.

🧹 Limpieza

- Eliminación de filas sin información de año o con anomalías físicas.
- Conversión date → datetime.
- Alineación completa entre train, test y full.

### 2️⃣ Análisis Exploratorio (EDA)

📈 Concentración de metales (Au, Ag, Pb)

- Au aumenta de manera consistente a través de rougher → cleaner → final.
- Ag aumenta pero luego disminuye en etapas profundas (más selectividad hacia Au).
- Pb se comporta de manera estable, con incrementos leves.

📊 Distribución del tamaño de partícula

- Se compararon train vs test.
- La prueba KS mostró diferencias estadísticamente significativas.
- Esto afecta la estabilidad del modelo → se optó por usar modelos robustos (RF).

⚠️ Identificación de anomalías

- Se sumó la concentración total de metales por etapa.
- Se detectaron valores cercanos a 0 o físicamente imposibles.
- Se eliminaron filas con totales < 1 para asegurar integridad del dataset.

## 🤖 Modelado y Métrica sMAPE
### 1️⃣ Creación de la Función sMAPE

Se implementó función personalizada para:

- sMAPE_rougher
- sMAPE_final
- sMAPE_total (25% rougher + 75% final)

Se integró como scorer para GridSearchCV y cross_val_score.

### 2️⃣ Entrenamiento y Selección de Modelos

Modelos probados:

| Modelo                    | Ventajas                | Resultado               |
| ------------------------- | ----------------------- | ----------------------- |
| **LinearRegression**      | simple, interpretable   | desempeño medio         |
| **DecisionTreeRegressor** | captura no-linealidades | tendencia a sobreajuste |
| **RandomForestRegressor** | robusto, estable        | **mejor desempeño**     |


Después de probar modelos con CV (cv=5), el mejor fue:

<p align="center"> <b>⭐ RandomForestRegressor </b></p>

Luego se realizó Grid Search:

<p align="center">
n_estimators = [10, 15, 20, 25, 30]
  
max_depth = [3, 4, 5, 6, 7]
</p>

#### ✔️ Mejor modelo final:

<p align="center">
  RandomForestRegressor(  
    random_state=12345,    
    n_estimators=30,    
    max_depth=4
    )

</p>

### 3️⃣ Resultados Finales

El modelo final fue entrenado de forma independiente para:

- rougher.output.recovery
- final.output.recovery

El desempeño sobre el conjunto de prueba fue:

<p align="center"><b> 🥇 sMAPE total ≈ 7.52% </b><p align="center">  </p>

Un valor excelente para un proceso industrial complejo con alto ruido en sensórica.

🧾 Conclusiones

- Se construyó un pipeline sólido para preparación, análisis y modelado.
- Se detectaron y eliminaron anomalías físicas que afectaban la estabilidad del modelo.
- El modelo RandomForestRegressor fue el más robusto frente a diferencias entre train y test.
- El sMAPE obtenido (≈7.5%) demuestra la capacidad predictiva del modelo en un entorno real.
- El proceso revela la importancia de:
  - selección correcta de variables
  - alineación entre train/test
  - métricas personalizadas
  - validación cruzada
