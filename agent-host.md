# Orchestrátory a hostitelia agentov – riadiace veže pre AI

V predchádzajúcich kapitolách sme spoznali jednotlivé jazykové modely a ich tokeny. Keď však začneme vyvíjať reálne aplikácie alebo písať kód, nestretávame sa len s jedným modelom, ale s celými systémami – **agentnými harnessmi** ako Claude Code, GitHub Copilot či Codex. Keď má vývojár k dispozícii viacero takýchto nástrojov naraz priamo v editore (napr. v okne Agentov vo VS Code), vzniká potreba ich efektívne riadiť. Tu prichádzajú na scénu **agentní hostitelia (Agent Hosts)** a **orchestrátory**.

*Stav k októbru 2026. Ekosystém vývojárskych agentov a vývojových prostredí sa rýchlo mení.*

## Čo je agentný harness a prečo ho potrebujeme riadiť

**Agentný harness** (*agent harness*) je špecializované programové rozhranie alebo obálka, ktorá umožňuje jazykovému modelu konať ako autonómny agent – spúšťať príkazy v termináli, čítať a upravovať súbory, volať nástroje a interagovať s vývojovým prostredím. Každý agent (napr. Claude Code, Copilot alebo Codex) prichádza s vlastným harnessom.

Ak chce vývojár v jednom projekte striedať silné stránki rôznych agentov, nemôže ich nechať bežať v úplnej izolácii. Na to slúži **orchestrátor agentov** (alebo **Agent Host**).

### Úloha Agent Hostu

Agent Host je aplikácia alebo vrstva v rámci vývojového prostredia (napr. špeciálne okno agentov vo VS Code), ktorá zabezpečuje:

* **Spúšťanie a správu** životného cyklu rôznych harnessov;
* **Jednotné používateľské rozhranie** pre zadávanie úloh;
* **Zdieľanie kontextu** a pracovného priestoru medzi rôznymi agentmi;
* **Riadenie bezpečnostných oprávnení** a prístupu k súborom či terminálu.

### Protokoly a komunikácia

Na to, aby hostiteľ dokázal komunikovať s rôznymi harnessmi od rôznych výrobcov, slúžia špecializované protokoly, ako je napríklad **Agent Host Protocol (AHP)**. Vďaka nemu môže hostiteľ jednotne posielať požiadavky, synchronizovať stav projektu a prijímať výstupy bez ohľadu na to, či v pozadí práve behá harness pre Claude Code alebo Copilot.

Tu je doplnenie kapitoly o podrobný popis používateľského rozhrania Agent Hostu tak, ako ho vidíme vo VS Code:


## Anatómia rozhrania Agent Hostu vo VS Code

Pohľad na reálne rozhranie Agent Hostu vo VS Code ukazuje, ako sú tieto pokročilé nástroje a harnessy integrované do jednotného pracovného prostredia vývojára:

* **Panel relácií (*Sessions*):** Ľavý bočný panel slúži na správu aktívnych a minulých konverzácií (*Chats*), automatizácií (*Automations*) a priradených pracovných adresárov.


* **Prispôsobenia (*Customizations*):** V spodnej časti ľavého panela sa nachádza centrálna správa rozšírení vrátane doplnkov (*Plugins*), serverov protokolu Model Context Protocol (*MCP Servers*), špecifických zručností (*Skills*) a globálnych inštrukcií.


* **Výber harnessov a modelov:** V strednej časti okna si vývojár môže vybrať typ interakcie (napr. *Chat*), prepínať medzi dostupnými harnessmi (napr. *Copilot*), zvoliť konkrétny jazykový model (napr. *DeepSeek Flash*) a kontrolovať veľkosť kontextového okna.


* **Riadenie oprávnení:** V spodnej časti panela pre zadávanie úloh (*Pitch your idea*) sa nachádzajú prepínače bezpečnostných režimov, ako je interaktívna práca alebo manuálne schvaľovanie oprávnení (*Interactive, Manual permissions*), ktoré bránia neautorizovaným zásahom do kódu.


## Porovnanie kľúčových pojmov

Pre lepšiu orientáciu v architektúre moderných AI nástrojov si zhrňme základné pojmy:

| Pojem | Význam v kontexte AI agentov |
| --- | --- |
| **Agent Harness** | Izolované prostredie a ovládač pre jedného konkrétneho agenta (napr. Claude Code). |
| **Agent Host / Orchestrator** | Nadradená aplikácia (napr. okno agentov v VS Code), ktorá spravuje a koordinuje viacero harnessov naraz. |
| **AHP (Agent Host Protocol)** | Komunikačný štandard a protokol medzi hostiteľom a jednotlivými harnessmi. |


## Prečo prechádzať na orchestrovaný prístup

Správa viacerých agentov cez jedného hostiteľa prináša niekoľko zásadných výhod:

1. **Flexibilita výberu:** Na komplexné refaktoringy môžeme použiť pokročilý agentný harness, zatiaľ čo na rýchle dopĺňanie kódu prepneme na iný harness bez nutnosti opustiť editor.
2. **Jednotný kontext:** Netreba znova a znova kopírovať stav projektu alebo otvárať externé okná; hostiteľ udržiava kontext nad rovnakým pracovným priestorom.
3. **Bezpečnosť a kontrola:** Jednotný bod kontroly pre oprávnenia, ktoré bránia agentom vykonať deštruktívne príkazy v systéme.

## Zhrnutie kapitoly

* **Agentný harness** je ovládač a prostredie, ktoré robí z jazykového modelu autonómneho agenta schopného pracovať s kódom a nástrojmi.
* **Agent Host / Orchestrátor** (napr. okno agentov v VS Code) je aplikácia, ktorá spravuje, prepína a koordinuje viacero harnessov pod jednou strechou.
* **AHP (Agent Host Protocol)** zabezpečuje štandardizovanú komunikáciu medzi hostiteľom a rôznymi agentnými harnessmi.
* Orchestrácia umožňuje vývojárom efektívne kombinovať silné stránky rôznych AI systémov v rámci jedného pracovného prostredia.

