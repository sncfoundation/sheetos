<p align="center">
  <img src="https://sncfoundation.github.io/logos/sheetos.svg" width="96" alt="SheetOS logo">
</p>

<h1 align="center">SheetOS</h1>

<p align="center"><b>A minimal node OS that bootstraps Sheeternetes</b><br>
A <a href="https://sncfoundation.github.io">Sheet-Native Computing Foundation</a> project &#183; analog of <b>Talos Linux</b></p>

---

**Status:** 📝 Unsaved Draft &#183; this project is a proposal. Design notes and contributions are welcome.

## About

A minimal node OS that bootstraps Sheeternetes Part of the spreadsheet-native stack — the Sheet stays the source of truth, and it reconciles.

SheetOS is **API-only, immutable, and declarative** — Talos for spreadsheets. There is no shell to log into and no config to hand-edit on a node. You declare a **machineconfig** (one JSON file per node) and the `sheetstrap` CLI applies it. To change anything, you regenerate the config; you never mutate the node in place.

## Usage

`sheetstrap` is a single bash script. Requires `bash` and `jq`.

### 1. Generate a declarative machineconfig

Everything about a node is declared up front — its name, role, the apiserver (Apps Script web app) URL, the join token, and Docker settings — and written to `machineconfig.<node>.json`.

```bash
# control-plane node
./sheetstrap gen-config --name cp-1 --controlplane \
  --webapp https://script.google.com/macros/s/XXXX/exec \
  --token  my-super-secret

# worker node
./sheetstrap gen-config --name node-a --worker \
  --webapp https://script.google.com/macros/s/XXXX/exec \
  --token  my-super-secret
```

The resulting artifact is immutable — regenerate it, do not hand-edit it:

```json
{
  "apiVersion": "sheetos/v1alpha1",
  "kind": "MachineConfig",
  "machine": { "name": "cp-1", "role": "controlplane", "immutable": true, "install": { "disk": "none", "wipe": false } },
  "cluster": { "controlPlane": { "apiserverURL": "https://script.google.com/macros/s/XXXX/exec" }, "token": "my-super-secret" },
  "docker":  { "network": "sheeternetes", "namePrefix": "sk_", "cpuTotalMillicores": 2000, "memTotalMiB": 2048, "heartbeatSeconds": 10 }
}
```

### 2. Validate

Check that all required fields are present and the role is valid before you bring a node up:

```bash
./sheetstrap validate machineconfig.cp-1.json
```

### 3. Bootstrap the control plane

Reads the machineconfig and prints the ordered, declarative bootstrap sequence — create a blank Google Sheet (the cluster's disk), deploy the Apps Script apiserver, run `setup()`, and register the first node. It also writes a ready-to-source `.skctl.env` (`WEBAPP_URL` / `TOKEN`) for use with `skctl`.

```bash
./sheetstrap bootstrap machineconfig.cp-1.json
. ./.skctl.env && ./skctl get nodes
```

### 4. Join a worker

Configures a worker with no manual editing — it writes the worker's `.skctl.env` and an immutable worker machineconfig, both inheriting the declared apiserver URL and token. The worker holds no local config of its own; it reconciles from the Sheet.

```bash
./sheetstrap join https://script.google.com/macros/s/XXXX/exec my-super-secret --name node-a
WEBAPP_URL=... TOKEN=... NODE_NAME=node-a ./kubelet.sh
```

### Commands

| Command | Purpose |
| --- | --- |
| `gen-config --name <node> [--controlplane\|--worker] [--webapp URL] [--token TOK]` | Render a declarative machineconfig JSON. |
| `bootstrap machineconfig.<node>.json` | Print the ordered bootstrap steps for a control-plane node; write `.skctl.env`. |
| `join <webapp_url> <token> --name <node>` | Configure a worker: write its `.skctl.env` + worker machineconfig. |
| `validate machineconfig.<node>.json` | Validate required fields. |
| `version` | Print the sheetstrap / machineconfig apiVersion. |

## Get involved

- 📋 Tracking issue &amp; design: [sncfoundation/sheeternetes#26](https://github.com/sncfoundation/sheeternetes/issues/26)
- 🗺️ [SNCF Landscape](https://sncfoundation.github.io/landscape.html)
- 🧩 [All projects](https://sncfoundation.github.io/projects.html)
- ⚖️ [Governance &amp; how to contribute](https://github.com/sncfoundation/governance)
- 🎓 [Get certified (CSFE)](https://sncfoundation.github.io/certification.html)

## Status legend

Everything starts as an **Unsaved Draft**. It reconciles up the tiers from there — see the
[maturity model](https://sncfoundation.github.io/foundation.html#maturity).

---

<sub>Licensed under Apache-2.0. The SNCF does not recommend running production on a spreadsheet. If you do, please film it.</sub>
