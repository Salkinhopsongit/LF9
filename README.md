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

### FUTURE IMPLEMENTATIONS

### 1. Dual-Stack-Betrieb (IPv4 & IPv6 parallel)

* **Was ist das?** Die gleichzeitige Ausführung von IPv4 und IPv6 auf allen Routern, Switches und Endgeräten.
* **Warum wichtig?** In der echten Welt sind noch nicht alle Webseiten und Dienste im Internet rein über IPv6 erreichbar. Ein modernes Unternehmensnetzwerk fährt deshalb meistens "Dual-Stack", um die volle Kompatibilität mit der alten IPv4-Welt zu behalten, während intern bereits alles zukunftssicher auf IPv6 läuft.

### 2. IPsec-VPN-Verschlüsselung zwischen den Standorten

* **Was ist das?** Verschlüsselte Tunnel über das Internet, die deine Standorte (Hamburg, Lübeck, Berlin, München) sicher miteinander verbinden.
* **Warum wichtig?** Aktuell schicken deine Router die Daten im Packet Tracer unverschlüsselt über die WAN-Strecken. Für ein echtes Unternehmen ist eine **Site-to-Site VPN-Verschlüsselung** via IPsec (oder GRE-Tunnel mit IPsec) absolute Pflicht, damit niemand die internen Daten im Internet abfangen kann.

### 3. Ausfallsicherheit mit First Hop Redundancy (VRRPv3)
* **Was ist das?** Ein ausfallsicheres Gateway-Protokoll (Virtual Router Redundancy Protocol für IPv6).
* **Warum wichtig?** Wenn an einem Standort der Core-Switch oder Router die Grätsche macht, sind die PCs offline. Mit VRRPv3 stellst du zwei Router nebeneinander, die sich eine virtuelle IPv6-Adresse teilen. Fällt einer aus, übernimmt der andere in Millisekunden – komplett unbemerkt für die Mitarbeiter. *(Das ist das Protokoll, das wir vorhin fälschlicherweise im Visier hatten – für IPv6 nutzt man im Multi-Vendor-Bereich am liebsten das standardisierte VRRPv3).*
### 4. Einbindung von IPv6-Subnetting-Tools (IPAM)
* **Was ist das?** IP Address Management (IPAM) Software (wie NetBox oder phpIPAM).
* **Warum wichtig?** IPv6-Adressen sind verdammt lanEin automatisiertes IPAM-System dokumentiert, welche `/64` Netze an welchem Standort vergeben sind, und verhindert Adresskonflikte.

Damit holst du dir definitiv die Bestnote im Fachgespräch!
```

**Letzte Aktualisierung:** 2026-06-03  
**Projektleitung:** Salkinhopsongit
