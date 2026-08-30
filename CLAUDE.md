# CLAUDE.md

Diese Datei gibt Claude Code (claude.ai/code) Anleitung für das Projekt `MA-HTML-Player`.

## Projekt

Ein selbst gehosteter HTML-Dashboard-Controller für [Music Assistant](https://www.music-assistant.io/) (MA). Die Datei `ma-dashboard.html` läuft direkt im Browser – auch auf alten iPads mit iOS 9 / Safari 9 – und spricht direkt mit der MA-WebSocket-API. Sie ersetzt eine frühere Fassung, die über Home Assistant REST-Services medierte und nur sehr eingeschränkte Funktionen bot.

## Zweck und Funktionsumfang

Die App ist als App-artiges Layout mit vier Seiten aufgebaut, zwischen denen eine Fußzeile mit vier reinen Icon-Buttons (ohne Textbeschriftung) umschaltet: **Now Playing**, **Radio**, **Bibliothek**, **Suche** (siehe „Seiten-Router (Fußnavigation)“).

- **Direkte MA-Steuerung** über WebSocket (`/ws`) mit Long-Lived-Token-Authentifizierung.
- **Player-Auswahl** als stationsartige Leiste oben in der Kopfzeile (alle oder eine konfigurierbare Whitelist). Ein Klick auf die bereits aktive Station öffnet ein Gruppierungs-Menü: aktuelle Gruppenmitglieder sind dort entfernbar, weitere gruppierfähige Player können hinzugefügt werden.
- **„Quick Actions“-Menü**: Der Button neben der Stationsleiste trägt jetzt den sichtbaren Text „Quick Actions ❯“ (zuvor nur der Pfeil) und öffnet dasselbe Dropdown wie zuvor (Liste „Alle Player“), ergänzt um einen Abschnitt „Quick Actions“ mit zwei quadratischen Buttons – „Play Music“ und „Sleep“ (siehe „Quick Actions“).
- **Seitentitel**: Jede der vier Seiten zeigt oben eine Überschrift (`.page-title`) – „Radio“, „Bibliothek“ bzw. „Suche“ statisch, auf der Now-Playing-Seite dynamisch „Now Playing“ bzw. bei laufender Wiedergabe „Now Playing - {Quelle} - {Titel}“ (siehe „Seiten-Router (Fußnavigation)“ und „Now Playing: Vollbild und Farbverlauf“).
- **Now-Playing-Seite** (Fußzeile: erstes Icon): vollflächige, zentrierte Ansicht mit großem Cover, Titel, Interpret, Album, Fortschrittsbalken, Transport, Shuffle/Repeat/Automix und Lautstärke. Der Hintergrund der Seite färbt sich, wenn möglich, in aus dem Cover extrahierten Farben ein (siehe „Now Playing: Vollbild und Farbverlauf“). Ein viertes Icon neben Favorit/Zur-Playlist/Songtexte öffnet die Warteschlange als Slide-in-Panel von rechts (siehe „Warteschlange: gemeinsames Slide-in-Panel“). Bei Gruppen-/Sync-Playern lässt sich ein aufklappbares Menü öffnen, um die Lautstärke jedes einzelnen Gruppenmitglieds zu regeln. Das Cover blendet bei Titelwechsel über (Crossfade), statt hart ausgetauscht zu werden (siehe „Cover-Crossfade und Queue-Slide“).
- **Radio-Seite** (Fußzeile: zweites Icon): Kacheln der favorisierten Radiosender, nach Land gruppiert (best-effort plus eine kleine manuelle Zuordnungsliste für Sender ohne Länder-Metadaten, siehe „Radio“), mit dem Sender-Artwork als Kachelhintergrund. Antippen spielt den Sender sofort ab und wechselt zur Now-Playing-Seite.
- **Bibliothek-Seite** (Fußzeile: drittes Icon): dasselbe zweispaltige Layout wie zuvor, jetzt mit deutlich schmaleren, kompakten Wiedergabe-Steuerelementen links (siehe „Seiten-Router (Fußnavigation)“), damit rechts mehr Platz für die Liste bleibt. Schneller Zugriff auf gespeicherte Playlists, Interpreten, Alben, Titel, Radio, Podcasts, Hörbücher – inklusive Favoriten-Herz-Anzeige. Toggle-Chip „Nur Favoriten“, Chips „Name“/„Jahr“ zum Umsortieren sowie ein Music-Source-Filter (siehe „Music-Source-Filter“). Ein von den Filter-Chips abgesetzter Button öffnet die Warteschlange als eigenes Slide-in-Panel (siehe „Warteschlange: gemeinsames Slide-in-Panel“).
- **Such-Seite** (Fußzeile: viertes Icon): dasselbe zweispaltige Layout mit denselben kompakten Steuerelementen links, rechts die Suche über Titel, Alben, Interpreten, Playlists, Radio, Podcasts, Hörbücher, ebenfalls mit Music-Source-Filter.
- **Einstellungen**: Zahnrad-Button in der Kopfzeile neben Theme-/Akzentfarben-Umschalter öffnet ein Panel mit bislang einer Einstellung – einer Standard-Musikquelle für Bibliothek und Suche (siehe „Einstellungen“).
- **Navigation** innerhalb von Bibliothek/Suche: Interpret → Alben + Top-Titel, Album → Titel, Playlist → Titel, Podcast → Folgen. Wird eine Detailansicht aus der Bibliothek oder Suche heraus geöffnet, bleibt der ursprüngliche Fußzeilen-Eintrag optisch aktiv markiert; „Zurück“ führt beim Verlassen der Detailansicht dorthin zurück (siehe „Seiten-Router (Fußnavigation)“).
- **Detail-Tabs mit Lazy Loading** in der Interpretansicht: „Top-Titel“ (echte Top-/Featured-Titel, `music/artists/top_tracks`), „Titel“ (vollständige Titelliste, `music/artists/artist_tracks`) und „Alben“ werden erst beim Öffnen des Tabs geladen. Bei Interpreten aus der Bibliothek wird dafür automatisch der echte Streaming-Provider aus `provider_mappings` verwendet statt des Pseudo-Providers `library`, da Letzterer serverseitig nur bibliotheksverknüpfte Einträge liefert (siehe Abschnitt „Bibliothekselemente und Provider-Auflösung“).
- **Music-Source-Filter** in Bibliothek und Suche: zusätzliche Chip-Zeile, die nur Treffer eines bestimmten Streaming-Providers (Tidal, YouTube Music, SoundCloud, …) zeigt, bereits beim Start befüllt statt erst nach der ersten Suche (siehe „Music-Source-Filter“).
- **Filter/Sortierung/Quelle getrennt**: In Bibliothek und Suche stehen die Medientyp-Filter-Chips, die Sortier-Chips und die Music-Source-Chips in drei optisch getrennten Abschnitten (dünne Trennlinie plus kleines Label „Filter“/„Sort“/„Source“ pro Abschnitt), nicht mehr in einer Reihe gemischt (siehe „Music-Source-Filter“).
- **Sortierung**: Bibliothek, Suche sowie die Tabs „Titel“ und „Alben“ der Interpretenansicht bieten Chips zum Umschalten der Sortierung ("Name"/"Jahr" bzw. in der Suche zusätzlich "Relevanz" als Default). Sortiert wird clientseitig über `sortMediaItems()`. Der Tab „Top-Titel“ ist bewusst nicht sortierbar, da seine Reihenfolge die Popularitäts-Rangfolge des Servers ist; ebenso bleiben Track-Nummern/Playlist-Reihenfolgen in anderen Detailansichten unverändert.
- **Aktionsblatt** pro Medium: Jetzt abspielen, Als Nächstes, Hinten anhängen, Zur Playlist hinzufügen, Favorit setzen/entfernen, Öffnen/Interpret/Album, Metadaten ansehen (siehe „Metadaten ansehen“).
- **Add to Playlist** sowohl aus dem Aktionsblatt als auch vom Now-Playing-Cover aus. Im Sheet werden favorisierte Playlists zuerst angezeigt, danach alle anderen alphabetisch nach Name sortiert.
- **Automix / Smart Crossfade** ein/aus schalten; bei laufender Wiedergabe wird dazu kurz pausiert und fortgesetzt.
- **Warteschlange** anzeigen, direkten Titel anwählen, Einzelpositionen löschen, Queue leeren – als gemeinsames Slide-in-Panel, erreichbar sowohl von der Bibliothek- als auch von der Now-Playing-Seite, nicht als eigener Fußzeilen-Eintrag (siehe „Warteschlange: gemeinsames Slide-in-Panel“).
- **Favoriten-Herz** sowohl im Now-Playing-Bereich als auch in jeder Detailansicht.
- **Dark/Light Mode**: folgt standardmäßig der Systemeinstellung (`prefers-color-scheme`), umschaltbar über einen Button in der Kopfzeile; die manuelle Wahl wird in `localStorage` gemerkt und übersteuert das System dauerhaft (siehe „Dark/Light Mode“).
- **Akzentfarbe**: über einen Farbkreis in der Kopfzeile aus einer festen Palette wählbar (Standard bleibt Gelb/Amber), Wahl wird in `localStorage` gemerkt (siehe „Akzentfarbe“).
- **Songtexte**: über einen Button im Now-Playing-Bereich einblendbar; ersetzt dabei den Cover-/Steuerungsblock auf der Now-Playing-Seite. Synchronisierte Anzeige mit hervorgehobener aktueller Zeile, wenn MA `metadata.lrc_lyrics` liefert, sonst Klartext-Fallback über `metadata.lyrics` (siehe „Songtexte“).
- **Sprache**: nur Englisch. Es gibt keinen DE/EN-Umschalter mehr; alle Texte kommen aus einem einzigen `STRINGS`-Wörterbuch.
- **Schriften**: Roboto Mono (Titel, Marken-Schriftzug, Beschriftungen/Chips, Zahlen) und Roboto (Fließtext) statt der reinen iOS-Systemschriften – siehe „ES5-/iPad-Kompatibilität“ und `fonts/`.

## Wichtige Dateien

- `ma-dashboard.html` – komplette Single-Page-App (HTML, CSS, JS). Keine Build-Schritt, keine externen Abhängigkeiten.
- `fonts/Roboto-Regular.woff2`, `fonts/RobotoMono-Regular.woff2` – lokal gebündelte Schriftdateien (Latin-Subset), per relativem `@font-face`-`url()` aus `ma-dashboard.html` eingebunden. Müssen beim Deployment neben `ma-dashboard.html` im Unterordner `fonts/` liegen; kein Laden von einem Font-CDN zur Laufzeit, damit die App weiterhin vollständig offline-fähig bleibt.
- `AppIcon.png` – App-Icon für Homescreen-Webclip (`apple-touch-icon`) und Browser-Favicon (`icon`), referenziert per relativem Pfad im `<head>`; muss beim Deployment neben `ma-dashboard.html` liegen (Groß-/Kleinschreibung beachten, falls der Zielserver case-sensitiv ist).
- `CLAUDE.md` – diese Datei.

## Architektur

### Warum WebSocket statt HTTP JSON-RPC?

MA bietet sowohl den WebSocket-Endpunkt `/ws` als auch eine POST-Route `/api` für JSON-RPC. Die POST-Route sendet **keine CORS-Header**, deshalb scheitern XHR/Fetch aus einer anderen Herkunft (z. B. `file://` auf einem iPad oder einem beliebigen lokalen Webserver) am Preflight. WebSocket-Handshakes unterliegen der Same-Origin-Policy nicht, daher funktioniert die Datei von überall.

### Kommunikationsablauf

1. `connect()` öffnet `ws://<host>/ws` bzw. `wss://<host>/ws`.
2. Als erstes Kommando muss `auth` mit `token` gesendet werden (`send('auth', { token: CFG.TOKEN }, …)`).
3. Nach erfolgreicher Authentifizierung werden initial alle Player und Queues geladen (`bootstrap`).
4. Alle weiteren Zustandsänderungen kommen als Server-Events herein (`player_updated`, `queue_updated`, `queue_time_updated`, `queue_items_updated`, `media_item_updated`).

### Partial-Response-Handling

Lange Listen (z. B. `players/all`, `player_queues/items`, Suchergebnisse) können in Blöcken zu bis zu 500 Einträgen mit `partial: true` eintreffen. Erst die letzte, nicht-partial Nachricht schließt die Antwort ab. Der Client sammelt in `pending[id].chunks` und liefert dann das zusammengefügte Array an den Callback.

### ES5-/iPad-Kompatibilität

- Kein CSS-Grid, keine `@supports`. CSS Custom Properties (Variablen) sind die eine bewusste Ausnahme – siehe „Dark/Light Mode“ – weil Safari sie ab 9.1 (iOS Safari 9.3) unterstützt, was ohnehin die Mindestanforderung des Projekts ist (siehe „Troubleshooting“).
- `display: -webkit-flex` und `display: flex` mit Vendor-Prefixes.
- Keine Arrow-Funktionen, kein `fetch`, kein `const`/`let`, kein `Promise`, kein `class`, kein `Object.assign`.
- `XMLHttpRequest` nur für den initialen `/info`-Request (zum Auslesen des `Date`-Headers für die Uhr-Korrektur).
- WebSocket ist in Safari 9 vorhanden.
- Schriften: zwei lokal gebündelte `@font-face`-Dateien (`fonts/Roboto-Regular.woff2`, `fonts/RobotoMono-Regular.woff2`, jeweils nur das Latin-Subset) statt reiner iOS-Systemschriften – `@font-face` selbst wird von Safari bereits lange vor iOS 9 unterstützt, es ist also keine Kollision mit der ES5-/Kompatibilitätsphilosophie. Beide Familien haben Systemschrift-Fallbacks (`"Roboto Mono", "Courier New", monospace` bzw. `"Roboto", "Helvetica Neue", Helvetica, Arial, sans-serif`), falls die Dateien einmal fehlen. Bewusst **kein** Laden von einem Font-CDN zur Laufzeit (Google Fonts o. ä.), weil MA-Setups oft ohne Internetzugang vom iPad aus laufen – die Dateien wurden einmalig heruntergeladen und liegen im Repo, exakt wie `AppIcon.png`.

### Seiten-Router (Fußnavigation)

Die vier Fußzeilen-Icons (`data-tab="now|radio|library|search"`) tragen bewusst dieselbe CSS-Klasse `tab` wie früher die textbasierten Reiter, damit `switchTab(name, pillName)` und die Klick-Verkabelung in `wireTabs()` unverändert wiederverwendet werden können – nur das Aussehen kommt über `.footer .tab` aus einem anderen CSS-Kontext (Icons statt Text, Fußzeile statt Spalte).

`TABS` bildet jeden Fußzeilen-Namen auf eine Pane-ID ab:

```javascript
var TABS = {
  now:     { pane: 'pageNow',       init: syncNowPageState },
  radio:   { pane: 'pageRadio',     init: loadRadioGrid },
  library: { pane: 'pageLibSearch', init: function () { showLibSearchSide('library'); … } },
  search:  { pane: 'pageLibSearch', init: function () { showLibSearchSide('search'); } }
};
```

`library` und `search` teilen sich dieselbe Pane (`#pageLibSearch`, das zweispaltige Layout mit den kompakten Steuerelementen links und der Liste rechts) – nur `showLibSearchSide('library'|'search')` schaltet innerhalb dessen `.right`-Spalte zwischen `#paneLibrary` und `#paneSearch` um. Da zwei `TABS`-Schlüssel auf dieselbe Pane-ID zeigen können, berechnet `switchTab()` die Sichtbarkeit einmal pro eindeutiger Pane-ID (statt sie bei jedem Schlüssel-Durchlauf blind zu überschreiben), sonst würde der zuletzt verarbeitete Schlüssel immer gewinnen.

`enterDetail()`/`navOriginTab`/`popView()` funktionieren unverändert wie zuvor (siehe „Entwicklung und Anpassung“) – Detailansichten laufen technisch immer über `switchTab('search', navOriginTab)`, sodass der ursprüngliche Fußzeilen-Eintrag (typischerweise „Bibliothek“) optisch aktiv bleibt.

Die kompakten Wiedergabe-Steuerelemente existieren **zweimal** im DOM: einmal mit den ursprünglichen IDs (`#cover`, `#title`, `#bPlay`, `#vol`, …) innerhalb von `#pageLibSearch .left`, und einmal mit `Big`-Suffix (`#coverBig`, `#titleBig`, `#bPlayBig`, `#volBig`, …) innerhalb der vollflächigen `#pageNow`-Seite. Damit nicht jede Render-Funktion doppelt geschrieben werden muss, gibt es die Helfer-Familie `npEls(base)`/`npText(base, s)`/`npClass(base, cls)`/`npHtml(base, html)`/`npOn(base, evt, handler)` (definiert direkt vor `crossfadeCover()`): sie iterieren über `NP_SUFFIXES = ['', 'Big']` und schreiben denselben Wert in beide Element-Sätze bzw. verkabeln denselben Handler auf beide. `renderNow()`, `tick()`, `paintAutomix()`, `renderVolumeMembers()`, `paintNowHeart()`, `paintLyricsButton()` und alle Transport-/Lautstärke-Click-Handler nutzen ausschließlich diese Helfer, damit beide Steuerelement-Sätze unabhängig davon synchron bleiben, welche Seite gerade sichtbar ist. Neue Steuerelemente im Now-Playing-Bereich sollten demselben Muster folgen (Element einmal in `.left` und einmal mit `Big`-Suffix in `#pageNow`, Verkabelung über `npOn()`).

Die `.cover`/`.title`/`.btn`/`.toggles`/`.tgl`/`.transport`/`.sub`-Klassen definieren dadurch **drei** Größenstufen über denselben Satz an Basisklassen, rein per Nachfahren-Selektor: die unveränderten Basiswerte (mittelgroß, historisch), `.left .cover { … }` usw. (kompakt, für die zweispaltigen Bibliothek-/Suche-Seiten – schmal genug, damit die Liste rechts mehr Platz bekommt) und `#pageNow .cover { … }` usw. (groß, für die Vollbild-Ansicht). Da `.left` und `#pageNow` nie gleichzeitig denselben Steuerelement-Satz enthalten, gibt es keine Selektor-Konflikte; neue Größenanpassungen sollten ebenfalls über `.left .klasse{…}` bzw. `#pageNow .klasse{…}` erfolgen, nie durch Ändern der Basiswerte selbst.

Jede Seite trägt oben eine `.page-title`-Überschrift: `#pageRadio`/`#pageLibSearch` statisch bzw. über `showLibSearchSide()` gesetzt (`#libSearchPageTitle`, „Bibliothek“/„Suche“ je nach `which`), `#pageNow` dynamisch über `renderNowPageTitle(rawTitle, artist, mi)` (aufgerufen aus `renderNow()`) – baut „Now Playing“ plus optional ` - {Quelle} - {Name}` als zweiten, gedimmten `<span class="pt-dim">`, nie per `innerHTML`-String-Interpolation, da Titel/Interpret/Sendername beliebige, ungefilterte Serverdaten sind. Die Quelle ist `t('media.radio')` bei `mi.media_type === 'radio'`, sonst das erste Ergebnis von `itemProviderDomains(mi)` durch `sourceLabel()` (siehe „Music-Source-Filter“); der Name ist bei Radio nur der Sendername, sonst „{Titel} by {Interpret}“.

Konsistent über alle vier Seiten: `.page-title` ist immer ein eigenes, `flex-shrink: 0`-Geschwisterelement **außerhalb** des scrollbaren/gepolsterten Inhaltsbereichs, nie ein Kind davon – `#pageLibSearch` > `.page-title` + `.libsearch-row`, `#pageRadio` > `.page-title` + `.radio-page-body` (das eigentliche `padding`/`overflow-y:auto` trägt), `#pageNow` > `.page-title` + `.now-content` (trägt ebenfalls `padding`/`overflow-y:auto`). Dadurch bleibt der Titel beim Scrollen immer oben stehen und links ausgerichtet mit demselben `border-bottom`-Trenner. `#pageNow .page-title` bekommt zusätzlich `background: var(--bg-elevated)` (dieselbe Fläche wie die Kopfzeile) – die einzige Seite mit einem `background-image` (dem aus dem Cover extrahierten Farbverlauf, siehe „Now Playing: Vollbild und Farbverlauf“) dahinter, ohne die eigene Fläche würde der Titel darüber je nach Cover-Farben schwer lesbar bzw. „vom Artwork verdeckt“ wirken.

## Konfiguration

Zugangsdaten und gerätespezifische Einstellungen liegen in der separaten Datei `ma-env.js`. Diese wird von `ma-dashboard.html` per `<script src="ma-env.js">` geladen und ist in `.gitignore` ausgeschlossen. Als Vorlage dient `ma-env.example.js`.

```javascript
var CFG = {
  MA: 'http://192.168.1.5:8095',   // MA-Basis-URL, ohne abschließenden /
  TOKEN: '<long-lived-token>',     // MA: Einstellungen > Benutzer > Token
  PLAYERS: [],                     // optionale Whitelist von player_ids
  DEFAULT_PLAYER: '',              // player_id, die beim Start aktiv sein soll; leer = automatisch der gerade spielende Player/Gruppe, sonst der erste
  SEARCH_LIMIT: 12,                // Treffer pro Kategorie
  LIST_LIMIT: 200,                 // Titel/Alben je Detailansicht
  IMG_COVER: 512,                  // Cover-Kantenlänge über imageproxy
  IMG_THUMB: 160                   // Thumbnail-Kantenlänge über imageproxy
};
```

Fehlt `ma-env.js` oder einzelne Werte, greifen Fallback-Defaults im Hauptscript. Ist `CFG.MA` leer, wird die aktuelle `location.host` verwendet (z. B. wenn MA die Datei selbst ausliefert).

## Wichtige WebSocket-Befehle

### Player

- `players/all` – alle Player auflisten.
- `players/cmd/play_pause` – Play/Pause umschalten.
- `players/cmd/next`, `players/cmd/previous` – Titel wechseln.
- `players/cmd/seek` – Position im aktuellen Titel setzen.
- `players/cmd/volume_set`, `players/cmd/group_volume` – Lautstärke; bei Sync-Gruppen ist `volume_level` null, dann `group_volume` verwenden.
- Für einzelne Mitglieder eines Gruppenplayers wird `players/cmd/volume_set` mit der jeweiligen `player_id` des Mitglieds gesendet.
- `players/cmd/group` – Player zu einer Gruppe hinzufügen (Parameter: `player_id` des hinzuzufügenden Players, `target_player` als Gruppenleiter).
- `players/cmd/ungroup` – Player aus jeder Sync-/Gruppenzugehörigkeit entfernen (Parameter: `player_id`).
- Ob ein Player überhaupt gruppierbar ist, zeigt das Feature-Flag `set_members` in `supported_features`.

### Queue

- `player_queues/all` – alle Queues laden.
- `player_queues/get_active_queue` – aktive Queue eines Players.
- `player_queues/items` – Inhalte einer Queue.
- `player_queues/play_media` – Medium abspielen (`option: 'replace'|'next'|'add'`).
- `player_queues/play_index` – zu einem bestimmten Queue-Eintrag springen.
- `player_queues/shuffle`, `player_queues/repeat` – Shuffle/Repeat steuern.
- `player_queues/clear`, `player_queues/delete_item` – Queue leeren / Eintrag entfernen.
- Bei aktivem `shuffle_enabled` oder `radio_mode` zeigt die Queue einen Hinweis-Banner an, dass Titel dynamisch gemischt oder generiert werden.
- Die Anzeige zeigt nur die letzten `QUEUE_HISTORY_LIMIT` (3) bereits gespielten Titel vor dem aktuellen Titel, nicht die komplette Verlaufs-Historie – betrifft nur das Rendering in `loadQueue()`, die Server-Queue bleibt unverändert.
- Bei einem reinen Titelwechsel wird die Liste nicht neu aufgebaut, sondern per Slide-Animation aktualisiert (siehe „Cover-Crossfade und Queue-Slide“).

### Medien

- `music/search` – globale Suche.
- `music/item_by_uri` – ein Medium anhand seiner URI auflösen (liefert Bibliothekselement mit `favorite`-Flag).
- `music/artists/artist_albums`, `music/artists/artist_tracks` – Detailansicht Interpret: vollständige Diskografie bzw. vollständige Titelliste.
- `music/artists/top_tracks` – echte Top-/Featured-Titel eines Interpreten (bei Bibliothekselementen providerübergreifend aggregiert/dedupliziert); nicht zu verwechseln mit `artist_tracks`, das die komplette Titelliste liefert.
- `music/albums/album_tracks` – Detailansicht Album.
- `music/playlists/playlist_tracks` – Detailansicht Playlist.
- `music/podcasts/podcast_episodes` – Detailansicht Podcast.
- `music/<type>s/library_items` – gespeicherte Einträge eines Medientyps auflisten (`type` = `playlists`, `artists`, `albums`, `tracks`, `radios`, `podcasts`, `audiobooks`).
- `music/playlists/create_playlist` – Playlist erstellen (Parameter: `name`).
- `music/playlists/add_playlist_tracks` – Titel zu Playlist hinzufügen (Parameter: `db_playlist_id`, `uris`).
- `music/favorites/add_item` – Favorit hinzufügen (Parameter: `item: uri`).
- `music/favorites/remove_item` – Favorit entfernen (Parameter: `media_type`, `library_item_id`).
- `metadata/get_track_lyrics` – Songtexte für einen Titel synchron und on-demand abrufen (Parameter: `track` = volles Track-Objekt; Rückgabe: `[lyrics, lrc_lyrics]`, siehe „Songtexte“).

### Player-Konfiguration

- `config/players/get_value` – Config-Wert lesen (Parameter: `player_id`, `key`).
- `config/players/save` – Config-Werte schreiben (Parameter: `player_id`, `values`).

Wird **Automix / Smart Crossfade** umgeschaltet (`smart_fades_mode`), muss bei laufender Wiedergabe zuerst pausiert, dann geschaltet und nach ~300 ms wieder fortgesetzt werden.

## Fortschrittsbalken und der Beta-Bug

In aktuellen MA-Beta-Versionen (ab Commit `cadcd96fd`) sendet der Server keine kontinuierlichen Positions-Ticks mehr. Stattdessen liefert er nur noch Anker-Paare:

- `elapsed_time` – Position im Titel in Sekunden.
- `elapsed_time_last_updated` – Server-Unixzeit dieses Ankers.

Wer den Balken nur bei jedem Event neu zeichnet, sieht einen stehenden Balken, der erst bei Pause/Seek/Titelwechsel nachspringt – genau das alte Verhalten der HA-Fassung.

Die korrekte Implementierung im Dashboard:

1. Anker merken (`anchorPos`, `anchorAt`).
2. Bei laufender Wiedergabe lokal interpolieren: `pos = anchorPos + (serverNow() - anchorAt)`.
3. Alle 500 ms `tick()` aufrufen, um den Balken flüssig zu aktualisieren.
4. Uhr-Korrektur gegen den MA-Server mittels HTTP `Date`-Header von `/info`, damit ein falsch gehendes iPad den Balken nicht verschiebt.

## Cover-Crossfade und Queue-Slide

Ein harter `innerHTML`-Austausch bei jedem Titelwechsel wirkt unruhig (Cover blitzt, Warteschlange baut sich sichtbar neu auf). Beides ist stattdessen als weiche Übergangsanimation gelöst, rein mit CSS-Transitions (kein Framework, ES5/iPad-tauglich):

- **Cover** (`renderNow()` → `crossfadeCover()`): Ein neues `<img>`- bzw. Platzhalter-Element wird über das bisherige gelegt (`.cover img`/`.cover .ph` sind `position: absolute` innerhalb des `position: relative`-Containers `.cover`) und per `opacity`-Transition (`.show`-Klasse) eingeblendet. Das alte Element wird erst nach Ablauf der Transition (`setTimeout`, 420 ms) aus dem DOM entfernt.
- **Queue** (`loadQueue()` → `tryQueueSlide()`): Bei jedem Refresh wird geprüft, ob sich seit dem letzten Rendern nur das sichtbare Fenster verschoben hat (`lastQueueIds`/`lastQueueStartIdx`) – d. h. Reihenfolge und Anzahl der Queue-Items sind identisch, nur der aktuelle Titel ist vorgerückt (Normalfall beim Weiterspielen). In diesem Fall wird nicht neu aufgebaut: die `.now`-Markierung wandert zur passenden Zeile, und die oben aus dem Fenster fallenden Zeilen werden per Höhen-/Opacity-Transition weggeslided (`slideRowAway()`, Klasse `.row.q-leave`), statt hart entfernt zu werden. Bei echten Änderungen (Shuffle, Hinzufügen/Löschen, Playerwechsel – `lastQueueIds` passt nicht mehr) greift weiterhin der volle Rebuild über `renderQueueRows()`. Der „Wird geladen …“-Hinweis erscheint nur noch, wenn die Liste aktuell leer ist (`box.getElementsByClassName('row').length === 0`), nicht mehr bei jedem stillen Hintergrund-Refresh.
- Beide Mechanismen nutzen die generischen Klassen-Helfer `hasClass()`/`addClass()`/`removeClass()` (String-basiert, kein `classList`, konsistent mit dem übrigen ES5-Stil der Datei).

## Now Playing: Vollbild und Farbverlauf

Die Now-Playing-Seite (`#pageNow`) ist zentriert und vollflächig; CSS-Klassen (`.cover`, `.titlerow`, `.transport`, `.tgl`, `.vol`, …) sind identisch zu den kompakten Steuerelementen, nur über `#pageNow`-Nachfahren-Regeln vergrößert (z. B. `#pageNow .cover { width: 320px; … }`) – eine einzige Quelle für das Aussehen jedes Steuerelements, siehe „Seiten-Router (Fußnavigation)“.

Beim Laden eines neuen Covers versucht `updateNowGradient(url)` zusätzlich, einen sanften Farbverlauf aus dem Artwork zu extrahieren: Ein **separates** `Image`-Objekt (`crossOrigin = 'anonymous'`) lädt dieselbe URL erneut, zeichnet sie auf einen kleinen Offscreen-`<canvas>` (12×12) und liest zwei Sample-Pixel per `getImageData` aus, um daraus ein zweifarbiges `linear-gradient(...)` mit reduzierter Deckkraft als `#pageNow`-Hintergrund zu setzen (überlagert die flache `--bg`-Fläche, bleibt dadurch in beiden Themes lesbar). Bewusst ein zweites `Image`-Objekt statt desselben, das im Cover angezeigt wird: Setzt der MA-Server auf `/imageproxy`-Antworten keine passenden CORS-Header, bleibt der Canvas „tainted“ und `getImageData()` wirft eine `SecurityError` – das wird in `updateNowGradient()` per `try/catch` abgefangen (Fallback: flacher `--bg`-Hintergrund), **ohne** dass die eigentliche Cover-Anzeige (die weiterhin über das erste, unveränderte `Image`-Objekt läuft) davon betroffen wäre. Der Farbverlauf ist also bewusst ein Best-Effort-Feature, kein hartes Erfordernis.

Die Songtextansicht (siehe „Songtexte“) ersetzt auf dieser Seite den Cover-/Steuerungsblock (`.now-inner`) durch `#lyricsBoxBig`, gesteuert über die CSS-Klasse `#pageNow.lyrics-open`.

## Radio

Die Radio-Seite (`#pageRadio` → `#radioGrid`, gefüllt von `loadRadioGrid()`) zeigt ausschließlich **favorisierte** Radiosender (`music/radios/library_items`, client-seitig auf `favorite === true` gefiltert) als Kachel-Grid (`.radio-tile`), gruppiert nach Land.

- Die Gruppierung ist **best effort**: `radioCountry(it)` prüft zuerst `RADIO_COUNTRY_OVERRIDES` – eine manuell gepflegte Zuordnung von exaktem Sendernamen zu Land, für Sender, deren Provider überhaupt keine Länder-Metadaten liefert (aktuell u. a. „2SER 107.3“, „ABC Jazz HLS“, „TrustFM“, „FBi Radio - Sydney - 94.5 FM (MP3)“ → „Australia“) – und liest sonst `it.metadata.country` bzw. `it.country`, falls vorhanden. MA garantiert kein Länderfeld für jeden Radio-Provider – Sender ohne verwertbaren Wert (auch nicht in der manuellen Liste) landen gesammelt in einer nachgestellten „Other“-Gruppe, statt dass die Gruppierung komplett entfällt. Neue Sender ohne Land: einfach einen weiteren Eintrag zu `RADIO_COUNTRY_OVERRIDES` hinzufügen.
- Jede Kachel (`radioTile()`) nutzt das Sender-Artwork (`imgUrl(it, CFG.IMG_THUMB)`) als `background-image` mit einem dunklen Verlaufs-Overlay (`.rt-shade`) für Lesbarkeit des Namens; ohne Artwork ein einfacher Notenglyph-Platzhalter (`.rt-ph`), konsistent mit dem übrigen Platzhalter-Muster der App.
- Antippen (`playRadioStation()`) spielt den Sender über `player_queues/play_media` (`option: 'replace'`) auf der aktiven Queue ab und wechselt danach per `switchTab('now')` zur Now-Playing-Seite.

## Music-Source-Filter

In Bibliothek und Suche lässt sich die Trefferliste zusätzlich nach Streaming-Quelle filtern (Tidal, YouTube Music, SoundCloud, …), über eine Chip-Zeile (`#libSourceFilters` bzw. `#sourceFilters`, gefüllt von `renderSourceFilters()`) in einem eigenen, per dünner Trennlinie und Label „Source“ abgesetzten Abschnitt (`.filter-section`/`.filter-label`, siehe unten) – ebenso wie die Medientyp-Filter-Chips (Label „Filter“, `#filters`/`#libFilters`) und die Sortier-Chips (Label „Sort“, `#sortFilters`/`#libSortFilters`) je ihren eigenen Abschnitt bekommen, statt wie zuvor alle in einer Reihe gemischt zu sein. In der Bibliothek blendet `buildLibFilters()` die Sort- und Source-Abschnitte (`#libSortSection`/`#libSourceSection`) im Warteschlangen-Modus (`currentLibType === 'queue'`) komplett aus, statt sie leer unter einem Label stehen zu lassen.

- Die Chip-Liste selbst kommt **nicht** aus den gerade geladenen Treffern, sondern aus `availableSourceDomains` – einer beim Start per `refreshAvailableSources()` (aufgerufen aus `bootstrap()`, parallel zu `players/all`/`player_queues/all`, sobald die WebSocket-Authentifizierung durchgelaufen ist) über `loadMusicSources()` geladenen Liste der tatsächlich auf dem Server konfigurierten Musik-Provider (`config/providers`, `provider_type: 'music'`, siehe auch „Einstellungen“). Dadurch sind die Chips schon sichtbar, bevor überhaupt eine Suche gestartet oder die Bibliothek zum ersten Mal geöffnet wurde, und sie bleiben stabil (verschwinden nicht, wenn ein Filter gerade keine Treffer hat).
- `itemProviderDomains(it)` liefert die tatsächlichen Provider-Domains eines einzelnen Items für die **Filterung selbst** (nicht für die Chip-Liste): bevorzugt `it.provider_mappings[].provider_domain` (dedupliziert), da Bibliothekselemente auf oberster Ebene nur `provider === 'library'` tragen (siehe „Bibliothekselemente und Provider-Auflösung“) und die echte Quelle nur in `provider_mappings` steht; Fallback auf `it.provider`, falls das nicht der Pseudo-Provider `library` ist (typischer Fall bei Suchergebnissen, die oft schon einen echten Provider mitbringen).
- `sourceLabel(domain)` übersetzt bekannte Domains in freundlichere Namen (`SOURCE_LABELS`, z. B. `ytmusic` → „YouTube Music“) mit einem Title-Case-Fallback für unbekannte Domains.
- Die Filterung selbst passiert rein client-seitig auf dem bereits geladenen Array (`itemMatchesSource()`), es gibt dafür keinen zusätzlichen Serverzugriff.
- Bewusst nur auf der jeweils obersten Trefferliste (Bibliothek-Chips bzw. Suchergebnis-Gruppen), nicht in den Detail-Tabs einer Interpreten-/Album-/Playlist-Ansicht.

## Metadaten ansehen

Das Aktionsblatt (`#sheetbg`) trägt ganz unten, direkt über „Abbrechen“, einen zusätzlichen Button „View metadata“ (`#aMetadata`, immer sichtbar, keine bedingte `.gone`-Logik wie bei „Öffnen“/„Interpret“/„Album“). Öffnet ein eigenes Sheet (`#metadataSheetbg`, `openMetadataSheet(it)`), das das Item über `resolveItem(it.uri, …)` (dieselbe Auflösung wie bei Favoriten, siehe „Favoriten-Asymmetrie“) neu vom Server holt und das Ergebnis komplett als eingerücktes JSON (`JSON.stringify(data, null, 2)`) in einem `<pre class="meta-json">` anzeigt – bewusst generisch statt einer kuratierten Feld-für-Feld-Ansicht, da sich der Feldumfang stark nach Medientyp und Provider unterscheidet. Läuft, wie `refreshNowFav()`/`refreshNowLyrics()`, über denselben Race-Guard (`metadataSheetUri`, verwirft die Antwort, wenn das Sheet inzwischen geschlossen oder für ein anderes Item neu geöffnet wurde). Ohne `it.uri` (sollte praktisch nicht vorkommen) wird direkt das clientseitig bereits vorhandene Objekt angezeigt, ohne Serveraufruf.

## Quick Actions

Das `#stationMenu`-Dropdown (geöffnet über den Button `#stationMore` neben der Stationsleiste, der den sichtbaren Text „Quick Actions ❯“ trägt) trägt unterhalb der Player-Liste eine zusätzliche Überschrift „Quick Actions“ und eine Zeile mit zwei quadratischen Buttons (`.qa-row` / `.qa-btn`):

- **Play Music** (`#qaPlayMusic`): Setzt den aktiven Player fort (`players/cmd/play_pause`, nur falls gerade nicht `isPlayerPlaying()`) und blendet dessen Lautstärke über 3 Sekunden auf 50 % hoch. MA kennt keinen eigenen „letzten Titel abspielen“-Befehl; da die Queue nach Pause/Stop an ihrer letzten Position stehen bleibt, ist ein einfaches Fortsetzen die nächstliegende Umsetzung von „spielt den zuletzt gespielten Titel“.
- **Sleep** (`#qaSleep`): Blendet **alle** bekannten Player (`playerIds`) parallel über 5 Sekunden auf 0 % Lautstärke herunter und pausiert danach jeden Player, der zu dem Zeitpunkt noch spielte.
- Beide nutzen den gemeinsamen Helfer `fadeVolume(ids, target, durationMs, cb)`: rampt linear per `setInterval` (alle 150 ms) via `players/cmd/volume_set` bzw. `players/cmd/group_volume` (dieselbe Umschaltregel wie beim normalen Lautstärke-Slider – `volume_level === null` bedeutet Gruppenplayer), ruft `cb` nach Ablauf der Dauer.

## Warteschlange: gemeinsames Slide-in-Panel

Die Warteschlange ist weder ein eigener Fußzeilen-Eintrag noch ein Bibliothek-Chip, sondern ein **einziges gemeinsames Panel** (`#queuePanel`, Markup direkt nach `.pages` und vor `.footer`), das von rechts über die gerade sichtbare Seite hereingleitet – unabhängig davon, ob gerade die Bibliothek- oder die Now-Playing-Seite aktiv ist. Zwei Trigger, eine Komponente: `#libQueueToggle` (eigener, von den Filter-Chips abgesetzter Button ganz oben auf der Bibliothek-Seite, siehe „Music-Source-Filter“) und `#npQueueBig` (viertes Icon in der `.titlerow` der Now-Playing-Seite, neben Favorit/Playlist/Songtexte) rufen beide dasselbe `toggleQueuePanel()` auf.

- `#queuePanel` ist `position: fixed`, zwischen Kopf- und Fußzeile eingepasst (`top: 68px; bottom: 60px;` – an die dort fest hinterlegten Höhen von `.top`/`.footer` angelehnt), standardmäßig per `-webkit-transform: translateX(100%)` rechts aus dem Sichtbereich geschoben; `.queue-panel.open` (gesetzt/entfernt von `openQueuePanel()`/`closeQueuePanel()`) blendet es per CSS-Transition ein.
- Innerhalb des Panels läuft alles unverändert über die bestehende Warteschlangen-Logik: `#queue` (dieselbe Liste, `loadQueue()`/`renderQueueRows()`/`queueRow()`/`tryQueueSlide()` aus dem Abschnitt „Cover-Crossfade und Queue-Slide“) sowie die Reload-/Leeren-Buttons in `#queueToolbar` (`#bQueueReload`/`#bQueueClear`) – nichts davon wurde dupliziert, nur das umgebende Panel ist neu.
- `queuePanelOpen` ersetzt die frühere `currentTab === 'library' && currentLibType === 'queue'`-Prüfung überall dort, wo Server-Events die Liste live nachladen sollen (`selectPlayer()`, `handleEvent()` für `queue_updated`/`queue_items_updated`) – die Warteschlange aktualisiert sich jetzt also, solange das Panel offen ist, unabhängig davon, welche Seite dahinter gerade sichtbar ist.
- `paintQueueToggle()` hält beide Trigger-Buttons synchron im `.on`-Zustand (Akzentfarbe), egal über welchen das Panel geöffnet wurde.
- Schließen: expliziter `#queuePanelClose`-Button, oder ein Klick außerhalb des Panels/der beiden Trigger (eigener `document.addEventListener('click', …)`, nach demselben Muster wie bei den Kopfzeilen-Dropdowns – siehe `closeStationMenu()` – aber bewusst ein separater Listener, da das Panel nicht zu deren sich gegenseitig ausschließender Gruppe gehört und parallel zu jeder Seite offen bleiben kann).

## Einstellungen

Ein Zahnrad-Button (`#settingsSwitch`) in der Kopfzeile öffnet ein Popover (`#settingsMenu`), das demselben Öffnen/Schließen-Muster wie `#stationMenu`/`#groupMenu`/`#accentMenu` folgt (siehe `document.addEventListener('click', …)` – alle vier Menüs schließen sich beim Öffnen eines anderen gegenseitig).

- Bislang eine Einstellung: eine **Standard-Musikquelle**, angewendet auf `librarySourceFilter` und `searchSourceFilter` (siehe „Music-Source-Filter“), sodass Bibliothek und Suche standardmäßig nach diesem Provider vorgefiltert starten.
- `loadMusicSources()` fragt `config/providers` mit `provider_type: 'music'` ab (liefert die tatsächlich auf dem Server konfigurierten/aktivierten Musik-Provider mitsamt `domain`/`enabled`), statt eine feste Liste zu hinterlegen – die Chip-Auswahl in `renderSettingsMenu()` spiegelt also immer den echten Provider-Bestand des jeweiligen MA-Servers.
- **Bewusst nur `localStorage`** (`DEFAULT_SOURCE_KEY = 'ma-dashboard-default-source'`), nicht geräteübergreifend synchronisiert: Music Assistants Server-API bietet aktuell keinen generischen Key-Value-Speicher, den ein Drittanbieter-Client für beliebige eigene Daten nutzen könnte – `config/core/*` ist pro eingebautem Subsystem validiert (Streams, Webserver, …), `config/providers/*` gehört den Provider-Instanzen selbst, `config/players/*` ist strikt pro Player, und der Dashboard-Controller verwaltet nur Cast-Sessions, keine Nutzereinstellungen (geprüft im `music-assistant/server`-Quellcode, `dev`-Branch, Stand dieser Version). Sollte MA künftig einen generischen Einstellungs-Endpunkt bekommen, wäre das die naheliegende Stelle, um hier auf echte geräteübergreifende Synchronisierung umzusteigen.
- Eine Auswahl in `chooseDefaultSource(domain)` schreibt sofort in `localStorage`, aktualisiert beide Filter-Variablen und lädt die gerade sichtbare Bibliothek-/Suchliste neu, falls eine davon offen ist.

## Dark/Light Mode

Alle Farben im `<style>`-Block sind als CSS Custom Properties auf `:root` definiert (`--bg`, `--bg-elevated`, `--bg-elevated-2`, `--surface`, `--surface-active`, `--border`, `--text`, `--text-dim`, `--text-secondary`, `--text-muted`, `--text-faint`, `--accent`, `--danger`, `--fav`, `--fav-border`; theme-unabhängig konstant: `--on-accent`, `--on-danger`, `--success`). Kein einzelner Farbwert im CSS oder in Inline-Styles darf mehr als Hex-Literal auftauchen – neue Regeln immer über `var(--...)` anlegen, sonst bricht das Theme an der Stelle.

Drei Ebenen bestimmen den aktiven Wertesatz, in aufsteigender Priorität:

1. `:root` liefert die Dark-Werte als Basis (kein Attribut nötig).
2. `@media (prefers-color-scheme: light) { :root:not([data-theme]) { … } }` überschreibt auf Light, wenn das System hell eingestellt ist und keine explizite Wahl vorliegt.
3. `:root[data-theme="light"] { … }` überschreibt explizit, unabhängig vom System, sobald `<html>` das Attribut `data-theme="light"` trägt (für Dark ist kein eigener `[data-theme="dark"]`-Block nötig, da das schon die Basis von `:root` ist).

Die JS-Seite (`THEME_STORAGE_KEY = 'ma-dashboard-theme'`, ab `detectTheme()`) liest eine gespeicherte Wahl aus `localStorage`, setzt/entfernt das `data-theme`-Attribut auf `document.documentElement` und pflegt Icon (`#themeSwitch`, Mond/Sonne je nach `effectiveTheme()`) sowie `<meta id="themeColorMeta" name="theme-color">` nach. Ein Klick auf `#themeSwitch` (`toggleTheme()`) setzt immer eine explizite Wahl, die das System dauerhaft übersteuert – es gibt bewusst keinen UI-Weg zurück zu „automatisch“, nur eine neue manuelle Wahl. Ein `matchMedia`-Listener hält Icon/Meta aktuell, falls sich die Systemeinstellung ändert, während noch keine explizite Wahl gespeichert ist.

Ein kleines Inline-`<script>` direkt im `<head>` (vor dem Cache-Buster-Script für `ma-env.js`) wendet eine gespeicherte Wahl schon vor dem ersten Paint an, damit beim Laden nicht kurz das falsche Theme aufblitzt.

Die Akzentfarbe `--accent` selbst ist zwischen den Themes bewusst *nicht* konstant (Dark `#f0a32e`, Light `#c97f17`): Sie wird im UI sowohl als Hintergrundfläche (mit `--on-accent`-Text) als auch direkt als Textfarbe verwendet; auf hellem Grund bräuchte reines `#f0a32e` als Text zu wenig Kontrast, daher ein etwas dunkleres Amber im Light-Theme.

## Akzentfarbe

Der Nutzer kann die Akzentfarbe (Standard: Gelb/Amber `#f0a32e`) über einen Farbkreis in der Kopfzeile (`#accentSwitch`) aus einer festen Palette (`ACCENT_PRESETS`, definiert im frühen Inline-`<script>` im `<head>`) wählen. Bewusst keine freie Farbwahl (kein `<input type="color">`) – das wird von Safari 9 nicht unterstützt, eine feste Palette aus antippbaren Swatches ist auf jedem iOS-Stand zuverlässig nutzbar.

- `ACCENT_STORAGE_KEY = 'ma-dashboard-accent'` in `localStorage`; `ACCENT_PRESETS[0]` ist immer der Standardwert `#f0a32e`.
- Anders als beim Theme wird die Akzentfarbe **nicht** direkt per `document.documentElement.style.setProperty(...)` gesetzt – das würde als Inline-Style auf `<html>` sowohl die Dark- als auch die Light-Variante gleichzeitig überschreiben und die Drei-Ebenen-Kaskade aus „Dark/Light Mode“ zerstören. Stattdessen injiziert `applyAccentOverride()` bei einer echten Nutzerauswahl ein `<style id="accentOverrideStyle">`-Element in den `<head>`, das exakt dieselbe dreistufige `:root` / `@media (prefers-color-scheme: light)` / `:root[data-theme="light"]`-Struktur für `--accent` reproduziert – nur eben mit der gewählten Farbe statt der Standardwerte. Bei Rückkehr zum Standard-Preset wird der Style-Inhalt einfach geleert, die fest im Haupt-CSS hinterlegten Standardwerte greifen wieder unverändert.
- `deriveLightAccent(hex)` (RGB→HSL→RGB, alle Helfer ebenfalls im frühen `<head>`-Script) berechnet die Light-Theme-Variante automatisch, indem die Lightness auf 78 % skaliert wird – dieselbe Kontrastlogik, die auch dem handverlesenen Standard-Farbpaar (`#f0a32e`/`#c97f17`) zugrunde liegt. Reine Presets, keine Nutzereingabe, daher ohne weitere Validierung sicher.
- `applyAccentOverride()` wird sowohl im frühen `<head>`-Script (FOUC-Vermeidung, gleiches Prinzip wie beim Theme) als auch von `setAccent()` im Hauptskript aufgerufen; beide Stellen teilen sich dieselbe globale Funktion statt Farb-Mathematik zu duplizieren.
- Das Popover-Menü (`#accentMenu`, Swatches in `#accentSwatches`, gerendert von `renderAccentSwatches()`) folgt demselben Öffnen/Schließen-Muster wie `#stationMenu`/`#groupMenu` (siehe `document.addEventListener('click', …)` weiter unten in der Datei) – alle drei Menüs schließen sich gegenseitig beim Öffnen eines anderen.

## Songtexte

Songtexte werden über den dedizierten Befehl `metadata/get_track_lyrics` abgerufen (Parameter `track`: volles Track-Objekt; Rückgabe `[lyrics, lrc_lyrics]` – Klartext bzw. LRC-Format mit Zeitstempeln pro Zeile, `[mm:ss.xx]Text`, teils mehrere Zeitstempel pro Zeile, beides ggf. `null`). Bewusst **nicht** über `music/item_by_uri`: Das liefert nur den aktuellen DB-Cache und stößt selbst mit `allow_update_metadata: true` bloß einen gedrosselten Hintergrund-Task an (serverseitig `schedule_update_metadata()`, global auf 1 Request/30s limitiert) – `metadata/get_track_lyrics` dagegen sucht synchron innerhalb derselben Anfrage (bei Bibliothekstiteln per sofortigem Metadaten-Update, sonst direkt beim Track-Provider bzw. den Lyrics-Metadaten-Providern LRCLIB/Genius) und liefert eine definitive Antwort in einem Rutsch. Genau das nutzt auch die offizielle MA-Weboberfläche (`getTrackLyrics()` im MA-Frontend).

Ein Button (`#npLyrics` in der kompakten `.titlerow`, `#npLyricsBig` in der Now-Playing-Vollbildansicht – beide über `npOn('npLyrics', 'onclick', toggleLyricsView)` auf denselben Handler verkabelt, siehe „Seiten-Router (Fußnavigation)“) schaltet die Songtextanzeige um. Anders als früher ist das **kein eigener Tab/keine eigene Pane** mehr, sondern ein Zustand innerhalb der Now-Playing-Seite: `nowLyricsOpen` (Boolean) plus die CSS-Klasse `#pageNow.lyrics-open`, die dort `.now-inner` (Cover/Steuerung) gegen `#lyricsBoxBig` austauscht (siehe „Now Playing: Vollbild und Farbverlauf“). `toggleLyricsView()` navigiert bei Bedarf zunächst per `switchTab('now')` zur Now-Playing-Seite (z. B. wenn der Button aus der Bibliothek/Suche heraus angetippt wird) und öffnet dort die Ansicht; `syncNowPageState()` (aufgerufen als `TABS.now.init`) hält die CSS-Klasse und die geladenen Lyrics-Daten bei jedem Wechsel auf die Now-Playing-Seite konsistent.

- **Lazy statt eager**: `refreshNowLyrics()` wird nur aufgerufen, wenn die Songtextansicht tatsächlich offen ist (`syncNowPageState()`, sowie im `uri !== npUri`-Block von `renderNow()` nur unter `if (currentTab === 'now' && nowLyricsOpen)`) – nicht bei jedem Titelwechsel im Hintergrund, um unnötige Provider-Anfragen für nie angesehene Titel zu vermeiden. `npLyricsForUri` merkt sich, für welche URI bereits geladen wurde/wird, und verhindert Doppel-Fetches beim erneuten Öffnen desselben Titels.
- `npTrack` (volles Track-Objekt, i. d. R. `q.current_item.media_item`) wird bei jedem `renderNow()`-Durchlauf aktuell gehalten, nicht nur bei URI-Wechsel, falls die Queue-Daten minimal hinter `current_media` hinterherhinken. Ohne `npTrack` (z. B. Radio-Stream ohne Queue-Item) gilt sofort und ohne Serveraufruf „keine Songtexte verfügbar“.
- Race-Guard wie bei `refreshNowFav()`: Antwort wird verworfen, wenn der Titel zwischen Anfrage und Antwort gewechselt hat (`want !== npUri`).
- `parseLrc()` ist ein kleiner ES5-Parser für das LRC-Format: extrahiert führende `[mm:ss.xx]`-Zeitstempel-Tags am Zeilenanfang (mehrere Tags pro Zeile ergeben mehrere Einträge mit identischem Text), überspringt Zeilen ohne Zeit-Tag (LRC-Kopfzeilen wie `[ar:...]`/`[ti:...]`) und sortiert das Ergebnis nach Zeit.
- Die aktuell gesungene Zeile wird über `updateLyricsHighlight(pos)` hervorgehoben, aufgerufen aus `tick()` mit derselben interpolierten Position wie der Fortschrittsbalken (siehe „Fortschrittsbalken und der Beta-Bug“) – keine zweite Zeitbasis. Scrollen erfolgt manuell über `scrollTop`/`offsetTop`/`clientHeight`, nicht über `scrollIntoView()` mit Optionsobjekt, da Safari 9 nur dessen boolesche Form kennt. Voraussetzung dafür ist `#lyricsBoxBig { position: relative; }` im CSS – ohne explizite Positionierung wäre der `offsetParent` der `.lyric-line`-Elemente `<body>` statt der Box selbst, was die Scroll-Zentrierung verfälscht.
- Der Button bleibt immer sichtbar, sobald ein Titel läuft (`npUri` gesetzt) – unabhängig davon, ob Lyrics-Daten schon bekannt sind. Der Leerzustand („Keine Songtexte verfügbar“) bzw. Ladezustand wird in der Pane selbst kommuniziert.
- Die Songtextanzeige bleibt beim Titelwechsel offen und aktualisiert sich automatisch auf den neuen Titel. `currentTab === 'now' && nowLyricsOpen` ist dabei die Zustandsquelle für „gerade sichtbar“.

## Favoriten-Asymmetrie

MA behandelt Favoriten absichtlich asymmetrisch:

- **Hinzufügen** geht über `music/favorites/add_item` mit der URI des Elements.
- **Entfernen** braucht dagegen `media_type` + `library_item_id` aus dem Bibliothekselement.

Das Dashboard löst deshalb vor jeder Favoriten-Aktion das Element über `music/item_by_uri` auf. Nur so ist sicher, ob das Element in der Bibliothek liegt und ob `favorite` bereits gesetzt ist.

## Bibliothekselemente und Provider-Auflösung

Elemente aus der Bibliothek (`music/<type>s/library_items`) tragen `provider === 'library'` statt eines echten Streaming-Provider-Domains. `music/artists/artist_albums`/`artist_tracks` liefert bei `provider_instance_id_or_domain: 'library'` serverseitig nur die bereits in der Bibliothek verknüpften Einträge (reine DB-Abfrage), nicht die vollständige Diskografie des Providers.

`resolveProviderRef(it)` löst deshalb bei Bibliothekselementen über `it.provider_mappings` (Pflichtfeld auf jedem MA-Medienobjekt) auf einen echten Provider auf und liefert `{ item_id, provider_instance_id_or_domain }` für den eigentlichen Streaming-Provider. `openArtist()` nutzt das, damit „Top-Titel“/„Alben“ auch bei Navigation über die Bibliothek die vollständige Liste zeigen, statt nur der wenigen bereits bibliotheksverknüpften Einträge.

## Bilder und Imageproxy

MA liefert zu jedem Medium eine `proxy_id`. Über `/imageproxy/<proxy_id>?size=<n>&fmt=jpeg` skaliert der Server das Bild auf die gewünschte Kantenlänge. Das ist auf alten iPads essenziell, da Original-Cover mehrere Megabyte groß sein können.

Der Imageproxy akzeptiert nur die Größen `{0, 80, 160, 256, 512, 1024}`; andere Werte führen zu HTTP 400. `CFG.IMG_COVER` und `CFG.IMG_THUMB` müssen daher auf diese Werte gesetzt werden (Standard: 512 für Cover, 160 für Thumbnails). Falls kein `proxy_id` vorhanden ist, wird als Fallback die Legacy-URL `/imageproxy?path=…&provider=…` verwendet.

Falls `current_media.image_url` aus einer falsch konfigurierten internen `base_url` des MA-Servers stammt (typisch bei Reverse-Proxy oder Docker), wird der Host für proxy-URLs auf `CFG.MA` umgebogen (`fixUrl`).

## Homescreen-Webclip und Update-Erkennung

Als „Zum Home-Bildschirm hinzugefügt“-App (`apple-mobile-web-app-capable`) hält WebKit auf iPads oft hartnäckig an einem einmal geladenen Snapshot von `ma-dashboard.html` fest – die `<meta http-equiv="Cache-Control">`-Tags im `<head>` reichen dafür nicht aus, weil sie nur echte HTTP-Response-Header ersetzen können, wenn der Server selbst keine setzt, und der Webclip-Modus eigene, aggressivere Regeln hat. Ohne Gegenmaßnahme bekommt man neue Versionen erst nach manuellem Löschen und Neuanlegen des Homescreen-Icons zu sehen.

Dagegen gibt es `checkForUpdate()`: Beim Start, bei Rückkehr aus dem Hintergrund (`visibilitychange`), bei `pageshow` **und periodisch alle `UPDATE_CHECK_INTERVAL_MS`** (Default 10 Minuten, `setInterval(checkForUpdate, UPDATE_CHECK_INTERVAL_MS)` neben `setInterval(tick, 500)`) wird `ma-dashboard.html` per `XMLHttpRequest` mit cache-bustendem Query-String (`?_v=<timestamp>`) sowie expliziten `Cache-Control`/`Pragma: no-cache`-Request-Headern frisch vom Server geholt und die darin per Regex gefundene `APP_VERSION`-Konstante mit der gerade laufenden verglichen. Weicht sie ab, erscheint ein Toast und nach kurzer Verzögerung eine harte Navigation (`location.href`) auf eine neue URL (anderer Query-String), damit WebKit nicht erneut die gecachte Fassung ausliefert.

Der periodische Timer ist der eigentlich verlässliche Pfad, nicht nur eine Ergänzung: Diese App ist als dauerhaft sichtbares Kiosk-Display gedacht (wandmontiertes iPad, `setInterval(tick, 500)` läuft durchgehend für den Fortschrittsbalken). Ein solches Gerät wechselt unter Umständen wochenlang nie wirklich in den Hintergrund und navigiert nie neu – `visibilitychange`/`pageshow` feuern dann schlicht nicht mehr, und ohne den Intervall-Timer bliebe die Update-Erkennung faktisch tot, obwohl der JS-Kontext nachweislich weiterläuft. Wird die Datei zusätzlich über eine Route mit langlebigem `Cache-Control` ausgeliefert (z. B. Home Assistants `/local`-Static-Route für Dateien im `www/`-Verzeichnis, bekannt für recht lange Cache-Zeiten), begünstigt das zusätzlich eine veraltete Fassung beim allerersten Laden – das ist serverseitig nicht aus diesem Repo heraus behebbar, daher der rein clientseitige, ereignisunabhängige Polling-Ansatz.

**Wichtig:** `APP_VERSION` (oben im `<script>`, vor dem Abschnitt „KONFIGURATION“) muss bei jeder funktionalen Änderung an `ma-dashboard.html` erhöht werden – sonst erkennt der Mechanismus keine neue Version und der Webclip bleibt trotz Deployment auf dem alten Stand.

## Entwicklung und Anpassung

- Änderungen nur in `ma-env.js` vornehmen; der Rest sollte normalerweise nicht angefasst werden müssen.
- `ma-env.js` wird in `.gitignore` ausgeschlossen; Änderungen daran nicht committen.
- Neue Medientypen oder API-Endpunkte in `NAVIGABLE`, `GROUPS`, `LIB_TYPES` und den Detail-Funktionen (`openArtist`, `openAlbum`, `openPlaylist`, `openPodcast`) ergänzen.
- Tabs in Detailansichten werden im View-Objekt als `tabs: [{ key, label, load }]`, `activeTab` und `cache` modelliert.
- „Zur Playlist hinzufügen“ und Automix erweitern das Aktionsblatt bzw. die Toggle-Leiste; neue UI-Elemente müssen ES5-tauglich bleiben.
- Im „Zur Playlist hinzufügen“-Sheet sortiert `loadUserPlaylists()` Favoriten nach oben und ordnet die restlichen Playlists alphabetisch nach Namen.
- Der Bibliothek-Tab ruft `music/<type>s/library_items` ohne `favorite`-Filter auf, damit jedes Item das `favorite`-Flag mitbringt und `mediaRow()` das Herz anzeigen kann. Ein Toggle-Chip „Nur Favoriten“ filtert die Liste client-seitig.
- Gruppen-Member-Lautstärke: Mitglieder werden aus `group_members`/`group_childs` bzw. als Fallback über `synced_to`/`active_group` ermittelt. Jedes Mitglied, das `supported_features` enthält, bekommt einen eigenen Slider unter dem Haupt-Volume-Slider.
- `rebuildPlayerOrder()` schließt Gruppenmitglieder konsequent aus `playerIds` aus – geprüft werden `p.synced_to`, `p.active_group` UND ob die ID in `group_members`/`group_childs` eines anderen Players auftaucht. Manche MA-Player spiegeln ihre Mitgliedschaft nämlich nur beim Gruppenleiter, nicht auf sich selbst; wird das nicht berücksichtigt, bleibt so ein Mitglied in `playerIds` und meldet über `players/all` oft ebenfalls `state: 'playing'` – dadurch erschien beim Laden kurzzeitig eine zweite „aktive“ Station neben dem eigentlich gewählten Player in der Kopfleiste (`buildStations()`).
- Gruppierungs-Menü: `resolveGroupMembers(id, filterVolume)` ist die gemeinsame Basis für Gruppenmitglieder – mit `filterVolume=true` liefert `getGroupMembers()` nur lautstärkefähige Mitglieder (für die Lautstärke-Slider), mit `filterVolume=false` liefert `renderGroupMenu()` alle Mitglieder (für Anzeige/Entfernen). Kandidaten zum Hinzufügen werden aus `playerIds` gefiltert nach `set_members` in `supported_features`.
- Reiter-Herkunft bei Detailnavigation: `enterDetail()` merkt sich in `navOriginTab`, aus welchem Fußzeilen-Eintrag (typischerweise Bibliothek) eine Detailansicht geöffnet wurde, und übergibt das an `switchTab(name, pillName)` als zweiten Parameter, damit der ursprüngliche Eintrag optisch aktiv bleibt, obwohl die Detailansicht technisch immer über `switchTab('search', …)` in `#paneSearch` (innerhalb von `#pageLibSearch`) läuft. `popView()` schaltet beim Leerwerden des Navigations-Stacks zurück auf `navOriginTab`. Neue Navigationseinstiege in Detailansichten sollten `enterDetail()` statt eines direkten `switchTab('search')` verwenden.
- Neue Fußzeilen-Seiten: `TABS` um einen Eintrag ergänzen (`{ pane, init }`), im HTML ein `<div class="tab" data-tab="…">`-Icon in `.footer` sowie ein `#page…`-Container in `.pages` anlegen, das Icon in `svgIcon()` ergänzen und beim Boot per `$('foot…').innerHTML = svgIcon(…)` setzen (siehe „Seiten-Router (Fußnavigation)“).
- Sortierung: `sortMediaItems(items, mode)` (`'name'`/`'year'`, bei `'year'` Fallback auf `item.album.year` für Titel ohne eigenes Jahresfeld) ist der gemeinsame Sortier-Helfer für Bibliothek (`libSortMode`), Suche (`searchSortMode`, zusätzlich `'relevance'` als Server-Reihenfolge) und die Tabs „Titel“/„Alben“ der Interpretenansicht (`artistSortMode`, angewendet in `renderActiveSections()`). Die gemeinsame `SORT_MODES`-Konstante (`Name`/`Jahr`) speist sowohl die Bibliothek-Chips als auch die Sortier-Chips in `renderTabs()`. Tabs im View-Objekt tragen dafür ein `sortable`-Flag; „Top-Titel“ ist bewusst `sortable: false`, da seine Reihenfolge die Popularitäts-Rangfolge ist. Nicht in der gemeinsam genutzten `renderSections()` angewendet, da diese auch von Detail-Titellisten mit fixer Reihenfolge (Track-Nummern, Playlist-Reihenfolge) genutzt wird.
- Default-Player-Auswahl: `pickDefaultPlayer()` (in `bootstrap()` verwendet) übernimmt `CFG.DEFAULT_PLAYER` nur, wenn `isPlayerUsable()` (`available`/`enabled` nicht `false`) zutrifft; sonst wird aus `playerIds` der erste Player mit `state === 'playing'` gewählt (Gruppen werden dabei über ihren Leader erkannt, siehe `synced_to`-Filterung in `rebuildPlayerOrder()`), sonst der erste aus `playerIds`.
- UI-Anpassungen nur mit ES5-tauglichem CSS und JS vornehmen.
- Keine externen Bibliotheken einbinden; die Datei soll weiterhin eigenständig lauffähig bleiben.

## Troubleshooting

- **„Anmeldung abgelehnt“** – `CFG.TOKEN` prüfen; Token muss Long-Lived sein und die Rolle mindestens `user` haben.
- **„Dieser Browser kann kein WebSocket“** – iPad/Safari zu alt; mindestens iOS 9.3 erforderlich.
- **Verbindungsampel bleibt rot** – `CFG.MA` prüfen, MA-Server erreichbar, Firewall/WebSocket-Proxy beachten.
- **Balken läuft nicht mit** – siehe Abschnitt „Fortschrittsbalken und der Beta-Bug“; ist kein Client-Bug, sondern erwartetes Server-Verhalten.
- **Bilder laden nicht** – `CFG.MA` muss korrekt sein; bei Reverse-Proxy muss `/imageproxy` erreichbar sein.
- **Homescreen-App zeigt alte Version** – prüfen, ob `APP_VERSION` beim letzten Deployment tatsächlich erhöht wurde (siehe „Homescreen-Webclip und Update-Erkennung“); ohne das erkennt `checkForUpdate()` keine neue Version. Ein bereits installiertes Homescreen-Icon, das älter als der periodische Update-Check (`UPDATE_CHECK_INTERVAL_MS`, eingeführt in dieser Version) ist, braucht dafür **einmalig noch** den manuellen Reset (Icon löschen → im Safari-Tab mit Cache-Busting-Query neu laden → Icon neu anlegen); danach aktualisiert sich die App künftig spätestens nach `UPDATE_CHECK_INTERVAL_MS` von selbst, auch ohne dass sie je in den Hintergrund/Vordergrund wechselt.
- **Theme wechselt nicht mit der Systemeinstellung** – meist liegt bereits eine explizite manuelle Wahl in `localStorage` (`ma-dashboard-theme`), die das System dauerhaft übersteuert (siehe „Dark/Light Mode“); Eintrag löschen, um wieder auf automatisch umzuschalten.
