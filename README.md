# WAZUH Home Lab — SIEM and File Integrity Monitoring

## 1. Overview

Wazuh is a free, open-source security platform that offers:

- Log analysis  
- File integrity monitoring  
- Intrusion detection  
- Vulnerability detection  
- Real-time alerting  

---

## 2. Lab Architecture

| Component | Host Role |
|----------|------------|
| Wazuh Manager (Kali Linux - VirtualBox) | Collects, analyzes, and stores data from agents |
| Wazuh Agent (Windows - host machine) | Sends logs and system events to the Wazuh manager |

---

## 3. Network Configuration

Use **Bridged Adapter** in VirtualBox to place the Kali Linux server on the same network as the host.

This allows access between the host and the guest.

---

## 4. Prerequisites

- VirtualBox installed
- Kali Linux (latest) installed in VirtualBox (bridged networking)
- Internet access on Kali VM
- Administrative access on the Windows host
- Optional: basic knowledge of Linux and system administration

---

## 5. Installing the Wazuh Manager (Kali Linux)

Run the following steps on your Kali Linux VirtualBox server.

### 5.1 Add Wazuh GPG Key

Run this via a terminal window:

```bash
curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | sudo gpg --dearmor -o /usr/share/keyrings/wazuh-archive-keyring.gpg
```

This adds the GPG key to verify Wazuh packages.

### 5.2 Download and Execute Wazuh Installation Script

```bash
curl -sO https://packages.wazuh.com/4.12/wazuh-install.sh && sudo bash ./wazuh-install.sh -a -i
```

- `-a`: Installs all components (manager, indexer, etc.)
- `-i`: Runs in interactive mode

The script installs all required services and configures them automatically.

---

## 6. Accessing the Wazuh Dashboard

After installation:

1. Check your Kali Linux VM’s IP address (Kali VM IP):

```bash
ifconfig
```

2. Open a browser on your machine and go to:

```text
https://<kali-vm-ip>
```

3. Accept any browser security warning due to the self-signed certificate.

4. Log in using the credentials displayed at the end of the installation script.

---

## 7. Installing the Wazuh Agent (Windows Host)

1. Download the latest Wazuh agent MSI installer from the official documentation:  
   **Wazuh Agent for Windows**

2. Install the MSI package on your Windows system using the default settings.

---

## 8. Registering the Agent with the Manager

### 8.1 Generate Agent Key on Kali Manager

Run the agent management utility on the Kali manager:

```bash
sudo /var/ossec/bin/manage_agents
```

Then:

- Select `A` to add an agent.
- Assign a name (e.g., `WindowsHost`).
- Leave IP address blank unless static assignment is needed.
- After creation, select `E` to extract the key.
- Copy the key output.

### 8.2 Apply Key in the Windows Agent

1. Open the **Wazuh Agent Manager** GUI from the Start Menu.
2. Paste the copied key into the appropriate field.
3. Save and apply the key.
4. Add the manager’s IP address (IP address of your Kali Linux manager).
5. Restart the agent service.

You can then open the Wazuh dashboard and verify that the agent has onboarded.

---

## 9. File Integrity Monitoring on Windows

Wazuh supports real-time monitoring of file and folder changes using **Syscheck**.

### 9.1 Edit Agent Configuration

Open this configuration file:

```text
C:\Program Files (x86)\ossec-agent\ossec.conf
```

Add the following entry inside the `Directory` block:

```xml
<directories realtime="yes">C:\Users\abc\Test</directories>
```

This monitors the specified folder in real-time.

> Replace `C:\Users\abc\Test` with your folder path.

### 9.2 Restart the Agent

After saving the changes, restart the Wazuh agent service to apply the configuration.

---

## 10. Verifying Setup

1. Open the Wazuh Dashboard in your browser.
2. Navigate to **Agents** → ensure the Windows agent is listed and status is **Active**.
3. Go to the **Integrity Monitoring** section.
4. Perform actions (create/modify/delete files) in the monitored folder.
5. Confirm that alerts appear in the dashboard.
