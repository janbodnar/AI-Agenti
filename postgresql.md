# PostgreSQL: databáza pre AI agentov a workflowy

*Verzia 1.0 · stav k októbru 2026. Nástroje sa menia rýchlo; pred  
nasadením si overte aktuálnu dokumentáciu PostgreSQL, pgAdmin a VS Code.*  

## 1. Prečo databáza v AI školení

AI agent alebo workflow si sám od seba nič nepamätá. Keď má evidovať  
objednávky, účastníkov kurzov alebo výsledky vlastných behov, potrebuje  
miesto, kde dáta bezpečne uloží a neskôr ich nájde. Na to slúži  
**databáza**. Súbor CSV alebo tabuľka v Exceli stačí na začiatok, no  
pri viacerých používateľoch, väčšom objeme a súbežných zápisoch prestáva  
stačiť.  

**PostgreSQL** (skrátene Postgres) je bezplatná databáza s otvoreným  
kódom, vyvíjaná od roku 1996. Používajú ju malé firmy aj veľké platformy  
a stojí na nej aj množstvo cloudových služieb, napríklad Supabase alebo  
Azure Database for PostgreSQL. Pre AI projekty je zaujímavá aj tým, že  
vie ukladať dokumenty JSON a s doplnkom pgvector aj vektory pre  
sémantické vyhľadávanie.  

V tejto kapitole prejdete celý postup:  

1. nainštalujete PostgreSQL a pgAdmin na Windows,  
2. pripojíte sa k databáze z VS Code cez doplnok od Microsoftu,  
3. vytvoríte tabuľky, naplníte ich testovacími dátami a naučíte sa  
   základné príkazy SQL,  
4. postavíte tabuľku na záznam behov AI workflowov,  
5. bezpečne sprístupníte databázu AI agentovi.  

```mermaid
flowchart LR
    A[Workflow alebo agent] -->|zápis výsledku| D[(PostgreSQL)]
    D -->|dotaz SQL| A
    B[VS Code<br>doplnok PostgreSQL] --> D
    C[pgAdmin 4] --> D
    E[psql v termináli] --> D
```

> **Jednoducho povedané:** Databáza je archív so šuplíkmi (tabuľkami).  
> Každý šuplík má presne určené kolónky. SQL je jazyk, ktorým archivárovi  
> poviete, čo má uložiť, nájsť alebo zmeniť.  

## 2. Ako funguje relačná databáza

PostgreSQL je **relačná databáza**. Dáta ukladá do **tabuliek**, ktoré  
majú pevne dané **stĺpce** (napríklad meno, e-mail) a ľubovoľný počet  
**riadkov** (jeden riadok = jeden účastník). Tabuľky sa navzájom  
prepájajú cez **kľúče**.  

| Pojem | Význam | Príklad |  
|---|---|---|  
| databáza | samostatný celok tabuliek | `skolenie` |  
| tabuľka | súbor záznamov jedného druhu | `ucastnici` |  
| stĺpec | jedna vlastnosť s dátovým typom | `email text` |  
| riadok | jeden záznam | Mária Kováčová |  
| primárny kľúč | jedinečný identifikátor riadku | `id` |  
| cudzí kľúč | odkaz na riadok inej tabuľky | `kurz_id` |  
| schéma | menný priestor v databáze | `public` |  

PostgreSQL beží ako **server**. To je program, ktorý stále beží na  
pozadí a čaká na pripojenie, štandardne na **porte 5432**. Vy sa k nemu  
pripájate **klientom**: príkazovým riadkom `psql`, grafickým pgAdminom  
alebo doplnkom vo VS Code. Všetci klienti rozprávajú so serverom  
rovnakým jazykom – **SQL** (Structured Query Language).  

### Príkazy SQL podľa účelu

| Skupina | Príkazy | Na čo slúži |  
|---|---|---|  
| definícia | `CREATE`, `ALTER`, `DROP` | vytvorenie a zmena tabuliek |  
| čítanie | `SELECT` | vyhľadanie dát |  
| zmena dát | `INSERT`, `UPDATE`, `DELETE` | pridanie, úprava, zmazanie |  
| transakcie | `BEGIN`, `COMMIT`, `ROLLBACK` | zmeny ako jeden celok |  
| práva | `CREATE ROLE`, `GRANT`, `REVOKE` | kto smie čo robiť |  

### Najpoužívanejšie dátové typy

| Typ | Čo ukladá | Príklad hodnoty |  
|---|---|---|  
| `integer`, `bigint` | celé čísla | `42` |  
| `numeric(8, 2)` | presné desatinné čísla (peniaze) | `290.00` |  
| `text` | text ľubovoľnej dĺžky | `'Bratislava'` |  
| `date` | dátum | `'2026-11-10'` |  
| `timestamptz` | dátum a čas s časovým pásmom | `now()` |  
| `boolean` | áno alebo nie | `true` |  
| `jsonb` | dokument JSON | `'{"model": "x"}'` |  

> **Tip:** Na peniaze používajte `numeric`, nie `real` ani `double`.  
> Desatinné typy s pohyblivou čiarkou zaokrúhľujú a súčty potom  
> nesedia na cent.  

## 3. Inštalácia na Windows

Aktuálna hlavná verzia je **PostgreSQL 18** (posledná opravná verzia  
18.6). Verzia 19 je v októbri 2026 vo fáze Release Candidate, na  
školenie preto použite 18. Oficiálne odporúčaný inštalátor pre Windows  
pripravuje firma **EDB**. Okrem servera obsahuje aj **pgAdmin 4**,  
nástroj **StackBuilder** na doplnky a príkazový riadok `psql`.  

### Možnosť A: winget (odporúčané)

**winget** je správca balíkov zabudovaný vo Windows 10 a 11. Otvorte  
PowerShell alebo Terminal a spustite:  

```powershell
winget search PostgreSQL
winget install --id PostgreSQL.PostgreSQL.18 -e --interactive
```

Prepínač `--interactive` otvorí sprievodcu inštaláciou. V ňom:  

1. ponechajte všetky komponenty vrátane **pgAdmin 4** a **Command Line  
   Tools**,  
2. zadajte heslo pre správcu `postgres` a zapíšte si ho,  
3. port ponechajte **5432**,  
4. locale ponechajte predvolené,  
5. na konci StackBuilder zatiaľ nespúšťajte.  

Ak by ste chceli pgAdmin samostatne alebo v novšej verzii, než je  
v inštalátore:  

```powershell
winget install --id PostgreSQL.pgAdmin -e
```

### Možnosť B: inštalátor z webu

Na stránke **postgresql.org/download/windows** kliknite na „Download the  
installer“. Stiahnete inštalátor EDB pre verziu 18 a prejdete rovnakým  
sprievodcom ako vyššie.  

### Možnosť C: Docker

Ak už používate **Docker Desktop**, server spustíte bez inštalácie  
jedným príkazom. Kontajner je izolovaný program s vlastným systémom;  
po jeho zmazaní nezostanú vo Windows žiadne stopy.  

```powershell
docker run --name pg-skolenie -e POSTGRES_PASSWORD=zmente-heslo `
  -p 5432:5432 -d postgres:18
```

> **Pozor:** Docker a inštalácia z možnosti A nemôžu naraz používať port  
> 5432. Ak máte oboje, jeden z nich presuňte, napríklad `-p 5433:5432`.  

### Overenie inštalácie

Inštalátor EDB ukladá programy štandardne do priečinka  
`C:\Program Files\PostgreSQL\18\bin`. Ak príkaz `psql` vo Windows  
nenájde, pridajte priečinok do premennej `Path` (Štart → „Upraviť  
premenné prostredia systému“). Na skúšku stačí pridať ho len do  
aktuálneho okna:  

```powershell
$env:Path += ";C:\Program Files\PostgreSQL\18\bin"
psql --version
psql -U postgres -h localhost
```

Po zadaní hesla uvidíte výzvu `postgres=#`. Overte verziu servera  
a vytvorte databázu pre túto kapitolu:  

```sql
SELECT version();
CREATE DATABASE skolenie;
```

Okno `psql` zatvoríte príkazom `\q`. Či server beží ako služba Windows,  
overíte takto:  

```powershell
Get-Service postgresql*
```

## 4. Nástroje na prácu s databázou

### psql: príkazový riadok

`psql` je oficiálny textový klient. Okrem SQL pozná vlastné  
**metapríkazy**, ktoré začínajú spätnou lomkou a nekončia bodkočiarkou.  

| Príkaz | Čo robí |  
|---|---|  
| `\l` | zoznam databáz |  
| `\c skolenie` | prepnutie do databázy `skolenie` |  
| `\dt` | zoznam tabuliek |  
| `\d kurzy` | štruktúra tabuľky `kurzy` |  
| `\i subor.sql` | spustí príkazy zo súboru |  
| `\x` | prepína zvislé zobrazenie širokých riadkov |  
| `\?` | nápoveda k metapríkazom |  
| `\q` | koniec |  

> **Tip:** SQL príkaz sa v `psql` vykoná až po bodkočiarke. Ak výzva  
> zmení tvar na `skolenie-#`, `psql` čaká na pokračovanie. Dopíšte `;`  
> a stlačte Enter.  

### Doplnok PostgreSQL pre VS Code od Microsoftu

Microsoft vydáva bezplatný doplnok s názvom **PostgreSQL** (identifikátor  
`ms-ossdata.vscode-pgsql`, k októbru 2026 vo verzii 1.33). Z VS Code  
z neho robí plnohodnotné prostredie pre databázu: prehliadač objektov,  
editor dotazov s dopĺňaním, tabuľku výsledkov s exportom aj diagram  
schémy.  

Inštalácia:  

1. Otvorte panel Extensions (`Ctrl+Shift+X`).  
2. Vyhľadajte „PostgreSQL“ a zvoľte doplnok od vydavateľa **Microsoft**.  
3. Kliknite na **Install**. V ľavej lište pribudne ikona slona.  

Alebo z terminálu:  

```powershell
code --install-extension ms-ossdata.vscode-pgsql
```

> **Pozor:** V Marketplace je viac doplnkov s názvom PostgreSQL.  
> Skontrolujte vydavateľa Microsoft a identifikátor  
> `ms-ossdata.vscode-pgsql`.  

#### Pripojenie k serveru

1. Kliknite na ikonu slona alebo stlačte `Ctrl+Alt+D`.  
2. V sekcii **Connections** kliknite na **+** (**Add New Connection**).  
3. Na karte **Parameters** vyplňte polia podľa tabuľky.  
4. Kliknite na **Test Connection** a potom na **Save & Connect**.  

| Pole | Hodnota |  
|---|---|  
| Server name | `localhost` |  
| Authentication Type | Password |  
| User name | `postgres` |  
| Password | heslo z inštalácie |  
| Save Password | zaškrtnuté (uloží sa do Credential Manager) |  
| Database name | `skolenie` |  
| Connection Name | `Lokálny Postgres` |  

Port sa nastavuje v časti **Advanced** a predvolene je 5432. Namiesto  
polí môžete na karte **Connection String** vložiť celý reťazec  
pripojenia, napríklad `postgresql://postgres@localhost:5432/skolenie`.  

Ak Postgres nemáte nainštalovaný, tlačidlo **Create New Server** v tej  
istej lište ponúka aj vytvorenie servera v Dockeri na pár kliknutí.  

#### Práca s dotazmi

| Akcia | Ako |  
|---|---|  
| nový dotaz | pravým na databázu → **New Query** |  
| spustiť celý súbor alebo výber | `Ctrl+Shift+E` alebo `Shift+Enter` |  
| spustiť príkaz pod kurzorom | `Ctrl+Shift+Enter` |  
| náhľad dát tabuľky | pravým na tabuľku → **Select Top 1000** |  
| úprava riadkov v mriežke | pravým na tabuľku → **Edit Data** |  
| diagram tabuliek a väzieb | pravým na databázu → **Visualize Schema** |  
| `psql` v termináli VS Code | pravým na databázu → **Connect with PSQL** |  

Skratka `Ctrl+Shift+E` inak vo VS Code otvára panel Explorer. Doplnok ju  
preberá len vtedy, keď máte aktívny editor so súborom `.sql`.  

#### Copilot a databáza

Doplnok je prepojený s **GitHub Copilot**. V chate môžete osloviť  
účastníka `@pgsql` a pýtať sa na databázu prirodzeným jazykom, napríklad  
„@pgsql ktoré kurzy majú najviac voľných miest?“. V režime Agent  
dostáva Copilot nástroje, ktorými vie dotazy aj sám spúšťať.  

Preto má každé pripojenie v **Advanced** sekciu **Copilot** s voľbou  
**Copilot access mode**: **Read Only** alebo **Read/Write**. Pre  
dôležité databázy nastavte **Read Only**.  

### pgAdmin 4: oficiálne grafické rozhranie

**pgAdmin 4** je oficiálny bezplatný nástroj komunity PostgreSQL na  
správu databáz (k októbru 2026 verzia 9.19). Hodí sa najmä na  
administráciu: používatelia a práva, zálohy, obnova, monitorovanie  
servera. Inštalátor EDB ho pridá automaticky, spustíte ho z ponuky Štart.  

Pripojenie k serveru:  

1. V paneli **Object Explorer** kliknite pravým na **Servers** →  
   **Register** → **Server…**.  
2. Na karte **General** zadajte názov, napríklad `Lokálny Postgres`.  
3. Na karte **Connection** vyplňte **Host name/address** `localhost`,  
   **Port** `5432`, **Username** `postgres` a heslo.  
4. Kliknite na **Save**.  

Dotazy píšete v **Query Tool** (pravým na databázu → **Query Tool**)  
a spúšťate klávesom `F5`. Zálohu databázy vytvoríte pravým kliknutím  
na databázu → **Backup…**.  

### Ktorý nástroj kedy

| Nástroj | Silná stránka | Kedy ho použiť |  
|---|---|---|  
| VS Code doplnok | SQL vedľa kódu, Copilot | každodenná práca, vývoj |  
| pgAdmin 4 | správa servera, zálohy, práva | administrácia |  
| `psql` | skripty, automatizácia | terminál, CI, AI agenti |  

## 5. Príklady SQL

Všetky príklady sú pripravené v priečinku `postgres-priklady`:  

```
postgres-priklady/
├── 01-tabulky/tabulky.sql
├── 02-data/data.sql
├── 03-dotazy/dotazy.sql
├── 04-zmeny/zmeny.sql
└── 05-projekt/behy.sql
```

Spúšťajte ich v databáze `skolenie` v uvedenom poradí. Vo VS Code  
otvorte súbor, zvoľte pripojenie a stlačte `Ctrl+Shift+E`. Z terminálu  
spustíte súbor prepínačom `-f`, v otvorenom `psql` metapríkazom `\i`:  

```powershell
cd postgres-priklady
psql -U postgres -h localhost -d skolenie -f 01-tabulky/tabulky.sql
psql -U postgres -h localhost -d skolenie -f 02-data/data.sql
```

### 5.1 Vytvorenie tabuliek

Evidencia školení má tri tabuľky: **kurzy**, **účastníci** a **prihlášky**,  
ktoré ich spájajú. Jeden účastník môže mať viac prihlášok a jeden kurz  
viac účastníkov.  

```mermaid
flowchart LR
    U[ucastnici] -->|1 : N| P[prihlasky]
    K[kurzy] -->|1 : N| P
```

Súbor `postgres-priklady/01-tabulky/tabulky.sql`:  

```sql
-- Tabuľky pre evidenciu školení
CREATE TABLE kurzy (
    id        integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    nazov     text NOT NULL,
    mesto     text NOT NULL,
    zaciatok  date NOT NULL,
    cena      numeric(8, 2) NOT NULL CHECK (cena >= 0),
    kapacita  integer NOT NULL DEFAULT 12
);

CREATE TABLE ucastnici (
    id        integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    meno      text NOT NULL,
    priezvisko text NOT NULL,
    email     text NOT NULL UNIQUE,
    firma     text,
    vytvoreny timestamptz NOT NULL DEFAULT now()
);

CREATE TABLE prihlasky (
    id           integer GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    ucastnik_id  integer NOT NULL REFERENCES ucastnici (id),
    kurz_id      integer NOT NULL REFERENCES kurzy (id),
    stav         text NOT NULL DEFAULT 'nova'
                 CHECK (stav IN ('nova', 'zaplatena', 'zrusena')),
    hodnotenie   integer CHECK (hodnotenie BETWEEN 1 AND 5),
    UNIQUE (ucastnik_id, kurz_id)
);
```

Čo jednotlivé časti znamenajú:  

| Zápis | Význam |  
|---|---|  
| `GENERATED ALWAYS AS IDENTITY` | databáza sama prideľuje čísla 1, 2, 3… |  
| `PRIMARY KEY` | jedinečný identifikátor riadku |  
| `NOT NULL` | hodnota je povinná |  
| `UNIQUE` | hodnota sa nesmie opakovať |  
| `DEFAULT` | hodnota, ak ju pri zápise nezadáte |  
| `CHECK (...)` | podmienka, ktorú musí hodnota splniť |  
| `REFERENCES ucastnici (id)` | cudzí kľúč, odkaz musí existovať |  

Pravidlá `CHECK`, `UNIQUE` a `REFERENCES` chránia dáta aj pred chybami  
AI agenta. Ak sa agent pokúsi zapísať neexistujúci stav alebo prihlášku  
na neexistujúci kurz, databáza zápis odmietne.  

### 5.2 Testovacie dáta

Príkaz `INSERT` pridáva riadky. Stĺpce `id` a `vytvoreny` nevypĺňame,  
doplní ich databáza. Text sa v SQL píše do **jednoduchých** úvodzoviek,  
chýbajúca hodnota je `NULL`.  

Súbor `postgres-priklady/02-data/data.sql`:  

```sql
-- Testovacie dáta
INSERT INTO kurzy (nazov, mesto, zaciatok, cena, kapacita) VALUES
    ('Úvod do AI agentov',      'Bratislava', '2026-11-10', 290.00, 12),
    ('Workflowy v praxi',       'Košice',     '2026-11-17', 350.00, 10),
    ('Prompt engineering',      'Bratislava', '2026-11-24', 240.00, 15),
    ('PostgreSQL pre analytikov', 'Žilina',   '2026-12-01', 320.00, 8),
    ('MCP a nástroje',          'online',     '2026-12-08', 190.00, 20);

INSERT INTO ucastnici (meno, priezvisko, email, firma) VALUES
    ('Mária',  'Kováčová', 'maria.kovacova@example.com', 'Alfa s.r.o.'),
    ('Peter',  'Horváth',  'peter.horvath@example.com',  'Beta a.s.'),
    ('Jana',   'Nováková', 'jana.novakova@example.com',  'Alfa s.r.o.'),
    ('Tomáš',  'Varga',    'tomas.varga@example.com',    NULL),
    ('Zuzana', 'Tóthová',  'zuzana.tothova@example.com', 'Gama s.r.o.'),
    ('Martin', 'Baláž',    'martin.balaz@example.com',   'Beta a.s.'),
    ('Eva',    'Szabová',  'eva.szabova@example.com',    NULL),
    ('Lukáš',  'Molnár',   'lukas.molnar@example.com',   'Gama s.r.o.');

INSERT INTO prihlasky (ucastnik_id, kurz_id, stav, hodnotenie) VALUES
    (1, 1, 'zaplatena', 5),
    (1, 2, 'zaplatena', 4),
    (2, 1, 'zaplatena', 4),
    (2, 4, 'nova',      NULL),
    (3, 1, 'zrusena',   NULL),
    (3, 3, 'zaplatena', 5),
    (4, 2, 'nova',      NULL),
    (5, 3, 'zaplatena', 3),
    (5, 5, 'zaplatena', 5),
    (6, 4, 'zaplatena', NULL),
    (7, 5, 'nova',      NULL),
    (8, 1, 'zaplatena', 4);
```

Mená a e-maily sú vymyslené. Doména `example.com` je vyhradená na  
príklady, takže na ňu nikdy neodíde skutočná pošta.  

> **Tip:** Čísla `ucastnik_id` a `kurz_id` v prihláškach platia len pre  
> čerstvo vytvorené tabuľky. Druhé spustenie súboru zlyhá na duplicitnom  
> e-maile. Na čistý začiatok tabuľky zmažte príkazom  
> `DROP TABLE prihlasky, ucastnici, kurzy;` a spustite oba súbory znova.  

### Vyskúšajte

- Pridajte vlastný kurz a sami sa naň prihláste.  
- Skúste vložiť prihlášku so stavom `'hotova'`. Akú chybu vráti databáza?  
- Skúste pridať druhého účastníka s e-mailom  
  `maria.kovacova@example.com`.  
- Vo VS Code otvorte **Visualize Schema** a pozrite si väzby tabuliek.  

### 5.3 Čítanie dát: SELECT

`SELECT` je najčastejší príkaz. Určuje, **ktoré stĺpce** chcete, **z  
ktorej tabuľky** (`FROM`), **ktoré riadky** (`WHERE`) a **v akom  
poradí** (`ORDER BY`). Príkazy sú v súbore  
`postgres-priklady/03-dotazy/dotazy.sql`; nižšie ich preberieme  
postupne.  

Súbor `postgres-priklady/03-dotazy/dotazy.sql` (dotazy 1 až 4):  

```sql
-- 1. Všetky stĺpce a riadky
SELECT * FROM kurzy;

-- 2. Vybrané stĺpce, filter a zoradenie
SELECT nazov, mesto, cena
FROM kurzy
WHERE cena < 300
ORDER BY cena DESC;

-- 3. Hľadanie podľa časti textu (bez ohľadu na veľkosť písmen)
SELECT meno, priezvisko, email
FROM ucastnici
WHERE email ILIKE '%nov%';

-- 4. Chýbajúca hodnota sa testuje cez IS NULL
SELECT meno, priezvisko
FROM ucastnici
WHERE firma IS NULL;
```

Výsledok dotazu 2:  

```
       nazov        |   mesto    |  cena
--------------------+------------+--------
 Úvod do AI agentov | Bratislava | 290.00
 Prompt engineering | Bratislava | 240.00
 MCP a nástroje     | online     | 190.00
```

Operátory vo `WHERE`:  

| Operátor | Význam | Príklad |  
|---|---|---|  
| `=`, `<>` | rovná sa, nerovná sa | `mesto = 'Košice'` |  
| `<`, `>`, `<=`, `>=` | porovnanie | `cena >= 200` |  
| `AND`, `OR`, `NOT` | spájanie podmienok | `cena < 300 AND mesto = 'online'` |  
| `IN (...)` | jedna z hodnôt | `stav IN ('nova', 'zaplatena')` |  
| `BETWEEN` | v rozsahu vrátane hraníc | `cena BETWEEN 200 AND 300` |  
| `LIKE`, `ILIKE` | vzor textu, `%` = čokoľvek | `nazov ILIKE '%ai%'` |  
| `IS NULL` | hodnota chýba | `firma IS NULL` |  

> **Pozor:** `firma = NULL` nevráti nikdy nič. `NULL` znamená „neznáme“  
> a neznáme sa nerovná ničomu, ani inému `NULL`. Vždy píšte `IS NULL`  
> alebo `IS NOT NULL`.  

### Vyskúšajte

- Vypíšte kurzy v Bratislave zoradené podľa dátumu začiatku.  
- Nájdite účastníkov, ktorých priezvisko začína na „K“.  
- Pridajte k dotazu 2 `LIMIT 1`. Čo vráti?  

### 5.4 Zoskupovanie a spájanie tabuliek

**Agregačné funkcie** spočítajú hodnoty z viacerých riadkov: `count`,  
`sum`, `avg`, `min`, `max`. S `GROUP BY` ich počítate pre každú skupinu  
zvlášť. `JOIN` spája riadky z viacerých tabuliek podľa kľúča.  

Súbor `postgres-priklady/03-dotazy/dotazy.sql` (dotazy 5 až 8):  

```sql
-- 5. Počet účastníkov podľa firmy
SELECT coalesce(firma, '(bez firmy)') AS firma, count(*) AS pocet
FROM ucastnici
GROUP BY firma
ORDER BY pocet DESC, firma;

-- 6. Spojenie tabuliek: kto je na ktorom kurze
SELECT u.meno, u.priezvisko, k.nazov, p.stav
FROM prihlasky p
JOIN ucastnici u ON u.id = p.ucastnik_id
JOIN kurzy k     ON k.id = p.kurz_id
ORDER BY k.nazov, u.priezvisko;

-- 7. Obsadenosť a tržby kurzov (zrušené sa nerátajú)
SELECT k.nazov,
       count(p.id) AS prihlasenych,
       k.kapacita - count(p.id) AS volnych,
       coalesce(sum(k.cena) FILTER (WHERE p.stav = 'zaplatena'), 0)
           AS trzby
FROM kurzy k
LEFT JOIN prihlasky p ON p.kurz_id = k.id AND p.stav <> 'zrusena'
GROUP BY k.id
ORDER BY trzby DESC;

-- 8. Kurzy s priemerným hodnotením aspoň 4
SELECT k.nazov, round(avg(p.hodnotenie), 1) AS priemer
FROM kurzy k
JOIN prihlasky p ON p.kurz_id = k.id
GROUP BY k.nazov
HAVING avg(p.hodnotenie) >= 4
ORDER BY priemer DESC;
```

Výsledok dotazu 7:  

```
           nazov           | prihlasenych | volnych | trzby
---------------------------+--------------+---------+--------
 Úvod do AI agentov        |            3 |       9 | 870.00
 Prompt engineering        |            2 |      13 | 480.00
 Workflowy v praxi         |            2 |       8 | 350.00
 PostgreSQL pre analytikov |            2 |       6 | 320.00
 MCP a nástroje            |            2 |      18 | 190.00
```

| Zápis | Význam |  
|---|---|  
| `AS pocet` | alias, nový názov stĺpca vo výsledku |  
| `prihlasky p` | skratka tabuľky, ďalej stačí písať `p.stav` |  
| `JOIN ... ON` | len riadky, ktoré majú pár v oboch tabuľkách |  
| `LEFT JOIN` | všetky riadky ľavej tabuľky, aj bez páru |  
| `coalesce(a, b)` | prvá hodnota, ktorá nie je `NULL` |  
| `FILTER (WHERE ...)` | agregácia len z vybraných riadkov |  
| `HAVING` | filter na skupiny, až po `GROUP BY` |  

Rozdiel `WHERE` a `HAVING`: `WHERE` vyraďuje jednotlivé riadky pred  
zoskupením, `HAVING` celé skupiny po ňom. Funkcia `avg` ignoruje riadky  
s `NULL`, takže priemer sa počíta len z vyplnených hodnotení.  

### Vyskúšajte

- Upravte dotaz 6 tak, aby ukázal len zaplatené prihlášky.  
- Pridajte kurz bez prihlášok a v dotaze 7 zmeňte `LEFT JOIN` na  
  `JOIN`. Ktorý riadok zmizne a prečo?  
- Zistite, koľko eur zaplatila každá firma spolu.  

### 5.5 Zmeny dát a transakcie

`UPDATE` mení existujúce riadky, `DELETE` ich maže. Oba príkazy bez  
`WHERE` zasiahnu **celú tabuľku**. **Transakcia** spojí viac príkazov  
do jedného celku: buď sa uložia všetky (`COMMIT`), alebo žiadny  
(`ROLLBACK`).  

Súbor `postgres-priklady/04-zmeny/zmeny.sql`:  

```sql
-- 1. Nový účastník; RETURNING vráti pridelené id
INSERT INTO ucastnici (meno, priezvisko, email)
VALUES ('Ivana', 'Kráľová', 'ivana.kralova@example.com')
RETURNING id, vytvoreny;

-- 2. Zmena ceny jedného kurzu
UPDATE kurzy
SET cena = 260.00
WHERE nazov = 'Prompt engineering';

-- 3. Transakcia: presun prihlášky na iný kurz
BEGIN;
UPDATE prihlasky SET stav = 'zrusena'
WHERE ucastnik_id = 4 AND kurz_id = 2;
INSERT INTO prihlasky (ucastnik_id, kurz_id) VALUES (4, 5);
COMMIT;

-- 4. Vyskúšanie mazania bez rizika: ROLLBACK všetko vráti
BEGIN;
DELETE FROM prihlasky WHERE stav = 'nova';
SELECT count(*) AS zostalo FROM prihlasky;
ROLLBACK;

-- 5. Index na rýchlejšie hľadanie prihlášok podľa kurzu
CREATE INDEX prihlasky_kurz_idx ON prihlasky (kurz_id);
```

**Index** je pomocná štruktúra, podobne ako register v knihe. Pri  
tisícoch riadkov zrýchli hľadanie podľa stĺpca z sekúnd na milisekundy.  
Primárne kľúče a stĺpce s `UNIQUE` majú index automaticky.  

> **Tip:** Pred `UPDATE` alebo `DELETE` si najprv spustite `SELECT`  
> s rovnakým `WHERE`. Uvidíte presne tie riadky, ktoré sa zmenia.  

### Vyskúšajte

- Zvýšte ceny všetkých kurzov v Bratislave o 10 %  
  (`SET cena = cena * 1.10`).  
- V transakcii zmažte všetky prihlášky, overte počet a vráťte zmenu  
  cez `ROLLBACK`.  
- Skúste zmazať kurz, na ktorý existujú prihlášky. Prečo to databáza  
  nedovolí?  

## 6. Projekt: denník behov AI workflowov

Keď workflow beží automaticky, napríklad každú hodinu triedi e-maily,  
potrebujete vedieť, či uspel, ako dlho trval a koľko tokenov minul.  
Tieto údaje sa dajú ukladať do jednej tabuľky a vyhodnocovať cez SQL.  

### Krok 1: tabuľka a dáta

Tabuľka `behy` má pevné stĺpce na údaje, ktoré chcete sčítavať,  
a stĺpec `detaily` typu `jsonb` na všetko ostatné. Každý workflow tam  
môže zapísať iné údaje bez zmeny štruktúry tabuľky.  

Súbor `postgres-priklady/05-projekt/behy.sql` (časť 1):  

```sql
-- Záznam behov AI workflowov
CREATE TABLE behy (
    id          bigint GENERATED ALWAYS AS IDENTITY PRIMARY KEY,
    workflow    text NOT NULL,
    stav        text NOT NULL CHECK (stav IN ('ok', 'chyba')),
    zaciatok    timestamptz NOT NULL DEFAULT now(),
    trvanie_ms  integer NOT NULL,
    tokeny      integer NOT NULL DEFAULT 0,
    detaily     jsonb NOT NULL DEFAULT '{}'
);

INSERT INTO behy (workflow, stav, zaciatok, trvanie_ms, tokeny, detaily)
VALUES
    ('triedenie-emailov', 'ok',    '2026-10-30 08:00+01', 1840, 2300,
     '{"model": "claude-haiku-5-5", "emailov": 14}'),
    ('triedenie-emailov', 'ok',    '2026-10-30 12:00+01', 2110, 2650,
     '{"model": "claude-haiku-5-5", "emailov": 17}'),
    ('triedenie-emailov', 'chyba', '2026-10-30 16:00+01', 30000, 0,
     '{"model": "claude-haiku-5-5", "chyba": "timeout"}'),
    ('suhrn-faktur',      'ok',    '2026-10-30 09:15+01', 5200, 8100,
     '{"model": "claude-sonnet-5-5", "faktur": 6}'),
    ('suhrn-faktur',      'ok',    '2026-10-31 09:15+01', 4800, 7600,
     '{"model": "claude-sonnet-5-5", "faktur": 5}'),
    ('denny-report',      'ok',    '2026-10-31 07:00+01', 9400, 15200,
     '{"model": "claude-opus-5-5", "zdrojov": 4}'),
    ('denny-report',      'chyba', '2026-10-30 07:00+01', 1200, 400,
     '{"model": "claude-opus-5-5", "chyba": "neplatný API kľúč"}');
```

### Krok 2: vyhodnotenie

Súbor `postgres-priklady/05-projekt/behy.sql` (časť 2):  

```sql
-- Úspešnosť a spotreba tokenov podľa workflowu
SELECT workflow,
       count(*) AS behov,
       round(100.0 * count(*) FILTER (WHERE stav = 'ok') / count(*))
           AS uspesnost_pct,
       sum(tokeny) AS tokeny_spolu,
       round(avg(trvanie_ms)) AS priemer_ms
FROM behy
GROUP BY workflow
ORDER BY tokeny_spolu DESC;

-- Posledné chyby s textom z poľa JSON
SELECT zaciatok, workflow, detaily ->> 'chyba' AS chyba
FROM behy
WHERE stav = 'chyba'
ORDER BY zaciatok DESC;
```

Výsledok prvého dotazu:  

```
     workflow      | behov | uspesnost_pct | tokeny_spolu | priemer_ms
-------------------+-------+---------------+--------------+------------
 suhrn-faktur      |     2 |           100 |        15700 |       5000
 denny-report      |     2 |            50 |        15600 |       5300
 triedenie-emailov |     3 |            67 |         4950 |      11317
```

Operátor `->>` vytiahne z JSON hodnotu ako text. Podobne  
`detaily ->> 'model'` vráti názov modelu, s ktorým sa dá ďalej  
zoskupovať.  

### Krok 3: rola len na čítanie pre agenta

AI agent, ktorý má z databázy len čítať, nemá dostať heslo správcu.  
Vytvorte mu vlastnú **rolu** (používateľa databázy) s právom `SELECT`.  

Súbor `postgres-priklady/05-projekt/behy.sql` (časť 3):  

```sql
-- Rola len na čítanie pre AI agenta
CREATE ROLE agent_citatel LOGIN PASSWORD 'zmente-heslo';
GRANT CONNECT ON DATABASE skolenie TO agent_citatel;
GRANT USAGE ON SCHEMA public TO agent_citatel;
GRANT SELECT ON ALL TABLES IN SCHEMA public TO agent_citatel;
ALTER DEFAULT PRIVILEGES IN SCHEMA public
    GRANT SELECT ON TABLES TO agent_citatel;
```

Príkaz `GRANT SELECT ON ALL TABLES` platí len pre tabuľky, ktoré už  
existujú. `ALTER DEFAULT PRIVILEGES` zabezpečí právo na čítanie aj  
pre tabuľky, ktoré správca vytvorí neskôr.  

Overenie: pripojte sa ako nová rola a skúste zapisovať.  

```powershell
psql "postgresql://agent_citatel:zmente-heslo@localhost:5432/skolenie"
```

```sql
SELECT count(*) FROM behy;
INSERT INTO behy (workflow, stav, trvanie_ms) VALUES ('test', 'ok', 1);
```

`SELECT` vráti 7, `INSERT` skončí chybou  
`permission denied for table behy`. Presne to chcete.  

> **Pozor:** Heslo `zmente-heslo` je len na školenie. Mimo testovacieho  
> počítača ho nahraďte silným heslom.  

### Krok 4: zápis z workflowu

Workflow (napríklad v n8n, Make alebo vlastnom skripte) na konci každého  
behu vykoná jeden `INSERT` do tabuľky `behy`. Na zápis mu vytvorte  
ďalšiu rolu s právom `INSERT` len na túto tabuľku.  

Súbor `postgres-priklady/05-projekt/behy.sql` (časť 4):  

```sql
-- Rola pre workflow: smie len pridávať záznamy do tabuľky behy
CREATE ROLE workflow_zapis LOGIN PASSWORD 'ine-silne-heslo';
GRANT CONNECT ON DATABASE skolenie TO workflow_zapis;
GRANT USAGE ON SCHEMA public TO workflow_zapis;
GRANT INSERT ON behy TO workflow_zapis;
```

Tak má každá časť systému len tie práva, ktoré naozaj potrebuje.  

## 7. AI agenti a PostgreSQL

### Claude Code a psql

Claude Code vie spúšťať príkazy v termináli, teda aj `psql`. Stačí mu  
dať reťazec pripojenia v premennej prostredia a pravidlá v `CLAUDE.md`.  
`psql` číta heslo z premennej `PGPASSWORD`, alebo celý reťazec  
pripojenia z argumentu.  

```powershell
$env:PGHOST = "localhost"
$env:PGDATABASE = "skolenie"
$env:PGUSER = "agent_citatel"
$env:PGPASSWORD = "zmente-heslo"
claude
```

Zadanie potom môže znieť: „Pomocou psql zisti, ktorý workflow mal  
včera najviac chýb, a navrhni príčinu.“ Agent si sám pozrie štruktúru  
cez `\d behy`, napíše dotaz a vysvetlí výsledok.  

Odporúčaný úryvok do `CLAUDE.md` projektu:  

```markdown
## Databáza
- Databáza skolenie, pripájaj sa len cez psql a premenné PG*.
- Nikdy nespúšťaj DROP, TRUNCATE ani DELETE bez WHERE.
- Pred každou zmenou dát ukáž SELECT s rovnakou podmienkou.
```

### MCP server pre PostgreSQL

**MCP** (Model Context Protocol) je štandard, cez ktorý AI aplikácia  
dostáva nástroje. MCP server **Postgres MCP Pro** (balík `postgres-mcp`)  
dá agentovi nástroje na prehliadanie schémy, spúšťanie SQL, analýzu  
plánu dotazu aj návrh indexov. Režim `--access-mode=restricted` povolí  
len čítanie a obmedzí čas behu dotazu.  

Pridanie do Claude Code (potrebujete nástroj `uv`):  

```bash
claude mcp add postgres \
  -e DATABASE_URI=postgresql://agent_citatel:zmente-heslo@localhost:5432/skolenie \
  -- uvx postgres-mcp --access-mode=restricted
```

| Nástroj servera | Čo robí |  
|---|---|  
| `list_schemas`, `list_objects` | zoznam schém, tabuliek a pohľadov |  
| `get_object_details` | stĺpce, kľúče a indexy tabuľky |  
| `execute_sql` | spustí SQL (v `restricted` len čítanie) |  
| `explain_query` | plán vykonania dotazu |  
| `analyze_db_health` | kontrola stavu databázy |  

Viac o MCP nájdete v kapitole `mcp.md`.  

### Copilot vo VS Code

Ak používate GitHub Copilot, doplnok PostgreSQL mu pridá nástroje  
priamo, bez samostatného MCP servera (pozri časť 4). Aj tu platí:  
pripojenie s **Copilot access mode** nastaveným na **Read Only**.  

> **Jednoducho povedané:** Agentovi dajte do archívu kľúč len od  
> čitárne, nie od skladu. Ak chcete, aby niečo zapísal, nech vám  
> pripraví príkaz a vy ho spustíte sami.  

## 8. Bezpečnosť

- **Agent nedostáva heslo správcu.** Používajte rolu len na čítanie  
  a režimy `restricted` či **Read Only**.  
- **Heslá nepatria do kódu ani do `CLAUDE.md`.** Ukladajte ich  
  do premenných prostredia, súboru `.env` mimo Gitu alebo do správcu  
  hesiel (VS Code používa Credential Manager Windows).  
- **Výsledok dotazu je dôveryhodný len natoľko ako dáta.** Ak tabuľka  
  obsahuje texty od zákazníkov, môžu v nich byť skryté pokyny pre AI  
  (prompt injection). Agent, ktorý ich číta, by nemal mať zároveň  
  právo zapisovať alebo posielať e-maily.  
- **Nezverejňujte port 5432 do internetu.** Server nechajte počúvať len  
  na `localhost`, pri cloudovej databáze povoľte len konkrétne adresy.  
- **Pred veľkou zmenou zálohujte.** V pgAdmine **Backup…**, v termináli  
  `pg_dump -U postgres -d skolenie -f zaloha.sql`.  
- **Skladajte SQL bezpečne.** Vo vlastnom kóde nikdy nevkladajte text  
  od používateľa priamo do reťazca SQL. Používajte parametre dotazu,  
  inak hrozí **SQL injection**.  
- **Osobné údaje chráňte.** Na školenie používajte vymyslené dáta, nie  
  kópiu ostrej databázy so skutočnými menami.  

## 9. Časté chyby

| Problém | Príčina | Riešenie |  
|---|---|---|  
| `psql` sa nenašiel | `bin` nie je v `Path` | pridať `...\PostgreSQL\18\bin` |  
| `Connection refused` | server nebeží | `Get-Service postgresql*` |  
| `password authentication failed` | zlé heslo alebo meno | overiť používateľa |  
| port 5432 obsadený | dva servery naraz | iný port, napr. 5433 |  
| `syntax error at or near` | preklep, chýba čiarka | pozrieť miesto chyby |  
| `psql` nič nerobí | chýba `;` na konci | dopísať `;` |  
| `column "Bratislava" does not exist` | text v `"..."` | použiť `'...'` |  
| `duplicate key value` | porušené `UNIQUE` | iná hodnota alebo `UPDATE` |  
| `violates foreign key constraint` | odkaz neexistuje | najprv vytvoriť rodiča |  
| zlá diakritika v `psql` | kódová stránka konzoly | `chcp 65001` |  
| `permission denied` | rola nemá právo | `GRANT` správcom |  
| `= NULL` nič nevráti | `NULL` sa neporovnáva | `IS NULL` |  

## 10. Úlohy na precvičenie

1. **Inštalácia.** Nainštalujte PostgreSQL 18 cez winget, overte  
   `psql --version` a vytvorte databázu `skolenie`.  
2. **Tri nástroje.** Pripojte sa k nej z `psql`, z pgAdminu aj  
   z doplnku vo VS Code. Spustite `SELECT version();` v každom.  
3. **Dáta.** Spustite súbory 01 a 02. V doplnku VS Code otvorte  
   **Select Top 1000** na tabuľke `prihlasky`.  
4. **Dotazy.** Napíšte dotaz, ktorý vypíše účastníkov, ktorí nemajú  
   žiadnu zaplatenú prihlášku. (Nápoveda: `LEFT JOIN` a `IS NULL`.)  
5. **Nová tabuľka.** Pridajte tabuľku `lektori` a stĺpec `lektor_id`  
   do tabuľky `kurzy` s cudzím kľúčom (`ALTER TABLE ... ADD COLUMN`).  
6. **Denník behov.** Rozšírte tabuľku `behy` o stĺpec `naklady_eur`  
   a napíšte dotaz s nákladmi podľa modelu z poľa `detaily`.  
7. **Agent.** Pripojte Claude Code k databáze cez rolu `agent_citatel`  
   alebo MCP server v režime `restricted`. Nechajte ho napísať týždenný  
   prehľad workflowov a overte, že nevie nič zmeniť.  

## 11. Zhrnutie

- **PostgreSQL** je bezplatná relačná databáza, aktuálne vo verzii  
  **18**. Dáta ukladá do **tabuliek** so stĺpcami a riadkami.  
- Na Windows ju nainštalujete cez **winget** alebo inštalátor **EDB**,  
  ktorý pridá aj **pgAdmin 4** a **psql**.  
- Doplnok **PostgreSQL** od Microsoftu (`ms-ossdata.vscode-pgsql`)  
  prináša do VS Code prehliadač objektov, editor dotazov  
  (`Ctrl+Shift+E`) a prepojenie s Copilotom.  
- **pgAdmin 4** je oficiálne grafické rozhranie, vhodné najmä na  
  správu servera, práva a zálohy.  
- Základ SQL: **`CREATE TABLE`**, **`INSERT`**, **`SELECT`** s `WHERE`,  
  `ORDER BY`, `GROUP BY` a `JOIN`, **`UPDATE`**, **`DELETE`**  
  a **transakcie**.  
- Pravidlá **`NOT NULL`**, **`UNIQUE`**, **`CHECK`** a **cudzie  
  kľúče** chránia dáta aj pred chybami AI agenta.  
- Stĺpec **`jsonb`** je praktický na záznamy behov workflowov  
  s rôznymi údajmi.  
- AI agent dostáva **rolu len na čítanie**, MCP server v režime  
  **`restricted`** alebo Copilot v režime **Read Only**.  

## 12. Slovníček

| Anglicky | Slovensky | Význam |  
|---|---|---|  
| database | databáza | organizované úložisko dát |  
| table | tabuľka | súbor riadkov s rovnakými stĺpcami |  
| row, column | riadok, stĺpec | záznam a jeho vlastnosť |  
| SQL | jazyk SQL | jazyk na prácu s relačnou databázou |  
| query | dotaz | príkaz, ktorý číta alebo mení dáta |  
| primary key | primárny kľúč | jedinečný identifikátor riadku |  
| foreign key | cudzí kľúč | odkaz na riadok inej tabuľky |  
| constraint | obmedzenie | pravidlo pre hodnoty, napr. `CHECK` |  
| schema | schéma | menný priestor tabuliek, napr. `public` |  
| index | index | štruktúra na rýchle hľadanie |  
| join | spojenie | prepojenie riadkov viacerých tabuliek |  
| transaction | transakcia | skupina zmien ako jeden celok |  
| role | rola | používateľ alebo skupina v databáze |  
| connection string | reťazec pripojenia | adresa, meno a databáza v jednom |  
| client | klient | program, ktorý sa pripája na server |  
| port | port | číslo „dverí“ servera, Postgres 5432 |  
| JSON, JSONB | JSON | textový formát dát; `jsonb` je binárny |  
| SQL injection | SQL injekcia | útok vložením SQL do vstupu |  
| backup, dump | záloha, výpis | kópia databázy do súboru |  
| winget | správca balíkov | inštalácia programov vo Windows |  

## Zdroje

- PostgreSQL – dokumentácia: postgresql.org/docs/18  
- PostgreSQL – podporované verzie: postgresql.org/support/versioning  
- PostgreSQL – inštalácia na Windows: postgresql.org/download/windows  
- pgAdmin 4 – stiahnutie a dokumentácia: pgadmin.org  
- Doplnok PostgreSQL pre VS Code: marketplace.visualstudio.com,  
  ms-ossdata.vscode-pgsql  
- Dokumentácia doplnku (Microsoft Learn):  
  learn.microsoft.com/azure/postgresql/development/vs-code-extension  
- Postgres MCP Pro: github.com/crystaldba/postgres-mcp  
- winget – balíky PostgreSQL: github.com/microsoft/winget-pkgs  
- Claude Code – dokumentácia: code.claude.com/docs  
