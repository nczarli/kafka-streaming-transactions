# Spark Kafka Demo

## Overview

This project demonstrates a real-time data pipeline using **Apache Spark Structured Streaming** with **Apache Kafka**. It processes transaction data, filtering eligible transactions and computing cashback before writing the results back to Kafka.

## Architecture

1. **Kafka** is used as a message broker for input and output data streams.
2. **Apache Spark** reads transaction data from Kafka, processes it, and writes the transformed data back to Kafka.
3. Transactions are filtered based on:
   - Amount ≥ 15
   - Transactions made on **Friday** (Day 6)
   - Merchant ID is **MerchantX**
4. If a transaction meets the criteria, a **cashback of 15%** is calculated.

## Prerequisites

Ensure you have the following installed:

- [Docker](https://www.docker.com/)
- [Docker Compose](https://docs.docker.com/compose/)
- [Apache Spark](https://spark.apache.org/)
- [Kafka](https://kafka.apache.org/)

## Getting Started

### 1. Start Kafka and Zookeeper
Run the following command to start the services:

```sh
docker compose -f ./docker-compose.yml --project-name sparkkafkademo up
```

### 2. Create a Kafka Topic
To create an input topic for transactions:

```sh
docker exec -it kafka kafka-topics --create --bootstrap-server kafka:9092 --replication-factor 1 --partitions 1 --topic input
```

### 3. Start the Spark Application
Run the Spark job to process Kafka events:

```sh
python spark_processing.py
```

### 4. Producing Transactions to Kafka
To send transactions to Kafka, run:

```sh
docker exec -it kafka kafka-console-producer --bootstrap-server kafka:9092 --topic input
```

Then enter JSON transaction data, e.g.:

```json
{"transaction_id": "1", "customer_id": "123", "amount": 20, "transaction_timestamp": "2023-04-21T09:30:00Z", "merchant_id": "MerchantX"}
```

### 5. Consuming Processed Transactions
To see the processed transactions (with cashback), run:

```sh
docker exec -it kafka kafka-console-consumer --bootstrap-server kafka:9092 --topic output
```

## Transaction Data Format

Each transaction must be in JSON format:

```json
{
  "transaction_id": "1",
  "customer_id": "123",
  "amount": 20,
  "transaction_timestamp": "2023-04-21T09:30:00Z",
  "merchant_id": "MerchantX"
}
```

## Spark Streaming Logic

The Spark job:
- Reads the `input` topic from Kafka.
- Parses the JSON data using the schema.
- Filters transactions based on the criteria.
- Computes a **15% cashback** for qualifying transactions.
- Writes the transformed data to the `output` Kafka topic.

## Kafka Configuration

**Input:**
```python
kafka_input_config = {
    "kafka.bootstrap.servers": "kafka:9092",
    "subscribe": "input",
    "startingOffsets": "latest",
    "failOnDataLoss": "false"
}
```

**Output:**
```python
kafka_output_config = {
    "kafka.bootstrap.servers": "kafka:9092",
    "topic": "output",
    "checkpointLocation": "./check.txt"
}
```

## Stopping the Services

To stop and remove containers:

```sh
docker compose -f ./docker-compose.yml --project-name sparkkafkademo down
```

## License

This project is open-source and available under the MIT License.
