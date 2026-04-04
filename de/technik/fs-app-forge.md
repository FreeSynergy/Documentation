# fs-app-forge — App Config File Configurator

[← Zurück zur Technik-Übersicht](../INDEX.md)

---

## Zweck

`fs-app-forge` stellt die Abstraktion bereit, mit der jedes Paket seine eigene Konfigurationsdatei schema-getrieben verwalten kann. Die Manager-UI generiert das Formular automatisch aus dem Schema — kein hardcodiertes UI.

**Repo:** `git@github.com:FreeSynergy/fs-app-forge.git`  
**Lokal:** `/home/kal/Server/fs-app-forge/`  
**Binary:** `fs-forge`

---

## Design Pattern

| Pattern | Verwendung |
|---|---|
| **Strategy** | `AppConfigurator`-Trait — je Paket eine eigene Implementierung |
| **Schema-driven UI** | `ConfigSchema → LayoutDescriptor` via `ConfigFormGenerator` |
| **Visitor** | `SchemaValidator` besucht Schema-Knoten und validiert Werte |

---

## Architektur

```
AppConfigurator (trait)
  ├── schema() → ConfigSchema
  ├── read() → ConfigValues
  ├── validate(changes) → ValidationResult
  ├── apply(changes) → Result<()>   (atomic write + backup)
  ├── export() → String             (standalone config-Datei)
  └── config_path() → &str

ConfigSchema
  └── ConfigSection[]
        └── ConfigField (key, label_key, description_key, FieldType, validator?)
              └── FieldType: String | Int | Bool | Enum | Secret | Path | List

ConfigFormGenerator
  └── ConfigSchema → LayoutDescriptor
        └── SectionLayout[]
              └── FieldLayout (key, label_key, widget: WidgetKind)
                    └── WidgetKind: TextInput | PasswordInput | Toggle | IntSpinner
                                  | Dropdown | PathPicker | ListEditor

ConfigAdapter (trait)
  ├── TomlConfigAdapter
  ├── YamlConfigAdapter
  └── EnvConfigAdapter
```

---

## Secret-Handling

Secret-Felder (`FieldType::Secret`) werden **nie im Klartext** gespeichert. Der Wert wird als `SecretRef` gespeichert:

```
SecretRef::Env(var_name)           → Wert aus Umgebungsvariable
SecretRef::File(path)             → Wert aus Datei lesen
SecretRef::PodmanSecret(name)     → Podman-Secret
```

---

## Implementierungen

| Paket | Configurator | Crate |
|---|---|---|
| Kanidm | `KanidmAppConfigurator` | `fs-manager-auth` |
| Stalwart | `StalwartAppConfigurator` | `fs-manager-mail` |

---

## Atomic Write

`apply()` schreibt die Konfiguration atomisch:

1. Aktuellen Inhalt lesen
2. Backup erstellen (`<path>.bak`)
3. Änderungen mergen
4. In temp-Datei schreiben (`<path>.tmp`)
5. `rename(<path>.tmp, <path>)` — atomar

Bei Fehler im `rename`-Schritt: Backup-Pfad im `AtomicWrite`-Error zurückgeben.

---

## CLI (`fs-forge`)

```bash
# Aktuelle Werte zeigen
fs-forge show /etc/freesynergy/kanidm/config.toml

# Config exportieren
fs-forge export /etc/freesynergy/stalwart/config.toml

# Schema anzeigen (wenn AppConfigurator registriert)
fs-forge schema kanidm
```

---

## i18n

FTL-Keys in `fs-i18n/locales/{lang}/app-forge.ftl`:

- `app-forge-error-field-required`
- `app-forge-error-type-mismatch`
- `app-forge-error-out-of-range`
- `app-forge-error-string-length`
- `app-forge-error-path-not-found`
- `app-forge-kanidm-*` / `app-forge-stalwart-*`
