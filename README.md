# IPv6 Netzwerk mit Cisco Packet Tracer

Dieses Repository dokumentiert die Einrichtung und Konfiguration eines IPv6-Netzwerks mit Cisco Packet Tracer.

## Projektbeschreibung

In diesem Projekt wird ein funktionales IPv6-Netzwerk aufgebaut, konfiguriert und getestet. Das Projekt zeigt:

- Grundlagen von IPv6-Adressierung
- Konfiguration von Routern und Switches
- Statische und dynamische Routing-Protokolle
- VLAN-Konfiguration
- Netzwerk-Tests und Validierung

## Inhaltsverzeichnis

- [Anforderungen](#anforderungen)
- [Netzwerk-Topologie](#netzwerk-topologie)
- [Konfigurationsschritte](#konfigurationsschritte)
- [Dateien](#dateien)
- [Ressourcen](#ressourcen)

## Anforderungen

- Cisco Packet Tracer (Version 8.0 oder neuer)
- Grundkenntnisse in Netzwerkkonfiguration
- Verständnis von IPv6-Adressierung

## Netzwerk-Topologie

```
┌─────────────┐       ┌─────────────┐
│   Router1   │───────│   Router2   │
│ (FE80::1/10)│       │ (FE80::2/10)│
└──────┬──────┘       └──────┬──────┘
       │                     │
   ┌───┴──────┐          ┌───┴──────┐
   │  Switch1 │          │  Switch2 │
   └───┬──────┘          └───┬──────┘
       │                     │
  ┌────┴─────────┐      ┌────┴─────────┐
  │   PC1        │      │   PC2        │
  │ (2001:db8::1)│      │ (2001:db8::2)│
  └──────────────┘      └──────────────┘
```

## Konfigurationsschritte

Siehe die detaillierte Dokumentation in den folgenden Dateien:

- [Schritt 1: Topologie aufbau](./docs/01_topologie.md)
- [Schritt 2: Router-Konfiguration](./docs/02_router_config.md)
- [Schritt 3: PC-Konfiguration](./docs/03_pc_config.md)
- [Schritt 4: Routing-Setup](./docs/04_routing.md)
- [Schritt 5: Tests und Validierung](./docs/05_tests.md)

## Dateien

```
├── README.md                          # Diese Datei
├── docs/
│   ├── 01_topologie.md               # Topologie-Aufbau
│   ├── 02_router_config.md           # Router-Konfiguration
│   ├── 03_pc_config.md               # PC-Konfiguration
│   ├── 04_routing.md                 # Routing-Protokolle
│   └── 05_tests.md                   # Tests und Validierung
├── config/
│   ├── router1_config.txt            # Konfiguration Router 1
│   ├── router2_config.txt            # Konfiguration Router 2
│   └── ipv6_adressen.csv             # IPv6 Adressliste
├── packet_tracer/
│   └── ipv6_netzwerk.pkt             # Packet Tracer Datei
└── scripts/
    └── test_netzwerk.sh              # Test-Skript
```

## Ressourcen

- [IPv6 Wikipedia](https://de.wikipedia.org/wiki/IPv6)
- [Cisco IPv6 Dokumentation](https://www.cisco.com/c/en/us/support/docs/ip/ip-version-6-ipv6/index.html)
- [Packet Tracer Tutorials](https://www.netacad.com)

## Lizenz

Dieses Projekt ist Open Source und steht unter der MIT-Lizenz.

---

**Erstellt**: 2026-06-03  
**Autor**: Salkinhopsongit
