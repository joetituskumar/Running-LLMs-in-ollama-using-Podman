# Ollama via podman

```elixir
Mix.install([
  {:pythonx, "~> 0.4.2"},
  {:kino_pythonx, "~> 0.1.0"}
])
```

```pyproject.toml
[project]
name = "project"
version = "0.0.0"
requires-python = "==3.13.*"
dependencies = ["ollama"]
```

## Ollama via podman

### Run Ollama via Podman

A single, practical guide to installing **Podman** on multiple operating systems, setting system requirements, running the **Ollama** container, pulling models, and using Ollama from **Python (Livebook/Jupyter)**.

---

### Table of contents

1. Quick checklist (what you'll need)
2. System requirements
3. Install Podman (macOS, Ubuntu/Debian, Fedora, Windows/WSL2, Podman Desktop)
4. Initialize Podman (macOS / macOS VM notes)
5. Common Podman settings (memory, ports, volumes)
6. Run Ollama container (create, start, restart, remove)
7. Pull and run models inside the container
8. Use Ollama from Python (Livebook / Jupyter)
9. Troubleshooting & common errors
10. Tips: models to use for low‑RAM machines
11. Security & cleanup

---

#### 1) Quick checklist (before starting)

* Podman installed and `podman` CLI available.
* Virtualization enabled on your OS (macOS uses a lightweight VM for Podman).
* At least **8 GB RAM recommended** on host for comfortable use; 4 GB minimum for small models.
* Disk: plan for model storage — allocate **10–30+ GB** depending on models you pull.
* Python and Livebook/Jupyter installed to run examples (optional).

---

#### 2) System requirements

* **CPU:** Any modern x86_64 CPU works (Intel/AMD). Apple Silicon needs images built for arm64 or run with proper emulation — prefer native arm images when on Apple Silicon.
* **RAM:**
  * Minimum: 4 GB (for very small models).
  * Recommended: 8–16 GB (for comfortable use with 1–7B models).
  * For 13B+ models you will need a lot more RAM (often >16–24 GB) or a remote machine.
* **Disk:** 10–50 GB free depending on how many models you will store.
* **Network:** Outbound internet to pull images/models; optionally closed network for runtime.

---

#### 3) Install Podman

> Use the section that matches your OS.

#### macOS (Intel or Apple Silicon)

#### Using Homebrew (recommended)

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"  # only if brew missing
brew update
brew install podman
```

Notes:

* Podman on macOS runs inside a lightweight VM (`podman machine`).
* After install, run `podman machine init` and `podman machine start`.

#### Ubuntu / Debian

```bash
# Debian/Ubuntu example
. /etc/os-release
sudo apt update
# Add Podman repo (recommended) or use distro package
. /etc/os-release && sudo sh -c "echo 'deb http://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$ID/ /' > /etc/apt/sources.list.d/devel:kubic:libcontainers:stable.list"
curl -L "https://download.opensuse.org/repositories/devel:/kubic:/libcontainers:/stable/$ID/Release.key" | sudo apt-key add -
sudo apt update
sudo apt install -y podman
```

Or simply: `sudo apt install podman` (may be older version).

#### Fedora / RedHat

```bash
sudo dnf -y install podman
```

#### Windows (recommended via WSL2 or Podman Desktop)

**Option A — Use WSL2 (Ubuntu inside WSL):**

1. Install WSL2 and enable virtualization. Install Ubuntu from Microsoft Store.
2. Inside WSL: follow the Ubuntu instructions above to `apt install podman`.
3. Use `podman` from WSL shell — you can forward ports to Windows localhost.

**Option B — Podman Desktop for Windows**

* Download Podman Desktop from the official site and install; it includes a GUI and manages a VM.

---

### 4) Initialize Podman (macOS specific)

Podman on macOS uses `podman machine` to create the VM that runs containers.

```bash
# create (optional if created by init)
podman machine init
# start the VM
podman machine start
# check status
podman machine inspect
podman info
```

Adjust memory and CPUs (macOS example):

```bash
# set memory to 8GB (8192 MB)
podman machine set --memory 8192
# set CPUs
podman machine set --cpus 4
podman machine stop
podman machine start
```

If you use Windows WSL2, configure WSL memory/CPU via `.wslconfig` or Podman Desktop settings.

---

### 5) Common Podman settings (memory, ports, volumes)

* **Memory:** `podman machine set --memory <MB>`
* **CPUs:** `podman machine set --cpus <N>`
* **Publish a port:** `-p HOSTPORT:CONTAINERPORT` in `podman run`
* **Persist models and data:** use a volume, e.g. `-v ollama:/root/.ollama`

Volumes are persistent across container recreate and are recommended for models.

---

### 6) Run Ollama container (create, start, restart, remove)

##### Create & run (first time)

```bash
podman run -d \
  --name ollama \
  -p 11434:11434 \
  -v ollama:/root/.ollama \
  docker.io/ollama/ollama:latest
```

* `-d` runs in background.
* `--name ollama` gives the container a friendly name.
* `-p 11434:11434` exposes Ollama API on host port 11434.
* `-v ollama:/root/.ollama ` stores models and data persistently.

##### Start an existing (stopped) container

```bash
podman start ollama
```

##### Restart

```bash
podman restart ollama
```

##### Remove & recreate

If container is broken and you want to recreate:

```bash
podman rm ollama
# then re-run podman run ... (see create above)
```

##### Check running containers

```bash
podman ps       # only running
podman ps -a    # include stopped containers
podman logs ollama  # view logs
```

---

### 7) Pull and run models inside the container

Use `podman exec` to run the Ollama CLI inside the container.

### Pull a model

```bash
podman exec -it ollama ollama pull llama3.2:1b
```

### List available models (inside container)

```bash
podman exec -it ollama ollama ls
# or
podman exec -it ollama ollama list
```

### Run a model (interactive)

```bash
podman exec -it ollama ollama run llama3.2:1b
```

### Example: run a single prompt

```bash
podman exec -it ollama ollama run llama3.2:1b --prompt "Write a haiku about tea."
```

---

### 8) Use Ollama from Python (Livebook / Jupyter)

### Install Python client

Inside your Livebook/Python cell or terminal:

```bash
pip install ollama
```

### Minimal Python example (Livebook cell)

<!-- livebook:{"force_markdown":true} -->

```python
from ollama import Client
client = Client(host="http://localhost:11434")

res = client.chat(
    model="llama3.2:1b",
    messages=[{"role": "user", "content": "Write a short poem about Bamberg."}]
)

print(res['message']['content'])
```

### Notes for Livebook / Jupyter connectivity

* If you started Podman with `-p 11434:11434` on macOS Podman machine, `localhost:11434` should be reachable from host processes.
* If Livebook runs in a separate VM/container (or remote), change `host` to the podman VM IP or `127.0.0.1` depending on networking.
* If you have connection issues, try `podman machine set --publish 11434:11434` and then restart the machine.

---

### 9) Troubleshooting & common errors

### 1) `model requires more system memory (X GiB) than is available (Y GiB)`

Solution:

* Increase Podman VM memory: `podman machine set --memory 8192; podman machine stop; podman machine start`.
* Use a smaller model if you can’t allocate more memory.

### 2) `connection refused` from Python / Livebook

* Ensure container is running: `podman ps`.
* Ensure port is published: `podman ps` should show `0.0.0.0:11434->11434/tcp` or similar.
* If using Podman VM, ensure `podman machine` published port: `podman machine set --publish 11434:11434` then restart.
* If Livebook runs in another container/VM, use the Podman VM IP or host networking.

### 3) `container name already in use` when creating

* List all containers: `podman ps -a`.
* Start existing: `podman start ollama`.
* Or remove it if you want a fresh one: `podman rm ollama` then recreate.

### 4) Model download stalls or fails

* Check container logs: `podman logs -f ollama`.
* Ensure the VM/container has enough disk space.

### 5) Networking issues on macOS with Podman

* Use `podman machine set --publish 11434:11434` and restart the machine.
* Or run container with host networking (not always available on macOS):
  * `podman run -d --name ollama --network host -v ollama:/root/.ollama ollama/ollama`
  * Note: `--network host` behaves differently across platforms.

---

### 10) Tips: models to use on lower‑RAM machines

* Tiny/smaller models (best for 4 GB VM): `llama3.2:1b`, `qwen2.5:0.5b` (if available), or other 0.5–1B parameter models.
* Medium (best for 8 GB+): 2–7B models (some quantized variants may fit in 8GB).
* Avoid 13B+ on machines with <16GB RAM unless you use swap or remote inference.

---

### 11) Security & cleanup

* To permanently remove Ollama and models:

```bash
podman rm -f ollama
podman volume rm ollama
```

* Keep model data on a volume if you want persistence across container recreation.
* Limit network exposure: do not bind container ports to public IPs unless you secure the API.

---

### Appendix — Useful commands reference

```bash
# Podman machine
podman machine init
podman machine start
podman machine stop
podman machine inspect
podman machine set --memory 8192 --cpus 4 --publish 11434:11434

# Containers
podman run -d --name ollama -p 11434:11434 -v ollama:/root/.ollama ollama/ollama:latest
podman ps
podman ps -a
podman start ollama
podman restart ollama
podman rm ollama
podman logs -f ollama

# Exec into container
podman exec -it ollama /bin/sh    # or bash if available
podman exec -it ollama ollama ls
podman exec -it ollama ollama pull llama3.2:1b
podman exec -it ollama ollama run llama3.2:1b --prompt "Hello"
```

---

```python
from ollama import Client

# Ollama server (inside your Podman container)
client = Client(host='http://localhost:11434')

response = client.chat(
    model='llama3.2:1b',   # use a small model for Intel Mac
    messages=[
        {"role": "user", "content": "tell me about bamberg"}
    ]
)

response['message']['content']

```
