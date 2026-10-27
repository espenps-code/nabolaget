# Erobre nabolaget

Et utendørsspill for skoleklasser, idrettslag, speidere, jobben og vennegjengen. Lagene løper rundt og tar bilde av gatenavnskilt. Gater bare ett lag har funnet, blir erobret med en gang. Gater som flere lag har funnet, avgjøres i en kort duell når dere er samlet igjen. Laget med flest meter erobret gate vinner.

Nettsiden er statisk (GitHub Pages) og bruker Firebase Realtime Database til å dele data mellom spillederens maskin og deltakernes mobiler eller nettbrett. Hvem som helst kan lage spill i sitt eget nabolag. I Norge hentes gatene fra Statens vegvesen, ellers fra OpenStreetMap.

## Slik spilles det

1. **Løp.** Spillederen viser kartet og starter klokka. Gruppene fotograferer gatenavnskilt innenfor sirkelen.
2. **Last opp.** Hver gruppe logger inn med spillkoden og sin egen firesifrede gruppekode, og laster opp bildene. Tekstleseren (Tesseract, som kjører på enheten) prøver å kjenne igjen gatenavnet. Appen leser også når bildet ble tatt, fra metadataene i bildefilen (EXIF), og sammenligner med tidspunktet klokka ble startet og fristen.
   - Riktig gate og tatt i løpet: teller med en gang, og bildet lagres ikke.
   - Tatt før start eller etter fristen: teller ikke, og bildet lagres ikke. Spillederen kan godkjenne likevel.
   - Usikker gate eller ukjent tidspunkt: bildet lagres til spillederen har godkjent eller avvist det, og slettes da automatisk.
   - Fristen er slutten av klokka. Spillederen kan også sette et eget klokkeslett. Det er en toleranse på ett minutt.
3. **Poeng.** Opplastingen stenger. Gater bare én gruppe har, er erobret. Poengene er gatas lengde i meter innenfor sirkelen.
4. **Duell.** Portalen trekker en enkel øvelse for hver gate flere lag har funnet. Spillederen trykker på vinneren, og gata skifter farge på kartet.

### Øvelser

Spillet har 20 innebygde øvelser som krever lite eller ikke noe utstyr. Under **Oppsett → Øvelser i duellene** kan spillederen slå av øvelser og lage egne. Egne øvelser lagres i spillet (`meta/custom`) og huskes i nettleseren på maskinen, slik at de kan legges til i neste spill med ett trykk.

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
2. Last opp `index.html`, `vilkar.html`, `personvern.html`, `firebase-config.js`, `database.rules.json` og `README.md`.
3. **Settings → Pages → Deploy from a branch → main / (root).**
4. Etter et minutt ligger spillet på `https://DITT-BRUKERNAVN.github.io/erobre-nabolaget/`.

Verdiene i `firebase-config.js` er ikke hemmelige. Det er databasereglene som bestemmer hvem som får lese og skrive.

## Slik er dataene beskyttet

- Ingen deltakere logger inn med navn. Hver enhet får en anonym Firebase-bruker, og gruppene heter det spillederen kaller dem.
- Bare den som har spillederlenken, kan endre spillet, se gruppekodene og se alle bildene.
- En deltaker kan bare laste opp til sin egen gruppe, og bare med riktig gruppekode. Bildene til andre grupper kan de ikke se.
- Etter fase 2 kan ingen deltakere laste opp mer.
- Bare bilder som spillederen må se på, lagres. De skaleres ned til cirka 50–100 kB og slettes automatisk når spillederen har godkjent eller avvist dem. Metadata i bildefilen, som GPS-posisjon, fjernes før opplasting. Spillederen kan slette alt som er igjen, eller hele spillet, under **Oppsett → Rydd opp**.
- Tidskontrollen skjer på deltakerens enhet. En deltaker som er god med data, kan i teorien jukse med den. Den er ment som en hjelp, ikke som bevis.
- Si til deltakerne at de skal ta bilde av skiltet og ikke av folk. Bruker du spillet i skolen eller en organisasjon, bør dere vurdere om Firebase (Google, datalagring i EU) er i tråd med rutinene deres for personvern.

## Grenser i gratisplanen (Spark)

Realtime Database gir 1 GB lagring og 10 GB nedlasting per måned. Ett spill med fem grupper og 100 bilder bruker rundt 5–10 MB. Slett bildene etter spillet, så holder gratisplanen lenge.

## Teknisk

- Én fil: `index.html`. Leaflet, qrcode-generator, Firebase (compat 10.12.2) og Tesseract.js lastes fra CDN.
- Bakgrunnskart: Kartverkets topografiske norgeskart (`cache.kartverket.no`, fargekart i oppsettet og gråtonekart i spillet). Utenfor Norge, eller hvis Kartverket ikke svarer, brukes OpenStreetMap-fliser automatisk.
- Gater i Norge: Statens vegvesens vegdatabase (NVDB, API Les v4, `nvdbapiles.atlas.vegvesen.no`). Gatenavnene kommer fra adresseregisteret. Avkjørsler som låner gatenavnet (`sideveg`), og gang- og sykkelveger som følger gata, tas ikke med. Lisens: NLOD.
- Gater utenfor Norge, eller hvis NVDB ikke svarer: OpenStreetMap via Overpass. Tre servere prøves med fem sekunders mellomrom. Etter 45 sekunder gir spillet opp og viser en knapp for å prøve igjen.
- Gatene hentes bare én gang, når spillet lages, og lagres i Firebase. På selve spilldagen trengs verken NVDB eller Overpass. Gatene klippes mot sirkelen, og lengden regnes bare for delen som ligger innenfor.
- Datamodell: `games/{kode}/meta | groups | streets | claims/{gruppe}/{id} | imgs/{gruppe}/{id} | verdicts | duels | secret | teachers | members`.

### Testing lokalt med Firebase-emulator

Legg til `?emu` i adressen, for eksempel `http://127.0.0.1:8080/?emu`. Da kobler siden seg til Auth-emulatoren på port 9099 og Database-emulatoren på port 9000:

```
firebase emulators:start --only auth,database --project demo-erobre
```
