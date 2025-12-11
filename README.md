# OpenMeteo Data Pipeline & Delta Lake

Este proyecto implementa un pipeline ETL (Extract, Transform, Load) robusto y escalable diseñado para la ingesta, almacenamiento y procesamiento de datos meteorológicos de alta frecuencia.

El sistema orquesta la extracción incremental de datos desde la API de Open-Meteo, gestiona un Data Lake local basado en formato **Delta Lake** y genera vistas analíticas agregadas para reportes climáticos.

## 🏗 Arquitectura del Sistema

El flujo de datos sigue una arquitectura **Medallion (Bronze/Silver)** simplificada:

1.  **Ingesta (Extract):** Conexión a endpoints REST (Geocoding y Weather Forecast).
2.  **Raw Layer (Bronze):** Almacenamiento de datos crudos en formato Delta Lake.
    * Estrategia de **Carga Incremental** basada en control de estado ("Watermarking").
    * Particionamiento físico por `city_id` y `fecha` para optimizar lecturas.
3.  **Processing Layer (Silver/Gold):** Limpieza, enriquecimiento y agregación.
    * Joins dimensionales (Facts + Dimensions).
    * Cálculo de métricas diarias (Min, Max, Avg).
    * Generación de indicadores de negocio (`alertas`).

## 🚀 Stack Tecnológico

* **Lenguaje:** Python 3.10+
* **Procesamiento:** Pandas (Transformaciones en memoria).
* **Storage & ACID:** `deltalake` (Python binding para Delta Lake/Rust).
* **Ingesta:** `requests` con manejo de excepciones y retries implícitos.
* **Configuración:** `python-dotenv` para gestión segura de variables de entorno.

## ✨ Características Clave

### 🔄 Extracción Incremental Inteligente
A diferencia de cargas completas tradicionales, este pipeline consulta el estado actual del Data Lake antes de realizar peticiones a la API.
* **Lógica:** `API Request = f(Ultima_Fecha_Registrada, Fecha_Actual)`
* **Beneficio:** Minimiza el consumo de cuota de la API, reduce la latencia de red y evita la duplicidad de datos en origen.

### 🛡️ Calidad y Seguridad del Dato
* **Idempotencia:** El pipeline puede ejecutarse múltiples veces sin generar registros duplicados gracias a las restricciones de unicidad en la transformación.
* **Aislamiento:** Las credenciales y endpoints se gestionan fuera del código fuente mediante variables de entorno.

## ⚙️ Instalación y Configuración

1.  **Clonar el repositorio:**
    ```bash
    git clone <repo-url>
    cd tp-integrador-data-eng
    ```

2.  **Instalar dependencias:**
    Se recomienda utilizar un entorno virtual.
    ```bash
    pip install -r requirements.txt
    ```

3.  **Configuración de Entorno:**
    El proyecto requiere un archivo `.env` en la raíz con las siguientes claves:
    ```ini
    BASE_URL_GEOCODING="[https://geocoding-api.open-meteo.com/v1/search](https://geocoding-api.open-meteo.com/v1/search)"
    BASE_URL_WEATHER="[https://api.open-meteo.com/v1/forecast](https://api.open-meteo.com/v1/forecast)"
    ```

## ▶️ Ejecución

El pipeline está orquestado en un Jupyter Notebook para facilitar la visualización del flujo de datos paso a paso.

1.  Abrir el archivo `main_etl.ipynb`.
2.  Ejecutar las celdas secuencialmente.
3.  Verificar la creación del directorio `./datalake` con las particiones correspondientes.

## 📊 Estructura de Datos (Output)

Los datos procesados se almacenan en:
`datalake/processed/clima_diario`

Esquema resultante:
| Columna | Tipo | Descripción |
| :--- | :--- | :--- |
| `country` | string | País (Partition Key) |
| `name` | string | Nombre de la ciudad |
| `fecha_dia` | date | Fecha del registro |
| `temp_avg` | double | Temperatura promedio diaria |
| `precipitacion_total` | double | Acumulado de lluvias |
| `hubo_alerta` | boolean | Indicador de eventos extremos |

---
**Desarrollado por:** Ignacio J López