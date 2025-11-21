# Multi-OS Developer Environment Setup

This Ansible playbook automates the provisioning of a unified developer environment across **macOS, Windows, Linux, and WSL**. It ensures a consistent set of tools and configurations regardless of the underlying operating system.

## 🚀 Supported Platforms

* **Linux:** Ubuntu, Debian, Fedora, CentOS
* **macOS:** Intel & Apple Silicon
* **Windows:** Windows 10/11 (via WinRM)
* **WSL 2:** Windows Subsystem for Linux (Ubuntu/Debian distros)

## 🛠 Roles Included

This playbook orchestrates the following roles:

### 1. `general-dev-tools`
Installs core system dependencies and CLI essentials specific to the OS package manager (Apt, Dnf, Brew, Scoop/Choco).
* **Includes:** Git, Curl, Wget, Python, Node.js, Build Essentials.

### 2. `k9s`
Installs the **K9s** Kubernetes CLI UI.
* Detects architecture (AMD64/ARM64) automatically.
* Adds binary to the system path.

### 3. `vscode`
Installs **Visual Studio Code** and synchronizes extensions.
* Installs the binary (Code on Mac/Linux, VS Code on Windows).
* Bulk installs a defined list of extensions (e.g., Python, Ansible, Docker).

### 4. `customised-terminal`
Configures a modern shell experience.
* **Linux/Mac/WSL:** Sets up **Zsh**, **Oh-My-Zsh**, and installs **Nerd Fonts**.
* **Windows:** Sets up **PowerShell Core**, **Oh-My-Posh**, and custom Terminal profiles.

---

## 📋 Prerequisites

### Control Node (Your Machine)
* **Ansible**: `pip install ansible`
* **Python**: 3.8+

### Ansible Collections
Install the required collections to handle cross-platform logic:

```bash
ansible-galaxy collection install community.general
ansible-galaxy collection install community.windows
ansible-galaxy collection install community.crypto
```
## Target Machine Configuration

### Windows Targets
Windows machines must have WinRM enabled to allow Ansible to connect. Open PowerShell as Administrator on the target Windows machine and run:

```bash
# Download the configuration script
$url = "[https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1](https://raw.githubusercontent.com/ansible/ansible/devel/examples/scripts/ConfigureRemotingForAnsible.ps1)"
$file = "$env:temp\ConfigureRemotingForAnsible.ps1"
(New-Object -TypeName System.Net.WebClient).DownloadFile($url, $file)

# Run the script
powershell.exe -ExecutionPolicy ByPass -File $file
```

### macOS Targets
Ensure "Remote Login" (SSH) is enabled in System Settings > General > Sharing.

### WSL Targets
If running the playbook inside the WSL instance you want to configure, no SSH setup is required (use local connection).

##  Inventory Configuration

Create a file named inventory.ini. It is crucial to set the correct connection variables for Windows vs. Linux/Mac.

```bash
# --- Linux & MacOS (Standard SSH) ---
[unix_hosts]
192.168.1.10 ansible_user=ubuntu
192.168.1.11 ansible_user=admin ansible_python_interpreter=/usr/bin/python3

# --- Windows (WinRM) ---
[windows_hosts]
192.168.1.15 ansible_user=Administrator ansible_password=YourPassword ansible_connection=winrm ansible_winrm_server_cert_validation=ignore

# --- WSL (Local Execution) ---
# Use this if running ansible FROM the wsl instance TO itself
[wsl_hosts]
localhost ansible_connection=local
```
## Usage

### Run on all hosts
```bash
ansible-playbook -i inventory.ini setup.yml --ask-become-pass
(Note: --ask-become-pass prompts for the sudo password required for Linux/Mac installations)
```
### Run on specific OS group
To only update Windows machines:

```bash
ansible-playbook -i inventory.ini setup.yml --limit windows_hosts
```
### Run locally on WSL
```bash
ansible-playbook -i inventory.ini setup.yml --limit wsl_hosts --ask-become-pass
```
## Customization
You can customize installed tools in group_vars/all.yml:

```yaml

# group_vars/all.yml

# Version pinning
k9s_version: "v0.32.4"

# VS Code Extensions to install
vscode_extensions:
  - ms-python.python
  - redhat.ansible
  - eamodio.gitlens

# Git Global Config
git_user_name: "John Doe"
git_user_email: "john@example.com"
```

## Common issues

### Note on Running Ansible Inside WSL

If you are running Ansible inside WSL, avoid placing your Ansible project inside Windows-mounted paths such as:

```
/mnt/c/Users/...
```

WSL mounts the Windows filesystem with permissive default permissions (`0777`).  
From Linux's perspective, this makes the directory **world-writable**, and Ansible will *not* trust or load any `ansible.cfg` file located in an insecure path.

Because of this, settings such as custom `roles_path`, inventory paths, callbacks, and other configuration options may be ignored, and you may see warnings like:


#### Recommended Fix

Move your Ansible files into the WSL Linux filesystem, for example:

```
/home/<user>/ansible/
```

This directory uses real POSIX permissions, is not world-writable, and Ansible will correctly load your `ansible.cfg`.


