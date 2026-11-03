# winget – správca balíčkov vo Windows

**winget** (Windows Package Manager) inštaluje programy jedným príkazom, bez sťahovania inštalátorov z webov a bez klikania cez sprievodcov. Je súčasťou Windows 10 a 11, overíte ho cez:

```powershell
winget --version
```

Ak príkaz nefunguje, nainštalujte alebo aktualizujte **App Installer** z Microsoft Store.

## Prečo winget

- **Rýchlosť:** jeden príkaz namiesto hľadania stránky, sťahovania a klikania.
- **Bezpečnosť:** balíčky pochádzajú z oficiálneho katalógu, nie z náhodných webov.
- **Aktualizácie:** všetky programy aktualizujete naraz.
- **Opakovateľnosť:** zoznam programov si uložíte a na novom počítači ho nainštalujete znova.

## Základné príkazy

| Príkaz | Čo robí |
|---|---|
| `winget search ripgrep` | Vyhľadá balíček |
| `winget show --id <ID>` | Zobrazí detaily balíčka (verzia, vydavateľ, licencia) |
| `winget install --id <ID> -e` | Nainštaluje balíček (`-e` = presná zhoda ID) |
| `winget list` | Zobrazí nainštalované programy |
| `winget upgrade` | Zobrazí programy s dostupnou aktualizáciou |
| `winget upgrade --id <ID> -e` | Aktualizuje jeden program |
| `winget upgrade --all` | Aktualizuje všetko naraz |
| `winget uninstall --id <ID> -e` | Odinštaluje program |

## Užitočné prepínače

| Prepínač | Význam |
|---|---|
| `-e`, `--exact` | Presná zhoda ID, aby sa nenainštaloval iný balíček s podobným názvom |
| `--silent` | Inštalácia bez okien a otázok |
| `--scope user` | Inštalácia len pre aktuálneho používateľa (často bez práv správcu) |
| `--accept-package-agreements` | Automaticky prijme licenčné podmienky |
| `--version <verzia>` | Nainštaluje konkrétnu verziu |

## Zaujímavé menšie programy na vyskúšanie

### Nástroje do príkazového riadka

```powershell
winget install --id=jqlang.jq -e                  # jq, spracovanie JSON v termináli
winget install --id=BurntSushi.ripgrep.MSVC -e    # ripgrep (rg), veľmi rýchle hľadanie v súboroch
winget install --id=sharkdp.fd -e                 # fd, rýchle hľadanie súborov podľa názvu
winget install --id=sharkdp.bat -e                # bat, výpis súborov so zvýraznením syntaxe
winget install --id=junegunn.fzf -e               # fzf, interaktívne vyhľadávanie v zoznamoch
winget install --id=Gyan.FFmpeg -e                # FFmpeg, konverzia videa a zvuku
winget install --id=yt-dlp.yt-dlp -e              # yt-dlp, sťahovanie videí z webu
winget install --id=ImageMagick.ImageMagick -e    # ImageMagick, úprava obrázkov z príkazového riadka
```

Tieto nástroje sa hodia aj pre AI agentov: agent ich vie spúšťať a rýchlo tak hľadať v kóde (`rg`), spracovať JSON (`jq`) alebo upraviť médiá (`ffmpeg`).

### Aplikácie s grafickým rozhraním

```powershell
winget install --id=Microsoft.PowerToys -e        # PowerToys, užitočné doplnky pre Windows
winget install --id=Microsoft.WindowsTerminal -e  # Windows Terminal, moderný terminál s kartami
winget install --id=voidtools.Everything -e       # Everything, okamžité hľadanie súborov na disku
winget install --id=Notepad++.Notepad++ -e        # Notepad++, rýchly textový editor
winget install --id=ShareX.ShareX -e              # ShareX, snímky obrazovky a nahrávanie
winget install --id=7zip.7zip -e                  # 7-Zip, práca s archívmi
winget install --id=VideoLAN.VLC -e               # VLC, prehrávač prehrá takmer všetko
winget install --id=WinDirStat.WinDirStat -e      # WinDirStat, čo zaberá miesto na disku
```

## Záloha a obnova zoznamu programov

Uloženie zoznamu nainštalovaných programov do súboru:

```powershell
winget export -o programy.json
```

Inštalácia všetkých programov zo súboru, napríklad na novom počítači:

```powershell
winget import -i programy.json
```

## Poznámky

- Presné ID zistíte vždy cez `winget search`, môže sa časom zmeniť.
- Po inštalácii nástrojov do príkazového riadka (`jq`, `rg`, `ffmpeg`) otvorte nový terminál, aby sa aktualizovala premenná `PATH`.
- Niektoré inštalácie si vyžiadajú práva správcu, Windows vtedy zobrazí okno UAC.
- Rovnaký princíp ako pri `uv`: jeden príkaz namiesto ručného sťahovania a inštalácie.

## Cvičenie: ripgrep (rg)

**ripgrep** (`rg`) je veľmi rýchly nástroj na hľadanie textu v súboroch. Prehľadá celý priečinok aj s podpriečinkami a za pár sekúnd nájde, v ktorom súbore sa nachádza hľadané slovo. Používajú ho aj AI agenti, keď hľadajú v súboroch.

> **Dôležité:** `rg` hľadá v **textových súboroch** (`.txt`, `.md`, `.csv`, `.html`, `.eml` a podobne). Vo vnútri dokumentov **Word (`.docx`) a PDF** nehľadá, ich **názvy** však nájsť vie (pozri časť 4).

### 1. Vyhľadanie a inštalácia

```powershell
winget --version
winget search ripgrep
winget show --id BurntSushi.ripgrep.MSVC
winget install --id BurntSushi.ripgrep.MSVC -e
```

### 2. Overenie (v novom termináli)

```powershell
rg --version
where.exe rg          # ukáže, kde je rg nainštalovaný
```

### 3. Hľadanie v súboroch

V PowerShelli dávajte hľadaný text do jednoduchých úvodzoviek `'...'`, aby PowerShell neupravoval špeciálne znaky.

Najprv sa presuňte do priečinka Dokumenty:

```powershell
cd $env:USERPROFILE\Documents
```

```powershell
rg 'zmluva'                              # hľadá slovo „zmluva“ vo všetkých súboroch a podpriečinkoch
rg -i 'bratislava'                       # bez ohľadu na veľké a malé písmená
rg -w 'mail'                             # len celé slová (nenájde „mailový“)
rg -n 'heslo'                            # s číslami riadkov
rg -C 2 'stretnutie'                     # 2 riadky pred a za výsledkom, aby ste videli súvislosti
rg -l 'faktúra'                          # len názvy súborov, ktoré obsahujú slovo
rg -c 'objednávka'                       # koľkokrát sa slovo nachádza v každom súbore
rg 'Novák|Nováková'                      # jedno alebo druhé slovo naraz
```

### 4. Hľadanie súborov podľa názvu

Prepínač `--files` nehľadá v obsahu, ale vypíše zoznam súborov. S `-g` ho zúžite podľa názvu alebo prípony. Takto nájdete aj dokumenty Word a PDF:

```powershell
rg --files -g '*.pdf'                              # všetky PDF súbory
rg --files -g '*.docx'                             # všetky dokumenty Word
rg --files -g '*.{jpg,png}'                        # všetky obrázky JPG a PNG
rg --files -g '*faktura*'                          # súbory, ktoré majú v názve „faktura“
rg --files -g '*.pdf' $env:USERPROFILE\Downloads   # PDF súbory v priečinku Stiahnuté
```

### 5. Hľadanie len v určitých súboroch

```powershell
rg -i 'recept' -g '*.txt'                # len v textových súboroch
rg -i 'dovolenka' -g '*.md'              # len v poznámkach Markdown
rg -i 'Košice' -g '*.csv'                # len v tabuľkách CSV (napr. export kontaktov)
rg -i 'rezervácia' -g '*.eml'            # len v uložených e-mailoch
rg -i 'kontakt' -g '!*.csv'              # všade okrem súborov CSV
```

### 6. Hľadanie v rôznych priečinkoch

```powershell
rg -i 'poistka' $env:USERPROFILE\Documents     # v Dokumentoch
rg -i 'lístok' $env:USERPROFILE\Downloads      # v priečinku Stiahnuté
rg -i 'nákup' $env:USERPROFILE\Desktop         # na Ploche
rg -i 'projekt' D:\Archiv                      # na inom disku
```

### 7. Filtrovanie výstupu iných príkazov

`rg` vie vyfiltrovať aj výstup iných príkazov, podobne ako `findstr` vo Windows, len rýchlejšie a pohodlnejšie:

```powershell
ipconfig | rg 'IPv4'                     # IP adresa počítača
tasklist | rg -i 'chrome'                # beží prehliadač Chrome?
winget list | rg -i 'vlc'                # je nainštalovaný VLC?
```

### 8. Úlohy na samostatnú prácu

1. Vytvorte v priečinku Dokumenty súbor `nakup.txt` s niekoľkými položkami a nájdite v ňom slovo `mlieko`.
2. Nájdite všetky textové súbory v Dokumentoch, ktoré obsahujú vaše meno (len názvy súborov).
3. Vypíšte všetky PDF súbory v priečinku Stiahnuté.
4. Zistite IP adresu svojho počítača cez `ipconfig` a `rg`.
5. Na konci spustite `winget upgrade` a pozrite sa, či je dostupná novšia verzia `rg`.

## Odkazy

- Dokumentácia: <https://learn.microsoft.com/windows/package-manager/winget/>
- Prehliadač balíčkov: <https://winget.run>
