# AI workflows

*Verzia 1.0 · stav k októbru 2026. Nástroje a modely sa menia rýchlo; pred  
nasadením si overte aktuálnu dokumentáciu.*  

## 1. Prečo workflow

Jeden dobrý prompt vyrieši jednu otázku. Skutočná práca sa však málokedy  
skončí jednou odpoveďou. Faktúru treba prečítať, overiť, zaúčtovať a poslať  
na schválenie. Článok treba rešeršovať, napísať, skontrolovať a publikovať.  
Keď tieto kroky robíte stále dokola, prestáva byť hlavný problém kvalita  
promptu a začína byť hlavný problém proces.  

AI workflow je spôsob, ako spojiť model, kód a nástroje do opakovateľného  
postupu. Model v ňom rieši to, čo nevie kód: porozumie textu, rozhodne  
o význame, napíše návrh. Kód naopak riadi poradie, kontroluje podmienky  
a vykonáva presné operácie.  

> **Jednoducho povedané:** workflow je recept, agent je kuchár, ktorý si  
> recept vymýšľa za pochodu. Recept sa ľahšie testuje, ladí a vysvetľuje.  

## 2. Čo je AI workflow

AI workflow je vopred navrhnutá postupnosť krokov, v ktorej aspoň jeden krok  
používa jazykový model. O poradí krokov rozhoduje program alebo konfigurácia,  
nie model. Každý krok má jasný vstup, výstup a spôsob kontroly.  

Príklady:  

- príjem faktúr: extrakcia údajov, kontrola súm, zaúčtovanie,  
- zákaznícka podpora: klasifikácia požiadavky, návrh odpovede, schválenie,  
- obsah: rešerš, osnova, koncept, faktická kontrola, publikovanie,  
- nábor: triedenie životopisov, zhrnutie, návrh otázok, pozvánka,  
- reporting: stiahnutie dát, výpočet metrík, komentár, odoslanie.  

Spoločný znak: postup sa opakuje, dá sa zmerať a pri každom behu má rovnakú  
kostru. Mení sa len obsah vstupov.  

### Workflow verzus agent

Rozdiel je v tom, kto rozhoduje o ďalšom kroku. Vo workflow je to vopred  
daný algoritmus. U agenta je to model, ktorý si postup volí sám podľa toho,  
čo práve zistil.  

| Otázka | Workflow | Agent |  
|---|---|---|  
| Kto určuje poradie krokov? | kód alebo konfigurácia | model |  
| Čo dostane na vstupe? | konkrétnu úlohu a dáta | cieľ |  
| Dá sa postup vopred vykresliť? | áno | len približne |  
| Predvídateľnosť | vyššia | nižšia |  
| Náklady na beh | zvyčajne nižšie | zvyčajne vyššie |  
| Vhodné pre | opakované procesy | otvorené problémy |  

Hranica nie je ostrá. Workflow môže mať jeden agentický krok, napríklad  
„prejdi dokumentáciu a nájdi odpoveď“. Agent môže vo vnútri používať  
workflow ako nástroj. V praxi často vyhráva kombinácia: pevná kostra  
a v nej niekoľko miest, kde sa model rozhoduje.  

## 3. Z čoho sa workflow skladá

```
Trigger (nový e-mail, súbor, časovač, webhook)
   │
   ▼
Krok 1: kód ──► Krok 2: model ──► Krok 3: kód
                                      │
                                 rozhodnutie
                                 ┌────┴────┐
                                 ▼         ▼
                            schváliť    človek
                                 │         │
                                 └────┬────┘
                                      ▼
                              výstup + záznam
```

| Prvok | Čo robí | Príklad |  
|---|---|---|  
| Trigger | spustí workflow | nový e-mail, nový súbor, cron |  
| Vstup | prinesie dáta a kontext | PDF faktúra, text ticketu |  
| Krok | vykoná jednu ohraničenú prácu | extrakcia, klasifikácia, výpočet |  
| Model | rieši nejednoznačný obsah | „o aký typ požiadavky ide?“ |  
| Nástroj | dosiahne na vonkajší systém | databáza, e-mail, API |  
| Stav | pamätá si priebeh behu | ID behu, spracované kroky |  
| Vetvenie | rozhodne podľa výsledku | istota nízka → človek |  
| Schválenie | zastaví nezvratnú akciu | platba, odoslanie zákazníkovi |  
| Výstup | uloží alebo doručí výsledok | záznam, e-mail, report |  
| Záznam | umožní spätnú kontrolu | vstupy, výstupy, tokeny, chyby |  

Prvky, ktoré sa často vynechajú a potom chýbajú: **stav**, **vetvenie pre  
chyby** a **záznam**. Bez nich sa workflow nedá zastaviť, obnoviť ani  
vysvetliť, prečo pri konkrétnom vstupe zlyhalo.  

## 4. Základné vzory

Väčšina workflow sa dá poskladať z niekoľkých opakujúcich sa vzorov.  
Nemusíte ich vymýšľať od nuly; stačí rozpoznať, ktorý sa hodí na vašu úlohu.  

### Reťazenie (prompt chaining)

Výstup jedného kroku je vstupom ďalšieho. Namiesto jedného obrovského  
promptu rozdelíte prácu na menšie, kontrolovateľné časti.  

- **Použitie:** článok (rešerš → osnova → koncept → úprava), dokument  
  (extrakcia → kontrola → zhrnutie).  
- **Výhoda:** každý krok sa dá samostatne testovať a vymeniť model.  
- **Riziko:** chyba z prvého kroku sa prenáša ďalej. Preto medzi kroky  
  patrí validácia.  

### Smerovanie (routing)

Najprv sa určí typ vstupu a podľa neho sa vyberie vetva. Klasifikácia  
býva lacná a rýchla, takže stačí malý model.  

- **Použitie:** ticket je fakturácia, technický problém, reklamácia alebo  
  iné; dokument je zmluva, faktúra alebo správa.  
- **Výhoda:** každá vetva môže mať vlastný prompt, nástroje a model.  
- **Riziko:** nesprávna kategória pošle vec do zlého procesu. Vždy majte  
  vetvu „neviem“ s odovzdaním človeku.  

### Paralelizácia

Nezávislé časti bežia súčasne a výsledky sa spoja.  

- **Použitie:** kontrola dokumentu z pohľadu práva, financií a IT;  
  viacero nezávislých návrhov odpovede.  
- **Výhoda:** kratší čas a niekedy lepší výsledok vďaka viacerým pohľadom.  
- **Riziko:** vyššia cena a potreba zlúčiť rozporné výstupy.  

### Orchestrátor a pracovníci

Jeden krok rozdelí úlohu na podúlohy, viacerí pracovníci ich spracujú  
a orchestrátor výsledky spojí.  

- **Použitie:** rešerš z viacerých zdrojov, kontrola veľkého projektu,  
  spracovanie dokumentu po kapitolách.  
- **Výhoda:** zvládne väčší rozsah, než sa zmestí do jedného kontextu.  
- **Riziko:** strata konzistencie a vyššie náklady. Hodí sa až pri väčších  
  úlohách, nie na jednoduchý preklad.  

### Hodnotiteľ a opravovateľ

Jeden model vytvorí návrh, druhý ho podľa kritérií ohodnotí a prvý ho  
opraví. Cyklus sa opakuje, kým hodnotiteľ neschváli alebo kým sa  
neminie limit pokusov.  

- **Použitie:** text, kód, formátovanie, kontrola súladu s pravidlami.  
- **Výhoda:** kvalita rastie bez zásahu človeka.  
- **Riziko:** model môže hodnotiť vlastnú chybu ako správnu. Dôležité je  
  mať konkrétne kritériá a tvrdý limit opakovaní.  

### RAG workflow

Pred odpoveďou sa z databázy alebo dokumentov vyhľadajú relevantné pasáže  
a vložia sa do kontextu. Model tak odpovedá z vašich dát, nie z pamäte.  

Typický postup: dotaz → vyhľadanie → preusporiadanie výsledkov → odpoveď  
so zdrojmi → kontrola, či odpoveď vychádza z nájdených pasáží.  

- **Výhoda:** aktuálne a firemné znalosti, odkazy na zdroje.  
- **Riziko:** zlý výber dokumentov. Model potom sebavedomo odpovie z  
  nesprávneho kontextu.  

### Človek v slučke

Workflow sa v určenom bode zastaví a čaká na schválenie alebo doplnenie.  

- **Použitie:** platby, zmluvy, odpovede zákazníkom, mazanie dát.  
- **Pravidlo:** človek patrí pred každú akciu, ktorá sa nedá vrátiť alebo  
  ktorá má právny či finančný dôsledok.  
- **Riziko:** príliš veľa schvaľovaní zabije výhody automatizácie.  
  Schvaľujte len to, čo naozaj treba, a zvyšok spoľahlivo validujte kódom.  

## 5. Ako workflow navrhnúť

1. **Definujte výstup.** Nie „pomôž s faktúrami“, ale „z PDF vytvor  
   zaúčtovateľný záznam alebo ho pošli na manuálnu kontrolu“.  
2. **Zmerajte súčasný stav.** Koľko prípadov, koľko času, koľko chýb.  
   Bez toho neviete, či sa workflow vôbec oplatí.  
3. **Rozdeľte prácu na kroky.** Každý krok musí mať overiteľný výstup.  
   Ak sa krok nedá skontrolovať, je pravdepodobne príliš veľký.  
4. **Rozdeľte prácu medzi kód a model.** Presné, deterministické operácie  
   nechajte kódu. Model dostane len nejednoznačnosť.  
5. **Navrhnite výstupnú schému.** JSON alebo Pydantic model s povinnými  
   poľami. Voľný text sa na ďalšie spracovanie nehodí.  
6. **Pridajte vetvy pre chyby.** Nízka istota, neplatný vstup, výpadok  
   API, prekročený limit. Každá vetva musí mať jasné správanie.  
7. **Otestujte na vzorke a merajte.** Kým nemáte výsledky na aspoň  
   niekoľkých desiatkach reálnych prípadov, nedávajte workflow naostro.  

> **Zlaté pravidlo:** čo sa dá overiť kódom, patrí kódu. Model nemá  
> počítať sumu, ktorú vie spočítať kalkulačka, ani rozhodovať, či je  
> dátum v minulosti.  

| Úloha | Kto ju má robiť |  
|---|---|  
| prečítanie PDF alebo DOCX | kód alebo knižnica |  
| rozpoznanie polí z textu | model |  
| súčet položiek a DPH | kód |  
| kontrola duplicitného čísla | kód a databáza |  
| klasifikácia zámeru | model |  
| voľba šablóny odpovede | kód podľa kategórie |  
| napísanie textu | model |  
| odoslanie e-mailu | kód s potvrdením |  

## 6. Príklad: spracovanie faktúr

Zadanie: „Každá faktúra, ktorá príde e-mailom, sa má dostať do  
účtovníctva. Podozrivé prípady majú ísť človeku.“  

```
e-mail s prílohou
      │
      ▼
1. kód: uloží PDF, založí ID behu
      │
      ▼
2. kód: vytiahne text (PDF parser)
      │
      ▼
3. model: extrahuje polia do schémy Invoice
      │
      ▼
4. kód: overí sumy, dátumy, DPH, duplicitu
      │
      ├── chyba alebo suma > limit ──► 5. človek
      │                                    │
      ▼                                    ▼
6. kód: zaúčtuje s idempotentným kľúčom
      │
      ▼
7. kód: zapíše výsledok a pošle oznámenie
```

Kritické je poradie. Model extrahuje údaje, ale nikdy nezaúčtuje. Sumu  
overí kód. Ak kontrola zlyhá, workflow sa zastaví a čaká na človeka.  
Zaúčtovanie používa číslo faktúry ako idempotentný kľúč, takže opakovaný  
beh nevytvorí duplicitný záznam.  

### Kostra v Pythone

```python
from pathlib import Path

def process_invoice(path: Path) -> None:
    text = extract_text(path)              # kód: PDF -> text
    invoice = extract_invoice(text)        # model: text -> Invoice
    problems = validate(invoice)           # kód: sumy, dátumy, duplicita

    if problems or invoice.total > LIMIT:
        queue_for_human(path, invoice, problems)
        return

    post_to_accounting(invoice, key=invoice.number)
    notify(f"Faktúra {invoice.number} zaúčtovaná")
```

Extrakcia cez štruktúrovaný výstup. Knižnica overí, či odpoveď sedí so  
schémou, a vráti rovno objekt, nie text na ďalšie parsovanie:  

```python
def extract_invoice(text: str) -> Invoice:
    response = client.messages.parse(
        model=MODEL,
        max_tokens=4000,
        messages=[{
            "role": "user",
            "content": PROMPT + text,
        }],
        output_format=Invoice,
    )
    return response.parsed_output
```

Validácia je obyčajný kód. Práve ona rozhoduje, či výsledok modelu smie  
pokračovať ďalej:  

```python
def validate(inv: Invoice) -> list[str]:
    errors = []

    if abs(sum(i.total for i in inv.items) - inv.total) > 0.01:
        errors.append("súčet položiek nesedí s celkovou sumou")
    if inv.due < inv.issued:
        errors.append("splatnosť je pred dátumom vystavenia")
    if invoice_exists(inv.number):
        errors.append("duplicitné číslo faktúry")

    return errors
```

Týmto spôsobom sa chyba modelu prejaví ako zoznam problémov, nie ako  
nesprávny zápis v účtovníctve. Workflow zároveň vie, čo má urobiť: poslať  
prípad človeku a zapísať dôvod.  

## 7. Príklad: triedenie zákazníckych požiadaviek

Zadanie: „Tickety majú ísť správnemu tímu a jednoduché majú dostať návrh  
odpovede.“ Ide o vzor smerovania.  

| Kategória | Komu | Automatická akcia |  
|---|---|---|  
| fakturácia | financie | návrh odpovede, bez odoslania |  
| technický problém | podpora 2. úrovne | doplnenie diagnostiky |  
| reklamácia | reklamačné oddelenie | žiadosť o číslo objednávky |  
| sťažnosť | vedúci podpory | len zatriedenie, bez návrhu |  
| neviem | univerzálny front | človek rozhodne |  

Postup: model vytriedi ticket do kategórie a vráti aj mieru istoty. Kód  
skontroluje, či kategória patrí medzi povolené. Ak je istota nízka, ticket  
ide na univerzálny front. Pri fakturácii sa návrh odpovede uloží ako  
koncept; odoslanie zostáva na človeku, pretože ide o finančnú komunikáciu.  

Tento workflow má tri dôležité vlastnosti:  

- **obmedzený priestor rozhodnutia:** model vyberá len z uzavretého  
  zoznamu kategórií,  
- **jasnú zodpovednosť:** kód rozhoduje, čo sa s kategóriou stane,  
- **spätnú väzbu:** ak ľudia kategóriu často prepisujú, je to signál na  
  úpravu promptu alebo kategórií.  

## 8. Príklad: obsahový pipeline

Zadanie: „Z témy vznikne publikovateľný článok so zdrojmi.“ Tento proces  
kombinuje reťazenie, paralelizáciu a hodnotiteľa.  

1. **Rešerš:** model navrhne otázky, nástroje vyhľadajú zdroje.  
2. **Osnova:** model zozbiera hlavné body a priradí im zdroje.  
3. **Koncept:** model napíše text po častiach, aby nezabudol na štruktúru.  
4. **Faktická kontrola:** samostatný krok overí tvrdenia proti zdrojom.  
5. **Štýl a úprava:** hodnotiteľ skontroluje jasnosť, dĺžku a tón.  
6. **Publikovanie:** človek schváli a kód článok zverejní.  

Najčastejšia chyba je vynechať krok 4. Text potom pôsobí dobre, ale  
obsahuje vymyslené alebo nepresné údaje. Faktická kontrola musí mať  
prístup k zdrojom a musí vracať konkrétne tvrdenia, ktoré sa nepodarilo  
overiť, nie len všeobecné „vyzerá to dobre“.  

## 9. Nástroje

| Nástroj | Typ | Kedy sa hodí |  
|---|---|---|  
| n8n | no-code, možno hostovať | prepojenie služieb, malé tímy |  
| Make, Zapier | no-code SaaS | rýchly štart, bežné aplikácie |  
| LangGraph | knižnica v Pythone alebo JS | stavové workflow s vetvením |  
| Temporal | infraštruktúra | dlhé procesy, spoľahlivé opakovanie |  
| Prefect, Airflow | orchestrácia dát | dávkové a plánované úlohy |  
| Dify, Flowise | low-code LLM aplikácie | prototypy, RAG, interné nástroje |  
| vlastný kód a SDK | kód | plná kontrola, menší rozsah |  

Voľba závisí od toho, kto bude workflow udržiavať. No-code nástroj zvládne  
bežnú integráciu za hodinu, ale zložité vetvenie a testovanie v ňom bolí.  
Knižnica alebo vlastný kód dáva kontrolu, no vyžaduje programátora.  
Pri dlhých procesoch s čakaním na človeka je dôležité, aby nástroj vedel  
stav uložiť a po reštarte pokračovať.  

Praktické pravidlo: začnite najjednoduchším nástrojom, ktorý zvládne  
požadovanú spoľahlivosť. Agentický framework nepotrebujete, ak má váš  
proces pevné kroky.  

## 10. Spoľahlivosť a testovanie

### Idempotencia

Každý krok, ktorý mení stav, musí byť bezpečné spustiť dvakrát. Použite  
jedinečný kľúč (číslo faktúry, ID ticketu, hash vstupu) a pred zápisom  
skontrolujte, či už nebol spracovaný. Výpadok siete alebo opakovanie  
inak vytvorí duplicitnú platbu či e-mail.  

### Opakovanie a časové limity

Pri prechodných chybách (timeout, limit API, dočasný výpadok) pomáha  
obmedzené opakovanie s rastúcou pauzou. Pri trvalej chybe sa má beh  
ukončiť a uložiť na neskoršie spracovanie. Nikdy neopakujte donekonečna  
krok, ktorý míňa peniaze alebo mení dáta.  

### Validácia výstupov

Každý výstup modelu overte: schémou, povolenými hodnotami, rozsahmi  
a obchodnými pravidlami. Ak výstup neprejde, neopravujte ho naslepo  
ďalším promptom. Pošlite ho na opravu alebo človeku.  

### Testovacia sada a eval

Vytvorte sadu aspoň 20 – 50 reprezentatívnych vstupov so správnymi  
výstupmi. Pri každej zmene promptu alebo modelu ju spustite a porovnajte  
výsledky. Sledujte nielen presnosť, ale aj podiel prípadov, ktoré musel  
prebrať človek, cenu a čas.  

### Pozorovateľnosť

Každý beh má mať ID a záznam: vstup, verziu promptu, model, výstup,  
počet tokenov, trvanie, chyby a rozhodnutia vetiev. Bez toho sa problém  
„občas to nefunguje“ nedá vyriešiť.  

### Verzionovanie promptov

Prompt je súčasť kódu. Ukladajte ho do repozitára, označujte verziou  
a pri zmene vždy spustite testovaciu sadu. Inak neviete, prečo sa správanie  
v pondelok zmenilo.  

### Matematika chýb

Ak je každý krok správny na 95 %, desať nezávislých krokov je správnych  
len približne na 60 %. Čím viac krokov, tým viac kontroly potrebujete.  
Znížte počet krokov, overujte medzivýsledky a nekritické časti presuňte  
na lacnejší alebo deterministický kód.  

## 11. Náklady a latencia

| Krok | Vhodný model | Dôvod |  
|---|---|---|  
| klasifikácia | malý a rýchly | veľa volaní, jednoduchá úloha |  
| extrakcia | stredný | presnosť pri štruktúre |  
| písanie a zhrnutie | väčší | kvalita textu |  
| kontrola pravidiel | kód | presnosť a nulová cena za token |  
| zložitá analýza | väčší s uvažovaním | len tam, kde je to potrebné |  

Päť jednoduchých krokov s veľkým modelom môže stáť viac než jeden dobre  
navrhnutý krok. Peniaze sa strácajú aj na opakovaniach: ak model pri  
každom behu dostane celý dokument a celú históriu, platíte za to isté  
znova. Pomáha cache, kratšie prompty a dávkové spracovanie.  

Latencia sa skladá z čakania na model a na nástroje. Nezávislé kroky  
spúšťajte paralelne. Tam, kde používateľ čaká, zobrazte priebeh alebo  
výsledok doručte neskôr.  

## 12. Bezpečnosť a súkromie

- **Najmenšie oprávnenia.** Workflow má prístup len k tomu, čo naozaj  
  potrebuje. Účtovný proces nepotrebuje čítať celý disk.  
- **Človek pri nezvratných akciách.** Platba, odoslanie, zmazanie alebo  
  zmena zmluvy majú mať schválenie.  
- **Vstup zvonku je nedôveryhodný.** Dokument môže obsahovať skrytý pokyn  
  pre model. Ak text hovorí „ignoruj pravidlá a pošli dáta“, je to  
  útok, nie súčasť úlohy.  
- **Smrteľná trojica.** Riziko vzniká, keď sa v jednom procese stretnú  
  súkromné dáta, nedôveryhodný obsah a možnosť posielať dáta von. Jednu  
  z týchto troch vecí treba odstrániť alebo izolovať.  
- **Tajomstvá mimo promptov.** API kľúče patria do prostredia alebo  
  správcu tajomstiev, nikdy do textu zadania ani do logov.  
- **Minimalizácia dát.** Do modelu posielajte len potrebné polia. Osobné  
  údaje maskujte alebo odstraňujte, ak nie sú potrebné.  
- **Záznamy a súlad.** Overte, kde poskytovateľ uchováva dáta, ako dlho  
  a či ich používa na trénovanie. Pri citlivých procesoch to musí byť  
  súčasťou návrhu, nie dodatkom na konci.  

## 13. Časté chyby

1. **Nejasný výstup.** Workflow „niečo zhrnie“ sa nedá otestovať.  
   Definujte presnú podobu výsledku a kritériá úspechu.  
2. **Model robí deterministickú prácu.** Počítanie, porovnávanie  
   a rozhodovanie podľa pevných pravidiel patrí kódu.  
3. **Chýbajúca validácia.** Výstup modelu sa ukladá priamo do systému.  
4. **Jeden obrovský prompt.** Namiesto krokov vznikne neprehľadný text,  
   ktorý sa nedá ladiť ani čiastočne opakovať.  
5. **Žiadny človek pri riziku.** Automatické odoslanie alebo platba bez  
   schválenia je najdrahšia chyba.  
6. **Žiadne záznamy.** Bez logov sa nedá zistiť, prečo sa výsledok zmenil.  
7. **Agent tam, kde stačí workflow.** Flexibilita sa platí vyššou cenou  
   a nižšou predvídateľnosťou.  
8. **Testovanie len na jednom príklade.** Reálne vstupy sú špinavé,  
   neúplné a neštandardné.  
9. **Ignorovanie nákladov.** Každý krok a každé opakovanie niečo stojí.  
10. **Nikto workflow nevlastní.** Bez zodpovednej osoby sa kvalita  
    postupne zhoršuje a chyby nikto neopraví.  

## 14. Kedy workflow nepotrebujete

- Úlohu robíte raz alebo veľmi zriedka; ručná práca je lacnejšia.  
- Výsledok sa nedá overiť, takže automatizácia len presunie riziko.  
- Proces sa mení pri každom prípade natoľko, že pevné kroky prekážajú.  
- Objem je malý a úspora času nevyváži náklady na údržbu.  
- Ide o citlivé rozhodnutie, ktoré musí posúdiť človek od začiatku do  
  konca.  
- Chýbajú dáta alebo prístup k systému, bez ktorých sa kroky nedajú  
  vykonať.  

V týchto prípadoch stačí lepší prompt, šablóna alebo jednoduchý agent  
s človekom v slučke. Workflow nie je cieľ, ale nástroj na opakovateľnú  
prácu.  

## 15. Úlohy na precvičenie

1. Vyberte si opakovanú úlohu zo svojho týždňa a nakreslite jej kroky.  
   Ku každému kroku napíšte, či ho má robiť kód, model alebo človek.  
2. Pre workflow „zápisnica zo stretnutia“ navrhnite vstup, výstup a tri  
   kontrolné body. Kde by ste zastavili proces a prečo?  
3. Do príkladu s faktúrami pridajte kontrolu duplicitného čísla ešte pred  
   extrakciou, ak je číslo dostupné z názvu súboru.  
4. Zmerajte cenu jedného behu svojho workflow. Koľko tokenov spotrebuje  
   každý krok a ktorý krok by zvládol menší model?  
5. Navrhnite testovaciu sadu 20 vstupov pre triedenie ticketov. Zahrňte  
   aj prázdny text, cudzí jazyk a pokus o vloženie pokynu.  
6. Rozhodnite, či je pre tieto zadania vhodnejší workflow alebo agent:  
   a) denný report z databázy, b) rešerš novej témy, c) schvaľovanie  
   dovoleniek, d) odpoveď na neznámu sťažnosť.  
7. Pridajte do faktúrového workflow limit, nad ktorým sa vždy vyžaduje  
   schválenie. Vysvetlite, podľa čoho ste limit zvolili.  

## 16. Zhrnutie

- **AI workflow** je vopred navrhnutá postupnosť krokov, v ktorej model  
  rieši nejednoznačný obsah a kód riadi priebeh a kontroly.  
- Od agenta sa líši tým, **kto rozhoduje o ďalšom kroku**. Workflow má  
  pevnú kostru, agent si postup volí sám.  
- Základné vzory sú **reťazenie, smerovanie, paralelizácia, orchestrátor  
  s pracovníkmi, hodnotiteľ s opravovateľom, RAG a človek v slučke**.  
- Každý krok má mať jasný vstup, výstup a validáciu. Deterministickú  
  prácu robí kód, nie model.  
- Spoľahlivosť stojí na **idempotencii, opakovaní s limitom, validácii,  
  testovacej sade, záznamoch a verzionovaní promptov**.  
- Pri nezvratných akciách patrí do procesu **človek**. Pri práci  
  s externým obsahom myslite na **vkladanie pokynov a smrteľnú trojicu**.  
- Workflow sa neoplatí pri jednorazovej, neoveriteľnej alebo veľmi  
  premenlivej práci.  

## 17. Slovníček

| Anglicky | Slovensky | Význam |  
|---|---|---|  
| workflow | postup alebo proces | vopred daná postupnosť krokov |  
| trigger | spúšťač | udalosť, ktorá začne beh |  
| step | krok | jedna ohraničená časť práce |  
| orchestration | riadenie toku | určovanie poradia a vetiev |  
| structured output | štruktúrovaný výstup | odpoveď vo vopred danej schéme |  
| validation | validácia | kontrola výstupu podľa pravidiel |  
| idempotency | idempotencia | bezpečné opakovanie bez duplicit |  
| retry | opakovanie | nový pokus po prechodnej chybe |  
| guardrail | ochranné pravidlo | limit alebo kontrola správania |  
| human-in-the-loop | človek v slučke | schválenie alebo zásah človeka |  
| eval | testovacia sada | vzorky so správnymi výstupmi |  
| observability | pozorovateľnosť | záznamy o priebehu a výsledkoch |  
| RAG | vyhľadávanie a generovanie | odpoveď z vyhľadaných dokumentov |  
| prompt injection | vloženie pokynu | skrytý pokyn v cudzom obsahu |  

## Zdroje

- Anthropic, „Building effective agents“ (2024): anthropic.com/engineering  
- OpenAI, „A practical guide to building agents“ (2025): openai.com  
- LangGraph – dokumentácia: langchain-ai.github.io/langgraph  
- Temporal – dokumentácia: docs.temporal.io  
- n8n – dokumentácia: docs.n8n.io  
- Prefect – dokumentácia: docs.prefect.io  
- Simon Willison, „The lethal trifecta for AI agents“ (2025):  
  simonwillison.net  
