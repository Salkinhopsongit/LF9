# 2. Netzwerk-Topologie - Streamline IPv6

## 🌍 Geografische Verteilung

```
┌─────────────────────────────────────────────────────────┐
│                    STREAMLINE NETZWERK                  │
├─────────────────────────────────────────────────────────┤
│                                                         │
│  Hamburg (Hauptsitz)      Berlin (Serverraum)         │
│  ┌──────────────────┐     ┌──────────────────┐        │
│  │   Router-HH      │────→│    Router-BE     │        │
│  │ (1.1.1.1)        │     │   (3.3.3.3)      │        │
│  └────────┬─────────┘     └─────────┬────────┘        │
│           │                         │                  │
│      LAN: 10/64                LAN: 30/64              │
│      DHCP: SLAAC              Services               │
│           │                         │                  │
│           ↓                         ↓                  │
│  ┌──────────────────┐     ┌──────────────────┐        │
│  │  Webserver       │     │   DNS/Mail       │        │
│  │  SSH Server      │     │   NTP Server     │        │
│  │  PCs (DHCP)      │     │   Backup-Server  │        │
│  └──────────────────┘     └──────────────────┘        │
│                                                         │
│           ↖ OSPFv3 Verbindung ↗                       │
│                                                         │
│  Lübeck (Zweigstelle)      Backup (Optional)          │
│  ┌──────────────────┐     ┌──────────────────┐        │
│  │   Router-LB      │────→│    Router-BK     │        │
│  │ (2.2.2.2)        │     │   (4.4.4.4)      │        │
│  └────────┬─────────┘     └─────────┬────────┘        │
│           │                         │                  │
│      LAN: 20/64                LAN: 40/64              │
│      DHCP: SLAAC              Standby                 │
│           │                         │                  │
│           ↓                         ↓                  │
│  ┌──────────────────┐     ┌──────────────────┐        │
│  │  PCs (DHCP)      │     │   Failover       │        │
│  │  Printer         │     │   Services       │        │
│  │  Peripherie      │     │                  │        │
│  └──────────────────┘     └──────────────────┘        │
│                                                         │
│         ↖ OSPFv3 Verbindung ↗                         │
│                                                         │
└─────────────────────────────────────────────────────────┘
```

## 📡 WAN-Verbindungen

### OSPFv3-Backbone (Area 0)

**Verbindungen (alle Punkt-zu-Punkt):**

| Von | Zu | Link-Subnetz | Protokoll | Redundanz |
|-----|----|--------------|-----------|---------:|
| Router-HH | Router-BE | 2001:db8:100:1::/64 | OSPFv3 | Primary |
| Router-HH | Router-LB | 2001:db8:100:2::/64 | OSPFv3 | Primary |
| Router-LB | Router-BE | 2001:db8:100:3::/64 | OSPFv3 | Secondary |
| Router-BE | Router-BK | 2001:db8:100:4::/64 | OSPFv3 | Backup |

---

## 🏗️ Detaillierte Topologie pro Standort

### Standort: Hamburg (Hauptsitz)

```
┌─────────────────────────────────────────┐
│          HAMBURG NETZWERK               │
├─────────────────────────────────────────┤
│                                         │
│          ┌──────────────────┐          │
│          │   Router-HH      │          │
│          │ Router-ID: 1.1.1.1         │
│          │ Ge0/0-Ge0/3: WAN │          │
│          │ Ge0/1: LAN       │          │
│          └────────┬─────────┘          │
│                   │                    │
│        2001:db8:10::/64               │
│        (SLAAC + DHCPv6)               │
│                   │                    │
│    ┌──────────────┼──────────────┐    │
│    │              │              │    │
│    ↓              ↓              ↓    │
│ ┌─────┐      ┌─────┐      ┌─────┐   │
│ │PC-1 │      │PC-2 │      │PC-3 │   │
│ │DHCP │      │DHCP │      │DHCP │   │
│ └─────┘      └─────┘      └─────┘   │
│                                         │
│ ┌──────────────────────────────────┐   │
│ │   Webserver (2001:db8:10::100)   │   │
│ │   SSH Server (2001:db8:10::101)  │   │
│ │   Port 22, 80, 443 offen         │   │
│ └──────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

**Eigenschaften:**
- **LAN:** 2001:db8:10::/64
- **DHCP-Mode:** SLAAC with DHCPv6-Server
- **Optionale DNS:** 2001:db8:30::53 (Berlin)
- **Services:** Web, SSH, Dateifreigabe
- **ACL:** SSH & HTTP/HTTPS erlaubt

---

### Standort: Lübeck (Zweigstelle)

```
┌─────────────────────────────────────────┐
│          LÜBECK NETZWERK                │
├─────────────────────────────────────────┤
│                                         │
│          ┌──────────────────┐          │
│          │   Router-LB      │          │
│          │ Router-ID: 2.2.2.2         │
│          │ Ge0/0-Ge0/3: WAN │          │
│          │ Ge0/1: LAN       │          │
│          └────────┬─────────┘          │
│                   │                    │
│        2001:db8:20::/64               │
│        (SLAAC + DHCPv6)               │
│                   │                    │
│    ┌──────────────┼──────────────┐    │
│    │              │              │    │
│    ↓              ↓              ↓    │
│ ┌─────┐      ┌─────┐      ┌─────┐   │
│ │PC-4 │      │PC-5 │      │PC-6 │   │
│ │DHCP │      │DHCP │      │DHCP │   │
│ └─────┘      └─────┘      └─────┘   │
│                                         │
│ ┌──────────────────────────────────┐   │
│ │   Drucker (2001:db8:20::150)     │   │
│ │   Netzwerk-Speicher              │   │
│ │   Remote-Access Port 443         │   │
│ └──────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

**Eigenschaften:**
- **LAN:** 2001:db8:20::/64
- **DHCP-Mode:** SLAAC with DHCPv6-Server
- **DNS:** Von Hamburg remote
- **Services:** Remote-Access (HTTPS nur)
- **ACL:** HTTPS erlaubt, HTTP DENY

---

### Standort: Berlin (Serverraum)

```
┌─────────────────────────────────────────┐
│          BERLIN NETZWERK                │
├─────────────────────────────────────────┤
│                                         │
│          ┌──────────────────┐          │
│          │   Router-BE      │          │
│          │ Router-ID: 3.3.3.3         │
│          │ Ge0/0-Ge0/3: WAN │          │
│          │ Ge0/1: Services  │          │
│          └────────┬─────────┘          │
│                   │                    │
│        2001:db8:30::/64               │
│        (Statische Adressen)           │
│                   │                    │
│    ┌──────────────┼──────────────┐    │
│    │              │              │    │
│    ↓              ↓              ↓    │
│ ┌──────┐    ┌──────┐        ┌──────┐ │
│ │DNS   │    │Mail  │        │NTP   │ │
│ │::53  │    │::25  │        │::123 │ │
│ └──────┘    └──────┘        └──────┘ │
│                                         │
│ ┌──────────────────────────────────┐   │
│ │   RADIUS Server (Auth)           │   │
│ │   Backup Services                │   │
│ │   Zentrale DHCPv6 (optional)     │   │
│ └──────────────────────────────────┘   │
│                                         │
└─────────────────────────────────────────┘
```

**Eigenschaften:**
- **LAN:** 2001:db8:30::/64
- **DHCP-Mode:** Für Server statisch
- **Services:** DNS, Mail, NTP, RADIUS
- **Erreichbarkeit:** Von allen Standorten

---

## 🔄 Routing-Design

### OSPFv3-Area-Struktur

```
Area 0 (Backbone)
│
├─ Router-HH (1.1.1.1)
│  └─ Network 2001:db8:10::/64
│
├─ Router-LB (2.2.2.2)
│  └─ Network 2001:db8:20::/64
│
├─ Router-BE (3.3.3.3)
│  └─ Network 2001:db8:30::/64
│
└─ Router-BK (4.4.4.4)
   └─ Network 2001:db8:40::/64
```

### Redundante Pfade

**Primary-Pfad:** HH → BE (direkt)  
**Secondary-Pfad:** HH → LB → BE  
**Backup-Pfad:** HH → BE → BK  

---

## 📊 IP-Adressierungsübersicht

| Bereich | CIDR | Verwendung | Status |
|---------|------|-----------|--------|
| 2001:db8:10::/64 | Hamburg LAN | Clients + DHCP | Active |
| 2001:db8:20::/64 | Lübeck LAN | Clients + DHCP | Active |
| 2001:db8:30::/64 | Berlin LAN | Services | Static |
| 2001:db8:40::/64 | Backup LAN | Failover | Static |
| 2001:db8:100::/62 | WAN Link 1 | HH↔BE | Active |
| 2001:db8:100:1::/64 | WAN Link 2 | HH↔LB | Active |
| 2001:db8:100:2::/64 | WAN Link 3 | LB↔BE | Active |
| 2001:db8:100:3::/64 | WAN Link 4 | BE↔BK | Standby |

---

## 🛡️ Sicherheitsperimeter

```
┌────────────────────────────────────────┐
│        SECURITY BOUNDARIES             │
├────────────────────────────────────────┤
│                                        │
│  Externe (Internet)                    │
│        ↓ Firewall                      │
│  ┌──────────────────────────────────┐  │
│  │ WAN - Router Perimeter           │  │
│  │ - SSH: Hamburg only              │  │
│  │ - HTTP/HTTPS: Differentiated    │  │
│  └──────────────────────────────────┘  │
│        ↓ ACL                           │
│  ┌──────────────────────────────────┐  │
│  │ Interne Networks                 │  │
│  │ - Hamburg: Voll Zugriff          │  │
│  │ - Lübeck: HTTPS only             │  │
│  │ - Berlin: Zentrale Services      │  │
│  └──────────────────────────────────┘  │
│        ↓ OSPFv3-Routing              │
│  ┌──────────────────────────────────┐  │
│  │ Backend Services                 │  │
│  │ - Statische IP-Adressen          │  │
│  │ - Privilegierter Zugriff         │  │
│  └──────────────────────────────────┘  │
│                                        │
└────────────────────────────────────────┘
```

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-06-03
