# e-Commerce Event-Driven Java/Scala Microservices with Apache Kafka

## Table of Contents

1. [Introduction](#introduction)
2. [Features](#features)
3. [Architecture Overview](#architecture-overview)
4. [Prerequisites](#prerequisites)
5. [Getting Started](#getting-started)

   * [Clone the Repository](#clone-the-repository)
   * [Build with Maven (Java)](#build-with-maven-java)
   * [Build with SBT (Scala)](#build-with-sbt-scala)
6. [Configuration](#configuration)

   * [Kafka Brokers](#kafka-brokers)
   * [Application Properties](#application-properties)
   * [Topic Setup Script](#topic-setup-script)
7. [Running the Services](#running-the-services)

   * [Order Service](#order-service)
   * [Inventory Service](#inventory-service)
   * [Payment Service](#payment-service)
8. [Usage Examples](#usage-examples)

   * [Publishing an Order](#publishing-an-order)
   * [Consuming Events](#consuming-events)
9. [Monitoring & Logging](#monitoring--logging)
10. [Testing](#testing)
11. [Deployment (Docker & Kubernetes)](#deployment-docker--kubernetes)
12. [Contributing](#contributing)
13. [License](#license)
14. [Contact](#contact)

---

## Introduction

This project demonstrates a **Java** (Spring Boot) and **Scala** (Akka Streams) based microservices architecture for an e-commerce platform using **Apache Kafka** as the event backbone. Each service publishes/consumes domain events (e.g., orders created, inventory updated, payments processed) in an asynchronous, resilient manner.

Services included:

* **Order Service** (Java/Spring Boot)
* **Inventory Service** (Scala/Akka Streams)
* **Payment Service** (Java/Spring Boot)

---

## Features

* **Event-Driven**: Loose coupling between services via Kafka topics.
* **Resilience**: Automatic retry, dead-letter queues, and idempotent producers.
* **Scalability**: Horizontally scalable Kafka consumers and producers.
* **Polyglot**: Java and Scala services in one ecosystem.
* **Extensible**: Easily add new microservices and event types.

---

## Architecture Overview

```
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
```

Each service:

* Produces events to one or more Kafka topics.
* Consumes events from subscribed topics.
* Handles failures with retries and DLQs.

---

## Prerequisites

* **Java 11+** (Corretto, OpenJDK)
* **Scala 2.13+**
* **Maven 3.6+** (for Java modules)
* **SBT 1.5+** (for Scala modules)
* **Apache Kafka 2.8+**
* **Docker & Docker Compose** (optional, for containerized dev)

---

## Getting Started

### Clone the Repository

```bash
git clone https://github.com/your-username/ecommerce-kafka-microservices.git
cd ecommerce-kafka-microservices
```

### Build with Maven (Java)

```bash
cd order-service
mvn clean package -DskipTests

cd ../payment-service
mvn clean package -DskipTests
```

### Build with SBT (Scala)

```bash
cd inventory-service
sbt clean assembly
```

---

## Configuration

All services load configuration from `src/main/resources/application.yml` (or `.conf` for Scala).

### Kafka Brokers

```yaml
kafka:
  bootstrap-servers: localhost:9092
  acks: all
  retries: 3
```

### Application Properties (Java Example)

```yaml
spring:
  kafka:
    consumer:
      group-id: order-group
      auto-offset-reset: earliest
    producer:
      key-serializer: org.apache.kafka.common.serialization.StringSerializer
      value-serializer: org.springframework.kafka.support.serializer.JsonSerializer
```

### Topic Setup Script

Use the bundled `scripts/create-topics.sh` to create required topics:

```bash
chmod +x scripts/create-topics.sh
./scripts/create-topics.sh
```

---

## Running the Services

Ensure Kafka is running (`docker-compose up -d kafka zookeeper` or local broker).

### Order Service

```bash
cd order-service
java -jar target/order-service-0.1.0.jar
```

### Inventory Service

```bash
cd inventory-service
java -jar target/scala-2.13/inventory-service-assembly-0.1.0.jar
```

### Payment Service

```bash
cd payment-service
java -jar target/payment-service-0.1.0.jar
```

---

## Usage Examples

### Publishing an Order

```bash
curl -X POST http://localhost:8080/api/orders \
  -H "Content-Type: application/json" \
  -d '{ "orderId": "1234", "items": [{"sku":"ABC","qty":2}], "customerId":"cust-001" }'
```

This call:

1. Persists order in DB.
2. Produces an `OrderCreated` event to `orders.topic`.

### Consuming Events (Inventory Service)

The Inventory Service subscribes to `orders.topic` and:

1. Checks stock availability.
2. Emits `InventoryUpdated` event to `inventory.topic`.

---

## Monitoring & Logging

* **Spring Actuator** endpoints (`/actuator/health`, `/metrics`).
* **Micrometer** + Prometheus + Grafana for metrics dashboards.
* **Logback** structured JSON logs for correlation.
* Kafka Consumer Lag monitoring via `kafka.tools.ConsumerOffsetChecker` or built-in JMX.

---

## Testing

* **Unit Tests**: JUnit (Java) & ScalaTest (Scala).
* **Integration Tests**: Embedded Kafka using `spring-kafka-test` & `EmbeddedKafkaCluster` in Scala.

```bash
# Java services
cd order-service && mvn test
cd payment-service && mvn test

# Scala service
cd inventory-service && sbt test
```

---

## Deployment (Docker & Kubernetes)

* Each service includes a `Dockerfile`.

```bash
# Build Docker images
docker build -t your-username/order-service:latest ./order-service
docker build -t your-username/inventory-service:latest ./inventory-service
docker build -t your-username/payment-service:latest ./payment-service
```

* Kubernetes manifests are under `k8s/` (Deployments, Services, ConfigMaps).
* Sample Helm chart available in `helm/`.

---

## Contributing

Contributions are welcome! Please follow these steps:

1. Fork the repository.
2. Create a feature branch: `git checkout -b feature/my-feature`.
3. Commit changes: `git commit -m "Add some feature"`.
4. Push to branch: `git push origin feature/my-feature`.
5. Open a Pull Request.

Please ensure tests pass and code follows existing style.

---

## License

This project is licensed under the [MIT License](LICENSE).

---

## Contact

Maintainer: **Your Name** ([youremail@example.com](mailto:youremail@example.com))

Project Link: [https://github.com/your-username/ecommerce-kafka-microservices](https://github.com/your-username/ecommerce-kafka-microservices)
