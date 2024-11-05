# Pipline de procesamiento de datos

## Diagrama y Secuencia de Pasos del Pipeline de Procesamiento de Datos

### Ingesta de Datos:

1. Los datos se suben a Amazon S3 en el bucket de almacenamiento en la carpeta `raw-data/`.
2. S3 activa una función de AWS Lambda al detectar un archivo nuevo, que inicia el proceso de ETL en AWS Glue.

### Preprocesamiento de Datos (ETL en AWS Glue):

- **Limpieza de Datos**: AWS Glue carga los datos y aplica las transformaciones básicas (eliminación de duplicados, corrección de formatos).
- **Transformación**: Se realizan tareas de limpieza específicas para mejorar la calidad del texto, como la eliminación de caracteres no deseados, tokenización, y generación de vectores de características con técnicas como TF-IDF o embeddings de palabras (por ejemplo, usando AWS Comprehend).
- **Enriquecimiento de Datos**: Se pueden agregar variables calculadas o realizar análisis de sentimiento usando servicios como AWS Comprehend.

Los datos procesados se almacenan en `processed-data/` en S3.

### Entrenamiento del Modelo (en Amazon SageMaker):

1. AWS Step Functions coordina el flujo y lanza un trabajo de entrenamiento en Amazon SageMaker.
2. SageMaker utiliza los datos de `processed-data/` para entrenar el modelo de clasificación.
3. **Almacenamiento del Modelo**: El modelo entrenado se guarda en S3 en `models/versioned/`.

### Inferencia en Batch (en SageMaker):

1. Los datos de entrada para inferencia se almacenan en `inference-data/` en S3.
2. SageMaker Batch Transform carga el modelo de `models/latest/` y realiza la inferencia en los datos de `inference-data/`.
3. **Resultados**: Las predicciones se guardan en S3 en `results/`.

### Monitoreo y Alerta (CloudWatch y SageMaker Model Monitor):

- **CloudWatch**: Monitorea métricas clave y alerta en caso de errores o problemas de rendimiento.
- **Model Monitor**: Evalúa la precisión del modelo con los datos nuevos, desencadenando un reentrenamiento si detecta una disminución de precisión.

## Exploración de Datos y Estrategias de Limpieza y Calidad

Se plantea un paso a paso del proceso de limpieza y calidad:

### Carga de Datos y Exploración Inicial:

1. Verifición el tamaño de los datos y el número de registros.
2. Inspección las primeras filas para revisar el formato de las variables.

### Revisión de Datos Nulos o Faltantes:

- Identificación de valores nulos. Completar valores nulos según el contexto o, si el porcentaje de nulos es alto en una columna no esencial se elimina.

### Eliminación de Duplicados:

- Busqueda y eliminación de registros duplicados si los hay. Tener en cuenta de no perder datos relevantes en la eliminación

### Normalización y Formato:

- Estandarización el texto a minúsculas para evitar duplicados basados en diferencias de mayúsculas.
- Eliminación de caracteres especiales y stop words: Para limpiar el texto de ruido y mejorar la precisión del modelo.

### Tokenización y Vectorización:

1. **Tokenización**: División del texto en palabras individuales para facilitar el análisis.
2. **Vectorización**: Conversión del texto en vectores de características usando TF-IDF o embeddings para alimentar el modelo de NLP.

### Análisis de Distribución de Clases:

- Revisión de si las clases están balanceadas para evitar sesgos en el modelo. Si existe un desbalance, se propone aplicar técnicas de sobremuestreo o submuestreo.

## Diccionario
| Campo                         | Descripción                                                                                  |
|-------------------------------|----------------------------------------------------------------------------------------------|
| DATE RECEIVED                 | La fecha en que el CFPB recibió la queja                                                     |
| PRODUCT                       | El tipo de producto identificado en la queja                                                 |
| SUB-PRODUCT                   | El tipo de subproducto identificado en la queja                                              |
| ISSUE                         | El problema identificado en la queja                                                         |
| SUB-ISSUE                     | El subproblema identificado en la queja                                                      |
| CONSUMER COMPLAINT NARRATIVE  | Explicación de la situación que le ocurre al cliente                                          |
| COMPANY PUBLIC RESPONSE       | La respuesta pública opcional de la empresa hacia el consumidor                              |
| COMPANY                       | La empresa sobre la cual se hace la queja                                                    |
| STATE                         | El estado de la dirección postal del consumidor                                              |
| ZIP CODE                      | El código postal proporcionado por el consumidor                                             |
| TAGS                          | Datos que facilitan la clasificación y búsqueda de quejas                                    |
| CONSUMER CONSENT PROVIDED     | Indica si el consumidor ha dado su consentimiento para publicar la queja                     |
| SUBMITTED VIA                 | Cómo fue presentada la queja                                                                 |
| DATE SENT TO COMPANY          | La fecha en que el CFPB envió la queja a la empresa                                          |
| COMPANY RESPONSE TO CONSUMER  | Cómo respondió la empresa                                                                    |
| TIMELY RESPONSE               | Indica si la empresa dio una respuesta a tiempo                                              |
| CONSUMER DISPUTED             | Indica si el consumidor disputó la respuesta de la empresa                                   |
| COMPLAINT ID                  | El número de identificación único de la queja                                                |

## Modelo de datos conceptual
```
+-----------------+          +-------------------+          +-----------------+
|  Raw Data       |          | Processed Data   |          |   Models        |
|  (raw_data)     |          |  (processed_data)|          |   (models)      |
+-----------------+          +-------------------+          +-----------------+
| - PRODUCT       |          | - id             |          | - model_id      |
| - SUB-PRODUCT   |          | - text_clean     |          | - model_name    |
| - COMPLAINT ID  | -------> | - sentiment      | -------> | - version       |
| - CONSUMER COMP.|          | - category       |          | - accuracy      |
+-----------------+          | - features       |          | - created_date  |
                             +-------------------+          +-----------------+

+-----------------+          +-------------------+          +-----------------+
| Inference Input |          | Inference Output |          | Monitoring      |
| (inference_in)  |          | (inference_out)  |          | (monitoring)    |
+-----------------+          +-------------------+          +-----------------+
| - inference_id  |          | - inference_id   |          | - monitor_id    |
| - text          | -------> | - prediction     |          | - model_id      |
| - timestamp     |          | - probability    |          | - drift         |
|                 |          | - timestamp      |          | - performance   |
+-----------------+          +-------------------+          +-----------------+
```


### Raw Data (raw_data)
- **Descripción**: Contiene los datos iniciales en su estado bruto, con campos básicos como `PRODUCT`, `COMPLAINT ID`, y `CONSUMER COMPLAINT NARRATIVE`, etc
  - Permite capturar la fuente y momento de cada registro, facilitando la trazabilidad.
  - Permite reiniciar el pipeline en caso de ser necesario.

### Processed Data (processed_data)
- **Descripción**: Almacena los datos después del preprocesamiento, incluyendo variables derivadas como `sentiment`, `category`, y `features` (vector de características).
  - Las columnas `sentiment` y `category` reflejan los resultados de las primeras inferencias de modelos de clasificación y análisis de sentimiento.
  - Permite reutilizar los datos procesados sin repetir el proceso de limpieza y enriquecimiento.

### Models (models)
- **Descripción**: Registra los modelos entrenados con campos como `model_id`, `model_name`, `version`, `accuracy`, y `created_date`.
  - Facilita el seguimiento de rendimiento y la comparación entre diferentes iteraciones del modelo gracias a la disponibilidad de versiones y métricas como `accuracy`.

### Inference Input (inference_in) e Inference Output (inference_out)
- **Inference Input (inference_in)**: Permite realizar inferencias en batch sin mezclar los datos de entrenamiento.
- **Inference Output (inference_out)**: Almacena las predicciones y la probabilidad asociada para cada entrada de inferencia.
 Facilita la auditoría y análisis de los resultados de inferencia en caso de problemas o ajustes de precisión.

### Monitoring (monitoring)
- **Descripción**: Captura métricas de rendimiento y deriva métricas de `drift` o desviación, para identificar cuándo el modelo comienza a perder precisión.
- **Beneficios**:
  - Permite almacenar históricos de rendimiento, ayudando a automatizar alertas de reentrenamiento.

## Ventajas 

### Escalabilidad y Flexibilidad
- La estructura facilita el almacenamiento de datos en varias etapas del pipeline, con la posibilidad de modificar o añadir nuevas variables derivadas sin interrumpir el flujo.

### Versionamiento y Trazabilidad
- Las entidades `models` y `monitoring` permiten realizar un seguimiento detallado de cada modelo y sus respectivas versiones, facilitando la actualización y manteniendo un historial completo de su rendimiento.

### Modularidad y Mantenimiento
- Las entidades separadas permiten que el pipeline sea modular, aislando cada fase y simplificando el mantenimiento y la actualización de cada componente por separado.