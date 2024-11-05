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