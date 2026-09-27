# Erobre nabolaget

Et gymspill for barneskolen, inspirert av *Jet Lag: The Game*. Elevene løper rundt i grupper og tar bilde av gatenavnskilt. Gater bare én gruppe har funnet, blir erobret med en gang. Gater som flere grupper har funnet, avgjøres i en kort duell i gymsalen. Gruppa med flest meter erobret gate vinner.

Nettsiden er statisk (GitHub Pages) og bruker Firebase Realtime Database til å dele data mellom lærerens maskin og elevenes iPader. Alle lærere kan lage spill i sitt eget nabolag. Gatene hentes fra OpenStreetMap.

## Slik spilles det

1. **Løp.** Læreren viser kartet og starter klokka. Gruppene fotograferer gatenavnskilt innenfor sirkelen.
2. **Last opp.** Hver gruppe logger inn med spillkoden og sin egen firesifrede gruppekode, og laster opp bildene. Tekstleseren (Tesseract, som kjører på iPaden) prøver å kjenne igjen gatenavnet. Gater den kjenner igjen, teller med en gang. Resten godkjenner eller avviser læreren.
3. **Poeng.** Opplastingen stenger. Gater bare én gruppe har, er erobret. Poengene er gatas lengde i meter innenfor sirkelen.
4. **Duell.** Portalen trekker en enkel øvelse for hver gate flere grupper har funnet. Læreren trykker på vinneren, og gata skifter farge på kartet.

## Oppsett (en gang)

### 1. Lag et Firebase-prosjekt

1. Gå til <https://console.firebase.google.com> og lag et nytt prosjekt, for eksempel `erobre-nabolaget`. Google Analytics trengs ikke.
2. **Build → Authentication → Kom i gang → Sign-in method:** slå på **Anonymous**.
3. **Build → Realtime Database → Opprett database.** Velg plassering **europe-west1 (Belgia)** og start i **låst modus**.
4. Åpne fanen **Regler** i Realtime Database. Lim inn hele innholdet i `database.rules.json` og trykk **Publiser**.
5. **Prosjektinnstillinger → Generelt → Dine apper → Nettapp (`</>`).** Registrer appen uten Hosting, og kopier `firebaseConfig`.
6. Lim verdiene inn i `firebase-config.js`. Pass på at `databaseURL` kommer med.
7. **Authentication → Innstillinger → Autoriserte domener:** legg til `DITT-BRUKERNAVN.github.io`.

### 2. Publiser på GitHub Pages

1. Lag et nytt offentlig repo, for eksempel `erobre-nabolaget`.
2. Last opp `index.html`, `firebase-config.js`, `database.rules.json` og `README.md`.
3. **Settings → Pages → Deploy from a branch → main / (root).**
4. Etter et minutt ligger spillet på `https://DITT-BRUKERNAVN.github.io/erobre-nabolaget/`.

Verdiene i `firebase-config.js` er ikke hemmelige. Det er databasereglene som bestemmer hvem som får lese og skrive.

## Slik er dataene beskyttet

- Ingen elever logger inn med navn. Hver enhet får en anonym Firebase-bruker, og gruppene heter det læreren kaller dem.
- Bare den som har lærerlenken, kan endre spillet, se gruppekodene og se alle bildene.
- En elev kan bare laste opp til sin egen gruppe, og bare med riktig gruppekode. Bildene til andre grupper kan de ikke se.
- Etter fase 2 kan ingen elever laste opp mer.
- Bildene lagres nedskalert, cirka 50–100 kB hver. Læreren kan slette alle bildene, eller hele spillet, under **Oppsett → Rydd opp**.
- Si til elevene at de skal ta bilde av skiltet og ikke av folk. Skolen bør vurdere om Firebase (Google, datalagring i EU) er i tråd med skolens rutiner for personvern.

## Grenser i gratisplanen (Spark)

Realtime Database gir 1 GB lagring og 10 GB nedlasting per måned. Ett spill med fem grupper og 100 bilder bruker rundt 5–10 MB. Slett bildene etter spillet, så holder gratisplanen lenge.

## Teknisk

- Én fil: `index.html`. Leaflet, qrcode-generator, Firebase (compat 10.12.2) og Tesseract.js lastes fra CDN.
- Gater: Overpass API (`highway` med `name` innenfor 2,1 km). Gangveger av typen `footway`, tunneler og navn på bruer tas ikke med. Gatene klippes mot sirkelen, og lengden regnes bare for delen som ligger innenfor.
- Kart: CARTO Voyager-fliser. © OpenStreetMap-bidragsytere (ODbL), © CARTO.
- Datamodell: `games/{kode}/meta | groups | streets | claims/{gruppe}/{id} | imgs/{gruppe}/{id} | verdicts | duels | secret | teachers | members`.

### Testing lokalt med Firebase-emulator

Legg til `?emu` i adressen, for eksempel `http://127.0.0.1:8080/?emu`. Da kobler siden seg til Auth-emulatoren på port 9099 og Database-emulatoren på port 9000:

```
firebase emulators:start --only auth,database --project demo-erobre
```
