# 📘 ACL-GPU-00 User Guide

> **Server IP:** `10.198.119.245`
> **Hardware:** 4x NVIDIA RTX 5090 | AMD EPYC 9254 | 256GB RAM
> **Location:** Hwlab X235

Welcome to the Hwlab High-Performance Computing Node.
**Please Note:** This server is a shared resource. **Users do not have `sudo` (root) privileges.** All software installations must be managed via Conda or within your home directory.

---

## 🛑 Step 1: First Login & Security (Mandatory)

### 1.1 Login via SSH

**First, you must login to DTU HPC** following the guide: [https://www.hpc.dtu.dk/?page_id=2501](https://www.hpc.dtu.dk/?page_id=2501)

From your terminal at DTU HPC node, run:

```bash
ssh username@10.198.119.245

```

* **Default Password:** `ChangeMeNow` (Or the one provided by admin)

### 1.2 Change Password Immediately!

For security reasons, you **must** change your password upon your first login.
Run the following command in the server terminal:

```bash
passwd

```

* Enter the current password.
* Enter your new strong password twice.
* *(Note: Characters will not appear on screen while typing)*.

---

## 🔑 Step 2: Configure SSH Keys (Recommended)

Setup SSH keys to log in without typing your password every time.

### 2.1 Generate Key Pair (On YOUR Local Computer)

*Run this on your **local machine**, NOT the server:*

```bash
ssh-keygen -t ed25519 -C "your_email@example.com"
# Press Enter to accept defaults

```

### 2.2 Copy Key to Server

*Run this on your **local machine**:*

```bash
ssh-copy-id username@10.198.119.245

```

* Enter your new server password one last time.
* **Success:** You can now log in without a password.

---

## 🐙 Step 3: Github Configuration

To clone repositories directly to the server, add the server's SSH key to your GitHub account.

### 3.1 Generate Key (On Server)

*Run this on the **server**:*

```bash
ssh-keygen -t ed25519 -C "hwlab_server_key"
# Press Enter to accept defaults

```

### 3.2 Get the Public Key

```bash
cat ~/.ssh/id_ed25519.pub

```

* **Copy** the output starting with `ssh-ed25519`.

### 3.3 Add to Github

1. Go to [Github SSH Settings](https://github.com/settings/keys).
2. Click **New SSH key**.
3. **Title:** `Hwlab Server`.
4. **Key:** Paste the copied key.
5. **Test:** `ssh -T git@github.com` (You should see a welcome message).

---

## 🐍 Step 4: Python Environment (Conda)

Since you do not have root access, **Miniconda** is the only way to manage dependencies.

### 4.1 Install Miniconda

```bash
mkdir -p ~/miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm -rf ~/miniconda3/miniconda.sh

# Initialize Shell
~/miniconda3/bin/conda init bash
source ~/.bashrc

```

### 4.2 Create Your Environment

Never run experiments in the `base` environment.

```bash
# Create a new environment named 'lab_env'
conda create -n lab_env python=3.10

# Activate it
conda activate lab_env

# Install Pytorch (Example)
pip install torch torchvision torchaudio

```

---

## 🖥️ Step 5: Process Management & GUI Ban

### ⚠️ STRICT NO-GUI POLICY

**DO NOT** install any Desktop Environments (Gnome, XFCE) or Remote Desktop Tools (TeamViewer, VNC, ToDesk).

* **Reason:** GUIs consume valuable VRAM/RAM and cause driver instability.
* **Penalty:** Users violating this rule will have their accounts suspended.

### ✅ Use Tmux

Use **Tmux** to keep your training running even if you disconnect.
Since you don't have sudo, install tmux via Conda if it's not available:

**Install Tmux (User Level):**

```bash
# Ensure you are in your conda base environment or lab_env
conda install -c conda-forge tmux

```

**Tmux Cheatsheet:**

* `tmux new -s experiment1`: Create a new session named "experiment1".
* **Detach (Keep running):** Press `Ctrl + B`, release, then press `D`.
* `tmux attach -t experiment1`: Re-enter the session.
* `tmux ls`: List all active sessions.

---

## 📊 Step 6: Monitoring Tools

Always check resource availability before running experiments.

### 6.1 Install Tools (No Sudo Required)

We use Conda and Pip to install monitoring tools into your user space.

```bash
# Install CPU/Memory monitoring tools
conda install -c conda-forge htop btop

# Install GPU monitoring (Highly Recommended)
pip install nvitop

```

### 6.2 How to Check

* **`nvitop`:** Real-time GPU usage. **MANDATORY: Check this before running `python train.py`!**
* Run with: `nvitop -m` (Monitor mode)


* **`btop`:** CPU and RAM usage visualization.
* Run with: `btop`



---

## 📂 Appendix: Storage & File Transfer

### Storage Rules

* **`/home/username/`**: For **Code** and **Scripts** only.
* **Large Datasets**: **DO NOT** extract large datasets (e.g., ImageNet) to your home directory.
* Please check the **`/archive`** or **`/work`** directory (or ask the admin for the path).


* **Temporary Files**: Please clean up after experiments.

### File Transfer (SCP)

To transfer files from your **local computer** to the server:

```bash
# Upload a file
scp local_file.zip username@10.198.119.245:~/

# Upload a folder
scp -r local_folder username@10.198.119.245:~/

```

