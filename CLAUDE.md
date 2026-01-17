# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Home Assistant custom integration for Meshtastic MQTT nodes. It consumes Protobuf MQTT messages from Meshtastic devices and exposes them as Home Assistant entities (sensors, binary sensors, device trackers).

## Architecture

### Core Components

- **Platform** (`coordinator.py`): Singleton that manages persistent storage across all node coordinators. Stored in `hass.data[DOMAIN]`.

- **Coordinator** (`coordinator.py`): Per-node `DataUpdateCoordinator` that:
  - Subscribes to MQTT topics for protobuf messages and optional stat messages
  - Decrypts encrypted payloads using AES-CTR (via `proto.py`)
  - Parses Meshtastic protobuf messages into typed dictionaries (position, device_metrics, environment_metrics, power_metrics, nodeinfo, neighborinfo, text_message)
  - Persists state to HA storage

- **Proto conversion** (`proto.py`): Handles protobuf parsing and decryption. Maps Meshtastic portnums to converter functions. Uses the `meshtastic` Python package for protobuf definitions.

- **Entity platforms**: Each platform (sensor, binary_sensor, device_tracker) creates entities that read from coordinator data dictionaries.

### Data Flow

1. MQTT message arrives on subscribed topic
2. Coordinator parses protobuf `ServiceEnvelope`
3. If encrypted, decrypts using configured key (or default key `AQ==`)
4. Converts to typed dictionary based on portnum
5. Updates coordinator data and persists to storage
6. Entities read from coordinator data via properties

### Configuration

Configured via config flow with:
- `id`: Node ID (e.g., `!aabbccdd`)
- `pb_topic`: Protobuf MQTT topic
- `key`: Optional base64 encryption key
- `stat_topic`: Optional status topic for online/offline

## Dependencies

- Requires Home Assistant MQTT integration
- Uses `meshtastic` Python package for protobuf definitions
- Uses `cryptography` for AES-CTR decryption
