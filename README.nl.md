# smbsync-updater

🇳🇱 Nederlands — [English version](README.md)

Deze repository bevat de actuele **smbsync**-binaries. smbsync houdt een
SMB-share in de gaten (bijv. een StyleShoots-machine) en synchroniseert
fotomappen naar Photopilot.

## Installatie

1. Download de binary voor jouw platform uit `smbsync/<versie>/`:
   - `windows-amd64.exe.gz` — Windows 64-bit (meest gebruikt)
   - `windows-386.exe.gz` — Windows 32-bit
   - `darwin-arm64.gz` / `darwin-amd64.gz` — macOS (Apple Silicon / Intel)
   - `linux-amd64.gz` / `linux-arm64.gz` — Linux
2. Pak het `.gz`-bestand uit (op Windows met 7-Zip, op macOS/Linux met
   `gunzip <bestand>`).
3. Zet de binary in een eigen map en maak daarnaast een `.env`-bestand aan:

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

4. Start de binary. Het programma blijft draaien en controleert de share elke
   15 seconden.

## Hoe mappen worden gesynchroniseerd

- **Shoots** — mappen met de naam `S-<set id>` (bijv. `S-1234`) worden
  geüpload als shoot voor die set.
- **Packshots** — elke andere map wordt behandeld als artikel-identifier: een
  gescande **EAN/barcode** (bijv. `8712345678906`) of een **publiek
  artikel-ID** (bijv. `12345-001`, een optionele `P-`-prefix wordt
  verwijderd). Het artikel moet al in Photopilot bestaan; onbekende mappen
  worden overgeslagen en elke 15 minuten opnieuw geprobeerd. Binnen een
  packshot-map worden afbeeldingen met `back`, `side` of `label` in de
  bestandsnaam als die weergave meegestuurd; elke andere afbeelding wordt de
  vooraanzicht-foto van een eigen packshot-opdracht.

Een map wordt pas gesynchroniseerd nadat er 10 minuten niets is gewijzigd,
zodat de fotograaf eerst kan uitfotograferen. Na een succesvolle synchronisatie
worden de bestanden en de map van de share verwijderd (mappen met RAW `.cr2`-
of videobestanden blijven staan).

## Foutmonitoring

Fouten worden gerapporteerd aan Sentry. Zet `SENTRY_DSN` in het `.env`-bestand
om de standaard-DSN te overschrijven.

## Nieuwe versie uitbrengen

Vanuit de [smbsync](https://github.com/photopilot/smbsync)-repository:

```bash
./update-version.sh   # verhoog het versienummer in main.go
./release.sh          # bouw alle platforms naar ../smbsync-updater
cd ../smbsync-updater && git add . && git commit -m "Update binaries" && git push
```
