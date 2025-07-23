-Commerce Event-Driven Java/Scala Microservices with Apache Kafka

Table of Contents

Introduction

Features

Architecture Overview

Prerequisites

Getting Started

Clone the Repository

Build with Maven (Java)

Build with SBT (Scala)

Configuration

Kafka Brokers

Application Properties

Topic Setup Script

Running the Services

Order Service

Inventory Service

Payment Service

Usage Examples

Publishing an Order

Consuming Events

Monitoring & Logging

Testing

Deployment (Docker & Kubernetes)

Contributing

License

Contact

Introduction

This project demonstrates a Java (Spring Boot) and Scala (Akka Streams) based microservices architecture for an e-commerce platform using Apache Kafka as the event backbone. Each service publishes/consumes domain events (e.g., orders created, inventory updated, payments processed) in an asynchronous, resilient manner.

Services included:

Order Service (Java/Spring Boot)

Inventory Service (Scala/Akka Streams)

Payment Service (Java/Spring Boot)

Features

Event-Driven: Loose coupling between services via Kafka topics.

Resilience: Automatic retry, dead-letter queues, and idempotent producers.

Scalability: Horizontally scalable Kafka consumers and producers.

Polyglot: Java and Scala services in one ecosystem.

Extensible: Easily add new microservices and event types.

Architecture Overview

+---------------+      orders.topic      +---------------+
|  Order UI /   |  -------------------->  | Order Service |
|     Client    |                       |               |
+---------------+                       +-------+-------+
                                              |
               +------------------------------+------------------------------+
               |                              |                              |
        payment.topic                 inventory.topic               dead-letter.topic
               |                              |                              |
    +----------v---------+         +----------v---------+         +----------v---------+
    | Payment Service    |         | Inventory Service |         |  DLQ Processor     |
    | (Spring Boot)      |         | (Akka Streams)    |         | (Spring Boot)      |
    +--------------------+         +-------------------+         +--------------------+

Each service:

Produces events to one or more Kafka topics.

Consumes events from subscribed topics.

Handles failures with retries and DLQs.

Prerequisites

Java 11+ (Corretto, OpenJDK)

Scala 2.13+

Maven 3.6+ (for Java modules)

SBT 1.5+ (for Scala modules)

Apache Kafka 2.8+

Docker & Docker Compose (optional, for containerized dev)

Getting Started

Clone the Repository

git clone https://github.com/your-username/ecommerce-kafka-microservices.git
cd ecommerce-kafka-microservices

Build with Maven (Java)

cd order-service
mvn clean package -DskipTests

cd ../payment-service
mvn clean package -DskipTests

Build with SBT (Scala)

cd inventory-service
sbt clean assembly

Configuration

All services load configuration from src/main/resources/application.yml (or .conf for Scala).

Kafka Brokers

kafka:
  bootstrap-servers: localhost:9092
  acks: all
  retries: 3

Application Properties (Java Example)

spring:
  kafka:
    consumer:
      group-id: order-group
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer

Topic Setup Script

Use the bundled scripts/create-topics.sh to create required topics:

chmod +x scripts/create-topics.sh
./scripts/create-topics.sh

Running the Services

Ensure Kafka is running (docker-compose up -d kafka zookeeper or local broker).

Order Service

cd order-service
java -jar target/order-service-0.1.0.jar

Inventory Service

cd inventory-service
java -jar target/scala-2.13/inventory-service-assembly-0.1.0.jar

Payment Service

cd payment-service
java -jar target/payment-service-0.1.0.jar

Usage Examples

Publishing an Order

curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{ "orderId": "1234", "items": [{"sku":"ABC","qty":2}], "customerId":"cust-001" }'

This call:

Persists order in DB.

Produces an OrderCreated event to orders.topic.

Consuming Events (Inventory Service)

The Inventory Service subscribes to orders.topic and:

Checks stock availability.

Emits InventoryUpdated event to inventory.topic.

Monitoring & Logging

Spring Actuator endpoints (/actuator/health, /metrics).

Micrometer + Prometheus + Grafana for metrics dashboards.

Logback structured JSON logs for correlation.

Kafka Consumer Lag monitoring via kafka.tools.ConsumerOffsetChecker or built-in JMX.

Testing

Unit Tests: JUnit (Java) & ScalaTest (Scala).

Integration Tests: Embedded Kafka using spring-kafka-test & EmbeddedKafkaCluster in Scala.

# Java services
cd order-service && mvn test
cd payment-service && mvn test

# Scala service
cd inventory-service && sbt test

Deployment (Docker & Kubernetes)

Each service includes a Dockerfile.

# Build Docker images
docker build -t your-username/order-service:latest ./order-service
docker build -t your-username/inventory-service:latest ./inventory-service
docker build -t your-username/payment-service:latest ./payment-service

Kubernetes manifests are under k8s/ (Deployments, Services, ConfigMaps).

Sample Helm chart available in helm/.

Contributing

Contributions are welcome! Please follow these steps:

Fork the repository.

Create a feature branch: git checkout -b feature/my-feature.

Commit changes: git commit -m "Add some feature".

Push to branch: git push origin feature/my-feature.

Open a Pull Request.

Please ensure tests pass and code follows existing style.

License

This project is licensed under the MIT License.
