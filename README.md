# Matter Controller Server

This project is a custom development and extension based on the **[Open Home Foundation Matter Server](https://github.com/matter-js/python-matter-server)**, which is an officially certified Software Component to create a Matter controller. 

While the upstream project serves as the foundation to provide Matter support to Home Assistant, this repository extends that foundation into a versatile, standalone Matter Controller Server. It is designed to allow multiple frontends, applications, and services to interact with a single Matter fabric over a centralized WebSocket API.

## What This Project Does

At a high level, this project acts as a Matter controller backend that:
- Runs and owns a Matter fabric/controller context.
- Stores commissioned device information and controller state persistently.
- Exposes a WebSocket API (`/ws`) for external consumers.
- Allows multiple clients to interact with the same Matter fabric simultaneously.
- Supports commissioning, device control, attribute reads/writes, and event streaming.

Instead of embedding the Matter stack directly into your frontend, this server centralizes the Matter operations, making it easy to build custom Home Assistant integrations, web dashboards, mobile apps (e.g., Flutter, Android, iOS), or automated test harnesses.

## System Architecture

The integration model separates the UI from the controller logic:

`Frontend / App / Automation Platform -> WebSocket API -> Matter Server -> Matter Devices`

This architecture ensures that the Matter fabric remains online even if frontends disconnect, and it simplifies frontend development since the heavy lifting of the Matter SDK is handled by this server.

## Getting Started

### Prerequisites

- **OS:** 64-bit Linux is strongly recommended for production and containerized deployments (due to host networking requirements for Matter). macOS is supported for development. Windows/WSL is not supported for the Matter server runtime.
- **Network:** IPv6, link-local multicast, and mDNS/zeroconf must be supported on your local network.

### Running with Docker

The easiest way to run the Matter Server is using Docker with host networking.

**Minimal Run (Persistent Storage):**
```sh
mkdir -p data

docker run -d \
  --name matter-server \
  --restart=unless-stopped \
  --security-opt apparmor=unconfined \
  -v "$(pwd)/data:/data" \
  --network=host \
  ghcr.io/matter-js/python-matter-server:stable
```

**Run with Bluetooth Commissioning Support:**
If your host machine has a Bluetooth adapter and you want to use BLE for commissioning, mount the D-Bus socket:
```sh
mkdir -p data

docker run -d \
  --name matter-server \
  --restart=unless-stopped \
  --security-opt apparmor=unconfined \
  -v "$(pwd)/data:/data" \
  -v /run/dbus:/run/dbus:ro \
  --network=host \
  ghcr.io/matter-js/python-matter-server:stable \
  --storage-path /data \
  --paa-root-cert-dir /data/credentials \
  --bluetooth-adapter 0
```

## Connecting a Frontend

Once the server is running, any WebSocket-capable client can connect to it on port `5580`.

**Connection URL:**
- Local: `ws://localhost:5580/ws`
- LAN: `ws://<server-ip>:5580/ws`

**Basic Flow:**
1. Open the WebSocket connection.
2. Parse the initial `ServerInfoMessage` to verify schema compatibility.
3. Send a `start_listening` command to initialize the client state stream.
4. Issue commands (e.g., `get_nodes`, `commission_with_code`, `device_command`).
5. Listen for events (e.g., `node_updated`, `attribute_updated`).

For specific domain integrations, such as **EV Chargers**, the server supports detailed clusters like `EnergyEvse`, `DeviceEnergyManagement`, and telemetry clusters to build rich dashboards and control flows.

## Documentation

For a comprehensive guide on integrating with this project, supported commands, and architectural constraints, please refer to the **[Developer Integration Guide](docs/project-overview-developer.md)**.

## Support & Contributions

- If you are looking to contribute or understand the core Matter SDK implementation, refer to the [upstream project](https://github.com/matter-js/python-matter-server).
- For issues related to this specific extension and architecture, please use the issue tracker in this repository.
