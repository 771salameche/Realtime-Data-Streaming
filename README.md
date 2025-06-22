# Real-Time Data Streaming Pipeline

A comprehensive real-time data streaming pipeline built with Apache Airflow, Kafka, Spark, and Cassandra, orchestrated using Docker containers. This project demonstrates end-to-end data processing from ingestion to storage and monitoring.

## 🏗️ Architecture Overview

![System Architecture]()

The pipeline consists of the following components:

- **Data Ingestion**: Apache Airflow orchestrates data extraction from various APIs
- **Message Streaming**: Apache Kafka handles real-time data streaming with ZooKeeper coordination
- **Data Processing**: Apache Spark cluster processes streaming data in real-time
- **Data Storage**: Apache Cassandra stores processed data for scalable access
- **Monitoring**: Control Center and Schema Registry provide monitoring and schema management
- **Containerization**: Docker ensures consistent deployment across environments

## 🚀 Features

- **Real-time Processing**: Sub-second latency data processing
- **Scalable Architecture**: Horizontally scalable Spark cluster
- **Fault Tolerance**: Built-in redundancy and error handling
- **Schema Evolution**: Confluent Schema Registry for schema management
- **Monitoring**: Comprehensive monitoring and alerting
- **Containerized Deployment**: Easy deployment with Docker Compose

## 🛠️ Technology Stack

### Core Components

| Component | Version | Purpose |
|-----------|---------|---------|
| **Apache Airflow** | 2.7+ | Workflow orchestration and scheduling |
| **Apache Kafka** | 3.5+ | Distributed streaming platform |
| **Apache Spark** | 3.4+ | Large-scale data processing engine |
| **Apache Cassandra** | 4.1+ | NoSQL distributed database |
| **Apache ZooKeeper** | 3.8+ | Coordination service for Kafka |
| **Docker** | 24.0+ | Containerization platform |

### Supporting Tools

- **Confluent Control Center**: Kafka cluster monitoring
- **Confluent Schema Registry**: Schema management and evolution
- **PostgreSQL**: Airflow metadata database

## 📋 Prerequisites

Before running this project, ensure you have:

- Docker (24.0+) and Docker Compose (2.20+)
- Minimum 8GB RAM available for containers
- Python 3.8+ (for development)
- Git for version control

### System Requirements

```bash
# Minimum hardware requirements
CPU: 4 cores
RAM: 8GB
Storage: 20GB free space
Network: Stable internet connection
```

## 🔧 Installation

### 1. Clone the Repository

```bash
git clone https://github.com/771salameche/Realtime-Data-Streaming
cd Realtime-Data-Streaming
```

### 2. Environment Configuration

```bash
# Copy environment template
cp .env.example .env

# Edit configuration variables
nano .env
```

### 3. Start the Pipeline

```bash
# Start all services
docker-compose up -d

# Verify all containers are running
docker-compose ps
```

### 4. Initialize Services

```bash
# Initialize Airflow database
docker-compose exec airflow-webserver airflow db init

# Create Airflow admin user
docker-compose exec airflow-webserver airflow users create \
    --username admin \
    --firstname Admin \
    --lastname User \
    --role Admin \
    --email admin@example.com \
    --password admin123

# Initialize Cassandra keyspace
docker-compose exec cassandra cqlsh -f /scripts/init.cql
```

## 🎯 Usage

### Access Web Interfaces

| Service | URL | Credentials |
|---------|-----|-------------|
| Airflow | http://localhost:8080 | admin/admin123 |
| Control Center | http://localhost:9021 | - |
| Spark Master | http://localhost:8081 | - |

### Running the Pipeline

1. **Start Data Ingestion**:
   ```bash
   # Access Airflow UI and enable the main DAG
   # Navigate to http://localhost:8080
   # Toggle the 'data_pipeline' DAG to ON
   ```

2. **Monitor Data Flow**:
   ```bash
   # Check Kafka topics
   docker-compose exec kafka kafka-topics --bootstrap-server localhost:9092 --list
   
   # Monitor Spark jobs
   # Navigate to http://localhost:8081
   ```

3. **Query Processed Data**:
   ```bash
   # Connect to Cassandra
   docker-compose exec cassandra cqlsh
   
   # Query data
   USE streaming_data;
   SELECT * FROM processed_events LIMIT 10;
   ```

### Sample Data Flow

```python
# Example: Sending data through the pipeline
import json
from kafka import KafkaProducer

producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda x: json.dumps(x).encode('utf-8')
)

# Send sample event
event = {
    'timestamp': '2024-01-15T10:30:00Z',
    'user_id': 'user123',
    'event_type': 'click',
    'data': {'page': 'home', 'session_id': 'abc123'}
}

producer.send('raw_events', event)
producer.flush()
```

## 📊 Pipeline Configuration

### Airflow DAGs

The main DAG (`dags/data_pipeline.py`) orchestrates:

- API data extraction
- Data validation and cleaning
- Kafka message publishing
- Pipeline monitoring

### Kafka Topics

```bash
# Default topics created
raw_events          # Incoming raw data
processed_events    # Spark-processed data
error_events        # Failed processing events
monitoring_events   # System monitoring data
```

### Spark Jobs

Located in `spark_jobs/` directory:

- `stream_processor.py`: Main streaming job
- `batch_processor.py`: Batch processing job
- `data_enrichment.py`: Data enrichment logic

### Cassandra Schema

```sql
-- Main tables
CREATE KEYSPACE IF NOT EXISTS streaming_data 
WITH replication = {'class': 'SimpleStrategy', 'replication_factor': 1};

CREATE TABLE streaming_data.processed_events (
    id UUID PRIMARY KEY,
    timestamp TIMESTAMP,
    user_id TEXT,
    event_type TEXT,
    processed_data TEXT,
    created_at TIMESTAMP
);
```

## 🔍 Monitoring and Troubleshooting

### Health Checks

```bash
# Check all services status
./scripts/health_check.sh

# Individual service checks
curl -f http://localhost:8080/health          # Airflow
curl -f http://localhost:9021/health          # Control Center
docker-compose exec kafka kafka-broker-api-versions --bootstrap-server localhost:9092
```

### Common Issues

#### 1. Services Not Starting

```bash
# Check Docker resources
docker system df
docker system prune -f  # Clean up if needed

# Check logs
docker-compose logs [service_name]
```

#### 2. Kafka Connection Issues

```bash
# Verify Kafka is accessible
docker-compose exec kafka kafka-topics --bootstrap-server localhost:9092 --list

# Reset Kafka if needed
docker-compose down
docker volume rm $(docker volume ls -q | grep kafka)
docker-compose up -d
```

#### 3. Spark Job Failures

```bash
# Check Spark logs
docker-compose logs spark-master
docker-compose logs spark-worker-1

# Restart Spark cluster
docker-compose restart spark-master spark-worker-1 spark-worker-2
```

#### 4. Cassandra Connection Issues

```bash
# Check Cassandra status
docker-compose exec cassandra nodetool status

# Restart Cassandra
docker-compose restart cassandra
```

### Performance Tuning

#### Kafka Optimization

```bash
# In docker-compose.yml, adjust Kafka settings:
KAFKA_NUM_PARTITIONS=6
KAFKA_DEFAULT_REPLICATION_FACTOR=2
KAFKA_LOG_RETENTION_HOURS=168
```

#### Spark Optimization

```python
# In spark jobs, tune these parameters:
spark.conf.set("spark.sql.adaptive.enabled", "true")
spark.conf.set("spark.sql.adaptive.coalescePartitions.enabled", "true")
spark.conf.set("spark.serializer", "org.apache.spark.serializer.KryoSerializer")
```

## 🤝 Contributing

We welcome contributions! Please follow these guidelines:

### Development Setup

```bash
# Create virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install development dependencies
pip install -r requirements-dev.txt

# Install pre-commit hooks
pre-commit install
```

### Code Standards

- Follow PEP 8 for Python code
- Use type hints where applicable
- Write comprehensive tests
- Document all functions and classes
- Use meaningful commit messages

### Pull Request Process

1. Fork the repository
2. Create a feature branch: `git checkout -b feature/your-feature`
3. Make your changes and add tests
4. Run the test suite: `pytest tests/`
5. Run linting: `flake8 . && black . && isort .`
6. Commit your changes: `git commit -m "Add your feature"`
7. Push to your fork: `git push origin feature/your-feature`
8. Create a Pull Request

### Testing

```bash
# Run all tests
pytest tests/

# Run specific test categories
pytest tests/unit/          # Unit tests
pytest tests/integration/   # Integration tests
pytest tests/e2e/          # End-to-end tests

# Run with coverage
pytest --cov=src tests/
```

## 📁 Project Structure

```
realtime-streaming-pipeline/
├── README.md
├── docker-compose.yml
├── .env.example
├── .gitignore
├── Makefile
├── requirements.txt
├── pyproject.toml
│
├── config/
│   ├── airflow/
│   │   ├── airflow.cfg
│   │   └── webserver_config.py
│   ├── kafka/
│   │   ├── server.properties
│   │   └── consumer.properties
│   ├── spark/
│   │   ├── spark-defaults.conf
│   │   └── log4j.properties
│   └── cassandra/
│       ├── cassandra.yaml
│       └── init-scripts/
│           └── keyspace_setup.cql
│
├── docker/
│   ├── airflow/
│   │   ├── Dockerfile
│   │   └── entrypoint.sh
│   ├── kafka/
│   │   └── Dockerfile
│   ├── spark/
│   │   ├── Dockerfile
│   │   └── spark-worker.sh
│   └── app/
│       └── Dockerfile
│
├── src/
│   ├── __init__.py
│   ├── common/
│   │   ├── __init__.py
│   │   ├── config.py
│   │   ├── logger.py
│   │   └── utils.py
│   ├── data_ingestion/
│   │   ├── __init__.py
│   │   ├── kafka_producer.py
│   │   ├── data_generator.py
│   │   └── api_connectors/
│   │       ├── __init__.py
│   │       ├── rest_client.py
│   │       └── streaming_client.py
│   ├── data_processing/
│   │   ├── __init__.py
│   │   ├── spark_streaming.py
│   │   ├── transformations/
│   │   │   ├── __init__.py
│   │   │   ├── data_cleaner.py
│   │   │   ├── aggregators.py
│   │   │   └── enrichers.py
│   │   └── schemas/
│   │       ├── __init__.py
│   │       └── data_schemas.py
│   └── data_storage/
│       ├── __init__.py
│       ├── cassandra_client.py
│       └── models/
│           ├── __init__.py
│           └── cassandra_models.py
│
├── dags/
│   ├── __init__.py
│   ├── data_pipeline_dag.py
│   ├── data_quality_dag.py
│   └── maintenance_dag.py
│
├── scripts/
│   ├── setup.sh
│   ├── start_services.sh
│   ├── stop_services.sh
│   ├── create_topics.sh
│   └── test_pipeline.sh
│
├── tests/
│   ├── __init__.py
│   ├── conftest.py
│   ├── unit/
│   │   ├── __init__.py
│   │   ├── test_data_ingestion.py
│   │   ├── test_data_processing.py
│   │   └── test_data_storage.py
│   ├── integration/
│   │   ├── __init__.py
│   │   ├── test_kafka_integration.py
│   │   ├── test_spark_integration.py
│   │   └── test_cassandra_integration.py
│   └── e2e/
│       ├── __init__.py
│       └── test_full_pipeline.py
│
├── monitoring/
│   ├── prometheus/
│   │   ├── prometheus.yml
│   │   └── rules/
│   ├── grafana/
│   │   ├── dashboards/
│   │   │   ├── kafka_dashboard.json
│   │   │   ├── spark_dashboard.json
│   │   │   └── pipeline_overview.json
│   │   └── provisioning/
│   └── alerts/
│       └── alert_rules.yml
│
├── docs/
│   ├── architecture.md
│   ├── setup_guide.md
│   ├── api_documentation.md
│   └── troubleshooting.md
│
└── data/
    ├── sample/
    │   └── sample_data.json
    └── schemas/
        └── avro_schemas/
```

## 📚 Additional Resources

- [Apache Airflow Documentation](https://airflow.apache.org/docs/)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)
- [Apache Spark Documentation](https://spark.apache.org/docs/latest/)
- [Apache Cassandra Documentation](https://cassandra.apache.org/doc/latest/)
- [Docker Documentation](https://docs.docker.com/)

## 📄 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.



## 🎉 Acknowledgments

- Apache Software Foundation for the amazing open-source tools
- Confluent for Kafka ecosystem enhancements
- Docker for containerization technology
- The open-source community for continuous inspiration

---

**Happy Streaming!** 🚀
