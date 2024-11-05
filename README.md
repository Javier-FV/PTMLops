# PTMLops
Prueba Técnica Ingeniero de Machine Learning Nequi.
# Propuesta de Arquitectura en la Nube en AWS
![Propuesta de Arquitectura en la Nube en AWS para Procesamiento NLP ]

## Componentes Principales

- **S3 (Amazon Simple Storage Service)**: Almacén de datos en bruto y preprocesados.
- **Glue**: Servicio de ETL para la preparación de datos y transformaciones.
- **SageMaker**: Plataforma para el entrenamiento, ajuste y despliegue de modelos.
- **Lambda y Step Functions**: Coordinación y automatización del flujo de trabajo batch.
- **CloudWatch**: Monitorización y registro de métricas y logs del sistema.
- **IAM (Identity and Access Management)**: Gestión de permisos y seguridad.

## Flujo de Trabajo

### Paso 1: Ingesta y Almacenamiento de Datos

- **S3 Buckets**: Los datos de entrada, como las quejas de clientes, se almacenan en un bucket S3 en formato de texto o CSV. S3 es escalable y puede almacenar grandes volúmenes de datos de manera económica. Puede organizarse en carpetas para datos en bruto, preprocesados y resultados.
- **Escalabilidad y Confiabilidad**: S3 ofrece alta disponibilidad y replicación automática de los datos, lo que garantiza durabilidad y accesibilidad sin importar el volumen de datos.

### Paso 2: Preparación de Datos y Procesos ETL

- **AWS Glue**: Glue facilita el procesamiento y transformación de datos a través de tareas ETL (Extract, Transform, Load). Glue puede aplicar limpieza, preprocesamiento y tokenización del texto.
- **Escalabilidad y Confiabilidad**: Glue es un servicio serverless que puede escalar automáticamente en función de la carga de trabajo, procesando grandes volúmenes de datos en paralelo. También permite manejar errores con su sistema de logs integrado en CloudWatch.

### Paso 3: Entrenamiento e Implementación del Modelo NLP

- **Amazon SageMaker**: Se utiliza para entrenar y ajustar el modelo NLP de clasificación. SageMaker permite entrenar modelos usando técnicas de clasificación de texto o extracción de información a partir de datos de texto y ofrece herramientas para ajustar hiperparámetros de forma automática.
- **Batch Transform en SageMaker**: SageMaker permite realizar inferencia batch mediante Batch Transform, adecuado para procesamiento masivo sin necesidad de desplegar un endpoint en tiempo real.
- **Escalabilidad y Confiabilidad**: SageMaker escala automáticamente los recursos de cómputo durante el entrenamiento y despliegue. Además, permite la integración con IAM para el control de acceso seguro a los modelos y datos en S3.

### Paso 4: Automatización y Orquestación

- **AWS Lambda y Step Functions**: Para ejecutar el flujo de trabajo en batch, Lambda inicia el proceso al detectar nuevos archivos en el bucket S3. Step Functions coordina los pasos ETL en Glue, el entrenamiento y la inferencia en SageMaker, y el almacenamiento de resultados.
- **Escalabilidad y Confiabilidad**: Lambda y Step Functions son serverless y permiten ejecutar y escalar flujos de trabajo sin necesidad de administrar servidores. La arquitectura en Step Functions permite manejar errores y reintentos, asegurando confiabilidad.

### Paso 5: Almacenamiento de Resultados y Visualización

- **Resultados en S3**: Los resultados de la inferencia se almacenan en un bucket de S3 específico para el análisis posterior o su integración con otras aplicaciones de análisis de datos.
- **Opcional - AWS QuickSight**: Para visualizar los resultados de clasificación o extracción de datos, QuickSight puede integrarse y ofrecer dashboards dinámicos y personalizables.

## Explicación de la Escalabilidad y Confiabilidad de la Arquitectura

- **Escalabilidad**: Todos los componentes de AWS son escalables y se ajustan automáticamente a la carga. Glue, SageMaker y S3 son serverless, lo que permite procesar grandes volúmenes de datos sin preocupaciones de capacidad.
- **Confiabilidad**: S3 asegura la durabilidad de los datos. AWS Glue y SageMaker están respaldados por la infraestructura de AWS, que garantiza alta disponibilidad y recuperación de fallos. Step Functions permite manejar errores en cada paso del flujo y reintentar tareas fallidas, manteniendo la confiabilidad del proceso.

## Elección de Componentes

- **S3**: Almacenamiento económico y escalable para datos sin procesar, preprocesados y resultados.
- **Glue**: Servicio ETL serverless para limpieza y preprocesamiento de texto, adecuado para transformar datos en forma escalable.
- **SageMaker**: Permite entrenar, ajustar y desplegar modelos en batch con capacidad de escalar el procesamiento para grandes volúmenes.
- **Lambda y Step Functions**: Automatización del flujo de trabajo y orquestación de procesos de manera serverless, asegurando un flujo robusto y fácil de mantener.
- **CloudWatch**: Proporciona monitoreo, alertas y logs centralizados para supervisar la ejecución del flujo de trabajo y detectar errores rápidamente.

