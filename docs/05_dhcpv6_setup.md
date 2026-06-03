# 5. DHCPv6 Setup - SLAAC mit Server

## 📋 DHCPv6 Strategien

### MUST-HAVE: SLAAC with DHCPv6-Server

**Modus:** Stateless Address Autoconfiguration + DHCPv6 für optionale Daten

```
Router sendet RA (Router Advertisement):
├── A-Flag = 1 (SLAAC erlaubt)
├── M-Flag = 0 (kein Stateful DHCP)
└── O-Flag = 1 (andere Daten per DHCP)

Client-Verhalten:
├── 1. Generiert IPv6-Adresse selbst (SLAAC)
├── 2. Kontaktiert DHCPv6-Server für DNS, Domain
└── 3. Erhält Leasing-Informationen
```

**Vorteile:**
- ✅ Clients sind unabhängig
- ✅ Dezentrale Adressvergabe
- ✅ Automatische Konvergenz
- ✅ Skalierbar (keine zentrale DB)
- ✅ Fallback bei Server-Ausfall

---

## 🏢 Hamburg Router - SLAAC+Server

### Konfiguration

```cisco
! Router-HH: SLAAC mit DHCPv6-Server
! ================================

configure terminal

! 1. IPv6 aktivieren
ipv6 unicast-routing

! 2. Router Advertisement für SLAAC konfigurieren
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:10::1/64
  ipv6 nd prefix 2001:db8:10::/64 infinite infinite
  ipv6 nd ra suppress all
  ipv6 nd ra interval 200 60
  ipv6 nd managed-config-flag
  ipv6 nd other-config-flag
  no shutdown
!

! 3. DHCPv6 Pool definieren
ipv6 dhcp pool Hamburg-DHCP-Pool
  address prefix 2001:db8:10::300/128
  dns-server 2001:db8:30::53
  dns-server 2001:db8:10::102
  domain-name hamburg.streamline.local
  lease 0 1 0 0
!

! 4. DHCPv6 auf Interface aktivieren
interface GigabitEthernet 0/1
  ipv6 dhcp server Hamburg-DHCP-Pool
!

end
write memory
```

### Flags-Erklärung

```
RA-Flags:
├── A-Flag = 1  → Clients nutzen SLAAC
├── M-Flag = 0  → Kein Stateful DHCPv6
└── O-Flag = 1  → DHCPv6 für optionale Daten (DNS, Domain)

Interval:
├── 200ms = Intervall zwischen RA-Paketen
└── 60s   = Timeout bis Router-Löschen
```

### Verifikation Hamburg

```cisco
Router-HH# show ipv6 dhcp pool
DHCPv6 pool: Hamburg-DHCP-Pool
  Address allocation prefix: 2001:db8:10::/64 valid 0 preferred 0 (infinite)
  DNS server: 2001:db8:30::53
  DNS server: 2001:db8:10::102
  Domain name: hamburg.streamline.local
  Lease: 0 days 1 hours 0 minutes 0 seconds

Router-HH# show ipv6 interface GigabitEthernet 0/1
GigabitEthernet0/1 is up, line protocol is up
  IPv6 is enabled, link-local address is FE80::1
  No Virtual link-local address(es)
  Global unicast address(es):
    2001:DB8:10::1, subnet is 2001:DB8:10::/64
  Joined group address(es):
    FF02::1
    FF02::1:FF00:1
  MTU is 1500 bytes
  ICMP error messages limited to one every 100 milliseconds
  ICMP redirects are enabled
  ND DAD is enabled, number of DAD attempts: 1
  ND reachable time is 30000 milliseconds (using 30000)
  ND advertised reachable time is 0 (unspecified)
  ND advertised retransmit interval is 0 (unspecified)
  ND router advertisements are sent every 200 seconds
  ND router advertisements live for 1800 seconds
  ND advertised default router preference is Medium
  Hosts use stateless autoconfig for addresses.
  Hosts use DHCP to obtain other information.
  Prefix 2001:db8:10::/64
    Valid lifetime infinite, preferred lifetime infinite
    On-link flag is on
    Autonomous flag is on
    A flag is on
```

---

## 🏢 Lübeck Router - SLAAC+Server

### Konfiguration

```cisco
! Router-LB: SLAAC mit DHCPv6-Server
! ================================

configure terminal

interface GigabitEthernet 0/1
  ipv6 address 2001:db8:20::1/64
  ipv6 nd prefix 2001:db8:20::/64 infinite infinite
  ipv6 nd ra suppress all
  ipv6 nd ra interval 200 60
  ipv6 nd managed-config-flag
  ipv6 nd other-config-flag
  no shutdown
!

ipv6 dhcp pool Luebeck-DHCP-Pool
  address prefix 2001:db8:20::300/128
  dns-server 2001:db8:30::53
  domain-name luebeck.streamline.local
  lease 0 1 0 0
!

interface GigabitEthernet 0/1
  ipv6 dhcp server Luebeck-DHCP-Pool
!

end
write memory
```

---

## 🏢 Berlin Router - SLAAC+Server (optional)

### Konfiguration

```cisco
! Router-BE: SLAAC (für lokale Services)
! ====================================

configure terminal

interface GigabitEthernet 0/1
  ipv6 address 2001:db8:30::1/64
  ipv6 nd prefix 2001:db8:30::/64 infinite infinite
  ipv6 nd ra suppress all
  ipv6 nd ra interval 200 60
  ipv6 nd managed-config-flag
  ipv6 nd other-config-flag
  no shutdown
!

! Berlin hat statische Adressen, DHCPv6-Server optional
ipv6 dhcp pool Berlin-DHCP-Pool
  address prefix 2001:db8:30::500/128
  dns-server 2001:db8:30::53
  domain-name berlin.streamline.local
  lease 0 1 0 0
!

interface GigabitEthernet 0/1
  ipv6 dhcp server Berlin-DHCP-Pool
!

end
write memory
```

---

## 💡 SHOULD-HAVE: DHCPv6 Stateful (Hamburg optional)

### Stateful DHCPv6 auf Hamburg

**Modus:** Router verwaltet alle IP-Adressen zentral

```cisco
! Hamburg mit Stateful DHCPv6 (optional upgrade)
! =============================================

configure terminal

! RA-Flags für Stateful:
interface GigabitEthernet 0/1
  ipv6 address 2001:db8:10::1/64
  ipv6 nd prefix 2001:db8:10::/64 infinite infinite
  ipv6 nd ra suppress all
  ipv6 nd ra interval 200 60
  ipv6 nd managed-config-flag    ! M-Flag = 1
  no ipv6 nd other-config-flag   ! O-Flag = 0
  no shutdown
!

! Stateful DHCPv6 Pool
ipv6 dhcp pool Hamburg-Stateful-Pool
  address prefix 2001:db8:10::300/128 lifetime 86400 43200
  dns-server 2001:db8:30::53
  dns-server 2001:db8:10::102
  domain-name hamburg.streamline.local
!

interface GigabitEthernet 0/1
  ipv6 dhcp server Hamburg-Stateful-Pool
!

end
write memory
```

**Unterschiede:**

| Aspekt | SLAAC+Server | Stateful |
|--------|--------------|----------|
| Adress-Generierung | Client (SLAAC) | Router |
| A-Flag | 1 | 0 |
| M-Flag | 0 | 1 |
| Zentrale Kontrolle | Nein | Ja |
| Fallback-Sicherheit | Hoch | Mittel |

---

## 🌐 COULD-HAVE: Zentrale DHCPv6-Lösung

### Szenario: Berlin als zentraler DHCP-Server

**Problem:** Cisco Packet Tracer unterstützt IPv6 Relay nicht vollständig

**Lösung (theoretisch):**

```cisco
! Berlin-Router als zentraler DHCPv6-Server
! ========================================

! 1. Zentrale DHCPv6 Pools
ipv6 dhcp pool Central-Hamburg
  address prefix 2001:db8:10::300/128 lifetime 86400 43200
  dns-server 2001:db8:30::53
  domain-name hamburg.streamline.local
!

ipv6 dhcp pool Central-Luebeck
  address prefix 2001:db8:20::300/128 lifetime 86400 43200
  dns-server 2001:db8:30::53
  domain-name luebeck.streamline.local
!

! 2. DHCPv6 Relay auf Hamburg & Lübeck
! (NICHT in PT unterstützt)

! Hamburg-Router:
interface GigabitEthernet 0/1
  ipv6 dhcp relay destination 2001:db8:30::67
  ! → Diese Funktion existiert in PT nicht!
!

! Lübeck-Router:
interface GigabitEthernet 0/1
  ipv6 dhcp relay destination 2001:db8:30::67
!

! 3. DHCP-Server auf Berlin
interface GigabitEthernet 0/1
  ipv6 dhcp server Central-Hamburg
  ipv6 dhcp server Central-Luebeck
!
```

**PT-Limitationen:**
- ❌ `ipv6 dhcp relay destination` wird nicht unterstützt
- ❌ Nur lokale DHCPv6-Server möglich
- ⚠️ Workaround: Jeder Router hat eigene Pools

**Dokumentation für reale Umgebung:**

```
Notwendige Schritte für zentrale DHCPv6:

1. DHCPv6-Server auf Berlin aktivieren
2. Relay-Agents auf Hamburg & Lübeck aktivieren
3. Relay-Destination auf Berlin-Server zeigen
4. Client-Requests werden automatisch weitergeleitet
5. Zentrale IPAM-Verwaltung möglich
```

---

## 📊 DHCPv6-Leasing

### Leasing-Zeiten

```
Lease-Format: days hours minutes seconds

Hamburg:   0 1 0 0  = 1 Stunde
Lübeck:    0 1 0 0  = 1 Stunde (Testing)
Berlin:    0 24 0 0 = 24 Stunden (Services)

Preferred Lifetime: 50% der Lease-Zeit
Valid Lifetime:     100% der Lease-Zeit
```

### Renewal-Prozess

```
Client erhält Adresse:
├── T1 (50% der Lease) → RENEW
├── T2 (75% der Lease) → REBIND
└── Nach Lease-Ende   → Neue Anfrage
```

---

## ✅ Validierungschecklist

- [ ] Router sendet RA-Pakete
- [ ] Clients erhalten SLAAC-Adressen
- [ ] DHCPv6-Server antwortet
- [ ] Clients erhalten DNS-Server
- [ ] Domain-Namen funktionieren
- [ ] Leasing läuft korrekt
- [ ] Renewal funktioniert
- [ ] Zentrale DHCPv6 dokumentiert

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-06-03  
**Modus:** SLAAC with DHCPv6-Server
