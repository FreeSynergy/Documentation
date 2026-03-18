# Resource Builder — Pakete bauen

[← Zurück zum Index](../../INDEX.md) | [Ressourcen](../../konzepte/ressourcen.md) | [Store](../store/README.md)

---

## Was der Resource Builder macht

Der Resource Builder (`fsn-builder`) baut Ressourcen für den Store. Er prüft ob alle Daten da sind, validiert, und deployt ins Store-Git.

**Abgrenzung:**
- **Container App Manager** → stellt Container-Apps EIN (konfiguriert, startet)
- **Resource Builder** → BAUT Pakete (analysiert, validiert, veröffentlicht)

## Workflow: YAML → Store-Paket

```
Schritt 1: YAML eingeben
  → User gibt docker-compose.yml

Schritt 2: Parsen
  → Services erkennen (Haupt + Sub)
  → Images, Ports, Volumes, Networks, Healthchecks

Schritt 3: Variablen analysieren
  → Typ + Rolle + Konfidenz pro Variable
  → variables.toml generieren (bearbeitbar)

Schritt 4: Fehlende Daten identifizieren
  → Kein Healthcheck? ⚠️ Warnung
  → Keine Rollen erkannt? Frage an User
  → Store fragen: "Kennst du dieses Image?" → Ergänzen

Schritt 5: Service-Verknüpfungen
  → Variablen die auf andere Services zeigen
  → SMTP_HOST → "Welcher SMTP-Service?"
  → REDIS_URL → "Welcher Cache, welcher Slot?"

Schritt 6: Bearbeitbare Ausgabe
  → manifest.toml (Metadaten, Rollen, Tags)
  → variables.toml (Variablen mit Typ und Verknüpfung)
  → quadlet.container (generiert)
  → config-templates/ (Tera-Dateien)

Schritt 7: Validierung
  → Alle Pflichtfelder da?
  → Alle Abhängigkeiten erfüllbar?
  → Alle Variablen typisiert?
  → Icon vorhanden?
  → Status: ✅ / ⚠️ / ❌

Schritt 8: In Git laden
  → Paket ins Store-Repo committen
  → Signieren
  → Ab jetzt für alle verfügbar
```

## Interfaces

| Interface | Beispiel |
|---|---|
| CLI | `fsn builder analyze compose.yml` |
| CLI | `fsn builder validate ./my-package/` |
| CLI | `fsn builder publish ./my-package/` |
| UI | Builder-View im Desktop |

## Was der Builder prüft

| Prüfung | Pflicht? | Ergebnis |
|---|---|---|
| id vorhanden | ✅ | ❌ wenn fehlt |
| name vorhanden | ✅ | ❌ wenn fehlt |
| version (SemVer) | ✅ | ❌ wenn ungültig |
| icon (SVG) | ✅ | ❌ wenn fehlt |
| description > 10 Zeichen | ✅ | ⚠️ wenn zu kurz |
| tags nicht leer | ✅ | ⚠️ wenn leer |
| YAML valide (Container-Apps) | ✅ | ❌ wenn Syntaxfehler |
| Alle Variablen typisiert | ✅ | ⚠️ wenn untypisiert |
| Healthcheck vorhanden | Empfohlen | ⚠️ wenn fehlt |
| Rollen definiert | Empfohlen | ⚠️ wenn fehlt |
| Bridge-Methoden vollständig | Wenn Bridge | ❌ wenn unvollständig |
| Bundle-Referenzen existieren | Wenn Bundle | ❌ wenn Referenz fehlt |
| Signatur | Bei Publish | ❌ wenn unsigniert |

## Repo

https://github.com/FreeSynergy/Builder

Der Builder ist ein eigenständiges Programm mit eigenem Repo. Er kann ohne Node und ohne Desktop eingesetzt werden — z.B. in CI/CD-Pipelines um Pakete zu bauen und in den Store zu veröffentlichen.

---

Weiter: [Ressourcen](../../konzepte/ressourcen.md) | [Container App Manager](../container_app/README.md) | [Store](../store/README.md)
