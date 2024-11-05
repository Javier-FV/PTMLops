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