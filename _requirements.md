# Anforderungs-Konventionen

Regeln, wie Anforderungen (Epics, Features, Stories, Akzeptanzkriterien) in diesem Spec-Ordner strukturiert und formuliert werden.

## Ordner- und Dateistruktur

```
specs/
├── guidelines/
│   ├── _Anforderungen.md     ← dieses Dokument
│   └── _Architektur.md       ← architekturelle Grundregeln
├── E{n} {Epic-Name}/
│   ├── _Datenstruktur.md     ← Entitäten des Epics
│   ├── _Menüstruktur.md      ← Menü-Einordnung
│   ├── _Backend.md           ← Endpunkte & Domain Events des Epics
│   └── F{n} {Feature-Name}/
│       └── S{n} {Story-Name}.md
```

- **E** = Epic, **F** = Feature, **S** = Story, jeweils mit laufender Nummer
- Nummerierung startet pro Ebene bei 1 (S1, S2, … pro Feature; F1, F2, … pro Epic)
- Nummern bleiben stabil: neue Stories werden hinten angehängt, nicht dazwischen eingeschoben

## Story-Format

Jede Story-Datei folgt dieser Struktur:

```markdown
# {Story-Titel}

## Meta
- **State:** Modified | Implemented

## User Story
Als {Rolle} möchte ich {Ziel}, damit {Nutzen}.

## Description
Kurze fachliche Beschreibung: was macht die Seite/Funktion, wie ist sie erreichbar, was sind die wichtigsten Interaktionen. Keine technische Umsetzung.

## Akzeptanzkriterien
- …
- …
```

Epic-/Feature-/Story-Nummerierung ergibt sich aus dem Datei-Pfad (`E{n} {Epic}/F{n} {Feature}/S{n} {Story}.md`) — sie wird **nicht** zusätzlich im Story-Header dupliziert.

Bei großen Stories dürfen Akzeptanzkriterien in thematische Unterabschnitte (`### ...`) gegliedert werden (z. B. „Stammdaten", „Sektion X").

## Meta-Block & State (verbindlich)

**Motivation:** Auf einen Blick sichtbar machen, welche Stories aktuell zum Code passen (`Implemented`) und welche eine Diskrepanz zur Implementierung haben (`Modified` — Spec wurde geändert oder noch nie umgesetzt).

Jede Story trägt direkt nach dem H1-Titel einen Meta-Block mit genau einem Pflichtfeld:

```
## Meta
- **State:** Modified | Implemented
```

### State-Werte

| State | Bedeutung |
|---|---|
| **`Modified`** | Spec wurde neu angelegt oder seit der letzten Implementierung inhaltlich geändert; der Code spiegelt die Spec aktuell **nicht** (mehr) wider. |
| **`Implemented`** | Spec und Code (UI + Backend) sind synchron — alle Akzeptanzkriterien sind umgesetzt. |

### Pflege-Regeln

- **Jede inhaltliche Spec-Änderung** an einer `Implemented`-Story setzt den State zurück auf `Modified` — auch reine Akzeptanzkriterien-Verschärfungen oder Description-Edits.
- **Sobald die Implementierung der Spec entspricht**, wird der State auf `Implemented` gesetzt — in derselben Session, in der der Code-Stand mit der Spec abgeglichen wurde.
- **Reine Code-Änderungen ohne Spec-Änderung** (Bugfixes, Refactorings, Performance) ändern den State **nicht**.
- **Bei Spec + Code in einer Session** koordiniert geändert: direkt auf `Implemented` springen, ohne `Modified` als Zwischenstand zu schreiben.
- **State ist Source of Truth über den Sync-Stand** — er muss bei jeder Spec-Bearbeitung explizit überprüft werden. Drift ist der Hauptfeind; lieber konservativ `Modified` setzen, wenn unklar.

## Querschnitts-Features (verbindlich)

Manche Features wiederholen sich über alle Epics, die fachlich denselben Mechanismus nutzen. Sie sind **kein optionales Extra**, sondern gehören für jedes betroffene Epic zum Mindest-Lieferumfang. Beim Anlegen eines neuen Epics ist diese Liste durchzugehen.

### Email-Logs (jedes Epic mit ausgehenden E-Mails)

Sobald ein Epic E-Mails an Kunden/Empfänger versendet (per n8n-Workflow über `POST /api/email-logs`), **muss** es eine eigene EmailLogs-Frontend-Feature im BackOffice haben — analog zu allen bestehenden Modulen (Referenz: `E6 Offers/F5 EmailLogs`).

- Ein Feature `F{n} EmailLogs` mit mindestens zwei Stories: `S1 Email-Logs Übersicht` (rein lesendes, sortier-/filterbares Grid) und `S2 Email-Log Detail` (Read-only-Detail inkl. aufklappbarem Mail-Body und Quicklinks zu den verknüpften Entitäten).
- Ein Menüeintrag „Email-Logs" in der zugehörigen Menügruppe (`_Menüstruktur.md`).
- **Backend-seitiges Logging allein erfüllt diese Konvention nicht.** Endpoints (`/api/email-logs`) und schreibende Workflows existieren technisch oft schon, damit die Mails überhaupt protokolliert werden — die lesende BackOffice-UI ist trotzdem separat zu spezifizieren und umzusetzen. Genau diese Trennung hat in `E8 Appointments` dazu geführt, dass die Übersicht zunächst fehlte, obwohl Backend und Workflows längst da waren.

## Akzeptanzkriterien: INVEST

Akzeptanzkriterien müssen die INVEST-Prinzipien erfüllen:

- **I**ndependent — So unabhängig wie möglich formuliert; Verweise auf andere Stories nur, wenn inhaltlich zwingend (dann als „siehe S{n}").
- **N**egotiable — Kriterien beschreiben das Verhalten, nicht die Implementierung.
- **V**aluable — Jedes Kriterium liefert einen Mehrwert für den Nutzer oder fachlichen Anwender.
- **E**stimable — Kriterien sind so konkret, dass der Aufwand abschätzbar ist.
- **S**mall — **Atomar**: ein Kriterium beschreibt genau eine prüfbare Aussage. Zusammengesetzte Aussagen werden gesplittet.
- **T**estable — Jedes Kriterium hat ein eindeutiges Pass/Fail.

### Formulierungs-Muster

- ✅ „Das Feld `DisplayName` ist ein Pflichtfeld."
- ✅ „Das Feld `DisplayName` akzeptiert max. 200 Zeichen."
- ✅ „Das Feld `DisplayName` muss über alle Zertifikate eindeutig sein."
- ❌ „DisplayName: Pflichtfeld, max. 200 Zeichen, eindeutig." *(nicht atomar)*
- ❌ „Alle Eingaben werden validiert." *(nicht testbar — was heißt „alle"?)*
- ❌ „Nur valide Eingaben werden zugelassen." *(nicht testbar)*

### Verb-Konventionen

- „ist sichtbar / ist aktiv / ist deaktiviert" — für UI-Zustände
- „öffnet sich ein modaler Dialog" — für Modale
- „wird das Event `X` ausgelöst" — für Domain Events
- „wird persistiert / wird aus der Datenbank entfernt" — für Datenzustände
- „entspricht dem Format `{…}`" — für URL- oder Datei-Formate

### Abgrenzung UI / Backend

Akzeptanzkriterien folgen der Architektur-Regel (siehe `_Architektur.md`): Fachlogik, Validierung und Uniqueness-Checks werden als Backend-Verhalten formuliert, das UI zeigt nur das Ergebnis an.

- ✅ „Das Backend prüft die Eindeutigkeit von `DisplayName`. Bei Konflikt zeigt das UI die Fehlermeldung des Backends an."
- ❌ „Das Formular prüft, ob der `DisplayName` bereits existiert."

Client-seitige Validierung ist als UX-Hilfe erlaubt, wird aber im Akzeptanzkriterium als „client- und serverseitig validiert" formuliert, um klarzustellen, dass das Backend die autoritative Quelle ist.

## Felder und Entitäten

- Feldnamen in Akzeptanzkriterien immer in Backticks und in der Originalschreibweise aus der `_Datenstruktur.md` (z. B. `DisplayName`, `OrganizerEmail`, `FK_TrainerId`)
- Event-Namen ebenfalls in Backticks (z. B. `WorkshopCreatedMessage`, `ParticipantAddedByAdmin`)
- Enum-Werte in Backticks (z. B. `"German"`, `"Admin"`)
