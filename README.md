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
