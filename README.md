# PTMLops
Prueba Técnica Ingeniero de Machine Learning Nequi.
# Propuesta de Arquitectura en la Nube en AWS

## Recursos
Entre las opciones de repositorios para elegir la base de datos se eligió:
- **Consumer Complaint Database** (Data.gov): Este dataset tiene registros de quejas de consumidores sobre productos financieros. Lo cual es ideal para construir un modelo para clasificar el tipo de queja, cumpliendo con el requisito de usar NLP, realizando una clasificación multiclase. La base de datos se actualiza constantemente y tiene varias columnas descriptivas, como el producto involucrado y el motivo de la queja. Actualmente dispone de 9817897 registros, lo cual también es ideal según lo solicitado en la prueba.

## Componentes Principales

- **S3 (Amazon Simple Storage Service)**: Almacén de datos en bruto y preprocesados.
- **Glue**: Servicio de ETL para la preparación de datos y transformaciones.
- **SageMaker**: Plataforma para el entrenamiento, ajuste y despliegue de modelos.
- **Lambda y Step Functions**: Coordinación y automatización del flujo de trabajo batch.
 
**Extra**:
- **CloudWatch**: Monitorización y registro de métricas y logs del sistema.
- **IAM (Identity and Access Management)**: Gestión de permisos y seguridad.

## Flujo de Trabajo

### Paso 1: Ingesta y Almacenamiento de Datos

- **S3 Buckets**: Los datos de entrada, como la base de datos, se almacenan en un bucket S3 en formato de texto o CSV (En este caso CSV). S3 es escalable y puede almacenar grandes volúmenes de datos de manera económica. Puede organizarse en carpetas para datos en bruto, preprocesados y resultados. S3 ofrece alta disponibilidad y replicación automática de los datos, lo que garantiza durabilidad y accesibilidad sin importar el volumen de datos.

### Paso 2: Preparación de Datos y Procesos ETL

- **AWS Glue**: Glue facilita el procesamiento y transformación de datos a través de tareas ETL (Extract, Transform, Load). Glue puede aplicar limpieza, preprocesamiento y tokenización del texto. Glue es un servicio serverless que puede escalar automáticamente en función de la carga de trabajo, procesando grandes volúmenes de datos en paralelo. Para este trabajo se propone trabajar con pyspark de python, ya que AWS Glue admite scripts de este lenguaje. La ventaja de spark es que permite trabajar con grandes volumenes de datos y brinda herramientas de paralelización para realizar un tareas de ETL.  También permite manejar errores con su sistema de logs integrado en CloudWatch.

### Paso 3: Entrenamiento e Implementación del Modelo NLP

- **Amazon SageMaker**: Se utiliza para entrenar y ajustar el modelo NLP de clasificación. SageMaker permite entrenar modelos usando técnicas de clasificación de texto o extracción de información a partir de datos de texto y ofrece herramientas para ajustar hiperparámetros de forma automática. SageMaker escala automáticamente los recursos de cómputo durante el entrenamiento y despliegue. Además, permite la integración con IAM para el control de acceso seguro a los modelos y datos en S3. Para este trabajo se propone realizar el diseño del modelo en un script de python. Esto con el fin de poder tener un control mas amplio en el diseño del modelo de ML y luego simplemente cargar en el script para realizar el entrenamiento aprovechando el servicio de AWS.

### Paso 4: Automatización y Orquestación

- **AWS Lambda y Step Functions**: Para ejecutar el flujo de trabajo en batch, Lambda inicia el proceso al detectar nuevos archivos en el bucket S3. Step Functions coordina los pasos ETL en Glue, el entrenamiento y la inferencia en SageMaker (En este caso sería solo la inferencia con el modelo entrenado cargado en otro bucket en S3), y el almacenamiento de resultados.
Lambda y Step Functions son serverless y permiten ejecutar y escalar flujos de trabajo sin necesidad de administrar servidores. La arquitectura en Step Functions permite manejar errores y reintentos, asegurando confiabilidad.

### Paso 5: Almacenamiento de Resultados y Visualización

- **Resultados en S3**: Los resultados de la inferencia se almacenan en un bucket de S3 específico para el análisis posterior o su integración con otras aplicaciones de análisis de datos.
- **Extra - AWS QuickSight**: Para visualizar los resultados de clasificación o extracción de datos, QuickSight puede integrarse y ofrecer dashboards dinámicos y personalizables.

## Escalabilidad y Confiabilidad de la Arquitectura

- **Escalabilidad**: Todos los componentes de AWS son escalables y se ajustan automáticamente a la carga. Glue, SageMaker y S3 son serverless, lo que permite procesar grandes volúmenes de datos sin preocupaciones de capacidad.
- **Confiabilidad**: S3 asegura la durabilidad de los datos. AWS Glue y SageMaker están respaldados por la infraestructura de AWS, que garantiza alta disponibilidad y recuperación de fallos. Step Functions permite manejar errores en cada paso del flujo y reintentar tareas fallidas, manteniendo la confiabilidad del proceso.

## Resumen de la elección de Componentes

- **S3**: Almacenamiento económico y escalable para datos sin procesar, preprocesados y resultados.
- **Glue**: Servicio ETL serverless para limpieza y preprocesamiento de texto, adecuado para transformar datos en forma escalable.
- **SageMaker**: Permite entrenar, ajustar y desplegar modelos en batch con capacidad de escalar el procesamiento para grandes volúmenes.
- **Lambda y Step Functions**: Automatización del flujo de trabajo y orquestación de procesos de manera serverless, asegurando un flujo robusto y fácil de mantener.
- **CloudWatch**: Proporciona monitoreo, alertas y logs centralizados para supervisar la ejecución del flujo de trabajo y detectar errores rápidamente.


# Step-by-Step

## Orquestación con AWS Step Functions

AWS Step Functions es el componente clave de orquestación en esta arquitectura. A continuación se describe como se contribuye a la ejecución del pipeline de NLP:

### Coordinación de Pasos
Step Functions permite coordinar cada etapa, asegurando que los procesos ETL, el entrenamiento y la inferencia se ejecuten en el orden correcto.

### Automatización
Simplifica la automatización al desencadenar el flujo cuando los datos nuevos son detectados en S3. A medida que el proceso avanza, cada paso se ejecuta de manera automatizada, eliminando la necesidad de intervención manual.

### Manejo de Errores y Reintentos
En caso de que alguna etapa falle (por ejemplo, problemas en SageMaker o Glue), Step Functions puede reintentar los pasos fallidos o redirigir a flujos alternativos, asegurando la confiabilidad del pipeline.

### Escalabilidad
Permite que el pipeline escale según el volumen de datos procesados. Step Functions es serverless y funciona en paralelo, permitiendo procesar varios trabajos sin cuellos de botella.

### Entrenamiento Continuo
Step Functions permite iniciar trabajos de reentrenamiento cuando Model Monitor detecta degradación en el rendimiento, asegurando que el modelo mantenga su precisión con datos nuevos.

# Estructura de directorios
```
project-root/
│
├── data/                    # Almacenamiento de datos local (opcional si se usa solo S3)
│   ├── raw/                 # Datos sin procesar
│   └── processed/           # Datos procesados para el modelo
│
├── scripts/                 # Scripts para ejecución de ETL, entrenamiento e inferencia
│   ├── data_preprocessing/  # Scripts de preprocesamiento de datos
│   ├── training/            # Scripts de entrenamiento del modelo
│   └── inference/           # Scripts de inferencia y predicción
│
├── models/                  # Directorio de modelos entrenados
│   ├── versioned/           # Modelos con control de versiones
│   │   ├── v1/              # Modelo en su versión 1
│   │   ├── v2/              # Modelo en su versión 2, etc.
│   └── latest/              # Última versión del modelo en producción
│
├── config/                  # Configuración del proyecto
│   ├── pipeline_config.yaml # Configuraciones del pipeline ETL
│   └── model_config.yaml    # Configuración de hiperparámetros del modelo
│
├── tests/                   # Pruebas unitarias y de integración
│   ├── test_data_processing/ # Pruebas para el pipeline de procesamiento
│   ├── test_training/        # Pruebas para el entrenamiento
│   └── test_inference/       # Pruebas para inferencia
│
├── utils/                   # Funciones auxiliares compartidas
│   └── logger.py            # Logger para la gestión de logs
│
├── logs/                    # Archivos de log para seguimiento del proceso
│   ├── data_processing.log
│   ├── training.log
│   └── inference.log
│
└── README.md                # Descripción general del proyecto y cómo ejecutar el pipeline
```


## Explicación

### Directorio `data/`
Almacena temporalmente los datos para procesamiento local, aunque la mayor parte se gestiona en S3.

### Directorio `scripts/`
Contiene subdirectorios específicos para cada etapa:

- **`data_preprocessing/`**: Scripts de ETL y transformación.
- **`training/`**: Scripts de entrenamiento del modelo.
- **`inference/`**: Scripts para hacer predicciones en batch con el modelo.

### Directorio `models/`
Maneja la versión de los modelos:

- **`versioned/`**: Subdirectorios para cada versión (v1, v2, etc.), lo cual facilita la recuperación de versiones anteriores.
- **`latest/`**: Última versión en producción, utilizada por los scripts de inferencia.

### Directorio `config/`
Contiene archivos `.yaml` de configuración, lo que permite una administración centralizada de los parámetros del pipeline y el modelo.

### Directorio `tests/`
Incluye pruebas para asegurar que cada paso del pipeline (procesamiento de datos, entrenamiento, inferencia) funciona correctamente antes de implementarse en producción.

### Directorio `utils/`
Funciones auxiliares, como un logger, para reutilizar en distintas partes del pipeline.

### Directorio `logs/`
Archivos de log para auditar y depurar, manteniendo un seguimiento del rendimiento y errores en cada etapa del pipeline.
