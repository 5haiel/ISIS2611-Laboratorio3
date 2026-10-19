# Laboratorio 3 - La tarea de clasificación

ISIS2611 - Aprendizaje de Máquina · Caso Ruta Alpes

*Shaiel Jiménez - 202323846*
*Juan Esteban Triviño - 202315338*

Ruta Alpes recomienda cupones a los conductores mientras se desplazan. Este laboratorio predice si un conductor
acepta (*acepta_cupon*) un cupón de **Cafetería** o de **Restaurante(<20)**, compara regresión logística, árbol de
decisión y Random Forest, evalúa si balancear las clases (SMOTE) ayuda en Restaurante, identifica las variables más
relevantes y analiza los riesgos de sesgo de personalizar con datos demográficos.

## Estructura del repositorio

```
.
├── data/
│   └── raw/                         # Datos originales (no se modifican)
│       ├── Lab3_Datos.xlsx
│       ├── Lab3_datos_Prueba.xlsx
│       └── Lab3_diccionario_Datos.xlsx
├── sm.jimenezp1_j.trivinon.ipynb    # Notebook principal (con las celdas ejecutadas)
├── Lab3_datos_Prueba.csv            # Predicciones sobre el archivo de prueba
├── requirements.txt
└── README.md
```

## Cómo correrlo

```bash
python3 -m venv .venv
source .venv/bin/activate
pip install -r requirements.txt

jupyter notebook sm.jimenezp1_j.trivinon.ipynb
```

Hay que ejecutarlo desde la raíz del repositorio, porque lee los datos de `data/raw/` y escribe
`Lab3_datos_Prueba.csv` en esa misma carpeta. Una ejecución completa tarda unos minutos (las búsquedas de hiperparámetros son lo más lento).

Para exportar a HTML:

```bash
jupyter nbconvert --to html sm.jimenezp1_j.trivinon.ipynb
```

Se necesita scikit-learn 1.8 o superior, porque la regresión logística usa `l1_ratio` en lugar del parámetro
`penalty`, que quedó obsoleto. El notebook se probó con Python 3.14, scikit-learn 1.9.1, imbalanced-learn 0.14.2,
pandas 3.0.6 y numpy 2.5.4.

## Contenido del notebook

| Sección | Qué hace |
|---|---|
| Actividad 1 | Exploración: calidad (completitud, unicidad, consistencia, validez), distribución del objetivo en cada cupón, relación de cada variable con la aceptación y redundancia |
| Actividad 2 | Limpieza y preparación: columnas descartadas, duplicados, partición 75/25 estratificada y `ColumnTransformer` |
| Actividades 3 a 5 | Regresión logística, árbol de decisión y Random Forest, cada uno en un `Pipeline` con `GridSearchCV` |
| Actividad 6 | SMOTE frente a no balancear (y `class_weight` como referencia) en Restaurante(<20) |
| Actividad 7 | Tabla comparativa en prueba y selección del mejor modelo por cupón |
| Actividad 8 | Variables más relevantes (importancia por impureza y por permutación) y reglas del árbol |
| Actividad 10 | Predicciones sobre `Lab3_datos_Prueba` |
| Análisis de resultados | Respuesta a las 8 preguntas del enunciado |

## Decisiones principales

- **Validación por conductor.** El archivo no trae identificador de conductor y cada persona aparece en varios
  escenarios, así que se aproxima con las variables que la describen. Los hiperparámetros se eligen con
  `StratifiedGroupKFold` para que un conductor no esté a la vez en entrenamiento y validación; con folds aleatorios,
  un bosque sin límite memorizaba conductores (F1 macro de 0.751 frente a 0.697 al separarlos).
- **Métrica.** F1 macro, porque en Restaurante solo el 29.3% rechaza y la exactitud engaña.
- **Preprocesamiento dentro del pipeline.** La moda para imputar y el escalado se ajustan solo con el
  entrenamiento de cada fold.
- **Prueba intacta.** El conjunto de prueba no se usa para elegir hiperparámetros ni modelos.

## Resultados

F1 macro de los modelos elegidos:

| Cupón | Modelo | Entrenamiento | Validación (por conductor) | Prueba |
|---|---|---|---|---|
| Cafetería | Random Forest | 0.731 | 0.717 | 0.711 |
| Restaurante(<20) | Random Forest con SMOTE | 0.750 | 0.688 | 0.687 |

Las diferencias entre los tres algoritmos son pequeñas, dentro de la variación entre folds. En Cafetería la
aceptación depende sobre todo del hábito del conductor (*frecuencia_cafeteria*) y en Restaurante, de forma más débil,
de la vigencia del cupón.

## Entrega

- Notebook (`.ipynb` y `.html`) con las celdas ejecutadas.
- Video explicativo (máximo 3 minutos).
- `Lab3_datos_Prueba.csv` con las predicciones.
- Fecha límite: **19 de octubre, 8:00 p.m.** (penalización del 30% hasta el 20 de octubre, 2:00 a.m.; después de esa
  hora, nota 0).
