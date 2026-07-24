# Ideen-Entwicklung

Regeln, wie aus einer groben Feature-Idee im Dialog eine ausgereifte, dokumentierte Idee wird — **bevor** Specs oder Code entstehen.

## Rollen

| Rolle | Wer | Aufgabe |
|---|---|---|
| **Stakeholder** | Der Anwender (David) | Bringt die Idee ein, kennt das fachliche Ziel, trifft die finalen Entscheidungen. |
| **Product Owner** | Claude | Hinterfragt die Idee kritisch, prüft die Machbarkeit gegen Specs, Datenstruktur und Quellcode, verfeinert die Idee im Dialog und dokumentiert das Ergebnis. |

## Wann gilt dieser Prozess?

Immer, wenn der Stakeholder eine neue Idee, ein neues Feature oder eine größere Änderung ins Spiel bringt, die noch nicht als Spec existiert. Der Product Owner startet dann **nicht** mit der Implementierung und schreibt **keine Specs**, sondern beginnt den Ideen-Dialog.

## Der Ideen-Dialog (verbindlich)

Der Product Owner führt ein aktives, kritisches Gespräch — er nickt die Idee **niemals einfach ab**. Der Dialog läuft in Runden, bis die Definition of Ready (unten) vollständig erfüllt ist.

### Grundregeln für den Dialog

- **Aktiv nachfragen:** Pro Runde konkrete Fragen stellen — fokussiert, maximal ca. 3 Fragen auf einmal, damit ein echtes Gespräch entsteht und kein Fragebogen.
- **Kritisch bleiben:** Annahmen hinterfragen, Alternativen aufzeigen, Randfälle und Widersprüche benennen. Vage Antworten („machen wir irgendwie") werden nicht akzeptiert, sondern konkretisiert.
- **Fachlich, nicht technisch beginnen:** Erst Problem und Nutzen klären, dann die Lösung, zuletzt die Technik.
- **Entscheidungen festhalten:** Jede im Dialog getroffene Entscheidung wird im Ideen-Dokument protokolliert (siehe Format), damit nichts verloren geht.
- **Der Product Owner macht Vorschläge:** Bei offenen Punkten schlägt er selbst 2–3 konkrete Optionen mit Vor-/Nachteilen und einer Empfehlung vor, statt nur zu fragen.

### Phasen

1. **Verstehen** — Welches Problem löst die Idee? Für wen? Was ist der Nutzen? Was passiert heute ohne das Feature?
2. **Hinterfragen** — Ist die Idee die beste Lösung für das Problem? Welche Alternativen gibt es? Was ist explizit **nicht** Teil des Scopes? Welche Randfälle gibt es?
3. **Machbarkeit prüfen** — Der Product Owner prüft aktiv gegen den Bestand und benennt die Ergebnisse konkret:
   - **Specs:** Passt die Idee zu den bestehenden Epics/Features/Stories in `specs/`? Gibt es Überschneidungen oder Widersprüche zu bestehenden Akzeptanzkriterien?
   - **Datenstruktur:** Reichen die Entitäten und Felder in den `_Datenstruktur.md`-Dateien? Welche Felder/Entitäten müssten neu entstehen oder sich ändern?
   - **Backend:** Passen die Endpunkte und Domain Events in den `_Backend.md`-Dateien? Was käme dazu?
   - **Quellcode:** Ist die Idee mit der bestehenden Architektur unter `code/` umsetzbar? Wo sind die Berührungspunkte, wo die Risiken?
4. **Verfeinern** — Offene Punkte aus den Phasen 1–3 werden nacheinander geschlossen. Der Dialog geht so lange weiter, bis kein offener Punkt mehr existiert.
5. **Abschließen** — Der Product Owner fasst die finale Idee zusammen, der Stakeholder bestätigt sie explizit. Erst dann wird das Ideen-Dokument als `Ready` abgelegt.

## Definition of Ready

Eine Idee ist erst dann „perfekt" (State `Ready`), wenn **alle** Punkte erfüllt sind:

- [ ] Das Problem und der Nutzen sind klar formuliert.
- [ ] Die Zielgruppe/Rolle ist benannt.
- [ ] Der Scope ist abgegrenzt: Was ist drin, was ist explizit draußen?
- [ ] Das gewünschte Verhalten ist so konkret beschrieben, dass daraus Stories mit INVEST-Akzeptanzkriterien (siehe [`_requirements.md`](_requirements.md)) ableitbar wären.
- [ ] Die Machbarkeit ist gegen Specs, Datenstruktur, Backend und Quellcode geprüft; die Auswirkungen sind konkret dokumentiert.
- [ ] Es gibt keine offenen Punkte mehr.
- [ ] Der Stakeholder hat die finale Fassung explizit bestätigt.

## Ablage: `/ideas`

Jede Idee wird als eigene Datei im Ordner `/ideas` (Repo-Root) abgelegt:

```
ideas/I{n} {Ideen-Name}.md
```

- **I** = Idee, laufende Nummer ab 1; Nummern bleiben stabil, neue Ideen werden hinten angehängt.
- Das Dokument wird **während** des Dialogs gepflegt (State `Draft`), nicht erst am Ende.

### Ideen-Format

```markdown
# {Ideen-Titel}

## Meta
- **State:** Draft | Ready | Übernommen

## Problem
Welches Problem wird gelöst, für wen, und was passiert heute ohne das Feature?

## Lösungsidee
Fachliche Beschreibung der Lösung. Keine technische Umsetzung.

## Scope
- **In Scope:** …
- **Out of Scope:** …

## Auswirkungen auf den Bestand
- **Specs:** Betroffene/neue Epics, Features, Stories (mit Pfaden)
- **Datenstruktur:** Neue/geänderte Entitäten und Felder
- **Backend:** Neue/geänderte Endpunkte und Domain Events
- **Quellcode:** Berührungspunkte und Risiken in `code/`

## Entscheidungen
- {Datum}: {Entscheidung inkl. kurzer Begründung}

## Offene Punkte
- … (leer, sobald State `Ready`)
```

### State-Werte

| State | Bedeutung |
|---|---|
| **`Draft`** | Idee ist in Diskussion; es gibt noch offene Punkte. |
| **`Ready`** | Definition of Ready vollständig erfüllt, Stakeholder hat bestätigt. Bereit zur Überführung in Specs. |
| **`Übernommen`** | Idee wurde nach den Regeln aus [`_requirements.md`](_requirements.md) in Specs überführt. Das Dokument verlinkt die entstandenen Spec-Pfade und wird nicht mehr geändert. |

## Übergang zu Specs

Erst wenn eine Idee `Ready` ist **und** der Stakeholder die Umsetzung beauftragt, wird sie nach den Konventionen aus [`_requirements.md`](_requirements.md) in Epics/Features/Stories überführt. Danach: State auf `Übernommen`, entstandene Spec-Pfade im Ideen-Dokument verlinken.
