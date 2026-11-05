# MCP: Model Context Protocol  

*Verzia 1.1 · stav k októbru 2026. MCP servery sa rýchlo menia: pribúdajú  
a menia sa nástroje, inštalačné príkazy aj nastavenia. Pred použitím si  
ich overte v dokumentácii konkrétneho servera.*  

## 1. Čo je MCP  

### Definícia  

**Model Context Protocol (MCP)** je otvorený štandard, ktorý určuje, ako  
AI aplikácia komunikuje s vonkajším svetom: s nástrojmi, dátami  
a službami. Je to *protokol*, teda dohoda o tom, kto s kým hovorí,  
v akom poradí a v akom tvare. Konkrétne MCP stanovuje:  

- **role**: aplikácia s modelom (host) sa cez klienta pripája k serverom,  
  ktoré sprístupňujú služby,  
- **čo môže server ponúknuť**: nástroje (akcie), zdroje (dáta) a šablóny  
  promptov, a ako ich klient zistí a použije,  
- **formát správ**: JSON-RPC 2.0,  
- **spôsob prenosu**: lokálne cez štandardný vstup a výstup (stdio),  
  vzdialene cez HTTP,  
- **prihlasovanie** k vzdialeným serverom: OAuth.  

Jednou vetou: MCP je spoločný jazyk, ktorým sa AI aplikácia pýta služby  
„čo vieš robiť?“ a „urob toto“, a služba odpovedá v dohodnutom tvare.  

MCP samo o sebe nie je program. Je to špecifikácia (dokument na  
modelcontextprotocol.io), ktorej verzie sa označujú dátumom, napríklad  
`2025-11-25`. K nej existujú oficiálne knižnice (SDK) pre Python,  
TypeScript, Javu, C#, Go a ďalšie jazyky. S nimi sa server alebo klient  
dá napísať na pár desiatok riadkov.  

### Aký problém rieši  

Jazykový model sám vie len čítať a písať text. Aby mohol prečítať e-mail,  
nájsť súbor alebo zapísať do databázy, aplikácia ho musí s danou službou  
prepojiť. Pred MCP si každá aplikácia písala vlastné prepojenie na každú  
službu:  

```
  Bez MCP: každá aplikácia má vlastné prepojenie na každú službu
    Claude  → GitHub, Gmail, databáza
    ChatGPT → GitHub, Gmail, databáza
    Cursor  → GitHub, Gmail, databáza          spolu 3 × 3 = 9 prepojení

  S MCP: každá aplikácia má MCP klienta, každá služba MCP server
    Claude, ChatGPT, Cursor → MCP ← GitHub, Gmail, databáza
                                               spolu 3 + 3 = 6 implementácií
```

Pri 5 aplikáciách a 20 službách to bez štandardu znamená až 100  
integrácií. S MCP stačí 5 klientov a 20 serverov. Výhodné je to pre obe  
strany:  

- **poskytovateľ služby** (GitHub, Supabase, Notion…) napíše jeden MCP  
  server a funguje v Claude, ChatGPT, Copilote, Cursore aj inde,  
- **tvorca aplikácie** implementuje MCP raz a získa prístup k tisícom  
  existujúcich serverov,  
- **používateľ** si vyberá aplikáciu aj služby nezávisle od seba.  

MCP sa preto často prirovnáva k **USB-C pre AI**. Jeho priamym vzorom  
bol Language Server Protocol (LSP), vďaka ktorému jedna podpora  
programovacieho jazyka funguje v mnohých editoroch.  

### MCP a volanie nástrojov (tool use)  

Moderné modely vedia požiadať o zavolanie funkcie (*tool use*, *function  
calling*). Aplikácia im pošle zoznam nástrojov s popisom a schémou  
parametrov. Model odpovie, ktorý nástroj chce zavolať a s akými  
argumentmi. Aplikácia nástroj vykoná a výsledok pošle späť modelu.  

MCP na tomto mechanizme stavia a rieši otázku, **odkiaľ sa nástroje  
berú**:  

- **bez MCP** programátor definuje nástroje priamo v kóde aplikácie  
  (ako v príklade `claude_tools.py`) a fungujú len v nej,  
- **s MCP** si aplikácia zoznam nástrojov vypýta od servera počas behu  
  a ten istý server môže použiť ktorákoľvek iná aplikácia.  

Model o MCP nič nevie. Vidí len zoznam nástrojov. Preklad medzi volaním  
nástroja a MCP správou robí host.  

### Odkiaľ pochádza  

- **November 2024**: MCP zverejnila firma Anthropic ako otvorený  
  štandard spolu s prvými servermi a podporou v Claude Desktop.  
- **2025**: MCP prijali aj OpenAI, Google a Microsoft. Vznikli tisícky  
  serverov a oficiálny register serverov (MCP Registry).  
- **December 2025**: Anthropic odovzdala MCP neutrálnej organizácii  
  **Agentic AI Foundation** pri Linux Foundation. Odvtedy ho spravuje  
  komunita viacerých firiem.  

### Čo MCP nie je  

- **Nie je to model ani agent.** Server väčšinou žiadnu umelú inteligenciu  
  neobsahuje. Rozhoduje model v hostiteľskej aplikácii.  
- **Nie je to náhrada API.** Server zvyčajne obaľuje existujúce API  
  služby, napríklad GitHub MCP volá GitHub REST API.  
- **Nie je to bezpečnostná vrstva.** Protokol sám nezaručí, že server je  
  dôveryhodný ani že model urobí len to, čo chcete (pozri kapitolu 9).  
- **Nie je to jediná cesta.** Agent môže pracovať aj cez vlastné funkcie,  
  príkazový riadok alebo skripty v skilloch (pozri kapitolu 3).  

### Kto MCP používa  

- **Hostiteľské aplikácie**: Claude (web, Desktop, Claude Code), ChatGPT,  
  GitHub Copilot vo VS Code, Cursor, Gemini CLI a mnohé ďalšie.  
- **Servery**: od veľkých firiem (GitHub, Supabase, Stripe, Notion,  
  Figma, Atlassian) cez komunitné projekty (Blender, Desktop Commander)  
  až po servery, ktoré si napíšete sami.  

### Čo sa v kapitole naučíte  

- ako MCP funguje vo vnútri (host, klient, server, JSON-RPC),  
- čím sa MCP líši od konektorov, pluginov a skillov,  
- kde zistiť, aké MCP servery máte zapnuté,  
- ako pracovať so servermi Desktop Commander, Blender, Exa, GitHub  
  a Supabase,  
- aké bezpečnostné riziká MCP prináša a ako ich zmenšiť,  
- ako v Pythone napísať vlastný MCP server, klienta a agenta,  
- ako vlastný server pripojiť ku Claude Code, Claude Desktop, OpenCode  
  a GitHub Copilot.  

## 2. Ako MCP funguje  

### Tri role  

```
  Host (Claude Desktop, Claude Code, VS Code, váš program)
   ├── MCP klient ──── MCP server: poznámky kurzu
   ├── MCP klient ──── MCP server: GitHub
   └── MCP klient ──── MCP server: Blender ──── Blender
```

- **Host** je aplikácia, s ktorou pracujete. Obsahuje jazykový model  
  a rozhoduje, čo model smie.  
- **Klient** je časť hostu, ktorá udržiava spojenie s jedným serverom.  
  Pre každý server je jeden klient.  
- **Server** je program, ktorý sprístupňuje konkrétnu službu alebo dáta.  

### Čo server ponúka  

| Prvok | Kto ho používa | Príklad |  
|---|---|---|  
| **Tools** (nástroje) | model sa sám rozhodne zavolať | `search_notes`, `execute_sql` |  
| **Resources** (zdroje) | aplikácia ich načíta ako kontext | obsah súboru, schéma databázy |  
| **Prompts** (šablóny) | používateľ si ich vyberie | „vytvor kvíz z kapitoly“ |  

V praxi sa najviac používajú nástroje. Niektoré hosty zdroje a šablóny  
zatiaľ nepodporujú vôbec alebo len čiastočne.  

Komunikácia funguje aj opačným smerom. Server môže požiadať klienta  
napríklad o zoznam povolených priečinkov (*roots*), o doplňujúci údaj od  
používateľa (*elicitation*) alebo o odpoveď jazykového modelu (*sampling*).  

### Ako sa správy prenášajú  

- **stdio**: host spustí server ako proces na vašom počítači a posiela mu  
  správy cez štandardný vstup a výstup. Takto bežia Desktop Commander,  
  Blender aj náš server z kapitoly 10.  
- **Streamable HTTP**: server beží na internete a host sa k nemu pripája  
  cez HTTPS, zvyčajne s prihlásením cez OAuth. Takto fungujú vzdialené  
  servery Exa, GitHub a Supabase. Starší transport SSE sa už nepoužíva.  

Samotné správy sú vo formáte **JSON-RPC 2.0**. Každé spojenie prebieha  
v rovnakom poradí:  

1. `initialize`: klient a server sa dohodnú na verzii protokolu a na tom,  
   čo kto podporuje,  
2. `tools/list`: klient zistí názvy, popisy a JSON schémy nástrojov,  
3. `tools/call`: model zavolá nástroj s konkrétnymi argumentmi.  

Ukážka požiadavky a odpovede:  

```json
{"jsonrpc": "2.0", "id": 3, "method": "tools/call",
 "params": {"name": "read_note", "arguments": {"name": "../.bashrc"}}}

{"jsonrpc": "2.0", "id": 3, "result": {"isError": true,
 "content": [{"type": "text", "text": "../.bashrc is not a note in examples/"}]}}
```

Všimnite si, že chyba nástroja nie je chyba protokolu. Prichádza ako  
bežný výsledok s `isError: true`, aby si ju model mohol prečítať a zmeniť  
postup.  

### MCP Inspector  

Najlepší nástroj na pochopenie MCP je **MCP Inspector**. V prehliadači  
ukáže každú správu, ktorú klient a server vymenia, a umožní ručne zavolať  
ľubovoľný nástroj:  

```bash
npx @modelcontextprotocol/inspector uv run mcp_server.py ~/kurz/poznamky
```

Pri samotnom používaní servera v Claude JSON-RPC správy neuvidíte.  
Inspector je preto vhodný na výučbu aj na hľadanie chýb vo vlastnom  
serveri.  

## 3. MCP, konektory, pluginy a skilly  

Tieto pojmy sa často zamieňajú, pretože všetky „rozširujú“ AI asistenta.  
Každý však rieši inú vec a pracuje na inej úrovni.  

### Konektor  

**Konektor** (*connector*) je v aplikáciách Claude (claude.ai, Claude  
Desktop, mobilná aplikácia) prepojenie s externou službou, ktoré zapnete  
v **Settings → Connectors**, napríklad Gmail, Google Drive alebo Exa.  

Technicky je konektor väčšinou **vzdialený MCP server** zabalený pre  
bežného používateľa: nájdete ho v katalógu, prihlásite sa jedným  
kliknutím a vo firemných plánoch ho môže povoliť alebo zakázať správca.  
Ak konektor v katalógu nie je, pridáte ho ako *custom connector* zadaním  
adresy MCP servera. Konektory z claude.ai sa po prihlásení zobrazia aj  
v Claude Code (napríklad `claude.ai Gmail`).  

Lokálnym náprotivkom sú **rozšírenia pre Claude Desktop** (balíky  
`.mcpb`): lokálny MCP server, ktorý sa nainštaluje jedným kliknutím.  

Pozor na význam slova v iných nástrojoch: v Make.com alebo Zapieri je  
„konektor“ ich vlastný integračný modul, nie MCP.  

### Plugin  

**Plugin** je v Claude Code **balík rozšírení**, ktorý sa nainštaluje  
jedným príkazom (`/plugin`) z *marketplace*, čo je zvyčajne Git  
repozitár s katalógom. Plugin môže obsahovať:  

- skilly,  
- vlastné príkazy (`/príkaz`),  
- subagentov,  
- hooky (automatické akcie pri udalostiach),  
- MCP servery.  

Plugin teda nie je nový protokol, ale **spôsob distribúcie**. Napríklad  
plugin `playwright` obsahuje MCP server na ovládanie prehliadača. Preto  
ho `claude mcp list` zobrazí ako `plugin:playwright:playwright`.  

Historická poznámka: v roku 2023 mal ChatGPT vlastné „pluginy“ postavené  
na OpenAPI. Fungovali len v ChatGPT a v roku 2024 ich OpenAI zrušila.  
MCP rieši rovnakú potrebu, ale ako otvorený štandard pre všetky  
aplikácie.  

### Skill  

**Skill** (Agent Skill) je **priečinok s návodom**. Jeho jadrom je súbor  
`SKILL.md` s názvom, popisom a inštrukciami. Môže obsahovať aj skripty,  
šablóny a podklady. Anthropic skilly predstavila v októbri 2025  
a v decembri 2025 ich zverejnila ako otvorený štandard, ktorý odvtedy  
podporujú aj ďalšie nástroje.  

Agent má v kontexte spočiatku len názov a popis každého skillu. Celý  
návod si načíta, až keď ho úloha potrebuje (*postupné načítanie*). Preto  
môžete mať desiatky skillov bez toho, aby zbytočne zaberali kontext.  
Definície nástrojov MCP servera sa naproti tomu zvyčajne načítajú celé  
hneď na začiatku.  

Skill dáva agentovi **znalosti a postup**, nie prístup. Príklady:  

- skill na faktúry vie, aké údaje faktúra musí mať a ako vypočítať DPH,  
- skill `imhd-odchody` vie, kde na imhd.sk nájsť odchody MHD a ako ich  
  prečítať. Samotný prehliadač mu však sprístupňuje MCP server Playwright.  

Posledný príklad ukazuje, ako sa tieto prvky dopĺňajú: **MCP dáva ruky,  
skill dáva návod.**  

### Porovnanie  

| | MCP server | Konektor | Plugin | Skill | Vlastné nástroje (tool use) |  
|---|---|---|---|---|---|  
| **Čo to je** | program podľa otvoreného protokolu | MCP server pripravený v aplikácii Claude | balík rozšírení pre Claude Code | priečinok s návodom a skriptmi | funkcie v kóde vašej aplikácie |  
| **Čo agentovi dáva** | prístup a akcie | prístup a akcie | čokoľvek z obsahu balíka | znalosti a postup | prístup a akcie |  
| **Kde beží** | lokálne alebo na serveri | na serveri poskytovateľa | v Claude Code | v prostredí agenta | vo vašom programe |  
| **Kto ho tvorí** | poskytovateľ, komunita, vy | poskytovateľ služby | ktokoľvek | ktokoľvek, aj neprogramátor | programátor aplikácie |  
| **Kde funguje** | v každej MCP aplikácii | v aplikáciách Claude | v Claude Code | v nástrojoch so štandardom Agent Skills | len vo vašom programe |  
| **Príklad** | `mcp-for-blender` | Gmail, Google Drive | `playwright` | `imhd-odchody` | `claude_tools.py` |  

Jednoduchá predstava z dielne: **MCP server** sú nástroje a kľúče od  
dverí, **konektor** je nástroj už zapojený do zásuvky, **skill** je  
pracovný postup a **plugin** je krabica, v ktorej to celé dostanete.  

### Kedy čo použiť  

- Agent sa potrebuje **dostať k službe alebo dátam** → MCP server, alebo  
  konektor, ak pre službu existuje.  
- Agent prístup má, ale robí veci **zle alebo inak, než chcete** → skill.  
- Chcete celé riešenie **odovzdať kolegom alebo študentom** jedným  
  príkazom → plugin.  
- Píšete **vlastnú aplikáciu** s jednou-dvoma funkciami → vlastné  
  nástroje priamo v kóde. MCP sa oplatí, až keď chcete nástroje použiť  
  aj v iných aplikáciách.  

## 4. Kde zistíte, aké MCP servery máte  

### Claude Code  

- V rozhovore napíšte `/mcp`. Uvidíte zoznam serverov, ich stav a môžete  
  ich znova pripojiť, prihlásiť sa k nim alebo ich vypnúť.  
- V termináli spustite `claude mcp list`. Podrobnosti o jednom serveri  
  zobrazí `claude mcp get <názov>`.  
- Nastavenia sú v súboroch:  
  - `~/.claude.json`: vaše servery pre všetky projekty (*user scope*)  
    a servery pre jednotlivé projekty (*local scope*, predvolené),  
  - `.mcp.json` v koreňovom priečinku projektu: servery zdieľané s  
    ostatnými, ktorí na projekte pracujú (*project scope*).  
- Konektory z claude.ai sa po prihlásení zobrazia tiež. Spravujú sa na  
  claude.ai v **Settings → Connectors**.  

Ukážka výstupu `claude mcp list`:  

```
claude.ai Gmail: https://gmailmcp.googleapis.com/mcp/v1 - ✔ Connected
claude.ai Exa: https://mcp.exa.ai/mcp - ✔ Connected
plugin:playwright:playwright: npx @playwright/mcp@latest - ✔ Connected
blender: /home/jano/.local/bin/uvx mcp-for-blender - ✔ Connected
```

### Claude Desktop  

- **Settings → Connectors**: vzdialené servery a konektory z claude.ai.  
- **Settings → Extensions**: rozšírenia, ktoré sa inštalujú jedným  
  kliknutím (balíky `.mcpb`).  
- **Settings → Developer → Edit Config**: otvorí súbor  
  `claude_desktop_config.json` s lokálnymi servermi:  
  - macOS: `~/Library/Application Support/Claude/claude_desktop_config.json`  
  - Windows: `%APPDATA%\Claude\claude_desktop_config.json`  

Po zmene konfigurácie treba Claude Desktop úplne ukončiť a znova spustiť.  
Nestačí zavrieť okno.  

### VS Code (GitHub Copilot)  

Servery pre GitHub Copilot sa nastavujú v súbore `.vscode/mcp.json` alebo  
príkazom **MCP: Add Server**. Zoznam a stav serverov ukáže príkaz  
**MCP: List Servers**. Podrobnosti sú v kapitole o VS Code.  

### OpenCode a GitHub Copilot CLI  

- OpenCode: `opencode mcp list`. Servery sú v súbore `opencode.json`  
  v projekte alebo v `~/.config/opencode/` (pre všetky projekty).  
- Copilot CLI: `copilot mcp list`. Servery sú v súbore  
  `~/.copilot/mcp-config.json` alebo v `.mcp.json` v projekte.  

## 5. Desktop Commander  

### Čo to je  

**Desktop Commander** (`@wonderwhy-er/desktop-commander`) je komunitný  
MCP server, ktorý z Claude Desktop robí niečo ako kódovacieho agenta.  
Okrem práce so súbormi totiž vie spúšťať príkazy v termináli.  

```bash
npx @wonderwhy-er/desktop-commander@latest setup
```

### Čo vie  

| Oblasť | Príklady |  
|---|---|  
| Súbory | čítanie, zápis, cielená úprava bloku textu, rýchle hľadanie (ripgrep) |  
| Terminál | spustenie príkazu, dlho bežiace procesy (REPL, dev server, SSH) a posielanie vstupu do nich |  
| Procesy | zoznam bežiacich procesov, ich ukončenie |  
| Nastavenia | povolené priečinky, zakázané príkazy, predvolený shell |  

### Riziko: spúšťanie príkazov  

Server, ktorý spúšťa príkazy, má v praxi rovnaké oprávnenia ako váš  
používateľský účet:  

- Nastavenie `allowedDirectories` platí len pre súborové nástroje.  
  Príkaz `cat ~/.ssh/id_rsa` ho obíde. Dokumentácia to sama uvádza.  
- Zoznam zakázaných príkazov (`blockedCommands`) sa dá ľahko obísť, napr.  
  inou cestou k programu, cez iný interpret alebo cez skript.  
- Model teda môže čítať vaše kľúče, posielať dáta na internet aj mazať  
  súbory, napríklad ak ho k tomu navedie skrytý pokyn v dokumente alebo  
  na webovej stránke (*prompt injection*).  

### Kedy ho použiť  

- Len keď v Claude Desktop **naozaj potrebujete spúšťať príkazy**.  
- Na programovanie na vlastnom počítači je bezpečnejší **Claude Code**.  
  Má rovnaké schopnosti a systém povolení, ktorý sa pýta pri každom  
  príkaze.  
- Ak agent potrebuje len čítať dokumenty, zvoľte server s **užším  
  rozsahom**, napríklad konektor Google Drive alebo vlastný server len  
  na čítanie (kapitola 10).  
- Nástroj na spúšťanie príkazov nenechávajte na „vždy povoliť“. Žiadosť  
  „zapíš súbor X“ posúdite ľahko, dlhý príkaz pre terminál oveľa horšie.  
- So študentmi ho používajte vo virtuálnom počítači alebo kontajneri.  

## 6. Blender MCP  

### Čo to je  

**MCP for Blender** (balík `mcp-for-blender`, predtým `blender-mcp`, autor  
Siddharth Ahuja) prepája jazykový model so spusteným programom Blender.  
Nevytvoril ho Blender Foundation. Skladá sa z dvoch častí:  

- **doplnok v Blenderi**, ktorý po spustení Blendera otvorí lokálny  
  socket server (predvolene port 9876),  
- **MCP server**, ktorý beží cez stdio a prekladá volania nástrojov na  
  príkazy pre doplnok.  

### Inštalácia  

```bash
uvx mcp-for-blender setup             # nastaví klientov aj doplnok
# alebo ručne:
claude mcp add blender uvx mcp-for-blender
uvx mcp-for-blender install-addon
```

Potom v Blenderi zapnite doplnok v **Edit → Preferences → Add-ons**  
(Interface: MCP for Blender). Panel doplnku nájdete v 3D zobrazení po  
stlačení klávesu `N`.  

### Hlavné nástroje  

| Nástroj | Čo robí |  
|---|---|  
| `get_scene_info` | stručný prehľad scény, jeden riadok na objekt |  
| `execute_blender_code` | spustí ľubovoľný Python kód (`bpy`) v Blenderi |  
| `look` | snímka zobrazenia, kamery alebo renderu, model „vidí“ výsledok |  
| `search_assets`, `import_asset` | modely a textúry z Poly Haven, Sketchfab, Poly Pizza |  
| `generate_3d` | vygeneruje 3D model z textu alebo obrázka (externé služby, môže stáť kredit) |  

Príklad zadania: „Vytvor jednoduchú scénu so stolom a lampou, nastav  
teplé svetlo a ukáž mi výsledok z kamery.“ Model si zistí stav scény,  
napíše Python kód, spustí ho a cez `look` skontroluje, či výsledok sedí.  
Je to pekná ukážka agentovej slučky: konaj, pozri sa, oprav.  

### Na čo si dať pozor  

- `execute_blender_code` spúšťa **ľubovoľný Python kód**. Python v Blenderi  
  má prístup k celému počítaču (súbory, sieť, iné programy). Pred prácou  
  si uložte projekt.  
- Server zbiera anonymné štatistiky používania. Úplne ich vypnete  
  premennou prostredia `DISABLE_TELEMETRY=true` v konfigurácii servera.  
  Zdieľanie promptov a snímok je predvolene vypnuté.  
- Naraz má bežať len jedna inštancia servera.  

## 7. Exa MCP  

### Čo to je  

**Exa** je vyhľadávač určený pre AI. Namiesto zoznamu odkazov vracia  
priamo obsah stránok, ktorý model vie spracovať. Exa prevádzkuje  
vzdialený MCP server:  

```bash
claude mcp add --transport http exa https://mcp.exa.ai/mcp
```

Na claude.ai ho pridáte ako konektor v **Settings → Connectors**.  

### Hlavné nástroje  

- `web_search_exa`: vyhľadá stránky podľa popisu toho, čo hľadáte,  
- `web_fetch_exa`: načíta obsah konkrétnej adresy,  
- ďalšie nástroje podľa nastavenia, napr. na viackrokový prieskum.  

### Kedy ho použiť  

Claude má aj vlastné vyhľadávanie na webe. Exa sa oplatí, keď hľadáte  
skôr podľa významu ako podľa kľúčových slov („firmy, ktoré robia  
X v Strednej Európe“), pri zostavovaní zoznamov a pri prieskume  
s mnohými zdrojmi. Nad bezplatný limit sa platí podľa počtu dopytov.  

### Riziko  

Obsah z webu je **nedôveryhodný vstup**. Stránka môže obsahovať skrytý  
pokyn pre model. Ak má ten istý agent aj prístup k súborom alebo  
e-mailu, vzniká nebezpečná kombinácia (pozri kapitolu 9).  

## 8. GitHub MCP a Supabase MCP  

### GitHub MCP  

Oficiálny server od GitHubu (`github/github-mcp-server`) sprístupňuje  
repozitáre, issues, pull requesty, GitHub Actions a bezpečnostné  
upozornenia. Môže bežať vzdialene (prihlásenie cez OAuth alebo osobný  
token) alebo lokálne v Dockeri:  

```bash
claude mcp add --transport http github https://api.githubcopilot.com/mcp/ \
  -H "Authorization: Bearer $GITHUB_PAT"
```

Dôležité nastavenia:  

- **Toolsets**: zapnete len skupiny nástrojov, ktoré potrebujete (napr.  
  `repos`, `issues`, `pull_requests`). Menej nástrojov znamená menej  
  chýb aj menší kontext.  
- **Režim len na čítanie** (`--read-only` pri lokálnom serveri, hlavička  
  `X-MCP-Readonly: true` pri vzdialenom).  
- **Fine-grained token** obmedzený na konkrétne repozitáre namiesto  
  tokenu s prístupom ku všetkému.  

Známy útok z roku 2025: útočník napísal do verejného repozitára issue so  
skrytým pokynom. Agent, ktorý mal prístup aj k súkromným repozitárom,  
issue prečítal, pokyn vykonal a obsah súkromného repozitára zverejnil  
v pull requeste. Nebola to chyba v kóde servera, ale v tom, ako široké  
oprávnenia agent dostal.  

V Claude Code často stačí príkaz `gh` (GitHub CLI), ktorý Claude Code  
vie používať priamo. MCP server sa viac oplatí v Claude Desktop, na  
claude.ai a vo vlastných agentoch.  

### Supabase MCP  

**Supabase** je platforma s databázou PostgreSQL, prihlasovaním  
a úložiskom. Jej oficiálny MCP server umožňuje agentovi pracovať  
s projektom: zobraziť tabuľky (`list_tables`), spustiť SQL  
(`execute_sql`), vytvoriť migráciu (`apply_migration`), čítať logy  
a hľadať v dokumentácii.  

```bash
claude mcp add --transport http supabase \
  "https://mcp.supabase.com/mcp?project_ref=<id-projektu>&read_only=true"
```

Pri prvom použití sa prihlásite cez prehliadač (OAuth).  

Odporúčania samotného Supabase:  

- **Nepripájajte produkčnú databázu.** Používajte vývojový projekt  
  alebo vetvu (*branch*) s testovacími dátami.  
- Zapnite **režim len na čítanie** (`read_only=true`).  
- Obmedzte server na **jeden projekt** (`project_ref`).  
- Kontrolujte každé volanie nástroja, najmä SQL príkazy.  

Známy útok z roku 2025: do tabuľky so zákazníckymi požiadavkami sa  
dostal text so skrytým pokynom. Vývojár požiadal agenta, aby požiadavky  
zhrnul. Agent mal plný prístup k databáze, pokyn vykonal a citlivé tokeny  
zapísal späť do požiadavky, kde si ich útočník prečítal.  

## 9. Bezpečnosť MCP  

Každý MCP server je kód, ktorý beží s vašimi oprávneniami, alebo služba,  
ktorá dostane prístup k vašim dátam. Platia tieto pravidlá:  

- **Smrteľná trojica** (*lethal trifecta*, Simon Willison). Najväčšie  
  riziko vzniká, keď má agent naraz:  
  1. prístup k súkromným dátam,  
  2. kontakt s nedôveryhodným obsahom (web, e-maily, issues, tikety),  
  3. možnosť posielať dáta von (internet, e-mail, komentár, PR).  

  Útoky na GitHub a Supabase vyššie sú presne tento prípad. Odstráňte  
  aspoň jednu z troch zložiek.  
- **Najmenšie oprávnenia.** Režim len na čítanie, obmedzené priečinky,  
  projekty a repozitáre, tokeny s úzkym rozsahom.  
- **Schvaľovanie krokov.** Citlivé nástroje (zápis, mazanie, SQL, príkazy)  
  nenechávajte na „vždy povoliť“.  
- **Dôveryhodný zdroj.** Inštalujte servery od známych autorov. Pri  
  `npx` a `uvx` sa pri každom spustení môže stiahnuť nová verzia. Pri  
  dôležitých nasadeniach používajte pevnú verziu.  
- **Tajomstvá patria do premenných prostredia**, nie do súborov, ktoré  
  zdieľate (napr. `.mcp.json` v repozitári).  
- **Menej je viac.** Každý server pridáva definície nástrojov do  
  kontextu a zvyšuje šancu, že model použije nesprávny nástroj. Zapínajte  
  len servery, ktoré pri danej práci potrebujete, a nezapínajte naraz  
  dva servery s podobnými nástrojmi.  

## 10. Príklady v Pythone  

Štyri krátke skripty ukazujú MCP z troch strán: server, klient a agent.  
Všetky sa spúšťajú cez `uv`, ktorý sám nainštaluje potrebné balíky.  
Nájdete ich v priečinkoch `mcp/proverbs/` a `mcp/notes/`. Príkazy  
spúšťajte z priečinka, v ktorom je daný skript.  

| Súbor | Čo ukazuje | Potrebuje |  
|---|---|---|  
| `proverbs/proverbs_server.py` | najjednoduchší MCP server: dva nástroje | `uv` |  
| `notes/mcp_server.py` | server s nástrojmi, zdrojom a šablónou, kontrola ciest | `uv` |  
| `notes/mcp_client.py` | čo robí host: `initialize`, `tools/list`, `tools/call` | `uv` |  
| `notes/mcp_agent.py` | Claude používa nástroje z MCP servera | `uv`, `ANTHROPIC_API_KEY` |  

Poznámka k verziám: od verzie 2 Python balíka `mcp` sa trieda `FastMCP`  
volá `MCPServer` (`from mcp.server.mcpserver import MCPServer`). Staršie  
návody s `FastMCP` fungujú len s `mcp<2`.  

### Príklad 1: najjednoduchší MCP server (príslovia)  

`proverbs_server.py` vracia slovenské príslovia aj s ich významom. Má len  
dva nástroje a nepotrebuje žiadne nastavenie, preto je dobrým prvým  
príkladom:  

```python
mcp = MCPServer("slovak-proverbs", version="1.0")

@mcp.tool(annotations=READ_ONLY)
def random_proverb() -> str:
    """Return one random Slovak proverb with its meaning."""
    return show(*random.choice(PROVERBS))

@mcp.tool(annotations=READ_ONLY)
def search_proverbs(word: str) -> str:
    """Find Slovak proverbs that contain a word, in the proverb or its meaning.

    Case and diacritics are ignored, so "prace" finds "práce".
    """
    ...

if __name__ == "__main__":
    mcp.run()  # stdio
```

Čo si všimnúť:  

- Každý nástroj je obyčajná funkcia s dekorátorom. JSON-RPC, schému  
  parametrov aj komunikáciu cez stdio rieši knižnica.  
- Docstring je popis nástroja pre model. Podľa neho sa model rozhoduje,  
  kedy nástroj použije a čo doň zadá.  
- `readOnlyHint=True` oznamuje hostu, že nástroj nič nemení. Je to len  
  informácia, nie ochrana. Server ju musí aj sám dodržať.  
- Vyhľadávanie ignoruje veľké písmená aj diakritiku, pretože model aj  
  ľudia píšu raz s mäkčeňmi, raz bez nich. Hľadá aj časti slov, takže  
  `kon` nájde „koňovi“, ale aj „na konci“.  

Vyskúšanie v MCP Inspectori:  

```bash
cd mcp/proverbs
npx @modelcontextprotocol/inspector uv run proverbs_server.py
```

V prehliadači kliknite na **Connect**, potom na kartu **Tools**,  
**List Tools**, vyberte nástroj a kliknite na **Run Tool**. Prvé  
spustenie stiahne balík `mcp`, ďalšie sú už rýchle.  

### Pripojenie servera k aplikáciám  

Server prísloví je lokálny server (stdio). Každá aplikácia si ho spúšťa  
sama tým istým príkazom:  

```bash
uv run --quiet /plna/cesta/k/proverbs_server.py
```

Líši sa len to, kde a v akom tvare príkaz zaregistrujete. Používajte  
**absolútne cesty**, pretože aplikácie nespúšťajú servery z vášho  
projektu. V príkazoch nižšie je cesta k skriptu v premennej `$P`:  

```bash
P=/home/jana/kurz/mcp/proverbs/proverbs_server.py      # Linux, macOS
$P = "C:\Users\jana\kurz\mcp\proverbs\proverbs_server.py"   # PowerShell
```

| Aplikácia | Pridanie | Konfiguračný súbor | Kľúč v súbore |  
|---|---|---|---|  
| Claude Code | `claude mcp add` | `~/.claude.json`, `.mcp.json` | `mcpServers` |  
| Claude Desktop | úprava súboru | `claude_desktop_config.json` | `mcpServers` |  
| OpenCode | `opencode mcp add` | `opencode.json` | `mcp` |  
| GitHub Copilot vo VS Code | **MCP: Add Server** | `.vscode/mcp.json` | `servers` |  
| GitHub Copilot CLI | `copilot mcp add` | `~/.copilot/mcp-config.json`, `.mcp.json` | `mcpServers` |  

#### Claude Code  

```bash
claude mcp add proverbs -- uv run --quiet $P              # len tento projekt
claude mcp add -s user proverbs -- uv run --quiet $P      # všetky projekty
```

Všetko za `--` je príkaz na spustenie servera. Stav overíte príkazom  
`/mcp` v Claude Code alebo `claude mcp list` v termináli.  

Bez `-s` sa server uloží do *local scope*: funguje len v adresári, kde ste  
ho pridali. V inom projekte ho Claude Code nenačíta. Ak ste ho takto už  
pridali a chcete ho mať všade, odoberte ho z local scope a pridajte znova  
s `-s user`:  

```bash
claude mcp remove -s local proverbs
claude mcp add -s user proverbs -- uv run --quiet $P
claude mcp get proverbs        # Scope: User config (available in all your projects)
```

Scope servera ukáže `claude mcp get <názov>`. Do `.mcp.json` v koreni  
projektu ho zapíše `-s project`, vtedy ho dostane každý, kto pracuje  
s repozitárom.  

#### Claude Desktop  

V **Settings → Developer → Edit Config** doplňte do  
`claude_desktop_config.json`:  

```json
{
  "mcpServers": {
    "proverbs": {
      "command": "/home/jana/.local/bin/uv",
      "args": ["run", "--quiet", "/home/jana/kurz/mcp/proverbs/proverbs_server.py"]
    }
  }
}
```

- Do `command` zadajte **plnú cestu k `uv`**. Claude Desktop často  
  nevidí premennú `PATH` z terminálu, najmä na macOS. Cestu zistíte  
  príkazom `which uv`, vo Windows `where uv`.  
- Vo Windows treba v JSON zdvojiť spätné lomky:  
  `"C:\\Users\\jana\\kurz\\mcp\\proverbs\\proverbs_server.py"`.  
- Claude Desktop úplne ukončite a znova spustite.  
- Stav servera a chybové hlásenia nájdete v **Settings → Developer**.  
  Nástroje servera sa zobrazia v ponuke nástrojov v okne rozhovoru.  

#### OpenCode  

```bash
opencode mcp add proverbs -- uv run --quiet $P             # projekt
opencode mcp add proverbs --global -- uv run --quiet $P    # všetky projekty
opencode mcp list
```

Príkaz zapíše do konfigurácie takýto záznam. Na rozdiel od ostatných  
aplikácií je v OpenCode celý príkaz jedno pole `command`:  

```json
{
  "mcp": {
    "proverbs": {
      "type": "local",
      "command": ["uv", "run", "--quiet", "/home/jana/kurz/mcp/proverbs/proverbs_server.py"],
      "enabled": true
    }
  }
}
```

#### GitHub Copilot vo VS Code  

V priečinku projektu vytvorte súbor `.vscode/mcp.json`. Hlavný kľúč je  
tu `servers`, nie `mcpServers`:  

```json
{
  "servers": {
    "proverbs": {
      "type": "stdio",
      "command": "uv",
      "args": ["run", "--quiet", "${workspaceFolder}/mcp/proverbs/proverbs_server.py"]
    }
  }
}
```

- `${workspaceFolder}` je priečinok otvorený vo VS Code, takže cesta  
  funguje na každom počítači.  
- Server spustíte tlačidlom **Start** nad záznamom v súbore alebo  
  príkazom **MCP: List Servers**.  
- V okne Copilot Chat prepnite režim na **Agent**. Nástroje servera  
  nájdete v zozname nástrojov. Pri prvom volaní sa Copilot spýta  
  o povolenie.  

#### GitHub Copilot CLI  

```bash
copilot mcp add proverbs -- uv run --quiet $P
copilot mcp list
```

Príkaz zapíše server do `~/.copilot/mcp-config.json`. Copilot CLI číta  
aj súbor `.mcp.json` v projekte, teda rovnako pomenovaný súbor, aký  
používa Claude Code.  

#### Vyskúšanie  

V ktorejkoľvek aplikácii zadajte napríklad:  

- „Povedz mi náhodné slovenské príslovie.“ (model zavolá  
  `random_proverb`)  
- „Aké príslovia poznáš o práci?“ (zavolá `search_proverbs` so slovom  
  `práca`)  
- „Vyber príslovie, ktoré sa hodí k tomu, že som zmeškal autobus.“  
  (model vyhľadáva viackrát a vyberie najvhodnejšie)  

#### Ak to nefunguje  

- **Server sa nespustí alebo aplikácia nenájde príkaz**: zadajte plnú  
  cestu k `uv` aj ku skriptu.  
- **Prvé spustenie skončí po časovom limite**: `uv` ešte sťahuje balíky.  
  Spustite server raz v MCP Inspectori a skúste to znova.  
- **Spojenie sa hneď preruší**: niečo píše na stdout. MCP server smie na  
  stdout posielať len JSON-RPC správy, ladiace výpisy patria na stderr.  
- **Nástroje sa nezobrazia**: reštartujte aplikáciu alebo znova pripojte  
  server (`/mcp` v Claude Code, **MCP: List Servers** vo VS Code).  

### Príklad 2: server s poznámkami kurzu  

`mcp_server.py` sprístupňuje poznámky (súbory `.md`) len na čítanie.  
Priečinok s poznámkami dostane ako parameter príkazového riadka, podobne  
ako iné servery dostávajú zoznam povolených priečinkov:  

```bash
uv run mcp_server.py ~/kurz/poznamky
```

Bez parametra alebo s neexistujúcim priečinkom server skončí s chybou  
a nezačne bežať. Je to bezpečnejšie, ako keby potichu sprístupnil  
nejaký predvolený priečinok.  

Server obsahuje všetky tri druhy prvkov:  

```python
mcp = MCPServer("course-notes", version="1.0")

@mcp.tool(annotations=READ_ONLY)
def search_notes(query: str, max_results: int = 20) -> str:
    """Find lines in the notes that contain a word or phrase."""
    ...

@mcp.resource("notes://index", mime_type="text/markdown")
def index() -> str:
    """Table of contents of all notes."""
    ...

@mcp.prompt()
def quiz(name: str, questions: int = 5) -> str:
    """Create a short quiz from one note."""
    ...
```

Z typov parametrov a z docstringu knižnica sama vytvorí JSON schému  
nástroja. Docstring je zároveň popis, podľa ktorého sa model rozhoduje,  
či nástroj použije. Píšte ho preto pre model, nie len pre programátora.  

Najdôležitejšia časť je kontrola cesty:  

```python
def note_path(name: str) -> Path:
    path = (NOTES / name).resolve()
    if path.parent != NOTES or path.suffix != ".md" or not path.is_file():
        raise ToolError(f"{name} is not a note in {NOTES.name}/")
    return path
```

Kontrola sa vyhýba dvom častým chybám, ktoré sa v roku 2025 našli aj  
v známych MCP serveroch:  

- **Porovnávanie ciest ako textu.** Ak je povolený priečinok  
  `/home/jana/kurz`, kontrola „cesta začína na `/home/jana/kurz`“  
  pustí aj `/home/jana/kurz_tajne`. Preto sa porovnávajú celé cesty.  
- **Symbolické odkazy.** Odkaz v povolenom priečinku môže smerovať  
  kamkoľvek. `resolve()` odkazy rozbalí a odstráni aj `..`, takže ani  
  odkaz, ani `../tajne.md` nevedú von.  

`ToolError` je očakávaná chyba: model dostane jej text a môže pokračovať  
inak. Pri inej výnimke dostane len všeobecnú správu.  

Server komunikuje cez stdout, preto **nikdy nepoužívajte `print()`**.  
Ladiace výpisy posielajte na stderr.  

Vyskúšanie v MCP Inspectori a pridanie do Claude Code:  

```bash
npx @modelcontextprotocol/inspector uv run mcp_server.py ~/kurz/poznamky
claude mcp add notes -- uv run --quiet /plna/cesta/k/mcp_server.py /plna/cesta/k/poznamkam
```

Do ostatných aplikácií server pridáte rovnako ako server prísloví.  
Rozdiel je len v tom, že za cestu ku skriptu pridáte ešte cestu  
k poznámkam, v JSON ako ďalší prvok poľa `args`. Aj tu používajte  
**absolútne cesty**, Claude Desktop napríklad spúšťa servery  
z priečinka `/`.  

### Príklad 3: MCP klient  

`mcp_client.py` robí to, čo host robí v pozadí. Spustí náš server,  
pripojí sa k nemu a prejde základné kroky protokolu. Žiadny jazykový  
model tu nie je, každé volanie je jedna JSON-RPC požiadavka.  

```python
async with stdio_client(server) as (read, write):
    async with ClientSession(read, write) as session:
        init = await session.initialize()                    # initialize
        tools = (await session.list_tools()).tools           # tools/list
        index = await session.read_resource("notes://index") # resources/read
        result = await session.call_tool(                    # tools/call
            "read_note", {"name": "../.bashrc"})
```

```bash
uv run mcp_client.py                    # poznámky = kapitoly kurzu
uv run mcp_client.py ~/kurz/poznamky    # iný priečinok
```

Skrátený výstup:  

```
Server: course-notes 1.0
Protocol: 2025-11-25

Tools (3):
  list_notes     List the course notes with their titles.
  search_notes   Find lines in the notes that contain a word or phrase ...
  read_note      Read the beginning of one note.

Resource notes://index (30 notes): - AGENTS (AGENTS.md) ...
Prompts: quiz

-> search_notes({'query': 'MCP', 'max_results': 5}) [ok]
ai-agenti.md:243: ## 8. Ako agenti dostanú nástroje: MCP a ďalšie štandardy
...

-> read_note({'name': '../.bashrc'}) [ERROR]
Error executing tool read_note: ../.bashrc is not a note in examples/
```

Posledné volanie ukazuje hranicu prístupu v praxi: súbor mimo povoleného  
priečinka server odmietne.  

### Príklad 4: agent s MCP nástrojmi  

`mcp_agent.py` spája oba svety. Pripojí sa k nášmu serveru, prevedie jeho  
MCP nástroje na nástroje pre Claude a nechá SDK riadiť slučku agenta.  

```python
tools = (await mcp_client.list_tools()).tools

runner = client.beta.messages.tool_runner(
    model=MODEL,
    max_tokens=16000,
    tools=[async_mcp_tool(t, mcp_client) for t in tools],
    messages=[{"role": "user", "content": prompt}],
)
async for message in runner:
    ...
```

Funkcia `async_mcp_tool()` z balíka `anthropic` (inštalácia  
`anthropic[mcp]`) vytvorí z MCP nástroja nástroj pre Claude. Keď ho Claude  
zavolá, volanie sa odošle cez MCP na server. Tool runner opakuje kroky,  
kým Claude nemá odpoveď.  

```bash
uv run mcp_agent.py
uv run mcp_agent.py "Ktoré kapitoly hovoria o API kľúčoch?"
```

Ukážka výstupu:  

```
MCP tools: list_notes, search_notes, read_note

-> search_notes({'query': 'MCP', 'max_results': 100})
-> list_notes({})

MCP sa spomína v štyroch kapitolách:
1. Úvod do AI agentov (ai-agenti.md): ...
```

Ten istý server funguje bez zmeny v Claude Desktop, Claude Code aj  
v tomto skripte. Práve to je hlavná výhoda MCP.  

## 11. Úlohy na precvičenie  

1. Vlastnými slovami vysvetlite, čo je MCP a aký problém rieši. Prečo je  
   výhodné pre poskytovateľa služby aj pre tvorcu aplikácie?  
2. Zistite, aké MCP servery a konektory máte v Claude Code (`/mcp`)  
   a v Claude Desktop. Ku každému napíšte, k čomu má prístup.  
3. Pre každú úlohu rozhodnite, či by ste použili MCP server, konektor,  
   skill alebo plugin, a zdôvodnite to:  
   - agent má čítať e-maily v Gmaile,  
   - agent má písať správy podľa firemného štýlu,  
   - celý tím má dostať rovnaké nástroje a postupy jedným príkazom,  
   - agent má pracovať s internou databázou firmy.  
4. Pripojte server prísloví aspoň k dvom aplikáciám (napr. Claude Code  
   a GitHub Copilot vo VS Code). Porovnajte, kde a v akom tvare ste ho  
   zaregistrovali a ako sa aplikácie pýtajú o povolenie.  
5. Do servera prísloví pridajte nástroj `proverb_of_the_day()`, ktorý  
   v jeden deň vráti vždy rovnaké príslovie. (Tip: `random.Random`  
   s dátumom ako semienkom.)  
6. Spustite `mcp_server.py` v MCP Inspectori. Zavolajte `search_notes`  
   a potom `read_note` s cestou mimo priečinka. Opíšte JSON-RPC správy,  
   ktoré ste videli.  
7. Do `mcp_server.py` pridajte nástroj `count_words(name)`, ktorý vráti  
   počet slov v poznámke. Vyskúšajte ho v Inspectori aj cez  
   `mcp_agent.py`.  
8. V `note_path()` nahraďte kontrolu `path.parent != NOTES` kontrolou  
   `str(path).startswith(str(NOTES))` a odstráňte `resolve()`. Nájdite  
   vstup, ktorý sa dostane mimo priečinka.  
9. Diskusia: kedy je prístup k terminálu (Desktop Commander) hodný  
   rizika a kedy stačí server s užším rozsahom?  
10. Navrhnite nastavenie agenta, ktorý odpovedá na tikety zákazníkov  
    a pracuje s databázou v Supabase. Ktorú zložku smrteľnej trojice  
    odstránite a ako?  
11. V Blenderi požiadajte Claude o jednoduchú scénu. Sledujte, ako  
    striedavo volá `execute_blender_code` a `look`.  

## 12. Zhrnutie  

- **MCP** je otvorený štandard, ktorým sa AI aplikácia pripája k  
  nástrojom a dátam. Namiesto N × M integrácií stačí N + M.  
- MCP má tri role: **host**, **klient** a **server**. Server ponúka  
  **nástroje**, **zdroje** a **šablóny**.  
- Lokálne servery bežia cez **stdio**, vzdialené cez **Streamable  
  HTTP**. Správy sú vo formáte **JSON-RPC 2.0**.  
- **Konektor** je MCP server pripravený v aplikácii Claude, **plugin**  
  je balík rozšírení a **skill** je návod. MCP dáva agentovi prístup,  
  skill znalosti.  
- Servery skontrolujete cez `/mcp` a `claude mcp list` v Claude Code  
  a v **Settings** v Claude Desktop.  
- **Desktop Commander**, **Blender**, **Exa**, **GitHub** a **Supabase**  
  ukazujú rôzne druhy serverov: terminál, lokálnu aplikáciu, web, kód  
  a databázu.  
- Najväčšie riziko je **smrteľná trojica**: súkromné dáta,  
  nedôveryhodný obsah a možnosť posielať dáta von.  
- Vlastný server v Pythone má pár desiatok riadkov. Najdôležitejšia  
  je v ňom kontrola vstupov.  
- Ten istý server pripojíte ku **Claude Code**, **Claude Desktop**,  
  **OpenCode** aj **GitHub Copilot**. Líši sa len miesto a tvar  
  konfigurácie.  

## 13. Slovníček  

| Anglicky | Slovensky | Význam |  
|----------|-----------|--------|  
| protocol | protokol | dohoda o tom, ako spolu programy komunikujú |  
| host | hostiteľská aplikácia | aplikácia s modelom, napr. Claude Desktop |  
| client | klient | spojenie hostu s jedným serverom |  
| server | server | program, ktorý sprístupňuje nástroje a dáta |  
| tool | nástroj | funkcia, ktorú môže model zavolať |  
| resource | zdroj | dáta, ktoré aplikácia načíta ako kontext |  
| prompt | šablóna promptu | pripravené zadanie, ktoré vyberá používateľ |  
| transport | prenos | spôsob doručenia správ (stdio, HTTP) |  
| connector | konektor | MCP server pripravený v aplikácii Claude |  
| plugin | plugin, rozšírenie | balík skillov, príkazov a MCP serverov |  
| skill | skill, zručnosť | priečinok s návodom a skriptmi pre agenta |  
| roots | povolené priečinky | priečinky, ktoré klient serveru sprístupní |  
| prompt injection | vloženie pokynu | skrytý pokyn v dátach, ktorý ovládne model |  
| least privilege | najmenšie oprávnenia | dať len prístup, ktorý je nevyhnutný |  

## Zdroje  

- Špecifikácia a dokumentácia MCP: modelcontextprotocol.io  
- Register MCP serverov: registry.modelcontextprotocol.io  
- MCP Inspector: github.com/modelcontextprotocol/inspector  
- Python SDK: github.com/modelcontextprotocol/python-sdk  
- MCP, pluginy a skilly v Claude Code: code.claude.com/docs  
- Štandard Agent Skills: agentskills.io  
- Desktop Commander: github.com/wonderwhy-er/DesktopCommanderMCP  
- MCP for Blender: github.com/ahujasid/blender-mcp  
- GitHub MCP Server: github.com/github/github-mcp-server  
- Supabase MCP: supabase.com/docs/guides/getting-started/mcp  
- Exa MCP: docs.exa.ai  
- Simon Willison, „The lethal trifecta for AI agents“ (2025):  
  simonwillison.net  
