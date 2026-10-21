# Clasificación de latidos para un laboratorio virtual de bioseñales

Código, pesos y página de medición del artículo:

> M. Mendoza-Barona, C. G. Hidalgo S., V. Valdés Echeverry y C. M. Paredes,
> «Comparación de modelos de clasificación de latidos para un laboratorio virtual
> de bioseñales: desempeño inter-paciente, tamaño y explicabilidad»,
> *Tecnura*, [pendiente de publicación].

El trabajo compara Random Forest, XGBoost, una red convolucional unidimensional
y una red recurrente bidireccional sobre la base MIT-BIH Arrhythmia, con
partición inter-paciente y las cuatro clases de la norma ANSI/AAMI EC57, para
decidir cuál de ellos reemplaza la regla determinista de interpretación del ECG
de la plataforma EduSignal.

## Qué reproduce cada cuaderno

| Cuaderno | Reproduce |
|---|---|
| `paso05_xgboost_y_shap.ipynb` | Figura 1 (valores SHAP sobre XGBoost) |
| `paso08_ablacion_final.ipynb` | Tabla 7 (estrategias de mejora) |
| `paso09b_exportar_pesos.ipynb` | Genera `pesos.json` y `latidos.json` para la Tabla 8 |
| `paso11_validacion_por_paciente.ipynb` | Tablas 3 y 4 (desempeño e intervalos de confianza) |
| `paso12_tablas5y6_por_paciente.ipynb` | Tabla 5 con la definición de un latido ectópico, y Tabla 6 |
| `paso13_segmento_filtro_oclusion.ipynb` | Tabla 5 con la definición de dos o más latidos ectópicos, verificación del filtro de 0,5 Hz, prueba de oclusión y Figura 2 |

La numeración corresponde a la secuencia interna del proyecto. Solo se incluyen
los cuadernos que producen resultados reportados en el artículo; los cuadernos
exploratorios previos, superados por estos, se omiten para evitar confusión con
cifras que no aparecen en la publicación.

## Datos

La base MIT-BIH Arrhythmia es de acceso público en
[PhysioNet](https://physionet.org/content/mitdb/). **No se redistribuye en este
repositorio**: cada cuaderno la descarga al inicio con `wfdb.dl_database`, de
modo que basta con ejecutarlo.

Se descartan los cuatro registros con marcapasos (102, 104, 107 y 217), como
indica la norma AAMI, y se emplea la derivación MLII cuando está disponible.

## Cómo ejecutar

Los cuadernos están escritos para Google Colab. Súbelos, activa la **GPU T4**
en Entorno de ejecución → Cambiar tipo de entorno de ejecución, y ejecútalos de
principio a fin. Las dependencias se instalan en la primera celda.

Tiempos aproximados con GPU T4:

| Cuaderno | Duración |
|---|---|
| `paso05_xgboost_y_shap.ipynb` | 20 min |
| `paso08_ablacion_final.ipynb` | 45 min |
| `paso09b_exportar_pesos.ipynb` | 10 min |
| `paso11_validacion_por_paciente.ipynb` | 75 min |
| `paso12_tablas5y6_por_paciente.ipynb` | 25 min |
| `paso13_segmento_filtro_oclusion.ipynb` | 35 min |

Los cuadernos se publican con sus salidas, de modo que los resultados pueden
consultarse sin volver a ejecutarlos.

## Partición de los registros

Se sigue la partición inter-paciente de De Chazal *et al.*: veintidós registros
de entrenamiento (DS1) y veintidós de prueba (DS2), sin que ningún sujeto
aparezca en ambos.

De DS1 se apartan cuatro registros completos —**112, 124, 205 y 223**— para la
detención temprana, la reducción de la tasa de aprendizaje y el ajuste de los
pesos de decisión, de modo que también la validación es inter-paciente. Los
dieciocho restantes entrenan todos los modelos. DS2 se emplea una única vez,
para la evaluación final.

La selección de esos cuatro registros es determinista: se sortean con semilla
fija entre las combinaciones con al menos quince latidos de cada clase, y los
cuadernos la imprimen al ejecutarse para que pueda verificarse.

## Nota sobre reproducibilidad

Las semillas controlan la inicialización de los pesos y el barajado de los
datos, pero no la totalidad del cómputo: las operaciones de convolución sobre
GPU acumulan en orden no determinista, de modo que **dos ejecuciones con la
misma semilla pueden diferir ligeramente** si no se habilita el determinismo de
operaciones. Las diferencias observadas son del orden de 0,01 en el F1 macro,
inferiores a la desviación entre semillas que reporta el artículo.

XGBoost sí es determinista bajo la configuración empleada, al no incorporar
submuestreo de instancias ni de características.

## Medición en el navegador

`medir_navegador_final.html` reproduce la Tabla 8. Ábrelo con doble clic, sin
necesidad de servidor, y selecciona los dos archivos que pide:

- `pesos.json` — los 27 652 parámetros de la red convolucional, exportados por
  `paso09b_exportar_pesos.ipynb`.
- `latidos.json` — 500 latidos de DS2 balanceados entre clases, con la exactitud
  de referencia obtenida en Python.

La página reconstruye la arquitectura con TensorFlow.js, carga los pesos y mide
el tiempo de inferencia con los tres motores disponibles (WebGL, WASM y
JavaScript puro), en modo secuencial y por lotes. Antes de reportar tiempos
comprueba que la exactitud en el navegador coincida con la de Python; si no
coincide, la exportación es incorrecta y lo advierte.

Los pesos se transfieren como texto JSON, por lo que el archivo pesa más que el
modelo: 27 652 parámetros ocupan 108 KB en precisión simple.

## Cómo citar

Si utiliza este código, cite el artículo. El repositorio está archivado en
Zenodo con DOI [pendiente].

## Licencia

MIT. La base MIT-BIH Arrhythmia se rige por los términos de PhysioNet y no se
redistribuye aquí.
