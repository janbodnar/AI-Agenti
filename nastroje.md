# Nástroje agentov: ako agent koná

*Verzia 1.0 · stav k októbru 2026. Názvy nástrojov, ich schémy a spôsob  
povoľovania sa menia s každou verziou. Pred použitím si ich overte  
v dokumentácii konkrétneho nástroja.*  

## 1. Ciele lekcie

Po tejto lekcii budete vedieť:  

- vysvetliť, čo je nástroj a prečo bez neho model len háda,  
- opísať cyklus volania nástroja od zadania po odpoveď,  
- prečítať schému nástroja a jeho parametre,  
- spustiť demonštrácie od najjednoduchšieho nástroja po MCP,  
- nájsť ekvivalentný nástroj v opencode, Copilot Chat a Claude Code,  
- nastaviť, čo agent smie urobiť sám a čo musí schváliť človek.  

Kapitola nadväzuje na [agenti-zaklady.md](agenti-zaklady.md) (čo je agent),  
[workflows.md](workflows.md) (opakovateľný postup),  
[prikazy.md](prikazy.md) (uložené zadania),  
[subagenti.md](subagenti.md) (delegovanie práce)  
a [mcp.md](mcp.md) (pripájanie externých služieb).  

## 2. Prečo nástroje

Jazykový model je uzavretý v texte. Nevidí váš priečinok, nepozná dnešný  
dátum, nevie, či testy prešli, a nedokáže poslať e-mail. Ak sa ho na to  
spýtate, odpovie z toho, čo sa naučil pri trénovaní: plynulo, sebavedomo  
a často nesprávne.  

Nástroj je most medzi modelom a svetom. Model povie „prečítaj tento  
súbor“ alebo „spusti tento príkaz“, aplikácia to vykoná a výsledok vráti  
späť do konverzácie. Až potom model odpovie používateľovi.  

Rozdiel je vidieť na jednej otázke:  

```text
Bez nástroja:
> Aký je dnes deň?
Dnes je piatok, 31. októbra 2026.        (model to len odhadol)

S nástrojom:
> Aký je dnes deň?
[nástroj: bash] {"command": "date +%A"}
[výsledok] Saturday
Dnes je sobota.
```

Prvá odpoveď vyzerá dobre, ale je to hádanie. Druhá je overená. Celý  
rozdiel medzi „chatbotom“ a „agentom“ sa začína tu: agent má nástroje  
a používa ich.  

## 3. Čo je nástroj

**Nástroj** (tool) je funkcia, ktorú model nevykonáva sám, ale vie o ňu  
požiadať. Skladá sa z troch častí:  

| Časť        | Kto ju vytvára | Príklad                                  |  
| ----------- | -------------- | ---------------------------------------- |  
| Deklarácia  | vývojár        | meno, popis a schéma parametrov          |  
| Volanie     | model          | `{"command": "date +%A"}`                |  
| Výsledok    | aplikácia     | text, štruktúrované dáta alebo chyba     |  

Deklaráciu dostane model v každom dotaze. Z nej sa dozvie, že nástroj  
existuje, kedy ho má použiť a aké parametre prijme. Volanie je len JSON;  
model nič nevykoná. Vykoná ho aplikácia — Claude Code, VS Code alebo  
opencode — a až tá rozhodne, či je volanie povolené.  

Z toho vyplýva prvé pravidlo bezpečnosti: **o právach nerozhoduje model,  
ale aplikácia.** Model môže navrhnúť `rm -rf`, ale aplikácia ho zastaví  
pred spustením. Preto sa povolenia nastavujú v konfigurácii, nie  
v prompte.  

## 4. Cyklus volania nástroja

Volanie nástroja nie je jeden krok, ale slučka. Vyzerá takto:  

```text
používateľ ──► model ──► požiadavka na nástroj ──► aplikácia  
                 ▲                                      │  
                 │                                      ▼  
                 └──────── výsledok ◄──────────── nástroj  
```

1. Používateľ napíše zadanie.  
2. Model odpovie buď textom, alebo požiadavkou na nástroj.  
3. Aplikácia overí povolenie a nástroj spustí.  
4. Výsledok sa vloží do konverzácie ako nová správa.  
5. Model pokračuje — volá ďalší nástroj, alebo odpovie.  

Slučka sa zastaví, keď model prestane žiadať nástroje. V praxi to  
znamená tri veci, ktoré sa hodia vedieť:  

- **Jeden ťah môže obsahovať viac volaní.** Agent prehľadá päť súborov  
  naraz, ak na sebe nezávisia.  
- **Každý výsledok stojí tokeny.** Čím väčší výstup nástroj vráti, tým  
  viac miesta zaberie v kontexte. Preto nástroje vracajú zhrnutia alebo  
  rozsahy riadkov, nie celé súbory.  
- **Každé volanie je viditeľné.** V Claude Code, VS Code aj opencode  
  vidíte, ktorý nástroj sa spustil a s akými parametrami. To je prvá vec,  
  ktorú kontrolujte, keď agent urobí niečo neočakávané.  

## 5. Anatómia nástroja

Deklarácia nástroja je obyčajný JSON objekt. Ukážme si jednoduchý nástroj  
na kurz meny:  

```json
{
  "name": "get_exchange_rate",
  "description": "Vráti kurz meny k euru pre daný deň. Použi, keď treba prepočítať sumu v cudzej mene.",
  "input_schema": {
    "type": "object",
    "properties": {
      "mena": {
        "type": "string",
        "description": "Kód meny podľa ISO 4217, napríklad USD alebo CZK."
      },
      "datum": {
        "type": "string",
        "description": "Dátum v tvare RRRR-MM-DD. Ak chýba, použi dnešný."
      }
    },
    "required": ["mena"]
  }
}
```

Čo je v deklarácii dôležité:  

- **`name`** — krátke, bez medzier, podľa akcie (`read_file`, `send_email`).  
- **`description`** — kedy nástroj použiť, nielen čo robí. Model sa podľa  
  nej rozhoduje medzi podobnými nástrojmi.  
- **`input_schema`** — typy, povinné polia a `enum` tam, kde má model  
  na výber z uzavretého zoznamu. Čím presnejšia schéma, tým menej  
  neplatných volaní.  
- **Výsledok** — aplikácia vráti text. Ak nastane chyba, vráti ju ako  
  text (`súbor neexistuje`), nie ako pád programu; model sa z nej má  
  poučiť.  

Dobrý popis je polovica úspechu. „Vráti kurz meny“ je málo. „Použi, keď  
treba prepočítať sumu v cudzej mene“ hovorí modelovi, kedy má nástroj  
vytiahnuť.  

## 6. Demonštrácie: od jednoduchého k zložitejšiemu

Nasledujúce príklady sú zámerne krátke. Každý ukazuje jeden nový prvok:  
argument, čítanie, hľadanie, zápis, web, delegovanie a externú službu.  
Formát je rovnaký pre všetky tri nástroje; mená nástrojov sa líšia,  
preto ich uvádzame všeobecne.  

### 6.1 Deň v týždni — jeden argument

Najjednoduchší nástroj, aký existuje: shell s jedným parametrom.  

```text
> Aký je dnes deň?

[nástroj: bash] {"command": "date +%A"}
[výsledok] Saturday

Dnes je sobota.
```

Čo sa naučíte: model nemá hodiny. Nástroj je jediný zdroj pravdy o stave  
sveta a jeho výstup patrí do odpovede. Volanie je viditeľné a dá sa  
skontrolovať.  

### 6.2 Zhrnutie pokynov — čítanie súboru

```text
> Zhrň mi, aké pravidlá mám v AGENTS.md.

[nástroj: read] {"filePath": "AGENTS.md"}
[výsledok] Text formátuj tak, aby sa lámal približne každých
           80 znakov. Každý riadok ukonči dvoma medzerami...

V súbore máte tri pravidlá: lámať text po 80 znakoch, ukončiť
každý riadok dvoma medzerami a formátovať tak aj tabuľky.
```

Čo sa naučíte: parameter je cesta k súboru. Nástroj vracia obsah, nie  
hotovú odpoveď — zhrnutie robí až model. Pri veľkých súboroch sa čítajú  
rozsahy riadkov, aby sa kontext nezaplnil.  

### 6.3 Hľadanie v projekte — dva nástroje naraz

```text
> Kde sú definované príkazy a ktoré používajú allowed-tools?

[nástroj: glob] {"pattern": "**/*.md"}
[výsledok] prikazy.md, subagenti.md, mcp.md, ...

[nástroj: grep] {"pattern": "allowed-tools", "include": "*.md"}
[výsledok] prikazy.md:281: allowed-tools: Read, Glob
           prikazy.md:305: allowed-tools: WebSearch, WebFetch
```

Čo sa naučíte: „kde je súbor“ a „čo je v súbore“ sú dva rôzne nástroje.  
Agent ich volá opakovane a nezávislé volania spúšťa paralelne. Výsledok  
hľadania má tvar `súbor:riadok`, takže sa dá hneď overiť.  

### 6.4 Oprava preklepu — zápis do súboru

```text
> Oprav v README.md preklep "instalace" na "inštalácia".

[nástroj: edit] {"filePath": "README.md",
                 "oldString": "instalace",
                 "newString": "inštalácia"}
[výsledok] Súbor upravený, 1 zmena.

[v aplikácii] diff: -instalace  +inštalácia
              [Povoliť úpravu? áno / nie / vždy]
```

Čo sa naučíte: nástroje na zápis menia stav a záleží na poradí volaní.  
Preto sa pýtajú na povolenie a ukazujú diff. `edit` nahradí presný text,  
`write` prepíše celý súbor — pri `write` je kontrola dôležitejšia.  

### 6.5 Rešerš — najprv hľadať, potom čítať

```text
> Zisti, koľko stojí Claude Pro v októbri 2026.

[nástroj: websearch] {"query": "Claude Pro cena október 2026"}
[výsledok] 5 odkazov: anthropic.com/pricing, ...

[nástroj: webfetch] {"url": "https://www.anthropic.com/pricing"}
[výsledok] Pro: 20 USD mesačne (výňatok stránky)

Claude Pro stojí 20 USD mesačne. Zdroj: anthropic.com/pricing.
```

Čo sa naučíte: `websearch` slúži na objavenie stránky, `webfetch` na  
prečítanie konkrétnej adresy. Obsah webu je cudzí text — môže obsahovať  
pokyny pre agenta (prompt injection), preto má nástroj na čítanie webu  
menej práv než nástroj na odosielanie správ.  

### 6.6 Delegovanie — nástroj, ktorý spustí iného agenta

```text
> Prejdi všetky markdown súbory a nájdi odkazy, ktoré neexistujú.

[nástroj: task] {"description": "kontrola odkazov",
                 "prompt": "Prejdi *.md, over odkazy, vráť zoznam."}
[výsledok] 3 nefunkčné odkazy: ... (len záver, bez medzikrokov)
```

Čo sa naučíte: subagent je tiež nástroj. Rodičovský agent dostane len  
výsledok, nie celú prácu vnútri — preto je to spôsob, ako ušetriť  
kontext. Platíte však latenciu a vlastný kontext subagenta.  

### 6.7 Externá služba — MCP

```text
> Naplánuj mi poradu na štvrtok o 14:00.

[nástroj: mcp__kalendar__vytvor_udalost]
         {"nazov": "Porada", "zaciatok": "2026-11-05T14:00"}
[výsledok] Udalosť vytvorená, ID 8f2a.

Porada je naplánovaná na štvrtok 5. novembra o 14:00.
```

Čo sa naučíte: MCP nástroje prichádzajú zo servera, ktorý musí byť  
pripojený. Meno zvyčajne obsahuje server (`mcp__kalendar__...` alebo  
`kalendar_vytvor_udalost`). Server je cudzí kód — povoľujte len tie,  
ktorým dôverujete, a začnite s právami na čítanie.  

## 7. Tie isté nástroje v troch aplikáciách

Mená sa líšia, úlohy nie. Tabuľka uvádza nástroje, ktoré pokrývajú bežnú  
prácu s projektom. Claude Code používa presné názvy aj v pravidlách  
povolení; Copilot Chat ich píše s `#`; opencode ich má v konfigurácii  
`permission`.  

| Úloha                      | opencode      | Copilot Chat             | Claude Code       |  
| -------------------------- | ------------- | ------------------------ | ----------------- |  
| prečítať súbor             | `read`        | `#read/readFile`         | `Read`            |  
| upraviť súbor              | `edit`        | `#edit/editFiles`        | `Edit`            |  
| vytvoriť súbor             | `write`       | `#edit/createFile`       | `Write`           |  
| nájsť súbory               | `glob`        | `#search/fileSearch`     | `Glob`            |  
| hľadať v texte             | `grep`        | `#search/textSearch`     | `Grep`            |  
| vypísať priečinok          | `list`        | `#search/listDirectory`  | `Bash`            |  
| spustiť príkaz             | `bash`        | `#execute/runInTerminal` | `Bash`            |  
| pozrieť zlyhané testy      | `bash`        | `#execute/testFailure`   | `Bash`            |  
| načítať webovú stránku     | `webfetch`    | `#web/fetch`             | `WebFetch`        |  
| hľadať na webe             | `websearch`   | — (cez rozšírenie)       | `WebSearch`       |  
| delegovať na subagenta     | `task`        | `#agent/runSubagent`     | `Agent`           |  
| sledovať úlohy             | `todowrite`   | `#todos`                 | `TaskList`        |  
| opýtať sa používateľa      | `question`    | `#vscode/askQuestions`   | `AskUserQuestion` |  
| načítať skill alebo príkaz | `skill`       | — (prompt files)         | `Skill`           |  
| diagnostika a chyby        | `lsp`         | `#read/problems`         | `LSP`             |  
| zmeny v gite               | `bash` (git)  | `#search/changes`        | `Bash`            |  
| externá služba cez MCP     | MCP nástroje  | MCP nástroje             | MCP nástroje      |  

Poznámky k tabuľke:  

- **Copilot Chat** má nástroje rozdelené na čítanie, úpravy, spúšťanie,  
  web a delegovanie; názvy v tabuľke sú referencie `#`, ktoré môžete  
  napísať priamo do správy. Dostupné nástroje závisia od verzie VS Code,  
  zvoleného agenta a rozšírení.  
- **opencode** ovláda nástroje cez `permission` v `opencode.json`;  
  `websearch` je dostupný len pri niektorých poskytovateľoch alebo po  
  zapnutí premennej prostredia.  
- **Claude Code** má okrem toho plánovací režim (`EnterPlanMode`),  
  poznámkový blok (`NotebookEdit`), PowerShell a prácu s prostrediami  
  (`EnterWorktree`). Nástroje, ktoré nie sú v zozname povolené, sa  
  opýtajú.  

## 8. Povolenia: čo agent smie

Každá z troch aplikácií rieši to isté: ktoré nástroje sa spustia bez  
otázky. Líši sa len miesto, kde sa to píše.  

**Claude Code.** Pravidlá sú v `settings.json` (allow, ask, deny) alebo  
v hlavičke príkazu a skillu cez `allowed-tools`. Pravidlo môže mať  
špecifikátor:  

```json
{
  "permissions": {
    "allow": ["Read", "Grep", "Bash(git status)"],
    "ask": ["Edit", "Write"],
    "deny": ["Bash(rm *)", "WebFetch"]
  }
}
```

**opencode.** Povolenie je jedna hodnota na nástroj — `allow`, `ask`  
alebo `deny`. Hviezdička pokrýva skupinu nástrojov, čo sa hodí pri MCP:  

```json
{
  "permission": {
    "edit": "deny",
    "bash": "ask",
    "webfetch": "allow",
    "kalendar_*": "ask"
  }
}
```

**Copilot Chat.** Povolenia sa nastavujú v rozhraní: pri každom volaní  
môžete povoliť alebo odmietnuť a zvoliť, či sa má voľba pamätať.  
Terminálové príkazy majú osobitné ovládanie, pretože menia prostredie.  

Tri pravidlá, ktoré platia všade:  

- **Predvolene čítaj, píš až po potvrdení.** Čítanie je reverzibilné,  
  zápis, mazanie a odosielanie nie.  
- **`allowed-tools` nie je bezpečnostná hranica.** Povoľuje nástroje na  
  jeden ťah; zákazy v nastaveniach platia stále.  
- **Nebezpečné nástroje patria za schválenie.** Mazanie, platby a správy  
  majú mať potvrdenie človeka, aj keď to znamená o jeden klik viac.  

## 9. Ako navrhnúť dobrý nástroj

Ak píšete vlastný nástroj alebo MCP server, rozhoduje popis a schéma:  

- **Jeden nástroj, jedna akcia.** `send_email` áno, `do_everything` nie.  
- **Popíšte, kedy nástroj použiť.** Model vyberá podľa popisu, nie podľa  
  názvu. Uveďte, čím sa líši od podobného nástroja.  
- **Obmedzte vstup.** Povinné polia, `enum`, maximálna dĺžka. Voľný  
  text tam, kde treba, ale nie všade.  
- **Vracajte málo.** Zhrnutie, zoznam riadkov, odkaz. Celý súbor alebo  
  celý web zaplní kontext a odpoveď predraží.  
- **Chybu vráťte ako text.** „Súbor neexistuje, v priečinku sú tieto“  
  vedie k oprave; výnimka ukončí slučku.  
- **Nástroj, ktorý mení svet, nech je idempotentný alebo potvrdený.**  
  Opakované volanie nemá vytvoriť druhý záznam.  

## 10. Časté chyby

| Prejav                          | Príčina                        | Riešenie                        |  
| ------------------------------- | ------------------------------ | ------------------------------- |  
| agent tvrdí niečo, čo neoveril  | použil pamäť namiesto nástroja | žiadajte zdroj a overenie       |  
| nástroj sa vôbec nespustí       | chýba povolenie                | povoľte ho alebo píšte „ask“    |  
| model volá nesprávny nástroj    | nejasný popis                  | doplňte, kedy ho použiť         |  
| volanie má zlý argument         | voľná schéma                   | sprísnite typy a `enum`         |  
| odpoveď je pomalá a drahá       | nástroj vracia priveľa dát     | skráťte výstup, čítajte rozsahy |  
| agent urobil nežiaducu zmenu    | zápis bez potvrdenia           | `ask` pri editácii a mazaní     |  
| MCP nástroj sa neukáže          | server nie je pripojený        | overte server a jeho oprávnenia |  
| rovnaký názov v dvoch serveroch | kolízia mien                   | premenujte alebo vypnite jeden  |  

## 11. Otázky na opakovanie

1. Prečo model bez nástrojov nevie, aký je dnes deň?  
2. Kto vykonáva volanie nástroja a kto rozhoduje o povolení?  
3. Prečo nástroj na čítanie súboru vracia obsah a nie zhrnutie?  
4. Aký je rozdiel medzi `websearch` a `webfetch`?  
5. Prečo je subagent nástroj a kedy sa hodí?  
6. Prečo `allowed-tools` nestačí ako bezpečnostná hranica?  
7. Čo urobíte, keď agent odpovie bez toho, aby niečo overil?  

## 12. Úlohy na precvičenie

1. Spustite najjednoduchší nástroj — zistite dnešný dátum — a všimnite  
   si, ako sa volanie zobrazí v aplikácii.  
2. Dajte agentovi prečítať jeden súbor a zhrnúť ho do troch odrážok.  
   Potom ho požiadajte o konkrétne riadky.  
3. Nájdite v projekte všetky výskyty jedného slova dvoma spôsobmi  
   (hľadanie súborov a hľadanie v texte) a porovnajte výsledky.  
4. Opravte preklep v súbore a všimnite si diff a potvrdenie.  
5. Zistite cenu jednej služby z webu a overte ju v druhom nezávislom  
   zdroji. K tvrdeniu uveďte odkaz.  
6. Vypnite jeden nástroj (napríklad `edit`) a skúste tú istú úlohu.  
   Zapíšte, čo sa zmenilo v správaní agenta.  

## 13. Zhrnutie

- Nástroj je deklarácia, volanie a výsledok; model ho nevykonáva.  
- Bez nástrojov model háda, s nástrojmi overuje.  
- Volanie je slučka: model žiada, aplikácia vykoná, výsledok sa vráti.  
- Viditeľnosť volaní je prvá kontrola pri podozrení na chybu.  
- Mená nástrojov sa medzi aplikáciami líšia, úlohy zostávajú rovnaké.  
- Povolenia patria do konfigurácie; nebezpečné akcie za potvrdenie.  

## 14. Slovníček

| Pojem               | Význam                                                       |  
| ------------------- | ------------------------------------------------------------ |  
| nástroj (tool)      | funkcia, ktorú model volá; skladá sa z deklarácie, volania a výsledku |  
| deklarácia          | meno, popis a JSON schéma parametrov, ktorú dostane model     |  
| volanie             | JSON požiadavka, ktorou model žiada o vykonanie nástroja      |  
| výsledok            | dáta, ktoré aplikácia vráti do konverzácie                    |  
| slučka agenta       | opakovanie model → nástroj → výsledok, kým úloha neskončí     |  
| read-only nástroj   | nástroj, ktorý nemení stav (čítanie, hľadanie)                |  
| idempotentný nástroj| opakované volanie má rovnaký účinok ako jedno                  |  
| povolenie           | pravidlo, či sa nástroj spustí sám, po potvrdení alebo nikdy  |  
| MCP                 | protokol, ktorým sa pripájajú externé nástroje a dáta          |  
| prompt injection    | pokyn ukrytý v cudzom texte, ktorý sa snaží ovládnuť agenta   |  
| diff                | zobrazenie zmien pred prevzatím výsledku                      |  
| subagent            | samostatný agent s vlastným kontextom, volaný ako nástroj     |  

## Zdroje

- Claude Code – Tools reference: code.claude.com/docs/en/tools  
- Claude Code – Permissions: code.claude.com/docs/en/permissions  
- VS Code – Tools and context reference:  
  code.visualstudio.com/docs/agents/reference/tools-reference  
- VS Code – Use tools with agents: code.visualstudio.com/docs/agents/run/tools  
- opencode – Tools: opencode.ai/docs/tools  
- Model Context Protocol: modelcontextprotocol.io  
