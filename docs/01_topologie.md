# Schritt 1: Netzwerk-Topologie aufbauen

## Übersicht

In diesem Schritt erstellen wir die Basis-Topologie des IPv6-Netzwerks in Cisco Packet Tracer.

## Erforderliche Geräte

| Gerät | Anzahl | Modell |
|-------|--------|--------|
| Router | 2 | Cisco 2911 |
| Switch | 2 | Cisco 2960 |
| PCs | 4 | Standard PC |
| Kabel | - | Copper Straight-Through |

## Aufbau-Schritte

### 1. Neue Datei erstellen
- Öffne Cisco Packet Tracer
- Wähle `File > New` oder `Ctrl + N`
- Speichere die Datei als `ipv6_netzwerk.pkt`

### 2. Router hinzufügen
- Gehe zum Bereich "Devices" auf der linken Seite
- Wähle "Network Devices > Routers"
- Wähle einen **Cisco 2911 Router**
- Platziere ihn im Workspace (oben links)
- Wiederhole dies für einen zweiten Router (oben rechts)

### 3. Switches hinzufügen
- Gehe zu "Network Devices > Switches"
- Wähle **Cisco 2960 Switch**
- Platziere einen Switch unter Router 1
- Platziere einen Switch unter Router 2

### 4. PCs hinzufügen
- Gehe zu "End Devices"
- Wähle **PC**
- Platziere 2 PCs unter Switch 1
- Platziere 2 PCs unter Switch 2

### 5. Verbindungen erstellen
- Wähle das **Copper Straight-Through** Kabel (gelbes Kabel)
- Verbinde Router 1 (GigabitEthernet 0/0) mit Router 2 (GigabitEthernet 0/0)
- Verbinde Router 1 (GigabitEthernet 0/1) mit Switch 1 (GigabitEthernet 0/1)
- Verbinde Router 2 (GigabitEthernet 0/1) mit Switch 2 (GigabitEthernet 0/1)
- Verbinde PC 1 und PC 2 mit Switch 1
- Verbinde PC 3 und PC 4 mit Switch 2

## Netzwerk-Diagramm

```
┌──────────────────────────────────────────────────────┐
│                 PACKET TRACER TOPOLOGIE              │
├──────────────────────────────────────────────────────┤
│                                                      │
│  ┌──────────────┐              ┌──────────────┐    │
│  │   Router 1   │──────────────│   Router 2   │    │
│  │ (2911)       │  Ge0/0-Ge0/0 │ (2911)       │    │
│  └──────┬───────┘              └──────┬───────┘    │
│         │ Ge0/1                       │ Ge0/1      │
│         │                             │            │
│  ┌──────▼───────┐              ┌──────▼───────┐   │
│  │  Switch 1    │              │  Switch 2    │   │
│  │  (2960)      │              │  (2960)      │   │
│  └──┬──────┬────┘              └──┬──────┬────┘   │
│     │      │                      │      │        │
│  ┌──▼┐  ┌──▼┐                  ┌──▼┐  ┌──▼┐    │
│  │PC1│  │PC2│                  │PC3│  │PC4│    │
│  └───┘  └───┘                  └───┘  └───┘    │
│                                                      │
└──────────────────────────────────────────────────────┘
```

## Verifikation

Nach dem Aufbau sollte Folgendes überprüft werden:

- [x] Alle 2 Router sind sichtbar
- [x] Alle 2 Switches sind sichtbar
- [x] Alle 4 PCs sind sichtbar
- [x] Alle Verbindungen sind mit grünen Linien markiert (eingeschaltet)
- [x] Keine fehlerhaften Verbindungen (keine roten/orangefarbenen Linien)

## Nächster Schritt

Wenn die Topologie erfolgreich aufgebaut wurde, gehe zu [Schritt 2: Router-Konfiguration](./02_router_config.md).

---

**Status**: ✅ Topologie erstellt  
**Dauer**: ~10-15 Minuten
