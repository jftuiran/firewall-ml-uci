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


## 8) Conclusiones.

Este proyecto me sirvió para responder una pregunta muy práctica: ¿podemos anticipar qué hará el firewall (allow/deny/drop/reset-both) solo mirando estadísticas del tráfico? Después de probar varias opciones con validación cruzada 10-fold y ajustar hiperparámetros, sí: se puede, y bastante bien.

¿Qué funcionó mejor?
El modelo que mejor balance logró entre todas las clases fue DecisionTree, con:

F1-macro (CV): 0.8897252371526377

Balanced accuracy (CV): 0.8667559702430794

Accuracy (CV): 0.9980925326092762

Hiperparámetros clave: {'clf__criterion': 'gini', 'clf__max_depth': None, 'clf__min_samples_leaf': 1, 'clf__min_samples_split': 2}

Aclaro que no me quedé solo con accuracy, porque en este dataset allow domina. Por eso prioricé F1-macro y balanced accuracy para no “hacer trampa” con la clase mayoritaria. Aun así, el modelo mantiene muy buenos números en allow, deny y drop. reset-both es la más dura (pocas filas), y ahí todavía hay margen de mejora.

Qué variables parecen tener más peso (intuición):
Los volúmenes y actividad (Bytes, Bytes Sent/Received, Packets, pkts_sent/received) y la duración (Elapsed Time (sec)) aportan mucha señal. Los puertos (origen/destino y NAT) también ayudan a contextualizar servicios/políticas.

Limitaciones que vi:

El desbalance pega fuerte en reset-both. Si miras solo accuracy, te engañas.

El modelo no conoce las reglas del firewall; aprende patrones de los datos.

Ensambles como RF/XGB son menos “explicables” de una pasada; conviene usar SHAP o importancia por permutación si lo llevamos a operación.

Qué haría después (si tuviera un rato más):

Ajustar umbrales por clase para subir el recall de reset-both sin romper el resto.

Calibrar probabilidades para tomar decisiones con confianza (Platt/Isotónica).

Probar SMOTE o, si el objetivo del curso lo permite, una versión binaria (permitir vs bloquear).

Añadir interpretabilidad (SHAP) para justificar decisiones ante el equipo de redes/seguridad.

Si hay tiempo/fecha, hacer validación temporal (entrenar con un tramo y testear en otro).

Pensar en monitoreo: data drift, métricas por clase y alertas si cae el recall de la minoritaria.

En resumen: me quedo con que sí es posible predecir la acción del firewall con buen rendimiento y, sobre todo, sin sesgarse por la clase mayoritaria. Con dos o tres ajustes de los que dejé arriba, esto podría usarse como apoyo a decisiones para revisar políticas, simular cambios y priorizar eventos antes de tocar producción.


## Licencia y créditos
- Datos: UCI Machine Learning Repository — Internet Firewall Data.
- Código: educativo / académico.
