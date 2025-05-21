
# Flink Streaming 

![Apache Flink](https://flink.apache.org/img/flink-header-logo.svg)

A complete walkthrough for using the Ibis API with an Apache Flink cluster and Kafka for data ingestion and output.

## 📋 Overview

This project demonstrates how to:
- Use Ibis with Apache Flink for stream processing
- Process Kafka messages in real-time
- Apply windowed aggregations on streaming data
- Deploy everything with Docker Compose

## 🧰 Prerequisites

Before you begin, ensure you have:

- **🐳 Docker & Docker Compose**  
  Powers the entire infrastructure ([installation guide](https://docs.docker.com/compose/install/))

- **☕ Java 11 (JDK)**  
  Required by Flink ([download from Eclipse Temurin](https://adoptium.net/temurin/releases/?version=11))

- **🐍 Python 3.9 or 3.10**  
  With pip available for dependency management

## 🚀 Setup

1. **Clone the repository**
   ```bash
   git clone https://github.com/claypotai/ibis-flink-example.git
   cd ibis-flink-example
   ```

2. **Create and activate a virtual environment**
   ```bash
   python3 -m venv .venv
   source .venv/bin/activate
   ```

3. **Upgrade packaging tools**
   ```bash
   pip install --upgrade pip setuptools wheel
   ```

4. **Install Python dependencies**
   ```bash
   pip install -r requirements.txt
   ```
   > 💡 If you encounter errors related to `setuptools.build_meta` or numpy builds, confirm steps 2 and 3 were successful.

## 🏃‍♂️ Launch Services

From the project root, start all services:

```bash
docker compose up
```

This command will:
- ✅ Create Kafka topics `payment_msg` and `sink`
- ✅ Start generating 20,000 sample payment records
- ✅ Bring up a Flink job manager and task manager

You should see logs similar to:

```
init-kafka-1      | Created topics: payment_msg, sink
data-generator-1  | Producing 20000 records to payment_msg
flink-jobmanager-1| Starting Flink cluster...
```

> 💡 **Tip:** To launch only Kafka and the data generator (no Flink), run:
> ```bash
> docker compose up init-kafka data-generator
> ```

## 🔍 Inspect Raw Messages (Optional)

In a new terminal (with your virtual environment active):

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer("payment_msg")
for _, msg in zip(range(3), consumer):
    print(msg.value)
```

Example output:

```json
{
  "createTime": "2023-12-08 22:19:02.808",
  "orderId": 1702074256,
  "payAmount": 79901.88673289565,
  "payPlatform": 1,
  "provinceId": 1
}
```

## ⚡ Run the Window-Aggregation Example

The script [window_aggregation.py](window_aggregation.py) reads from `payment_msg`, applies a 10-second sliding window per province, and writes results to `sink`.

### 💻 Local Mode

Start an embedded Flink cluster:

```bash
python window_aggregation.py local
```

Results from the `sink` topic will print to your console. Use **Ctrl+C** to stop.

### 🌐 Remote Mode

Submit the job to the Docker-Compose Flink cluster:

1. **Locate the Flink CLI**
   ```bash
   python -c "import pyflink; from pathlib import Path; print(Path(pyflink.__spec__.origin).parent / 'bin' / 'flink')"
   ```

2. **Submit the job**
   ```bash
   /path/to/pyflink/bin/flink run \
     --jobmanager localhost:8081 \
     --python window_aggregation.py
   ```

You will see a confirmation message:
```
Job has been submitted with JobID <your-job-id>
```

## 📊 View Aggregated Results

Consume from the `sink` topic:

```python
from kafka import KafkaConsumer

consumer = KafkaConsumer("sink")
for _, msg in zip(range(10), consumer):
    print(msg.value)
```

Example output:

```json
{"province_id":1, "pay_amount":102381.88254099473}
{"province_id":5, "pay_amount":65711.48588438489}
```

## 🧹 Tear Down

1. Press **Ctrl+C** in the `docker compose up` terminal to stop services.
2. Clean up containers and networks:
   ```bash
   docker compose down
   ```

---

Feel free to modify [window_aggregation.py](window_aggregation.py) to explore other Ibis queries and windowing strategies!

## 📚 Additional Resources

- [Apache Flink Documentation](https://flink.apache.org/docs/stable/)
- [Ibis Framework Documentation](https://ibis-project.org/docs/)
- [Kafka Documentation](https://kafka.apache.org/documentation/)

