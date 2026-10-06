# MeshGrid-Node

Binary firmware releases for **MeshGrid** mesh nodes.

Phone and companion apps talk to a node over Bluetooth; the node carries MeshGrid traffic across the mesh to peer nodes. This repository publishes **flash images only** (no firmware source). Product information: [meshgrid.org](https://meshgrid.org).

## What it does

- Private mesh networking for MeshGrid deployments
- Encrypted link between the phone, the node, and peer nodes on the same network
- Delivery feedback for chat and related traffic (sent / acknowledged / failed, when supported by the companion app)
- Optional authorized-node membership (roster) so only known nodes participate
- Optional location beacons when the product configuration enables them

## Current release

Download the latest package from **[Releases](https://github.com/toygar/MeshGrid-Node/releases)**.

Each zip includes:

- Preferred full flash image (`*_flash0x0.bin` at address `0x0`)
- Optional app-only image (`*_app_0x10000.bin` at `0x10000`)
- `README_FLASH.txt`, `RELEASE_NOTES.txt`, and `SHA256SUMS.txt`

After flash, confirm the boot log shows the **BUILD** number that matches the release you installed.

## Flash (full image)

Requirements: [esptool](https://github.com/espressif/esptool) and a USB serial connection to the node.

```bash
# macOS / Linux — replace PORT and the bin filename from the release zip
esptool.py --chip esp32 --port /dev/cu.usbserial-XXXX --baud 921600 \
  write_flash -z 0x0 MeshGrid_ESP32_BUILDXXX_flash0x0.bin
```

```powershell
# Windows — replace COMx and the bin filename from the release zip
esptool.py --chip esp32 --port COM3 --baud 921600 `
  write_flash -z 0x0 MeshGrid_ESP32_BUILDXXX_flash0x0.bin
```

### App-only update

If you only need to replace the application image (settings often survive):

```bash
esptool.py --chip esp32 --port PORT --baud 921600 \
  write_flash -z 0x10000 MeshGrid_ESP32_BUILDXXX_app_0x10000.bin
```

Flash layout and checksums are documented inside each release zip. Use the filenames and BUILD number from that release—not outdated examples.

## First-time setup

Unprovisioned nodes do not join a private mesh until the network is configured.

1. Flash the release image.
2. Provision with **MeshGrid Commander** (or your approved provisioning flow) using the same network values on every node that should interoperate.
3. Pair the MeshGrid companion app to the node.

Changing the network password for a deployment typically requires wiping stored settings and re-provisioning **all** nodes that should share that mesh. Do not publish passwords, pairing PINs, or production keys.

## Dual-radio vs single-radio images

Some releases target **dual-radio** hardware variants; others target **single-radio** variants. Install only the image line that matches your product. See the release notes in each zip.

## Companion software

- Product site: [meshgrid.org](https://meshgrid.org)
- Use the MeshGrid companion app and MeshGrid Commander for pairing, chat, maps/shapes, and provisioning as supported by your release.

## Support

Open an issue for flash or release-asset problems. Include the release tag, esptool log, and the boot line that shows `BUILD …` (never paste network passwords or PINs).
