2.1 Objective
Build an async Python service that consumes real-time quotes / trades from Polygon (or IEX Cloud sandbox) and publishes them to a Kafka topic (quotes, trades) using Avro-encoded messages.

2.2 Tasks & Solo-Tests (list format)
2.2.1 Local Kafka sandbox

Action: add docker-compose.yml with single-broker Kafka + Schema Registry.

Immediate test: docker compose up -d kafka exposes localhost:9092; kafka-topics --bootstrap-server localhost:9092 --list returns an empty list.

2.2.2 Python project scaffold

Action: create package tg_data_adapter/ with __init__.py and main.py.

Immediate test: python -m tg_data_adapter.main --help prints CLI usage text.

2.2.3 Dependency pinning

Action: add poetry or requirements.txt entries:

css
Copy
Edit
aiohttp
confluent-kafka[avro]
python-dotenv
Immediate test: pip install -r requirements.txt finishes without error.

2.2.4 Avro schema

Action: define schemas/quote.avsc (top-of-book fields) and schemas/trade.avsc.

Immediate test: run avro-tools cat schemas/quote.avsc and verify valid JSON.

2.2.5 Config management

Action: add .env.sample with POLYGON_API_KEY, KAFKA_BOOTSTRAP, SCHEMA_REGISTRY_URL.

Immediate test: dotenv-linter reports no issues.

2.2.6 Async websocket client

Action: implement PolygonClient in client.py using aiohttp to subscribe to T.* (trades) and Q.* (quotes).

Immediate test: pytest tests/test_polygon_client.py::test_handshake asserts the client receives a connected status message from sandbox within 2 s.

2.2.7 Avro producer wrapper

Action: implement KafkaAvroProducer with schema-registry auto-registration.

Immediate test: pytest tests/test_producer.py serializes a fake record, produces to quotes, consumes it back (with kafka-console-consumer) and decodes identical payload.

2.2.8 Adapter orchestration

Action: in main.py, wire client → async queue → producer; add graceful shutdown.

Immediate test: run python -m tg_data_adapter.main --dry-run 5 (5 s) and ensure at least one Produced log line appears.

2.2.9 Metrics & logging

Action: expose Prometheus counter messages_produced_total on :8000/metrics.

Immediate test: curl localhost:8000/metrics contains the counter name.

2.2.10 PyTest integration (self-test)

Action: tests/test_data_adapter.py spins up the adapter for 60 s against sandbox, then queries Kafka via kafka-python to count records.

Immediate test: asserts ≥ 100 messages/min in the quotes topic. (Use @pytest.mark.skipif when sandbox is offline.)

2.3 Deliverables
Directory tg_data_adapter/ with production-ready code

Avro schemas in schemas/quote.avsc and schemas/trade.avsc

docker-compose.yml for local Kafka + Schema Registry

Full unit & integration test suite under tests/

Prometheus metrics endpoint

2.4 End-to-End Self-Test
bash
Copy
Edit
# 1. Spin up infra
docker compose up -d kafka schema-registry

# 2. Run adapter for one minute
POLYGON_API_KEY=$YOUR_KEY python -m tg_data_adapter.main --run 60

# 3. Count messages
kafka-consumer-groups --bootstrap-server localhost:9092 \
  --describe --group test-group --topic quotes

# 4. Verify ≥ 100 messages/min
pytest tests/test_data_adapter.py
When the integration test confirms the 100-messages-per-minute threshold and the Prometheus endpoint responds, Step 2 is complete, and downstream services can safely consume live market data from Kafka.
