# 🛡️ Enterprise Home Lab & Security Infrastructure

## 📌 Projektübersicht

Dieses Repository dokumentiert den Aufbau eines professionellen Homelab-Labs mit dem Schwerpunkt **Virtualisierung, Netzwerksegmentierung und IT-Security**.  
Ziel ist es, eine realistische, segmentierte Infrastruktur aufzubauen, die für praktische Erfahrung in den Bereichen Proxmox, pfSense, Firewalling, VLANs, VMs, Domain Services und Security-Hardening genutzt werden kann. :contentReference[oaicite:1]{index=1}

---

## 📍 Ziele des Projekts

- Aufbau einer **Proxmox-Umgebung** als zentrale Virtualisierungsplattform  
- Einsatz von **pfSense** als Security-Firewall & Router  
- Segmentierung des Netzwerks mit **VLANs & Subnetzen**  
- Einrichtung einer **DMZ** für isolierte Dienste  
- Integration von **Windows- und Linux-VMs**  
- Dokumentation aller Schritte für Lernprozess & Portfolio  
- Optionaler Ausbau Richtung Docker/Kubernetes

---

## 🧱 Hardware-Stack

| Komponente | Beschreibung |
|------------|--------------|
| **Hypervisor** | AOOSTAR WTR PRO (AMD Ryzen 7 5825U, 64 GB RAM) |
| **Firewall/Router** | TP-Link Archer AX18 |
| **ISP Gateway** | Magenta Fiber Box (aktuell Double-NAT) |
| **Admin Client** | Linux Mint 22.2 Xfce VM |

> Dein Homelab-Host agiert als alleiniger Server für Virtualisierung und Services. :contentReference[oaicite:2]{index=2}

---

## 📊 Architektur & Netzwerk-Topologie

### 🗺️ Übersicht

graph TD
subgraph Internet
ISP[Magenta Fiber Box]
end

subgraph Physische Infrastruktur
    Archer[TP-Link Archer AX18 - 192.168.1.1]
    ProxmoxHost[AOOSTAR WTR PRO - Proxmox]
end

subgraph Virtuelle Umgebung (Proxmox)
    WAN_Bridge((vmbr0 - WAN))
    LAN_Bridge((vmbr1 - LAN))
    pfSense[pfSense Firewall]
    Mint[Linux Mint Management]
end

ISP --- Archer
Archer --- WAN_Bridge
WAN_Bridge --- pfSense
pfSense --- LAN_Bridge
LAN_Bridge --- Mint

→ Diese Darstellung visualisiert das Netzwerk-Setup inkl. Virtual Bridges und Firewall-Zonen. :contentReference[oaicite:3]{index=3}

---

## 🗂️ Installation & Konfiguration

### 🛠️ Proxmox VE

1. Proxmox VE ISO herunterladen und auf einen Boot-USB schreiben  
2. BIOS-Einstellungen:  
   - UEFI  
   - Virtualisierung (SVM/IOMMU) aktiv  
   - Secure Boot deaktiviert  
3. Proxmox installieren und ersten Host konfigurieren  
4. Management-IP festlegen (statisch oder DHCP reserviert)

📌 **Ziel:** Stable Hypervisor mit Webadmin-Zugang.  
📸 Screenshot: *Proxmox Dashboard* :contentReference[oaicite:4]{index=4}

---

### 📡 Netzwerk & VLANs

- `vmbr0`: WAN  
- `vmbr1`: Internes LAN  
- Trunking über physische NIC zum Router  
- spätere VLAN-Zuordnung für DMZ, Server …

📸 Screenshot: *Netzwerk-Konfiguration* :contentReference[oaicite:5]{index=5}

---

### 🔐 pfSense Firewall

1. VM in Proxmox anlegen  
2. Interfaces:
   - WAN (vmbr0)  
   - LAN (vmbr1)
3. Firewall-Regeln:
   - LAN → WAN (erlaubt)  
   - DMZ → LAN (blockiert)
4. DNSBL / pfBlockerNG aktivieren für Tracking-Blockierung

📸 Screenshot: *pfSense Dashboard, Firewall Rules* :contentReference[oaicite:6]{index=6}

---

## 💻 VMs & Dienste

### 🟢 Management VM

- **Linux Mint 22.2 Xfce**  
- Verwaltungstools: SSH, Webbrowser, Terminal  
- Ressourcen gering gehalten für hohe Proxmox-Leistung

📸 Screenshot: *Ping-Test / Status* :contentReference[oaicite:7]{index=7}

---

## 🌐 Dienste & Tests

| Dienst | Zweck | Status |
|--------|-------|--------|
| Apache2 Webserver | Host Dienst in LAN | Running |
| Firewall DNSBL | Tracking Block | Aktiv |
| NAT Firewall | HTTP/HTTPS Public | Konfiguriert |

📸 Screenshot: *Webserver in Aktion* :contentReference[oaicite:8]{index=8}

---

## 📊 Security & Hardening

- Management-Port auf **8443** gelegt  
- DNS Blocklists via pfBlockerNG  
- SSH Hardening (optional dokumentieren)  
- VLAN Isolation  
- Monitoring (optional in Zukunft)

→ Das zeigt *Security-Fokus und Nachvollziehbarkeit*.

---

## 📦 Roadmap & To-Do

**Kurzfristig**

- VLANs komplett einrichten  
- DMZ voll konfigurieren  
- Windows AD Domain Lab

**Mittelfristig**

- Docker / Portainer Lab  
- Kubernetes/k3s Testcluster  
- Monitoring (Grafana/Prometheus)

**Langfristig**

- Multi-Node Proxmox Cluster  
- Automatisierung (Ansible, Terraform)

---

## 📸 Screenshots & Verzeichnisstruktur

Empfohlene Repo-Ordnerstruktur:
/
├── README.md
├── docs/
│ ├── proxmox_install.md
│ ├── pfsense_setup.md
│ ├── vlan_design.md
│ └── security_hardening.md
├── img/
│ ├── proxmox_dashboard.png
│ ├── pfsense_rules.png
│ └── network_topology.png
└── CHANGELOG.md


📸 **Screenshots sollten enthalten:**  
- Netzwerkkonfiguration  
- Firewall Regeln  
- Proxmox Status  
- Testverbindungen

---

## 🧾 CHANGELOG (Beispiel)

```markdown
## v0.1 – Basis
- Proxmox installiert
- pfSense V1
- Management-VM

→ Hilft beim Fortschritts-Tracking.

📚 Ressourcen & Inspiration

Homelab Beispiele aus der Community

VLAN und Netzwerkdesign Best Practices

Proxmox & pfSense offizielle Dokumentation

📜 Lizenz

Dieses Projekt ist frei dokumentiert und steht als Lern-Ressource zur Verfügung


---

# 🧠 Warum diese Struktur?

Die Struktur entspricht Best Practices aus erfolgreichen Homelab-Repos, in denen **Dokumentation reproduzierbar, verständlich und modular** ist. Sie zeigt nicht nur Screenshots, sondern **Step-by-Step Installationswissen, Netzwerkdesign, Tests und Roadmap** – genau das, was Recruiter im IT-Security-Kontext sehen wollen. :contentReference[oaicite:10]{index=10}

---

## 📌 Nächster Schritt

Wenn du möchtest, können wir **für jede Unterseite (`docs/...`) die Inhalte auch einzeln ausformulieren** – z. B.:

✔ Proxmox Installations-Guide  
✔ pfSense Setup Schritt-für-Schritt  
✔ VLAN & DMZ Konfiguration  
✔ Windows AD Integrierung

