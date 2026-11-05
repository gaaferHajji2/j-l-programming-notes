Based on your interest in modern frameworks and automation tools, here are the best open-source ETL (Extract, Transform, Load) tools:

## **Top Open-Source ETL Tools**

### 1. **Apache Airflow**
- **Best for**: Workflow orchestration and scheduling
- **Language**: Python
- **Strengths**: Highly extensible, large community, excellent for complex data pipelines
- **Use case**: When you need to schedule and monitor ETL workflows

### 2. **Apache NiFi**
- **Best for**: Real-time data flow management
- **Strengths**: Visual interface, handles high-volume data streams, strong security features
- **Use case**: Real-time data ingestion and routing between systems

### 3. **dbt (data build tool)**
- **Best for**: SQL-based transformations
- **Language**: SQL + Jinja templating
- **Strengths**: Version control friendly, testing built-in, great documentation
- **Use case**: Transforming data already loaded into a warehouse

### 4. **Apache Spark**
- **Best for**: Large-scale data processing
- **Languages**: Scala, Python, Java, R
- **Strengths**: Distributed computing, handles massive datasets, versatile
- **Use case**: Big data processing and complex transformations

### 5. **Meltano (formerly Singer.io)**
- **Best for**: Modular, composable ETL pipelines
- **Language**: Python
- **Strengths**: Taps (extractors) and targets (loaders) are interchangeable, cloud-native
- **Use case**: Building custom data pipelines with reusable components

### 6. **Kestra**
- **Best for**: Modern orchestration with YAML/Python
- **Strengths**: Declarative pipelines, event-driven, growing rapidly
- **Use case**: Alternative to Airflow with simpler configuration

### 7. **Pentaho Data Integration (Kettle)**
- **Best for**: Visual ETL design
- **Strengths**: GUI-based, mature platform, enterprise features
- **Use case**: Teams preferring visual pipeline design

### 8. **Talend Open Studio**
- **Best for**: Comprehensive data integration
- **Strengths**: Rich component library, good for diverse data sources
- **Use case**: Complex integration scenarios with many connectors

## **Recommendation Based on Your Stack**

Given your interests:
- **For Python-heavy workflows**: Apache Airflow + dbt
- **For real-time scraping/data collection**: Apache NiFi or Meltano
- **For large-scale processing**: Apache Spark
- **For modern DevOps-friendly approach**: Kestra