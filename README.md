# BibBot (WISO-IP) – Fork für Firefox

Inoffizieller Fork von [BibBot](https://github.com/stefanw/bibbot) von Stefan Wehrmeyer
([Website des Originals](https://stefanw.github.io/bibbot/)).

BibBot entfernt die Paywall auf deutschen Nachrichtenseiten, indem es den Artikel
im Hintergrund über den Pressedatenbank-Zugang deiner Bibliothek (Genios/WISO) lädt.

Dieser Fork ergänzt vor allem **WISO-Zugänge, die per IP-Adresse freigeschaltet sind**
(typisch für Hochschulnetze), und repariert einige Nachrichtenseiten. Er wird nur für
**Firefox** angeboten.

> Dieser Fork steht in keiner Verbindung zum Original-Projekt, zu GBI-Genios/WISO
> oder zu einem Verlag. Fehler bitte hier melden, nicht beim Original.

## Installation (Firefox)

1. Unter [Releases](https://github.com/RideMyByte/bibbot/releases) die neueste
   `bibbot-wiso-v….xpi` herunterladen.
2. Datei in Firefox öffnen (oder ins Firefox-Fenster ziehen) und „Hinzufügen“ bestätigen.
3. Ist das Original-BibBot installiert: unter `about:addons` **deaktivieren**, sonst
   greifen beide Erweiterungen auf dieselben Seiten zu.
4. BibBot-Einstellungen öffnen und deine Bibliothek bzw. deinen Zugang wählen.

Die Erweiterung ist von Mozilla signiert (Self-Distribution, keine öffentliche
Seite auf addons.mozilla.org) und bleibt dauerhaft installiert. Benötigt wird
Firefox 140 oder neuer.

**Updates kommen nicht automatisch.** Neue Versionen einfach wieder unter
Releases herunterladen und installieren – Einstellungen bleiben erhalten.

## Zugang „WISO – Hochschule (Zugang per IP, ohne Login)“

Neu in diesem Fork. Für Einrichtungen, bei denen wiso-net.de ohne Login
funktioniert, sobald man sich in ihrem Netz befindet.

- Funktioniert nur, wenn dein Browser-Traffic aus dem freigeschalteten Netz kommt
  (vor Ort oder über das offizielle VPN deiner Einrichtung).
- Benutzername/Passwort werden nicht benötigt.
- Test: Öffnest du https://www.wiso-net.de/ und kannst ohne Anmeldung Artikel lesen,
  sollte dieser Zugang funktionieren.
- Bitte beachte die Lizenzbedingungen deiner Einrichtung: Der Zugang gilt in der
  Regel nur für deren Angehörige.

Alle anderen Bibliotheken des Originals funktionieren wie gewohnt.

## Änderungen gegenüber dem Original

Basis: Original-Stand `a265ba2` (Version 0.40.8).

| Bereich | Datei | Änderung |
|---|---|---|
| Neuer Zugang | `src/providers.ts` | Provider `ip.wiso-net.de`: überspringt das Login komplett (Zugang per IP), nutzt die neue Genios-Oberfläche auf `www.wiso-net.de` |
| WISO-Suche | `src/sources.ts`, `src/types.ts` | Suchpfad pro Provider einstellbar (`searchCategory`). WISO nutzt `/searchResult/Alle%20Quellen`, Genios eBib `/searchResult/Alle` (sonst HTTP 403) |
| waz.de | `src/sites.ts` | Artikelbereich jetzt `article .article-body` (alter Selektor existierte nicht mehr → Erweiterung brach ohne Meldung ab) |
| waz.de | `src/sites.ts` | Quelle „Westdeutsche Allgemeine Zeitung online“ (Kürzel `WAZO`) ergänzt |
| waz.de | `src/sites.ts` | Suchtext: nur die ersten vier Wörter (≥ 6 Buchstaben) der Überschrift; die volle Überschrift mit Satzzeichen findet WISO nicht |
| Firefox | `manifest.json` | `background.scripts` statt `service_worker` (Firefox unterstützt MV3-Service-Worker nicht) |
| Firefox | `manifest.json` | Eigene Add-on-ID `bibbot-wiso@ridemybyte.github.io`, Name „BibBot (WISO-IP)“, Version `0.40.8.1`, mindestens Firefox 140, `data_collection_permissions` (Pflicht für neue Erweiterungen) |
| Release | `.github/workflows/release.yml` | Baut, lässt von Mozilla „unlisted“ signieren und veröffentlicht `.xpi` + Quellcode als GitHub-Release. Kein Chrome Web Store, keine GitHub Pages |

## Datenschutz

Wie im Original: Zugangsdaten und Einstellungen werden nur lokal im Browser
gespeichert (`browser.storage`). Für die Suche wird der Artikeltitel bzw. ein
Textausschnitt an die Pressedatenbank deiner Bibliothek geschickt, Zugangsdaten
nur an deine Bibliothek. Siehe [privacy.md](privacy.md).

## Entwicklung

```sh
npm install
npm start          # Rollup im Watch-Modus
npm run lint && npm run check-types
./dist.sh          # baut dist/bibbot/ und dist/bibbot.zip
```

Zum Testen ohne Signatur: `about:debugging#/runtime/this-firefox` → „Temporäres
Add-on laden…“ → `dist/bibbot.zip` (verschwindet beim Neustart von Firefox).

### Aufbau

- `src/content.ts` läuft auf der Artikelseite und spricht mit dem Background-Script
- `src/background.ts` öffnet Tabs im Hintergrund, navigiert und liest den Artikel aus
- `src/options.ts` ist die Einstellungsseite, `popup/` das Popup am Symbol
- `src/providers.ts`: Bibliotheken/Zugänge, gegen die man sich anmeldet
- `src/sources.ts`: Datenbanken (z. B. Genios, WISO), die über Provider erreichbar sind
- `src/sites.ts`: Nachrichtenseiten, wie Metadaten ausgelesen werden, welche Quelle passt
- `src/types.ts`: Typen für all das

### Eine Nachrichtenseite reparieren

- **Keine BibBot-Meldung auf der Seite:** meist passt `selectors.main` oder
  `selectors.paywall` in `src/sites.ts` nicht mehr zum Layout der Seite.
- **„Artikel nicht gefunden“:** URL des von BibBot geöffneten WISO-Tabs ansehen und
  Suchtext (`requestText`), Datum (`date=`) und Quelle (`source=`) einzeln prüfen.

## Release (für Maintainer)

Einmalig: auf https://addons.mozilla.org/developers/addon/api/key/ API-Zugangsdaten
erzeugen und im Repository unter *Settings → Secrets and variables → Actions*
als `AMO_API_KEY` (JWT issuer) und `AMO_API_SECRET` (JWT secret) hinterlegen.

Pro Release:

1. `version` in `manifest.json` erhöhen (z. B. `0.40.8.2`) und committen.
2. `git tag v0.40.8.2 && git push origin main v0.40.8.2`
3. Die Action signiert bei Mozilla (unlisted) und legt das Release mit `.xpi` und
   Quellcode-ZIP an. Die erste Signierung kann einige Minuten dauern.

## Lizenz

GPL-3.0 wie das Original, siehe [LICENSE](LICENSE). Quellcode jedes Releases liegt
als ZIP am Release und in diesem Repository.
