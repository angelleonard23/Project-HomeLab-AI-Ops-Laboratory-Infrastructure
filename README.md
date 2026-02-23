# Project HomeLab - AI-Ops Laboratory Infrastructure

This repository documents my learning and development process of a security-focused HomeLab 
utilizing Proxmox VE, pfSense Firewall, network segmentation, and virtualization.

The objective is to gain hands-on experience for a career as a System Administrator with a focus 
on Networking and IT Security.

---

## Table of Contents

- [Project Objectives](#project-objectives)
- [Hardware & Initial Setup](#hardware--initial-setup)
- [Architecture Overview](#architecture-overview)
- [Network Design](#network-design)
- [Proxmox Installation](#proxmox-installation)
- [pfSense Firewall](#pfsense-firewall)
- [Virtual Machines](#virtual-machines)
- [Testing & Validation](#testing--validation)
- [Security Measures](#security-measures)
- [Current Status](#current-status)
- [Roadmap / Next Steps](#roadmap--next-steps)

---

## Project Objectives

- Build a Proxmox HomeLab on Bare Metal.
- Implement pfSense as the central Firewall & Router.
- Network isolation using VLANs & Subnets.
- Implementation of a DMZ for public-facing services.
- Operation of Linux and Windows VMs.
- Systematic documentation of the learning process.

---

## Hardware & Initial Setup

| Component | Description |
|-----------|-------------|
| Server    | AOOSTAR WTR PRO – AMD Ryzen 7 5825U, 64 GB RAM |
| Router    | TP-Link Archer AX18 |
| ISP       | Magenta Gateway |
| Client    | Linux Mint / Laptop |

---

## Network Documentation (As of Feb 10, 2026)

This project includes the setup of an isolated laboratory environment (VLAN 30) behind a pfSense firewall on a Proxmox host.

### Core Components
* **VM Name:** `ai-ops-01`
* **OS:** Ubuntu Server
* **IP Address:** `192.168.30.20`
* **VLAN ID:** 30
* **Gateway:** `192.168.30.1` (pfSense)

---

## Troubleshooting Process: Path to Connectivity

To establish internet connectivity for the VM in VLAN 30, the following potential issues were systematically excluded using the OSI Model approach:

### 1. Layer 2: Physical & Logical Connection (Proxmox/VLAN)
* **Issue:** The VM could not reach the gateway.
* **Solution:** Ensured the Proxmox bridge (`vmbr1`) is "VLAN-aware" and the VM is assigned the correct VLAN tag (30).
* **Validation:** `ip neigh` showed the status `REACHABLE` for the gateway's MAC address.

### 2. Layer 3: Routing & Firewall (pfSense)
* **Issue:** Packets were blocked by pfSense (`Default deny rule`).
* **Solution:** * Created a **Pass Rule** on the `AIOPS` interface.
    * Changed the source from a specific IP to **`Any`** to rule out subnet misconfigurations.
* **Validation:** Successful ping to gateway `192.168.30.1`.

### 4. Layer 4 & NAT: Internet Access (WAN)
* **Issue:** Packets reached pfSense but did not exit correctly to the internet.
* **Solution:**
    * Configured **Outbound NAT** on the `WAN` interface for the `192.168.30.0/24` network.
    * Disabled **Static Port** to ensure compatibility with the upstream primary router.
* **Validation:** `ping -n 1.1.1.1` was successful.

### 5. Application Layer: Name Resolution (DNS)
* **Issue:** Internet access via IP was functional, but `google.com` could not be resolved.
* **Solution:** Added the `192.168.30.0/24` subnet to the **DNS Resolver Access Lists** in pfSense.
* **Validation:** `ping google.com` was successful.

---

## Sensitive Data & Vault
In accordance with the project policy (Jan 28, 2026), all sensitive variables and the final network topology are managed within the `vault_passwords.yml` file.

## 5. System Optimization & Management (As of Feb 11, 2026)

After establishing basic connectivity, the VM was prepared for professional operation within Proxmox and for automation.

### QEMU Guest Agent Integration
To improve communication between the Proxmox host and the VM (e.g., for graceful shutdowns and IP display), the Guest Agent was installed.
* **Command:** `sudo apt install qemu-guest-agent`
* **Status:** Service successfully activated, even though Ubuntu manages it as a static unit.
* **Result:** The VM's IP address is now directly visible in the Proxmox summary.

> ![Proxmox Guest Agent Status](./img/proxmox_guest_agent.png)

---

### Static Network Configuration (Netplan)
To ensure the VM remains reachable at a consistent address for automation, it was transitioned from DHCP to a static configuration.
* **File:** `/etc/netplan/50-cloud-init.yaml`
* **Configuration:**
  * IP: `192.168.30.20/24`
  * Gateway: `192.168.30.1`
  * DNS: `192.168.30.1` (pfSense) & `8.8.8.8`

> **Insert Screenshot here:** (Content of Netplan YAML file)
> ![Netplan Config](./img/netplan_yaml.png)

---

## 6. Automation with Ansible & Vault

Laboratory access is now managed centrally from the Management PC (Linux Mint) via Ansible.

### Security Infrastructure (SSH & Vault)
* **SSH Keys:** The public key from the Management PC was deployed (`ssh-copy-id`) to enable passwordless logins.
* **Ansible Vault:** Sensitive data, such as the `sudo` password for user `angel`, is stored encrypted in `group_vars/all.yml`.
* **Vault Automation:** Optimized the workflow using a local `.vault_pass.txt` and `ansible.cfg` to eliminate manual password prompts during playbook execution.

> **Insert Screenshot here:** (Successful Ansible Ping or `whoami` test run)
> ![Ansible Success](./img/ansible_test_root.png)

### Automated Maintenance Playbooks
Implemented workflows:
1. **`update_system.yml`**: Performs a full `apt upgrade`.
2. **`check_reboot.yml`**: Checks for the existence of `/var/run/reboot-required` and securely reboots the VM if necessary.

> **Insert Screenshot here:** (Ansible Playbook Run without errors)
> ![Ansible Playbook Run](./img/ansible_playbook_run.png)

---

## Current Project Status (Milestone 1 reached)
- [x] Isolated network environment (VLAN 30) active.
- [x] Internet access & DNS via pfSense stable.
- [x] VM management via Proxmox Guest Agent active.
- [x] Full control established via Ansible automation.

## 7. Infrastructure & Container Automation (As of Feb 11, 2026)

Following successful network integration, the VM `ai-ops-01` was fully provisioned as a **Docker Host** via automation.

### Milestone: Infrastructure as Code (IaC)
The entire setup process is managed via Ansible, including:
* **System Maintenance:** Intelligent reboot checks (`check_reboot.yml`).
* **Docker Stack:** Automated installation of Docker Engine & Docker Compose (`setup_docker.yml`).
* **Security:** Secure management of database passwords and API keys using **Ansible Vault**.

> **Proof 1: Intelligent System Maintenance**
> Ansible independently detects if a reboot is required and skips the task (`skipping`) if the system is up to date.
> ![Ansible Reboot Check](./img/ansible_reboot_logic.png)

---

### Milestone: Container Host Ready
The VM is now ready for the deployment of n8n and AI tools. The user `angel` has been authorized to manage Docker containers without `sudo`, improving security and workflow.

> **Proof 2: Successful Docker Provisioning**
> The final playbook report shows the successful setup of all components and user permissions.
> ![Docker Setup Success](./img/docker_setup_recap.png)

---

## Current Project Status
- [x] Static IP & Guest Agent configured.
- [x] Ansible Vault & SSH Key authentication active.
- [x] Docker & Docker Compose fully operational.
- [ ] **Next Step:** Start the n8n stack (n8n + Postgres).

## 9. Deployment of the AI Workflow System (n8n)

The heart of the automation, **n8n**, was deployed as a container stack alongside a **Postgres** database.

### Stack Details:
* **Orchestration:** Docker Compose (via Ansible `community.docker` collection).
* **Persistence:** Docker Volumes for n8n data and database content.
* **Security:** Dynamic injection of DB credentials via Ansible Vault during deployment.

> **Proof: Successful Stack Deployment**
> All tasks were executed without errors; the stack is production-ready.
> ![n8n Deployment Success](./img/n8n_deployment_recap.png)

### 9.1 Troubleshooting & Configuration Adjustments
During deployment, specific adjustments were made to make the stack operational in a local development environment:

* **YAML Validation:** Corrected indentation in the Docker Compose template to resolve the `additional properties not allowed` error.
* **Security Override:** Set `N8N_SECURE_COOKIE=false` to allow local HTTP access without SSL.
* **Vault Integration:** Used `vault_passwords.yml` for secure injection of the `POSTGRES_PASSWORD` variable.

### 9.2 Operational Verification
After deployment, system reachability was successfully verified.

| Component | Status | URL / Port |
| :--- | :--- | :--- |
| **n8n Frontend** | ✅ Online | `http://192.168.30.20:5678` |
| **Postgres DB** | ✅ Connected | Internal Port 5432 |

> **Proof: n8n Readiness**
> The system is ready for initial account creation and building the first workflow.

---

## 10. Integration of Local LLMs (Ollama & Llama3)

To ensure privacy-compliant and cost-free AI processing, the system was expanded with a local LLM interface.

### 10.1 Automated Ollama Installation
Installed via a dedicated Ansible playbook (`install_ollama.yml`), handling:
* Download and installation of the Ollama binary.
* Systemd service configuration for auto-start.
* Initial pull of the **Llama3** model (approx. 4.7 GB).

### 10.2 n8n AI Agent Configuration
An intelligent workflow was created in n8n, acting as the bridge between the automation server and the local AI.

**Configuration Details:**
* **Node Structure:** An `AI Agent` acts as the brain, supported by an `Ollama Chat Model`.
* **Connectivity:** Connected via VM IP on port `11434`.
* **Model:** Utilizing `llama3:latest`.

> **Proof: Successful AI Execution**
> The image shows the validated workflow. The green indicators confirm that the AI Agent successfully communicated with Llama3 and generated a response.


---

## 11. Modernizing the AI Control Plane: Open WebUI & MCP (As of Feb 23, 2026)

Following extensive testing, the architecture migrated from OpenClaw to **Open WebUI** for robust **Model Context Protocol (MCP)** integration and stable connection to local inference engines.

### 11.1 Decentralized Architecture ("The Control Plane")
Roles are distributed to optimize resources and isolate the management layer:
* **Management Hub (VM 102 - Mint):** Hosts the frontend (**Open WebUI**) and automation engine (**n8n**).
* **AI Service Node (ai-ops-01):** Hosts the inference engine (**Ollama**) and interface logic (**MCP Server & mcpo Bridge**).

### 11.2 Model Context Protocol (MCP) & Bridge Technology
Implemented a `FastMCP`-based server to grant the AI direct access to Proxmox infrastructure.
* **The Challenge:** Open WebUI requires a standard OpenAPI/REST interface, while MCP communicates natively via SSE (Server-Sent Events).
* **The Solution:** Implementation of the **`mcpo` Bridge**. This runs the MCP Python script as a stdio subprocess and translates tools into a dynamic `openapi.json` on port `5002`.
* **Security Patch:** Implemented a Python monkey-patch to disable `dns_rebinding_protection` in FastMCP, allowing cross-VLAN access from the Management VM to the Docker host.

---

## 12. Final AI-Ops Stack: Status & Validation

Provisioned entirely via Ansible (`deploy_ai_brain.yml`) using encrypted secrets from `vault_passwords.yml`.

### 12.1 Component Matrix
| Service | Port | Host | Role |
| :--- | :--- | :--- | :--- |
| **Open WebUI** | 8080 | Mint-VM (102) | Primary Chat Interface & Tool Hub |
| **n8n** | 5678 | Mint-VM (102) | Event-Handling & Workflow Automation |
| **mcpo (Bridge)** | 5002 | ai-ops-01 | REST Translator for Proxmox Tools |
| **Ollama** | 11434 | ai-ops-01 | Local LLM Inference (Llama3) |
| **Proxmox API** | 8006 | Host (WTR Pro) | Target Infrastructure for AI Control |

### 12.2 Connectivity Validation
Verified via the generated OpenAPI specification:
`curl http://192.168.1.10:5002/openapi.json | python3 -m json.tool`

**Result:** The AI now has "hands" within the HomeLab. It can autonomously:
1. Fetch VM lists from the Proxmox host.
2. Analyze resource utilization (CPU/RAM) across nodes.
3. Start/Stop VMs based on natural language commands.

---

## 13. Roadmap: Next Steps

- [ ] **pfSense Integration:** n8n workflows to analyze firewall logs via AI.
- [ ] **Self-Healing:** Automated snapshots prior to critical AI-triggered actions.
- [ ] **GitOps:** Versioning all MCP scripts and Ansible playbooks in a local repository.
