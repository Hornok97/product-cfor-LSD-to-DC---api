# Livesport AI Agent — Prototype Demo

Interaktivní simulace AI agenta postaveného na Claude API s přístupem k interním systémům Livesport (Elasticsearch, PostgreSQL, GitLab, Puppeteer screenshots) skrz MCP server.

**Účel:** Demo pro meeting s vedením — žádost o produkční přístup ke Claude API pro crawling/onboarding tým.

## Co demo ukazuje

- **Terminálové UI** stylizované jako CLI agenta
- **Streamované tool calls** — uživatel vidí v reálném čase, co agent volá a kam (Elastic, DB, GitLab)
- **Multi-source orchestrace** — jeden user dotaz spustí 3 paralelní volání do různých systémů
- **Detekce anomálií** — agent si sám všimne časové nesrovnalosti mezi insertem a check eventem
- **Confirmation flow** pro write operace (vytváření ticketu) — žádný silent zápis
- **Audit log** referencovaný u každé write operace

## Připravené scénáře

1. **Vyšetření zápasu 359845** — hlavní demo. Agent najde insert event, cml_check a aktivní providery, propojí to a upozorní na anomálii.
2. **Aktivita uživatele** — agregace insertů uživatele `janoseka` za 24h s rozpadem podle provideru
3. **Historie editů** — kompletní timeline změn na entitě
4. **Vytvoření ticketu** — write operace s confirmation flow před zápisem do GitLabu

## Spuštění

### Lokálně
Stačí otevřít `index.html` v prohlížeči. Žádný build, žádné dependencies.

```bash
open index.html
# nebo
python3 -m http.server 8000
```

### Na GitHub Pages (doporučeno pro prezentaci)

1. Push do repository
2. Settings → Pages → Source: `main` branch, root
3. Demo bude dostupné na `https://hornok97.github.io/product-cfor-LSD-to-DC---api/`

GitHub Actions workflow v `.github/workflows/pages.yml` to deployne automaticky.

## Jak prezentovat

**Otevři demo na velké obrazovce.** První věc, kterou uvidíš, je čistý terminál se 4 scénáři dole.

1. Klikni **„1. Vyšetřit zápas 359845"** — dotaz se sám naťuká, vypadá to filmově
2. Komentuj během běhu:
   - „Agent rozdělil můj dotaz na tři paralelní volání do různých systémů — Elastic logy, DB"
   - „Tady si sám všiml, že check je o 11 měsíců starší než insert — to není v žádném SQL dashboardu, to je interpretace dat"
   - „Screenshoty z puppeteer alertů má rovnou linknuté"
3. Pak klikni **„4. Ticket s confirmation"** — ukaž confirmation flow:
   - „Pro produkční zápisy vždy potvrzujeme. AI nikdy nezasáhne sama."
   - Klikni „potvrdit" → uvidí se audit log

## Co tohle není

Tohle je **demo**, ne funkční systém. Žádné reálné volání API, žádný skutečný Elastic, data jsou hardcoded v JS. V realitě by za tím stál:

- **MCP server** (Node.js) vystavující tooly `search_logs`, `query_db`, `get_user`, `fetch_screenshot`, `search_gitlab`
- **Backend** (Node.js + Anthropic SDK) řešící autentizaci, audit, rate limiting
- **Auth integrace** s vaším internim SSO a Vault
- **Per-role allowlist** toolů (produkťák ≠ ops)

## Architektura cílového produktu

```
Slack/CLI ──▶ Backend (Node.js + Anthropic SDK)
                   │
                   ├─▶ Audit log (append-only)
                   ├─▶ Auth/RBAC
                   │
                   ▼
              MCP Server
                   │
       ┌───────────┼───────────┐
       ▼           ▼           ▼
  Elasticsearch  Postgres    GitLab API
  (crawler logs) (DB)        (tickets)
```

## Tech stack v demu

- Pure HTML + CSS + JS, žádný framework
- Inter (UI) + JetBrains Mono (terminál) z Google Fonts
- Zero dependencies, zero build step

## Otázky pro vedení

- Můžeme získat Claude API access (Anthropic Console, Teams plan minimum)?
- Kdo bude product owner? (návrh: někdo z crawling/onboarding teamu)
- Pilot scope: read-only operace pro 3 uživatele × 2 týdny, pak rozšíření?
- Budget odhad: ~$200-500/měsíc na pilotní fázi (Sonnet 4.6, ~1k konverzací)
