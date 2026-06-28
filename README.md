# ColdWatch

> Open-source IoT temperature monitoring for restaurants — Raspberry Pi, BLE sensors, SMS alerts.

ColdWatch monitors fridge and freezer temperatures in real time and sends an SMS the moment a threshold is exceeded or a device goes silent. It has been running in a real Parisian restaurant since mid-2026.

---

## Architecture

Two components, designed to run on the same Raspberry Pi Zero:

```
[Jaalee BLE sensors]
        │  passive BLE advertisements
        ▼
[coldwatch-client]  ──── MQTT ────▶  [broker]
  collector.py                           │
  asyncio · bleak                        │
                                         ▼
                               [coldwatch-server]
                                 chori-mqtt-server   ←── AlertState
                                 paho-mqtt · SQLite       SMS via Free Mobile
                                         │
                                         ▼
                               [chori-web]
                                 Flask dashboard
                                 read-only · same DB
```

### coldwatch-client

Single-file asyncio service (`collector.py`, ~800 lines). Passively scans for BLE advertisements from Jaalee temperature/humidity sensors, decodes manufacturer data (iBeacon and compact formats, SHT20/SHT31 models), and publishes scalar MQTT topics plus a periodic heartbeat. No database, no HTTP, no UI — BLE in, MQTT out.

- **Runtime**: Python 3.11+, asyncio throughout
- **BLE**: [bleak](https://github.com/hbldh/bleak) ≥ 0.22 — passive scanning, no pairing
- **MQTT**: paho-mqtt ≥ 1.6
- **Topics published**: `chori/<mac>/temp`, `chori/<mac>/rh`, `chori/<mac>/batt`, heartbeat
- **Config**: `/etc/default/chori-client` (env file)
- **Deployment**: systemd service via `scripts/install-systemd-service.sh`
- **Dependencies**: installed via `apt` (`python3-bleak`, `python3-paho-mqtt`) — no pip, no venv

### coldwatch-server

One package (`chori-mqtt`), two processes:

**`chori-mqtt-server`** — MQTT subscriber. Listens for sensor readings from the local broker, writes to SQLite, and runs alert checks on a periodic loop. Sends SMS via the Free Mobile HTTP API (raw `urllib`, no library).

**`chori-web`** — Flask dashboard. Read-only UI querying the same SQLite database: live sensor readings, device status, heartbeat indicator, per-device alert snooze.

- **Framework**: Flask (dashboard only)
- **MQTT**: paho-mqtt
- **Storage**: SQLite with WAL mode — one file, schema managed in-code, hand-rolled migrations
- **Tables**: `devices`, `sensors`, `readings`, `readings_archive`, `heartbeats`, `snoozes`
- **Alerting**: in-memory `AlertState` machine — checks temperature thresholds, battery levels, device timeouts; seeded from DB at startup; snoozes persisted in SQLite (per-device, optional expiry)
- **SMS**: Free Mobile HTTP API (`FREE_MOBILE_LOGIN`, `FREE_MOBILE_API_KEY` env vars)
- **Config**: `alerts.json` + environment variables
- **Deployment**: two systemd services on a Pi Zero — no Docker, no CI

## Hardware

- **Raspberry Pi Zero** (or any Pi running Raspberry Pi OS / Debian Bookworm)
- **Sensors**: Jaalee BLE temperature/humidity sensors — SHT20 or SHT31 models, battery-powered, no wiring

## Getting started

### Prerequisites

- Raspberry Pi Zero running Raspberry Pi OS Bookworm
- Jaalee BLE temperature sensors
- A local MQTT broker (e.g. Mosquitto)
- A [Free Mobile](https://mobile.free.fr/) account with SMS API enabled

### Client

```bash
git clone https://github.com/virgile-dev/coldwatch.git
cd coldwatch/client

# Install dependencies via apt
sudo apt install python3-bleak python3-paho-mqtt

# Configure
sudo nano /etc/default/chori-client
# Set MQTT_HOST, SENSOR_MACS, SENSOR_MODEL (sht20 or sht31)

# Install and start the systemd service
sudo bash scripts/install-systemd-service.sh
sudo systemctl enable --now chori-client
```

### Server

```bash
cd coldwatch/server

# Install dependencies
pip install flask paho-mqtt

# Configure
cp alerts.json.example alerts.json
export FREE_MOBILE_LOGIN=your_login
export FREE_MOBILE_API_KEY=your_key

# Start both processes
systemctl enable --now chori-mqtt-server
systemctl enable --now chori-web
```

## Contributing

Contributions, bug reports, and feature requests are welcome.

- Open an issue for bugs or ideas
- Submit a PR for improvements or documentation
- If you're a restaurant owner and want to test ColdWatch, [reach out](mailto:contact@coldwatch.fr)

## License

[MIT](LICENSE)
