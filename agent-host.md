# Orchestrátory a hostitelia agentov – riadiace veže pre AI  

V predchádzajúcich kapitolách sme spoznali jazykové modely a tokeny.  
Pri vývoji reálnych aplikácií však nepracujeme len s jedným modelom.  
Stretávame sa so systémami – agentnými harnessmi, napr. Claude Code,  
GitHub Copilot či Codex. Keď má vývojár viac nástrojov v editore,  
napr. v okne Agentov vo VS Code, potrebuje ich efektívne riadiť.  
Preto vstupujú na scénu agentní hostitelia a orchestrátory.  

*Stav k októbru 2026. Ekosystém sa rýchlo mení.*  

## Čo je agentný harness a prečo ho riadiť  

Agentný harness je rozhranie alebo obálka, ktorá umožňuje modelu  
konať ako autonómny agent: spúšťať príkazy v termináli, čítať a  
upravovať súbory, volať nástroje a interagovať s prostredím.  
Každý agent – Claude Code, Copilot alebo Codex – má vlastný harness.  
Ak chce vývojár v jednom projekte striedať silné stránky agentov,  
nemôže ich nechať bežať úplne izolovane. Preto použije orchestrátor,  
teda Agent Host, ktorý ich spravuje a koordinuje.  

### Úloha Agent Hostu  

Agent Host je aplikácia alebo vrstva vo vývojovom prostredí,  
napr. špeciálne okno agentov vo VS Code. Zabezpečuje:  
* spúšťanie a správu životného cyklu rôznych harnessov;  
* jednotné používateľské rozhranie na zadávanie úloh;  
* zdieľanie kontextu a pracovného priestoru medzi agentmi;  
* riadenie bezpečnostných oprávnení a prístupu k súborom či  
  terminálu.
  
### Protokoly a komunikácia  

Aby hostiteľ komunikoval s harnessmi od rôznych výrobcov,  
používajú sa špecializované protokoly, napr. Agent Host Protocol  
(AHP). Ten umožňuje jednotne posielať požiadavky, synchronizovať  
stav projektu a prijímať výstupy bez ohľadu na to, či beží harness  
pre Claude Code alebo Copilot.  


## Anatómia rozhrania Agent Hostu vo VS Code  

Pohľad na reálne rozhranie Agent Hostu vo VS Code ukazuje,  
ako sú nástroje a harnessy integrované do jedného prostredia:  

* Panel relácií (Sessions): ľavý bočný panel spravuje aktívne  
  a minulé konverzácie (Chats), automatizácie (Automations)  
  a priradené pracovné adresáre.  
* Prispôsobenia (Customizations): v spodnej časti ľavého panela  
  je centrálna správa rozšírení – doplnkov (Plugins), serverov  
  Model Context Protocol (MCP Servers), zručností (Skills)  
  a globálnych inštrukcií.  
* Výber harnessov a modelov: v strednej časti okna možno zvoliť  
  typ interakcie (napr. Chat), prepínať harnessy (napr. Copilot),  
  vybrať model (napr. DeepSeek Flash) a kontrolovať veľkosť  
  kontextového okna.  
* Riadenie oprávnení: v spodnej časti panela na zadávanie úloh  
  (Pitch your idea) sú prepínače režimov, napr. Interactive  
  a Manual permissions, ktoré bránia neautorizovaným zásahom.
  
## Porovnanie kľúčových pojmov  

* Agent Harness: izolované prostredie a ovládač pre jedného  
  konkrétneho agenta, napr. Claude Code.  
* Agent Host / Orchestrator: nadradená aplikácia, napr. okno  
  agentov vo VS Code, ktorá spravuje a koordinuje viac harnessov.  
* AHP (Agent Host Protocol): komunikačný štandard a protokol  
  medzi hostiteľom a jednotlivými harnessmi.
  
## Prečo prejsť na orchestrovaný prístup  

Správa viacerých agentov cez jedného hostiteľa prináša výhody:  

1. Flexibilita výberu: na komplexné refaktoringy použijeme  
   pokročilý harness; na rýchle dopĺňanie kódu prepneme iný  
   harness bez opustenia editora.  
2. Jednotný kontext: netreba znova kopírovať stav projektu  
   ani otvárať externé okná; hostiteľ udržiava kontext nad  
   tým istým pracovným priestorom.  
3. Bezpečnosť a kontrola: jednotný bod kontroly oprávnení  
   bráni agentom vykonať deštruktívne príkazy v systéme.  
## Zhrnutie kapitoly  
* Agentný harness je ovládač a prostredie, ktoré robí z modelu  
  autonómneho agenta schopného pracovať s kódom a nástrojmi.  
* Agent Host / Orchestrátor, napr. okno agentov vo VS Code,  
  spravuje, prepína a koordinuje viacero harnessov pod jednou  
  strechou.  
* AHP (Agent Host Protocol) zabezpečuje štandardizovanú  
  komunikáciu medzi hostiteľom a rôznymi agentnými harnessmi.  
* Orchestrácia umožňuje vývojárom efektívne kombinovať silné  
  stránky rôznych AI systémov v jednom pracovnom prostredí.
