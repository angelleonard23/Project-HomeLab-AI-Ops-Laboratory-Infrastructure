# Project HomeLab – Proxmox & pfSense Security Lab

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

