# Endkunden-Dokumentation

Regeln, wie die Anwenderdokumentation in diesem Repository aufgebaut, formuliert und bebildert wird.

> **Geltungsbereich:** alles unter `docs/user/` — Texte für die Menschen, die die Anwendung bedienen.
> Entwicklerdokumentation (`docs/dev/`), Anforderungen (`specs/`) und Ideen (`ideas/`) fallen **nicht**
> unter diese Datei.
>
> **Regel:** Kein Kapitel ohne Screenshot, keine Seite ohne FAQ, kein Fachjargon. Bei Konflikt zwischen
> dieser Richtlinie und allgemeinen Doku-Instinkten gewinnt die Richtlinie.

---

## 1. Die fünf Grundregeln

| # | Regel | Bedeutung |
|---|---|---|
| 1 | **Null Vorwissen** | Die Leserin hat die Anwendung noch nie gesehen und kennt keinen einzigen Fachbegriff. Alles, was zum Ziel führt, steht in der Anleitung — nichts wird als bekannt vorausgesetzt. |
| 2 | **Eine Datei pro Seite** | Jede Seite bzw. jeder Dialog der Anwendung bekommt genau eine Dokumentationsdatei, jedes Modul einen Überblick. |
| 3 | **Screenshot zu jedem Schritt** | Jeder Arbeitsablauf wird bebildert. Text allein reicht nie. |
| 4 | **FAQ pro Modul** | Jedes Modul schließt mit häufigen Fragen und Fehlermeldungen ab. |
| 5 | **Aus der Anwenderperspektive** | Beschrieben wird, was der Anwender sieht und klickt — niemals Technik, Datenbank, API oder Quellcode. |

Diese fünf Punkte sind nicht verhandelbar. Eine Doku-Seite, die einen davon verletzt, wird nicht übernommen.

---

## 2. Ablage und Struktur

```
docs/user/
├── README.md                          ← Startseite: was ist die Anwendung, Modulübersicht
├── 00-erste-schritte.md               ← Anmelden, Aufbau des Bildschirms, Grundbegriffe
├── E1-personenverwaltung/
│   ├── README.md                      ← Modulüberblick + Inhaltsverzeichnis des Moduls
│   ├── 01-personenliste.md            ← eine Datei pro Seite/Dialog
│   ├── 02-person-anlegen.md
│   ├── 03-person-bearbeiten.md
│   ├── 04-person-loeschen.md
│   ├── faq.md                         ← Pflicht: häufige Fragen des Moduls
│   └── bilder/
│       ├── personenliste-01-uebersicht.png
│       └── personenliste-02-suche.png
└── E2-haustierverwaltung/
    └── …
```

- **Ein Ordner pro Modul.** Ein Modul entspricht einem Epic aus `specs/`
  (siehe [`_requirements.md`](_requirements.md)). Ordnername: `E{n}-{modulname-kebab}`.
- **Eine Datei pro Seite**, nummeriert in der Reihenfolge, in der ein neuer Anwender sie kennenlernt —
  nicht in der Reihenfolge der Stories. Erst ansehen, dann anlegen, dann ändern, dann löschen.
- **Bilder liegen im Modul**, in `bilder/` neben den Texten. Kein globaler Bilderordner. Bilder der
  modulübergreifenden Seiten (`README.md`, `00-erste-schritte.md`) liegen in `docs/user/bilder/`.
- **Dateinamen** durchgängig klein, ohne Umlaute, mit Bindestrich: `person-anlegen.md`,
  nicht `Person anlegen.md`.
- **Kein Modul ohne `faq.md`** und kein Modul ohne `README.md`.

---

## 3. Aufbau einer Seiten-Datei (verbindlich)

Jede Seiten-Datei folgt exakt dieser Kapitelfolge. Kapitel werden nicht umsortiert und nicht
weggelassen; ein Kapitel ohne Inhalt bekommt den Satz „Für diese Seite gibt es hier nichts zu beachten."

```markdown
# {Was der Anwender hier tut}

## Worum geht es hier?
Zwei bis vier Sätze in einfacher Sprache: Wofür ist diese Seite da und wann brauche ich sie?

## Wo finde ich das?
Der Klickweg vom Start der Anwendung bis hierher, als nummerierte Schritte.
Danach ein Screenshot der geöffneten Seite.

## Was Sie auf dem Bildschirm sehen
Screenshot mit nummerierten Markierungen, darunter eine Tabelle:
| Nr. | Element | Bedeutung |

## Schritt für Schritt: {Aufgabe}
Nummerierte Schritte, ein Klick pro Schritt, mit Screenshots.
Pro Aufgabe ein eigener Abschnitt (z. B. „Schritt für Schritt: Eine Person anlegen").

## Gut zu wissen
Kurze Hinweise, Einschränkungen, Grenzwerte — als Aufzählung.

## Wenn etwas nicht klappt
Tabelle: | Das sehen Sie | Das bedeutet es | Das tun Sie |
```

**Modul-`README.md`** ist kürzer: ein Absatz „Was kann dieses Modul?", ein Screenshot der Hauptseite,
eine Liste der Seiten mit je einem Satz, und ein Link auf die `faq.md` des Moduls.

---

## 4. Sprache (verbindlich)

**Motivation:** Die Dokumentation wird von Menschen gelesen, die gerade nicht weiterkommen. Jeder Satz,
den sie zweimal lesen müssen, ist ein Fehler in der Dokumentation.

- **Deutsch, Sie-Form.** Freundlich und sachlich, nie herablassend, nie flapsig. Register wie in der
  Oberfläche (siehe [`_uiux.md`](_uiux.md)).
- **Ein Gedanke pro Satz.** Maximal rund 15 Wörter. Keine Schachtelsätze, keine Klammereinschübe.
- **Ein Klick pro Schritt.** „Klicken Sie oben rechts auf **Anlegen**." — nicht „Legen Sie über die
  Aktionsleiste einen neuen Datensatz an und füllen Sie die Pflichtfelder."
- **Beschriftungen wörtlich und **fett**** zitieren, exakt wie in der Anwendung: die Schaltfläche
  **Speichern**, das Feld **Nachname**. Erfundene oder sinngemäße Beschriftungen sind ein Fehler.
- **Ortsangaben immer mitgeben:** „oben rechts", „in der linken Leiste", „unter der Tabelle".
- **Fachbegriffe verbieten oder erklären.** Verboten: Datensatz, Entität, Endpunkt, API, Validierung,
  Filter-Query, Persistenz, Cache. Erlaubt nur, wenn beim ersten Vorkommen erklärt:
  „ein Eintrag (eine gespeicherte Person)".
- **Ergebnis jedes Schritts benennen:** „Der Dialog schließt sich, die neue Person steht in der Liste."
  Der Leser muss prüfen können, ob er richtig liegt.
- **Keine Abkürzungen** (z. B. → „zum Beispiel"), **keine Emojis**, keine Ausrufezeichen.
- **Nichts verstecken:** Was zwei Wege hat, bekommt einen empfohlenen Weg — der zweite steht unter
  „Gut zu wissen", nicht mitten im Ablauf.

---

## 5. Screenshots (verbindlich)

**Motivation:** Bilder sind der eigentliche Inhalt der Anwenderdoku. Sie müssen jederzeit reproduzierbar
sein, sonst veralten sie unbemerkt.

### Erzeugung

- Screenshots werden **automatisiert mit Playwright** aus dem UI-Test-Projekt erzeugt, nie von Hand
  abfotografiert oder ausgeschnitten. Ablage der Skripte: `tests/ui/docs/{modul}.shots.ts`.
- Sie laufen gegen die **Testdatenbank mit dem Standard-Seed** (siehe [`_uitests.md`](_uitests.md)).
  Damit zeigen alle Bilder dieselben, unverfänglichen Beispieldaten.
- **Feste Aufnahmebedingungen** für alle Bilder: Viewport `1440 × 900`, `deviceScaleFactor: 2`,
  Sprache `de-DE`, Animationen aus (`animations: 'disabled'`), heller Modus.
- Ausgabepfad direkt in die Doku: `docs/user/E{n}-{modul}/bilder/`.
- Ein Screenshot zeigt **eine Sache**: entweder die ganze Seite (`fullPage: false`, Viewport-Ausschnitt)
  oder einen Ausschnitt (`locator.screenshot()`) für Dialoge und einzelne Bereiche.

```ts
// tests/ui/docs/personenverwaltung.shots.ts
test('Personenliste – Übersicht', async ({ page }) => {
  await page.goto('/personen')
  await page.screenshot({
    path: '../../docs/user/E1-personenverwaltung/bilder/personenliste-01-uebersicht.png',
    animations: 'disabled',
  })
})
```

### Übergangsregelung (bis das UI-Test-Projekt steht)

Solange `tests/ui/` nicht existiert, werden Screenshots ersatzweise über die Browser-Automatisierung
gegen den laufenden Entwicklungs-Stack aufgenommen (`docker compose up`, `http://localhost:8081`),
mit denselben festen Aufnahmebedingungen und anschließendem Zuschnitt auf PNG. Diese Bilder zeigen
Entwicklungsdaten statt Seed-Daten und sind **nicht** reproduzierbar. Die Ausnahme endet, sobald das
Playwright-Projekt existiert; die Bilder sind dann einmalig neu zu erzeugen.

### Benennung

`{seite}-{nr}-{inhalt}.png` — klein, ohne Umlaute, Nummer zweistellig und in der Reihenfolge der
Verwendung im Text: `person-anlegen-03-pflichtfelder.png`.

### Einbindung im Text

- Bild **direkt unter** dem Schritt, den es zeigt, mit erklärendem Alternativtext und Bildunterschrift:

  ```markdown
  ![Die Personenliste mit der Schaltfläche Anlegen oben rechts](bilder/personenliste-01-uebersicht.png)
  *Bild 1: Die Personenliste nach dem Öffnen.*
  ```

- **Alternativtext beschreibt das Bild**, er wiederholt nicht die Bildunterschrift.
- **Markierungen:** Werden einzelne Elemente erklärt, trägt das Bild nummerierte orange Kreise
  (Akzentfarbe aus [`_uiux.md`](_uiux.md)), und direkt darunter steht die Tabelle
  `| Nr. | Element | Bedeutung |`. Keine Pfeile, keine Freihandkritzelei, keine roten Rahmen.
- **Keine echten Daten.** Keine realen Namen, Adressen, E-Mails, Telefonnummern, Kundennummern.
  Ausschließlich Seed-Daten.
- **Keine Bildschirmränder**, keine Browserleiste, keine Betriebssystem-Elemente im Bild.

### Aktualität

- Ändert sich eine Seite sichtbar, werden **im selben Arbeitsschritt** die Screenshots neu erzeugt.
  Ein veralteter Screenshot gilt als Fehler in der Dokumentation, nicht als Kleinigkeit.
- Ein Screenshot, auf den kein Text mehr verweist, wird gelöscht.

---

## 6. FAQ (verbindlich)

Jedes Modul hat eine `faq.md`. Zusätzlich gibt es unter `docs/user/faq.md` eine modulübergreifende FAQ
für alles, was die ganze Anwendung betrifft (Anmeldung, Navigation, Browser, Drucken).

- **Format:** Frage als Überschrift der dritten Ebene, Antwort darunter in höchstens fünf Sätzen.

  ```markdown
  ### Warum kann ich eine Person nicht löschen?
  Eine Person lässt sich nur löschen, wenn ihr kein Haustier zugeordnet ist. …
  ```

- **Fragen in der Sprache des Anwenders**, nicht in der des Entwicklers: „Warum finde ich meine Person
  nicht?" statt „Verhalten des Suchfilters bei Teilstrings".
- **Jede Fehlermeldung der Oberfläche gehört in die FAQ** — im Wortlaut, mit Ursache und Lösung.
- **Jede FAQ-Antwort endet mit einem Link** auf die Seiten-Datei, in der der Vorgang ausführlich steht.
- **Mindestens fünf Fragen pro Modul.** Weniger bedeutet, dass die Aufgaben des Moduls nicht
  durchdacht wurden.
- **Reihenfolge:** häufigste Frage zuerst, Sonderfälle zuletzt. Keine alphabetische Sortierung.

---

## 7. Bezug zu den Anforderungen

- Die Dokumentation beschreibt den **implementierten Stand**. Es wird nur dokumentiert, was in `specs/`
  auf `State: Implemented` steht (siehe [`_requirements.md`](_requirements.md)).
- **Ausnahme für Module in Vorbereitung:** Soll ein noch nicht implementiertes Modul vorab dokumentiert
  werden, ist das eine ausdrückliche Einzelfall-Entscheidung des Stakeholders. Jede betroffene Datei
  trägt dann direkt nach der Überschrift den Hinweisblock „Dieses Modul befindet sich in Vorbereitung …"
  und enthält keine Bildverweise. Sobald das Modul steht, werden die Seiten am Code geprüft, bebildert
  und der Hinweisblock entfernt.
- **Vollständigkeit:** Zu jeder implementierten Story existiert der zugehörige Ablauf in der
  Anwenderdoku — als eigener Abschnitt „Schritt für Schritt" oder als eigene Seiten-Datei.
- **Akzeptanzkriterien sind keine Doku.** Sie werden nicht abgeschrieben, sondern in Handlungsanweisungen
  übersetzt.
- Ändert sich eine Story samt Code, wird die betroffene Seiten-Datei **in derselben Session**
  nachgezogen — Doku, Spec und Code gehen gemeinsam in einen Commit.

---

## 8. Checkliste vor dem Commit

Eine Seiten-Datei ist fertig, wenn alle Punkte zutreffen:

- [ ] Alle Kapitel aus Abschnitt 3 sind vorhanden und in der richtigen Reihenfolge.
- [ ] Jeder Arbeitsablauf ist nummeriert, ein Klick pro Schritt, mit benanntem Ergebnis.
- [ ] Jeder Ablauf hat mindestens einen Screenshot; alle Screenshots sind aktuell erzeugt.
- [ ] Alle Beschriftungen stimmen wörtlich mit der Oberfläche überein.
- [ ] Kein unerklärter Fachbegriff, keine Abkürzung, keine echten Personendaten.
- [ ] Das Modul hat eine `faq.md` mit mindestens fünf Fragen inklusive aller Fehlermeldungen.
- [ ] Modul-`README.md` und `docs/user/README.md` verlinken die neue Seite.
- [ ] Kein toter Link und kein fehlendes Bild — alle Verweise zeigen auf vorhandene Dateien.
- [ ] Eine Person, die die Anwendung nicht kennt, käme allein mit diesem Text ans Ziel.
