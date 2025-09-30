# Proyecto ML — Telecom/Redes (UCI): Internet Firewall Data

**Objetivo:** Clasificar la acción del firewall (`Action`) a partir de métricas de flujo (puertos, bytes, paquetes, tiempo).  
**Clases:** `allow`, `deny`, `drop`, `reset-both` (desbalanceadas).

## Dataset
- Fuente: UCI Machine Learning Repository — *Internet Firewall Data* (archivo `log2.csv`).
- Instancias: ~65k; Atributos: 11 numéricos + 1 objetivo (`Action`).

## Metodología
- EDA (nulos, distribución de clases, estadísticos).
- Preprocesamiento: imputación mediana + estandarización (`StandardScaler`) de numéricas.
- Validación: **10-fold estratificada**.
- Modelos entrenados con **GridSearchCV**: Árboles, Bagging (LR y DT), RandomForest, AdaBoost, XGBoost (opcional).
- Métrica principal: **F1-macro**; secundarias: `balanced_accuracy`, `accuracy`.

## Resultados (resumen)
| model                                         |   accuracy_cv |   f1_macro_cv |   balanced_accuracy_cv | best_params                                                                                                                                                                     |
|:----------------------------------------------|--------------:|--------------:|-----------------------:|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Decision Tree — Grid + Reporte                |        0.9981 |        0.8897 |                 0.8668 | {'clf__criterion': 'gini', 'clf__max_depth': None, 'clf__min_samples_leaf': 1, 'clf__min_samples_split': 2}                                                                     |
| Bagging (DecisionTree) — Grid + Reporte       |        0.9983 |        0.8764 |                 0.8479 | {'clf__bootstrap': True, 'clf__estimator__max_depth': None, 'clf__estimator__min_samples_leaf': 1, 'clf__max_features': 1.0, 'clf__max_samples': 1.0, 'clf__n_estimators': 200} |
| Random Forest — Grid + Reporte                |        0.9981 |        0.8738 |                 0.8485 | {'clf__max_depth': None, 'clf__max_features': 0.6, 'clf__min_samples_leaf': 1, 'clf__min_samples_split': 10, 'clf__n_estimators': 500}                                          |
| XGBoost (opcional) — Grid + Reporte           |        0.9984 |        0.8505 |                 0.8356 | {'clf__colsample_bytree': 1.0, 'clf__learning_rate': 0.2, 'clf__max_depth': 8, 'clf__min_child_weight': 1, 'clf__reg_lambda': 2.0, 'clf__subsample': 1.0}                       |
| AdaBoost — Grid + Reporte                     |        0.9553 |        0.7253 |                 0.8853 | {'clf__learning_rate': 0.2, 'clf__n_estimators': 200}                                                                                                                           |
| Bagging (LogisticRegression) — Grid + Reporte |        0.9191 |        0.698  |                 0.8019 | {'clf__bootstrap': True, 'clf__estimator__C': 1.0, 'clf__max_features': 0.8, 'clf__max_samples': 1.0, 'clf__n_estimators': 10}                                                  |

**Mejor modelo:** Decision Tree — Grid + Reporte — F1-macro CV: 0.8897

> Nota: el mejor se selecciona por F1-macro (mejor equilibrio entre clases).

## Reproducibilidad
```bash
pip install -r requirements.txt
# opcional
pip install xgboost imbalanced-learn
```
Ejecutar: abrir el notebook `Proyecto_ML_Telecom_Firewall.ipynb` descargar el dataset, modificar la ruta del dataset y correr todas las celdas.

## Artefactos
- `/artifacts/best_model.joblib` — pipeline completo (prep + modelo) y clases del label encoder.
- `/artifacts/metadata.json` — hiperparámetros y métricas CV del mejor.

## Conclusiones (plantilla)
- El modelo Decision Tree — Grid + Reporte obtuvo el mejor **F1-macro** sobre CV, manteniendo buenas métricas en clases mayoritarias y razonable desempeño en las minoritarias.
- Las variables de volumen/actividad (`Bytes`, `Packets`, `Elapsed Time (sec)`) muestran señal para distinguir acciones.
- Siguientes pasos: calibración de probabilidades, *threshold tuning* para minoritarias (`reset-both`), interpretación (SHAP / permutaciones).

## Licencia y créditos
- Datos: UCI Machine Learning Repository — Internet Firewall Data.
- Código: educativo / académico.
