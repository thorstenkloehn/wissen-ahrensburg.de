# Dedizierte KI-Code-Wiki-Tools (direkt im Repository)

*Hinweis: Diese Inhalte wurden mit Unterstützung von Künstlicher Intelligenz erstellt und redaktionell überprüft (Transparenzhinweis gemäß Art. 50 EU AI Act).*

Dieses Dokument beschreibt die Werkzeugkategorie **„dedizierte KI-Code-Wiki-Tools“**, die eine
maschinell erzeugte Wissensbasis über eine Codebasis anlegen und diese **direkt im Repository**
ablegen – als versionierte Markdown-Dateien neben dem Quellcode. In diesem Repository wird
dieser Ansatz über **OpenWiki** genutzt (Ordner [`openwiki/`](openwiki/), Agent-Hinweise in
[`AGENTS.md`](AGENTS.md)).

## Inhalt

- [Worum geht es?](#worum-geht-es)
- [Das LLM-Wiki-Pattern (Karpathy-Muster)](#das-llm-wiki-pattern-karpathy-muster)
- [Google Open Knowledge Format (OKF)](#google-open-knowledge-format-okf)
- [Abgrenzung](#abgrenzung)
- [Vor- und Nachteile](#vor--und-nachteile)
- [Eigenständige Knowledge-Bases & Wikis mit KI](#eigenständige-knowledge-bases--wikis-mit-ki)
- [Kombination mit KI-Prompt-Bundlern](#kombination-mit-ki-prompt-bundlern)
- [Gehostete KI-Code-Wiki-Dienste](#gehostete-ki-code-wiki-dienste)
- [Vergleich der Ansätze](#vergleich-der-ansätze)
- [Nutzung in diesem Repository](#nutzung-in-diesem-repository)
- [Agenten-Skills in diesem Projekt](#agenten-skills-in-diesem-projekt)
- [Co-Wiki: Mensch & KI](#co-wiki-mensch--ki)
- [Software für ein großes Wissensprojekt](#software-für-ein-großes-wissensprojekt)
- [Weiterführend](#weiterführend)

## Worum geht es?

Ein KI-Code-Wiki analysiert Quellcode, Konfiguration, Tests und vorhandene Dokumentation und
erzeugt daraus eine strukturierte, verlinkte Wissensbasis: Überblick, Architektur, Datenflüsse,
zentrale Module, oft mit automatisch generierten Diagrammen (meist Mermaid).

Bei der hier gemeinten Variante gilt:

- Die Ausgabe wird **in das Repository committet** (nicht nur auf einer externen Website gehostet).
- Die Aktualisierung läuft **automatisiert**, typischerweise über einen geplanten CI-Lauf
  (z. B. GitHub Actions), der die Seiten neu generiert und per Commit/PR einspielt.
- Die Seiten sind **generiert** und werden nicht von Hand gepflegt – Änderungen gehören in den
  Quellcode bzw. die Quelldokumentation, das Wiki wird danach neu erzeugt.

## Das LLM-Wiki-Pattern (Karpathy-Muster)

Der gemeinsame Kern dieser Werkzeuge geht auf ein von **Andrej Karpathy** beschriebenes Muster
zurück ([Gist](https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f)): Statt bei
jeder Frage roh über die Quellen zu suchen (RAG), pflegt ein LLM-Agent **inkrementell ein
persistentes, quervernetztes Markdown-Wiki**. Man kuratiert die Quellen und stellt Fragen – der
Agent übernimmt die Buchführung: neue Quelle lesen, in typisierte Seiten einordnen, Verweise
setzen.

Kennzeichen des Musters:

- **Markdown-Dateien** als Format – für Menschen wie für LLMs gut lesbar, versionierbar.
- **Typisierte Seiten** mit Frontmatter (`type`, `title`, `description`, `tags`) und benannten
  Abschnitten.
- **Interlinks** zwischen den Seiten; das Wiki ist ein Graph, kein flacher Ordner.
- **Der Agent schreibt, der Mensch kuratiert** – die Pflege ist ein wiederholbarer Lauf, kein
  manuelles Redigieren.

In diesem Repository folgen `openwiki/*.md` sowie [`CLAUDE.md`](CLAUDE.md) und
[`AGENTS.md`](AGENTS.md) diesem Schema. Verwandt, aber nach außen gerichtet, ist
[`llms.txt`](https://llmstxt.org/) (Jeremy Howard): eine Datei, die externen LLMs die
Website/Repo-Struktur erklärt.

**Google Code Wiki** (<https://codewiki.google>) ist eine gehostete Umsetzung desselben
Gedankens: Ein Gemini-Agent erzeugt aus einem GitHub-Repository ein interaktives Wiki mit
Architektur-, Klassen- und Sequenzdiagrammen, Modul-Rundgängen und einem Chatbot über den Code,
und regeneriert es nach jedem Commit. Zugriff für öffentliche Repos über
`codewiki.google/github.com/owner/repo`.

## Google Open Knowledge Format (OKF)

Googles Open Knowledge Format (OKF) ist ein offener, herstellerneutraler Standard, den Google
Cloud im Juni 2026 eingeführt hat
([Spezifikation](https://github.com/GoogleCloudPlatform/open-knowledge-format)). Er strukturiert
Unternehmenswissen und -kontext in portable Verzeichnisse von Markdown-Dateien für KI-Agenten.

OKF formalisiert das oben beschriebene Muster:

- **Ein Verzeichnis von Markdown-Dateien** mit YAML-Frontmatter, für Menschen wie für Agenten
  lesbar.
- **Minimales Schema**: ein Pflichtfeld (`type`), optionale Metadaten (`title`, `description`,
  `resource`, `tags`, `timestamp`), darunter ein frei formulierter Markdown-Text.
- **Produzenten und Konsumenten sind entkoppelt**: Wer OKF schreibt (Mensch, Agent,
  Export-Pipeline) und wer es liest (Dateiserver, Wissens-Oberfläche, LLM-Kontext, Suchindex,
  Graph-Viewer), müssen sich nicht kennen – kein Aufwand für individuelle Integrationen.
- Ziel ist das „Context-Assembly-Problem“: verstreutes Wissen in **ein** Format bringen, das
  jeder Agent ohne Übersetzung konsumieren kann. v0.1 im Juni 2026, v0.2 im Juli 2026.

Die `openwiki/`-Seiten dieses Repositories nutzen bereits dasselbe Frontmatter-Schema
(`type` / `title` / `description` / `tags`) und sind damit OKF-nah.

## Abgrenzung

| Ansatz | Ablageort | Beispiele |
|---|---|---|
| **Dediziert, im Repository** (dieses Dokument) | Markdown im Repo, versioniert | OpenWiki, DeepWiki‑Open (self‑hosted), AIGNE DocSmith |
| Gehostet / extern | Externe Website, nicht im Repo | DeepWiki.com (Cognition), Google Code Wiki |
| In den Code eingebettet | Doku-Snippets neben dem Code, an Code-Zeilen gekoppelt | Swimm |
| Klassische Doku-Generatoren | Aus Docstrings/Kommentaren, kein KI-Verständnis | Doxygen, Sphinx, TypeDoc |

## Vor- und Nachteile

**Vorteile**

- Wiki ist **offline und versioniert** verfügbar, Diffs pro Änderung nachvollziehbar.
- Funktioniert bei **privaten Repositories**, ohne Code an einen Drittanbieter-Dienst zu geben
  (abhängig vom gewählten Modell-Backend).
- Nutzbar als **Just-in-time-Kontext für Agenten** (siehe `AGENTS.md`), ohne Netzabruf.
- Kein Vendor-Lock-in am Hosting: Die Dateien bleiben, auch wenn das Tool wechselt.

**Nachteile**

- **Repo-Rauschen**: generierte Dateien und regelmäßige Commits/PRs.
- **Veralten** zwischen zwei Generierungsläufen; Stand ist immer der des letzten Laufs.
- **Kosten/Token** für das verwendete Sprachmodell bei jedem Lauf.
- Gefahr, generierte Aussagen für autoritativ zu halten – **Quellcode und Tests bleiben maßgeblich**.

## Eigenständige Knowledge-Bases & Wikis mit KI

Neben den code-orientierten Werkzeugen oben steht eine breitere Kategorie: **allgemeine
Wissensdatenbanken und Wikis, die KI einsetzen** – für Inhaltserstellung, Zusammenfassung,
semantische Suche oder ein Frage-Antwort-Interface über den gesamten Bestand. Sie sind nicht an
eine Codebasis gebunden und dienen Redaktion, Team-Dokumentation oder persönlichem Wissen.

| Typ | Beispiele | KI-Rolle |
|---|---|---|
| KI-natives Team-Wiki | Notion (Q&A, Agents), Guru, Slite, Tettra | Antworten, Verifikation, Entwurf |
| Open-Source-Wiki / KB | MediaWiki, Outline, BookStack, Wiki.js, Docmost | KI über Erweiterungen/Plugins |
| Persönliche Wissensbasis | Obsidian, Logseq, Mem | lokale Notizen, Verlinkung, KI-Assistenz |
| KI-Orchestrierung über Quellen | Dust, Glean | Agenten lesen/suchen in vorhandenem Wissen |
| Statische, KI-gestützte KB | mdBook / Docusaurus / MkDocs + LLM-Autoren | Autoren-Workflow, Text KI-assistiert |

**Dieses Repository** gehört in die letzte Zeile: eine eigenständige, öffentlich publizierte
Wissensdatenbank (mdBook über Ahrensburg), deren Texte KI-assistiert entstehen und redaktionell
geprüft werden – mit Transparenzhinweis nach Art. 50 EU AI Act. Der `openwiki/`-Index ist davon
getrennt und beschreibt das Repository selbst, nicht die Stadt Ahrensburg.

Abgrenzung zu den Repo-Code-Wikis:

- **Zielgruppe**: Menschen/Redaktion statt Entwickler und Agenten.
- **Quelle der Wahrheit**: die kuratierten Inhalte selbst, nicht ein Quellcode-Stand.
- **Pflege**: laufende redaktionelle Arbeit, nicht ein Regenerierungslauf.

## Kombination mit KI-Prompt-Bundlern

KI-Prompt-Bundler wie [Repomix](https://github.com/yamadashy/repomix) oder
[Gitingest](https://github.com/cyclotruc/gitingest) fassen ein ganzes Repository in einer einzigen,
LLM-tauglichen Textdatei zusammen (XML, Markdown oder Plain Text). Zusammen mit einem
KI-Code-Wiki ergibt das einen praktikablen Ablauf:

1. **Bündeln** – Repomix/Gitingest erzeugen einen kompakten Repository-Snapshot als Kontext.
2. **Generieren** – ein CLI-Agent schreibt bzw. aktualisiert daraus die Wiki-Kapitel als Markdown.
3. **Versionieren** – die Markdown-Dateien werden im Git-Repository eingecheckt und mitentwickelt.
4. **Aktualisieren** – bei Änderungen am Code läuft der Zyklus (geplant oder pro PR) erneut.

So bleibt die Dokumentation als Markdown im Repository versioniert, während CLI-Agenten die
Kapitel automatisch generieren und aktuell halten. Der Bündler liefert den Kontext, das Wiki-Tool
die Struktur, Git die Historie.

## Gehostete KI-Code-Wiki-Dienste

Das Gegenmodell zum Ansatz „direkt im Repository“: Ein externer Dienst indiziert das Repository
und stellt das Wiki auf **seiner eigenen Plattform** bereit – nichts wird ins Repository
committet.

| Dienst | Zugang | Merkmale |
|---|---|---|
| [DeepWiki](https://deepwiki.com) (Cognition/Devin) | `github.com` → `deepwiki.com` in der URL ersetzen | Auto-Wiki für öffentliche Repos, Architektur­diagramme, Chat über den Code, MCP-Server für Cursor/Claude Code; private Repos über ein Devin-Konto |
| [Google Code Wiki](https://codewiki.google) | `codewiki.google/github.com/owner/repo` | Gemini-generiertes Wiki, Architektur-/Klassen-/Sequenzdiagramme, Codebase-Chat, Regenerierung nach jedem Commit; öffentliche Repos gratis, private per Gemini-CLI (Warteliste) |
| [GitBook](https://gitbook.com), [Mintlify](https://mintlify.com) | gehostete Doku-Plattform + Repo-Sync | KI-Suche und Q&A über gepflegte Doku, teils Auto-Entwürfe aus dem Code |

**Vorteile**

- **Kein Repo-Rauschen**, keine CI-Pipeline, keine Wartung der Generierung.
- Sofort nutzbar; für populäre Open-Source-Repos oft schon vorindiziert.
- Interaktives Q&A/Chat-Interface über die Codebasis ohne eigenes Setup.

**Nachteile**

- **Code verlässt die eigene Infrastruktur** – für private/vertrauliche Repos meist ein Ausschlusskriterium
  (bzw. nur mit kostenpflichtigem Konto und Vertrauensstellung).
- **Nicht versioniert im Repo**: kein Diff, kein Offline-Zugriff, keine Nutzung als lokaler Agenten-Kontext.
- **Abhängigkeit vom Anbieter**: Verfügbarkeit, Preismodell, Einstellung des Dienstes.
- Stand richtet sich nach dem Indizierungs-Rhythmus des Anbieters, nicht nach eigenen Läufen.

Für dieses Repository ist der In-Repo-Ansatz (OpenWiki) gewählt; ein gehosteter Dienst wie
DeepWiki funktioniert hier zusätzlich, weil das Repository öffentlich ist.

## Vergleich der Ansätze

| Ansatz | Im Repo versioniert | Aktualisierung | Code bleibt lokal | Q&A-/Chat-Interface | Zielgruppe | Beispiele |
|---|---|---|---|---|---|---|
| Dediziert, im Repository | ja | CI-Lauf / pro PR | ja | nur über einen Agenten | Entwickler & Agenten | OpenWiki, DeepWiki-Open (self-hosted), AIGNE DocSmith |
| Gehosteter Dienst | nein | Indizierungs-Rhythmus des Anbieters (Code Wiki: nach jedem Commit) | nein (bei öffentlichen Repos unkritisch) | ja | Entwickler | DeepWiki.com, Google Code Wiki |
| Prompt-Bundler + CLI-Agent | ja | Skript / CI | ja | nein | Entwickler & Agenten | Repomix / Gitingest + CLI-Agent |
| In den Code eingebettet | ja, an Code-Zeilen gekoppelt | manuell, CI prüft auf Drift | ja | nein | Entwickler beim Lesen des Codes | Swimm |
| Eigenständige KI-Wissensbasis | statisch: ja · SaaS: nein | laufende redaktionelle Arbeit | statisch: ja · SaaS: nein | SaaS: ja | Redaktion, Team, Leserschaft | mdBook/MkDocs + LLM-Autoren, Notion, MediaWiki, Outline |
| Klassischer Doku-Generator | ja | Build aus Docstrings/Kommentaren | ja | nein | Entwickler | Doxygen, Sphinx, TypeDoc |

Dieses Repository nutzt Zeile 1 (OpenWiki für `openwiki/`) und ist zugleich selbst ein Fall von
Zeile 5 (die mdBook-Wissensdatenbank über Ahrensburg, KI-assistiert verfasst).

## Nutzung in diesem Repository

- Der generierte Index liegt unter [`openwiki/`](openwiki/):
    - [`openwiki/overview.md`](openwiki/overview.md) – Projektüberblick
    - [`openwiki/architecture.md`](openwiki/architecture.md) – Build- und Content-Architektur
    - [`openwiki/source-map.md`](openwiki/source-map.md) – Verzeichnis der Quelldateien
    - [`openwiki/INSTRUCTIONS.md`](openwiki/INSTRUCTIONS.md) – Kurzbeschreibung des Wikis
- [`AGENTS.md`](AGENTS.md) beschreibt, wie Agenten das Wiki behandeln sollen: optionaler
  Kontext, keine Startpflichtlektüre, generierte Seiten **nicht** von Hand editieren.
- Ein geplanter OpenWiki-GitHub-Actions-Workflow aktualisiert die Seiten. Inhaltliche
  Korrekturen erfolgen im Quellcode bzw. in den Quelltexten unter `src/`, danach regeneriert
  OpenWiki.
- `src/openwiki/{overview,architecture,source-map}.md` sind **manuelle Kopien** dieser Seiten,
  damit sie in `src/SUMMARY.md` verlinkt und mit ins mdBook gebaut werden können (mdBook baut nur
  Dateien unter `src/`). Sie tragen den Transparenzhinweis und einen Kopie-Vermerk. Der
  OpenWiki-Workflow aktualisiert `openwiki/`, **nicht** diese Kopien – bei Änderungen an den
  Originalen von Hand nachziehen. Verifikations-Screenshots liegen unter
  [`docs/verification/`](docs/verification/).

## Agenten-Skills in diesem Projekt

Für Coding- und Redaktions-Agenten (Claude Code, Antigravity/agy, Codex) sind im Repository dedizierte **Skills** hinterlegt (gespiegelt unter `.claude/skills/` und `.agents/skills/`):

| Skill | Pfad | Beschreibung |
|---|---|---|
| **`co-wiki`** | [`.claude/skills/co-wiki/SKILL.md`](.claude/skills/co-wiki/SKILL.md) / [`.agents/skills/co-wiki/SKILL.md`](.agents/skills/co-wiki/SKILL.md) | Das Co-Wiki-Modell dieses Repos: Mensch-KI-Zusammenarbeit, die vier Arbeitsmodelle, Docs-as-Code (mdBook), MCP/RAG, Multi-Agenten-Redaktionspipeline (`redaktion-*`) und Event-Driven Documentation. |
| **`urheberrecht-pruefung`** | [`.claude/skills/urheberrecht-pruefung/SKILL.md`](.claude/skills/urheberrecht-pruefung/SKILL.md) / [`.agents/skills/urheberrecht-pruefung/SKILL.md`](.agents/skills/urheberrecht-pruefung/SKILL.md) | Systematischer Audit-Leitfaden zur Prüfung aller Inhaltsseiten (`src/*.md`) auf wörtliche Textübernahmen, enge Paraphrasen (§ 23 UrhG), Schutz regionaler Quellen und Einhaltung der CC BY-SA 4.0-Lizenzbedingungen bei Wikipedia-Inhalten. |

Weitere operative Richtlinien und die Rollenverteilung der Redaktions-Subagenten finden sich in [`AGENTS.md`](AGENTS.md) und [`CLAUDE.md`](CLAUDE.md).

## Co-Wiki: Mensch & KI

Ein Co-Wiki ist ein digitales Wissensbuch, bei dem Menschen und künstliche Intelligenz (KI) Hand in Hand zusammenarbeiten. Diese Wissensdatenbank über Ahrensburg ist auf diese Weise entstanden.

### Was teilen sich Mensch und KI?

Beide sitzen virtuell am selben Schreibtisch: Sie greifen auf denselben Ordner zu und lesen dieselben Textdateien (oft im einfachen Notizformat namens Markdown).

### Was macht der Mensch? (Der Chef)

Der Mensch hat den Durchblick, trägt die Verantwortung und trifft alle wichtigen Entscheidungen:

- **Themen aussuchen:** Bestimmen, worüber überhaupt geschrieben werden soll.
- **Aufträge erteilen (Prompts):** Der KI genau sagen, was sie tun soll (z. B. „Erkläre dieses Thema in 5 einfachen Schritten“).
- **Hausaufgaben-Kontrolle (Faktencheck):** Prüfen, ob alle Angaben stimmen und die KI keine Märchen erfindet (sogenannte Halluzinationen).
- **Feinschliff & Gefühl:** Den Text lebendig machen, damit er sich spannend und leicht lesen lässt.
- **Freigabe:** Entscheiden, wann ein Text fertig ist und für alle veröffentlicht werden darf.

### Was macht die KI? (Der fleißige Assistent)

Die KI ist blitzschnell und übernimmt die anstrengenden Fleißaufgaben:

- **Erste Entwürfe schreiben:** In wenigen Sekunden einen vollständigen Rohtext verfassen.
- **Zusammenfassen:** Riesige, komplizierte Dokumente oder Notizen in kurze, knackige Zusammenfassungen verwandeln.
- **Ordnung & Formatierung:** Unordentliche Notizen automatisch in saubere Tabellen, Aufzählungen und Absätze umwandeln.
- **Verlinken & Stichwörter verteilen:** Passende Querverweise (Links) zu anderen Seiten im Lexikon vorschlagen.
- **Fehlerteufel jagen:** Rechtschreibung, Grammatik und Satzbau blitzschnell korrigieren.

### Was ist die Aufgabe des Wikis selbst? (Die Bibliothek)

Das Wiki ist das digitale Bücherregal, in dem alles aufbewahrt wird:

- **Zentraler Speicherort:** Alle Anleitungen, Notizen und Ideen liegen an einem einzigen Ort, den man durchsuchen kann.
- **Gemeinsame Schnittstelle:** Mensch und Computer können gleichzeitig darin lesen und schreiben.
- **Ordnung & Vernetzung:** Inhalte werden mit Kategorien, Suchbegriffen (Tags) und Links wie ein Spinnennetz verknüpft.
- **Eingebaute Zeitmaschine (Versionsgeschichte):** Das Wiki merkt sich jeden einzelnen Tastendruck. Wenn jemand aus Versehen etwas löscht, kann man einfach per Knopfdruck in die Vergangenheit springen.

### Wie arbeiten Mensch und KI zusammen? (Arbeitsmodelle)

- **Mensch schreibt, KI korrigiert:** Du schreibst den Aufsatz, die KI prüft Rechtschreibung und sucht passende Links.
- **KI schreibt vor, Mensch verbessert:** Die KI liefert den ersten Entwurf, du machst ihn richtig gut und besser verständlich.
- **Gleichzeitig nebeneinander:** Du schreibst an Kapitel 1, während die KI parallel schon an Kapitel 2 arbeitet.
- **Selbstständiger Roboter:** Die KI arbeitet Aufgaben alleine ab, aber am Ende muss immer ein Mensch auf „Freigeben“ drücken.

### Der typische Ablauf (Vom Gedanken zum fertigen Artikel)

1. **Idee & Thema:** Der Mensch wählt ein Thema (oder die KI bemerkt, dass ein wichtiges Thema noch fehlt).
2. **Recherche & Spickzettel (RAG):** Die KI durchsucht alte Notizen und Unterlagen, um echte Fakten zu sammeln.
3. **Erster Entwurf:** Die KI schreibt die Rohfassung mit ersten Überschriften.
4. **Menschliche Prüfung (Human-in-the-Loop):** Ein Mensch liest alles durch, korrigiert Fehler und passt den Schreibstil an.
5. **Veröffentlichung:** Der fertige Text wird für alle im Wiki sichtbar gemacht.
6. **Jungbrunnen-Pflege:** KI-Helfer prüfen regelmäßig, ob Links noch funktionieren oder Infos veraltet sind.

### Wie funktioniert das technisch? (Einfach erklärt)

- **Dateien wie am Computer (Docs-as-Code):** Das Wiki besteht aus einfachen Textdateien, die mit Programmen wie Obsidian, Quartz oder MkDocs in schicke Webseiten verwandelt werden. Diese Seite nutzt mdBook.
- **Universal-Stecker für KI (Model Context Protocol / MCP):** Wie ein universelles USB-Kabel, über das die KI sicher auf Dateien, Notizen und das Internet zugreifen kann.
- **Schlaue Bedeutungssuche (Vektordatenbank & RAG):** Die KI sucht nicht nur nach genauen Wörtern, sondern versteht die Bedeutung (z. B. findet sie „Hund“, wenn du nach „Haustier mit vier Pfoten“ suchst).

### Das Roboter-Team (Multi-Agenten-Redaktion)

Für größere Artikel arbeitet nicht nur eine KI, sondern ein ganzes Team aus spezialisierten Roboter-Helfern zusammen – wie in einer echten Zeitungsredaktion:

- **Der Forscher:** Durchstöbert Unterlagen und das Internet nach echten Fakten und packt alles in eine Info-Mappe.
- **Der Architekt:** Erstellt den Bauplan des Textes – welche Überschriften gibt es, worum geht es in jedem Absatz?
- **Der Schreiberling:** Verfasst anhand des Bauplans den eigentlichen Fließtext.
- **Der Lektor:** Prüft, ob der Text spannend klingt, leicht verständlich ist und keine doppelten Sätze enthält.
- **Der Detektiv & Faktenchecker:** Vergleicht jedes Wort mit der Info-Mappe, damit keine erfundenen Märchen im Text landen.
- **Der Verlinker:** Setzt passende Links zu anderen Artikeln und vergibt Schlagwörter.
- **Der Chef vom Dienst:** Behält den Überblick, verteilt die Aufgaben und sagt „Weiter!“ oder „Bitte nochmal überarbeiten!“.

Der Ablauf: Briefing → Recherche → Bauplan → Text schreiben → gemeinsame Kontrolle durch Faktenchecker und Lektor (bei Fehlern geht es zwei bis drei Runden zurück zum Schreiberling) → und zum Schluss die menschliche Freigabe.

### Das Buch, das sich selbst aktualisiert (Ereignisgesteuerte Pflege)

Oft vergisst man, eine Anleitung neu zu schreiben, wenn sich etwas geändert hat. Hier merkt der Computer die Veränderung von allein:

1. **Klingelzeichen:** Das System meldet: „Achtung, es gibt eine wichtige Neuerung!“
2. **Wichtigkeits-Check:** Die KI prüft, ob das für eine Wiki-Seite überhaupt wichtig ist (Kleinkram wird ignoriert).
3. **Alte Seite finden:** Die KI schaut nach, wo die veraltete Erklärung steht.
4. **Gezieltes Ausbessern:** Die KI tauscht nur den veralteten Satz oder das Schaubild aus, statt den ganzen Artikel wegzuwerfen.
5. **Vorschlag an den Menschen:** Die KI zeigt genau, was sich ändern würde (rot = alt, grün = neu).
6. **Freigabe per Klick:** Ein Mensch bestätigt – und das Lexikon ist wieder brandaktuell.

Der Vorteil: Das Wissen veraltet nicht, mehrere kleine Änderungen werden gesammelt in einem sauberen Durchgang erledigt, und niemand muss veraltete Seiten mühsam von Hand suchen.

### Mehr zum technischen Hintergrund

Wie dieses Repository aufgebaut ist und wie daraus die Website wird, steht in [`openwiki/overview.md`](openwiki/overview.md) und [`openwiki/architecture.md`](openwiki/architecture.md).

## Software für ein großes Wissensprojekt

Dieses Wiki über Ahrensburg soll langfristig sehr groß werden – viele tausend
Artikel, geschrieben von Menschen und künstlicher Intelligenz (KI) gemeinsam
(siehe Co-Wiki: Mensch & KI). Für so ein Projekt muss man am Anfang
eine wichtige Frage beantworten: **Mit welchem Programm sammelt und verwaltet man
das ganze Wissen?**

Diese Seite erklärt, worauf es dabei ankommt.

### Kurz erklärt: Wiki, CMS, Doku-Generator

Alle drei Arten von Programmen helfen dabei, Texte im Internet zu
veröffentlichen. Sie sind aber für unterschiedliche Zwecke gebaut.

| Art | Wofür gemacht | Bild dazu |
|---|---|---|
| **Wiki** | Viele Menschen schreiben gemeinsam an einem großen Nachschlagewerk. Jeder darf Seiten anlegen und ändern, jede Änderung wird gespeichert. | Ein öffentliches Notizbuch, in das alle hineinschreiben dürfen. |
| **CMS** (Redaktionssystem, englisch *Content-Management-System*) | Eine Redaktion pflegt eine Website mit Artikeln, Bildern, Menüs und festem Aussehen. Wenige Leute schreiben, viele lesen. | Eine Zeitungsredaktion mit fester Ausgabe. |
| **Doku-Generator** | Aus einfachen Textdateien (im Notizformat *Markdown*) wird automatisch ein sauberes Online-Buch. Die Texte liegen als Dateien, nicht in einer Datenbank. | Eine Druckmaschine: vorne Zettel rein, hinten fertiges Buch raus. |

Dieses Wiki nutzt heute einen Doku-Generator namens **mdBook**. Ob das auf Dauer
für zehntausende Artikel reicht, ist genau die Frage, um die es hier geht.

### Was dieses Projekt vom Programm verlangt

Die folgende Liste ist der Maßstab. Jedes Programm in der Praxis wird
an diesen Punkten gemessen.

#### 1. Sehr viele Artikel verwalten

Das Ziel sind **mehr als 10.000 Artikel**, später gern noch viel mehr. Das
Programm muss so viele Seiten schnell anzeigen, durchsuchbar halten und im
Bearbeiten flüssig bleiben. Manche Programme sind für ein paar hundert Seiten
gedacht und werden dann langsam.

#### 2. Offen und kostenlos (Open Source)

*Open Source* heißt: Der Bauplan des Programms (der Quelltext) ist öffentlich.
Jeder darf das Programm gratis benutzen, verändern und weitergeben. Wichtig, weil
das Projekt niemandem gehören soll und nicht von einer Firma abhängig sein darf,
die es später abschaltet oder teuer macht.

#### 3. Keine persönlichen Daten (PII-frei)

*PII* steht für „persönlich identifizierbare Informationen" – also Namen,
E-Mail-Adressen, Zugangsdaten von Nutzern. Die **Inhalte** des Wikis sollen
komplett frei von solchen Nutzerdaten sein. So kann jeder den gesamten Text
herunterladen, ohne dass Datenschutz-Probleme entstehen. Konten und Passwörter
der Autorinnen und Autoren bleiben getrennt davon und werden nicht mitgegeben.

#### 4. Datenbank-neutrale Sicherungsdatei

Das komplette Wissen soll sich in **eine einzige Datei** exportieren lassen, die
jeder öffnen und weiterverwenden kann – zum Beispiel `ahrensburg.xml`,
`ahrensburg.json` oder `ahrensburg.yaml`. „Datenbank-neutral" bedeutet: Die Datei
funktioniert auch ohne genau das Programm, das sie erzeugt hat. Sie ist kein
verschlüsselter Klumpen, sondern lesbarer Text in einem verbreiteten Format.

- **XML, JSON und YAML** sind drei solche Textformate. Sie schreiben Daten so
  auf, dass sowohl Menschen als auch Programme sie verstehen. XML nutzt
  spitze Klammern, sogenannte Tags (`<titel>…</titel>`); JSON und YAML sind
  kompakter.
- Mit so einer Datei kann jemand das Wissen auf einer anderen Website, in einem
  anderen Programm oder in einem gedruckten Buch weiterverwenden.

#### 5. Speicherort: Datenbank oder Dateien

Es gibt zwei Wege, wie ein Programm die Texte ablegt:

| Weg | Wie es funktioniert | Vorteil | Nachteil |
|---|---|---|---|
| **Datenbank** (oft *PostgreSQL* – eine bekannte, kostenlose Datenbank) | Alle Texte stecken in einer Datenbank, einer Art riesiger sortierter Tabelle. | Schnell auch bei sehr vielen Seiten; viele Leute können gleichzeitig schreiben. | Man braucht einen laufenden Datenbank-Server; Sicherung ist aufwendiger. |
| **Dateisystem** (jede Seite eine Datei) | Jede Seite liegt als einzelne Textdatei in einem Ordner. | Einfach zu sichern, zu kopieren, per Versionsverwaltung *Git* zu verfolgen. | Wird bei zehntausenden Dateien und mehreren gleichzeitigen Autoren unübersichtlich und langsam. |

Für dieses Projekt wäre **PostgreSQL oder ein sauberes Dateisystem** in Ordnung –
solange Punkt 4 (die Export-Datei) erfüllt ist.

#### 6. Reifegrad und aktive Entwicklung

„Reif" heißt: Das Programm gibt es seit vielen Jahren, es läuft stabil, es wird
von großen Organisationen im Alltag eingesetzt, und Sicherheitslücken werden
schnell geschlossen. Eine **große, aktive Entwicklergemeinschaft** ist wichtig,
damit das Programm auch in zehn Jahren noch gepflegt wird.

#### 7. Zugang für KI-Werkzeuge (Schnittstelle / REST-API)

Damit KI-Programme wie **Claude Code**, **Codex CLI**, **Gemini CLI** oder
**GitHub Copilot** beim Schreiben helfen können, müssen sie an die Texte
herankommen. Am besten geht das über

- eine **Schnittstelle** (englisch *API*, meist *REST-API* oder *GraphQL*) – ein
  standardisierter Weg, über den andere Programme Seiten lesen und ändern dürfen,
  oder
- **einfache Textdateien im Ordner**, die ein KI-Werkzeug direkt öffnen kann (so
  arbeitet dieses Wiki heute).

### Sonderfall: LLM-Wiki / Co-Wiki

Es gibt eine neue Gruppe von Wissenssystemen, die von Anfang an für die
Zusammenarbeit mit KI gedacht sind. Man nennt sie **LLM-Wiki** oder **Co-Wiki**
(*LLM* = großes Sprachmodell, die Technik hinter Chat-KI). Kennzeichen:

- Die Inhalte sind kleine, klar benannte Markdown-Dateien – gut für Menschen
  **und** für die KI lesbar. Oft tragen sie ein paar **Kopfzeilen** (auch
  *Frontmatter* genannt: einige Zeilen am Dateianfang mit Titel,
  Kurzbeschreibung und Schlagwörtern).
- Eine Technik namens **RAG** (etwa: „Nachschlagen vor dem Antworten") lässt die
  KI erst die echten Notizen durchsuchen und dann daraus schreiben, statt sich
  etwas auszudenken.
- Dieses Wiki folgt diesem Modell: Markdown-Dateien, KI-lesbar, mit menschlicher
  Redaktion. Nur die technischen Dokumentationsseiten im Ordner `openwiki/`
  tragen ausführliche Kopfzeilen, die Ahrensburg-Artikel selbst nicht. Details in
  Co-Wiki: Mensch & KI.

### Was ist mit „Vibe Coding"?

*Vibe Coding* ist ein Begriff aus der Softwareentwicklung: Man beschreibt einer
KI in normaler Sprache, was man haben will, und lässt sie den Code schreiben,
ohne jede Zeile selbst zu prüfen. Übertragen auf das Schreiben von Texten heißt
das: die KI formuliert, der Mensch nickt nur ab. Für schnelle Versuche ist das
praktisch. Für ein Wissensprojekt, das
richtig sein muss, ist es **allein nicht genug**: Jeder von der KI geschriebene
Text muss von einem Menschen auf Fakten geprüft und freigegeben werden
(*Human-in-the-Loop*, „der Mensch bleibt in der Schleife"). Die KI darf also
kräftig zuarbeiten – die letzte Entscheidung trifft immer ein Mensch.

### Kurzfazit

- **Reine Doku-Generatoren** (wie heute mdBook) sind einfach und robust, stoßen
  aber bei zehntausenden Seiten und vielen gleichzeitigen Autoren an Grenzen.
- **Wiki-Programme** sind für genau diesen Zweck gebaut. Für dieses Projekt sind
  vor allem **XWiki**, **MediaWiki** und **Wiki.js** interessant, weil sie viele
  Seiten, eine Schnittstelle und einen Export in ein offenes, datenbank-neutrales
  Format bieten (XML bei MediaWiki und XWiki, Markdown-Dateien per Git bei
  Wiki.js).
- **CMS** sind eher für klassische Websites gedacht; datei-basierte Flat-File-CMS
  wie **Grav** liegen nah am heutigen Aufbau.
- Egal welches Programm: Ausschlaggebend bleiben zwei Dinge. Erstens ein
  **Export in ein offenes, datenbank-neutrales Format**, dessen Inhalte keine
  E-Mail-Adressen oder Passwörter enthalten. Zweitens eine **große, aktive
  Entwicklergemeinschaft**.

### Fachwörter kurz erklärt

Diese Begriffe tauchen auf dieser Seite immer wieder auf.

| Wort | Was es bedeutet |
|---|---|
| **Open Source** | Der Bauplan (Quelltext) des Programms ist öffentlich; jeder darf es gratis nutzen, ändern und weitergeben. |
| **Schnittstelle** (englisch *API*) | Ein fester Weg, über den ein Programm einem anderen Programm Daten gibt oder abnimmt. |
| **REST-API** | Die heute verbreitetste Art solcher Schnittstellen: Andere Programme rufen über Web-Adressen einzelne Seiten ab oder ändern sie. |
| **GraphQL** | Eine neuere Schnittstellen-Art; das fragende Programm sagt genau, welche Angaben es haben will, und bekommt nur die. |
| **Datenbank** | Ein Programm, das große Mengen Daten sortiert speichert und schnell durchsuchbar hält. **PostgreSQL** ist eine bekannte, kostenlose Datenbank. |
| **XML / JSON / YAML** | Drei verbreitete Textformate für Daten. XML nutzt spitze Klammern (Tags), JSON und YAML sind kompakter; alle sind für Menschen und Programme lesbar. |
| **Markdown** | Ein einfaches Notizformat: normaler Text mit ein paar Zeichen für Überschriften, Fettschrift und Listen. |
| **Git** | Eine „Zeitmaschine" für Dateien: Sie merkt sich jede gespeicherte Änderung, man kann jederzeit zurück. |
| **Node.js** | Eine Umgebung, um JavaScript-Programme außerhalb des Webbrowsers zu nutzen – auf Servern oder als Werkzeuge. |
| **MCP** (Model Context Protocol) | Ein einheitlicher, geregelter Weg, über den KI-Programme auf Dateien und Werkzeuge zugreifen. |
| **RAG** | „Erst nachschlagen, dann antworten": Die KI durchsucht echte Notizen, bevor sie einen Text schreibt. |

### Quellen

- [WikiMatrix – Vergleich von Wiki-Programmen](https://www.wikimatrix.org/compare/bookstack+wiki-js+dokuwiki+mediawiki+xwiki+wackowiki)
- [Wiki.js – Git-Speicher (Dokumentation)](https://docs.requarks.io/storage/git)
- [MediaWiki – Handbuch: PostgreSQL](https://www.mediawiki.org/wiki/Manual:PostgreSQL)
- [MediaWiki – Hilfe: Export](https://www.mediawiki.org/wiki/Help:Export)
- [XWiki.org – Lizenz](https://www.xwiki.org/xwiki/bin/view/Main/License)
- [BookStack – Inhalte exportieren und importieren](https://www.bookstackapp.com/docs/user/export-import/)

## Weiterführend

- LLM Wiki (Karpathy) – das ursprüngliche Muster als Gist:
  <https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f>
- Google Code Wiki – gehostete Gemini-Umsetzung: <https://codewiki.google>
- Open Knowledge Format (OKF) – herstellerneutraler Markdown-Standard von Google Cloud:
  <https://github.com/GoogleCloudPlatform/open-knowledge-format>
- llms.txt (Jeremy Howard) – nach außen gerichtetes Gegenstück: <https://llmstxt.org/>
- DeepWiki-Open – Open-Source-Wiki-Generator für Git-Repositories:
  <https://github.com/AsyncFuncAI/deepwiki-open>
- DeepWiki (gehosteter Dienst, Cognition/Devin): <https://deepwiki.com>
- GitBook – gehostete Doku-Plattform mit KI-Suche und Repo-Sync: <https://gitbook.com>
- Mintlify – gehostete Doku-Plattform mit KI-Q&A und Auto-Entwürfen aus dem Code: <https://mintlify.com>
- Swimm – an Code-Zeilen gekoppelte Doku: <https://swimm.io>
- Repomix – Repository in eine KI-taugliche Datei bündeln: <https://github.com/yamadashy/repomix>
- Gitingest – Repository als Prompt-Text extrahieren: <https://github.com/cyclotruc/gitingest>
