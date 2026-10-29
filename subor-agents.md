# Kapitola: AGENTS.md – návod na obsluhu projektu pre AI agenta

## 1. Prečo to vôbec potrebujeme

AI agent, ktorý pracuje s vaším projektom (priečinkom so súbormi), si medzi  
reláciami nič nepamätá. Pri každom novom zadaní začína takmer od nuly: nevie,  
ako sa projekt spúšťa, kde čo leží, ani aké zvyky v ňom platia.  

Predstavte si nového kolegu, ktorý prvý deň príde do firmy bez zaškolenia.  
Môže byť šikovný, ale bude strácať čas hádaním a občas urobí chybu, ktorej sa  
dalo vyhnúť. Súbor AGENTS.md je takéto zaškolenie napísané raz a použité vždy.  

## 2. Čo je AGENTS.md

AGENTS.md je štandard pre AI agentov: otvorený formát, ktorý hovorí,  
ako má agent pracovať s konkrétnym projektom. Je to obyčajný textový  
súbor vo formáte Markdown, ktorý leží priamo v projekte. Často sa  
označuje ako README pre agentov.  

- Nemá žiadnu povinnú štruktúru ani špeciálnu syntax. Stačia nadpisy a  
  odrážky.  
- Agent ho prečíta automaticky na začiatku práce, bez toho, aby ste mu ho  
  zakaždým vkladali do zadania.  
- Nie je to nástroj jedného výrobcu. Štandard používa veľa nástrojov,  
  napríklad OpenAI Codex, GitHub Copilot vo VS Code, Cursor, Gemini CLI,  
  Aider, Zed, Warp alebo Kilo Code.  

Podľa stránky agents.md štandard využíva viac ako 60 000 open-source  
projektov. Jeho správu má organizácia Agentic AI Foundation pod Linux  
Foundation.  

Pozor na názvy: niektoré nástroje majú vlastný súbor s rovnakou úlohou,  
napríklad Claude Code používa CLAUDE.md. Ak používate viac nástrojov, overte  
si v ich dokumentácii, ktorý názov čítajú.  

## 3. Prečo nestačí README

README.md píšeme pre ľudí: čo projekt robí, ako ho nainštalovať, ako prispieť.  
AGENTS.md dopĺňa to, čo agent potrebuje pri samotnej práci a čo by v README  
len prekážalo.  

- Presné príkazy na spustenie, testy a kontrolu kvality.  
- Pravidlá štýlu a konvencie, ktoré v projekte platia.  
- Upozornenia, čomu sa vyhnúť a čo nikdy nemeniť.  

Vďaka tomu zostane README čitateľné pre ľudí a pokyny pre agenta majú svoje  
vlastné, jasne určené miesto.  

## 4. Kde súbor leží a ako sa číta

- Koreň projektu: hlavný AGENTS.md platí pre celý projekt.  
- Podpriečinky: v zložitejších projektoch môže mať vlastný AGENTS.md aj  
  podpriečinok, napríklad docs/ alebo backend/.  
- Priorita: agent zvyčajne použije najbližší súbor k miestu, kde pracuje.  
  Konkrétnejší pokyn teda prebije všeobecnejší.  
- Vaše zadanie v chate má vždy väčšiu váhu ako obsah súboru. Ak povieš  
  agentovi niečo iné, riadi sa tebou.  

## 5. Čo do súboru patrí

Dobrý AGENTS.md odpovedá na otázky, ktoré by si položil nový kolega. Nie je  
potrebné pokryť všetko, stačí to, čo agent nevie odvodiť z kódu sám.  

- Prehľad: jednou-dvoma vetami, o aký projekt ide a aké technológie používa.  
- Príkazy: ako projekt spustiť, otestovať a skontrolovať. Presne tak, ako ich  
  napíšete do terminálu.  
- Štruktúra: ktorý priečinok na čo slúži.  
- Štýl a konvencie: jazyk textov, pomenovávanie, formátovanie.  
- Testovanie: čo musí prejsť pred dokončením práce.  
- Hranice: čo agent smie robiť sám, na čo sa má najprv opýtať a čo nesmie  
  nikdy.  

## 6. Ako písať dobre

- Buďte struční. Súbor zaberá miesto v pamäti agenta, ktorú zdieľa so zadaním  
  a kódom. Dlhý text zníži pozornosť, ktorú dostane to podstatné.  
- Buďte konkrétni. Namiesto Píš kvalitný kód napíšte Funkcie majú najviac 40  
  riadkov a každá nová má test.  
- Uvádzajte príkazy, nie opisy. Presný príkaz agent môže rovno spustiť.  
- Opisujte, čo robiť, a nie len čo nerobiť. Zákaz bez alternatívy agenta  
  zmätie.  
- Žiadne heslá, kľúče ani osobné údaje. Súbor je súčasť projektu a môže sa  
  zdieľať.  
- Udržiavajte ho aktuálny. Zastaraný pokyn škodí viac ako žiadny.  

## 7. Ako súbor vznikne

1. Automatický návrh: mnohé nástroje majú príkaz (často /init), ktorý projekt  
   prejde a AGENTS.md vygeneruje.  
2. Kontrola a skrátenie: vygenerovaný text prečítajte, opravte nepresnosti a  
   vyhoďte všetko, čo nie je dôležité.  
3. Postupné dopĺňanie: keď agent opakovane urobí rovnakú chybu, pridajte jedno  
   krátke pravidlo, ktoré jej predíde.  
4. Pravidelné čistenie: raz za čas súbor prejdite a odstráňte pravidlá, ktoré  
   už neplatia.  

## 8. Čo si treba uvedomiť

- Agent nemusí poslúchnuť na 100 %. AGENTS.md zvyšuje pravdepodobnosť  
  správneho správania, nie je to záruka. Výsledky stále kontrolujte.  
- Súbor nenahrádza zabezpečenie. Skutočné obmedzenia (prístupy, oprávnenia,  
  zálohy) nastavte v systéme, nie len v texte.  
- Viac pravidiel neznamená lepší výsledok. Pri dlhom zozname sa dôležité  
  pokyny strácajú medzi nepodstatnými.  

## 9. Zhrnutie

- AGENTS.md je krátky Markdown súbor s pokynmi pre AI agenta.  
- Patria doňho príkazy, štruktúra, konvencie a hranice.  
- Platí pravidlo: stručne, konkrétne, aktuálne.  
- Začnite automatickým návrhom a dopĺňajte ho podľa skúseností.  

## 10. Príklady na záver

### Príklad 1: Malá webstránka

Statická stránka bez frameworku. Dôraz je na príkazoch, štruktúre a pár  
jasných pravidlách.  

```markdown  
# AGENTS.md  
  
## Prehľad  
Webstránka malej kaviarne. Statické HTML, CSS a trochu JavaScriptu.  
Žiadny framework.  
  
## Príkazy  
- Lokálny náhľad: `npx serve .`  
- Kontrola odkazov: `npm run check-links`  
  
## Štruktúra  
- `index.html`: úvodná stránka  
- `menu/`: stránky s jedálnym lístkom  
- `assets/`: obrázky a štýly  
  
## Pravidlá  
- Texty na stránke píš po slovensky.  
- Nepridávaj nové knižnice bez schválenia.  
- Obrázky nad 300 kB najprv zmenši.  
- Nikdy nemeň súbor `CNAME`.  
```  

### Príklad 2: Python skripty na spracovanie dát

Projekt s testami. Agent vie, ako overiť svoju prácu.  

```markdown  
# AGENTS.md  
  
## Prehľad  
Python 3.12 skripty na spracovanie mesačných exportov z účtovníctva.  
Vstup: súbory CSV. Výstup: súbory XLSX.  
  
## Príkazy  
- Inštalácia: `pip install -r requirements.txt`  
- Testy: `pytest -q`  
- Kontrola štýlu: `ruff check .`  
  
## Konvencie  
- Každá nová funkcia má typové anotácie a test v `tests/`.  
- Cesty k súborom čítaj z `config.toml`, nepíš ich priamo do kódu.  
  
## Pred dokončením práce  
- Spusti testy a kontrolu štýlu. Obe musia prejsť bez chýb.  
```  

### Príklad 3: Vnorený súbor pre dokumentáciu

Väčší projekt má hlavný súbor v koreni a špeciálny súbor v priečinku docs/,  
kde platia iné pravidlá. Pri práci v docs/ má prednosť ten bližší.  

```markdown  
# AGENTS.md (koreň projektu)  
  
- Odpovedaj stručne a po slovensky.  
- Pred väčšou zmenou štruktúry sa najprv opýtaj.  
- Testy: `npm test`  
```  

```markdown  
# docs/AGENTS.md  
  
- Píš jednoduchým jazykom pre čitateľov bez technického zázemia.  
- Každá kapitola začína krátkym zhrnutím na 2 až 3 vety.  
- Pojmy z angličtiny pri prvom použití vysvetli v zátvorke.  
- Súbory v `docs/generated/` nikdy needituj, vznikajú automaticky.  
```  

### Príklad 4: Hranice a bezpečnosť

Trojstupňové rozdelenie, ktoré dobre funguje v každom projekte. Agent vie, čo  
môže, na čo sa má opýtať a čo je zakázané.  

```markdown  
## Hranice  
  
### Smieš robiť sám  
- Čítať súbory a spúšťať testy.  
- Upravovať súbory v `src/` a `tests/`.  
  
### Najprv sa opýtaj  
- Pridanie alebo odstránenie knižnice.  
- Zmena štruktúry priečinkov.  
- Mazanie súborov.  
  
### Nikdy  
- Neukladaj do projektu heslá ani prístupové kľúče.  
- Nemeň súbory v `.github/` ani konfiguráciu nasadenia.  
- Nespúšťaj príkazy, ktoré mažú dáta mimo projektu.  
```  

### Príklad 5: Zlý a dobrý pokyn

Rozdiel medzi vágnym a použiteľným pravidlom je v konkrétnosti.  

```markdown  
Zlé:   Píš čistý a kvalitný kód.  
Dobré: Funkcie majú najviac 40 riadkov a každá má test.  
  
Zlé:   Buď opatrný s databázou.  
Dobré: Na databázu sa pripájaj len v režime čítania. Zápis vždy  
       najprv navrhni a počkaj na schválenie.  
  
Zlé:   Nepoužívaj zastaranú knižnicu.  
Dobré: Na HTTP požiadavky použi `httpx`, nie `requests`.  
```
