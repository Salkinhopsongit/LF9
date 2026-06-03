# IPv6 Adressierungsschema

## Übersicht

Dieses Dokument definiert das IPv6 Adressierungsschema für das gesamte Netzwerk basierend auf der in Schritt 1 erstellten Topologie.

## Verwendetes Netzwerk

**Öffentliches IPv6 Netzwerk:** `2001:db8:1234::/48`

Diese Adresse basiert auf dem Dokumentationspräfix von RFC 3849 und wird für Lernzwecke verwendet.

## Subnetzaufteilung

Das `/48` Netzwerk wird in `/64` Subnetze aufgeteilt:

| Bereich | IPv6 Netzwerk | Zweck | Geräte |
|---------|---------------|-------|--------|
| WAN (Router-zu-Router) | `2001:db8:1234:0::/64` | Verbindung Router 1 ↔ Router 2 | R1, R2 |
| Netzwerk 1 (Switch 1) | `2001:db8:1234:1::/64` | PC1, PC2 | R1, Switch 1 |
| Netzwerk 2 (Switch 2) | `2001:db8:1234:2::/64` | PC3, PC4 | R2, Switch 2 |
| Management | `2001:db8:1234:ff::/64` | Management/Reserve | - |

## Detaillierte IPv6 Adressierung

### WAN-Link: Router 1 ↔ Router 2

**Netzwerk:** `2001:db8:1234:0::/64`

| Gerät | IPv6 Adresse | Schnittstelle | Funktion |
|-------|--------------|---------------|----------|
| Router 1 | `2001:db8:1234:0::1/64` | GigabitEthernet 0/0 | Gateway WAN-Link |
| Router 2 | `2001:db8:1234:0::2/64` | GigabitEthernet 0/0 | Gateway WAN-Link |

**Default Route:** 
- R1: Ziel `2001:db8:1234:2::/64` via `2001:db8:1234:0::2`
- R2: Ziel `2001:db8:1234:1::/64` via `2001:db8:1234:0::1`

---

### Lokales Netzwerk 1 (Switch 1 / Router 1)

**Netzwerk:** `2001:db8:1234:1::/64`

| Gerät | IPv6 Adresse | MAC-Adresse | Funktion |
|-------|--------------|-------------|----------|
| Router 1 | `2001:db8:1234:1::1/64` | - | Gateway/DHCPv6-Server |
| PC 1 | `2001:db8:1234:1::10/64` | *wird zugewiesen* | Client |
| PC 2 | `2001:db8:1234:1::11/64` | *wird zugewiesen* | Client |
| Switch 1 | `2001:db8:1234:1::253/64` | - | Management (optional) |

**Broadcast-Adresse:** `2001:db8:1234:1:ffff:ffff:ffff:ffff`  
**Verfügbare Adressen:** ~18,4 Quintillionen

---

### Lokales Netzwerk 2 (Switch 2 / Router 2)

**Netzwerk:** `2001:db8:1234:2::/64`

| Gerät | IPv6 Adresse | MAC-Adresse | Funktion |
|-------|--------------|-------------|----------|
| Router 2 | `2001:db8:1234:2::1/64` | - | Gateway/DHCPv6-Server |
| PC 3 | `2001:db8:1234:2::10/64` | *wird zugewiesen* | Client |
| PC 4 | `2001:db8:1234:2::11/64` | *wird zugewiesen* | Client |
| Switch 2 | `2001:db8:1234:2::253/64` | - | Management (optional) |

**Broadcast-Adresse:** `2001:db8:1234:2:ffff:ffff:ffff:ffff`  
**Verfügbare Adressen:** ~18,4 Quintillionen

---

## IPv6 Adressierungsübersicht (Tabellarisch)

```
┌─────────────────────────────────────────────────────────────────┐
│                   IPv6 ADRESSIERUNGSPLAN                         │
├─────────────────────────────────────────────────────────────────┤
│                                                                 │
│  WAN-Link (Router ↔ Router)                                    │
│  ├─ Netzwerk: 2001:db8:1234:0::/64                            │
│  ├─ Router 1 Ge0/0: 2001:db8:1234:0::1/64                     │
│  └─ Router 2 Ge0/0: 2001:db8:1234:0::2/64                     │
│                                                                 │
│  Netzwerk 1 (Router 1 - Switch 1)                             │
│  ├─ Netzwerk: 2001:db8:1234:1::/64                            │
│  ├─ Router 1 Ge0/1: 2001:db8:1234:1::1/64 (Gateway)          │
│  ├─ PC 1: 2001:db8:1234:1::10/64                              │
│  └─ PC 2: 2001:db8:1234:1::11/64                              │
│                                                                 │
│  Netzwerk 2 (Router 2 - Switch 2)                             │
│  ├─ Netzwerk: 2001:db8:1234:2::/64                            │
│  ├─ Router 2 Ge0/1: 2001:db8:1234:2::1/64 (Gateway)          │
│  ├─ PC 3: 2001:db8:1234:2::10/64                              │
│  └─ PC 4: 2001:db8:1234:2::11/64                              │
│                                                                 │
└─────────────────────────────────────────────────────────────────┘
```

## IPv6 Grundkonzepte

### Adressformat

**IPv6 verwendet 128-Bit Adressen** (vs. IPv4: 32-Bit)

- **Schreibweise:** 8 Gruppen à 4 Hexadezimalziffern, durch Doppelpunkte getrennt
- **Beispiel:** `2001:0db8:1234:0001:0000:0000:0000:0001`
- **Vereinfachte Schreibweise:** `2001:db8:1234:1::1` (führende Nullen und aufeinanderfolgende Null-Gruppen können weggelassen werden)

### Präfixnotation (CIDR)

- `/64` bedeutet: Die ersten 64 Bits sind das Netzwerk, die restlichen 64 Bits sind für Hosts
- **Subnetzmaske:** Grundkonzept existiert bei IPv6 nicht, stattdessen wird die Präfixlänge verwendet

### Link-Local Adressen

Geräte erstellen automatisch Link-Local Adressen im Format `fe80::/10`:

| Gerät | Link-Local Adresse |
|-------|--------------------|
| Router 1 | `fe80::1` (automatisch auf allen Interfaces) |
| Router 2 | `fe80::2` (automatisch auf allen Interfaces) |
| PC 1 | `fe80::...` (automatisch generiert) |
| PC 2 | `fe80::...` (automatisch generiert) |
| PC 3 | `fe80::...` (automatisch generiert) |
| PC 4 | `fe80::...` (automatisch generiert) |

### DHCPv6 & Stateless Address Autoconfiguration (SLAAC)

**Zwei Methoden der IPv6-Konfiguration:**

1. **SLAAC (Empfohlen):** Router sendet Prefix, Clients generieren Host-Teil aus MAC-Adresse
2. **DHCPv6:** Router agiert als DHCP-Server und weist Adressen zu

---

## Konfigurationsbeispiele für Cisco Router

### Router 1 - Konfiguration

```cisco
! IPv6 Unicast Routing aktivieren
ipv6 unicast-routing

! WAN-Interface (Ge0/0)
interface GigabitEthernet 0/0
 ipv6 address 2001:db8:1234:0::1/64
 description WAN Link to Router 2
 no shutdown

! LAN-Interface (Ge0/1)
interface GigabitEthernet 0/1
 ipv6 address 2001:db8:1234:1::1/64
 description LAN Network 1
 ipv6 nd ra suppress all  ! RA unterdrücken, wenn DHCPv6 verwendet wird
 no shutdown

! Routing (oder statisch)
ipv6 route 2001:db8:1234:2::/64 2001:db8:1234:0::2
```

### Router 2 - Konfiguration

```cisco
! IPv6 Unicast Routing aktivieren
ipv6 unicast-routing

! WAN-Interface (Ge0/0)
interface GigabitEthernet 0/0
 ipv6 address 2001:db8:1234:0::2/64
 description WAN Link to Router 1
 no shutdown

! LAN-Interface (Ge0/1)
interface GigabitEthernet 0/1
 ipv6 address 2001:db8:1234:2::1/64
 description LAN Network 2
 ipv6 nd ra suppress all  ! RA unterdrücken, wenn DHCPv6 verwendet wird
 no shutdown

! Routing (oder statisch)
ipv6 route 2001:db8:1234:1::/64 2001:db8:1234:0::1
```

---

## Verifikations-Befehle

### IPv6 Adressen überprüfen
```cisco
show ipv6 interface brief
show ipv6 interface
```

### Routing-Tabelle anzeigen
```cisco
show ipv6 route
```

### Konnektivität testen
```cisco
ping 2001:db8:1234:0::1
ipv6 ping 2001:db8:1234:1::10
```

---

## Zusammenfassung

| Komponente | IPv6 Adresse | Bemerkung |
|-----------|--------------|----------|
| **Gesamt-Netzwerk** | `2001:db8:1234::/48` | Dokumentationspräfix |
| **WAN-Link** | `2001:db8:1234:0::/64` | Router-zu-Router |
| **LAN 1** | `2001:db8:1234:1::/64` | Switch 1 + PC1, PC2 |
| **LAN 2** | `2001:db8:1234:2::/64` | Switch 2 + PC3, PC4 |
| **Router 1 WAN** | `2001:db8:1234:0::1/64` | Gateway WAN |
| **Router 1 LAN** | `2001:db8:1234:1::1/64` | Gateway LAN 1 |
| **Router 2 WAN** | `2001:db8:1234:0::2/64` | Gateway WAN |
| **Router 2 LAN** | `2001:db8:1234:2::1/64` | Gateway LAN 2 |

---

**Status**: ✅ IPv6-Schema definiert  
**Dauer**: ~5 Minuten zum Verstehen  
**Nächster Schritt**: [Router-Konfiguration](./02_router_config.md)
