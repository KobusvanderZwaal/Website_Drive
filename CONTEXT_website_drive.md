# CONTEXT — Website Drive (coverband)

_Laatst bijgewerkt: 2026-06-10_

## 🎯 Doel van het project

Een website voor de coverband **Drive**. De site toont onder andere de
band-introductie en de **setlist** (repertoire), gegroepeerd per decennium.

Daarnaast wordt er een **Telegram-bot** opgezet zodat alle vier de bandleden
zelf eenvoudig wijzigingen in de setlist kunnen doorvoeren via een chat —
zonder code of GitHub-kennis.

## 🌐 Live & repositories

| Wat | Waar |
|-----|------|
| Live website | https://kobusvanderZwaal.github.io/Website_Drive/ |
| Website-repo | https://github.com/KobusvanderZwaal/Website_Drive (branch `main`) |
| Bot-repo | https://github.com/KobusvanderZwaal/Website_Drive_bot (branch `main`) |
| Hosting site | GitHub Pages (automatisch vanaf `main`) |
| Hosting bot | Railway.app (nog te koppelen) |

## 📁 Bestandsstructuur

### Lokale werkmap
`C:\Users\k.vd.zwaal\OneDrive - Oosterhoff Group\Projecten\AI\Website Drive\`

```
drive-band.html              # Bronbestand van de site (~1.1 MB). Wordt op
                             #   GitHub als index.html gepubliceerd.
CONTEXT_website_drive.md     # Dit document.
```

### Tijdelijke git-clone (blijft tussen sessies bestaan)
`%TEMP%\Website_Drive_clone\`  → bevat `index.html` (kopie van drive-band.html)
en wordt gebruikt om naar GitHub te committen/pushen.

### Bot-code
`%TEMP%\Drive_bot\` (lokaal) → gepusht naar repo `Website_Drive_bot`:

```
main.py            # De Telegram-bot (python-telegram-bot). Commands:
                   #   /start /help /setlist /toevoegen /verwijderen
requirements.txt   # python-telegram-bot==20.8, requests==2.31.0
Procfile           # worker: python main.py  (start-commando voor Railway)
```

## 🔑 Key decisions

- **Bronbestand = `drive-band.html`, gepubliceerd als `index.html`.** De site is
  één groot HTML-bestand; de setlist staat in een JavaScript-array waarin elk
  nummer een object is: `{ decade: '70s', title: "...", artist: '...' }`.
- **Setlist-telling staat op twee plekken** in de HTML ("X nummers") en moet bij
  elke wijziging meebewegen. De bot doet dit automatisch via regex.
- **Bestand te groot om volledig te lezen** (~1.1 MB) → gericht bewerken met
  Grep/targeted Read, niet in z'n geheel.
- **Bot commit via de GitHub Git Data API** (blob → tree → commit → ref) in
  plaats van de simpele Contents-API, omdat `index.html` groter is dan de 1 MB
  limiet van die simpele API.
- **Toegangscontrole via Telegram user-ID's** (`ALLOWED_USERS`). Alleen bekende
  ID's mogen de setlist wijzigen; iedereen kan via `/start` zijn eigen ID zien.
- **Hosting**: site op GitHub Pages (gratis), bot op Railway.app (gratis tier).
- **Secrets** (Telegram-token, GitHub PAT) staan NIET in deze repo — ze worden
  als environment variables in Railway gezet.

## 📝 Recente wijzigingen (al live)

Commit `1ab0dd9` — "Update setlist intro with actual bands, remove Fortunate Son":
- Intro-tekst aangepast naar bands die de band echt speelt
  (Deep Purple, Foo Fighters, The Cult, Triggerfinger i.p.v. AC/DC, Led Zeppelin, QOTSA).
- "Fortunate Son" (CCR) van de setlist gehaald.
- Tellerstand bijgewerkt van 57 → 56 nummers (beide plekken).

## ✅ Status Telegram-bot

| Stap | Status |
|------|--------|
| 1. Telegram-bot aangemaakt via @BotFather (token binnen) | ✅ |
| 2. GitHub Personal Access Token aangemaakt | ✅ |
| 3. Botcode geschreven + gepusht naar `Website_Drive_bot` | ✅ |
| 4. Bot deployen op Railway.app | ⬜ TODO |
| 5. Environment variables in Railway zetten | ⬜ TODO |
| 6. Eigen Telegram-ID ophalen + als beheerder instellen | ⬜ TODO |
| 7. Andere drie bandleden toevoegen aan `ALLOWED_USERS` | ⬜ TODO |

## 🚧 Openstaande taken

1. **Bot live zetten op Railway** (zie "Wat moet ik nu doen").
2. **`ALLOWED_USERS` vullen** met de Telegram-ID's van de vier bandleden.
3. **Bot testen**: `/setlist`, een nummer `/toevoegen`, `/verwijderen`.
4. (Optioneel) De vier bandleden een korte instructie sturen hoe ze de bot
   gebruiken.

## ▶️ Wat moet ik nu (morgen) doen

**Railway koppelen — ±3 minuten:**

1. Ga naar **railway.app** → "Login with GitHub".
2. **New Project** → "Deploy from GitHub repo" → kies **`Website_Drive_bot`** → **Deploy Now**.
3. Open de service → tabblad **Variables** → voeg toe:
   - `TELEGRAM_TOKEN` = de bottoken van @BotFather (begint met `8830872874:AAG...`)
   - `GITHUB_TOKEN` = de GitHub PAT (begint met `ghp_...`)
   - `ALLOWED_USERS` = (voorlopig leeg laten)
4. Deploy laten draaien.
5. Stuur **`/start`** naar je bot in Telegram → je krijgt je **Telegram-ID** terug.
6. Zet dat ID (komma-gescheiden voor meerdere bandleden) in de variabele
   `ALLOWED_USERS` en deploy opnieuw.
7. Test met `/setlist`.

> 🔒 **Let op:** de Telegram-token en GitHub-token staan in de chatgeschiedenis
> van deze sessie, niet in deze repo. Bewaar ze veilig. Vernieuwen kan altijd:
> bottoken via `/revoke` bij @BotFather, GitHub-token via GitHub → Settings →
> Developer settings.

## 🧰 Handige commando's

```powershell
# Site-wijziging pushen (vanuit de temp-clone):
Copy-Item "C:\Users\k.vd.zwaal\OneDrive - Oosterhoff Group\Projecten\AI\Website Drive\drive-band.html" "$env:TEMP\Website_Drive_clone\index.html"
Set-Location "$env:TEMP\Website_Drive_clone"
git add index.html
git commit -m "Omschrijving van de wijziging"
git push
```

De setlist zelf wijzigt straks het makkelijkst via de Telegram-bot.
