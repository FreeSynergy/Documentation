# fs-i18n

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-i18n` ist die Internationalisierungs-Bibliothek und der i18n-Service für FreeSynergy.
Sie basiert auf [Mozilla Fluent](https://projectfluent.org/) und stellt zwei Ebenen bereit:

1. **Library** — synchrone `t()` / `t_with()` API; in jeden Prozess eingebunden
2. **Daemon** — gRPC + REST Service; OCI Artifact Pull für Sprachpakete; inotify Hotplugging

---

## Architektur

```
I18n (lib)
  ├── t(key)         → Translation          (schnell, in-process)
  ├── t_with(key, [args])                   (Fluent-Variable-Substitution)
  ├── init(dir)                             (.ftl Dateien laden)
  └── LanguageProvider-Trait               (pluggable Backends)

fs-i18n (Daemon)
  ├── LocaleLoader   — lädt .ftl Dateien aus Verzeichnis
  ├── HotplugWatcher — inotify auf locales/, reload ohne Restart
  ├── ArtifactPuller — ArtifactSource-Trait (OCI via skopeo)
  └── AppState       — Arc<RwLock<…>>, geteilt zwischen gRPC + REST
```

---

## Library-API

| Funktion                          | Beschreibung                                      |
|-----------------------------------|---------------------------------------------------|
| `init(dir)`                       | .ftl Dateien aus Verzeichnis laden                |
| `init_with_lang(dir, lang)`       | mit expliziter Sprache initialisieren             |
| `init_with_builtins(lang)`        | nur eingebaute Snippets laden (kein Dateisystem)  |
| `t(key)`                          | Schlüssel übersetzen → `Translation`              |
| `t_with(key, &[("var","val")])`   | mit Variablen übersetzen                          |
| `set_active_lang(lang)`           | Sprache zur Laufzeit wechseln                     |
| `active_lang()`                   | aktuelle Sprache abfragen                         |
| `locale()`                        | Formatierungsregeln der aktiven Sprache           |

---

## Daemon-API

### gRPC (Port 50052)

Proto-Datei: `proto/i18n.proto`

| RPC                  | Beschreibung                                   |
|----------------------|------------------------------------------------|
| `Translate`          | Schlüssel + optionale Variablen übersetzen     |
| `AvailableLanguages` | Installierte Sprachen abfragen                 |
| `Reload`             | .ftl Dateien neu laden                         |
| `PullArtifact`       | Sprachpaket aus OCI Registry nachladen         |

### REST (Port 8082)

| Methode | Pfad                    | Beschreibung                        |
|---------|-------------------------|-------------------------------------|
| `GET`   | `/translate/{key}`      | Schlüssel übersetzen                |
| `GET`   | `/languages`            | Verfügbare Sprachen                 |
| `POST`  | `/reload`               | Übersetzungen neu laden             |
| `POST`  | `/artifacts/pull`       | OCI Sprachpaket nachladen           |
| `GET`   | `/health`               | Health-Probe                        |
| `GET`   | `/swagger-ui`           | OpenAPI-Dokumentation               |

---

## Translation-Dateien

Alle `.ftl` Dateien liegen in `fs-i18n/locales/{lang}/{programm}.ftl`:

```
locales/
  en/
    common.ftl          — wiederverwendbare Fehlermeldungen
    db.ftl              — fs-db Meldungen
    db-engine-sqlite.ftl
    db-engine-postgres.ftl
    llm-mistral.ftl
    llm-openai.ftl
    channel-matrix.ftl
    channel-telegram.ftl
    …
  de/
    (gleiche Struktur)
```

Kein roher String im Code — alles über FTL-Keys. Keys: `kebab-case` mit Bindestrichen.

---

## Hotplugging

`HotplugWatcher` überwacht `locales/` via inotify.
Bei Änderung an `.ftl` Dateien wird der Bundle automatisch neu geladen — kein Restart nötig.

---

## OCI Artifact Pull

Sprachpakete können als OCI Artifacts aus einer Registry nachgeladen werden:

```
ArtifactSource-Trait → SkopeoArtifactSource (via skopeo CLI)
```

Format: `oci:ghcr.io/freesynergy/fs-i18n-de:latest`
Nach dem Pull: `.ftl` Dateien in `locales/` → HotplugWatcher lädt nach.

---

## Konfiguration (Umgebungsvariablen)

| Variable              | Standard                               | Beschreibung                     |
|-----------------------|----------------------------------------|----------------------------------|
| `FS_I18N_LOCALES_DIR` | `/var/lib/freesynergy/i18n/locales`    | Verzeichnis der .ftl Dateien     |
| `FS_I18N_LANG`        | `en`                                   | Aktive Sprache                   |
| `FS_GRPC_ADDR`        | `0.0.0.0:50052`                        | gRPC-Adresse                     |
| `FS_REST_ADDR`        | `0.0.0.0:8082`                         | REST-Adresse                     |

---

## Repo

- Lokal: `/home/kal/Server/fs-i18n/`
- GitHub: `git@github.com:FreeSynergy/fs-i18n.git`

---

Weiter: [i18n-Konzept](i18n.md) | [← Index](../INDEX.md)
