# linux-homeserver-config 🏡🐧🔧

All the configuration of my Linux homeserver, used to back up my phone's data with self-hosted services:

- [📁 Syncthing](https://syncthing.net/) for folders like `DCIM`, `Music`, etc.
- [📅 Baïkal](https://sabre.io/baikal/) for contacts and calendars (CalDAV & CardDAV).

> [!NOTE]  
> This is just a dead-simple homeserver for my simple needs, with only what I actually use.  
> No access from outside, no VMs, no HTTPS, no streaming, not even reverse proxy... I want to keep everything simple.

## 📑 Table of Contents

- [linux-homeserver-config 🏡🐧🔧](#linux-homeserver-config-)
  - [📑 Table of Contents](#-table-of-contents)
  - [🛠️ General Setup](#️-general-setup)
    - [💻 Hardware](#-hardware)
    - [🐧 Operating System](#-operating-system)
  - [⚙️ OS Installation Notes](#️-os-installation-notes)
    - [🧩 Basic Configuration](#-basic-configuration)
    - [💽 Disk \& Partitioning](#-disk--partitioning)
    - [👤 User Configuration](#-user-configuration)
    - [🔐 Security \& Snaps](#-security--snaps)
  - [🐧 Server Configuration](#-server-configuration)
    - [🦑 Clone Project](#-clone-project)
    - [📡 SSH Access](#-ssh-access)
    - [📦 Install Dependencies](#-install-dependencies)
    - [🚀 Run the Playbooks](#-run-the-playbooks)
    - [🔍 Check it Worked](#-check-it-worked)
    - [🔐 Change Your Server Password](#-change-your-server-password)
    - [⛓️‍💥 Troubleshooting](#️-troubleshooting)

## 🛠️ General Setup

### 💻 Hardware

Running on an old Acer laptop (VX5-591G) with the following specs:

| Component | Specs                               |
| --------- | ----------------------------------- |
| CPU       | Intel i5-7300HQ                     |
| RAM       | 8 GiB DDR4 @ 2400 MHz               |
| Storage   | 128 GB SSD (boot) + 1 TB HDD (data) |

> [!TIP]
> I removed every laptop component I could (as long as it was easy) to reduce power consumption, including the battery and screen.  
> I don't know if it makes a big difference, but at least now it looks cool.

### 🐧 Operating System

- OS: [Ubuntu Server 24.04.3 LTS](https://ubuntu.com/download/server).
- Why? Stable, widely supported, and lots of tutorials available.

## ⚙️ OS Installation Notes

### 🧩 Basic Configuration

- Language: English (for easier tutorial searching).
- Keyboard: French, with `Variant: French` (so original) 🥖.
- Install type: Ubuntu Server (non-minimal), without third-party drivers (I don't need any of them).
- Network: Ethernet — *“If it doesn't move, use Ethernet.”*
- Proxy: None.
- Mirror: Default Ubuntu archive.

### 💽 Disk & Partitioning

- Guided storage configuration: Use the entire disk (SSD) for faster booting. No encryption — the server may shut down often and I don't want to retype the password every time.

- Storage configuration: Format the HDD, then on the free space, add a GPT partition mounted at `/srv/` (to store data). No RAID (disks aren't identical) and no LVM (adds complexity and isn't useful in my case).

### 👤 User Configuration

| Prompt   | Value                                                          |
| -------- | -------------------------------------------------------------- |
| Name     | `lechartreux`                                                  |
| Hostname | `homeserver`                                                   |
| Username | `lechartreux`                                                  |
| Password | Something simple (to be changed later with a password manager) |

### 🔐 Security & Snaps

- Skip Ubuntu Pro
- ✅ Install OpenSSH Server
- ✅ Import SSH keys from GitHub (`le-chartreux`)
- 🚫 Do not install any featured server snaps (they may conflict with ports used later)

Then restart, remove the USB flash drive, reboot again — and boom! A fresh homeserver.

## 🐧 Server Configuration

> [!NOTE]  
> By default, all commands should be run from your PC unless specified otherwise.

### 🦑 Clone Project

Clone the project:

```sh
git clone git@github.com:le-chartreux/linux-homeserver-config.git
```

### 📡 SSH Access

If you don’t know your server’s IP address, log in locally and run:

```sh
$ hostname -I
192.168.1.7  # Followed by an IPv6 address we don't care about.
```

From your PC, verify you can connect to your server using the IP and your username:

```sh
# Update with your IP and username.
$ ssh lechartreux@192.168.1.7
```

Boom, you're in!
Next, update the [hosts.ini](hosts.ini) file at the project root, with your server's IP and username.

### 📦 Install Dependencies

Navigate to the project directory, create and activate a virtual environment, then install the dependencies:

```sh
cd linux-homeserver-config
python -m venv .venv
source .venv/bin/activate
pip install .
```

### 🚀 Run the Playbooks

From your PC (not the server), run:

```sh
ansible-playbook playbooks/site.yml --ask-become-pass -v
```

When prompted, enter your server password (the one you set during the OS installation).

### 🔍 Check it Worked

Open your web browser and visit:

- Syncthing’s web interface at [http://homeserver.local:8384](http://homeserver.local:8384).
- Baikal's web interface at [http://homeserver.local:8586](http://homeserver.local:8586).

You should see the pages load correctly with all styles and images displayed.
If you encounter any issues, see [⛓️‍💥 Troubleshooting](#️-troubleshooting).

> [!TIP]  
> If you want to change these ports, update the vars section in the respective playbooks and run them again.

### 🔐 Change Your Server Password

For better security, change your server’s password to a strong one, generated from and stored in your preferred password manager.

Connect via SSH and run the password change command:

```bash
ssh homeserver.local
passwd
```

### ⛓️‍💥 Troubleshooting

> **Problem:** When accessing the IP from your browser, you see **HTTPS-Only Mode Alert – Secure Site Not Available**.

This happens because the server is not set up to support HTTPS.  
Simply choose **“Continue to HTTP Site”** (or the equivalent option) to proceed.

> **Problem:** Your browser cannot resolve `homeserver.local`.

This is often caused by [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) not being enabled on your device.  
To confirm, try accessing the server directly via its IP address (e.g., `http://192.168.1.8:8384/` for Syncthing).  

- If the IP works, enable [mDNS](https://en.wikipedia.org/wiki/Multicast_DNS) on your device to use the `.local` address, or continue using the IP directly.  
- If the IP does **not** work, check the troubleshooting steps below.

> **Problem:** When accessing the IP from your browser, you see **“Unable to connect”**.

Make sure you’re using **HTTP** (e.g., `http://your-server-ip:PORT`) instead of **HTTPS**, unless you’ve explicitly set up SSL/TLS for the service (not done by the playbooks).

> **Problem:** When accessing the IP from your browser, you see **“Warning: Potential Security Risk Ahead”**.

This happens because the connection is not secured with a trusted certificate.  
Click **`Advanced...`** then **`Accept the Risk and Continue`** to proceed.
