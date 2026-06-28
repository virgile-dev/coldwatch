# ColdWatch

> Open-source IoT temperature monitoring for restaurants — built with a Raspberry Pi and wireless sensors.

ColdWatch is a hobbyist side project I built to solve a real problem: restaurant owners have no reliable way of knowing if their fridges or freezers fail in the middle of the night. The system monitors temperatures in real time and sends an SMS alert the moment a threshold is exceeded or a device goes silent.

It has been running in a real Parisian restaurant since mid-2026.

---

## Features

- **Real-time monitoring** — sensors report continuously, 24/7
- **SMS alerts** — instant notification on threshold breach or device silence
- **Fault detection** — alerts if a sensor goes offline (dead battery, hardware failure, power outage)
- **Battery tracking** — per-sensor battery level visible in the dashboard
- **Web dashboard** — live readings, alert history, per-device alert snooze
- **HACCP compliance** — automated temperature log satisfies French food safety regulations

## Hardware

- **Hub**: Raspberry Pi (any Wi-Fi-capable model)
- **Sensors**: wireless, battery-powered temperature sensors (no wiring required)
- **Communication**: [protocol — e.g. Zigbee, BLE, LoRa]

## Stack

- Hub software: runs on Raspberry Pi, connected to the restaurant Wi-Fi
- Web dashboard: [add stack]
- SMS provider: [add provider — e.g. Free Mobile API, Twilio, OVH SMS]

_Stack documentation is a work in progress — contributions welcome._

## Getting started

### Prerequisites

- Raspberry Pi with Wi-Fi and Raspberry Pi OS
- Compatible wireless temperature sensors
- An SMS provider account and API key

### Installation

```bash
git clone https://github.com/virgile-dev/coldwatch.git
cd coldwatch
cp .env.example .env
# Fill in your SMS credentials and sensor config in .env
./setup.sh
```

_Full setup guide coming soon._

## Deployment

ColdWatch is designed for on-site deployment:

1. Install the Raspberry Pi at the restaurant and connect it to the local Wi-Fi
2. Fix wireless sensors on fridges and freezers (no wiring needed)
3. Log in to the web dashboard, set temperature thresholds and alert phone numbers
4. Done — ColdWatch monitors 24/7 and sends SMS on any anomaly

## Contributing

Contributions, bug reports, and feature requests are welcome.

- Open an issue for bugs or ideas
- Submit a PR for improvements or documentation
- If you're a restaurant owner and want to test ColdWatch, [reach out](mailto:contact@coldwatch.fr)

## License

[MIT](LICENSE) — use it, fork it, build on it. If you do something interesting with ColdWatch, I'd love to hear about it.
