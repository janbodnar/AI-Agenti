# Terminály vo Windows – cmd a PowerShell

## Kde sa s terminálom stretneme

V kurze AI agentov a kustomizácie sa s terminálom stretneme pri:

- **inštalácii nástrojov** ako uv, Node.js, Git či Claude Code,
- **spúšťaní agentov** a MCP serverov z príkazového riadka,
- **hookoch**, ktoré sú krátke príkazy spúšťané pri udalostiach agenta,
- **čítaní výstupu**, keď agent spúšťa príkazy a my kontrolujeme, čo urobil.

Agent vo Windows väčšinou píše príkazy pre PowerShell. Oplatí sa preto
rozumieť, čo robí.

---

## Čo je terminál

**Terminál** je okno, v ktorom píšeme príkazy. **Shell** je program,
ktorý ich vykonáva. Vo Windows máme dva hlavné shelly:

- **cmd** (Príkazový riadok) – starý shell z čias MS-DOS. Jednoduchý,
  pracuje s textom.
- **PowerShell** – moderný shell. Pracuje s objektmi, má vlastný
  skriptovací jazyk.

PowerShell existuje v dvoch verziách:

| Verzia | Príkaz | Poznámka |
|---|---|---|
| Windows PowerShell 5.1 | `powershell` | Predinštalovaný vo Windows 10 a 11 |
| PowerShell 7 | `pwsh` | Novší, treba doinštalovať, beží aj na Linuxe a macOS |

Obidva shelly sa dajú otvoriť vo **Windows Terminal**. Je to moderné okno
s kartami, ktoré je súčasťou Windows 11.

## Ako terminál otvoriť

- `Win + R`, napíšeme `cmd` alebo `powershell` a stlačíme Enter.
- Pravý klik na tlačidlo Štart a zvolíme **Terminál**.
- Vo VS Code cez menu **Terminal > New Terminal** (alebo ``Ctrl + ` ``).
- V Prieskumníkovi napíšeme do adresného riadka `cmd` alebo
  `powershell`. Terminál sa otvorí priamo v danom priečinku.

Inštalácia PowerShell 7:

```powershell
winget install --id Microsoft.PowerShell -e
```

Overenie verzie:

```powershell
$PSVersionTable.PSVersion
```

---

## Klasický cmd

### Najčastejšie príkazy

| Príkaz | Čo robí |
|---|---|
| `cd priecinok` | Prejde do priečinka |
| `cd ..` | Prejde o úroveň vyššie |
| `dir` | Vypíše obsah priečinka |
| `mkdir projekt` | Vytvorí priečinok |
| `copy a.txt b.txt` | Skopíruje súbor |
| `move a.txt archiv\` | Presunie súbor |
| `del a.txt` | Zmaže súbor |
| `type a.txt` | Vypíše obsah súboru |
| `cls` | Vyčistí obrazovku |
| `where python` | Zistí, kde je program nainštalovaný |
| `help` alebo `prikaz /?` | Zobrazí nápovedu |

### Príklady

Vytvorenie priečinka a súboru:

```cmd
mkdir poznamky
cd poznamky
echo Ahoj svet > ahoj.txt
type ahoj.txt
```

Premenné prostredia sa zapisujú medzi percentá:

```cmd
echo %USERNAME%
echo %PATH%
set API_KEY=abc123
echo %API_KEY%
```

Spojenie príkazov. Druhý príkaz sa spustí, iba ak prvý uspel:

```cmd
mkdir test && cd test
```

Presmerovanie výstupu do súboru a hľadanie textu:

```cmd
dir > zoznam.txt
dir | findstr ".md"
```

### Absolútne a relatívne cesty

**Absolútna cesta** začína písmenom disku. Funguje vždy, nezáleží
na tom, kde v termináli práve sme:

```cmd
cd C:\Users\Jano\Documents\projekty
type C:\Users\Jano\Documents\poznamky.txt
```

**Relatívna cesta** vychádza z aktuálneho priečinka. Je kratšia,
ale jej význam sa mení podľa toho, kde sa nachádzame:

```cmd
cd projekty
type poznamky\ahoj.txt
```

Špeciálne značky v cestách:

| Značka | Význam | Príklad |
|---|---|---|
| `.` | Aktuálny priečinok | `.\skript.bat` |
| `..` | Nadradený priečinok | `cd ..\obrazky` |
| `\` na začiatku | Koreň aktuálneho disku | `cd \Windows` |
| `%USERPROFILE%` | Domovský priečinok používateľa | `cd %USERPROFILE%\Desktop` |

Kde sa práve nachádzame, zistíme príkazom `cd` bez parametrov:

```cmd
cd
```

Príklad: sme v `C:\Users\Jano\Documents\projekty`. Rovnaký súbor
otvoríme oboma spôsobmi:

```cmd
type C:\Users\Jano\Documents\poznamky.txt
type ..\poznamky.txt
```

Prechod na iný disk. Samotné `cd` disk nezmení, treba pridať `/d`:

```cmd
cd /d D:\zaloha
```

Alebo stačí napísať písmeno disku s dvojbodkou:

```cmd
D:
```

Cesty s medzerami dávame do úvodzoviek:

```cmd
cd "C:\Program Files"
```

### Dávkové súbory

Príkazy pre cmd sa dajú uložiť do súboru s príponou `.bat` alebo `.cmd`:

```cmd
@echo off
echo Spustam projekt...
cd C:\projekty\ahoj-agent
uv run main.py
pause
```

Súbor spustíme dvojklikom alebo jeho názvom v termináli.

---

## PowerShell

### Cmdlety

Príkazy v PowerShelli sa volajú **cmdlety**. Majú tvar `Sloveso-Podstatné`,
napríklad `Get-ChildItem` alebo `Remove-Item`. Pre známe príkazy existujú
aliasy, takže funguje aj `dir`, `ls`, `cd` či `cat`.

| Cmdlet | Alias | Čo robí |
|---|---|---|
| `Set-Location` | `cd` | Prejde do priečinka |
| `Get-ChildItem` | `dir`, `ls` | Vypíše obsah priečinka |
| `New-Item -ItemType Directory` | `mkdir` | Vytvorí priečinok |
| `Copy-Item` | `copy`, `cp` | Skopíruje súbor |
| `Move-Item` | `move`, `mv` | Presunie súbor |
| `Remove-Item` | `del`, `rm` | Zmaže súbor |
| `Get-Content` | `type`, `cat` | Vypíše obsah súboru |
| `Get-Command` | `gcm` | Zistí, kde je program |
| `Get-Help` | `help` | Zobrazí nápovedu |

### Príklady

Vytvorenie priečinka a súboru:

```powershell
mkdir poznamky
cd poznamky
"Ahoj svet" | Out-File ahoj.txt -Encoding utf8
Get-Content ahoj.txt
```

Premenné prostredia majú predponu `$env:`:

```powershell
$env:USERNAME
$env:Path -split ";"
$env:API_KEY = "abc123"
```

Vlastné premenné začínajú znakom `$`:

```powershell
$meno = "Jano"
"Ahoj, $meno!"
```

Kde je program nainštalovaný:

```powershell
(Get-Command python).Source
```

### Pipeline s objektmi

Hlavný rozdiel oproti cmd: pipeline (`|`) neposiela text, ale objekty.
Môžeme preto filtrovať a triediť podľa vlastností.

Päť najväčších súborov v priečinku:

```powershell
Get-ChildItem -File | Sort-Object Length -Descending | Select-Object -First 5 Name, Length
```

Všetky Markdown súbory vrátane podpriečinkov:

```powershell
Get-ChildItem -Recurse -Filter *.md
```

Hľadanie textu v súboroch (obdoba `grep`):

```powershell
Select-String -Path *.md -Pattern "uv"
```

Procesy, ktoré zaberajú najviac pamäte:

```powershell
Get-Process | Sort-Object WorkingSet -Descending | Select-Object -First 5
```

Stiahnutie stránky (obdoba `curl`):

```powershell
Invoke-RestMethod https://api.github.com/repos/astral-sh/uv
```

### Skripty

Skripty v PowerShelli majú príponu `.ps1`:

```powershell
# spusti.ps1
param([string]$Meno = "svet")

Write-Host "Ahoj, $Meno!"
Get-ChildItem -Filter *.md | ForEach-Object { $_.Name }
```

Spustenie:

```powershell
.\spusti.ps1 -Meno Jano
```

Ak Windows spustenie skriptu zablokuje, povolíme lokálne skripty
pre aktuálneho používateľa:

```powershell
Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
```

Toto nastavenie je trvalé. Platí aj po reštarte, kým ho nezmeníme.
Ak chceme skripty povoliť iba dočasne, použijeme `-Scope Process`:

```powershell
Set-ExecutionPolicy -Scope Process RemoteSigned
```

Nastavenie platí iba v aktuálnom okne PowerShellu. Po jeho zatvorení
zanikne.

---

## Porovnanie cmd, PowerShell a bash

| Úloha | cmd | PowerShell | bash (Linux, macOS) |
|---|---|---|---|
| Obsah priečinka | `dir` | `Get-ChildItem` | `ls` |
| Výpis súboru | `type a.txt` | `Get-Content a.txt` | `cat a.txt` |
| Zmazanie súboru | `del a.txt` | `Remove-Item a.txt` | `rm a.txt` |
| Hľadanie textu | `findstr "uv" *.md` | `Select-String "uv" *.md` | `grep "uv" *.md` |
| Premenná prostredia | `%PATH%` | `$env:Path` | `$PATH` |
| Nastavenie premennej | `set X=1` | `$env:X = "1"` | `export X=1` |
| Kde je program | `where python` | `Get-Command python` | `which python` |
| Skript | `.bat`, `.cmd` | `.ps1` | `.sh` |

## Časté nástrahy

- **`&&` vo Windows PowerShell 5.1 nefunguje.** Použijeme `;` alebo
  `if ($?) { ... }`. V PowerShell 7 už `&&` funguje.
- **Cesty s medzerami** dávame do úvodzoviek:
  `cd "C:\Program Files"`.
- **Program s cestou s medzerami** spustíme v PowerShelli cez `&`:
  `& "C:\Program Files\Git\bin\git.exe" --version`.
- **Kódovanie.** Vo Windows PowerShell 5.1 pridávame pri zápise
  `-Encoding utf8`, inak sa môže pokaziť diakritika.
- **Príkazy z návodov pre Linux** (`ls -la`, `rm -rf`, `export`) vo Windows
  nefungujú alebo fungujú inak. Pozor na to, keď kopírujete z internetu.

## Poznámka: terminál a AI agent

Keď agent vo Windows spúšťa príkazy, vidíme ich v jeho výstupe. Pomôže,
ak im rozumieme:

- `Get-ChildItem`, `Get-Content`, `Select-String` iba čítajú – sú bezpečné.
- `Remove-Item`, `Move-Item`, `Set-Content` menia súbory – tie si
  pred povolením prečítajte.
- Príkaz s `-Recurse -Force` pri mazaní maže celý priečinok bez opýtania.

## Cvičenie

1. Otvorte cmd aj PowerShell a v oboch zistite, v akom priečinku ste
   (`cd` v cmd, `Get-Location` v PowerShelli).
2. V cmd vytvorte priečinok `test` a v ňom súbor cez `echo`.
3. V PowerShelli vypíšte všetky `.md` súbory zoradené podľa veľkosti.
4. Nastavte premennú prostredia `API_KEY` v oboch shelloch a vypíšte ju.
5. Napíšte krátky skript `.ps1`, ktorý pozdraví zadané meno.

---

## Príklady: generatívna AI s Pythonom

Vyskúšame si model Gemini od Google. Kľúč si vytvoríme zadarmo
v Google AI Studio: <https://aistudio.google.com/apikey>.

Kľúč nepíšeme priamo do kódu. Uložíme ho do premennej prostredia
`GEMINI_API_KEY`, ktorú knižnica načíta sama.

### Nastavenie premennej graficky

1. Stlačíme `Win + R`, napíšeme `sysdm.cpl` a stlačíme Enter.
   (Alebo v ponuke Štart hľadáme „premenné prostredia“ a zvolíme
   **Upraviť premenné prostredia pre vaše konto**.)
2. Na karte **Spresnenie** klikneme na **Premenné prostredia**.
3. V hornej časti **Používateľské premenné** klikneme na **Nová**.
4. Názov: `GEMINI_API_KEY`, hodnota: náš kľúč. Potvrdíme **OK**.
5. Zatvoríme a znova otvoríme terminál (aj VS Code). Staré okná novú
   premennú nevidia.

### Nastavenie premennej v termináli

Iba pre aktuálne okno (po zatvorení zanikne):

```cmd
set GEMINI_API_KEY=vas-kluc
```

```powershell
$env:GEMINI_API_KEY = "vas-kluc"
```

Trvalo pre aktuálneho používateľa (platí v nových oknách):

```cmd
setx GEMINI_API_KEY "vas-kluc"
```

```powershell
[Environment]::SetEnvironmentVariable("GEMINI_API_KEY", "vas-kluc", "User")
```

Overenie:

```cmd
echo %GEMINI_API_KEY%
```

```powershell
$env:GEMINI_API_KEY
```

### Príklad 1: jednoduchá otázka

```python
# otazka.py
from google import genai

client = genai.Client()

response = client.models.generate_content(
    model="gemini-3.5-flash-lite",
    contents="Vysvetli v troch vetách, čo je PowerShell.",
)
print(response.text)
```

Spustenie cez uv, knižnica sa doinštaluje sama:

```powershell
uv run --with google-genai otazka.py
```

Bez uv knižnicu nainštalujeme cez pip a skript spustíme priamo:

```powershell
pip install google-genai
python otazka.py
```

Ak príkaz `pip` nie je dostupný, použijeme `python -m pip install google-genai`.

### Príklad 2: chat v termináli

Model si pamätá predchádzajúce správy. Prázdny riadok chat ukončí.

```python
# chat.py
from google import genai

client = genai.Client()
chat = client.chats.create(model="gemini-3.5-flash-lite")

while True:
    otazka = input("Ty: ")
    if not otazka:
        break
    odpoved = chat.send_message(otazka)
    print("AI:", odpoved.text)
```

```powershell
uv run --with google-genai chat.py
```

### Príklad 3: zhrnutie súboru

Skript načíta textový súbor a požiada model o zhrnutie.

```python
# zhrn.py
from pathlib import Path

from google import genai

subor = "uv.md"
text = Path(subor).read_text(encoding="utf-8")

client = genai.Client()
response = client.models.generate_content(
    model="gemini-3.5-flash-lite",
    contents=f"Zhrň tento text do piatich bodov po slovensky:\n\n{text}",
)
print(response.text)
```

```powershell
uv run --with google-genai zhrn.py
```

Všimnite si relatívnu cestu `uv.md`. Súbor sa hľadá v aktuálnom
priečinku.

Názvy modelov sa časom menia. Aktuálny zoznam nájdete
v dokumentácii: <https://ai.google.dev/gemini-api/docs/models>.

## Odkazy

- PowerShell: <https://learn.microsoft.com/powershell/>
- Príkazy cmd: <https://learn.microsoft.com/windows-server/administration/windows-commands/windows-commands>
- Windows Terminal: <https://learn.microsoft.com/windows/terminal/>
- Google Gen AI SDK: <https://ai.google.dev/gemini-api/docs/quickstart>
