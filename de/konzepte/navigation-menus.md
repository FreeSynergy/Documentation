# Navigation: Corner Menus + Side Menus

[← Zurück zum Index](../INDEX.md)

---

## Warum neue Navigation?

Klassische Sidebars belegen dauerhaft Platz, sind auf Smartphones unhandlich
und sehen auf allen Seiten gleich aus — egal was man gerade macht.

FreeSynergy nutzt stattdessen **ortsbasierte Menüs**: Jede Ecke und jede Seite
kann ein eigenes Menü enthalten. Der Ort kommuniziert Bedeutung:
Oben links = Aufgaben, Unten links = Einstellungen, usw.

---

## Corner Menus

Ein Corner Menu sitzt in einer der vier Ecken des Bildschirms.

**Indikator:** Viertelkreis — immer sichtbar, zeigt dass hier ein Menü ist.

**Aufklapp-Verhalten:**
```
Hover/Klick → Items erscheinen auf einem Kreisbogen
              Abstände gleichmäßig, radialer Fächer
              Hover auf Item → Icon wächst (HoverMagnification)
                               Nachbar-Icons wachsen proportional mit
              Klick auf Item → Aktion oder Sub-Menü (weiterer Kreisbogen)
              Esc / Klick außen → Menü schließt
```

**Sub-Menüs:** Ein Item kann eine neue Ebene öffnen (weitere Kreisbögen).
Beliebig tief schachtelbar.

**Scroll-Fallback:** Wenn mehr Items als Platz → Items werden scrollbar.
Wichtig für Mobile / kleine Screens.

---

## Side Menus

Ein Side Menu kommt von einer der vier Seiten (links, rechts, oben, unten).

**Indikator:** Halbkreis — immer sichtbar, zeigt dass hier ein Menü ist.

**Aufklapp-Verhalten (Accordion-Stil):**
```
Hover/Klick → Items wandern zuerst leicht vom Rand weg
              Dann verteilen sie sich nach oben und unten gleichzeitig
              Gleichmäßige Verteilung (muss Ränder nicht berühren)
              Wenn wenige Items: zentriert im verfügbaren Raum
              Wenn viele Items: bis zu den Rändern + Scroll-Fallback
```

**Sub-Menüs:** Item-Klick öffnet neue Accordion-Zeile (eingerückt).

---

## Sidebar Panel (Alternative)

Wer klassische Sidebars bevorzugt, wählt in **Settings > Ansicht: Menü-Stil: Sidebar**.

**Unterschied zu Corner/Side Menus:**
- Kommt als Panel **über** den Inhalt (kein Layout-Shift)
- Bildschirm-Inhalt verschiebt sich **nicht**
- Gleiche Items, anderes Aussehen

---

## HoverMagnification

Inspiriert vom macOS Dock: Icons wachsen beim Hover, Nachbarn auch.

```
base_size:  Standard-Größe (konfigurierbar in Settings > Ansicht)
max_size:   Maximale Größe beim direkten Hover
spread:     Wie weit der Vergrößerungs-Effekt auf Nachbarn ausstrahlt
```

SVG-Icons sind verlustfrei skalierbar — kein Qualitätsverlust egal wie groß.
Icon-Hintergründe sind immer transparent.

---

## Composite Icons (Multi-Instance)

Wenn mehrere Instanzen desselben Programms installiert sind:

```
Stamm-Icon     ← immer sichtbar, zeigt was das Programm ist
Instanz-Badge  ← leicht überlappend, zeigt welche Instanz
```

Konfigurierbar in `package.toml`:
```toml
[icon]
primary  = "icon.svg"       # Stamm-Programm
secondary = "badge.svg"     # Instanz
overlap_factor = 0.3        # 0.0 = kein Überlapp, 1.0 = vollständig
```

---

## Traits & Types (fs-render — G1.1 ✅)

| Typ | Art | Zweck |
|-----|-----|-------|
| `Corner` | enum | TopLeft \| TopRight \| BottomLeft \| BottomRight |
| `Side` | enum | Left \| Right \| Top \| Bottom |
| `IndicatorStyle` | enum | QuarterCircle \| HalfCircle |
| `Distribution` | enum | Centered \| SpreadToEdges |
| `IconRef` | struct | Namespaced Icon-Key (`"fs:apps/browser"`) |
| `CompositeIcon` | struct | Primary + optionales Instanz-Icon + overlap_factor |
| `MenuItemDescriptor` | struct | ID, Icon, label_key (FTL), action, sub_items (beliebig tief) |
| `CornerMenuDescriptor` | trait | Beschreibt ein Corner Menu (corner + items + indicator_style) |
| `SideMenuDescriptor` | trait | Beschreibt ein Side Menu (side + items + indicator_style + distribution) |
| `HoverMagnification` | trait | MacOS Dock-Stil: base_size, max_size, spread; `size_at_distance()` berechnet |
| `ProgramView` | enum | Start \| Info \| Manual \| SettingsConfig \| SettingsContainer \| Binding |
| `ProgramViewProvider` | trait | `available_views()` + `supports(view)` |

Alle Traits sind **engine-agnostisch**. Implementierungen:
- `fs-gui-engine-iced` (G1.2 — Standard)
- `fs-gui-engine-tui` (G2)
- `fs-gui-engine-bevy` (G2)

---

## Default-Layout Desktop

| Position | Inhalt |
|----------|--------|
| Oben links | Task-Menü (installierte + laufende Programme) |
| Unten links | Settings (Desktop-Konfiguration) |
| Oben rechts | Help (General + Focus Help) |
| Unten rechts | AI (nur wenn "ai.chat" Capability vorhanden) |

---

## Responsiveness (Mobile / Smartphone)

- Alle Menüs haben einen Scroll-Fallback wenn Items nicht reinpassen
- Touch Gestures (G1.9): Swipe von Ecke = Corner Menu, Swipe von Kante = Side Menu
- Icon-Größe konfigurierbar (Settings > Ansicht): kleinere Icons = mehr Platz auf kleinen Screens
