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

## Architekturübersicht

Die gesamte Infrastruktur basiert auf einem einzelnen Proxmox-Host.
pfSense läuft als VM und übernimmt Routing, Firewalling und Netzwerksegmentierung.

### Architekturdiagramm

**Screenshot hier einfügen:**
