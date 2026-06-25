# CONTEXT: Drive Band Website Bot

_Bijgewerkt: 25 juni 2026_

## Project overzicht

Coverband **Drive** heeft een statische website op GitHub Pages. Een Telegram-bot (@website_drive_bot) stelt bandleden in staat om via chat de website direct aan te passen — zowel de setlist als vrije HTML-wijzigingen. Claude (claude-opus-4-8) doet de feitelijke HTML-bewerkingen.

---

## Repositories

| Repo | URL | Doel |
|------|-----|------|
| Website | https://github.com/KobusvanderZwaal/Website_Drive | `index.html` op GitHub Pages (branch: main) |
| Bot | https://github.com/KobusvanderZwaal/Website_Drive_bot | `main.py`, `Procfile`, `requirements.txt`, `.python-version` |

### Context-bestand
Dit bestand (`CONTEXT_website_drive.md`) staat in de **Website**-repo.

---

## Railway — werkend project

| Gegeven | Waarde |
|---------|--------|
| Projectnaam | **Website-Drive-bot** (hernoemd van "awake-radiance" op 25 jun 2026) |
| Project-ID | `b650e6fd-7573-4ab5-a843-a6329a5f8090` |
| Service | `worker` |
| Service-ID | `0b41a6b7-e9c5-4048-b393-ab74416242d6` |
| Environment-ID | `64b3339b-ee03-4846-8d76-65100af9acef` |
| Region | US West |
| Runtime | python@3.12.13 |
| Start | `Procfile`: `worker: python main.py` |

### Railway-omgevingsvariabelen (service "worker")

| Variabele | Status | Toelichting |
|-----------|--------|-------------|
| `TELEGRAM_TOKEN` | ✅ ingesteld | Bot-token van BotFather |
| `GITHUB_TOKEN` | ✅ ingesteld | Persoonlijk GitHub-token met repo-schrijfrechten |
| `ALLOWED_USERS` | ✅ ingesteld | Kommagescheiden Telegram user-ID's |
| `ANTHROPIC_API_KEY` | ✅ ingesteld | Anthropic API-sleutel |
| `GROUP_CHAT_ID` | ❌ **ontbreekt** | Moet nog worden ingesteld: `-5349146756` |
| `CLAUDE_MODEL` | optioneel | Standaard: `claude-opus-4-8` |

> **TODO**: Voeg `GROUP_CHAT_ID = -5349146756` toe via Railway → Variables → New Variable, dan Deploy.

### Oud/kapot project (negeren)

`dazzling-kindness` (project-ID: `ec6061e6-b84a-4def-88e2-22430956453b`) — dit project heeft verkeerde variabelen en kan worden verwijderd.

---

## Bot-architectuur (`main.py`)

```
Telegram bericht (privé of groep)
        │
        ▼
  Toegangscheck (ALLOWED_USERS)
        │
   ┌────┴────┐
Commando   Vrije tekst / Foto
   │              │
Setlist-    Claude (claude-opus-4-8)
functies    via tools + tool_choice
   │              │
GitHub      zoek/vervang op index.html
Pages             │
              commit_html() → GitHub API
```

### Commando's

| Commando | Functie |
|----------|---------|
| `/start` | Welkomstbericht + uitleg |
| `/help` | Alle commando's uitleggen |
| `/chatid` | Toont chat-ID (handig voor groep-setup) |
| `/setlist` | Toont huidige setlist |
| `/toevoegen Artiest \| Titel \| Decennium` | Nummer toevoegen (direct live) |
| `/verwijderen Titel` | Nummer verwijderen (direct live) |

### Vrije tekst / foto
- Elk bericht in privéchat of bandgroep → Claude bewerkt `index.html` direct
- Foto met bijschrift → foto opgeslagen als `images/foto_YYYYMMDD_HHMMSS.jpg` in Website-repo, dan HTML bijgewerkt
- Website is ~1 minuut na commit live

### Technische details
- Claude-aanroep: `tools` + `tool_choice={"type":"tool","name":"html_edits"}` (stabiele structured output)
- Grote base64-afbeeldingen worden gestript vóór Claude-aanroep en daarna teruggezet
- `anthropic.Anthropic()` client wordt aangemaakt in `claude_edit()`, niet bij module-load

---

## Bekende bugs (opgelost)

| Bug | Oorzaak | Oplossing |
|-----|---------|-----------|
| `KeyError: 'ANTHROPIC_API_KEY'` bij startup | `anthropic.Anthropic()` op module-niveau, key ontbrak | Client nu lazy (in `claude_edit()`) |
| `AttributeError: filters.NEVER` | Bestaat niet in PTB 20.8 | Vervangen door `chat_filter = filters.ChatType.PRIVATE` + optioneel `\| filters.Chat(...)` |
| `thinking={"type":"adaptive"}` + `output_config` | Niet stabiel in SDK | Vervangen door `tools` + `tool_choice` |

---

## Openstaande taken

- [ ] **GROUP_CHAT_ID instellen** in Railway (waarde: `-5349146756`) → bot reageert dan ook in de bandgroep
- [ ] **AI-flow testen**: stuur een testbericht naar @website_drive_bot (privé), bijv. "Verander de intro-tekst naar: ..." — controleer of de site ~1 minuut later bijgewerkt is
- [ ] **Andere bandleden toevoegen**: hun Telegram-ID's ophalen (stuur `/start` naar de bot) en toevoegen aan `ALLOWED_USERS` in Railway

---

## Hoe verder in een nieuwe chat

1. Lees dit bestand: `CONTEXT_website_drive.md` in repo `KobusvanderZwaal/Website_Drive`
2. Lees de huidige bot-code: `main.py` in repo `KobusvanderZwaal/Website_Drive_bot`
3. Ga naar Railway-project **Website-Drive-bot** om status te controleren
4. Werk de openstaande taken hierboven af
