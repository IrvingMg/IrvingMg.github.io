+++
date = '2025-12-29T11:58:28+01:00'
draft = false
title = 'Remote Development on a Linux Machine from macOS or Windows'
+++

If your main development machine is a Mac or Windows computer, at some point you’ll probably need to develop or test something on Linux. Maybe you’re building for multiple platforms, working a lot with containers (which often run faster natively on Linux), or simply want to offload some workloads to a different machine. Whatever the case, sometimes you need Linux even if it’s not your primary system.

If you have a second machine running Linux, setting it up for remote development over SSH is a great option. This lets you use the full power and performance of Linux while still working from your Mac or Windows environment.

In my case, I’m using a machine running Ubuntu Desktop 24.04 and accessing it from my Mac over SSH within my local network. Here’s how I configured it.

> **Note:** These commands are for Ubuntu. On other Linux distros, the package manager or firewall commands may vary.

---

## PART 1: Set Up the Ubuntu Machine (SSH Server)

### 1. Install and enable the SSH server

```bash
sudo apt update
sudo apt install openssh-server
sudo systemctl enable ssh
sudo systemctl start ssh
```

### 2. Find the machine’s IP address

```bash
hostname -I
```

Take note of the first IP shown (for example, `192.168.1.100`).

### 3. Configure the firewall to allow local SSH connections

```bash
sudo ufw enable
sudo ufw default deny incoming
sudo ufw default allow outgoing

# Allow SSH only from devices on your local network
# Use your IP with the last number replaced by 0, followed by /24
# Example: if your IP is 192.168.1.100, use 192.168.1.0/24
sudo ufw allow from 192.168.1.0/24 to any port 22

sudo ufw status verbose
```

### 4. Verify that SSH is running

```bash
sudo systemctl status ssh
```

It should show `active (running)`.

---

## PART 2: Set Up the Mac (SSH Client)

> **Note:** If you're on Windows, you can run the same SSH commands from PowerShell or Windows Terminal — Windows 10/11 already includes OpenSSH.

### 1. Test the SSH connection

```bash
ssh your_ubuntu_username@192.168.1.100
```

Replace the username and IP address with your own.  
Type **yes** when asked about the fingerprint, then enter your Ubuntu password.

If the connection works, disconnect with:

```bash
exit
```

### 2. Create an SSH key for passwordless login

```bash
ssh-keygen -t ed25519
```

Press Enter to accept the defaults. (You can optionally add `-C "your_email"` as a label.)

Copy the key to the Ubuntu machine:

```bash
ssh-copy-id your_ubuntu_username@192.168.1.100
```

Enter your password one last time.

---

## Optional Enhancements

### 1. Create an SSH alias (recommended)

Creating an alias makes it easier to connect without typing your full username and IP each time.

```bash
nano ~/.ssh/config
```

Add:

```
Host ubuntu-dev
    HostName 192.168.1.100
    User your_ubuntu_username
    IdentityFile ~/.ssh/id_ed25519
    # Allows your local SSH keys to be used on the remote machine
    # (e.g., to clone private Git repos from your Linux environment)
    ForwardAgent yes
```

Save and close the file.

Now you can connect with:

```bash
ssh ubuntu-dev
```

### 2. Set a static IP for your Ubuntu machine

By default, your Ubuntu machine gets its IP address from your router (DHCP). This can change after a reboot, which would break your SSH connection until you update it.

If you prefer your Ubuntu machine to always keep the same IP, you can set a **static IP address**.

You can do this in two ways:

1. **Reserve the IP on your router** (easier and recommended)  
2. **Configure a static IP directly on Ubuntu using Netplan**

If you want to configure it directly on Ubuntu, here is a clear step-by-step guide for Ubuntu 24.04:

[Ubuntu 24.04 static IP guide](https://linuxconfig.org/setting-a-static-ip-address-in-ubuntu-24-04-via-the-command-line)

---

## Connect and Start Developing

Once everything is set up, you can open a terminal on your Mac or Windows machine and connect to your Linux development environment instantly:

```bash
ssh ubuntu-dev
```

Your Linux machine stays fully usable locally, and you get access to its entire environment for development.

If you use VS Code, you can also install the **Remote – SSH** extension:  
[VS Code Remote – SSH](https://code.visualstudio.com/docs/remote/ssh)

This makes it easy to work on Linux projects seamlessly from your Mac or Windows setup.
