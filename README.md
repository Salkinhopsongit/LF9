# Streamline IPv6 Netzwerk - Vollständige Dokumentation

## 📋 Projektübersicht

**Auftraggeber:** Firma Streamline  
**Projektziel:** Sichere IPv6-Netzwerk-Infrastruktur mit automatisierter Konfiguration  
**Status:** In Bearbeitung

## 🎯 Anforderungen

### MUST-HAVE

#### 1. ACL IPv6 - Zugriffskontrolle
- SSH-Zugang nur von Hamburg
- HTTP/HTTPS für Hamburg (beide Protokolle)
- HTTPS nur für Lübeck
- Dokumentation aller Regeln

#### 2. DHCPv6 - SLAAC mit Server
- Implementierung "SLAAC with DHCPv6-Server" auf Routern
- Konfigurationen in IPAM dokumentieren

#### 3. OSPFv3 - Dynamisches Routing
- Prozess-ID: 42
- Area: 0
- Router-IDs: 1.1.1.1 bis 4.4.4.4
- Ersatz für statische Routen

### SHOULD-HAVE

- Mindestens eine zusätzliche Sicherheitsregel
- DHCPv6 Stateful-Modus Hamburg
- IPAM-Dokumentation

### COULD-HAVE

- Zentrale DHCPv6-Lösung (Berlin-Server oder zentraler Router)
- Relay-Agent Konfiguration (Hinweis: PT-Limitation dokumentieren)

## 📁 Projektstruktur

```
Streamline-IPv6-Netzwerk/
├── docs/
│   ├── 01_anforderungen.md
│   ├── 02_netzwerk_topologie.md
│   ├── 03_ipv6_adressen.md
│   ├── 04_acl_konfiguration.md
│   ├── 05_dhcpv6_setup.md
│   ├── 06_ospfv3_routing.md
│   ├── 07_ipam_dokumentation.md
│   └── 08_testing_validierung.md
├── config/
│   ├── router_hamburg.txt
│   ├── router_luebeck.txt
│   ├── router_berlin.txt
│   ├── router_backup.txt
│   ├── acl_rules.txt
│   └── ospfv3_config.txt
├── ipam/
│   ├── adressplan.csv
│   ├── dhcpv6_pools.csv
│   └── routing_tabellen.csv
├── testing/
│   ├── test_plan.md
│   ├── test_results.md
│   └── validation_checklist.md
└── README.md
```

## 🏢 Standorte

| Standort | Stadt | Router | LAN-Subnetz | WAN-IP |
|----------|-------|--------|------------|--------|
| Hauptsitz | Hamburg | Router-HH | 2001:db8:10::/64 | 2001:db8:100::1/64 |
| Zweigstelle | Lübeck | Router-LB | 2001:db8:20::/64 | 2001:db8:100::2/64 |
| Serverraum | Berlin | Router-BE | 2001:db8:30::/64 | 2001:db8:100::3/64 |
| Backup | --- | Router-BK | 2001:db8:40::/64 | 2001:db8:100::4/64 |

## 📊 Zeitplan

| Phase | Aufgabe | Status |
|-------|---------|--------|
| 1 | Anforderungsanalyse | ✅ |
| 2 | Netzwerkdesign | 🔄 |
| 3 | ACL-Konfiguration | 🔄 |
| 4 | DHCPv6-Setup | 🔄 |
| 5 | OSPFv3-Implementation | 🔄 |
| 6 | Testing & Validierung | ⏳ |
| 7 | Dokumentation | ⏳ |
| 8 | Leistungsnachweis | ⏳ |

## 🔗 Dokumentation

Folgen Sie den Schritten in dieser Reihenfolge:

1. **[Anforderungen](./docs/01_anforderungen.md)** - Detaillierte Anforderungsanalyse
2. **[Topologie](./docs/02_netzwerk_topologie.md)** - Netzwerk-Architektur
3. **[IPv6-Adressen](./docs/03_ipv6_adressen.md)** - Adressierungsplan
4. **[ACL](./docs/04_acl_konfiguration.md)** - Zugriffskontrolle
5. **[DHCPv6](./docs/05_dhcpv6_setup.md)** - DHCP-Konfiguration
6. **[OSPFv3](./docs/06_ospfv3_routing.md)** - Routing-Protokoll
7. **[IPAM](./docs/07_ipam_dokumentation.md)** - IP-Adressverwaltung
8. **[Testing](./docs/08_testing_validierung.md)** - Validierung

---

**Letzte Aktualisierung:** 2026-06-03  
**Projektleitung:** Salkinhopsongit
