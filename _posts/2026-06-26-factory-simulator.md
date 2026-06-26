---
layout: post
title: A Python Factory Simulator for IIoT Testing
subtitle: Simulate real industrial devices over OPC UA, MQTT, and Sparkplug B — without any hardware
cover-img: /assets/img/path.jpg
thumbnail-img: /assets/img/thumb.png
share-img: /assets/img/path.jpg
tags: [IIoT, Python, OPC-UA, MQTT, Sparkplug]
author: Pankaj K
---

Building an IIoT data pipeline — whether for ingestion, alarming, or analytics — usually requires a live PLC or field device to test against. That hardware is expensive, shared, and often unavailable in a development environment. This project is a lightweight Python simulator that stands in for real factory equipment by generating realistic tag data and publishing it over three common industrial protocols.<!--more-->

The GitHub repo: **https://github.com/pankaj-k/simple_simulator**


## What it simulates

The simulator models a small factory with four categories of device, each running independently on a configurable tick interval (default: 2 seconds):

| Device | Area | Example IDs |
|---|---|---|
| CNC machines / robot arm | Assembly | `CNC_01`, `CNC_02`, `Robot_Arm_01` |
| Reactor tanks | Process | `Reactor_Tank_01`, `Reactor_Tank_02` |
| Energy meters | Energy | `Main_Meter`, `Line_A_Meter`, `Line_B_Meter` |
| Conveyor/packaging line | Packaging | `Packaging_Line_01` |

All devices share the same base contract: a `tick(elapsed)` method advances internal state, and `get_tags()` returns the current tag dictionary. Tags carry a value, an engineering unit, a datatype, and a quality field (`Good` / `Bad` / `Uncertain`).


## Device behaviour

**Assembly machines** cycle through three states — RUNNING, FAULT, and IDLE. Tool wear accumulates while running and resets after a maintenance-style idle. A small random fault probability (`0.2% per tick`) occasionally trips a machine into FAULT, and it self-recovers after 30 seconds. Tags published include `State`, `PartCount`, `CycleTime_sec`, `ToolWear_pct`, and `Alarm`.

**Process tanks** use sinusoidal functions with added Gaussian noise to model temperature and pressure — similar to what you see in real control loops. A simple level balance handles inlet/outlet flow so the tank doesn't drain to zero. Tags include `Temperature_C`, `Pressure_bar`, `Level_pct`, `FlowRate_L_min`, and `HeaterOn`.

**Energy meters** observe the assembly machines and compute a load factor based on how many are currently running. 30% of the base load is always present (HVAC, lighting, idle equipment); the rest tracks production. Tags include `ActivePower_kW`, `EnergyTotal_kWh`, `PowerFactor`, `Voltage_V`, and `Current_A`.

**Packaging lines** are downstream of the assembly machines — supply factor is computed from how many feeder machines are running. A low supply factor increases reject rate and makes recovery harder. OEE (Overall Equipment Effectiveness) is computed as Availability × Performance × Quality and published as `OEE_pct`.


## Three protocol modes

The simulator ships three connectors, selectable at startup:

```
python -m simulator.main opcua
python -m simulator.main mqtt
python -m simulator.main sparkplug
```

**OPC UA** (`asyncua` library) builds a proper address space under `Factory/<Area>/<DeviceID>/<TagName>` and serves a live OPC UA server on `opc.tcp://0.0.0.0:4840/factory/`. Each tag is a writable variable node with the correct variant type. Tag quality maps to OPC UA status codes: `BadDeviceFailure` for Bad, `UncertainLastUsableValue` for Uncertain.

**Plain MQTT** publishes a JSON payload every tick to `factory/<Area>/<DeviceID>`. Each message contains a millisecond timestamp, area, device ID, and a tag map with value, unit, and quality:

```json
{
  "timestamp": 1750900000000,
  "area": "Assembly",
  "device": "CNC_01",
  "tags": {
    "State":        { "v": "RUNNING", "u": "",    "q": "Good" },
    "ToolWear_pct": { "v": 42.3,      "u": "%",   "q": "Good" },
    "Alarm":        { "v": false,     "u": "",    "q": "Good" }
  }
}
```

**Sparkplug B** (spBv1.0) implements the full birth/death lifecycle over MQTT. On connect, the edge node publishes an NBIRTH, then a DBIRTH for every device. Each tick produces a DDATA per device. A pre-configured MQTT will message handles NDEATH automatically on disconnect. This is the right choice if you are testing a Sparkplug-aware SCADA or historian.


## Fault injection

Pass `--test` to enable the `FaultInjector`:

```
python -m simulator.main opcua --test
```

On each tick, every tag has a small probability of having its quality flipped to `Bad` (60%) or `Uncertain` (40%), for a random duration between 2 and 120 seconds. Recovery is logged at INFO level, faults at WARNING. With ~40 tags across all devices you can expect the first fault within roughly 25 seconds of starting — useful for verifying that your alarm pipeline actually fires on Bad quality and clears on recovery.


## Configuration

Everything lives in `config/factory.yaml`:

```yaml
simulation:
  tick_interval: 2.0   # seconds between updates

opcua:
  endpoint: "opc.tcp://0.0.0.0:4840/factory/"

mqtt:
  broker: "localhost"
  port: 1883
  topic_prefix: "factory"

sparkplug:
  broker: "localhost"
  port: 1883
  group_id: "Factory_01"
  edge_node_id: "Edge_Node_01"
```


## Getting started

```bash
git clone https://github.com/pankaj-k/simple_simulator.git
cd simple_simulator
python -m venv .venv && source .venv/bin/activate   # Windows: .venv\Scripts\activate
pip install -r requirements.txt

# Start an OPC UA server (no broker needed)
python -m simulator.main opcua

# Or point at a local Mosquitto broker and publish MQTT
python -m simulator.main mqtt

# Sparkplug B with fault injection enabled
python -m simulator.main sparkplug --test
```

Dependencies are intentionally minimal: `asyncua`, `paho-mqtt`, `pyyaml`, and `numpy`.


## Extending it

Each device is a subclass of `Device` in `simulator/factory/base.py`. Adding a new device type is four steps: subclass `Device`, implement `tick()`, call `self._set()` to update tags, and register it in `build_factory()` in `main.py`. The connector layer is protocol-only — it never needs to know what any tag means.

This makes the simulator a convenient starting point for testing anything that speaks OPC UA, MQTT, or Sparkplug B: historians, time-series databases, dashboards, alerting engines, or ML pipelines that need a steady stream of labelled industrial data.
