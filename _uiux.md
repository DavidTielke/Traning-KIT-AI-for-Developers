# UI/UX-Richtlinie — David Tielke Business-App

> Vorlage für Claude-Code-Harness. Editorial, typografisch geführtes B2B-Verwaltungs-Interface
> für **David Tielke — Development Consulting** (Winterberg, DE). Stack-neutral: vanilla HTML + eine
> globale CSS-Datei (`dt-app.css`). Klassenbasiert, token-getrieben.
>
> **Regel:** Erfinde keine Farben, Typo, Abstände oder Komponenten außerhalb dieser Datei.
> Bei Konflikt zwischen dieser Richtlinie und allgemeinen Web-Instinkten gewinnt die Richtlinie.

Begleitdatei: **`dt-app.css`** (Tokens + alle Komponentenklassen). Diese Datei laden, dann Markup mit den
`dt-*`-Klassen aufbauen.

---

## 1. Was gebaut wird

Verwaltungsoberflächen: Sidebar-Navigation + Inhaltsbereich, Dashboard mit Statistik-Kacheln,
dichte sortier- und durchsuchbare Datentabellen, CRUD über Modal-Dialoge, Löschbestätigung,
Formulare mit Inline-Validierung. Referenzumsetzung: Verwaltung von Personen, Tierarten, Haustieren.

Archetyp einer Seite:

```
dt-app
├── dt-sidebar   (Marke oben · Navigation · User-Chip unten)
└── dt-main
    ├── dt-topbar   (Kicker + Titel · Suche · Primär-Aktion)
    └── dt-content  (Dashboard ODER Tabellen-Karte)
+ dt-backdrop > dt-modal   (nur wenn geöffnet)
```

---

## 2. Voice & Copy (verbindlich)

- **Deutsch, Sie-Form.** Niemals „Du". B2B-Register durchhalten — auch in Micro-Copy.
- **Editorial, sachlich.** Keine Ausrufezeichen im Fließtext, keine Superlative ohne Beleg,
  keine rhetorischen Fragen. Aktiv vor nominal.
- **Überschriften:** kurz, deklarativ, enden mit orangem Punkt → `Titel<span class="dt-dot">.</span>`.
  Mehrteilig optional mit grüner Kommasetzung (`.dt-acc`).
- **Kicker:** ein bis zwei Wörter, UPPERCASE, orange, über jedem Titel. Beschreibend, nicht werblich
  (`Übersicht`, `Stammdaten`, `Katalog`, `Bestand`) — nie „Was wir bieten".
- **Buttons: Verb zuerst** (`Anlegen`, `Speichern`, `Platz reservieren`, `Zugang aktivieren`).
  Nie „Mehr erfahren", „Klick hier", „Weiter →" (Ausnahme: mehrstufige Modals).
- **Daten-Micro-Copy:** Beträge `1.490 €` (Komma-Dezimal, Mono), Daten `10.–12. Juni 2026` (En-Dash).
  Statuswerte als Nomen: `Verfügbar`, `Wenige Plätze`, `Ausgebucht`.
- **Keine Emojis in der UI. Keine Unicode-Glyphen als Icons.**

---

## 3. Farben (semantisch, nicht dekorativ)

| Token | Wert | Rolle |
|---|---|---|
| `--dt-ink` | `#2b2b2b` | Text + Überschriften (95 % der Pixel) |
| `--dt-bg` | `#ffffff` | Karten, Inhaltsflächen |
| `--dt-panel` | `#f2f2f2` | App-Hintergrund, Tabellenkopf, Modal-Fuß, Inputs auf Karten |
| `--dt-muted` | `#7a7a7a` | Sekundärtext, Labels |
| `--dt-line` | `#e5e5e5` | 1px-Rahmen, Trenner |
| `--dt-orange` | `#f38a2c` | **Primär-Aktion:** CTA, aktiver Zustand, Headline-Punkt, Kicker, Fokus |
| `--dt-orange-dark` | `#e07620` | Hover primärer Button |
| `--dt-orange-soft` | `#fff3e5` | nur Chip-/Badge-BG, aktive Nav, Zeilen-Hover |
| `--dt-green` | `#7cb342` | **Struktur/Erfolg:** Kartenrahmen oben/unten, Nav-Bottom, Häkchen, Balken |
| `--dt-green-soft` | `#ecf4dd` | nur Badge-BG |
| `--dt-error` | `#c0392b` | Validierung, destruktive Aktion |

**Verboten:** Gradienten (überall), Glow, Glas/Blur (einzige Ausnahme: Modal-Backdrop),
weiche Schatten außer den drei definierten. Soft-Tints nie als Sektionshintergrund.

---

## 4. Typografie

- **Plus Jakarta Sans** für alles UI. **JetBrains Mono ausschließlich für Datenmarker:**
  IDs (`#001`), Zählwerte, Alter/Jahre, Preise, Counter, Code. Nie Mono für Fließtext, Titel, Labels.
- Enges Tracking auf großer Type (Topbar-Titel `-0.8px`, Stat-Zahlen `-1px`). Body `0`.
- Pro Bereich **genau eine** H2 mit einem Kicker. Keine konkurrierenden Titel.
- Größen: Topbar-Titel 26px/700 · Karten-H3 20px/700 · Stat-Zahl 42px Mono/500 ·
  Body 14–15px · Label/Kicker 11px UPPERCASE.

---

## 5. Layout, Abstände, Form

- **Sidebar** 266px, weiß, `border-right: 1px var(--dt-line)`, sticky volle Höhe.
  Marke oben mit `border-bottom: 2px var(--dt-green)`, User-Chip unten mit `border-top: 1px`.
- **Topbar** sticky, weiß, `border-bottom: 2px var(--dt-green)`, Padding `20px 34px`.
- **Content** Padding `34px`, scrollt eigenständig, Hintergrund `--dt-panel`.
- **Spacing-Skala** (Vielfache von 4): `4 6 8 10 12 14 16 20 24 28 32 48 56 64 72 80 96 112`.
  Karten-Innenpolster 22–28px, Formzeilen 14–16px, Inline-Gaps 4–12px.
- **Radien:** 4 Badges/Section-Buttons · 6 Inputs/Modal-Buttons · 8 Standardkarten ·
  10 Stat-/Summary-Karten & Modal-Ecke · 12 Modal-Container · 999 Pills.
- **Schatten — nur drei:** keiner (Default) · `0 1px 2px rgba(0,0,0,.02)` (leichte Karte) ·
  `0 30px 100px rgba(0,0,0,.4)` (Modal).
- **Editorial-Whitespace:** großzügig, nie „vollgepackt".

---

## 6. Die Karten-Signatur

Jede Karte trägt dieselbe Rahmenbehandlung — das markanteste Markenelement:

```html
<div class="dt-card"><div class="dt-card__pad">…</div></div>
```

= grüner 2px-Rahmen oben **und** unten, 1px graue Seiten, Radius 8 (bzw. `dt-card--lg`: 3px/Radius 10).
**Keine alternativen Kartenformen erfinden.**

---

## 7. Komponenten (Klasse → Zweck)

**Navigation** — `dt-nav__item` (aktiv: `.is-active` → oranger Soft-BG + oranger Text),
optional `dt-nav__count` (Mono-Pill mit Anzahl, aktiv orange gefüllt).

**Dashboard-Kachel** — `dt-card` + `dt-stat`: `dt-stat__label` (UPPERCASE muted) · `dt-stat__value`
(große Mono-Zahl) · `dt-stat__sub`. Verteilungsbalken: `dt-bar > dt-bar__fill` (grün).

**Datentabelle** — `dt-tablecard > dt-tablecard__head (Ergebniszähler) + dt-table`.
Kopf `th` auf `--dt-panel`, UPPERCASE muted, klickbar via `.is-sortable` + `dt-sort-ind` (↑/↓ orange).
Zeilen-Hover orange-soft. Zahlen-Spalten `dt-table__num` + `dt-cell-mono`.
Zell-Treatments: `dt-cell-strong`, `dt-cell-mono`, `dt-cell-muted`.

**Badges/Status** — `dt-badge--green|orange|neutral` (Soft-Tint-BG, Pill). Status `dt-status--green|orange|muted`.

**Zeilenaktionen** — `dt-iconbtn` (Hover orange), `dt-iconbtn--danger` (Hover rot). Nur Icon, 32px, Radius 6.

**Reihenfolge (manuell)** — `dt-reorder`: `dt-reorder__grip` (Drag-Griff, `cursor: grab`) +
`dt-reorder__arrows` (vertikal gestapelte `dt-reorder__arrow`, an den Enden `:disabled`).
Zeile beim Ziehen `.dt-row--dragging`, nach Verschieben kurz `.dt-row--flash`.

**Buttons** — `dt-btn dt-btn--primary` (orange CTA) · `--secondary` (Linie) · `--danger` ·
`--block` · `--modal`. Hover = Farbwechsel, nie Skalierung.

**Formular** — `dt-form` (12-Spalten-Grid) mit `dt-field--12|6|4`; `dt-field__label` (UPPERCASE) +
`dt-input`/`dt-select` (Fokus → oranger Rahmen). Fehler: `.is-error` am Feld + `dt-field__error`
(eine Zeile, rot, 12px). Checkbox: `dt-check` / `.is-checked`.

**Modal** — `dt-backdrop` (einziger Blur-Einsatz) > `dt-modal` mit `__head` / `__body` / `__foot`
(Fuß auf Panel-BG). Schließen: `dt-modal__close`. Bestätigungsdialog: `dt-modal--sm` +
`dt-danger-tile` + `dt-btn--danger`. Esc und Backdrop-Klick schließen.

---

## 8. Zustände & Animation

- **Nur Reaktion auf Nutzeraktion.** Keine Scroll-Trigger, keine Count-ups, keine Reveals, kein Bounce.
- **Farbübergänge 160ms** (Links, Nav, Buttons, Inputs, Icon-Buttons).
- **Transform/Modal 200–240ms** (`dt-fade`, `dt-pop`).
- **Fokus Inputs:** Rahmen `--dt-line` → `--dt-orange`, kein Glow.
- **Primär-Button-Hover:** `--dt-orange` → `--dt-orange-dark`, keine Größenänderung.
- **Manuelles Verschieben:** kurzer `dt-row-flash` (oranger Puls + Akzentbalken).

---

## 9. Ikonografie

- Inline-SVG, Stil **Font Awesome 6 Sharp Thin**: `stroke="currentColor"`, `stroke-width` 2–2.5,
  `linecap/linejoin="round"`, `fill="none"`. Farbe erbt vom Parent-`color`.
- Größen: 16px in Listen/Buttons, 24–28px in Karten.
- **Keine** mehrfarbigen Icons, keine Icon-Hintergründe/Kreise als Deko, keine Emojis,
  keine Unicode-Glyphen als Icons, keine erfundenen Konzept-Illustrationen (fehlt ein Icon →
  Platzhalter 24×24 mit Namen, dann Asset anfordern).

---

## 10. Diagramme (verbindlich)

- **Bibliothek: Recharts.** Keine anderen Chart-Bibliotheken, keine handgebauten Canvas-Charts.
- **Erlaubte Typen:** Donut (Anteile am Bestand) und Säulen/Balken (Verteilungen, Vergleiche).
  Linien-/Flächendiagramme nur bei echtem Zeitbezug; keine 3D-, Radar- oder Gauge-Charts.
- **Kategoriale Farbreihe** (feste Reihenfolge, ab der 7. Kategorie wiederholend):
  `#f38a2c` → `#7cb342` → `#e07620` → `#5a8a2e` → `#f9c08a` → `#a8cf7a`.
  Grau `#b0b0b0` ausschließlich für „Sonstige"/„Ohne Angabe". Keine Farben außerhalb dieser Reihe.
- **Eine Datenreihe = eine Farbe:** Grün als Standard, Orange nur zur Hervorhebung.
- **Typo:** Achsen und Legenden 11–12px Sans (muted); Zahlenwerte in Tooltips/Labels Mono.
- **Tooltip:** weiße Karte, 1px `--dt-line`, Radius 6, Schatten höchstens `--dt-shadow-card`.
- **Keine Lade-Animationen** (`isAnimationActive` aus) — Bewegung nur als Reaktion auf Hover/Klick.
- Donut mit Innenloch, ohne Beschriftung in den Segmenten; Namen und Anzahlen gehören in Legende und Tooltip.
- Diagramme leben in Karten (`dt-card` / `dt-card--lg`) mit eigener H3 und folgen den übrigen Verboten
  (keine Gradienten, kein Glow, keine Deko-Schatten, keine Scroll-Trigger).

## 11. Anti-Patterns (nicht tun)

- Gradienten, Glow, Glas/Blur (außer Modal-Backdrop), Deko-Schatten.
- Mono für Fließtext/Titel/Labels; Soft-Tints als Flächenhintergrund.
- Zwei gleiche Sektions-Hintergründe hintereinander (weiß/panel alternieren).
- „Du"-Ansprache, Marketing-Ton, Ausrufezeichen, generische Button-Labels.
- Alternative Kartenformen statt `dt-card` / `dt-card--lg`.
- Hover-Skalierung, Spring-Physik, Scroll-Animationen.
