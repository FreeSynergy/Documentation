# VPN & Netzwerk

[← Zurück zum Index](../INDEX.md) | [Node](../programme/node/README.md) | [Node-Plattformen](node-plattformen.md)

---

## Der Node als Netzwerk-Knoten

"Node" bedeutet nicht nur "Verwaltungsprogramm" — es bedeutet **Knoten im Netz**. Der Node ist der Punkt, über den alles läuft: wer rein darf, wer raus darf, welche Verbindungen existieren.

```
Dein Node
├── Lässt deine Geräte rein        (VPN, WireGuard)
├── Lässt befreundete Nodes rein   (Föderation)
├── Entscheidet was nach draußen darf
└── Verbindet sich mit anderen Nodes zu einem Mesh
```

Das ist der Unterschied zu allem was es bisher gibt: **Das Netz gehört niemandem — und allen.**

---

## Warum VPN?

Ohne VPN funktioniert der Heimzugriff nur im lokalen Netz. Sobald du das Haus verlässt, ist dein Node nicht mehr erreichbar — außer du öffnest Ports in deiner Firewall (unsicher, wartungsintensiv).

Mit VPN:
- Mobiles Gerät tunnelt über das Internet direkt zum Node
- Node entscheidet wer sich verbinden darf
- Alle lokalen Services sind erreichbar als wäre man zuhause
- Kein offener Port, keine DMZ

---

## WireGuard

FreeSynergy setzt auf **WireGuard** als VPN-Protokoll:

| Eigenschaft | Wert |
|---|---|
| Protokoll | WireGuard (UDP) |
| Verschlüsselung | ChaCha20, Poly1305, Curve25519 |
| Konfiguration | Node verwaltet alles automatisch |
| Performance | Sehr niedrige Latenz, minimal Overhead |
| Kernelintegration | Im Linux-Kernel seit 5.6 |

Der Mensch muss **nie** WireGuard-Konfigurationen anfassen. Node generiert Keys, verteilt sie an Clients, verwaltet das Regelwerk.

---

## Heimnetz-Szenario

### Einrichten (einmalig)

```
Node auf Heimrechner → Einladung erstellen → QR-Code
    ↓
Mobiles Gerät scannt QR
    ↓
Desktop-App kennt jetzt:
  - Node-Adresse (IP oder hostname)
  - CA-Zertifikat (mTLS)
  - OIDC-Client-ID
  - WireGuard-Konfiguration
    ↓
Einmal anmelden mit Passwort/Passkey
→ Refresh-Token gespeichert, WireGuard-Peer angelegt
```

### Zuhause ankommen

```
Handy verbindet sich mit Heim-WLAN
    ↓
Desktop-App erkennt Node via mDNS (direkte Verbindung, kein VPN nötig)
    ↓
Refresh-Token noch gültig? → automatisch eingeloggt
Abgelaufen? → Biometrie (Face ID / Fingerprint) → neu eingeloggt
    ↓
Alle lokalen Services verfügbar
```

### Unterwegs

```
Handy im Mobilfunknetz
    ↓
WireGuard-Tunnel öffnet sich automatisch zum Node
    ↓
Alle Services erreichbar als wäre man zuhause
Node entscheidet was erlaubt ist (Rechte-Kaskade)
```

---

## Token-Strategie

| Token-Typ | Laufzeit | Erneuerung |
|---|---|---|
| Access-Token | 15 Minuten | Automatisch via Refresh-Token |
| Refresh-Token | 30 Tage | Bei jedem Heimnetz-Besuch erneuert |
| WireGuard-Key | Dauerhaft | Manuell rotierbar |

Solange der Mensch mindestens einmal im Monat zuhause ist, passiert nichts. Refresh-Token abgelaufen → einmal Biometrie → fertig.

---

## Node-Mesh — Das eigentliche Ziel

Einzelne Nodes verbinden sich zu einem **Mesh-Netzwerk**:

```
Node A (Familie)  ←──────────────→  Node B (Verein)
        │                                    │
        └──────────────┬─────────────────────┘
                       │
               Node C (Freund)
                       │
               Node D (Community)
```

Jeder Node:
- Entscheidet selbst welchen anderen Nodes er vertraut
- Gibt nur die Rechte weiter die er selbst hat (Rechte-Kaskade)
- Kommuniziert über Föderations-Protokolle (ActivityPub, OIDC, SCIM)
- Kann Services anderer Nodes einbinden (mit Erlaubnis)

**Das ist das dezentrale Netz** das wir bauen. Keine großen Plattformen dazwischen. Keine Datensilo-Betreiber. Jeder Knoten ist souverän.

---

## Abgrenzung: Was VPN ist, was Föderation ist

| | VPN (WireGuard) | Föderation (ActivityPub/OIDC) |
|---|---|---|
| Zweck | Netzwerk-Zugang | Daten- und Identitäts-Austausch |
| Wer nutzt es | Eigene Geräte | Andere Nodes / Organisationen |
| Wann | Immer wenn du unterwegs bist | Wenn Nodes zusammenarbeiten |
| Verwaltet von | Node (automatisch) | Node (manuell konfigurierbar) |

VPN ist für **deine Geräte**. Föderation ist für **andere Nodes**.

---

## Node-Zuständigkeit für Netz

Node ist die einzige Stelle die das Netzwerk kennt und verwaltet:

- WireGuard-Keys generieren und verteilen
- Peers anlegen und entfernen
- Eingehende VPN-Verbindungen erlauben/verweigern
- Routing zwischen lokalem Netz und VPN-Tunnel
- Föderations-Verbindungen aufbauen und verwalten

Kein anderes Programm (Desktop, Store, Container App Manager) hat Netzwerk-Zugriff außer über den Node.

---

Weiter: [Node-Plattformen](node-plattformen.md) | [Föderation](foederation.md) | [Rechte-Kaskade](rechte.md)
