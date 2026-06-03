# 6. IPAM Dokumentation - Streamline IPv6

## 📋 IPAM-Übersicht (IP Address Management)

**IPAM** = Zentrales System zur Verwaltung aller IP-Adressen

```
┌────────────────────────────────────────────────┐
│         IPAM - STREAMLINE NETZWERK            │
├────────────────────────────────────────────────┤
│                                                │
│ Verwaltete Objekte:                           │
│ ├── IPv6 Subnets                             │
│ ├── DHCPv6 Pools                             │
│ ├── Statische IP-Adressen                    │
│ ├── DNS-Einträge                             │
│ ├── Reservierte Adressen                     │
│ └── RADIUS-Server                            │
│                                                │
│ Redundanz: Zentrale Datenbank in Berlin      │
│ Backup: Automatische Synchronisation         │
│                                                │
└────────────────────────────────────────────────┘
```

---

## 🗂️ IPv6 Adressierungsplan

### Gesamtstruktur

```
Root: 2001:db8::/32
│
├── LAN-Bereiche (2001:db8:10::/48)
│   ├── Hamburg:  2001:db8:10::/64
│   ├── Lübeck:   2001:db8:20::/64
│   └── Berlin:   2001:db8:30::/64
│
├── Backup-Bereich (2001:db8:40::/48)
│   └── Failover: 2001:db8:40::/64
│
└── WAN-Punkt-zu-Punkt (2001:db8:100::/48)
    ├── Link 1: 2001:db8:100:1::/64
    ├── Link 2: 2001:db8:100:2::/64
    ├── Link 3: 2001:db8:100:3::/64
    └── Link 4: 2001:db8:100:4::/64
```

---

## 🏢 Standort: Hamburg

### LAN-Bereich: 2001:db8:10::/64

#### Subnetz-Details

| Parameter | Wert |
|-----------|------|
| **Netzwerk** | 2001:db8:10::/64 |
| **Usable Start** | 2001:db8:10::1 |
| **Usable End** | 2001:db8:10::ffff:ffff:ffff:ffff |
| **Broadcast** | 2001:db8:10::ffff:ffff:ffff:ffff |
| **Größe** | 2^64 Adressen |
| **Type** | LAN (Client Network) |
| **DHCP-Mode** | SLAAC with DHCPv6 |
| **DHCPv6-Pool** | 2001:db8:10::300/128 |
| **Status** | Active |

#### Statische IP-Adressen

| Name | IPv6-Adresse | MAC-Adresse | Funktion | Notizen |
|------|--------------|-------------|----------|----------|
| Router-HH | 2001:db8:10::1 | 00:00:00:00:00:01 | Gateway | Router-ID: 1.1.1.1 |
| Webserver | 2001:db8:10::100 | 00:00:00:00:00:10 | Web/SSH | Port 22, 80, 443 |
| DNS-Relay | 2001:db8:10::102 | 00:00:00:00:00:12 | DNS Cache | Zeigt auf 2001:db8:30::53 |
| NTP-Relay | 2001:db8:10::103 | 00:00:00:00:00:13 | Zeitsync | Zeigt auf 2001:db8:30::123 |

#### DHCP Assignments (dynamisch via SLAAC+Server)

```
DHCPv6-Pool-Bereich: 2001:db8:10::300/128 und höher

Client-Adressierung (SLAAC-Suffix):
├── PC-1:      2001:db8:10::xxxx:xxxx:xxxx:xxxx (SLAAC generiert)
├── PC-2:      2001:db8:10::xxxx:xxxx:xxxx:xxxx (SLAAC generiert)
└── PC-3:      2001:db8:10::xxxx:xxxx:xxxx:xxxx (SLAAC generiert)

Von DHCPv6 Server erhältlich:
├── DNS-Server: 2001:db8:30::53
├── Domain:     hamburg.streamline.local
├── Lease:      1 Stunde (3600 Sekunden)
└── Preferred:  30 Minuten (1800 Sekunden)
```

#### SHOULD-HAVE: Stateful DHCPv6 Mode (Alternative)

```
!!! OPTIONAL - nur für zentralisierte Verwaltung !!!

Pool: Hamburg-Stateful-Pool
├── Präfix:     2001:db8:10::300/128
├── Modus:      Stateful (alle Adressen zentral verwaltet)
├── Lease:      24 Stunden (86400 Sekunden)
├── Preferred:  12 Stunden (43200 Sekunden)
├── DNS:        2001:db8:30::53 (Berlin)
├── Domain:     hamburg.streamline.local
└── Status:     Konfiguriert, nicht aktiv (nutzen nur bei Bedarf)

RA-Flags im Stateful-Modus:
├── A-Flag: 0 (Clients nutzen NICHT SLAAC)
├── M-Flag: 1 (Stateful DHCPv6 ERFORDERLICH)
└── O-Flag: 0 (DHCPv6-Server optional)
```

---

## 🏢 Standort: Lübeck

### LAN-Bereich: 2001:db8:20::/64

#### Subnetz-Details

| Parameter | Wert |
|-----------|------|
| **Netzwerk** | 2001:db8:20::/64 |
| **Usable Start** | 2001:db8:20::1 |
| **Gateway** | 2001:db8:20::1 |
| **Type** | LAN (Branch Office) |
| **DHCP-Mode** | SLAAC with DHCPv6 |
| **Status** | Active |

#### Statische IP-Adressen

| Name | IPv6-Adresse | MAC-Adresse | Funktion | Notizen |
|------|--------------|-------------|----------|----------|
| Router-LB | 2001:db8:20::1 | 00:00:00:00:00:02 | Gateway | Router-ID: 2.2.2.2 |
| Drucker | 2001:db8:20::150 | 00:00:00:00:00:50 | Peripherie | Port 9100 |
| Netzwerk-Speicher | 2001:db8:20::151 | 00:00:00:00:00:51 | NAS | Port 445, 139 |

#### DHCP Assignments

```
DHCPv6-Pool-Bereich: 2001:db8:20::300/128 und höher

Client-Adressierung (SLAAC-Suffix):
├── PC-4:      2001:db8:20::xxxx:xxxx:xxxx:xxxx (SLAAC)
├── PC-5:      2001:db8:20::xxxx:xxxx:xxxx:xxxx (SLAAC)
└── PC-6:      2001:db8:20::xxxx:xxxx:xxxx:xxxx (SLAAC)

Von DHCPv6 Server erhältlich:
├── DNS-Server: 2001:db8:30::53 (Berlin)
├── Domain:     luebeck.streamline.local
├── Lease:      1 Stunde (3600 Sekunden)
└── Preferred:  30 Minuten (1800 Sekunden)
```

---

## 🏢 Standort: Berlin (Zentrales Serverraum)

### LAN-Bereich: 2001:db8:30::/64

#### Subnetz-Details

| Parameter | Wert |
|-----------|------|
| **Netzwerk** | 2001:db8:30::/64 |
| **Type** | LAN (Data Center) |
| **Adressierung** | Statisch (keine DHCP) |
| **Status** | Active |

#### Statische IP-Adressen

| Name | IPv6-Adresse | MAC-Adresse | Funktion | Notizen |
|------|--------------|-------------|----------|----------|
| Router-BE | 2001:db8:30::1 | 00:00:00:00:00:03 | Gateway | Router-ID: 3.3.3.3 |
| **DNS Server** | 2001:db8:30::53 | 00:00:00:00:00:30 | DNS | Port 53, Zentral |
| **Mail Server** | 2001:db8:30::25 | 00:00:00:00:00:31 | SMTP | Port 25, Zentral |
| **NTP Server** | 2001:db8:30::123 | 00:00:00:00:00:32 | Time Sync | Port 123, Zentral |
| **RADIUS Server** | 2001:db8:30::1645 | 00:00:00:00:00:33 | Auth | Port 1645/1812 |
| **DHCP Relay Agent** | 2001:db8:30::67 | 00:00:00:00:00:34 | DHCPv6 Relay | (CouldHave) |

#### Zentrale Dienste

```
DNS Server (2001:db8:30::53)
├── Port: 53 (UDP/TCP)
├── Domänen:
│   ├── hamburg.streamline.local → 2001:db8:10::*
│   ├── luebeck.streamline.local → 2001:db8:20::*
│   ├── berlin.streamline.local → 2001:db8:30::*
│   └── backup.streamline.local → 2001:db8:40::*
├── Autoritativ für: streamline.local
└── Forwarder zu: Public DNS (optional)

Mail Server (2001:db8:30::25)
├── Port: 25 (SMTP)
├── Domäne: streamline.local
├── DNSSEC: Empfohlen
└── SPF/DKIM Records: Erforderlich

NTP Server (2001:db8:30::123)
├── Port: 123 (UDP)
├── Stratum: 2 oder besser
├── Clients: Alle Router (via OSPFv3)
└── Upstream: (optional) public.ntp.pool

RADIUS Server (2001:db8:30::1645 & 1812)
├── Auth-Port: 1645/1812
├── Accounting-Port: 1646/1813
├── Shared-Secret: (konfigurieren)
├── Clients: Alle Router
└── Benutzer: (optional) Administrators
```

---

## 🏢 Standort: Backup (Optional)

### LAN-Bereich: 2001:db8:40::/64

#### Subnetz-Details

| Parameter | Wert |
|-----------|------|
| **Netzwerk** | 2001:db8:40::/64 |
| **Type** | LAN (Failover/Backup) |
| **Adressierung** | Statisch (Standby) |
| **Status** | Standby |

#### Statische IP-Adressen

| Name | IPv6-Adresse | MAC-Adresse | Funktion | Notizen |
|------|--------------|-------------|----------|----------|
| Router-BK | 2001:db8:40::1 | 00:00:00:00:00:04 | Gateway | Router-ID: 4.4.4.4 |
| Failover-Services | 2001:db8:40::100 | 00:00:00:00:00:40 | Redundanz | Spiegelt Berlin |

---

## 📡 WAN-Punkt-zu-Punkt Links (2001:db8:100::/48)

### Link 1: Hamburg ↔ Berlin (Primary)

| Parameter | Wert |
|-----------|------|
| **Subnetz** | 2001:db8:100:1::/64 |
| **Router-HH Interface** | 2001:db8:100:1::1/64 |
| **Router-BE Interface** | 2001:db8:100:1::2/64 |
| **OSPFv3 Cost** | 100 |
| **Protokoll** | OSPFv3 P2P |
| **Status** | Primary (Active) |
| **Redundanz** | Ja (via Lübeck) |

### Link 2: Hamburg ↔ Lübeck (Primary)

| Parameter | Wert |
|-----------|------|
| **Subnetz** | 2001:db8:100:2::/64 |
| **Router-HH Interface** | 2001:db8:100:2::1/64 |
| **Router-LB Interface** | 2001:db8:100:2::2/64 |
| **OSPFv3 Cost** | 100 |
| **Protokoll** | OSPFv3 P2P |
| **Status** | Primary (Active) |
| **Redundanz** | Ja (via Berlin) |

### Link 3: Lübeck ↔ Berlin (Secondary)

| Parameter | Wert |
|-----------|------|
| **Subnetz** | 2001:db8:100:3::/64 |
| **Router-LB Interface** | 2001:db8:100:3::1/64 |
| **Router-BE Interface** | 2001:db8:100:3::2/64 |
| **OSPFv3 Cost** | 100 |
| **Protokoll** | OSPFv3 P2P |
| **Status** | Secondary (Active) |
| **Redundanz** | Ja (Backup-Pfad) |

### Link 4: Berlin ↔ Backup (Backup)

| Parameter | Wert |
|-----------|------|
| **Subnetz** | 2001:db8:100:4::/64 |
| **Router-BE Interface** | 2001:db8:100:4::1/64 |
| **Router-BK Interface** | 2001:db8:100:4::2/64 |
| **OSPFv3 Cost** | 50 |
| **Protokoll** | OSPFv3 P2P |
| **Status** | Backup (Standby) |
| **Redundanz** | Ja (für Berlin-Failover) |

---

## 🌐 COULD-HAVE: Zentrale DHCPv6-Lösung

### Architektur: Berlin als zentraler DHCPv6-Server

```
┌────────────────────────────────────────────────┐
│     ZENTRALE DHCPV6 ARCHITEKTUR               │
├────────────────────────────────────────────────┤
│                                                │
│  Hamburg-Router              Lübeck-Router    │
│  (DHCPv6 Relay Agent)       (DHCPv6 Relay)   │
│         │                          │           │
│         └──────────────┬───────────┘           │
│                        │                       │
│              ipv6 dhcp relay destination      │
│              2001:db8:30::67                  │
│                        ↓                       │
│            Berlin-Router/Server                │
│            (DHCPv6 Server zentral)            │
│            (Pool Management)                  │
│            (IPAM Integration)                 │
│                                                │
└────────────────────────────────────────────────┘
```

### Zentrale DHCPv6 Pools

#### Pool 1: Hamburg Central

```
Pool-Name:      Hamburg-Central-DHCP
Netzwerk:       2001:db8:10::/64
Präfix-Start:   2001:db8:10::300
Präfix-End:     2001:db8:10::ffff:ffff:ffff:ffff
Lease-Time:     86400 Sekunden (24 Stunden)
Preferred:      43200 Sekunden (12 Stunden)
DNS-Server:     2001:db8:30::53
Domain:         hamburg.streamline.local
Router:         2001:db8:10::1 (Hamburg-Router)
Status:         Konfiguriert (Could-have)
```

#### Pool 2: Lübeck Central

```
Pool-Name:      Luebeck-Central-DHCP
Netzwerk:       2001:db8:20::/64
Präfix-Start:   2001:db8:20::300
Präfix-End:     2001:db8:20::ffff:ffff:ffff:ffff
Lease-Time:     86400 Sekunden (24 Stunden)
Preferred:      43200 Sekunden (12 Stunden)
DNS-Server:     2001:db8:30::53
Domain:         luebeck.streamline.local
Router:         2001:db8:20::1 (Lübeck-Router)
Status:         Konfiguriert (Could-have)
```

### Konfigurationsschritte (Dokumentation für reale Umgebung)

```cisco
! SCHRITT 1: Berlin-Router als DHCPv6-Server konfigurieren
Router-BE(config)# ipv6 dhcp pool Central-Hamburg
Router-BE(config-dhcp)# address prefix 2001:db8:10::300/128 lifetime 86400 43200
Router-BE(config-dhcp)# dns-server 2001:db8:30::53
Router-BE(config-dhcp)# domain-name hamburg.streamline.local
!

Router-BE(config)# ipv6 dhcp pool Central-Luebeck
Router-BE(config-dhcp)# address prefix 2001:db8:20::300/128 lifetime 86400 43200
Router-BE(config-dhcp)# dns-server 2001:db8:30::53
Router-BE(config-dhcp)# domain-name luebeck.streamline.local
!

! SCHRITT 2: Hamburg-Router als DHCPv6-Relay konfigurieren
Router-HH(config)# interface GigabitEthernet 0/1
Router-HH(config-if)# ipv6 dhcp relay destination 2001:db8:30::67
! (ACHTUNG: In Cisco Packet Tracer NICHT UNTERSTÜTZT)
!

! SCHRITT 3: Lübeck-Router als DHCPv6-Relay konfigurieren
Router-LB(config)# interface GigabitEthernet 0/1
Router-LB(config-if)# ipv6 dhcp relay destination 2001:db8:30::67
! (ACHTUNG: In Cisco Packet Tracer NICHT UNTERSTÜTZT)
!

! SCHRITT 4: Berlin-Router DHCPv6-Server auf Interface aktivieren
Router-BE(config)# interface GigabitEthernet 0/1
Router-BE(config-if)# ipv6 dhcp server Central-Hamburg
Router-BE(config-if)# ipv6 dhcp server Central-Luebeck
!
```

### Vorteil der zentralen Lösung

```
✓ Einheitliche Verwaltung aller DHCP-Pools
✓ Zentrale IPAM-Datenbank
✓ Leichtere Audits und Compliance
✓ Konsistente Policies über alle Standorte
✓ Bessere Nachverfolgbarkeit
✓ Failover zu Backup-Pool möglich

✗ Abhängigkeit von Berlin-Router
✗ Höherer initiale Konfigurationsaufwand
✗ Nur in realen Umgebungen möglich (nicht in PT)
```

---

## 📊 IPAM-Verwaltungsmatrix

### Subnetz-Übersicht

| ID | Subnetz | Präfix | Zweck | DHCP-Modus | Status |
|----|---------|--------|-------|-----------|--------|
| 1 | Hamburg LAN | 2001:db8:10::/64 | Clients | SLAAC+Server | Active |
| 2 | Lübeck LAN | 2001:db8:20::/64 | Clients | SLAAC+Server | Active |
| 3 | Berlin LAN | 2001:db8:30::/64 | Services | Statisch | Active |
| 4 | Backup LAN | 2001:db8:40::/64 | Failover | Statisch | Standby |
| 5 | WAN Link 1 | 2001:db8:100:1::/64 | HH-BE | Statisch | Active |
| 6 | WAN Link 2 | 2001:db8:100:2::/64 | HH-LB | Statisch | Active |
| 7 | WAN Link 3 | 2001:db8:100:3::/64 | LB-BE | Statisch | Active |
| 8 | WAN Link 4 | 2001:db8:100:4::/64 | BE-BK | Statisch | Standby |

### Reservierte Adressbereiche

```
2001:db8:10::/64 (Hamburg)
├── 2001:db8:10::1        → Router Gateway [RESERVIERT]
├── 2001:db8:10::100      → Webserver [RESERVIERT]
├── 2001:db8:10::102      → DNS Relay [RESERVIERT]
├── 2001:db8:10::103      → NTP Relay [RESERVIERT]
├── 2001:db8:10::200-299  → Reserviert für Erweiterung
└── 2001:db8:10::300+     → DHCPv6 Client Pool [DYNAMISCH]

2001:db8:20::/64 (Lübeck)
├── 2001:db8:20::1        → Router Gateway [RESERVIERT]
├── 2001:db8:20::150      → Drucker [RESERVIERT]
├── 2001:db8:20::151      → Netzwerk-Speicher [RESERVIERT]
├── 2001:db8:20::200-299  → Reserviert für Erweiterung
└── 2001:db8:20::300+     → DHCPv6 Client Pool [DYNAMISCH]

2001:db8:30::/64 (Berlin)
├── 2001:db8:30::1        → Router Gateway [RESERVIERT]
├── 2001:db8:30::53       → DNS Server [RESERVIERT]
├── 2001:db8:30::25       → Mail Server [RESERVIERT]
├── 2001:db8:30::123      → NTP Server [RESERVIERT]
├── 2001:db8:30::1645     → RADIUS Server Auth [RESERVIERT]
├── 2001:db8:30::1812     → RADIUS Server Alt [RESERVIERT]
├── 2001:db8:30::67       → DHCPv6 Relay (CouldHave) [RESERVIERT]
└── 2001:db8:30::200-999  → Reserviert für weitere Services

2001:db8:40::/64 (Backup)
├── 2001:db8:40::1        → Router Gateway [RESERVIERT]
└── 2001:db8:40::100      → Failover Services [RESERVIERT]
```

---

## ✅ IPAM Validierungschecklist

### Subnetz-Überprüfung
- [ ] Alle Subnets eindeutig
- [ ] Keine Überlappungen
- [ ] Größen angemessen
- [ ] Reservierungen dokumentiert
- [ ] DHCP-Pools definiert

### DNS-Integration
- [ ] Reverse DNS Zonen konfiguriert
- [ ] A/AAAA Records konsistent
- [ ] PTR Records aktuell
- [ ] DNS Redundanz vorhanden

### DHCP-Funktionalität
- [ ] SLAAC funktioniert
- [ ] DHCPv6 Pool antwortet
- [ ] Leasing korrekt
- [ ] Renewal funktioniert
- [ ] Stateful DHCPv6 (Hamburg, optional) getestet
- [ ] Zentrale DHCPv6 (Berlin, could-have) dokumentiert

### Dokumentation
- [ ] IPAM aktuell
- [ ] Alle Adressen eingetragen
- [ ] Versionskontrolle aktiv
- [ ] Change-Log gepflegt
- [ ] Backups vorhanden

---

## 📝 Konfigurationspflege

### Wöchentliche Aufgaben
- [ ] DHCP-Pool-Auslastung überprüfen
- [ ] DNS-Einträge auf Konsistenz prüfen
- [ ] Logfiles auf Fehler analysieren

### Monatliche Aufgaben
- [ ] IPAM-Datenbank aktualisieren
- [ ] Backup der Konfigurationen
- [ ] Audit-Report generieren

### Quartalsmäßige Aufgaben
- [ ] Capacity Planning Review
- [ ] Disaster Recovery Test
- [ ] Sicherheits-Audit

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-06-03  
**Verwaltung:** Zentrale IPAM in Berlin  
**Redundanz:** Automatische Synchronisation  
**Status:** Active (mit Optional-Features dokumentiert)
