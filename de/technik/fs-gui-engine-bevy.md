# fs-gui-engine-bevy

[← Zurück zum Index](../INDEX.md)

---

## Was ist das?

`fs-gui-engine-bevy` implementiert die `fs-render`-Traits via [Bevy ECS](https://bevyengine.org/).
3D-fähige Render-Engine für FreeSynergy — animierte Desktops, Workspace-Visualisierungen, Shader-Effekte.

---

## Architektur

```
BevyEngine   ← implementiert RenderEngine-Trait (fs-render)
BevyWindow   ← implementiert FsWindow-Trait
BevyWidget   ← implementiert FsWidget-Trait
BevyTheme    ← implementiert FsTheme-Trait (Bevy Resource)
FsRenderPlugin ← Bevy Plugin, registriert alle Bevy-Systeme
WorkspaceScene ← Bevy Resource, Builder Pattern (Grid / Circular / Freeform)
AnimatedBackground ← Bevy Component (Particles / Gradient / WaveField / Static)
```

**Adapter Pattern**: BevyEngine adaptiert Bevy ECS für die `fs-render`-Traits.
Consumer-Code kennt nur `dyn RenderEngine` aus `fs-render`.

---

## Capability

| Feld           | Wert                        |
|----------------|-----------------------------|
| ID             | `render.engine.bevy`        |
| Registriert in | `fs-registry`               |

---

## 3D-API

`fs-render` bietet `Fs3dExtension` + `Fs3dDescriptor` als optionale Erweiterung.
`BevyEngine` nutzt diese API für Workspace-Szenen ohne die 2D-FsView-Basis zu ändern.

---

## Repo

- Lokal: `/home/kal/Server/fs-gui-engine-bevy/`
- GitHub: `git@github.com:FreeSynergy/fs-gui-engine-bevy.git`

---

Weiter: [fs-render](../programme/fs-render.md) | [← Index](../INDEX.md)
