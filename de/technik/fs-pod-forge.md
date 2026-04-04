# fs-pod-forge — Container YAML Configurator

[← Zurück zur Technik-Übersicht](../INDEX.md)

---

## Zweck

`fs-pod-forge` stellt die Abstraktion bereit, mit der jedes Container-Paket seine eigene `pod.yml` generieren kann. Der Manager ruft den `PodConfigurator`-Trait auf — er muss das konkrete Paket nicht kennen.

**Repo:** `git@github.com:FreeSynergy/fs-pod-forge.git`  
**Lokal:** `/home/kal/Server/fs-pod-forge/`  
**Binary:** `fs-pod`

---

## Design Pattern

| Pattern | Verwendung |
|---|---|
| **Strategy** | `PodConfigurator`-Trait — je Paket eine eigene Implementierung |
| **Builder** | `PodManifestBuilder` — typsicherer, fluenter Manifest-Aufbau |
| **Template Method** | `BasePodConfigurator` — gemeinsame `validate` / `export_yaml` / `diff` Logik |

---

## Architektur

```
PodConfigurator (trait)
  ├── generate(config) → PodManifest
  ├── validate(manifest) → ValidationResult
  ├── export_yaml(manifest) → String (standalone pod.yml)
  └── diff(old, new) → ManifestDiff

BasePodConfigurator
  └── shared impl für validate, export_yaml, diff

PodManifestBuilder (fluent API)
  └── PodManifest (pod_name, containers, volumes, secrets, network_policies)
        ├── ContainerSpec (image, ports, volume_mounts, env, restart_policy)
        ├── PodPort (host_port, container_port, protocol)
        ├── PodVolume (name, host_path?)
        ├── PodEnvVar (name, EnvValue)
        │     └── EnvValue: Plain | SecretRef | EnvRef
        └── PodSecret (name, entries)

ManifestDiff
  └── DiffItem: ContainerAdded/Removed, ImageChanged, PortAdded/Removed, …

SystemdUnit
  └── generate_systemd_unit(manifest, pod_yml_path) → SystemdUnit
```

---

## PodConfig

```rust
let mut config = PodConfig::default();
config.values.insert("domain".into(), "idm.example.com".into());
config.values.insert("https_port".into(), "8443".into());
```

`PodConfig::require(key)` gibt einen `Err` zurück wenn der Key fehlt.

---

## Implementierungen

| Paket | Configurator | Crate |
|---|---|---|
| Kanidm | `KanidmPodConfigurator` | `fs-manager-auth` |
| Stalwart | `StalwartPodConfigurator` | `fs-manager-mail` |

Weitere Pakete implementieren `PodConfigurator` eigenständig.

---

## CLI (`fs-pod`)

```bash
# Manifest validieren
fs-pod validate /etc/freesynergy/kanidm/pod.yml

# Diff zwischen zwei Manifesten
fs-pod diff old.yml new.yml

# Systemd-Unit generieren
fs-pod systemd /etc/freesynergy/kanidm/pod.yml
```

---

## Workflow (Manager-Integration)

```
1. Admin gibt Config-Werte ein (UI oder CLI)
2. Manager baut PodConfig
3. Configurator::generate(config) → PodManifest
4. Configurator::validate(manifest) → ValidationResult prüfen
5. Configurator::export_yaml(manifest) → Datei schreiben
6. podman play kube pod.yml (via fs-container oder direkt)
7. Systemd-Unit aktivieren (via fs-manager-core SystemdServiceController)
```

---

## i18n

FTL-Keys in `fs-i18n/locales/{lang}/pod-forge.ftl`:

- `pod-forge-error-missing-key`
- `pod-forge-error-port-conflict`
- `pod-forge-error-unknown-volume`
- `pod-forge-error-unknown-secret`
- `pod-forge-cli-validate-ok` / `-invalid`
- `pod-forge-cli-diff-*`
- `pod-forge-apply-success` / `-failed`
