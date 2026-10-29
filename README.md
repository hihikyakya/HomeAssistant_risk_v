# Home Assistant on RISC-V

## 1. Overview

This project provides a Docker-based build and deployment configuration for running Home Assistant on RISC-V systems.

The Docker image is built locally using the provided Dockerfile. Depending on the target board, operating system, and available dependencies, additional configuration or build modifications may be required.

## 2. Requirements

- A supported RISC-V 64-bit system
- A compatible Linux distribution
- Docker Engine
- Docker Compose v2
- Sufficient storage and memory for building and running Home Assistant

## 3. Configuration

### Persistent configuration

The following volume mapping stores Home Assistant configuration files on the host:

```yaml
volumes:
  - ./homeassistant/config:/config
```

This allows configuration data to persist when the container is recreated.

### Time zone

The default configuration uses UTC:

```yaml
environment:
  TZ: UTC/GMT
```

Modify the `TZ` value if you prefer a different time zone, such as `Asia/Seoul`.

## 4. Optional device configuration

Some integrations may require direct access to hardware devices exposed by the host operating system.

Docker Compose supports device mappings through the `devices` option.

Example:

```yaml
services:
  homeassistant:
    # Other configuration omitted
    devices:
      - /dev/ttyUSB0:/dev/ttyUSB0
```

Replace `/dev/ttyUSB0` with the actual device path on your system.

Common examples include:

- USB serial adapters
- Zigbee coordinators
- Z-Wave controllers
- Other supported serial devices

Check available device paths on the host:

```bash
ls -l /dev/ttyUSB*
ls -l /dev/serial/by-id/
```

If a device is not present, the corresponding command may return an error. Use the paths that actually exist on your system.

**Important notes:**

- Device mappings are optional. Do not add them unless your hardware or integration requires them.
- Device paths may differ between boards and operating systems.
- Prefer stable paths under `/dev/serial/by-id/` when available.
- Only expose devices required by your integrations.
- Some devices may require additional permissions or host configuration.
- Device availability does not guarantee compatibility with Home Assistant or the target architecture.

Some integrations may also require additional network configuration for device discovery or communication.

## 5. Build and start

Run the following commands from the project root directory.

Build the image:

```bash
docker compose build
```

Start Home Assistant:

```bash
docker compose up -d
```

Check the container status:

```bash
docker compose ps
```

View the logs:

```bash
docker compose logs -f homeassistant
```

Access Home Assistant at:

```text
http://<HOST_IP>:8123
```

Replace `<HOST_IP>` with the IP address of your host system.

## 6. Troubleshooting

If the container fails to start or the build fails, check the logs first:

```bash
docker compose logs --tail=200 homeassistant
```

Depending on the error, you may need to:

- Install additional system dependencies.
- Modify the Dockerfile for your target environment.
- Add device mappings for hardware integrations.
- Resolve architecture-specific package or compilation issues.
- Adjust permissions for files or devices accessed by the container.

Requirements may differ across RISC-V boards and Linux distributions. Check the error message before modifying the configuration.

## 7. Updating

To rebuild the image after modifying the Dockerfile or other build-related files:

```bash
docker compose build --no-cache
docker compose up -d
```

The configuration directory is mounted from the host, so rebuilding the image does not itself remove files stored in `./homeassistant/config`.

However, configuration persistence is not a substitute for a backup. Back up important Home Assistant configuration data before making significant changes.

## 8. Notes

This project provides a starting point for running Home Assistant on RISC-V systems.

Successful image compilation does not necessarily guarantee that every Home Assistant integration or hardware device will work correctly. Compatibility depends on the target architecture, operating system, Python packages, native dependencies, and individual integration requirements.

Additional configuration may be necessary for specific hardware or deployment environments.
