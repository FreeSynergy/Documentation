# Node — Plattformen & Kompatibilität

[← Zurück zum Index](../INDEX.md) | [Node](../programme/node/README.md) | [Architektur](../architektur/uebersicht.md)

---

## Die Entscheidung: Node ist Linux-only

Node läuft ausschließlich auf **Linux**. Das ist keine technische Einschränkung — es ist eine bewusste Entscheidung.

**Warum?** Der Container App Manager braucht Podman + Quadlet + systemd. Diese Kombination existiert nur auf Linux. Alles andere würde bedeuten, dass wir für jede Plattform eine andere Container-Lösung bauen müssten — das ist Arbeit ohne Mehrwert.

**Was das bedeutet:** Wer einen Node betreiben will, braucht Linux. Wer kein Linux hat, verbindet sich mit einem fremden Node — oder kauft sich eine kleine Linux-Box.

---

## Node als Single Point of Truth

Weil Node die einzige Stelle ist, die weiß wo Container laufen, welche Hosts existieren und wer ins Netz darf, gilt:

> **Wenn wir irgendwann macOS oder Windows als Node-Plattform unterstützen wollen, ist es eine Änderung an einer Stelle — Node.**

Desktop, Store, Container App Manager, alle anderen Programme müssen nicht angefasst werden. Node abstrahiert die Plattform vollständig nach oben.

---

## Plattform-Übersicht

| Plattform | Als Node? | Als Desktop-Client? | Anmerkung |
|---|---|---|---|
| Linux (Server) | ✅ Ideal | ✅ | Volle Unterstützung |
| Linux (Desktop) | ✅ Möglich | ✅ | Node + Desktop auf einer Maschine |
| Raspberry Pi | ✅ Sehr gut | — | Günstigste Home-Node-Option |
| Windows 11 (WSL2) | ⚠️ Möglich | ✅ | Node in WSL2, mit Einschränkungen |
| Windows (nativ) | ❌ | ✅ | Kein systemd → kein Node |
| macOS (Podman machine) | ⚠️ Möglich | ✅ | Podman läuft via Linux-VM (QEMU) |
| macOS (nativ) | ❌ | ✅ | Kein systemd → kein Node |
| iOS | ❌ | ✅ (später) | Immer Remote-Client |
| Android | ❌ | ✅ (später) | Immer Remote-Client |

**Legende:**
- ✅ = Empfohlen / voll unterstützt
- ⚠️ = Technisch möglich, aber nicht offiziell unterstützt
- ❌ = Nicht möglich (keine Container-Runtime)

---

## Szenarien

### Szenario 1 — Linux-Server (optimal)

```
Linux-Server (VPS oder Heimserver)
└── Node läuft als systemd-Dienst
    └── Container App Manager verwaltet alle Services
    └── Desktop-Clients verbinden sich von überall
```

Empfehlung für alle die ernsthaft mit FreeSynergy arbeiten wollen.

### Szenario 2 — Raspberry Pi als Home Node

```
Raspberry Pi 4/5 (35–80€, 5W Stromverbrauch)
└── Fedora / Debian / Raspberry Pi OS
    └── Node + Container App Manager
    └── Läuft 24/7, macht nichts außer Node sein
```

**Empfehlung für Heimnutzer.** Günstig, effizient, echter Linux-Server.

### Szenario 3 — Cloud-VPS

```
Hetzner CAX11 (3,29€/Monat), DigitalOcean, etc.
└── Echter Linux-Server
    └── Überall erreichbar, kein VPN nötig für Remote-Zugriff
    └── Daten liegen außerhalb des Hauses
```

Für Nutzer ohne eigene Hardware. Datenschutz-Kompromiss muss bewusst eingegangen werden.

### Szenario 4 — WSL2 auf Windows 11

```
Windows 11
└── WSL2 (systemd-Support seit 2022)
    └── Node läuft in der WSL2-VM
        └── Startet automatisch mit Windows
```

**Nicht offiziell unterstützt**, aber technisch möglich. Netzwerk-Quirks, kein echter Boot-Prozess. Für Fortgeschrittene.

### Szenario 5 — macOS mit Podman machine

```
macOS
└── Podman Desktop
    └── podman machine (= QEMU Linux-VM, unsichtbar)
        └── Node läuft in der VM
```

**Nicht offiziell unterstützt.** Node läuft eigentlich in einer Linux-VM — macOS ist nur die Hülle. Weniger stabil als echter Linux-Server.

### Szenario 6 — Kein eigener Node (reiner Desktop-Nutzer)

```
Windows / macOS / Linux Desktop
└── FreeSynergy Desktop (Standalone)
    └── Verbindet sich mit einem fremden Node (Freund, Organisation, Community)
    └── Oder: Standalone-Modus (lokale Auth, keine Services)
```

Desktop funktioniert auch ohne eigenen Node. Der Mensch verbindet sich mit einem Node dem er vertraut — oder betreibt Desktop rein lokal.

---

## Was passiert wenn ein Node nachträglich eingerichtet wird?

```
Desktop (Standalone, lokale Auth)
    ↓  Node wird irgendwo eingerichtet
Desktop erkennt Node → "Willst du dich verbinden?"
    ↓  Bestätigung
Desktop wechselt Auth: lokal → Node's Kanidm (OIDC)
Alle lokalen Daten bleiben erhalten — nur Auth-Provider wechselt
```

Der Wechsel von lokal → Node ist ein definierter, reversibler Prozess. Desktop verliert dabei keine Daten.

---

## Die Kernregel

> **Node ist ein Server-Programm, kein Desktop-Programm.**
>
> Ein Gerät das dauerhaft läuft ist ein Node-Kandidat.
> Ein Gerät das auf- und zugeklappt wird ist ein Desktop-Client.

Das ist die eigentliche Grenze — nicht das Betriebssystem.

---

Weiter: [VPN & Netzwerk](vpn.md) | [Node](../programme/node/README.md) | [Architektur](../architektur/uebersicht.md)
