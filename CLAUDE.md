# Nochmal? – Projektregeln

Persönliche Web-App: Lieferdienst-Bestellungen nach der Lieferung bewerten (🟢 wieder / 🟡 okay / 🔴 nie wieder), durchsuchen und mit einer zweiten Person per Datei teilen. Hauptgerät: iPhone, abends auf dem Sofa. Zweitgerät: PC.

Live: https://ovestridson.github.io/Nochmal/

## Harte Regeln
- **Eine Datei:** alles in `index.html` (HTML, CSS, JS). Kein Build, kein npm, kein Framework, keine Bibliothek, kein CDN.
- **Zero-API-Key, kein Server, kein Tracking.** Muss als statische Seite auf GitHub Pages und per `python3 -m http.server` laufen.
- **Keine externen Fonts.** Schrift: System-Stack (`ui-rounded`, `system-ui` …).
- **Keine Marken:** keine Lieferando-Logos, kein Lieferando-Orange, keine echten Logos. Icons sind eigene SVGs.
- **UI-Texte auf Deutsch.**
- Nutzertext immer mit `esc()` in `innerHTML` einsetzen.

## Speicher
- Nur `localStorage`, Präfix **`nochmal_`** (Domain wird mit GameRegal geteilt!). Keys: `nochmal_v1` (Daten), `nochmal_user`, `nochmal_anim`, `nochmal_accent`.
- Lesen/Schreiben immer über `store.get/set` (try/catch).

## Datenmodell
Ein Eintrag = eine Bewertung:
`{ id, restaurant, cuisine, dish, price|null, verdict:'good'|'ok'|'bad', note, photo|null, author, createdAt, updatedAt, deleted, demo? }`
- `photo`: JPEG-Data-URL, beim Hinzufügen per `shrinkPhoto()` auf max. 640 px / ca. 60 KB verkleinert (liegt im localStorage – Platz ist knapp!). Passt es nicht mehr, wird ohne Foto gespeichert. Beim Löschen wird das Foto entfernt.
- Küchen (`CUISINES`): Pizza, Pasta, Burger, Döner, Asiatisch, Sushi, Indisch, Mexikanisch, Bowl & Salat, Süßes, Sonstiges – jede mit eigenem SVG-Icon + Poke-Animation.
- `id` über `uuid()` (hat Fallback, weil `crypto.randomUUID` über http fehlt).
- Jede Änderung setzt `updatedAt` neu.
- **Nie hart löschen**, sondern `deleted:true` (Lösch-Markierung, sonst holt der Import den Eintrag zurück). Ausnahme: Demo-Daten.
- Restaurants und Gerichte werden abgeleitet, nicht gespeichert: gruppiert über `norm()` (klein, ohne Akzente, ß→ss).
- Gericht-Status: letzte Bewertung je Autor; sind die Urteile uneins → `mixed` („Geschmackssache“).
- Bearbeiten/Löschen nur für eigene Einträge (`author === user`).
- `demo:true`: wird nie exportiert, geteilt oder importiert; „Demo-Daten entfernen“ löscht nur diese.

## Teilen & Import
- Format (`.txt`): lesbare Kopfzeilen, dann `\n---\n`, dann JSON `{ app:'nochmal', v:2, from, exportedAt, entries }` (v2: Fotos werden mitgeschickt).
- Senden über `navigator.share({files})`, sonst Download. Gedacht für Telegram.
- Import = Zusammenführen über `id`: neu → hinzufügen, bekannt → neueres `updatedAt` gewinnt. Nichts doppelt.
- Formatänderungen nur abwärtskompatibel (`v` hochzählen, alte Dateien weiter lesbar).
- Bekannte iOS-Grenze: Homescreen-Web-Apps haben eigenen Speicher, Links aus Telegram öffnen nicht dorthin. Darum Datei statt Link.

## Design
- **Nur Abendmodus (dunkel).** Farben ausschließlich über CSS-Variablen.
- Akzentfarbe ist wählbar: `applyTheme()` leitet `--accent`, `--accent-rgb`, `--accent-ink`, `--bg`, `--surface*`, `--line` daraus ab. **Nie eine Akzentfarbe hart codieren**, sondern `var(--accent)` bzw. `rgba(var(--accent-rgb), x)`.
- Ampel `--good/--ok/--bad` ist fix und reserviert. Immer Farbe **plus** Symbol (✓ ~ ✕).
- Mobile first: Eingabefelder mindestens **16px** (sonst zoomt iOS), `touch-action: manipulation`, Tippflächen ≥ 44px, `env(safe-area-inset-*)`, keine horizontale Scrollleiste, max. Breite 680px.
- Niemals `user-scalable=no` / `maximum-scale`.

## Animationen
- Nur CSS-Keyframes, Neustart über `restart(el, cls)`.
- UI-Feedback kurz (150–300 ms), darf nie das Tippen blockieren. Charakter-Animationen (Icon-„Poke“, Stempel, Lieferroller) dürfen länger.
- Alles respektiert den Schalter (`animOn()`, Klasse `.no-anim`) und `prefers-reduced-motion`.
- View Transitions nur über `go()`, mit Fallback ohne Animation.

## Code-Aufbau
- Abschnitte im Script sind mit `/* ===== Name ===== */` markiert. Erst den passenden Abschnitt lesen, nicht die ganze Datei.
- Klicks laufen über Event-Delegation mit `data-act="…"` (ein `switch`). Neue Aktionen dort ergänzen.
- Ansichten: `renderHome` / `renderList` / `renderDetail`; Sheets: `openSheet` / `closeSheet`.

## Arbeitsweise
- Kleine, gezielte Änderungen statt Datei neu schreiben.
- Nach Änderungen kurz in 390px Breite prüfen (Handy).
- Commit-Nachrichten kurz, auf Deutsch.

## Ideen (noch nicht gebaut)
- Hash-Link als Abkürzung fürs Teilen einzelner Einträge (gleiches Datenformat).
