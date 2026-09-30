# MeshGrid-Node

ESP32 firmware for **MeshGrid** mesh radios: phone ↔ BLE ↔ node ↔ LoRa (Ebyte **E22**) ↔ peer nodes.

This repository publishes **binary firmware releases** for flashing MeshGrid hardware. Source lives in the private development tree; release assets here are flash images only (no sketch source).

## What it does

- **MeshLink v2** over LoRa (UART to E22-900T22D-class modules)
- **Encrypted mesh**: shared network password → HKDF → BLE (AES-GCM) + LoRa (Ascon-128 AEAD)
- **BLE bridge** to the MeshGrid companion app (pairing + encrypted notify path)
- Link ACK / retry, RF dedup, optional multi-hop relay, chat ARQ for MSG/ALERT/shape traffic
- Optional **authorized-node roster** (reject unknown sender IDs)
- Optional GPS UART path when hardware is fitted

## Hardware

| Item | Notes |
|------|--------|
| MCU | ESP32 Dev Module or ESP32S (4 MB flash, 2 core) |
| Radio | Ebyte E22 (900 MHz class), UART + AUX |
| Antenna | Matched to the E22 band / region |
| USB | CP2102 (or similar) for flash / serial provision |

Typical ESP32 ↔ E22 wiring used by this firmware build: UART RX/TX, AUX, and mode pins as documented with each release. Confirm pinout on your PCB before flashing.

## Current release

See **[Releases](https://github.com/toygar/MeshGrid-Node/releases)** for the latest package.

| Field | Value |
|-------|--------|
| Build | **135** |
| Artifact | `MeshGrid_ESP32_BUILD135_release.zip` |
| Preferred image | `MeshGrid_ESP32_BUILD135_flash0x0.bin` @ address **0x0** |

Boot log should show:

```text
BLE ready BUILD 135 node=<id> LoRa=enc BLE=enc ...
```

## Flash (full image)

Requirements: [esptool](https://github.com/espressif/esptool), USB serial driver, board in download mode if your USB-UART bridge needs it.

```bash
# macOS / Linux example — replace PORT
esptool.py --chip esp32 --port /dev/cu.usbserial-XXXX --baud 921600 \
  write_flash -z 0x0 MeshGrid_ESP32_BUILD135_flash0x0.bin
```

```powershell
# Windows example — replace COMx
esptool.py --chip esp32 --port COM3 --baud 921600 `
  write_flash -z 0x0 MeshGrid_ESP32_BUILD135_flash0x0.bin
```

### App-only update

If bootloader/partitions are already correct and you only want to replace the application (NVS often survives):

```bash
esptool.py --chip esp32 --port PORT --baud 921600 \
  write_flash -z 0x10000 MeshGrid_ESP32_BUILD135_app_0x10000.bin
```

Split binaries (`bootloader.bin` @ `0x1000`, `partitions.bin` @ `0x8000`, `boot_app0.bin` @ `0xe000`, app @ `0x10000`) are included for advanced flashing. See `README_FLASH.txt` and `SHA256SUMS.txt` inside the zip.

**Flash layout (4 MB):** bootloader `0x1000` · partitions `0x8000` · otadata `0xe000` · app `0x10000`.

## First-time provisioning

Unprovisioned nodes stay offline from the mesh until a network password is written over USB serial. Use the same password on every node (and in the phone app) that should share one private mesh.

1. Flash firmware.
2. Open serial at **115200** baud (or use the project `provision.py` from your build tree).
3. Provision password (+ optional short BLE name / roster).
4. Pair the MeshGrid app with the node’s BLE name and PIN policy for your deployment.

**Important:** NVS (keys, name, roster) **survives** normal app uploads. To change the network password cleanly, erase flash (or wipe NVS), re-flash, then re-provision **all** nodes that should interoperate.

## Serial lab commands (debug builds)

When `MESH_DEBUG` is enabled in a given build, USB serial accepts commands such as:

| Command | Purpose |
|---------|---------|
| `stats` / `health` | Counters, radio sanity, heap |
| `txmsg LAB <text>` | Enqueue a chat MSG over LoRa (`text` ≤ 12 chars, must include a letter) |
| `roster` / `roster set id,id,…` | Show / set authorized node IDs |

Success for over-the-air chat on a peer is a decoded frame such as:

```text
[LORA_RX] type=0 from=<peer> | cs=LAB | <text>
```

Queue accept (`uartAccepted`) is **not** the same as peer delivery. Prefer `[LORA_RX]` on the receiving node (and `peerAcked` when link ACK is in use).

## Security notes

- Same password → same mesh. Different passwords → separate networks (cannot decrypt each other).
- Roster limits which **node IDs** may talk; it is not full per-node cryptographic identity.
- Do not publish provision passwords, pairing PINs, or production keys in issues or release notes.

## Companion app

Use the MeshGrid iOS/macOS companion (**MeshGrid-Commander** / MeshGrid app) for BLE pairing, chat, shapes, and GPS when those features are enabled on the node.

## License / support

Release binaries are provided for MeshGrid hardware owners. Open an issue on this repository for flash or release-asset problems. Include board type, esptool log, and the `BLE ready BUILD …` boot line (never paste network passwords).

