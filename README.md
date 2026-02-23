`# Project HomeLab - AI-Ops Labor Infrastruktur

Dieses Repository dokumentiert meinen Lern- und Aufbauprozess eines sicherheitsfokussierten HomeLabs
mit Proxmox VE, pfSense Firewall, Netzwerksegmentierung und Virtualisierung.

Ziel ist es, praktische Erfahrung für eine Karriere als Systemadministrator mit Schwerpunkt
Netzwerk & IT-Security zu sammeln.

---

## Inhaltsverzeichnis

- [Projektziele](#projektziele)
- [Hardware & Ausgangslage](#hardware--ausgangslage)
- [Architekturübersicht](#architekturübersicht)
- [Netzwerkdesign](#netzwerkdesign)
- [Proxmox Installation](#proxmox-installation)
- [pfSense Firewall](#pfsense-firewall)
- [Virtuelle Maschinen](#virtuelle-maschinen)
- [Tests & Validierung](#tests--validierung)
- [Security Maßnahmen](#security-maßnahmen)
- [Aktueller Stand](#aktueller-stand)
- [Nächste Schritte / Roadmap](#nächste-schritte--roadmap)

---

## Projektziele

- Aufbau eines Proxmox HomeLabs auf Bare Metal
- Einsatz von pfSense als zentrale Firewall & Router
- Trennung von Netzwerken mittels VLANs & Subnetzen
- Vorbereitung einer DMZ für öffentlich erreichbare Services
- Betrieb von Linux- und Windows-VMs
- Saubere Dokumentation des Lernprozesses

---

## Hardware & Ausgangslage

| Komponente | Beschreibung |
|----------|-------------|
| Server | AOOSTAR WTR PRO – AMD Ryzen 7 5825U, 64 GB RAM |
| Router | TP-Link Archer AX18 |
| ISP | Magenta Gateway |
| Client | Linux Mint / Laptop |

---
# Projekt: AI-Ops Labor Infrastruktur

## Netzwerk-Dokumentation (Stand: 10.02.2026)

Dieses Projekt umfasst die Einrichtung einer isolierten Labor-Umgebung (VLAN 30) hinter einer pfSense-Firewall auf einem Proxmox-Host.

### Kern-Komponenten
* **VM-Name:** `ai-ops-01`
* **Betriebssystem:** Ubuntu Server
* **IP-Adresse:** `192.168.30.20`
* **VLAN-ID:** 30
* **Gateway:** `192.168.30.1` (pfSense)

---

## Troubleshooting Prozess: Der Weg zur Konnektivität

Um die Internetverbindung für die VM im VLAN 30 zu etablieren, wurden folgende Fehlerquellen systematisch ausgeschlossen (OSI-Modell Ansatz):

### 1. Layer 2: Physikalische & Logische Verbindung (Proxmox/VLAN)
* **Problem:** Die VM konnte das Gateway nicht erreichen.
* **Lösung:** Sicherstellung, dass die Proxmox-Bridge (`vmbr1`) "VLAN-aware" ist und die VM den korrekten VLAN-Tag (30) besitzt.
* **Erfolgskontrolle:** `ip neigh` zeigte den Status `REACHABLE` für die MAC-Adresse des Gateways.

### 2. Layer 3: Routing & Firewall (pfSense)
* **Problem:** Pakete wurden von der pfSense blockiert (`Default deny rule`).
* **Lösung:** * Erstellung einer **Pass-Regel** auf dem Interface `AIOPS`.
    * Umstellung der Source von einer spezifischen IP auf **`Any`**, um Fehlkonfigurationen im Subnetz auszuschließen.
* **Erfolgskontrolle:** Ping auf das Gateway `192.168.30.1` war erfolgreich.

### 3. Layer 4 & NAT: Internet-Zugriff (WAN)
* **Problem:** Pakete kamen an der pfSense an, verließen diese aber nicht korrekt ins Internet.
* **Lösung:**
    * Konfiguration des **Outbound NAT** auf dem `WAN`-Interface für das Netz `192.168.30.0/24`.
    * Deaktivierung von **Static Port**, um Kompatibilität mit dem vorgeschalteten Hauptrouter sicherzustellen.
* **Erfolgskontrolle:** `ping -n 1.1.1.1` war erfolgreich.

### 4. Application Layer: Namensauflösung (DNS)
* **Problem:** Internetzugriff per IP möglich, aber `google.com` konnte nicht aufgelöst werden.
* **Lösung:** Freigabe des Subnetzes `192.168.30.0/24` in den **DNS Resolver Access Lists** der pfSense.
* **Erfolgskontrolle:** `ping google.com` war erfolgreich.

---

## Sensitive Daten & Vault
Entsprechend der Projektrichtlinie vom 28.01.2026 werden alle sensiblen Variablen und die finale Netzwerk-Topologie in der Datei `vault_passwords.yml` verwaltet.

## 5. System-Optimierung & Management (Stand: 11.02.2026)

Nachdem die grundlegende Konnektivität hergestellt war, wurde die VM für den professionellen Betrieb in Proxmox und die Automatisierung vorbereitet.

### QEMU Guest Agent Integration
Um eine bessere Kommunikation zwischen dem Proxmox-Host und der VM zu ermöglichen (z.B. für sauberes Herunterfahren und IP-Anzeige), wurde der Guest Agent installiert.
* **Befehl:** `sudo apt install qemu-guest-agent`
* **Status:** Der Dienst wurde erfolgreich aktiviert, auch wenn Ubuntu ihn als statische Unit verwaltet.
* **Ergebnis:** Die IP-Adresse der VM ist nun direkt in der Proxmox-Übersicht sichtbar.

> ![Proxmox Guest Agent Status](./img/proxmox_guest_agent.png)

---

### Statische Netzwerk-Konfiguration (Netplan)
Um sicherzustellen, dass die VM für die Automatisierung immer unter derselben Adresse erreichbar bleibt, wurde von DHCP auf eine statische Konfiguration umgestellt.
* **Datei:** `/etc/netplan/50-cloud-init.yaml`
* **Konfiguration:**
  * IP: `192.168.30.20/24`
  * Gateway: `192.168.30.1`
  * DNS: `192.168.30.1` (pfSense) & `8.8.8.8`

> **Hier Screenshot einfügen:** (Inhalt der Netplan YAML Datei)
> ![Netplan Config](./img/netplan_yaml.png)

---

## 6. Automatisierung mit Ansible & Vault

Der Zugriff auf das Labor erfolgt nun zentral vom Management-PC (Linux Mint) via Ansible.

### Sicherheits-Infrastruktur (SSH & Vault)
* **SSH-Keys:** Der öffentliche Schlüssel des Management-PCs wurde übertragen (`ssh-copy-id`), um passwortlose Logins zu ermöglichen.
* **Ansible Vault:** Sensible Daten, wie das `sudo`-Passwort des Users `angel`, werden verschlüsselt in `group_vars/all.yml` gespeichert.
* **Vault-Automatik:** Über eine lokale `.vault_pass.txt` und die `ansible.cfg` wurde der Workflow so optimiert, dass keine manuelle Passwortabfrage bei der Ausführung von Playbooks mehr nötig ist.

> **Hier Screenshot einfügen:** (Erfolgreicher Ansible Ping oder `whoami` Testlauf)
> ![Ansible Success](./img/ansible_test_root.png)

### Automatisierte Wartungs-Playbooks
Bisher implementierte Workflows:
1. **`update_system.yml`**: Führt ein vollständiges `apt upgrade` durch.
2. **`check_reboot.yml`**: Prüft, ob die Datei `/var/run/reboot-required` existiert und startet die VM bei Bedarf sicher neu.

> **Hier Screenshot einfügen:** (Ansible Playbook Run ohne Fehler)
> ![Ansible Playbook Run](./img/ansible_playbook_run.png)

---

## Aktueller Projektstatus (Meilenstein 1 erreicht)
- [x] Isolierte Netzwerkumgebung (VLAN 30) aktiv.
- [x] Internetzugriff & DNS über pfSense stabil.
- [x] VM-Management via Proxmox Guest Agent aktiv.
- [x] Vollständige Steuerung über Ansible-Automatisierung etabliert.

## 7. Infrastruktur & Container-Automatisierung (Stand: 11.02.2026)

Nach der erfolgreichen Netzwerkanbindung wurde die VM `ai-ops-01` vollständig automatisiert als **Docker-Host** provisioniert.

### Meilenstein: Infrastructure as Code (IaC)
Der gesamte Setup-Prozess wird über Ansible gesteuert. Dies umfasst:
* **System-Wartung:** Intelligente Prüfung auf notwendige Neustarts (`check_reboot.yml`).
* **Docker-Stack:** Automatisierte Installation der Docker Engine & Docker Compose (`setup_docker.yml`).
* **Security:** Sichere Verwaltung von Datenbank-Passwörtern und API-Keys mittels **Ansible Vault**.

> **Beweis 1: Intelligente Systemwartung**
> Ansible erkennt eigenständig, ob ein Neustart erforderlich ist und überspringt den Task (`skipping`), wenn das System aktuell ist.
> ![Ansible Reboot Check](./img/ansible_reboot_logic.png)

---

### Meilenstein: Container-Host Ready
Die VM ist nun bereit für das Deployment von n8n und KI-Tools. Der Benutzer `angel` wurde berechtigt, Docker-Container ohne `sudo` zu verwalten, was die Sicherheit und den Workflow verbessert.

> **Beweis 2: Erfolgreiche Docker-Provisionierung**
> Der Abschlussbericht des Playbooks zeigt die erfolgreiche Einrichtung aller Komponenten und User-Berechtigungen.
> ![Docker Setup Success](./img/docker_setup_recap.png)

---

## Aktueller Projektstatus
- [x] Statische IP & Guest-Agent konfiguriert.
- [x] Ansible Vault & SSH-Key Authentifizierung aktiv.
- [x] Docker & Docker Compose vollständig einsatzbereit.
- [ ] **Nächster Schritt:** Start des n8n-Stacks (n8n + Postgres).

## 9. Deployment des AI-Workflow-Systems (n8n)

Das Herzstück der Automatisierung, **n8n**, wurde zusammen mit einer **Postgres-Datenbank** als Container-Stack deployed.

### Stack-Details:
* **Orchestrierung:** Docker Compose (via Ansible `community.docker` Collection).
* **Datenhaltung:** Persistente Docker-Volumes für n8n-Daten und Datenbank-Inhalte.
* **Sicherheit:** Dynamische Injektion von Datenbank-Credentials über Ansible Vault während des Deployments.

> **Beweis: Erfolgreiches Stack-Deployment**
> Alle Tasks wurden fehlerfrei ausgeführt, der Stack ist produktiv.
> ![n8n Deployment Success](./screenshots/n8n_deployment_recap.png)

### 9.1 Troubleshooting & Konfigurations-Anpassungen
Während des Deployments wurden spezifische Anpassungen vorgenommen, um den Stack in einer lokalen Entwicklungsumgebung lauffähig zu machen:

* **YAML-Validierung:** Korrektur der Einrückungen im Docker Compose Template, um den Fehler `additional properties not allowed` zu beheben.
* **Security-Override:** Da der Zugriff lokal ohne SSL erfolgt, wurde die Variable `N8N_SECURE_COOKIE=false` gesetzt, um den Login über HTTP zu ermöglichen.
* **Vault-Integration:** Nutzung der Datei `vault_passwords.yml` zur sicheren Injektion der `POSTGRES_PASSWORD` Variable.

### 9.2 Verifizierung des Betriebs
Nach dem Deployment wurde die Erreichbarkeit des Systems erfolgreich geprüft.

| Komponente | Status | URL / Port |
| :--- | :--- | :--- |
| **n8n Frontend** | ✅ Online | `http://192.168.30.20:5678` |
| **Postgres DB** | ✅ Verbunden | Interner Port 5432 |

> **Beweis: n8n Login-Bereitschaft**
> Das System ist bereit für die initiale Account-Erstellung und den ersten Workflow-Bau.
> ![n8n Setup Page](./screenshots/n8n_live_interface.png)

---
## 10. Integration lokaler LLMs (Ollama & Llama3)

Um eine datenschutzkonforme und kostenfreie KI-Verarbeitung zu ermöglichen, wurde das System um eine lokale LLM-Schnittstelle erweitert.

### 10.1 Automatisierte Installation von Ollama
Die Installation von **Ollama** erfolgte über ein dediziertes Ansible-Playbook (`install_ollama.yml`). Dieses übernimmt:
* Den Download und die Installation der Ollama-Binary.
* Die Konfiguration des Systemd-Services für automatischen Start.
* Den initialen Pull des **Llama3** Modells (ca. 4,7 GB).

### 10.2 n8n AI-Agent Konfiguration
In n8n wurde ein intelligenter Workflow erstellt, der die Brücke zwischen dem Automatisierungsserver und der lokalen KI schlägt.

**Konfigurations-Details:**
* **Node-Struktur:** Ein `AI Agent` fungiert als Gehirn, unterstützt durch ein `Ollama Chat Model`.
* **Konnektivität:** Verbindung über die VM-IP auf Port `11434`.
* **Modell:** Nutzung von `llama3:latest`.

> **Beweis: Erfolgreicher KI-Durchlauf**
> Das Bild zeigt den validierten Workflow. Die grünen Indikatoren bestätigen, dass der AI-Agent erfolgreich mit Llama3 kommuniziert und eine Antwort generiert hat.
> ![n8n AI Success](./screenshots/10_n8n_ai_success.png)

---
## 11. Modernisierung der KI-Steuerebene: Open WebUI & MCP (Stand: 23.02.2026)

Nach intensiven Tests wurde die Architektur von OpenClaw auf **Open WebUI** migriert. Dieser Wechsel ermöglicht eine robustere Integration des **Model Context Protocol (MCP)** und eine stabilere Verbindung zu lokalen Inferenz-Engines.

### 11.1 Dezentrale Architektur ("The Control Plane")
Um die Ressourcen des Hypervisors optimal zu nutzen und Management von Workload zu trennen, wurde der Stack geografisch verteilt:

* **Management-Zentrale (VM 102 - Mint):** Beherbergt das Frontend (**Open WebUI**) und das Automatisierungs-Backend (**n8n**).
* **KI-Service-Knoten (ai-ops-01):** Beherbergt die Rechenpower (**Ollama**) und die Schnittstellen-Logik (**MCP-Server & mcpo Bridge**).

### 11.2 Model Context Protocol (MCP) & Bridge-Technologie
Um der KI (Llama3) direkten Zugriff auf die Proxmox-Infrastruktur zu geben, wurde ein MCP-Server auf Basis von `FastMCP` implementiert.

* **Die Herausforderung:** Open WebUI erwartet eine standardisierte OpenAPI/REST-Schnittstelle, während MCP nativ über SSE (Server-Sent Events) kommuniziert.
* **Die Lösung:** Einsatz der **`mcpo` Bridge**. Diese startet das MCP-Python-Skript als Subprozess (stdio) und übersetzt die Tools in eine dynamische `openapi.json` auf Port `5002`.
* **Security-Patch:** Da FastMCP restriktive Host-Checks durchführt, wurde ein Python-Monkey-Patch implementiert, der die `dns_rebinding_protection` deaktiviert. Dies erlaubt den Zugriff aus dem Management-VLAN (Mint-VM) auf den Docker-Host.



---

## 12. Finaler AI-Ops Stack: Status & Validierung

Der gesamte Stack ist nun vollständig über Ansible (`deploy_ai_brain.yml`) provisioniert und nutzt verschlüsselte Secrets aus der `vault_passwords.yml`.

### 12.1 Komponenten-Matrix
| Dienst | Port | Host | Rolle |
| :--- | :--- | :--- | :--- |
| **Open WebUI** | 8080 | Mint-VM (102) | Zentrales Chat-Interface & Tool-Nutzung |
| **n8n** | 5678 | Mint-VM (102) | Event-Handling & Workflow-Automatisierung |
| **mcpo (Bridge)** | 5002 | ai-ops-01 | REST-Übersetzer für Proxmox-Tools |
| **Ollama** | 11434 | ai-ops-01 | Lokale LLM-Inferenz (Llama3) |
| **Proxmox API** | 8006 | Host (WTR Pro) | Ziel-Infrastruktur für die KI-Steuerung |

### 12.2 Validierung der Konnektivität
Die Funktionalität der "Bridge" wurde durch Abfrage der generierten Spezifikation verifiziert:
`curl http://192.168.1.10:5002/openapi.json | python3 -m json.tool`

**Ergebnis:** Die KI verfügt nun über "Hände" im Homelab. Sie kann autonom:
1.  VM-Listen vom Proxmox-Host abrufen.
2.  Den Ressourcenstatus (CPU/RAM) der Nodes analysieren.
3.  VMs starten oder stoppen basierend auf natürlicher Sprache.

---

## 13. Roadmap: Nächste Schritte

- [ ] **pfSense Integration:** n8n-Workflows zur Analyse von Firewall-Logs durch die KI.
- [ ] **Self-Healing:** Automatisierte Snapshots vor kritischen KI-Aktionen.
- [ ] **GitOps:** Versionierung aller MCP-Skripte und Ansible-Playbooks in einem lokalen Repository.
