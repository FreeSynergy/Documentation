# Storage-Layer (S3)

[← Zurück zum Index](../INDEX.md) | [Node](../programme/node/README.md)

---

## Philosophie

Statt dass jeder Service seine Dateien isoliert in eigenen Container-Volumes speichert, gibt es einen **zentralen Storage-Layer**. Alle Programme schreiben an denselben Ort. Das macht Dateien durchsuchbar, backupbar und teilbar.

## Architektur: Eigener S3-Server in Rust

Kein Garage. Kein MinIO. Kein externer Container. Der Node **ist** der S3-Server.

```
Node
├── S3-Server (s3s Crate)
│   ├── S3-API: https://s3.node1.helfa.org   ← für Services die S3 reden
│   └── FUSE-Mount: /mnt/fsn-storage/        ← für alles andere (normales Dateisystem)
│
├── Backend (opendal Crate, austauschbar):
│   ├── Lokal: /var/lib/freesynergy/storage/
│   ├── Hetzner Storagebox (SFTP)
│   ├── Anderer Host (verteilter S3)
│   └── Beliebig (40+ Backends via opendal)
```

**Drei Zugriffswege, ein Speicher:**

1. **S3-API** — Services die S3-Protokoll sprechen (Outline, Forgejo LFS, Matrix) reden direkt mit unserem S3-Server
2. **Dateisystem** — Programme die einfach Dateien lesen/schreiben, nutzen `/mnt/fsn-storage/` wie einen normalen Ordner
3. **Node-API** — Für programmatischen Zugriff über die Node-REST-API

## Warum eigener S3-Server?

- **Kein externer Container** — läuft direkt im Node, kein Garage/MinIO nötig
- **Alles Rust** — s3s + opendal, passt in unser Ökosystem
- **Backend austauschbar** — heute lokale Disk, morgen Hetzner Storagebox, übermorgen verteilt
- **Volle Kontrolle** — wir bestimmen Rechte, Verschlüsselung, Routing

## Bucket-Struktur

```
/mnt/fsn-storage/
├── profiles/        ← Öffentliche Visitenkarten (extern lesbar)
├── backups/         ← Rustic-Backups (nur intern)
├── media/           ← Medien-Dateien aller Services
│   ├── outline/     ← Wiki-Bilder, Dokumente
│   ├── forgejo/     ← Git LFS, Releases
│   ├── matrix/      ← Chat-Media
│   └── cryptpad/    ← Verschlüsselte Dokumente
├── packages/        ← Store-Pakete (lokal gecached)
└── shared/          ← Geteilte Dateien zwischen Services
```

## Intern vs. Extern

**Standard: Alles ist intern** (nur im Podman-Netzwerk erreichbar).

Einzelne Buckets können über [Zentinel](../programme/node/README.md) (Proxy) extern freigegeben werden:

```toml
# Zentinel-Konfiguration
[[proxy.public_buckets]]
bucket = "profiles"
access = "read"          # Nur lesen, nicht schreiben
url = "profiles.node1.helfa.org"

# Alles andere: NICHT von außen erreichbar
```

## Profile als öffentliche Visitenkarten

Jeder User kann ein öffentliches Profil haben:

```
profiles/superman.json         ← Maschinenlesbar
profiles/superman/avatar.webp  ← Profilbild
```

```json
{
  "id": "superman",
  "name": "Superman",
  "bio": "FreeSynergy Maintainer",
  "avatar": "superman/avatar.webp",
  "links": {
    "website": "https://freesynergy.net",
    "matrix": "@superman:matrix.helfa.org",
    "forgejo": "https://git.helfa.org/superman"
  },
  "pgp_key": "-----BEGIN PGP PUBLIC KEY BLOCK-----\n..."
}
```

Erreichbar unter: `https://profiles.node1.helfa.org/superman.json`

**Nutzen:**
- Andere Nodes/Föderationen holen das Profil automatisch ab → Profil ist sofort überall ausgefüllt
- Kein manuelles Eintippen bei Föderation-Beitritt
- ActivityPub-kompatibel erweiterbar

## Verteilter Speicher über mehrere Hosts

Wenn ein Projekt mehrere Hosts hat, kann der Storage verteilt werden:

```
Host 1 (Köln):   Node mit S3-Server → 500 GB lokale Disk
Host 2 (Berlin): Node mit S3-Server → 500 GB lokale Disk
                  ───────────────────────────────
                  opendal verbindet beide → 1 TB Gesamt-Speicher
```

Die Nodes reden untereinander S3-Protokoll. opendal abstrahiert das — für die Services sieht es aus wie ein einziger Speicher.

## Storagebox als Backend

Hetzner Storagebox wird über opendal als SFTP-Backend eingebunden:

```rust
// opendal Konfiguration
let backend = Sftp::default()
    .endpoint("u123456.your-storagebox.de")
    .user("u123456")
    .key("/path/to/ssh-key")
    .root("/backups")
    .build()?;
```

Backup und Storage in einem. Rustic schreibt über S3-API in den `backups/` Bucket, der landet auf der Storagebox.

## Durchsuchbarkeit

Weil alle Dateien an einem Ort liegen, kann [Search](../programme/search/README.md) den Storage durchsuchen:

- Dateinamen-Suche über alle Buckets
- Volltext-Suche in Text-Dateien
- Metadaten-Suche (wer hat wann was hochgeladen)
- Rechte werden beachtet: Nur Buckets durchsuchen auf die man `search`-Recht hat

## Bibliotheken

| Crate | Zweck |
|---|---|
| `s3s` | S3-Server-Framework (Samsung, aktiv maintained) |
| `s3s-fs` | Referenz-Backend: lokales Dateisystem |
| `opendal` | 40+ Storage-Backends (Apache-Projekt) |
| `rust-s3` | S3-Client (für Programme die mit unserem S3 reden) |

## Installation

S3 ist **Kern-Infrastruktur** — kein optionales Paket. Der Node installiert den S3-Server automatisch beim Setup, noch vor allen anderen Services.

```
Node-Installation:
  1. Node-Binary installieren
  2. S3-Server starten (Teil von Node)          ← HIER
  3. Storage-Backend konfigurieren
  4. Container App Manager starten
  5. Services installieren (nutzen S3)
```

---

Weiter: [Node](../programme/node/README.md) | [Sicherheit](sicherheit.md) | [Search](../programme/search/README.md)
