# 📘 ACL-GPU-00 User Guide

> **Server IP:** `10.198.119.245`
> **Hardware:** 4x NVIDIA RTX 5090 | AMD EPYC 9254 | 256GB RAM
> **Location:** Hwlab X235

Welcome to the Hwlab High-Performance Computing Node.
**Please Note:** This server is a shared resource. **Users do not have `sudo` (root) privileges.** All software installations must be managed via Conda or within your home directory.

---

# DTU HPC jump host + internal server (10.198.119.245) tunnel (VS Code Remote-SSH)

Goal: use **VS Code** to edit/run code on `10.198.119.245` (via Remote-SSH). Because you must first log in to DTU HPC (login nodes) as a jump host, we use a **local SSH tunnel**: forward remote `10.198.119.245:22` to your local `localhost:2222`, so VS Code only needs to connect to `localhost:2222` (the `lab245-tunnel` host below).

> Reboot / network changes / sleep will break the tunnel. Start it again when that happens.

---

## 0) Parameters

- DTU username: `username`
- Jump hosts (pick any): `login.hpc.dtu.dk` / `login1.hpc.dtu.dk` / `login2.hpc.dtu.dk`
- Target internal server: `10.198.119.245`
- Local tunnel port: `2222`

---

## 0.1 Where to start (important)

Many users **already** have passwordless SSH set up (they already have a private key such as `~/.ssh/id_rsa` or `~/.ssh/id_ed25519`). In that case, you do **not** need to generate/install new keys.

### A. Test: can you already SSH into HPC without typing your HPC password?

On your laptop/desktop, run (replace `username`):

```bash
ssh username@login1.hpc.dtu.dk
```

- If you can log in **without typing your HPC account password** (you might be asked for your local key passphrase, or DTU interactive/MFA prompts—those are fine), then:
  - **Skip Section 2 (installing your public key on HPC)**
  - In Section 1, you can also skip “generate HPC key” (unless you want a dedicated key).
- If you see `Permission denied` / it keeps asking for your HPC password: follow Sections 1–2.

### B. Test: can you already SSH into 10.198.119.245 (via the HPC jump host) without typing the target password?

On your laptop/desktop, run:

```bash
ssh -J username@login1.hpc.dtu.dk username@10.198.119.245
```

- If you can log in **without typing the target account password**:
  - **Skip Section 3 (installing your public key on 10.198.119.245)**
- Otherwise: follow Section 3.

---

## 1) Prepare SSH keys (you may reuse one key, or use two separate keys)

> You do **not** have to use different keys for HPC and for `10.198.119.245`.
>
> - **Option 1: reuse one key (simplest)** — keep using your existing `id_rsa` / `id_ed25519` for both HPC and the target host.
> - **Option 2: separate keys (recommended)** — one key dedicated to the HPC jump host, and another dedicated to the target host (clearer separation).

Check what you already have:

```bash
ls -al ~/.ssh
```

If you see `id_rsa` or `id_ed25519` (and the matching `.pub`), you can usually reuse it.

### 1.1 (Optional) Generate a dedicated “jump host key” (for HPC)

If you already have passwordless SSH to HPC, you can skip this.

```bash
mkdir -p ~/.ssh
ssh-keygen -t ed25519 -f ~/.ssh/dtu_hpc
```

Files:
- Private key: `~/.ssh/dtu_hpc`
- Public key: `~/.ssh/dtu_hpc.pub`

### 1.2 (Optional) Generate a dedicated “target host key” (for 10.198.119.245)

If you want a separate key for the target host:

```bash
ssh-keygen -t ed25519 -f ~/.ssh/lab245_key -C "username@10.198.119.245"
```

Files:
- Private key: `~/.ssh/lab245_key`
- Public key: `~/.ssh/lab245_key.pub`

> If you prefer reusing one key for everything, you can skip this section and later point `IdentityFile` to your existing `~/.ssh/id_rsa` or `~/.ssh/id_ed25519`.

---

## 2) Install your public key on DTU HPC (for passwordless jump host login)

> If test 0.1A already works for you: **skip this section**.

### 2.1 Pick which public key to install on HPC

Typical choices:

- Existing key: `~/.ssh/id_rsa.pub` or `~/.ssh/id_ed25519.pub`
- New dedicated jump key: `~/.ssh/dtu_hpc.pub`

We’ll refer to it as `PUBKEY` below.

### 2.2 Recommended: ssh-copy-id (usually available on macOS/Linux)

```bash
PUBKEY=~/.ssh/dtu_hpc.pub  # <- change to your .pub
ssh-copy-id -i "$PUBKEY" username@login1.hpc.dtu.dk
```

### 2.3 Manual method (works everywhere)

1) Log in to HPC:

```bash
ssh username@login1.hpc.dtu.dk
```

2) On the HPC login node, run:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat >> ~/.ssh/authorized_keys
# On your local machine, run: cat <your-public-key>.pub
# Copy the full line (ssh-ed25519 / ssh-rsa ...), paste it here, press Enter
# Press Ctrl-D to finish
chmod 600 ~/.ssh/authorized_keys
```

---

## 3) Install your public key on 10.198.119.245 (for passwordless target login)

> If test 0.1B already works for you: **skip this section**.

### 3.1 Log in to 10.198.119.245

Pick one:

**Option A (recommended: from your local machine via jump host):**

```bash
ssh -J username@login1.hpc.dtu.dk username@10.198.119.245
```

**Option B (if you are already inside an HPC login node):**

```bash
ssh username@10.198.119.245
```

### 3.2 Add your public key to the target host

On `10.198.119.245`, run:

```bash
mkdir -p ~/.ssh
chmod 700 ~/.ssh
cat >> ~/.ssh/authorized_keys
# On your local machine, run: cat <your-public-key>.pub
# Copy the full line, paste it here, press Enter
# Press Ctrl-D to finish
chmod 600 ~/.ssh/authorized_keys
```

> You may install **the same** public key on both HPC and the target host, or different ones.

---

## 4) Configure your local ~/.ssh/config (with login/login1/login2 fallback)

Edit `~/.ssh/config` and add (replace `username` and `IdentityFile` paths to match your setup):

```sshconfig
### --- DTU HPC jump hosts ---
Host dtu-login dtu-login1 dtu-login2
  User username
  # IdentityFile can be your existing id_rsa/id_ed25519,
  # or the dedicated jump key ~/.ssh/dtu_hpc
  IdentityFile ~/.ssh/dtu_hpc
  IdentitiesOnly yes
  ServerAliveInterval 30
  ServerAliveCountMax 3
  ConnectTimeout 8

Host dtu-login
  HostName login.hpc.dtu.dk

Host dtu-login1
  HostName login1.hpc.dtu.dk

Host dtu-login2
  HostName login2.hpc.dtu.dk

### --- target behind DTU (10.198.119.245) ---
Host lab245
  HostName 10.198.119.245
  User username
  # You can reuse the same key as above,
  # or use a dedicated key ~/.ssh/lab245_key
  IdentityFile ~/.ssh/lab245_key
  IdentitiesOnly yes
  ProxyCommand sh -c 'ssh -o ConnectTimeout=5 -W %h:%p dtu-login || ssh -o ConnectTimeout=5 -W %h:%p dtu-login1 || ssh -o ConnectTimeout=5 -W %h:%p dtu-login2'

### --- local tunnel endpoint (for VS Code Remote-SSH / testing) ---
Host lab245-tunnel
  HostName localhost
  Port 2222
  User username
  # Usually use the same key as Host lab245
  IdentityFile ~/.ssh/lab245_key
  IdentitiesOnly yes
```

> If you use ssh-agent and don’t want to hardcode `IdentityFile`, you can remove the `IdentityFile ...` lines and let OpenSSH pick the key.

Validate:

```bash
ssh -v dtu-login1
ssh -v lab245
```

---

## 5) Create the local tunnel

### 5.1 Start the tunnel (background, with login/login1/login2 fallback)

```bash
ssh -fN -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
  -L 2222:10.198.119.245:22 dtu-login \
|| ssh -fN -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
  -L 2222:10.198.119.245:22 dtu-login1 \
|| ssh -fN -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
  -L 2222:10.198.119.245:22 dtu-login2
```

> `-fN` will move SSH to the background after authentication. Password/MFA input won’t echo—this is normal.

### 5.2 Check the tunnel

Check if port 2222 is listening:

```bash
lsof -iTCP:2222 -sTCP:LISTEN -n -P
```

Probe the SSH connection:

```bash
ssh lab245-tunnel 'echo TUNNEL_OK && hostname'
```

### 5.3 Stop the tunnel

```bash
lsof -ti tcp:2222 | xargs kill
```

---

## 6) Optional: autossh (auto-reconnect)

Install:

```bash
brew install autossh
```

Start (pick a stable login host, e.g. `dtu-login1`):

```bash
autossh -M 0 -fN \
  -o ExitOnForwardFailure=yes \
  -o ServerAliveInterval=30 -o ServerAliveCountMax=3 \
  -L 2222:10.198.119.245:22 \
  dtu-login1
```

Stop:

```bash
pkill autossh
lsof -ti tcp:2222 | xargs kill
```

---

## 7) VS Code setup (Remote-SSH to localhost:2222)

> If `ssh lab245-tunnel` (or `ssh lab245`) works in your terminal, VS Code should work too.

### 7.1 Install VS Code extensions

Install these extensions:

- **Remote - SSH** (`ms-vscode-remote.remote-ssh`)
- **Python** (`ms-python.python`)
- (Optional) **Jupyter** (`ms-toolsai.jupyter`) if you run notebooks remotely

### 7.2 Method A: connect through the local tunnel (recommended)

1) Make sure the tunnel is running (Section 5/6) and probe it:

```bash
ssh lab245-tunnel 'echo TUNNEL_OK && hostname'
```

2) In VS Code, open Command Palette:

- macOS: `⌘ + Shift + P`
- Windows/Linux: `Ctrl + Shift + P`

Run:

- `Remote-SSH: Connect to Host...`

Select **`lab245-tunnel`** (from your `~/.ssh/config`).

3) On first connect you may be asked the remote platform—choose **Linux**.

4) Once connected, the bottom-left should show `>< SSH: lab245-tunnel`.

### 7.3 Method B: connect directly via the jump host (optional, no local port 2222)

If you don’t want to maintain a local tunnel, VS Code Remote-SSH can also use the `ProxyCommand` of `lab245` from Section 4:

1) Command Palette → `Remote-SSH: Connect to Host...`

2) Select **`lab245`**

### 7.4 Open a remote folder and run commands

After connecting:

- `File → Open Folder...` and choose a remote directory (e.g. `/home/username/`)
- `Terminal → New Terminal` runs on the remote side

Quick sanity checks:

```bash
hostname
whoami
pwd
python3 -V
```

### 7.5 Select the remote Python interpreter

1) Command Palette in the remote VS Code window:

- `Python: Select Interpreter`

2) Choose the interpreter you want (system `python3`, conda env, venv, ...).

Example: create a venv on the remote host:

```bash
python3 -m venv ~/venvs/lab245
source ~/venvs/lab245/bin/activate
python -m pip install -U pip
# pip install -r requirements.txt
```

Then select:

- `~/venvs/lab245/bin/python`

### 7.6 Common issues

- **Can’t connect / stuck**: first confirm `ssh lab245-tunnel` (or `ssh lab245`) works in a terminal; fix SSH/tunnel first (Sections 1–6).
- **`Permission denied (publickey)`**: check your `IdentityFile` paths and whether you installed the correct public key into `~/.ssh/authorized_keys` on the right machine.
- **VS Code Server install fails (quota/permission)**: clean `~/.vscode-server` on the remote host and check your home quota.

---

## 8) ⚠️ Warning: DTU/HPC may refuse some networks (Connection refused)

Symptoms:

- `nc -vz github.com 22` works
- but `nc -vz login1.hpc.dtu.dk 22` returns `Connection refused`

This usually means DTU/HPC blocks your current public IP/network. SSH/tunnels won’t work.

Suggested fixes:
1) Connect to **DTU VPN**
2) Or switch networks (mobile hotspot / other Wi‑Fi)
