# 🚀 Proyecto Data Lakehouse: Análisis de Datos de E-commerce

## 📄 Descripción General

Este proyecto implementa una robusta arquitectura **Data Lakehouse** en Azure, diseñada para procesar, transformar y analizar datos de un e-commerce provenientes de **múltiples fuentes y con diversos formatos**. La solución utiliza una **Landing Zone** como punto de entrada inicial para los datos crudos, **Azure Data Lake Storage Gen2 (ADLS Gen2)** como almacenamiento central para las capas Medallion, **Azure Event Hubs** para la ingesta de datos en tiempo real, **Azure Data Factory (ADF)** para la orquestación e ingesta, **Azure Databricks** para transformaciones de datos escalables (ETL) y **Power BI** para la visualización y análisis de negocio. El objetivo es proporcionar insights clave sobre ventas, productos, clientes y uso de aplicaciones, democratizando el acceso a datos limpios y estructurados.

---

## ✨ Características Clave

* **Enfoque de Datos Multiorigen:** El proyecto maneja y orquesta la ingesta de datos provenientes de tres tipos principales de fuentes:
    1.  **Streaming (Reseñas de Productos):** Datos de `product_reviews` enviados en tiempo real a Azure Event Hubs.
    2.  **Simulación de API (Eventos de Aplicación):** Datos de `app_events` extraídos de una fuente externa (simulando una API vía GitHub).
    3.  **Batch Tradicional (Clientes, Productos, Pedidos):** Datos estructurados de `customers`, `products` y `orders` en formato CSV.

* **Zonas de Almacenamiento:**
    * **Landing Zone:** Área de almacenamiento temporal inicial en ADLS Gen2 para todos los datos brutos al llegar, antes de cualquier procesamiento o movimiento a las capas de medallón.
    * **Capas de Datos (Bronce, Plata, Oro - Arquitectura Medallion):**
        * **Bronce (Raw):** Almacenamiento inmutable de datos crudos tal cual se reciben de la Landing Zone. Se mantienen formatos originales como **Avro** (para streaming) y **Parquet** (para eventos de aplicación), junto con **CSV** para datos batch.
        * **Plata (Cleaned/Refined):** Datos limpios, desduplicados y transformados con esquema aplicado. Se consolidan en formato **Delta Lake**.
        * **Oro (Curated/Aggregated):** Datos agregados y modelados en formato dimensional (`fact_sales`, `dim_products`, `dim_customers`, etc.) optimizados para análisis de negocio. Se mantienen en formato **Delta Lake**.

* **Tecnología de Almacenamiento:** Todas las capas residen en **Azure Data Lake Storage Gen2 (ADLS Gen2)**, aprovechando su capacidad de escalabilidad y el espacio de nombres jerárquico. El formato **Delta Lake** se utiliza para las capas Plata y Oro, garantizando transacciones ACID, esquema evolutivo y soporte para operaciones `MERGE`.

* **Ingesta y Orquestación de Datos (ELT/ETL):**
    * **Azure Event Hubs con Capture:** Implementación de un pipeline de ingesta de datos en tiempo real para **reseñas de productos (`product_reviews`)**. Los datos son enviados a Azure Event Hubs mediante un script personalizado y luego, utilizando la funcionalidad **Capture**, son automáticamente volcados y persistidos en la **Landing Zone** (ADLS Gen2) en formato **Avro**, listos para su procesamiento posterior.
    * **Azure Data Factory (ADF):** Utilizado para orquestar y automatizar los pipelines de datos.
        * **Pipeline de `app_events`:** Un **Data Flow** específico dentro de ADF se encarga de:
            * Extraer datos de `app_events` simulando una API a través de un archivo en **GitHub** (usando un Linked Service HTTP).
            * Unir estos datos (JSON) con un histórico existente en la **Landing Zone**.
            * Transformar y consolidar la información.
            * Finalmente, guardar los datos resultantes en formato **Parquet** en la capa **Bronce**.
        * **Ingesta de Datos Batch:** Pipelines de copia de ADF trasladan datos estructurados (CSV) de `customers`, `products` y `orders` desde la Landing Zone a la capa **Bronce**.
    * **Azure Databricks:** Motores de cómputo Spark para transformaciones complejas entre las capas Bronce, Plata y Oro, generando las tablas Delta finales.

* **Autenticación Segura:** Configuración de acceso a ADLS Gen2 mediante **Azure Key Vault** y **Service Principals (OAuth2)** para una gestión segura de credenciales en Databricks.

* **Visualización y Análisis:**
    * **Power BI:** Conexión directa a las tablas Delta de la capa Oro en ADLS Gen2 para la creación de dashboards interactivos. Esta conexión directa aprovecha la capacidad nativa de Power BI para leer Delta Lake, optimizando la latencia y los costos de cómputo para las consultas de BI.

---

## 🛠️ Tecnologías Utilizadas

* **Azure Data Lake Storage Gen2 (ADLS Gen2):** Almacenamiento central de todas las capas y Landing Zone.
* **Azure Event Hubs:** Ingesta de datos en streaming (ej. `product_reviews`).
* **Event Hubs Capture:** Persistencia automática de streams de Event Hubs a ADLS Gen2.
* **Azure Data Factory (ADF):** Orquestación de pipelines, ingesta de datos (Batch, Simulación API) y transformaciones con Data Flows.
* **Delta Lake:** Formato de tabla para las capas Plata y Oro.
* **Apache Avro:** Formato de datos para `product_reviews` en la capa Bronce (originado en Landing).
* **Apache Parquet:** Formato de datos para `app_events` en la capa Bronce (originado en Landing).
* **CSV:** Formato de datos para Batch en la capa Bronce (originado en Landing).
* **Azure Databricks:** Plataforma de análisis y ciencia de datos para transformaciones ETL de Spark.
* **Power BI:** Herramienta de inteligencia de negocio.
* **Azure Key Vault:** Gestión segura de secretos.
* **Spark SQL / PySpark:** Para lógica de transformación en Databricks.
* **GitHub (como fuente de datos externa):** Para el archivo de `app_events` (simulación de API).

---

## 📂 Estructura del Proyecto (Ejemplo)

├── adf_pipelines/                     # Archivos de configuración/templates de Azure Data Factory
│   ├── landing_to_bronze_batch_copy.json # Para CSVs de clientes, productos, pedidos y Avro de reviews
│   └── app_events_data_flow_pipeline.json # Pipeline para unir y convertir app_events a Parquet en Bronce
├── notebooks/
│   ├── 01_bronze_to_silver.py        # Limpia y refina datos (Avro, Parquet, CSV) a Delta Plata
│   └── 02_silver_to_gold_modeling.py # Agrega y modela datos a la capa Oro (Tablas Delta)
├── powerbi_reports/
│   └── Sales_Analytics_Dashboard.pbix # Archivo del dashboard de Power BI
├── landing_zone_data/                 # Zona de aterrizaje inicial en ADLS Gen2
│   ├── historical_app_events.json     # Histórico para Data Flow de app_events
│   ├── product_reviews/               # Datos de Event H
│   ├── customers.csv
│   ├── products.csv
│   └── orders.csv
├── event_hub_scripts/                 # (Opcional) Scripts para enviar datos a Event Hubs
│   └── send_review_data.py
└── README.md                          # Este archivo


## 🚀 Cómo Ejecutar el Proyecto

### Requisitos Previos

* Suscripción de Azure activa.
* Recursos de Azure aprovisionados: Azure Data Lake Storage Gen2, Azure Event Hubs, Azure Data Factory, Azure Databricks Workspace, Azure Key Vault.
* Un Service Principal de Azure AD con los permisos necesarios para ADF (Contribuir a datos de blobs de almacenamiento) y para Databricks (Lector/Colaborador de datos de blobs de almacenamiento).
* Secretos del Service Principal (Client ID, Client Secret, Tenant ID) almacenados en Azure Key Vault.
* Power BI Desktop instalado.

### Pasos

1.  **Configurar Azure Key Vault y Secretos:**
    * Almacenar `client-id`, `client-secret` y `tenant-id` de tu Service Principal en Azure Key Vault.
    * Crear un Secret Scope en Databricks para acceder a estos secretos (ej. `tom-keyvault`).

2.  **Configurar Ingesta de Datos Multiorigen a Landing Zone:**
    * **Event Hubs:**
        * Crear un Event Hub (ej. para `product_reviews`).
        * Habilitar la función **Capture** para volcar automáticamente los mensajes al contenedor **Landing** de tu ADLS Gen2 en formato **Avro**.
        * (Opcional) Usar un script (ej. `send_review_data.py`) para enviar datos de ejemplo a tu Event Hub.
    * **Azure Data Factory:**
        * Crear Linked Services para ADLS Gen2, GitHub y Event Hubs (si es necesario para monitoreo o triggers).
        * **Implementar el pipeline de `app_events` con Data Flow:** Configurar el Data Flow para leer desde GitHub, unir con el histórico en **Landing**, y escribir en formato **Parquet** a la capa **Bronce** en ADLS Gen2.
        * Implementar pipelines de copia para trasladar los archivos batch (CSV) desde la **Landing Zone** a la capa **Bronce**.

3.  **Configurar Cluster de Databricks:**
    * Crear un cluster en Azure Databricks.
    * **Importante:** Añadir las configuraciones de Spark para el acceso OAuth2 a ADLS Gen2 directamente en la **"Spark Config"** de tu cluster, usando las referencias a tus secretos de Key Vault (o los valores directos para pruebas).

4.  **Ejecutar Notebooks de Databricks:**
    * Importar los notebooks (`01_bronze_to_silver.py`, `02_silver_to_gold_modeling.py`) a tu espacio de trabajo Databricks.
    * Adjuntar los notebooks al cluster configurado.
    * Ejecutar los notebooks secuencialmente para transformar datos de **Bronce** (Avro, Parquet, CSV) a **Plata** (Delta) y de **Plata** a **Oro** (Delta), creando las tablas Delta finales en tu contenedor `gold` (ej. `abfss://gold@tomdatalakehouse.dfs.core.windows.net/fact_sales/`).

5.  **Conectar Power BI:**
    * Abrir Power BI Desktop.
    * "Obtener datos" -> "Azure Data Lake Storage Gen2".
    * Para cada tabla Gold, introducir la URL directa de su carpeta Delta (ej. `https://tomdatalakehouse.dfs.core.windows.net/gold/fact_sales/`).
    * Autenticar usando la **Clave de Cuenta** de tu Storage Account.
    * En el Editor de Power Query:
        * **Filtrar la columna "Extension" por ".parquet"**.
        * **Combinar el contenido** de la columna "Content" (usando el icono de flechas opuestas).
        * Renombrar la consulta a tu nombre de tabla (ej. `fact_sales`).
    * "Cerrar y aplicar" para cargar las tablas en el modelo de Power BI.

6.  **Construir Reportes en Power BI:**
    * Una vez cargadas las tablas, puedes establecer las relaciones entre ellas y empezar a crear visualizaciones interactivas.

---
