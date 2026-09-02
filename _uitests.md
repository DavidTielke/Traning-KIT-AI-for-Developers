# UI-Test-Konventionen

Regeln, wie automatisierte Oberflächentests (End-to-End über den Browser) in diesem Repository
geschrieben, strukturiert und ausgeführt werden.

> **Geltungsbereich:** alle Tests, die die laufende Anwendung durch den Browser bedienen.
> Unit- und Komponententests (Vitest, xUnit) fallen **nicht** unter diese Datei.
>
> **Regel:** Kein anderes Browser-Test-Werkzeug, keine andere Testsprache, keine Tests gegen die
> Entwicklungs- oder Produktivdatenbank — und **kein Test ohne Akzeptanzkriterium**.
> Bei Konflikt zwischen dieser Richtlinie und allgemeinen Test-Instinkten gewinnt die Richtlinie.

---

## 1. Die sechs Grundregeln

| # | Regel | Bedeutung |
|---|---|---|
| 1 | **Nur Akzeptanzkriterien** | UI-Tests prüfen **ausschließlich** Akzeptanzkriterien aus `specs/`. Jeder Test ist auf konkrete Kriterien zurückführbar; alles andere wird nicht getestet. |
| 2 | **Gruppen statt Einzeltests** | Die Kriterien einer Story werden zu fachlichen Gruppen zusammengefasst; pro Gruppe entsteht genau ein Test. |
| 3 | **Playwright** | Einziges Werkzeug für UI-Tests. Kein Cypress, Selenium, Puppeteer, TestCafe. |
| 4 | **TypeScript** | Alle Tests, Fixtures, Page Objects und Seed-Daten in `.ts`, `strict` aktiviert. Kein JavaScript, kein `any`. |
| 5 | **Unabhängigkeit** | Jeder Test läuft allein und in beliebiger Reihenfolge grün. Keine Reihenfolgeabhängigkeit, kein geteilter Zustand. |
| 6 | **Eigene Testdatenbank, Seed pro Test** | UI-Tests laufen ausschließlich gegen eine dedizierte Test-Datenbank; vor **jedem** Test wird sie geleert und deterministisch neu befüllt. |

Diese sechs Punkte sind nicht verhandelbar. Ein Test, der einen davon verletzt, wird nicht übernommen.

---

## 2. Die Spec ist die einzige Testquelle (verbindlich)

**Motivation:** Die Akzeptanzkriterien sind laut [`_requirements.md`](_requirements.md) atomar und
testbar formuliert — sie sind damit bereits die Testspezifikation. Wird daneben „nach Gefühl"
getestet, entsteht eine zweite, ungepflegte Anforderungsquelle, die bei jeder Spec-Änderung driftet.

- **Jeder UI-Test bildet ein oder mehrere Akzeptanzkriterien ab.** Es gibt keinen Test ohne
  zugeordnetes Kriterium — auch nicht „zur Sicherheit", nicht für Randfälle, die niemand
  spezifiziert hat, und nicht für technische Eigenheiten der Implementierung.
- **Jedes über die UI prüfbare Kriterium einer Story ist in genau einer Testgruppe abgedeckt** —
  keine Lücke, keine Doppelung.
- **Fehlt für ein gewünschtes Verhalten das Kriterium, wird nicht getestet, sondern spezifiziert.**
  Das Kriterium wird zuerst nach [`_requirements.md`](_requirements.md) in die Story aufgenommen
  (State der Story auf `Modified`), danach entsteht der Test. Nie umgekehrt.
- **Ändert sich ein Kriterium, ändert sich der Test** — in derselben Session. Ein Test, der ein
  überholtes Kriterium prüft, wird gelöscht, nicht auskommentiert.
- **Nicht über die UI prüfbare Kriterien** (reines Backend-Verhalten ohne sichtbare Wirkung,
  Persistenz-, Event- oder Eindeutigkeitszusagen) werden nicht in UI-Tests nachgebaut. Sie werden
  im Deckungsnachweis der Story ausdrücklich als „nicht über die UI prüfbar" mit Begründung geführt
  und gehören in Backend-Tests. Hat ein Backend-Kriterium eine sichtbare Wirkung (Fehlermeldung,
  Zeile verschwindet), wird **nur diese Wirkung** geprüft.

---

## 3. Kriterien gruppieren (verbindlich)

Vor dem ersten Test einer Story werden deren Akzeptanzkriterien gelesen und zu **Testgruppen**
zusammengefasst. Erst danach wird Code geschrieben.

### Was eine Gruppe zusammenhält

Kriterien gehören in dieselbe Gruppe, wenn sie **mindestens eines** davon teilen:

- **Denselben fachlichen Ablauf** — die Kriterien beschreiben aufeinanderfolgende Schritte einer
  Handlung (Dialog öffnen → Felder füllen → speichern → Zeile erscheint → Meldung sichtbar).
- **Dieselbe Vorbedingung** — sie werden am selben Zustand der Oberfläche geprüft und würden sonst
  denselben Aufbau mehrfach wiederholen.
- **Dieselbe Oberflächen-Sektion** — sie beschreiben ein und dasselbe Element (eine Tabellenspalte,
  ein Formularfeld, eine Statistik-Kachel).
- **Dieselbe Kriterien-Untergliederung der Spec** — nutzt die Story bereits `### `-Abschnitte
  (siehe [`_requirements.md`](_requirements.md)), sind diese die Vorgabe für den Zuschnitt.

### Was Gruppen trennt

- **Widersprüchliche Vorbedingungen** — Gültig- und Fehlerfall desselben Formulars sind getrennte
  Gruppen, weil sie unterschiedliche Ausgangsdaten und Abläufe brauchen.
- **Storygrenzen** — eine Gruppe enthält **niemals** Kriterien aus zwei Stories. Fällt das schwer,
  ist der Story-Schnitt zu prüfen, nicht die Gruppe.
- **Größe** — Richtwert 2 bis 6 Kriterien pro Gruppe. Wird eine Gruppe größer, zerfällt sie meist in
  mehrere Abläufe; ein einzelnes Kriterium bleibt allein, wenn es fachlich für sich steht
  (z. B. der Leerzustand einer Liste).

### Ergebnis

- Pro Gruppe entsteht **genau ein** `test(...)`. Der Titel benennt den fachlichen Ablauf der Gruppe,
  nicht die technischen Schritte.
- Die Kriterien der Gruppe werden **im Test vollständig geprüft** — jedes mit einer eigenen
  Assertion. Gruppieren fasst den Aufbau zusammen, nicht die Prüfungen.
- Die Reihenfolge der Assertions folgt der Reihenfolge der Kriterien in der Spec.

---

## 4. Deckungsnachweis (verbindlich)

Jedes Spec-File beginnt mit einer Zuordnung von Kriterien zu Gruppen, jeder Test listet die von ihm
abgedeckten Kriterien im Wortlaut der Spec. Damit ist ohne Werkzeug prüfbar, ob eine Story
vollständig getestet ist.

```ts
// specs/E1-F2-S1-person-anlegen.spec.ts
//
// Story: specs/E1 Personenverwaltung/F2 Personenpflege/S1 Person anlegen.md
//
// Gruppe 1 „Person über den Dialog anlegen"      → AK 1, 2, 3, 6
// Gruppe 2 „Pflichtfelder werden abgewiesen"     → AK 4, 5
// Gruppe 3 „Dialog abbrechen"                    → AK 7
// Nicht über die UI prüfbar: AK 8 (Domain Event `PersonCreated`) → Backend-Test

import { expect, test } from '../fixtures/test'

test.describe('S1 Person anlegen', () => {
  /**
   * Deckt ab:
   * - AK 1: Die Schaltfläche „Anlegen" öffnet einen modalen Dialog.
   * - AK 2: Das Feld `Name` ist ein Pflichtfeld.
   * - AK 3: Nach dem Speichern erscheint die Person in der Liste.
   * - AK 6: Der Dialog schließt sich nach erfolgreichem Speichern.
   */
  test('Person über den Dialog anlegen', async ({ page }) => { … })
})
```

- **AK-Nummern** sind die Position des Kriteriums in der Liste der Story (1-basiert, in Reihenfolge
  der Datei). Verschiebt sich die Liste, wird der Nachweis mitgezogen.
- `test.describe(...)` trägt Nummer und Titel der Story, exakt wie in `specs/`.
- Der Kommentarblock zitiert die Kriterien **wörtlich**, nicht paraphrasiert. Weicht der Wortlaut ab,
  ist entweder die Spec oder der Test veraltet.
- Eine Story gilt erst als getestet, wenn jedes Kriterium entweder in einer Gruppe steht oder
  begründet als „nicht über die UI prüfbar" geführt wird. Sie darf erst dann auf
  `State: Implemented` gesetzt werden.

---

## 5. Ablage und Struktur

```
tests/
└── ui/
    ├── package.json                 ← eigenes Node-Projekt, nur Playwright + TypeScript
    ├── tsconfig.json                ← strict: true
    ├── playwright.config.ts
    ├── fixtures/
    │   ├── test.ts                  ← erweiterter `test`-Export inkl. DB-Reset
    │   └── seed.ts                  ← getypte Seed-Datensätze
    ├── pages/
    │   └── PersonenListPage.ts      ← Page Objects (eins pro Seite/Dialog)
    └── specs/
        └── E1-F1-S1-personenliste-anzeigen.spec.ts
```

- Das UI-Test-Projekt liegt **getrennt** vom Frontend-Projekt (`code/frontend`) und hat eigene
  Abhängigkeiten. Playwright gehört nicht in die `package.json` der Anwendung.
- **Ein Spec-File pro Story**, benannt nach der Nummerierung aus `specs/`
  (siehe [`_requirements.md`](_requirements.md)): `E{n}-F{n}-S{n}-{story-name-kebab}.spec.ts`.
  Ein File enthält alle Testgruppen dieser Story und nur diese.
- Page Objects kapseln Locators und Interaktionen einer Seite. **Keine Assertions im Page Object** —
  die gehören in den Test, weil dort die Kriterien stehen.
- Seed-Daten und Fixtures werden geteilt, Testzustand niemals.

---

## 6. Testdatenbank (verbindlich)

**Motivation:** UI-Tests löschen und verändern Daten. Sie dürfen das niemals in der Datenbank tun,
mit der entwickelt oder produktiv gearbeitet wird.

- Die Test-Umgebung wird über eine **eigene Compose-Datei** `docker-compose.tests.yml` gestartet,
  mit eigenem Volume und eigenen Ports — der Dev-Stack läuft parallel weiter.
- Die Verbindungszeichenfolge zeigt auf eine **separate SQLite-Datei** mit dem Suffix `.tests.db`
  (z. B. `Data Source=/data/personen.tests.db`).
- Die Test-Endpunkte des Backends werden **ausschließlich** über die Umgebungsvariable
  `EnableTestEndpoints=true` aktiviert. Diese Variable steht in keiner anderen Compose-Datei
  und in keinem `appsettings.json`.
- **Sicherung im Backend:** Der Reset-Endpunkt verweigert die Ausführung (HTTP 400), wenn die
  aktive Verbindungszeichenfolge nicht auf eine Datei mit dem Suffix `.tests.db` zeigt.
  Ein falsch konfigurierter Testlauf darf keine echten Daten löschen können.
- Ports der Test-Umgebung liegen neben denen des Dev-Stacks (z. B. Frontend `8082`, Backend `5077`),
  damit ein laufender `docker compose up` nicht kollidiert.

---

## 7. Seeding pro Test (verbindlich)

- Das Backend stellt einen test-only Endpunkt `POST /api/testdata/reset` bereit, der in **einer
  Transaktion** alle Tabellen leert, die Autoincrement-Zähler zurücksetzt und den Basisdatensatz
  einfügt. Danach ist der Zustand für jeden Test bitgleich — inklusive der IDs.
- Der Aufruf erfolgt zentral in einer Fixture, nicht per Copy-Paste in jedem Spec:

```ts
// fixtures/test.ts
import { test as base, expect } from '@playwright/test'

export const test = base.extend<{ seeded: void }>({
  seeded: [async ({ request }, use) => {
    const response = await request.post('/api/testdata/reset')
    expect(response.ok(), 'Seed der Testdatenbank fehlgeschlagen').toBeTruthy()
    await use()
  }, { auto: true }],
})

export { expect }
```

- Specs importieren `test` und `expect` **immer** aus `fixtures/test.ts`, nie direkt aus
  `@playwright/test`. Damit ist der Reset nicht vergessbar.
- **Der Basisdatensatz ist klein und stabil**: gerade so viele Zeilen, dass die Kriterien zu
  Sortierung, Suche und Darstellung prüfbar sind. Er wird nicht für eine einzelne Gruppe erweitert.
- Braucht eine Gruppe besondere Daten, legt der Test sie **selbst** an — über die UI, wenn das
  Anlegen Teil der geprüften Kriterien ist, sonst über die API (`request.post('/api/personen', …)`).
- Aufräumen am Testende ist überflüssig und daher unerwünscht: der nächste Test seedet ohnehin neu.

---

## 8. Unabhängigkeit (verbindlich)

- **Kein geteilter Zustand:** keine Variablen auf Modulebene, die zwischen Tests beschrieben werden,
  keine in Gruppe A angelegten und in Gruppe B verwendeten Datensätze.
- **Kein `test.describe.serial`.** Wer eine Reihenfolge zwischen Gruppen braucht, hat falsch
  gruppiert: ein zusammenhängender Ablauf gehört in **eine** Gruppe und damit in einen Test.
- **Keine festen IDs aus vorherigen Tests.** Zulässig sind nur IDs aus dem Basisdatensatz.
- **Keine Abhängigkeit vom Browserzustand.** Jeder Test startet mit frischem Kontext
  (Playwright-Standard); `storageState` wird nicht wiederverwendet.
- **Prüfung:** Jeder Test muss einzeln grün sein.
  ```
  npx playwright test specs/E1-F2-S1-person-anlegen.spec.ts -g "Pflichtfelder"
  ```
  Wer eine neue Gruppe schreibt, führt sie einmal isoliert und einmal im vollen Lauf aus.
- **Ausführung:** Da Frontend und Backend der Test-Umgebung eine gemeinsame Datenbank nutzen, läuft
  die Suite **seriell** (`workers: 1`, `fullyParallel: false`). Parallelität ist nur zulässig, wenn
  pro Worker ein vollständiger eigener Stack (Frontend + Backend + Datenbank) gestartet wird —
  das ist eine bewusste Erweiterung, kein Standard.

---

## 9. Playwright-Konfiguration

```ts
// playwright.config.ts
import { defineConfig, devices } from '@playwright/test'

export default defineConfig({
  testDir: './specs',
  fullyParallel: false,
  workers: 1,
  retries: 0,
  forbidOnly: !!process.env.CI,
  reporter: [['list'], ['html', { open: 'never' }]],
  use: {
    baseURL: process.env.UI_TEST_BASE_URL ?? 'http://localhost:8082',
    trace: 'retain-on-failure',
    screenshot: 'only-on-failure',
    video: 'off',
    locale: 'de-DE',
    timezoneId: 'Europe/Berlin',
  },
  projects: [{ name: 'chromium', use: { ...devices['Desktop Chrome'] } }],
})
```

- **`retries: 0`** ist Absicht: ein Test, der erst im zweiten Versuch grün wird, ist kaputt und soll
  sichtbar kaputt sein. Wiederholungen verstecken genau die Abhängigkeiten, die Abschnitt 8 verbietet.
- **Chromium ist die Referenz.** Weitere Browser nur, wenn ein konkreter Bug das erzwingt.
- **`locale`/`timezoneId` sind fixiert**, damit Datums- und Zahlenformate der deutschen Oberfläche
  (siehe [`_uiux.md`](_uiux.md), Abschnitt 2) deterministisch geprüft werden können.

---

## 10. Locators und Assertions

- **Locator-Reihenfolge:** `getByRole` → `getByLabel` → `getByText` → `getByTestId`.
  Die Oberfläche ist deutsch und ihre Beschriftungen sind laut UI/UX-Richtlinie stabil — deshalb ist
  die rollenbasierte Abfrage der Normalfall.
  ```ts
  page.getByRole('button', { name: 'Anlegen' })
  page.getByRole('row', { name: 'Anna Berger' })
  ```
- **`data-testid`** nur dort, wo es keinen stabilen zugänglichen Namen gibt (Icon-Buttons in
  Tabellenzeilen, Drag-Griffe). Format: `data-testid="person-row-delete"` (kebab-case, fachlich).
- **Verboten:** CSS-Selektoren auf `dt-*`-Klassen, `nth-child`, XPath, Selektoren auf
  DOM-Struktur. Die Klassen aus `dt-app.css` sind Gestaltung, kein Testvertrag.
- **Eine Assertion je Kriterium.** Kriterien sind laut [`_requirements.md`](_requirements.md) atomar;
  eine Sammel-Assertion über mehrere Kriterien macht im Fehlerfall unklar, welches gebrochen ist.
- **Nur web-first Assertions** (`await expect(locator).toBeVisible()`), niemals
  `expect(await locator.isVisible())` — nur die erste Form wartet automatisch.
- **Keine festen Wartezeiten.** `page.waitForTimeout()` ist verboten; gewartet wird auf Zustand
  (`toBeVisible`, `toHaveCount`, `toHaveText`, `waitForResponse`).
- Zahlen- und Statuswerte werden im Anzeigeformat der Oberfläche geprüft (`1.490 €`, `#001`),
  nicht im Rohformat — sofern das Kriterium das Format vorgibt.

---

## 11. Was nicht getestet wird

Alles, was kein Akzeptanzkriterium ist. Insbesondere:

- **Gestaltung** — Farben, Abstände, Schriftgrößen, Radien, Schatten. Das regelt
  [`_uiux.md`](_uiux.md) und wird nicht durch Tests abgesichert.
- **Animationen und Übergänge**, solange kein Kriterium ein sichtbares Ergebnis fordert.
- **Implementierungsdetails** — Komponentenstruktur, State-Verwaltung, konkrete Requests, es sei
  denn, ein Kriterium beschreibt sie ausdrücklich (z. B. ein URL-Format).
- **Fachlogik im Detail** — sie gehört in Backend-Tests; über die UI wird nur die sichtbare Wirkung
  geprüft (siehe Abschnitt 2).
- **Snapshot-/Screenshot-Vergleiche** als Regressionsnetz — sie brechen bei jeder Layoutänderung und
  prüfen kein Kriterium. Screenshots dienen ausschließlich der Fehlersuche (Abschnitt 9).

---

## 12. Ausführung

```
docker compose -f docker-compose.tests.yml up -d --build   # Test-Stack starten
cd tests/ui && npm ci && npx playwright test                # Suite ausführen
docker compose -f docker-compose.tests.yml down -v          # Stack inkl. Test-Volume entfernen
```

- Der Test-Stack wird vor dem Lauf gestartet und danach mit `-v` entfernt — die Testdatenbank ist
  Wegwerfware.
- Nach Änderungen an der Oberfläche werden die betroffenen Specs ausgeführt, **bevor** committet wird.
- Fehlschläge werden behoben, nicht übersprungen: `test.skip` und `test.fixme` bleiben nur mit
  Kommentar und Verweis auf die zugehörige Story oder Idee im Code stehen.

---

## 13. Anti-Patterns (nicht tun)

- Verhalten testen, für das es kein Akzeptanzkriterium gibt — statt das Kriterium zuerst in die
  Spec aufzunehmen.
- Ein Kriterium in mehreren Gruppen doppelt prüfen oder ein Kriterium ungeprüft und ohne Begründung
  im Deckungsnachweis lassen.
- Kriterien aus zwei Stories in eine Gruppe mischen oder ein Spec-File über mehrere Stories spannen.
- Alle Kriterien einer Story in einen einzigen Riesentest packen — oder umgekehrt jeden Schritt
  eines Ablaufs zu einem eigenen Test aufblähen.
- Kriterien im Testkommentar paraphrasieren statt zu zitieren.
- Ein anderes Browser-Test-Framework als Playwright einführen.
- Tests in JavaScript schreiben oder `any` verwenden, um Typfehler zu umgehen.
- Gegen `personen.db` des Dev- oder Produktiv-Stacks testen.
- Den Seed nur einmal pro Datei (`beforeAll`) statt pro Test ausführen.
- `test.describe.serial`, Tests, die aufeinander aufbauen, oder Daten aus einem anderen Test verwenden.
- `page.waitForTimeout()`, `expect(await …isVisible())`, Selektoren auf `dt-*`-Klassen oder `nth-child`.
- Retries aktivieren, um Flakiness zu kaschieren.
- Test-Endpunkte ohne `EnableTestEndpoints`-Schalter oder ohne `.tests.db`-Sicherung ausliefern.
