# 3. ACL IPv6 Konfiguration - Streamline

## 🔐 Access Control Lists für Streamline

### Übersicht Sicherheitsanforderungen

```
┌─────────────────────────────────────────────────┐
│        SECURITY REQUIREMENTS MATRIX             │
├─────────────────────────────────────────────────┤
│                                                 │
│ Anforderung          │ Hamburg │ Lübeck │ Berlin│
│ ──────────────────────┼─────────┼────────┼───────│
│ SSH (Port 22)        │   ✓     │   ✗    │   ✗   │
│ HTTP (Port 80)       │   ✓     │   ✗    │   ~   │
│ HTTPS (Port 443)     │   ✓     │   ✓    │   ~   │
│ DNS (Port 53)        │   ✓     │   ✓    │   ✓   │
│ Mail (Port 25)       │   ✓     │   ✓    │   ✓   │
│ NTP (Port 123)       │   ✓     │   ✓    │   ✓   │
│                                                 │
│ ✓ = Erlaubt                                    │
│ ✗ = Verboten                                   │
│ ~ = Intern nur (nicht von außen)               │
└─────────────────────────────────────────────────┘
```

---

## 🏢 Hamburg Router - ACL IPv6 Konfiguration

### MUST-HAVE: SSH nur von Hamburg LAN

```cisco
! Hamburg Router - ACL für SSH (Port 22)
! ====================================

configure terminal

! 1. Named ACL für SSH
ipv6 access-list Hamburg-SSH
  ! Nur Clients aus Hamburg LAN
  permit tcp 2001:db8:10::/64 any eq 22
  ! Alle anderen SSH-Requests ablehnen
  deny tcp any any eq 22
  ! Alle anderen Protokolle erlauben
  permit ipv6 any any
!

! 2. ACL auf WAN-Interface anwenden (eingehend)
interface GigabitEthernet 0/0
  ipv6 traffic-filter Hamburg-SSH in
!

end
write memory
```

### MUST-HAVE: HTTP/HTTPS für Hamburg

```cisco
! Hamburg Router - HTTP/HTTPS ACL
! ===============================

configure terminal

ipv6 access-list Hamburg-HTTP
  ! Alle HTTP-Requests erlauben (intern)
  permit tcp 2001:db8:10::/64 any eq 80
  permit tcp 2001:db8:10::/64 any eq 443
  ! Externe HTTP-Requests ablehnen, HTTPS nur auf secure channel
  deny tcp any any eq 80
  permit tcp any any eq 443
  permit ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Hamburg-HTTP in
!

end
write memory
```

### MUST-HAVE: DNS-Zugriff erlauben

```cisco
! Hamburg Router - DNS ACL
! =======================

configure terminal

ipv6 access-list Hamburg-DNS
  ! DNS von Hamburg zu Berlin
  permit udp 2001:db8:10::/64 2001:db8:30::53 eq 53
  permit tcp 2001:db8:10::/64 2001:db8:30::53 eq 53
  ! DNS lokal
  permit udp 2001:db8:10::/64 2001:db8:10::1 eq 53
  ! Mail-Server Zugriff
  permit tcp 2001:db8:10::/64 2001:db8:30::25 eq 25
  ! NTP Server Zugriff
  permit udp 2001:db8:10::/64 2001:db8:30::123 eq 123
  ! Rest erlauben
  permit ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Hamburg-DNS in
!

end
write memory
```

---

## 🏢 Lübeck Router - ACL IPv6 Konfiguration

### MUST-HAVE: HTTPS only für Lübeck

```cisco
! Lübeck Router - HTTPS only
! ===========================

configure terminal

! 1. HTTP DENY - nur HTTPS erlaubt
ipv6 access-list Luebeck-Web
  ! HTTPS erlauben
  permit tcp 2001:db8:20::/64 any eq 443
  ! HTTP explizit verbieten
  deny tcp 2001:db8:20::/64 any eq 80
  ! SSH verbieten
  deny tcp 2001:db8:20::/64 any eq 22
  ! DNS, Mail, NTP erlauben
  permit udp 2001:db8:20::/64 2001:db8:30::53 eq 53
  permit tcp 2001:db8:20::/64 2001:db8:30::53 eq 53
  permit tcp 2001:db8:20::/64 2001:db8:30::25 eq 25
  permit udp 2001:db8:20::/64 2001:db8:30::123 eq 123
  ! Rest deny
  deny ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Luebeck-Web in
!

end
write memory
```

### MUST-HAVE: SSH von Lübeck verbieten

```cisco
! Lübeck Router - SSH Block
! ==========================

configure terminal

ipv6 access-list Luebeck-SSH
  ! SSH verbieten
  deny tcp 2001:db8:20::/64 any eq 22
  permit ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Luebeck-SSH in
!

end
write memory
```

---

## 🏢 Berlin Router - ACL IPv6 Konfiguration

### Services-Layer ACL

```cisco
! Berlin Router - Services Protection
! ====================================

configure terminal

! 1. DNS Server schützen
ipv6 access-list Berlin-DNS
  ! Nur Port 53 (DNS)
  permit udp any 2001:db8:30::53 eq 53
  permit tcp any 2001:db8:30::53 eq 53
  ! Mail Server
  permit tcp any 2001:db8:30::25 eq 25
  ! NTP Server
  permit udp any 2001:db8:30::123 eq 123
  ! RADIUS für Authentifizierung
  permit udp any 2001:db8:30::1645 eq 1645
  permit udp any 2001:db8:30::1812 eq 1812
  ! Alles andere deny
  deny ipv6 any any
!

interface GigabitEthernet 0/1
  ipv6 traffic-filter Berlin-DNS in
!

end
write memory
```

---

## 🛡️ SHOULD-HAVE: Zusätzliche Sicherheitsregeln

### SHOULD-HAVE: Rate-Limiting gegen DDoS

```cisco
! Alle Router - DDoS Protection
! ==============================

configure terminal

! 1. Named ACL mit Rate-Limiting
ipv6 access-list DDoS-Protection
  ! Limit HTTPS Connections (Port 443)
  permit tcp any any eq 443
  ! Limit DNS Queries
  permit udp any any eq 53
  ! Kritische Services
  permit tcp any any eq 25
  permit udp any any eq 123
  ! Implizites Deny für unbekannte Quellen
  deny ipv6 any any
!

! 2. Queue Limiting
interface GigabitEthernet 0/0
  ipv6 traffic-filter DDoS-Protection in
  ! Maximale Anzahl von Verbindungen pro Quell-IP
  rate-limit input 100000 8000 8000 conform-action transmit exceed-action drop
!

end
write memory
```

### SHOULD-HAVE: Loopback Prevention

```cisco
! Alle Router - Loopback Prevention
! ==================================

configure terminal

! 1. Link-Local nur lokal
ipv6 access-list Loopback-Filter
  ! Loopback Addresses (FE80::/10) nur lokal
  deny ipv6 any fe80::/10
  ! Multicast lokal
  deny ipv6 any ff00::/8
  ! Rest erlauben
  permit ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Loopback-Filter in
!

end
write memory
```

### SHOULD-HAVE: Spoofing Protection

```cisco
! Alle Router - Anti-Spoofing
! ===========================

configure terminal

! 1. Quell-basierte ACL
ipv6 access-list Anti-Spoofing
  ! Keine Private Adressen von außen
  deny ipv6 2001:db8:10::/64 any
  deny ipv6 2001:db8:20::/64 any
  deny ipv6 2001:db8:30::/64 any
  deny ipv6 2001:db8:40::/64 any
  ! Keine undefinierten Adressen
  deny ipv6 ::/128 any
  deny ipv6 ::1/128 any
  ! Valide externe Adressen
  permit ipv6 any any
!

interface GigabitEthernet 0/0
  ipv6 traffic-filter Anti-Spoofing in
!

end
write memory
```

---

## 📊 ACL-Matrix Übersicht

| Regel | Quell-LAN | Ziel-Port | Aktion | Router | Priorität |
|-------|-----------|-----------|--------|--------|----------|
| SSH | Hamburg | 22 | PERMIT | HH | 1 |
| SSH | Lübeck | 22 | DENY | LB | 1 |
| SSH | Berlin | 22 | DENY | BE | 1 |
| HTTP | Hamburg | 80 | PERMIT | HH | 2 |
| HTTP | Lübeck | 80 | DENY | LB | 2 |
| HTTP | Berlin | 80 | DENY | BE | 2 |
| HTTPS | Hamburg | 443 | PERMIT | HH | 3 |
| HTTPS | Lübeck | 443 | PERMIT | LB | 3 |
| HTTPS | Berlin | 443 | PERMIT | BE | 3 |
| DNS | Hamburg | 53 | PERMIT | HH | 4 |
| DNS | Lübeck | 53 | PERMIT | LB | 4 |
| DNS | Berlin | 53 | PERMIT | BE | 4 |
| Mail | Hamburg | 25 | PERMIT | HH | 5 |
| Mail | Lübeck | 25 | PERMIT | LB | 5 |
| Mail | Berlin | 25 | PERMIT | BE | 5 |
| NTP | Hamburg | 123 | PERMIT | HH | 6 |
| NTP | Lübeck | 123 | PERMIT | LB | 6 |
| NTP | Berlin | 123 | PERMIT | BE | 6 |

---

## ✅ Validierungschecklist

### Hamburg Tests
- [ ] SSH funktioniert von Hamburg-Clients
- [ ] SSH funktioniert NICHT von Lübeck
- [ ] HTTP funktioniert von Hamburg
- [ ] HTTPS funktioniert von Hamburg
- [ ] DNS-Queries erfolgreich
- [ ] Mail-Server erreichbar
- [ ] NTP-Synchronisation erfolgt

### Lübeck Tests
- [ ] SSH funktioniert NICHT
- [ ] HTTP funktioniert NICHT
- [ ] HTTPS funktioniert
- [ ] DNS-Queries erfolgreich
- [ ] Mail-Server erreichbar
- [ ] NTP-Synchronisation erfolgt

### Berlin Tests
- [ ] DNS antwortet auf Port 53
- [ ] Mail-Server erreichbar auf Port 25
- [ ] NTP-Server antwortet auf Port 123
- [ ] RADIUS-Authentifizierung funktioniert
- [ ] SSH NICHT erreichbar

### Security Tests
- [ ] DDoS-Protection aktiv
- [ ] Loopback-Prevention funktioniert
- [ ] Spoofing-Protection aktiv
- [ ] Rate-Limiting wirksam
- [ ] Keine unautorisierten Protokolle durchgelassen

---

## 🔍 Troubleshooting

### ACL nicht wirksam

```cisco
! Überprüfung
Router# show ipv6 access-lists
Router# show ipv6 interface GigabitEthernet 0/0
Router# debug ipv6 access-list
```

### Legitime Verbindungen blockiert

```cisco
! ACL-Reihenfolge überprüfen und ggf. neu sortieren
Router# show ipv6 access-lists
! Zu restriktive Regeln identifizieren
Router# clear access-list counters
! Verkehr monitoren
Router# show access-lists
```

---

**Version:** 1.0  
**Letzte Aktualisierung:** 2026-06-03  
**Status:** MUST-HAVE + SHOULD-HAVE Implementiert
