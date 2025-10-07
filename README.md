# Ubuntu Desktop Autoinstaller (24.04 LTS)

This project provides a fully automated, hands-free installation for **Ubuntu Desktop 24.04 LTS (Noble Numbat)** using the `autoinstall.yaml` mechanism. Starting with Ubuntu 24.04.1, the Desktop installer fully supports the same autoinstall functionality as Ubuntu Server.

This README includes instructions to prepare your bootable drive, configure your autoinstall file, and run the installation.

---

## Features

* Fully automated Desktop installation.
* Pre-configured sudo user.
* SSH server installation for Ansible/remote management.
* Automatic locale, timezone, keyboard, and network configuration.
* Post-installation commands and error handling.

---

## Prerequisites

* Ubuntu Desktop 24.04.1 ISO or newer.
* USB drive or second partition for the autoinstall file.
* SSH key pair (for passwordless access to the installed system).
* WSL (Windows Subsystem for Linux) or Linux host for building the bootable ISO.

---

## Step 1: Generate a Password Hash

Ubuntu autoinstall requires a hashed password for the default user.

```bash
openssl passwd -6
```

Copy the resulting hash into your `autoinstall.yaml` under `password`.

---

## Step 2: Prepare SSH Keys

Ensure your personal computer’s public SSH key is available. Add it to the `authorized-keys` section in the `autoinstall.yaml`:

```yaml
ssh:
  install-server: True
  authorized-keys:
    - ssh-ed25519 AAAAC3Nzac user@host
  allow-pw: false
```

For detailed guidance on generating SSH keys: [SSH Key Generation](https://www.ssh.com/academy/ssh/keygen)

---

## Step 3: Create a Bootable Autoinstall Drive (Using WSL)

1. Create a directory for your autoinstall files:

```bash
mkdir -p ~/autoinstall
```

2. Copy the ISO contents:

```bash
cp /mnt/c/Path/To/UbuntuISO.iso ~/autoinstall
```

3. Install ISO generation tools:

```bash
sudo apt update && sudo apt install -y genisoimage
```

4. Generate a bootable ISO:

```bash
genisoimage -o ~/autoinstall.iso -V AUTOINSTALL -r -J ~/autoinstall
```

5. Copy it back to your Windows desktop:

```bash
cp ~/autoinstall.iso /mnt/c/(Where ever you want)
```

---

## Step 4: Preinstallation Phase

During installation:

* Skip any prompts until the autoinstall option appears.
* To locate your USB or attached autoinstall drive:

```bash
lsblk
sudo su
mkdir -p /media/autoinstall
mount /dev/sdb /media/autoinstall
ls /media/autoinstall
# Should show: autoinstall.yaml
```

* Use the path for manual autoinstall if needed:

```text
file:///media/autoinstall/autoinstall.yaml
```

---

## Step 5: Using Autoinstall

1. Boot the Ubuntu 24.04.1 Desktop ISO.
2. If the installer does not auto-detect your `autoinstall.yaml`, provide a manual path:

   * Press `Ctrl+Alt+F2` (or `Ctrl+Alt+T` in GUI) to open a shell.
   * Attach your second drive or USB.
   * List disks: `lsblk -f`
   * Mount your autoinstall partition if needed.
   * Restart to GRUB menu and append to the Linux line:

```text
subiquity.autoinstallpath=/media/ubuntu/CONFIG/autoinstall.yaml
```

---

## Step 6: autoinstall.yaml Example

```yaml
# Use spaces, no tabs
autoinstall:
  version: 1
  identity:
    realname: "Your Name"
    username: username
    password: "<generated-hash>"
    hostname: computer-name
    group: sudo

  interactive-sections:
    - network

  keyboard:
    layout: "us"

  locale: "en_US.UTF-8"
  timezone: "America/New_York"

  ssh:
    install-server: True
    authorized-keys:
      - ssh-ed25519 AAAAC3Nzac user@host
    allow-pw: false

  early-commands:
    - sudo apt update -y && sudo apt upgrade -y

  shutdown: reboot

  error-commands:
    - tar -czf /installer-logs.tar.gz /var/log/installer
    - journalctl -b > /installer-journal.log

  late-commands:
    - sudo apt update -y && sudo apt upgrade -y
    - sudo systemctl start sshd
    - sudo systemctl enable sshd
    - sudo systemctl restart sshd
    - firewall-cmd --service=ssh --permanent
    - firewall-cmd --reload
```

---

## Notes

* Desktop ISOs do not include `openssh-server` by default, so network access is required.
* Using a single sudo user during install simplifies repetitive deployments.
* For advanced post-install configurations, modify `late-commands`.

---

## References & Further Reading

* [Autoinstall Quick Start Guide (Canonical)](https://canonical-subiquity.readthedocs-hosted.com/en/latest/howto/autoinstall-quickstart.html#autoinstall-quick-start) – Step-by-step guide for creating autoinstall configurations.
* [Autoinstall Reference (Canonical)](https://canonical-subiquity.readthedocs-hosted.com/en/latest/reference/autoinstall-reference.html#) – Full documentation of all configuration options for `autoinstall.yaml`.
* [SSH Key Generation](https://www.ssh.com/academy/ssh/keygen) – How to generate secure SSH key pairs.
* [Subiquity 24.04.1 Release Notes](https://discourse.ubuntu.com/t/subiquity-24-04-1-has-been-released-to-the-stable-channel/44493?utm_source=chatgpt.com) – Updates and improvements in the latest Subiquity installer.
* [Autoinstall Video Tutorial](https://youtu.be/ibvxiybT96M?si=bBZ25yunys_H9CKb) – Visual walkthrough of autoinstall setup and usage.
