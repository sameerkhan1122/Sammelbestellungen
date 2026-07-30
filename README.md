# Sammelbestellungen – gemeinsame Web-Version

Diese Version speichert alle Daten **zentral auf einem Server**. Jeder, der den
Link öffnet, sieht denselben Stand. Änderungen (neue Produkte, Preise, Personen,
Versandkosten) werden sofort auf dem Server gespeichert und bei allen anderen
offenen Browsern innerhalb weniger Sekunden automatisch übernommen (Polling
alle 3 Sekunden – kein manuelles Neuladen nötig).

## Deployment auf Render.com (empfohlen, kostenlos möglich)

1. Gehe auf https://render.com und erstelle einen kostenlosen Account.
2. Lade dieses Projekt zu einem GitHub-Repository hoch (oder verbinde Render
   direkt mit einem neuen leeren Repo und lade die Dateien dort hoch).
3. In Render: **New +** → **Web Service** → dein Repository auswählen.
4. Einstellungen:
   - **Build Command:** `npm install`
   - **Start Command:** `npm start`
   - **Instance Type:** Free reicht zum Ausprobieren
5. Damit die Daten dauerhaft gespeichert bleiben (auch nach Neustarts/Deploys):
   - Unter **Disks** einen kostenlosen Persistent Disk anlegen, z. B. 1 GB.
   - **Mount Path:** `/opt/render/project/src/data`
   - Umgebungsvariable setzen: `DATA_DIR=/opt/render/project/src/data`
   - (Auf dem kostenlosen Plan ohne Disk gehen die Daten bei jedem Neu-Deploy
     verloren, funktionieren zwischendurch aber ganz normal.)
6. Nach dem Deploy bekommst du eine URL wie
   `https://sammelbestellung-xyz.onrender.com` – diesen Link kannst du an alle
   Beteiligten schicken.

## Alternative: Railway.app

Funktioniert nach demselben Prinzip (GitHub-Repo verbinden, `npm start`,
Volume für `/data` mounten und `DATA_DIR=/data` setzen).

## Lokal testen

```bash
npm install
npm start
```

Dann im Browser `http://localhost:3000` öffnen.

## Wie es technisch funktioniert

- `server.js`: kleiner Express-Server mit zwei Endpunkten:
  - `GET /api/state` – liefert den aktuellen gemeinsamen Zustand + Versionsnummer
  - `PUT /api/state` – speichert einen neuen Zustand und erhöht die Version
- Der Zustand liegt als JSON-Datei unter `data/state.json` (Pfad über
  `DATA_DIR` konfigurierbar).
- `public/app.js` wurde so angepasst, dass sie beim Start den Zustand vom
  Server lädt, jede Änderung dorthin zurückschreibt (leicht verzögert/
  gebündelt, damit z. B. beim Tippen nicht bei jedem Buchstaben ein Request
  rausgeht) und alle 3 Sekunden prüft, ob sich der Zustand serverseitig
  geändert hat (z. B. weil jemand anderes etwas hinzugefügt hat).
- Läuft dein Gerät offline oder der Server ist kurz nicht erreichbar, wird
  weiterhin lokal in `localStorage` zwischengespeichert, damit nichts verloren
  geht.

## Einschränkungen dieser einfachen Lösung

- Es gibt **keine Konfliktlösung**: Wenn zwei Personen exakt gleichzeitig
  etwas ändern, gewinnt einfach der zuletzt gespeicherte Stand ("last write
  wins"). Für eine kleine Gruppe, die nacheinander Produkte einträgt, ist das
  in der Praxis kein Problem.
- Es gibt keinen Login/keine Berechtigungen – jeder mit dem Link kann alles
  sehen und bearbeiten. Das passt zum ursprünglichen Zweck (Sammelbestellung
  im Freundeskreis), sollte aber nicht für sensible Daten genutzt werden.
