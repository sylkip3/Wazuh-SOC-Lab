Wazuh SOC Lab & Threat Monitoring Environment

Project Overview
This project documents the end-to-end deployment of a **Security Operations Center (SOC)** lab using the **Wazuh 4.9.2** stack. The environment is designed to simulate real-world cyberattacks and monitoring by networking an attacker (**Kali Linux**), a vulnerable target (**Metasploitable 2**), and a central monitoring server (**Ubuntu 24.04**) within a hardened virtual infrastructure.

> **Note:** For a comprehensive, step-by-step walkthrough with raw logs and additional screenshots, please refer to the [Full Project Documentation Word File](./Documentation/Sylvester_Wazuh_Lab_Walkthrough.docx).

---

Technical Architecture

### 1. Virtualized Infrastructure
* **Hypervisor:** VirtualBox
* **Network Backbone:** A custom NAT Network named `Sylvester_Security_Net`
* **Network Configuration:** All VMs utilize **Promiscuous Mode (Allow All)** to facilitate deep packet inspection and cross-node monitoring.

### 2. Node Specifications
| Machine | OS | Role | Resources |
| :--- | :--- | :--- | :--- |
| **Wazuh Manager** | Ubuntu 24.04 | SOC / SIEM Host | 5GB RAM / 23GB Disk |
| **Attacker** | Kali Linux | Penetration Testing | Pre-configured Image |
| **Target** | Metasploitable 2 | Vulnerable Endpoint | 512MB RAM |
<img width="1432" height="533" alt="image" src="https://github.com/user-attachments/assets/0d267111-b276-4fd4-abbc-26d1f0e7bdc2" />

### 3. Network & Access Configuration
To bypass NAT isolation and manage the SOC from the Windows host, the following **Port Forwarding** rules were implemented:
* **Wazuh Dashboard:** Host Port `4433` → Guest Port `443`
* <img width="1058" height="647" alt="image" src="https://github.com/user-attachments/assets/690ef23d-5ee4-49b2-a523-9f46934a7967" />

* **SSH Access:** Host Port `5555` → Guest Port `22`
  <img width="893" height="812" alt="image" src="https://github.com/user-attachments/assets/451b807c-b215-4551-bd0c-2d9c19dc4a1b" />

* **Firewall Persistence:** Configured Ubuntu `ufw` to permit traffic on port `443/tcp`.
Engineering Challenges & Solutions
A significant portion of this project involved navigating infrastructure-level deadlocks during the Wazuh installation:
<img width="730" height="443" alt="image" src="https://github.com/user-attachments/assets/febc01dd-2de6-4391-8b77-8959367149ca" />

### **Challenge: DPKG Metadata Corruption**
* **Issue:** Encountered "No such file" errors for core binaries like `wazuh-keystore` due to a "dirty" package state.
* **Solution:** Performed a full `purge` of the manager service and manually sanitized the `/var/ossec` filesystem to reach a "zero-state" for a clean reinstall.

### **Challenge: Indexer Initialization Timeout**
* **Issue:** The OpenSearch-based indexer failed to start due to permission deadlocks and locked shards.
* **Solution:** Purged corrupted shards in `/var/lib/wazuh-indexer/` and reassigned recursive ownership to the service user.
* <img width="1645" height="887" alt="image" src="https://github.com/user-attachments/assets/7661dcd6-f2de-41dc-9057-7fcbc0102cf5" />


### **Challenge: Resource Starvation**
* **Issue:** The Node.js dashboard optimization phase caused system hangs on 2GB RAM.
* **Solution:** Scaled resources to **5GB RAM** and ensured a minimum of **16GB free disk space** before final deployment.
<img width="1036" height="485" alt="image" src="https://github.com/user-attachments/assets/285a711d-ed99-447e-afef-b8220beb04f4" />

---

## Troubleshooting Commands Used
If you are replicating this lab and run into similar issues, these commands proved vital:
```bash
# Resetting the Indexer
sudo rm -rf /var/lib/wazuh-indexer/*
sudo chown -R wazuh-indexer:wazuh-indexer /var/lib/wazuh-indexer

# Repairing DPKG & OSSEC
sudo apt-get purge wazuh-manager -y

sudo rm -rf /var/ossec
Service Persistence
To ensure high availability of the SOC environment, all Wazuh components were configured as system services using systemctl. This ensures that in the event of a power failure or VM restart, the Indexer, Manager, and Dashboard initialize automatically without manual intervention.


