# uv – moderný nástroj pre Python

## Kde sa s uv stretneme

V kurze AI agentov a kustomizácie sa s uv stretneme pri:

- **spúšťaní Python skriptov** a nástrojov, ktoré agent používa (`uv run`),
- **MCP serveroch** napísaných v Pythone, ktoré sa často spúšťajú cez `uvx`,
- **vlastných skills a hookoch**, ktoré sú malé Python skripty so závislosťami,
- **vlastných agentoch** postavených na SDK, kde potrebujeme čisté a reprodukovateľné prostredie.

---

## Čo je uv

**uv** je rýchly správca Pythonu, balíčkov a projektov (napísaný v Ruste). Jeden nástroj nahrádza `pip`, `venv`, `pyenv` a čiastočne aj `pipx`.

## Inštalácia

Windows (PowerShell):

```powershell
powershell -c "irm https://astral.sh/uv/install.ps1 | iex"
```

Windows (alternatíva cez winget):

```powershell
winget install --id=astral-sh.uv -e
```

macOS / Linux:

```bash
curl -LsSf https://astral.sh/uv/install.sh | sh
```

Overenie:

```bash
uv --version
```

## Najčastejšie príkazy

| Príkaz | Čo robí |
|---|---|
| `uv init mojprojekt` | Vytvorí nový projekt |
| `uv add requests` | Pridá závislosť do projektu |
| `uv remove requests` | Odstráni závislosť |
| `uv run main.py` | Spustí skript v správnom prostredí |
| `uv sync` | Nainštaluje všetky závislosti z projektu |
| `uv python install 3.12` | Nainštaluje danú verziu Pythonu |
| `uvx ruff check .` | Spustí nástroj jednorazovo, bez inštalácie |
| `uv tool install ruff` | Nainštaluje nástroj globálne |

## Prvý projekt za minútu

```bash
uv init ahoj-agent
cd ahoj-agent
uv add anthropic
uv run main.py
```

uv sám vytvorí virtuálne prostredie (`.venv`) a súbory `pyproject.toml` a `uv.lock`. Prostredie netreba ručne aktivovať.

## Samostatný skript so závislosťami

Závislosti sa zapíšu priamo do skriptu:

```bash
uv add --script skript.py requests
uv run skript.py
```

Hodí sa to na malé nástroje, ktoré používame opakovane.

## Poznámka: podobne je to v JavaScripte

Ak poznáte JavaScript, uv má svoje obdoby v ekosystéme Node.js:

| Python (uv) | JavaScript (Node.js) | Význam |
|---|---|---|
| `uvx ruff check .` | `npx eslint .` | Jednorazové spustenie nástroja bez trvalej inštalácie |
| `uv tool install ruff` | `npm install -g eslint` | Globálna inštalácia nástroja |
| `uv add requests` | `npm install axios` | Pridanie závislosti do projektu |
| `uv run main.py` | `node index.js` / `npm run` | Spustenie kódu v prostredí projektu |
| `pyproject.toml` + `uv.lock` | `package.json` + `package-lock.json` | Zoznam závislostí a ich zamknuté verzie |
| `.venv` | `node_modules` | Lokálne uložené závislosti projektu |

Pri MCP serveroch to uvidíte v praxi: server v Node.js sa spúšťa cez `npx`, server v Pythone cez `uvx`. Princíp je rovnaký.

## Cvičenie

1. Nainštalujte uv a overte `uv --version`.
2. Vytvorte projekt cez `uv init` a pridajte jednu knižnicu.
3. Spustite `main.py` cez `uv run`.
4. Skúste `uvx ruff --version` a všimnite si, že nič sa neinštaluje natrvalo.

## Odkazy

- Dokumentácia: <https://docs.astral.sh/uv/>
