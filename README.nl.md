# smbsync-updater

🇳🇱 Nederlands — [English version](README.md)

Deze repository bevat de actuele **smbsync**-binaries. smbsync houdt een
SMB-share in de gaten (bijv. een StyleShoots-machine) en synchroniseert
fotomappen naar Photopilot.

## Installatie

> Draai je nog **v25 of ouder**? Die versies updaten zichzelf niet. Installeer
> één keer handmatig de nieuwste versie volgens de stappen hieronder; daarna
> gaat het vanzelf.

1. Download de binary voor jouw platform van de
   [**laatste release**](https://github.com/photopilot/smbsync-updater/releases/latest)
   (niet uit de map `smbsync/` in deze repo — die stopt bij v25):
   - `windows-amd64.exe` — Windows 64-bit (meest gebruikt, ondertekend)
   - `windows-386.exe` — Windows 32-bit
   - `darwin-arm64` / `darwin-amd64` — macOS (Apple Silicon / Intel)
   - `linux-amd64` / `linux-arm64` — Linux
2. Zet de binary in een **eigen map** (bijv. `C:\smbsync\`) en hernoem hem
   naar `smbsync.exe`. De map moet schrijfbaar zijn voor het account waaronder
   smbsync draait: het logbestand en de self-update schrijven daar.
3. Maak in **dezelfde map** een `.env`-bestand:

```
CUSTOMER=Bedrijfsnaam
SMB_IP=00.00.00.00
SMB_USERNAME=styleshootssharing
SMB_PASSWORD=
SMB_MOUNT=StyleshootsDrive
START_DIR=test
API_TOKEN=
API_URL=
LOG_LEVEL=info
TZ=Europe/Amsterdam
```

   **`CUSTOMER` is verplicht en moet per installatie uniek zijn** (bijv.
   `Van Dijk`, of `Van Dijk studio 2` bij meerdere machines). Onder die naam
   verschijnt de installatie in de monitoring; twee installaties met dezelfde
   naam verbergen elkaars storingen.

4. Start smbsync. Het blijft draaien en controleert de share elke 15 seconden.

### Windows: als geplande taak

Zo blijft smbsync draaien na een herstart van de server, ook zonder dat er
iemand is ingelogd:

1. Taakplanner → **Taak maken**.
2. *Algemeen*: "Uitvoeren ongeacht of gebruiker is aangemeld".
3. *Triggers*: alleen "Bij opstarten". Stel **geen** herhalende trigger in: na
   een self-update draait smbsync los van de taak verder, en een herhaling zou
   dan een tweede exemplaar ernaast starten (dubbele uploads).
4. *Acties*: programma `C:\smbsync\smbsync.exe`, "Beginnen in" `C:\smbsync`.
5. *Instellingen*: "Als de taak al wordt uitgevoerd: **geen nieuw exemplaar
   starten**", en vink "Taak stoppen als deze langer wordt uitgevoerd dan" **uit**.

### Controleren of het werkt

- In de map verschijnt `smbsync.log`. De eerste regels horen te zijn:
  `Starting smbsync currentVersion=<nummer>` en `Connected to SMB share`.
- Geef de gekozen `CUSTOMER`-naam door aan Photopilot; wij zien de installatie
  dan binnen 5 minuten in onze monitoring.
- Werkt het niet? Stuur `smbsync.log` mee — daar staat de precieze fout in
  (verkeerd wachtwoord, share onbereikbaar, `.env` niet gevonden, …).

### Netwerk

smbsync heeft uitgaand HTTPS (poort 443) nodig naar de Photopilot-API
(`API_URL`), `api.github.com` en `github.com` (updates) en
`*.ingest.de.sentry.io` (monitoring), plus poort 445 naar de SMB-share.

## Updates

Vanaf v26 update smbsync zichzelf: elke 30 minuten kijkt het naar de laatste
release hier, controleert de SHA256 en herstart tussen twee sync-rondes — nooit
midden in een upload. Zet `SELF_UPDATE=false` in `.env` om dat uit te schakelen.

## Hoe mappen worden gesynchroniseerd

- **Shoots** — mappen met de naam `S-<set id>` (bijv. `S-1234`) worden
  geüpload als shoot voor die set.
- **Packshots** — elke andere map wordt behandeld als artikel-identifier: een
  gescande **EAN/barcode** (bijv. `8712345678906`) of een **publiek
  artikel-ID** (bijv. `12345-001`). Het artikel moet al in Photopilot
  bestaan; onbekende mappen worden overgeslagen en elke 15 minuten opnieuw
  geprobeerd. Binnen een packshot-map worden afbeeldingen met `back`, `side`
  of `label` in de bestandsnaam als die weergave meegestuurd; elke andere
  afbeelding wordt de vooraanzicht-foto van een eigen packshot-opdracht.

Een map wordt pas gesynchroniseerd nadat er 10 minuten niets is gewijzigd,
zodat de fotograaf eerst kan uitfotograferen. Na een succesvolle synchronisatie
worden de bestanden en de map van de share verwijderd (mappen met RAW `.cr2`-
of videobestanden blijven staan).

## Monitoring

- **Heartbeat** — elke installatie meldt zich elke 5 minuten bij Photopilot
  (Sentry Crons, monitor `smbsync-<customer>`). Blijft dat ~15 minuten uit, of
  is de share zo lang onbereikbaar, dan krijgt Photopilot een melding.
- **Logs** — logregels gaan naar `smbsync.log` naast de executable (maximaal
  10 MB, daarna `smbsync.log.1`) én naar Photopilot, zodat wij kunnen meekijken
  zonder toegang tot de server. `LOG_FILE=<pad>` wijzigt de locatie,
  `LOG_FILE=off` schakelt het bestand uit.
- **Fouten** — fouten en crashes worden gerapporteerd aan Sentry. Zet
  `SENTRY_DSN` in `.env` om de standaard-DSN te overschrijven.
- Een weggevallen share beëindigt smbsync niet: het verbindt elke 30 seconden
  opnieuw.

## Nieuwe versie uitbrengen

Releases worden automatisch gebouwd, ondertekend en hier gepubliceerd zodra er
een wijziging in `smbsync/` op `main` van de Photopilot-monorepo landt. Het
versienummer is de laatste release + 1. De map `smbsync/` in deze repo bevat
alleen nog de oude versies t/m v25.
