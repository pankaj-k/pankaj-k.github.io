---
layout: post
title: Factory Data to Kafka - OPC UA, Ignition, and Confluent Cloud
subtitle: A config-only pipeline with git version control baked into the SCADA layer
cover-img: /assets/img/watercolor_night_logstash_dockers.png
thumbnail-img: /assets/img/ignitionkafkathumb.png
share-img: /assets/img/ignitionkafkathumb.png
tags: [IIoT, Ignition, Kafka, OPC-UA, Confluent, Docker]
author: Pankaj K
---


We will use Ignition modules to stream machine data to Kafka. Configurations are already done and docker compose is used to create the Ignition instance. It also shows how version control can be used with Ignition infrastructure. The README.md and Playbook.md are the best documentation for this code repo<!--more-->

**[GitHub repo: pankaj-k/ignition\_kafka](https://github.com/pankaj-k/ignition_kafka)**

---

## The pipeline

```
Factory Simulator (OPC UA) → Ignition Gateway (Docker) → Confluent Cloud Kafka
```

The repo comes with a Python OPC UA server publishing realistic tag data across four device categories: assembly machines, process tanks, energy meters, and a packaging line. Ignition connects to it as an OPC UA client and forwards tag changes to Kafka via its built-in Event Streams and Kafka Connector modules.

No application code anywhere in this repo. Just configuration.

---

## SCADA version control

Ignition 8.3 stores its configuration as plain JSON files on disk. OPC UA connections, Kafka connectors, Event Stream definitions — all of it lives in a directory called `data/` inside the Ignition installation. Not in a proprietary database, not in a binary blob.

That makes this possible in `docker-compose.yml`:

```yaml
volumes:
  - ./gateway-data:/usr/local/bin/ignition/data
```

`./gateway-data` is a directory inside the cloned repo, mounted as Ignition's data directory inside the container. When you configure an OPC UA connection through the Ignition UI and hit Save, a JSON file appears in `gateway-data/` on your laptop. `git diff` shows exactly which fields changed.

```
Make a change in Ignition UI
        ↓
Ignition writes JSON to its data directory
        ↓  (same physical files, via bind mount)
File appears in gateway-data/ on your host
        ↓
git status → git diff → git commit
```

The OPC UA connection to the simulator and all five Event Stream definitions are already committed in the repo. Clone it, run `docker compose up`, and they're pre-configured — no UI steps for those.

---

## What you still configure manually

One resource deliberately isn't committed: the Kafka connection. The password is encrypted with an instance-specific key, and the API key and bootstrap server are real credentials that shouldn't live in a public repo.

In the Ignition UI: **Connections → Service Connectors → Create Connection → Kafka**

A few fields that need non-default values:

| Field | Value |
|-------|-------|
| Name | `confluent-kafka` — Event Streams reference the connector by name; it must match exactly |
| Security Protocol | `SASL_SSL` — the UI defaults to `SASL_PLAINTEXT`, which Confluent Cloud silently rejects |
| SASL Mechanism | `PLAIN` |

The `SASL_SSL` vs `SASL_PLAINTEXT` mismatch has an unhelpful failure mode: the connection status shows "Errored" with no explanation in the UI. The actual TLS error is in the gateway logs. Worth knowing before you spend time looking at the wrong thing.

---

## Event Streams

Five streams are pre-configured in the repo:

| Stream | Kafka topic |
|--------|-------------|
| Assembly machines | `factory.assembly` |
| Process tanks | `factory.process` |
| Energy meters | `factory.energy` |
| Packaging line | `factory.packaging` |
| Alarm events | `factory.quality` |

Each stream has a transform script that enriches events with area, machine ID, and tag name before they reach Kafka. Subscriptions use wildcards (`[default]Factory/Assembly/**`) so new simulator tags are automatically included without touching the stream config.

Create the five topics in Confluent Cloud before starting Ignition — the repo includes a [Terraform config](https://github.com/pankaj-k/ignition_kafka/tree/main/terraform) if you prefer that over the UI. Either way takes about two minutes.
And I have not yet tested the Terraform module.


---

## Getting started

Prerequisites: Python 3.9+, Docker, Docker Compose, Confluent Cloud account (free tier is sufficient).

```bash
# 1. Start the factory OPC UA server
cd simulator
python -m venv .venv
source .venv/bin/activate      # Windows: .venv\Scripts\activate
pip install -r requirements.txt
python -m simulator.main opcua

# 2. Start Ignition
cd infra
docker compose up -d
```

Ignition is at `http://localhost:8088` (admin / Ignition@123). Create the Kafka connection as described above. The OPC UA connection to the simulator is already wired up.

**Linux note:** Ignition runs as uid 999 inside the container. On Linux hosts, the gateway-data directory needs to match:

```bash
sudo chown -R 999:999 infra/gateway-data
```

Not needed on Windows or macOS — Docker Desktop handles this automatically.

---

## What this is for

It is a hands-on lab, not a production template. Run it against a real Ignition instance and you can see how tags travel from OPC UA through Event Streams into Kafka topics, inspect the payload structure at each stage, watch alarm events flow through, and add new device types to the simulator without touching any Ignition configuration.

The git + bind mount approach works beyond this demo. Any Ignition configuration change made through the UI — a new connection, a modified stream, a Perspective dashboard — becomes a reviewable, diff-able commit. For teams used to code review, that is a useful property to have in a SCADA layer.
