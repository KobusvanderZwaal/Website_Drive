# CONTEXT — Website Drive (coverband)

_Laatst bijgewerkt: 2026-06-12_

## 🎯 Doel van het project

Een website voor de coverband **Drive**. De site toont onder andere de
band-introductie en de **setlist** (repertoire), gegroepeerd per decennium.

Daarnaast draait er een **Telegram-bot** zodat alle vier de bandleden zelf
wijzigingen kunnen doorvoeren via een chat — zonder code of GitHub-kennis:

- **Setlist**: via `/toevoegen` en `/verwijderen` — gaat **direct** live.
- **Rest van de website** (teksten, koppen, foto's): bandleden sturen gewoon
  een berichtje of foto in vrije taal. De bot verzamelt deze in een wachtrij
  en voert ze **elke avond om 22:00** in één keer door met behulp van Claude
  (AI), waarna een samenvatting naar de bandgroepschat gaat.

## 🌐 Live & repositories

| Wat | Waar |
|-----|------|
| Live website | https://kobusvanderZwaal.github.io/Website_Drive/ |
| Website-repo | https://github.com/KobusvanderZwaal/Website_Drive (branch `main`) |
| Bot-repo | https://github.com/KobusvanderZwaal/Website_Drive_bot (branch `main`) |
| Hosting site | GitHub Pages (automatisch vanaf `main`) |
| Hosting bot | Railway.app ✅ (live, met `.python-version` → Python 3.12) |

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
                   #   /start /help /chatid /setlist /toevoegen /verwijderen
                   #   /wachtrij /annuleer /publiceer
                   #   + vrije tekst & foto's (privéchat) → wachtrij
.python-version    # "3.12" — python-telegram-bot 20.8 is kapot op 3.13
requirements.txt   # python-telegram-bot[job-queue]==20.8, requests, anthropic, pytz
Procfile           # worker: python main.py  (start-commando voor Railway)
pending_changes.json  # (door de bot beheerd) wachtrij met wijzigingsverzoeken
```

## 🔑 Key decisions

- **Bronbestand = `drive-band.html`, gepubliceerd als `index.html`.** De site is
  één groot HTML-bestand; de setlist staat in een JavaScript-array waarin elk
  nummer een object is: `{ decade: '70s', title: "...", artist: '...' }`.
- **Setlist-telling staat op twee plekken** in de HTML ("X nummers") en moet bij
  elke wijziging meebewegen. De bot doet dit automatisch via regex.
- **De ~1,1 MB van het bestand zit in 10 base64-foto's.** Zonder die blobs is de
  HTML maar ~23 KB. De bot stript de foto's naar placeholders
  (`base64,__IMG_n__`) voordat hij de HTML aan Claude geeft, en zet ze daarna
  terug. Lokaal bewerken: gericht met Grep/targeted Read, niet in z'n geheel.
- **AI-bewerking via SEARCH/REPLACE-paren** (model: `claude-opus-4-8`,
  structured outputs met JSON-schema). De bot valideert dat elke zoekstring
  exact één keer voorkomt, dat alle foto-placeholders intact zijn en dat
  `</html>` nog bestaat — anders wordt er NIET gecommit en blijft de wachtrij
  staan.
- **Nieuwe foto's via Telegram** worden direct als los bestand gecommit naar
  `images/` in de website-repo (onzichtbaar tot ernaar verwezen wordt); de
  verwijzing (`<img src="images/...">`) gaat 's avonds mee met de batch.
- **Wachtrij persistent** als `pending_changes.json` in de bot-repo (Contents
  API) — overleeft Railway-herstarts.
- **Bot commit via de GitHub Git Data API** (blob → tree → commit → ref), omdat
  `index.html` groter is dan de 1 MB-limiet van de simpele Contents-API.
- **Toegangscontrole via Telegram user-ID's** (`ALLOWED_USERS`). Iedereen kan
  via `/start` zijn eigen ID zien.
- **Python 3.12 gepind** via `.python-version` — python-telegram-bot 20.8
  crasht op Python 3.13 (`AttributeError` in `Updater.__init__`).
- **Hosting**: site op GitHub Pages (gratis), bot op Railway.app.
- **Secrets** staan NIET in de repo's — als environment variables in Railway.

## ⚙️ Railway environment variables

| Variabele | Status | Toelichting |
|-----------|--------|-------------|
| `TELEGRAM_TOKEN` | ✅ | Bottoken van @BotFather |
| `GITHUB_TOKEN` | ✅ | GitHub PAT (classic, scope `repo`), juni 2026 opnieuw aangemaakt |
| `ALLOWED_USERS` | ✅ | `8990259893` (Kobus) — andere bandleden nog toevoegen |
| `ANTHROPIC_API_KEY` | ⬜ TODO | Nodig voor de AI-bewerking — bot crasht zonder |
| `GROUP_CHAT_ID` | ⬜ TODO | Chat-id van de bandgroep (via /chatid in de groep). Zolang leeg: samenvatting naar alle ALLOWED_USERS |
| `PUBLISH_HOUR` | optioneel | Standaard 22 (= 22:00 Europe/Amsterdam) |
| `CLAUDE_MODEL` | optioneel | Standaard `claude-opus-4-8` |

## ✅ Status

| Stap | Status |
|------|--------|
| Telegram-bot aangemaakt via @BotFather | ✅ |
| GitHub PAT aangemaakt | ✅ |
| Botcode geschreven + gepusht | ✅ |
| Bot live op Railway (Python 3.12 fix) | ✅ |
| Eigen Telegram-ID (8990259893) in ALLOWED_USERS | ✅ |
| /setlist getest en werkend | ✅ |
| AI-uitbreiding gepusht (vrije taal, foto's, avondbatch) | ✅ 2026-06-12 |
| `ANTHROPIC_API_KEY` in Railway zetten | ⬜ TODO |
| Bot in bandgroep + `GROUP_CHAT_ID` instellen | ⬜ TODO |
| AI-flow testen (berichtje sturen → /publiceer) | ⬜ TODO |
| Andere drie bandleden toevoegen aan `ALLOWED_USERS` | ⬜ TODO |

## ▶️ Wat moet ik nu doen

1. **Anthropic API-key aanmaken**: console.anthropic.com → API Keys →
   Create Key → in Railway als `ANTHROPIC_API_KEY` zetten.
   (Zonder deze key crasht de bot bij het opstarten!)
2. **Bot in de bandgroep zetten**: voeg de bot toe aan de Telegram-groep,
   stuur daar `/chatid`, en zet het getoonde (negatieve) id in Railway als
   `GROUP_CHAT_ID`.
3. **Testen**: stuur de bot privé een berichtje als "Verander de kop X naar Y",
   check `/wachtrij`, en voer direct door met `/publiceer`.
4. **Bandleden toevoegen**: laat ze `/start` sturen, voeg hun ID's
   komma-gescheiden toe aan `ALLOWED_USERS`.

## 🧰 Handige commando's

```powershell
# Site-wijziging handmatig pushen (vanuit de temp-clone):
Copy-Item "C:\Users\k.vd.zwaal\OneDrive - Oosterhoff Group\Projecten\AI\Website Drive\drive-band.html" "$env:TEMP\Website_Drive_clone\index.html"
Set-Location "$env:TEMP\Website_Drive_clone"
git add index.html
git commit -m "Omschrijving van de wijziging"
git push
```

> ⚠️ Let op: de bot wijzigt `index.html` op GitHub. Als je daarna lokaal
> handmatig pusht vanaf een oude `drive-band.html`, overschrijf je de
> bot-wijzigingen. Haal eerst de laatste versie op:
> `git -C "$env:TEMP\Website_Drive_clone" pull` en kopieer `index.html`
> terug naar `drive-band.html`.
