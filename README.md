# LLMTune Worker Agent — Releases

This repository distributes the **LLMTune Worker Agent** for Windows: the onboarding
tool users run on their own machines to contribute hardware to the LLMTune network.

## What the agent is

The agent is a small, self-contained infrastructure program. It does exactly four things:

1. **Joins the NetBird mesh** — connects the machine to the LLMTune private network
   using a one-time setup key issued by the platform.
2. **Probes hardware** — inventories the GPU(s) (name, VRAM) and reports live GPU
   stats (utilization, VRAM in use, temperature) plus host metrics: CPU model, cores,
   utilization, RAM, disk, uptime, and OS.
3. **Generates a chassis signature** — a hardware fingerprint derived from the primary
   physical network adapter's MAC address, used to deduplicate multiple OS installs
   (e.g. two WSL distros) on one physical machine.
4. **Registers and heartbeats** — registers the machine with the LLMTune worker API,
   then posts a heartbeat every 60 seconds with fresh hardware stats.

## Models are platform-managed

> **This agent contains no model names, no model weights paths, and no model
> configuration of any kind.** The agent is pure infrastructure. Which model a
> worker serves is decided exclusively by platform administrators through the
> admin panel / model-assignment API, and is delivered to the agent in heartbeat
> responses. Releases from this repository will never carry model artifacts.

## Source code

The agent's source code is **proprietary** and lives in LLMTune's private
repositories. This public repository contains only:

- release binaries (`llmtune-worker-agent-<version>-win-x64.exe`)
- SHA-256 checksums (`checksums.txt`)
- this README and the release workflow

## Download & run

1. Grab the latest `llmtune-worker-agent-<version>-win-x64.exe` from
   [Releases](../../releases).
2. Verify it against `checksums.txt`:
   ```powershell
   Get-FileHash .\llmtune-worker-agent-0.1.0-win-x64.exe -Algorithm SHA256
   ```
3. Run with the values from your onboarding page (nothing is baked into the binary):
   ```powershell
   .\llmtune-worker-agent-0.1.0-win-x64.exe `
     -Api https://api.llmtune.example `
     -Token <your-worker-token> `
     -SetupKey <your-netbird-setup-key> `
     -KeyId <setup-key-id> `
     -NetbirdIp <assigned-mesh-ip> `
     -Name my-gpu-box
   ```
   All values can also be supplied via environment variables
   (`LLMTUNE_API_URL`, `LLMTUNE_WORKER_TOKEN`, `NETBIRD_SETUP_KEY`,
   `NETBIRD_KEY_ID`, `NETBIRD_IP`, `LLMTUNE_WORKER_NAME`).

4. The agent starts immediately, registers once, and keeps heartbeating.
   `nvidia-smi` must be on PATH for GPU stats. Log file:
   `C:\ProgramData\llmtune-worker-agent.log`.

## Security notes

- Setup keys and worker tokens are entered at runtime — never bundled into the binary.
- The setup key is burned on first registration.
- New workers register as `pending` and require admin approval before joining the network.

## Versioning

Releases are tagged `vX.Y.Z`. See [Releases](../../releases) for the changelog.
