# Príkazy v Claude Code

*Verzia 1.1 · stav k októbru 2026. Príkazy sa menia s každou verziou  
Claude Code. Pred použitím si ich overte v dokumentácii.*  

## 1. Ciele lekcie

Po tejto lekcii budete vedieť:  

- vysvetliť, čo je príkaz a čím sa líši od bežného promptu,  
- poznať najdôležitejšie zabudované príkazy a vedieť, kedy ktorý použiť,  
- napísať vlastný príkaz na prácu s dokumentmi a webovým hľadaním,  
- odovzdať príkazu argumenty a aktuálne dáta z počítača,  
- obmedziť, čo príkaz smie urobiť, a rozhodnúť, čo zdieľať s tímom.  

Kapitola nadväzuje na [workflows.md](workflows.md) (opakovateľný postup),  
[subor-agents.md](subor-agents.md) (pokyny pre agenta)  
a [subagenti.md](subagenti.md) (delegovanie práce).  

## 2. Prečo príkazy

Každý týždeň ten istý postup: prečítať tri dokumenty, zhrnúť ich,  
vytiahnuť rozhodnutia a napísať, čo treba urobiť. Prvý raz to napíšete  
do chatu. Druhý raz to napíšete znova, len trochu inak. Po desiatich  
týždňoch máte desať verzií toho istého zadania a ani jedna nie je presne  
tá správna.  

Príkaz je **uložené zadanie**. Napíšete ho raz, dáte mu krátke meno  
a odvtedy ho spúšťate ako `/zhrn-dokument`. Nemusíte si pamätať, čo má  
zhrnutie obsahovať, ani v akom formáte ho chcete dostať.  

Príkazy dopĺňajú trvalé pokyny. Claude Code má súbor `CLAUDE.md` s pokynmi,  
ktoré prečíta na začiatku každej relácie, teda jedného spustenia Clauda.  
Ten je na fakty: aký jazyk sa používa, ako sa u vás pomenúvajú súbory, čo  
sa nikdy nemá meniť. Príkaz je na postup, ktorý sa hodí len pri niektorých  
úlohách.  

| Vlastnosť       | `CLAUDE.md`                 | Príkaz                |  
| --------------- | --------------------------- | --------------------- |  
| Kedy sa načíta  | vždy na začiatku            | až keď ho spustíte    |  
| Čo obsahuje     | fakty a stále pravidlá      | postup na jednu úlohu |  
| Cena v kontexte | platíte ju v každej relácii | len pri použití       |  

Preto sa do `CLAUDE.md` píšu fakty a do príkazov postupy. Dlhý kontrolný  
zoznam v `CLAUDE.md` by zaberal miesto pri každej úlohe, aj keď ho práve  
nepotrebujete. Rovnaký zoznam v príkaze sa načíta len vtedy, keď ho  
použijete.  

## 3. Čo je príkaz

**Príkaz** je text, ktorý začína lomítkom a menom, napríklad  
`/deep-research`. Claude Code ho rozpozná len na **začiatku správy**. Text  
za menom sa stane argumentom príkazu. Keď príkaz odošlete, jeho obsah sa  
vloží do konverzácie ako inštrukcia pre Clauda, akoby ste ho napísali sami.  

Príkazy sú troch druhov:  

| Druh              | Kto ho vytvoril | Príklad          | Ako funguje                       |  
| ----------------- | --------------- | ---------------- | --------------------------------- |  
| Zabudovaný príkaz | Claude Code     | `/clear`         | pevná logika priamo v programe    |  
| Zabalený skill    | Anthropic       | `/deep-research` | prompt, ktorý dostane Claude      |  
| Vlastný príkaz    | vy              | `/zhrn-dokument` | Markdown súbor vo vašom priečinku |  

Rozdiel medzi prvými dvoma je praktický: zabudovaný príkaz vykoná presne  
naprogramovanú akciu, kým zabalený skill je len dobre napísaný prompt,  
ktorý Claude vykoná pomocou svojich nástrojov. Pre vás sa oba spúšťajú  
rovnako.  

Ešte tri pravidlá, ktoré sa hodia poznať:  

- Príkaz uprostred vety sa nespustí. `/plan` v texte je len text.  
- Ak príkaz odošlete počas odpovede, Claude Code ho zaradí do fronty  
  a vykoná po dokončení aktuálneho ťahu. Niektoré príkazy, napríklad  
  `/usage`, sa vykonajú okamžite.  
- V jednej správe môžete spojiť až šesť skills, napríklad  
  `/skill-a /skill-b urob XYZ`. Spoločný text za menami dostane každý z nich  
  ako argument.  

## 4. Najdôležitejšie príkazy

Keď napíšete `/`, Claude Code ponúkne zoznam dostupných príkazov. Ponuka  
závisí od verzie, platformy a plánu, preto sa zoznam v dvoch počítačoch  
môže líšiť. Tabuľka uvádza príkazy, ktoré pokrývajú bežný pracovný deň  
s dokumentmi, webom a e-mailom.  

| Príkaz            | Čo urobí                                         | Kedy ho použiť                              |  
| ----------------- | ------------------------------------------------ | ------------------------------------------- |  
| `/help`           | vypíše príkazy a klávesové skratky               | keď neviete, čo je k dispozícii             |  
| `/init`           | vytvorí `CLAUDE.md` s pokynmi pre priečinok      | pri prvom otvorení projektu                 |  
| `/add-dir`        | pridá ďalší priečinok, do ktorého smie čítať     | keď sú podklady v inom priečinku            |  
| `/memory`         | upraví trvalé pokyny a pamäť                     | keď sa pokyny pre projekt menia             |  
| `/model`          | prepne model                                     | keď chcete iný pomer ceny a kvality         |  
| `/effort`         | nastaví, ako dlho má model premýšľať             | pri ťažkej úlohe, alebo naopak rýchlo       |  
| `/plan`           | najprv navrhne plán, až potom koná               | pred väčšou zmenou                          |  
| `/context`        | ukáže, čo zaplňuje kontextové okno               | keď začnú byť odpovede nepresné             |  
| `/compact`        | zhrnie konverzáciu a uvoľní miesto               | pri dlhej úlohe                             |  
| `/clear`          | začne novú konverzáciu                           | pri novej, nesúvisiacej úlohe               |  
| `/btw`            | položí vedľajšiu otázku mimo histórie            | keď nechcete zmeniť smer práce              |  
| `/resume`         | vráti vás k skoršej konverzácii                  | na pokračovanie po prestávke                |  
| `/branch`         | vytvorí odbočku konverzácie                      | keď chcete vyskúšať iný smer                |  
| `/fork`           | skopíruje konverzáciu do relácie na pozadí       | keď má práca pokračovať inde                |  
| `/rewind`         | vráti súbory aj konverzáciu k skoršiemu bodu     | keď agent urobil priveľa zmien              |  
| `/tasks`          | ukáže úlohy, ktoré bežia na pozadí               | keď chcete vidieť, čo sa práve deje         |  
| `/background`     | odpojí reláciu a nechá ju bežať                  | keď potrebujete uvoľniť príkazový riadok    |  
| `/remote-control` | sprístupní reláciu na claude.ai                  | na pokračovanie z telefónu                  |  
| `/teleport`       | pritiahne cloudovú reláciu do príkazového riadka | keď ste začali v cloude                     |  
| `/usage`          | ukáže spotrebu a limity plánu                    | keď kontrolujete, koľko míňate              |  
| `/status`         | ukáže verziu, model, účet a pripojenie           | keď si chcete overiť, v čom pracujete       |  
| `/export`         | uloží konverzáciu do textového súboru            | na archiváciu alebo zdieľanie               |  
| `/deep-research`  | rešerš s overením zdrojov a odkazmi              | pri téme, ktorú treba podložiť              |  
| `/loop`           | opakuje zadanie v zadanom intervale              | na pravidelnú kontrolu                      |  
| `/diff`           | ukáže zmeny v súboroch                           | pred prevzatím výsledku                     |  
| `/doctor`         | skontroluje nastavenie a navrhne opravy          | keď niečo nefunguje                         |  
| `/permissions`    | nastaví pravidlá povoľovania nástrojov           | keď agent žiada priveľa schválení           |  
| `/mcp`            | pripojí ďalšie služby, napríklad kalendár        | keď potrebujete pracovať s externou službou |  
| `/feedback`       | odošle spätnú väzbu alebo hlásenie chyby         | keď chcete nahlásiť problém                 |  

Štyri z nich si zaslúžia osobitnú poznámku:  

- **`/compact` verzus `/clear`.** `/compact` zhrnie konverzáciu a pokračuje  
  v tej istej úlohe. `/clear` začne od nuly a nechá len trvalé pokyny. Pri  
  novej úlohe je správny `/clear`, pri dlhej úlohe `/compact`.  
- **`/btw`.** Vedľajšia otázka, ktorá sa neuloží do histórie. Hodí sa, keď  
  potrebujete niečo zistiť, ale nechcete, aby to zmenilo smer rozpracovanej  
  úlohy.  
- **`/remote-control`.** Sprístupní bežiacu reláciu na claude.ai, takže  
  v nej pokračujete z telefónu alebo z iného počítača. Vyžaduje predplatné  
  claude.ai; skratka je `/rc`.  
- **`/rewind`.** Vráti súbory aj konverzáciu k skoršiemu bodu. Je to prvá  
  vec, ktorú skúste, keď agent upravil viac, než ste chceli. Skratka  
  `/undo` funguje rovnako.  

## 5. Vlastný príkaz

Vlastný príkaz je obyčajný Markdown súbor. Miesto, kam ho uložíte, určuje,  
kde bude dostupný. Projekt je priečinok, v ktorom ste Claude spustili.  

| Umiestnenie           | Platnosť                     | Pre koho         |  
| --------------------- | ---------------------------- | ---------------- |  
| `.claude/commands/`   | daný priečinok               | pre celý tím     |  
| `~/.claude/commands/` | všetky vaše projekty         | len pre vás      |  
| `commands/` v plugine | všade, kde je plugin aktívny | pre používateľov |  

Meno príkazu vzniká z názvu súboru bez prípony. Súbor  
`.claude/commands/zhrn-dokument.md` vytvorí príkaz `/zhrn-dokument`.  

Podpriečinky vytvárajú menný priestor: každé lomítko v ceste sa zmení na  
dvojbodku. Súbor `.claude/commands/faktury/kontrola.md` teda spustíte ako  
`/faktury:kontrola`. V ponuke `/` a v `/help` je vidieť, odkiaľ príkaz  
prišiel, napríklad `(project)` alebo `(user)`.  

> **Dôležité:** Anthropic vlastné príkazy zlúčil so **skills**. Súbor  
> `.claude/commands/zhrn-dokument.md` a skill  
> `.claude/skills/zhrn-dokument/SKILL.md` vytvoria ten istý príkaz  
> `/zhrn-dokument` a správajú sa rovnako. Starý formát funguje ďalej, ale  
> nové veci sa odporúča písať ako skill, pretože vie mať vedľa seba pomocné  
> súbory. Ak existujú oba s rovnakým menom, vyhrá skill.  

## 6. Formát: telo je prompt

Telo súboru je to, čo Claude dostane. Píšte ho ako pokyn pre Clauda, nie  
ako text pre človeka. Namiesto „Tento príkaz zhrnie dokument“ napíšte  
„Zhrň dokument“.  

Najjednoduchší príkaz nepotrebuje žiadnu hlavičku. Súbor  
`.claude/commands/zhrn-dokument.md`:  

```markdown
Prečítaj dokument v tomto priečinku a zhrň ho do piatich odrážok.
Na konci uveď tri veci, ktoré treba overiť u autora.
```

Keď napíšete `/zhrn-dokument`, Claude dostane presne tento text.  

Telo zostaňte krátke. Obsah príkazu zostáva v kontexte aj v ďalších ťahoch,  
takže každý riadok sa počíta pri každej ďalšej správe. Podrobnú  
dokumentáciu nechajte v samostatnom súbore a v príkaze naň odkážte.  

## 7. Hlavička

Voliteľná YAML hlavička medzi `---` mení správanie príkazu. Všetky polia sú  
nepovinné, odporúča sa len `description`.  

| Pole                       | Význam                                                                   |  
| -------------------------- | ------------------------------------------------------------------------ |  
| `description`              | čo príkaz robí a kedy ho použiť; Claude podľa nej pozná, kedy ho nasadiť |  
| `argument-hint`            | nápoveda pri dopĺňaní, napríklad `[subor]`                               |  
| `allowed-tools`            | nástroje bez pýtania sa na povolenie počas tohto ťahu                    |  
| `model`                    | model pre tento príkaz; `inherit` nechá aktuálny                         |  
| `arguments`                | mená pozičných argumentov pre zápis `$meno`                              |  
| `disable-model-invocation` | `true` zabráni Claudovi spustiť príkaz sám                               |  
| `user-invocable`           | `false` skryje príkaz z ponuky, vyvolá ho len Claude                     |  
| `context`                  | `fork` spustí príkaz v samostatnom subagentovi                           |  
| `agent`                    | ktorý typ subagenta použiť pri `context: fork`                           |  

Súbor v `.claude/commands/` prijme rovnaké polia okrem `name` a `paths`.  
Meno tam určuje názov súboru, nie hlavička.  

Dve veci sa ľahko pokazia:  

- Hlavička sa načíta len vtedy, keď je otváracia `---` na **prvom riadku**  
  súboru. Ak je pred ňou prázdny riadok alebo nadpis, celý súbor sa berie  
  ako telo a polia sa ignorujú.  
- Neznáme pole Claude Code **ticho ignoruje**. Preklep v `allowed-tools`  
  teda neohlási chybu, len sa nič nestane.  

## 8. Argumenty

Argumenty robia z príkazu nástroj, nie pevný text. Napíšete  
`/zhrn-dokument vyrocna-sprava.pdf` a príkaz pracuje s týmto súborom.  

| Zápis           | Čo vloží                                          |  
| --------------- | ------------------------------------------------- |  
| `$ARGUMENTS`    | všetky argumenty tak, ako ste ich napísali        |  
| `$0`, `$1`      | prvý, druhý argument                              |  
| `$ARGUMENTS[0]` | to isté ako `$0`, len dlhším zápisom              |  
| `$dokument`     | pomenovaný argument z poľa `arguments` v hlavičke |  

Príklad s pomenovanými argumentmi:  

```markdown
---
description: Preloží dokument do zadaného jazyka
argument-hint: [subor] [jazyk]
arguments: [dokument, jazyk]
---

Prelož súbor $dokument do jazyka $jazyk.
Zachovaj čísla, mená a tabuľky. Nejasné miesta označ a vypíš na konci.
```

Pravidlá, ktoré sa hodia:  

- Hodnoty s medzerou dajte do úvodzoviek: `/priklad "hello world" druhy`.  
  `$0` potom obsahuje `hello world` a `$1` slovo `druhy`.  
- Ak argument chýba, indexovaný znak zostane v texte doslova.  
  Pomenovaný argument sa nahradí prázdnym reťazcom.  
- `$ARGUMENTS`, ktorý v tele nie je, Claude Code pripojí na koniec ako  
  `ARGUMENTS: ...`. Príkaz teda o argumenty nepríde.  
- Znak dolára v bežnom texte uveďte s lomítkom: `\$1.00`.  

## 9. Živé dáta v prompte

Príkaz môže pred odoslaním spustiť shell a vložiť výsledok. Riadok  
``!`date +"%d. %m. %Y"` `` sa nahradí dnešným dátumom, takže Claude vidí  
skutočný stav, nie to, čo si domyslí.  

```markdown
## Dnes je

!`date +"%d. %m. %Y"`

## Podklady v priečinku

!`ls -1 *.pdf *.docx 2>/dev/null`

## Pokyny

Vyber z podkladov tie, ktoré sú z posledných 30 dní, a zhrň ich.
```

Na Windows vymeňte `ls` za `dir`; zvyšok príkazu zostane rovnaký.  

Tento mechanizmus sa volá **dynamické vkladanie kontextu**. Okrem `!`  
príkazu sa hodia aj premenné `${CLAUDE_PROJECT_DIR}` (koreň projektu)  
a `${CLAUDE_SKILL_DIR}` (priečinok so skillom).  

Pozor: príkaz v `!` sa spúšťa na vašom počítači s vašimi právami. Nikdy do  
neho nevkladajte heslá ani kľúče a nepúšťajte sa do neho s dátami, ktoré  
nechcete mať v kontexte.  

## 10. Príklad: zhrnutie dokumentu

Súbor `.claude/commands/zhrn-dokument.md`. Príkaz len číta, preto môže mať  
vopred povolené nástroje a nemusí sa pýtať pri každom kroku.  

```markdown
---
description: Zhrnie dokument a vytiahne rozhodnutia a termíny
argument-hint: [nazov-suboru]
allowed-tools: Read, Glob
---

Nájdi v tomto priečinku súbor $ARGUMENTS a prečítaj ho.

Napíš:
1. Tri vety, o čom dokument je.
2. Rozhodnutia a termíny, ktoré obsahuje.
3. Otvorené otázky.

Ak súbor neexistuje, vypíš, čo v priečinku je, a skonči.
```

Použitie: `/zhrn-dokument vyrocna-sprava.pdf`.  

## 11. Príklad: rešerš z webu

Súbor `.claude/commands/resers.md`. Príkaz smie len hľadať a čítať,  
nemôže nič meniť ani odosielať.  

```markdown
---
description: Urobí rešerš na tému a vráti prehľad so zdrojmi
argument-hint: [téma]
allowed-tools: WebSearch, WebFetch
---

Urob rešerš na tému: $ARGUMENTS.

Postup:
1. Nájdi aspoň päť nezávislých zdrojov, z toho dva mimo prvých výsledkov.
2. Over, či sa kľúčové čísla zhodujú aspoň v dvoch zdrojoch.
3. Napíš prehľad na jednu stranu: čo je isté, čo je sporné a čo chýba.

Ku každému tvrdeniu uveď odkaz. Ak zdroj chýba, napíš to, nevymýšľaj.
```

Použitie: `/resers vplyv umelej inteligencie na trh práce 2026`.  

## 12. Príklad: odpoveď na e-mail

Príkaz s vedľajšími účinkami. Nechceme, aby ho Claude spustil sám, len  
pretože našiel e-mail, ktorý sa mu zdá neodpovedaný. Preto  
`disable-model-invocation` a jediný povolený nástroj na čítanie.  

```markdown
---
description: Pripraví návrh odpovede na e-mail
argument-hint: [subor-s-emailom]
disable-model-invocation: true
allowed-tools: Read
---

Prečítaj e-mail v súbore $ARGUMENTS.

Priprav návrh odpovede:
1. Zhrň, čo druhá strana žiada.
2. Odpovedz vľúdne a krátko, najviac 150 slov.
3. Ak niečo chýba, polož najviac dve otázky.

Návrh vypíš v chate. Nikdy ho neodosielaj.
```

Použitie: `/odpoved odberatel-email.txt`. Nástroje, ktoré v `allowed-tools`  
nie sú, sa jednoducho opýtajú na povolenie. Posledný krok tak zostáva na  
človekovi, čo je pri odosielaní správ správne.  

## 13. Ako písať dobré príkazy

- **Jeden príkaz, jedna úloha.** Ak má príkaz päť nezávislých cieľov, sú to  
  skôr štyri príkazy.  
- **Píšte v rozkazovacom spôsobe.** „Prečítaj“, „over“, „zastav sa“.  
  Claude nemá hádať, čo od neho chcete.  
- **Uveďte, kedy prestať.** „Ak dokument chýba, vypíš, čo v priečinku je“ je  
  lepšie než „pokračuj, kým to nebude hotové“.  
- **Pomenujte výstup.** Formát, dĺžka a jazyk patria do príkazu, inak ich  
  budete dopisovať pri každom použití.  
- **Žiadajte zdroje.** Pri rešerši určte, koľko zdrojov a ako ich overiť.  
  Bez toho dostanete sebavedomý text bez podkladov.  
- **Overte na jednom dokumente.** Príkaz, ktorý zlyhá na jednom dokumente,  
  zlyhá aj na celej zbierke, len drahšie.  
- **Uložte ho do projektu.** Príkaz je súčasť pracovného postupu. Zmena  
  postupu má mať svoju históriu a svojho autora.  

## 14. Bezpečnosť a hranice

- **`allowed-tools` nie je bezpečnostná hranica.** Iba dopredu schváli  
  nástroje na jeden ťah; grant sa pri ďalšej správe zruší. Zákazy  
  v nastaveniach ostávajú v platnosti.  
- **Príkazy cestujú s priečinkom.** Príkaz od kolegu je návod, ktorý váš  
  agent vykoná. Prečítajte si ho pred spustením, hlavne keď obsahuje `!`  
  alebo `Bash`.  
- **Cudzí text je vstup od cudzích ľudí.** Webová stránka aj dokument môžu  
  obsahovať pokyny pre agenta (prompt injection). Príkaz, ktorý číta web,  
  nemá mať právo odosielať správy, platiť ani mazať súbory.  
- **Žiadne tajomstvá v príkazoch.** Súbor sa zdieľa s tímom a jeho obsah sa  
  dostane do kontextu. Kľúče patria do premenných prostredia.  
- **Osobné zvyky do `~/.claude`.** Čo nemá vidieť tím, nepatrí do  
  `.claude/commands/` v projekte.  
- **Nebezpečné akcie za schválenie.** Mazanie, odosielanie a platby si  
  zaslúžia `disable-model-invocation: true` a posledný krok v rukách  
  človeka.  

## 15. Časté chyby

| Prejav                        | Príčina                          | Riešenie                           |  
| ----------------------------- | -------------------------------- | ---------------------------------- |  
| príkaz sa neukáže v ponuke    | zlý priečinok alebo názov súboru | `.claude/commands/meno.md`         |  
| hlavička sa ignoruje          | `---` nie je na prvom riadku     | odstráňte riadky pred hlavičkou    |  
| pole nemá účinok              | preklep v názve poľa             | porovnajte s dokumentáciou         |  
| argumenty sa nevyplnia        | chýba argument alebo úvodzovky   | píšte `$0` a hodnoty do úvodzoviek |  
| príkaz robí priveľa           | miešanie úloh v jednom súbore    | rozdeľte ho                        |  
| rovnaké meno ako iný príkaz   | kolízia so skillom               | premenujte jeden z nich            |  
| Claude spustil príkaz sám     | chýba `disable-model-invocation` | doplňte pole                       |  
| rešerš nemá zdroje            | chýba požiadavka na odkazy       | doplňte pravidlo do príkazu        |  
| dlhý príkaz spomaľuje reláciu | telo zostáva v kontexte          | skráťte ho, zvyšok do súboru       |  

## 16. Príkazy, skills a subagenti

Tieto tri veci sa často zamieňajú, hoci riešia iný problém.  

| Nástroj  | Čo to je                     | Kto ho spustí                | Čo zmení                                 |  
| -------- | ---------------------------- | ---------------------------- | ---------------------------------------- |  
| Príkaz   | uložený prompt               | vy cez `/meno`               | inštrukcie v aktuálnom kontexte          |  
| Skill    | príkaz s pomocnými súbormi   | vy alebo Claude podľa popisu | to isté, plus priložené súbory           |  
| Subagent | samostatný agent s kontextom | rodič alebo vy               | dostane vlastný kontext a vráti výsledok |  

Praktické pravidlo: keď si chcete **zapamätať postup**, urobte príkaz alebo  
skill. Keď chcete **ušetriť kontext** tým, že prácu odsuniete bokom, použite  
subagenta. A keď potrebujete, aby sa príkaz správal ako subagent, nastavte  
mu `context: fork`.  

## 17. Otázky na opakovanie

1. Prečo sa postup nehodí do `CLAUDE.md` a prečo patrí do príkazu?  
2. Ako zistíte, aké príkazy máte práve k dispozícii?  
3. Aký je rozdiel medzi `/compact` a `/clear`?  
4. Kde leží osobný príkaz a kde projektový? Kedy použiť ktorý?  
5. Ako by ste v tej istej relácii pokračovali z telefónu?  
6. Prečo má príkaz na odpoveď na e-mail `disable-model-invocation: true`?  
7. Čo sa stane, keď v hlavičke urobíte preklep v názve poľa?  

## 18. Úlohy na precvičenie

1. Napíšte príkaz `/tyzdenny-prehlad`, ktorý zobrazí dnešný dátum, vypíše  
   dokumenty v priečinku a zhrnie tie najnovšie. Použite dynamické  
   vkladanie.  
2. Vytvorte `/resers` s požiadavkou na päť zdrojov a overenie čísel  
   v dvoch nezávislých zdrojoch.  
3. Napíšte `/zapisnica`, ktorá z prepisu porady vytiahne rozhodnutia,  
   úlohy a otvorené otázky.  
4. Prepíšte `/zhrn-dokument` do formátu skillu v `.claude/skills/`.  
   Všimnite si, čo sa zmenilo.  
5. Pridajte do `/zhrn-dokument` pomenované argumenty a overte, čo sa  
   stane, keď jeden z nich vynecháte.  
6. Schválne umiestnite príkaz do nesprávneho priečinka a zistite, ako sa  
   to prejaví v ponuke `/`.  

## 19. Zhrnutie

- Príkaz je uložený prompt, ktorý spustíte lomítkom a menom.  
- Načíta sa až pri použití, preto je vhodný na postupy, nie na fakty.  
- Vlastný príkaz je Markdown súbor v `.claude/commands/` alebo  
  v `~/.claude/commands/`; novšie sa píše ako skill.  
- Hlavička určuje nápovedu, povolené nástroje, model a to, kto smie  
  príkaz spustiť.  
- Argumenty a `!` príkazy robia z príkazu nástroj, nie pevný text.  
- Príkazy sú súbory s pokynmi: čítajte ich, zdieľajte vedome  
  a nedávajte do nich tajomstvá.  

## 20. Slovníček

| Pojem                  | Význam                                                       |  
| ---------------------- | ------------------------------------------------------------ |  
| príkaz                 | text začínajúci `/`, ktorý spustí uložený postup             |  
| zabudovaný príkaz      | príkaz s pevnou logikou priamo v Claude Code                 |  
| zabalený skill         | prompt od Anthropicu dostupný ako príkaz                     |  
| vlastný príkaz         | váš Markdown súbor v `.claude/commands/`                     |  
| skill                  | príkaz s priečinkom, pomocnými súbory a popisom pre Clauda   |  
| hlavička (frontmatter) | YAML hlavička medzi `---` na začiatku súboru                 |  
| argument               | text za menom príkazu, dostupný ako `$ARGUMENTS` alebo `$0`  |  
| dynamické vkladanie    | spustenie `!` príkazu pred odoslaním a vloženie jeho výstupu |  
| menný priestor         | prefix z podpriečinka, napríklad `/frontend:component`       |  
| remote control         | ovládanie bežiacej relácie z iného zariadenia cez claude.ai  |  
| prompt injection       | pokyn ukrytý v cudzom texte, ktorý sa snaží ovládnuť agenta  |  

## Zdroje

- Claude Code – Commands: code.claude.com/docs/en/commands  
- Claude Code – Extend Claude with skills: code.claude.com/docs/en/skills  
- Claude Code – Plugins: code.claude.com/docs/en/plugins  
- Anthropic – Agent Skills: agentskills.io  
