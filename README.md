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
```

**Letzte Aktualisierung:** 2026-06-03  
**Projektleitung:** Salkinhopsongit
