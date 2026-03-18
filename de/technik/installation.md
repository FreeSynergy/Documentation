# Installation & Invite-System

[← Zurück zum Index](../INDEX.md) | [Init](../programme/init/README.md) | [Node](../programme/node/README.md)

---

## Der Installationsweg

```
1. User lädt FreeSynergy.Init herunter
   (vorkompiliert von GitHub Releases ODER cargo install)

2. Init klont den Store via gitoxide

3. Store übernimmt — User wählt was installiert werden soll:

   Server:
     fsn store install server-minimal    # Node + Container App Manager + Proxy + S3
     fsn store install server-full       # + Mail + Wiki + Chat + Git + ...

   Desktop:
     fsn store install desktop           # Desktop + Themes + Sprachen

   Einzeln:
     fsn store install kanidm
     fsn store install outline

4. Pakete werden installiert:
   - Abhängigkeiten aufgelöst
   - Signatur geprüft
   - pre_install Script ausgeführt
   - Dateien installiert
   - post_install Script ausgeführt

5. Bei Node-Installation:
   - S3-Server wird gestartet (Teil von Node)
   - Storage-Backend konfiguriert
   - Container App Manager wird gestartet
   - Services werden installiert
```

## Zwei Installationspfade

### 1. Node (Server)

1. Init → Store → `fsn store install server-minimal`
2. Neues Netzwerk erstellen ODER bestehendem beitreten (Invite-Datei)
3. IP-Typ (statisch → aktiver Modus, dynamisch → passiver Modus)
4. Sprachen auswählen
5. Zeitzone (nur für Anzeige — Server laufen immer UTC)
6. Store-Quellen konfigurieren
7. IAM wählen (Kanidm empfohlen)
8. Proxy wählen (Zentinel empfohlen)
9. Weitere Services installieren

### 2. Desktop (Client)

1. Init → Store → `fsn store install desktop`
2. Invite-Datei einlesen (Token + Host-Adresse + CA-Hash)
3. TLS-Verbindung aufbauen (CA-Hash verifiziert)
4. IAM-Login via OAuth2/OIDC
5. Fertig — Desktop ist API-Client

### 3. Mobile (Android/iOS)

Kein Init, kein Store. Desktop-App aus dem App Store (Google Play / Apple). Verbindet sich per Invite-Token mit einem bestehenden Node. Reiner Client.

## Invite-Datei

```toml
[invite]
token = "a1b2c3.x9y8z7w6v5u4t3s2"
ca_hash = "sha256:4f8a..."
host = "node1.example.com:9443"
network_id = "uuid-..."
created_at = "2026-03-16T12:00:00Z"
expires_at = "2026-03-17T12:00:00Z"
permissions = "full"    # full | readonly | desktop-only
```

Verschlüsselt. Port pro Einladung (öffnen/schließen). CA-Hash verhindert MITM.

## mDNS Discovery (optional)

Im LAN: `_freesynergy._tcp.local`. Token trotzdem erforderlich.

---

Weiter: [Init](../programme/init/README.md) | [Store](../programme/store/README.md) | [Sicherheit](sicherheit.md)
