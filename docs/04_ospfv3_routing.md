# 4. OSPFv3 Routing Konfiguration - Streamline

## 🌐 OSPFv3 Überblick

### Was ist OSPFv3?

**OSPFv3** = OSPF version 3 für IPv6

```
┌─────────────────────────────────────┐
│      OSPFv3 Charakteristiken        │
├─────────────────────────────────────┤
│ Protokoll:   Link-State Routing     │
│ Metrik:      Kosten (1-65535)       │
│ Konvergenz:  Schnell (< 1s)         │
│ Skalierung:  Große Netzwerke        │
│ HOP-Limit:   Keine (Metrik-basiert) │
│ Prozess-ID:  42 (Streamline)        │
│ Area:        0 (Backbone)           │
│ Router-IDs:  1.1.1.1 - 4.4.4.4      │
└─────────────────────────────────────┘
```

### Vorteile gegenüber statischem Routing

| Aspekt | Statisch | OSPFv3 |
|--------|----------|--------|
| Redundanz | Manuell | Automatisch |
| Ausfallsicherheit | Keine | Built-in |
| Skalierung | Gering | Sehr hoch |
| Verwaltung | Aufwändig | Automatisiert |
| Pfadberechnung | Fest | Dynamisch |
| Konvergenz | Keine | < 1 Sekunde |
| Load-Balancing | Nein | Ja (ECMP) |

---

## 🏗️ Streamline OSPFv3 Topologie

```
┌──────────────────────────────────────────────────┐
│        OSPFV3 AREA 0 (BACKBONE)                 │
│        Prozess-ID: 42                           │
├──────────────────────────────────────────────────┤
│                                                  │
│      Router-HH (1.1.1.1)    Router-BE (3.3.3.3)│
│      ├─ Network: 2001:db8:10::/64             │
│      │                                          │
│      │  Link: 2001:db8:100:1::/64              │
│      │  (Primary)                              │
│      └─────────────────────────────┐           │
│             ↙         ↘             │           │
│        /              \            │           │
│       /    ↖ OSPFv3 ↗  \           │           │
│      /                   \          │           │
│  ┌──────────────────────┐  ┌──────────────┐   │
│  │  Link: 100:2::/64    │  │ Link: 100:4::│   │
│  │  (Primary HH-LB)     │  │ /64 (BE-BK)  │   │
│  │  (Secondary LB-BE)   │  │ (Backup)     │   │
│  └────────┬─────────────┘  └──────────────┘   │
│           │                        │            │
│    Router-LB (2.2.2.2)    Router-BK (4.4.4.4) │
│    └─ Network: 2001:db8:20::/64              │
│       └─ Network: 2001:db8:40::/64           │
│                                                  │
└──────────────────────────────────────────────────┘
```

---

## 📡 WAN-Links (Punkt-zu-Punkt)

### Link-Übersicht

| Link | Von | Zu | Subnetz | Kosten | Status |
|------|-----|----|---------|--------|--------|
| 1 | Router-HH | Router-BE | 2001:db8:100:1::/64 | 100 | Primary |
| 2 | Router-HH | Router-LB | 2001:db8:100:2::/64 | 100 | Primary |
| 3 | Router-LB | Router-BE | 2001:db8:100:3::/64 | 100 | Secondary |
| 4 | Router-BE | Router-BK | 2001:db8:100:4::/64 | 50 | Backup |

---

## ⚙️ Router-Konfiguration

### MUST-HAVE: Router-HH (Hamburg) - OSPFv3 Setup

```cisco
! Router-HH: OSPFv3 Konfiguration
! ================================

configure terminal

! 1. IPv6 Unicast Routing aktivieren
ipv6 unicast-routing

! 2. OSPFv3 Prozess starten (ID: 42)
router ospfv3 42
  ! Router-ID setzen (1.1.1.1)
  router-id 1.1.1.1
  
  ! Area 0 definieren
  area 0
!

! 3. LAN-Interface (Ge0/1) OSPFv3 aktivieren
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:10::1/64
  ipv6 ospf 42 area 0
  ! Netzwerk-Typ auf Broadcast
  ipv6 ospf network broadcast
  ! Kosten (optional, Priorität für LAN)
  ipv6 ospf cost 1
  no shutdown
!

! 4. WAN-Link zu Router-BE (Ge0/0) - Punkt-zu-Punkt
interface GigabitEthernet 0/0
  ipv6 address 2001:db8:100:1::1/64
  ipv6 ospf 42 area 0
  ! Punkt-zu-Punkt Netzwerk-Typ
  ipv6 ospf network point-to-point
  ! Kosten für Link (100 = Standard)
  ipv6 ospf cost 100
  ! Hello & Dead Interval für schnelle Konvergenz
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

! 5. WAN-Link zu Router-LB (Ge0/2) - Punkt-zu-Punkt
interface GigabitEthernet 0/2
  ipv6 address 2001:db8:100:2::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 100
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

end
write memory
```

### MUST-HAVE: Router-BE (Berlin) - OSPFv3 Setup

```cisco
! Router-BE: OSPFv3 Konfiguration
! ================================

configure terminal

ipv6 unicast-routing

router ospfv3 42
  router-id 3.3.3.3
  area 0
!

! Services-Interface (Ge0/1)
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:30::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network broadcast
  ipv6 ospf cost 1
  no shutdown
!

! WAN-Link zu Router-HH (Ge0/0) - Punkt-zu-Punkt
interface GigabitEthernet 0/0
  ipv6 address 2001:db8:100:1::2/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 100
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

! WAN-Link zu Router-LB (Ge0/2) - Punkt-zu-Punkt
interface GigabitEthernet 0/2
  ipv6 address 2001:db8:100:3::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 100
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

! WAN-Link zu Router-BK (Ge0/3) - Punkt-zu-Punkt
interface GigabitEthernet 0/3
  ipv6 address 2001:db8:100:4::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 50  ! Niedrigere Kosten = höhere Priorität
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

end
write memory
```

### MUST-HAVE: Router-LB (Lübeck) - OSPFv3 Setup

```cisco
! Router-LB: OSPFv3 Konfiguration
! ================================

configure terminal

ipv6 unicast-routing

router ospfv3 42
  router-id 2.2.2.2
  area 0
!

! LAN-Interface (Ge0/1)
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:20::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network broadcast
  ipv6 ospf cost 1
  no shutdown
!

! WAN-Link zu Router-HH (Ge0/0) - Punkt-zu-Punkt
interface GigabitEthernet 0/0
  ipv6 address 2001:db8:100:2::2/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 100
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

! WAN-Link zu Router-BE (Ge0/2) - Punkt-zu-Punkt
interface GigabitEthernet 0/2
  ipv6 address 2001:db8:100:3::2/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 100
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

end
write memory
```

### MUST-HAVE: Router-BK (Backup) - OSPFv3 Setup

```cisco
! Router-BK: OSPFv3 Konfiguration
! ================================

configure terminal

ipv6 unicast-routing

router ospfv3 42
  router-id 4.4.4.4
  area 0
!

! Backup-Interface (Ge0/1)
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:40::1/64
  ipv6 ospf 42 area 0
  ipv6 ospf network broadcast
  ipv6 ospf cost 1
  no shutdown
!

! WAN-Link zu Router-BE (Ge0/0) - Punkt-zu-Punkt
interface GigabitEthernet 0/0
  ipv6 address 2001:db8:100:4::2/64
  ipv6 ospf 42 area 0
  ipv6 ospf network point-to-point
  ipv6 ospf cost 50
  ipv6 ospf hello-interval 10
  ipv6 ospf dead-interval 40
  no shutdown
!

end
write memory
```

---

## 📊 OSPFv3 Parameter Erklärung

### Hello & Dead Intervals

```
Standard-Werte:
├── Hello-Interval: 30 Sekunden
├── Dead-Interval:  120 Sekunden (4 × Hello)

Streamline optimiert:
├── Hello-Interval: 10 Sekunden (schnellere Erkennung)
├── Dead-Interval:  40 Sekunden (4 × Hello)

Vorteil: Schnellere Konvergenz bei Ausfällen
Nachteil: Höhere CPU/Bandbreite
```

### Netzwerk-Typen

```
1. Broadcast
   ├── Einsatz: Ethernet LAN
   ├── Neighbor-Discovery: Automatisch
   ├── DR/BDR: Ja (bei > 2 Router)
   └── Beispiel: LAN-Interfaces

2. Point-to-Point
   ├── Einsatz: WAN-Links, Punkt-zu-Punkt
   ├── Neighbor-Discovery: Automatisch
   ├── DR/BDR: Nein
   └── Beispiel: Router-zu-Router Links
```

### Kosten-Berechnung

```
OSPF Cost = 100000 / Bandbreite (in Bits/Sekunde)

Beispiele:
├── Gigabit Ethernet: 100000 / 1000000000 = 1
├── Fast Ethernet:    100000 / 100000000 = 1
├── Serial 10Mbps:    100000 / 10000000 = 10
├── Serial 64Kbps:    100000 / 64000 = 1562

Streamline (WAN-Links):
├── Router-HH ↔ Router-BE: 100 (Primary)
├── Router-HH ↔ Router-LB: 100 (Primary)
├── Router-LB ↔ Router-BE: 100 (Secondary)
├── Router-BE ↔ Router-BK: 50 (Backup - höher priorisiert)
```

---

## ✅ Verifikation und Troubleshooting

### OSPFv3 Status prüfen

```cisco
! Prozess-Status
Router# show ospfv3
OSPFv3 Routing Process "ospfv3 42" with ID 1.1.1.1
  Supports only single TOS(TOS0) routes
  Supports opaque LSA
  It is an area border router
  SPF schedule delay 5 secs, Hold time between two SPFs 10 secs
  Minimum LSA interval 5 secs. Minimum LSA arrival 1 secs
  LSA group pacing timer interval 240 secs
  Interface flood pacing timer 33 secs
  Retransmission pacing timer 66 secs
  Number of external LSA 0. Checksum Sum 0x000000
  Number of areas in this router is 1. 1 normal 0 stub 0 nssa
    Area BACKBONE(0)
      Number of interfaces in this area is 3
      Area has no authentication
      SPF algorithm executed 2 times
      Area ranges are
      Number of LSA 8. Checksum Sum 0x00003dda
      Number of transit virtual links is 0
```

### Neighbor-Status prüfen

```cisco
Router# show ospfv3 neighbor
OSPFv3 Process "ospfv3 42" Neighbors

Addr         InfRtr ID Pri  State      Dead Time Interface ID Interface
2001:db8:100:1::2
             3.3.3.3  128  FULL/ -    00:00:33 2            Ge0/0
2001:db8:100:2::2
             2.2.2.2  128  FULL/ -    00:00:35 3            Ge0/2
```

### Routing Table prüfen

```cisco
Router-HH# show ipv6 route ospf
IPv6 Routing Table - Default - 8 entries
Codes: C - Connected, L - Local, S - Static, U - Per-user Static route
       B - BGP, R - RIP, I1 - ISIS L1, I2 - ISIS L2
       IA - ISIS interarea, IS - ISIS summary, D - EIGRP, EX - EIGRP external
       O - OSPF Intra, OI - OSPF Inter, OE1 - OSPF ext 1, OE2 - OSPF ext 2
       ON1 - OSPF NSSA ext 1, ON2 - OSPF NSSA ext 2

O   2001:DB8:20::/64 [110/200]
     via FE80::2, GigabitEthernet0/2
     via FE80::1, GigabitEthernet0/0
O   2001:DB8:30::/64 [110/100]
     via FE80::1, GigabitEthernet0/0
O   2001:DB8:40::/64 [110/150]
     via FE80::1, GigabitEthernet0/0
```

### LSA Datenbank anschauen

```cisco
Router# show ospfv3 database
OSPFv3 Router with ID (1.1.1.1) (Process ID 42)

Router Link States (Area 0)
ADV Router    Age Seq#      Fragment ID Link count Bits
1.1.1.1      1   0x80000002 0           2          B E
2.2.2.2      4   0x80000002 0           2          B E
3.3.3.3      4   0x80000002 0           3          B E
4.4.4.4      5   0x80000002 0           1          B E
```

### Interface-Konfiguration prüfen

```cisco
Router# show ipv6 ospf interface Gigabitethernet 0/0
GigabitEthernet0/0 is up, line protocol is up
  Link Local Address FE80::1, Interface ID 1
  Area 0, Process ID 42, Instance ID 0, Router ID 1.1.1.1
  Network Type POINT_TO_POINT, Cost: 100
  Transmit Delay is 1 sec, State POINT_TO_POINT
  Timer intervals configured, Hello 10, Dead 40, Wait 40, Retransmit 5
    Hello due in 00:00:08
  Neighbor Count is 1, Adjacent neighbor count is 1
    Adjacent with neighbor 3.3.3.3
  Suppress hello for 0 neighbor(s)
  Message authentication status: Not enabled
```

### Konvergenz-Test

```cisco
! Interface ausschalten (Linkfehler simulieren)
Router-HH# configure terminal
Router-HH(config)# interface GigabitEthernet 0/0
Router-HH(config-if)# shutdown

! Nachbarschaften überwachen
Router-HH# show ospfv3 neighbor
! Nach ca. 40 Sekunden (Dead-Interval):
! Router sollte sich von Neighbor abmelden

! Dann Backup-Pfade nutzen
Router-HH# show ipv6 route ospf
! Routes sollten via Router-LB → Router-BE gehen

! Interface wieder aktivieren
Router-HH# configure terminal
Router-HH(config)# interface GigabitEthernet 0/0
Router-HH(config-if)# no shutdown
```

---

## 🔍 Troubleshooting

### Neighbors werden nicht FULL

```cisco
! Problem: State bleibt bei 2WAY oder LOADING

! Überprüfungen:
1. Interface aktiviert?
   Router# show interface Gigabitethernet 0/0

2. IPv6-Adresse korrekt?
   Router# show ipv6 interface Gigabitethernet 0/0

3. OSPF-Prozess läuft?
   Router# show ospfv3

4. MTU korrekt?
   Router# show ipv6 interface Gigabitethernet 0/0 | include MTU

5. Hello/Dead Intervals identisch?
   Router# show ospfv3 interface
```

### Routen werden nicht gelernt

```cisco
! Problem: Andere Netze nicht in Routing Table

! Überprüfungen:
1. Neighbors FULL?
   Router# show ospfv3 neighbor

2. LSA-Datenbankeinträge vorhanden?
   Router# show ospfv3 database | include 2001:db8

3. Router-IDs eindeutig?
   Router# show ospfv3

4. Area-Konfiguration korrekt?
   Router# show running-config | include ospf
```

---

## 📈 Performance-Optimization

### Für WAN-Umgebungen

```cisco
! Längere Intervals sparen Bandbreite
interface GigabitEthernet 0/0
  ipv6 ospf hello-interval 30
  ipv6 ospf dead-interval 120
!
```

### Für kritische Links

```cisco
! Kürzere Intervals = schnellere Konvergenz
interface GigabitEthernet 0/0
  ipv6 ospf hello-interval 3
  ipv6 ospf dead-interval 12
!
```

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-06-03  
**Prozess-ID:** 42  
**Area:** 0 (Backbone)  
**Router-IDs:** 1.1.1.1 - 4.4.4.4
