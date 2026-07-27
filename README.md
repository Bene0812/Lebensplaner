# Lebensplaner

Eine persönliche, zentrale Lebensplaner-Web-App als Single-File-HTML-App (kein Framework, kein Build-Schritt). Läuft im Browser auf Windows und mobil, Daten werden geräteübergreifend über ein privates GitHub-Repository synchronisiert.

## Architektur

- **Frontend**: `index.html` enthält die komplette App (HTML/CSS/JS in einer Datei).
- **Hosting**: Statisch über GitHub Pages – die App ist von jedem Gerät per URL erreichbar.
- **Datenspeicherung & Sync**: Kein eigenes Backend. Alle Daten liegen als JSON-Dateien in einem **privaten** GitHub-Repository deiner Wahl und werden direkt über die GitHub REST Contents API gelesen/geschrieben.
- **Auth**: Ein GitHub Personal Access Token (PAT) mit minimalen Rechten (nur Contents Read/Write auf das eine Daten-Repo). Der Token wird einmalig pro Gerät eingegeben und ausschließlich lokal im `localStorage` des Browsers gespeichert – nie im Code oder im Repo selbst.
- **Sync-Verhalten**: Beim Öffnen der App werden alle Modul-JSON-Dateien vom konfigurierten Repo geladen. Bei jeder Änderung wird das betroffene Modul (debounced, ~900ms) automatisch als neuer Commit zurückgeschrieben. Konfliktverhalten: "letzter Schreibvorgang gewinnt" – bei einem veralteten `sha` (z. B. weil von einem anderen Gerät zwischenzeitlich geschrieben wurde) holt die App automatisch den aktuellen `sha` und überschreibt damit erneut.

## Entscheidungen zu den offenen Punkten

**1. Struktur der JSON-Dateien: eine Datei pro Modul.**
Statt einer großen Datei gibt es `data/goals.json`, `data/calendar.json`, `data/tasks.json`, `data/habits.json`, `data/finances.json`, `data/journal.json`, `data/learning.json`. Vorteile: kleinere, nachvollziehbare Commits pro Änderung, geringeres Risiko sich gegenseitig überschreibender Schreibvorgänge zwischen Modulen, und jedes Modul kann unabhängig debounced gespeichert werden, ohne dass ein Tippen im Journal einen gleichzeitigen Kalender-Edit blockiert.

**2. UI-Struktur: Sidebar (Desktop) / Bottom-Tab-Bar (Mobile).**
Eine feste Sidebar links mit allen Modulen für Desktop-Nutzung, die bei schmalen Viewports (< 760px) zu einer unteren Tab-Leiste wird – das ist auf dem Handy der bekannteste und daumenfreundlichste Navigationsstil. Navigation erfolgt über Hash-Routing (`#/dashboard`, `#/goals`, …), sodass die App eine echte Single-Page-App ohne Reload bleibt.

**3. PAT-Eingabe-Flow beim ersten Start.**
Beim allerersten Öffnen (kein gespeicherter Zustand in `localStorage`) zeigt die App einen Setup-Screen mit Feldern für GitHub-Benutzer/Organisation, Repository, Branch (Default `main`) und Token. Beim Absenden wird die Verbindung sofort getestet (`GET /repos/{owner}/{repo}`), Fehler (ungültiger Token, Repo nicht gefunden) werden direkt angezeigt. Erst nach erfolgreichem Test wird die Konfiguration in `localStorage` gespeichert und die Daten geladen. Der Token kann später jederzeit über **Einstellungen → Verbindung trennen** entfernt werden.

## Ersteinrichtung

1. **Privates Daten-Repository anlegen** auf GitHub (z. B. `lebensplaner-daten`) – separat vom App-Repo, damit die App selbst öffentlich über GitHub Pages gehostet werden kann, während deine persönlichen Daten privat bleiben.
2. **Fine-grained Personal Access Token erstellen**: `github.com/settings/personal-access-tokens/new`
   - Resource owner: dein Account
   - Repository access: *Only select repositories* → das Daten-Repo auswählen
   - Permissions → *Repository permissions* → **Contents: Read and write**
3. **GitHub Pages aktivieren** für dieses App-Repository (Settings → Pages → Deploy from branch), damit `index.html` unter einer festen URL erreichbar ist.
4. App öffnen, im Setup-Screen Benutzer/Repo/Branch/Token eintragen und auf **„Verbinden & testen“** klicken.

Die Datendateien (`data/*.json`) werden beim ersten Speichern automatisch im Daten-Repo angelegt – es muss vorher nichts manuell erstellt werden.

## Module

| Modul | Datei | Inhalt |
|---|---|---|
| Dashboard | – | Aggregierter Tagesüberblick über alle Module |
| Ziele & Vision | `data/goals.json` | Ziele (1/5/10 Jahre), Lebensbereiche |
| Kalender & Zeitplanung | `data/calendar.json` | Termine, Deadlines, wiederkehrende Events (Monatsansicht + Agenda) |
| Aufgaben & Projekte | `data/tasks.json` | To-dos mit Priorität, Fälligkeit, Projekten |
| Gewohnheiten & Routinen | `data/habits.json` | Habit-Tracker mit Streaks (täglich) bzw. Wochenzielen |
| Finanzen | `data/finances.json` | Budgets, Ausgaben, Sparziele mit Fortschrittsbalken |
| Journal & Reflexion | `data/journal.json` | Tages-/Wochen-/Monatsreview-Einträge mit Stimmung |
| Lernen & Wachstum | `data/learning.json` | Bücherliste, Kurse mit Fortschritt, Skill-Ziele |

Bewusst weggelassen: Gesundheit/Fitness-Tracking, Beziehungen/Soziales.

## Sicherheitshinweis

Der PAT liegt im Klartext im `localStorage` des Browsers. Nutze die App nur auf vertrauenswürdigen Geräten und beschränke den Token strikt auf das eine Daten-Repository mit minimalen Rechten (nur Contents Read/Write). Über **Einstellungen → Verbindung trennen** wird der Token wieder aus dem Browser entfernt.
