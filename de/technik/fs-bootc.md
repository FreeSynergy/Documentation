# fs-bootc

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-bootc` baut und publiziert FreeSynergy bootc OS-Images.
Basiert auf [Fedora bootc](https://fedoraproject.org/bootc/) und erzeugt OCI-Images
die auf Bare-Metal oder in VMs via `bootc install to-disk` genutzt werden können.

---

## Varianten

| Variante         | Basis                           | Inhalt                              |
|------------------|---------------------------------|-------------------------------------|
| `fs-server`      | `fedora-bootc:41`               | Headless, kein Wayland              |
| `fs-workstation` | `fedora-bootc:41`               | Wayland, Mesa, PipeWire             |

---

## Architektur

```
ImageVariant-Trait   ← Strategy Pattern
       ↑
ServerImage + WorkstationImage  ← konkrete Implementierungen
       ↑
builder::build() + builder::push()  ← podman invocations
       ↑
fs-bootc CLI (clap)
```

---

## CLI

```
fs-bootc build server [--tag latest]
fs-bootc build workstation [--tag latest]
fs-bootc push server [--tag latest]
fs-bootc push workstation [--tag latest]
```

---

## Containerfiles

| Datei                      | Beschreibung                  |
|----------------------------|-------------------------------|
| `Containerfile.server`     | Headless Server-Image         |
| `Containerfile.workstation`| Workstation-Image mit Wayland |

---

## GitHub Actions (offen)

Build + Push zu `ghcr.io/freesynergy/` bei Tag `v*` — noch nicht implementiert.

---

## Repo

- Lokal: `/home/kal/Server/fs-bootc/`
- GitHub: `git@github.com:FreeSynergy/fs-bootc.git`

---

Weiter: [← Index](../INDEX.md)
