# AGENTS.md – Pravidla projektu a instrukce pro AI asistenta

Tento soubor slouží jako centrální a závazná sada instrukcí pro AI asistenta v prostředí IDE (Antigravity, Cursor, Windsurf, VS Code, Gemini).

---

# ČÁST I: PRAVIDLA TOHOTO PROJEKTU (Project-Specific)

## 1. Specifikace a technologický stack
- **Název projektu / repozitáře:** Descriptron (`v2.8.8`, upstream `https://github.com/alexrvandam/Descriptron`)
- **Hlavní účel:** Morfologicko-fenomická a BioRAG pipeline pro automatizované taxonomické popisy druhů, tvorbu dichotomických klíčů, delimitaci druhů, AI-asistovanou segmentaci a landmarking (SAM2-PAL, DINOLand, DINOMask, SAM 3, Detectron2) a 2D geometrickou morfometriku / analýzu barev, textur a fylogenetického signálu.
- **Tech stack:** Python (`>=3.9`, doporučeno `3.12`), NumPy, Pandas, SciPy, scikit-learn, scikit-image, OpenCV (`opencv-python-headless`), pycocotools, matplotlib, seaborn, statsmodels, Biopython, python-docx; volitelně PyTorch / torchvision, SAM2 / SAM 3, DINOv3 (`transformers`), Detectron2, Anthropic SDK (`anthropic`), RAG (`requests`, `pdfplumber`, `faiss-cpu`, `sentence-transformers`), MCP server (`descriptron-mcp`), Docker, Conda; referenční validační skripty v R (`geomorph`, `RRPP`, `ape`, `phytools`, `vegan`, `patternize`, `Colormesh`).
- **Spouštěcí a testovací příkazy:**
  - Grafické rozhraní (GUI): `descriptron-gui` (nebo `python descriptron/descriptron-v2-v99.py`)
  - CLI analýzy: `descriptron --list` (nebo `python -m descriptron_core.cli --list`)
  - Testy: `pytest descriptron/measure/tests`
  - Synchronizace balíčků v `packages/`: `python packages/sync_sources.py`

## 2. Architektura a specifické konvence projektu
- **Klíčové složky a soubory:**
  - `descriptron/` – hlavní aplikace a skripty (GUI `descriptron-v2-v99.py`, `sam2_pal_batch_v21.py`, `dinov3_landmark_transfer_v52.py`, `dinomask/`, `detectron2/`, `torchvision_det/`, `test_brush/`)
  - `descriptron/measure/` – měření, geometrická morfometrika, barvy, textury, fylogenetika (`descriptron_phylo.py`, `descriptron_trait_stats.py`) a BioRAG pipeline (`run_full_pipeline_v2.py`)
  - `descriptron/measure/tests/` a `descriptron/measure/validation/` – jednotkové testy (`pytest`) a R validační benchmarky
  - `packages/` – definice distribučních balíčků (`descriptron`, `descriptron-core`, `descriptron-vision`, `descriptron-gui`, `descriptron-mcp`, `descriptron-sam3`) a `sync_sources.py`
  - `docker/` a `environments/` – Dockerfiles, build skripty a Conda/pip prostředí
  - `docs/` – dokumentace a návody (`TUTORIAL.md`, `DOCKER_RECIPE.md`, `INSTALL_PIP.md`, `SAM2PAL_DINOLand_Annotation_SOP.md`)
- **Specifická pravidla pro data a stav:**
  - Hmotnosti modelů a checkpointy (`*.pth`, `*.pt`, `*.ckpt`, `*.onnx`, `*.safetensors`), složky `checkpoints/`, `outputs/` a `*.log` jsou v `.gitignore` a do repozitáře nepatří.
  - Popisy generované přes BioRAG musí vycházet výhradně z naměřené datové matice a projít nezávislým auditem (`biorag_confabulation_checker_v2.py`).
- **Větvení a deployment pravidla:**
  - Výchozí větev repozitáře je `main` (`origin` -> `https://github.com/alexrvandam/Descriptron.git`). Před úpravami nebo verzováním ověřit dotčené testy pomocí `pytest descriptron/measure/tests`.
- **Přístup k souborům:**
  - Před pokusem o přístup mimo složku tohoto workspace se vždy zeptej. Je nutné zamezit nepovoleným přístupům na jiné disky a k souborům mimo tento workspace.

  
  

---

# ČÁST II: UNIVERZÁLNÍ GLOBÁLNÍ STANDARDY (Global Core)

## 1. Komunikace, minimalismus a kontrola nad změnami

- **Méně je někdy více:**
  - Preferuj jednoduchá, čistá, funkční a věcná řešení.
  - Vyvaruj se překombinovaných návrhů a zbytečného kódu navíc.
- **Audit a transparentnost nepožádaných změn kódu:**
  - Pokud v kódu provedeš jakoukoliv změnu, refaktoring nebo optimalizaci, **o kterou uživatel v zadání výslovně nežádal**, VŽDY tuto změnu v odpovědi zřetelně zdůrazni a vysvětli její důvod.
  - Nápady a návrhy nad rámec původního zadání neimplementuj svévolně do kódu.
- **Aktivní dotazování při plánování a změnách (`ask_question`):**
  - Při plánování dalšího postupu nebo pokud má být provedena jakákoliv změna, která nebyla agentovi specificky a výslovně zadána, se agent **frekventovaně ptá uživatele pomocí nástroje `ask_question`**.
  - Nevytvářej domněnky a před zahájením implementace si vždy ujasni varianty a priority.
- **Automatické ukládání postřehů a návrhů (SUGGESTIONS.md):**
  - **Zásada nezdržování:** Asistent aktivně a cíleně nevyhledává vedlejší problémy a neodvádí pozornost od zadaného úkolu. Zaznamenává výhradně postřehy, na které přirozeně narazí během řešení (např. technický dluh, skrytý bug, neoptimální kód, bezpečnostní riziko, chybějící typování), a které by jinak pouze napsal do chatu, kde hrozí jejich zapadnutí.
  - **Umístění a inicializace:** Návrhy ukládej do souboru `SUGGESTIONS.md` v kořenovém adresáři projektu. Pokud soubor ještě neexistuje, asistent jej vytvoří s logickým rozdělením do kategorií.
  - **Prevence duplicit:** Před zápisem vždy nejprve zkontroluj existující obsah `SUGGESTIONS.md`. Pokud obdobný záznam již existuje, duplicitu nevytvářej (případně pouze doplň nový kontext či odkaz na další soubor).
  - **Formát zápisu:** Každý záznam vlož jako odškrtávací položku checklistu do odpovídající kategorie:
    ```markdown
    - [ ] **[Výstižný název problému či námětu]**
      - **Kde:** `cesta/k/souboru` (včetně řádku či symbolu)
      - **Kontext:** Stručný popis zjištěného stavu nebo rizika (1–2 věty).
      - **Návrh řešení:** Konkrétní doporučený postup nápravy.
    ```
  - **Doporučené kategorie:**
    - `## Architektura a kód`
    - `## UI, UX a frontend`
    - `## Bugy a potenciální rizika`
    - `## Výkon a optimalizace`
    - `## Dokumentace a procesy`
  - **Chování v chatu:** Nezahlcuj chat rozsáhlými rozbory vedlejších nápadů. Do odpovědi vlož pouze stručnou jednovětou notifikaci s odkazem na soubor, např. *„Při úpravách jsem zaznamenal prostor pro refaktoring – detailní návrh jsem zapsal do [SUGGESTIONS.md](SUGGESTIONS.md).*“
- **Striktní zákaz emoji:**
  - V celém projektu (uživatelské rozhraní, dialogy, toast zprávy, kód, komentáře, commit zprávy, texty i odpovědi asistenta) je **přísně zakázáno používat emoji**.
  - Místo emotikonů používej výhradně **pěkné, minimalistické, jednobarevné inline SVG**.
  - Zákaz Unicode vlaječek států (na Windows se vykreslují jako textová písmena) – vždy nahradit inline SVG vlajkou.
- **Faktická přesnost bez domýšlení:**
  - Nikdy si nevymýšlej neověřené parametry, čísla nebo fakta. Pokud chybí přesný zdroj či data, zeptej se uživatele nebo nastav prvek do stavu konceptu.

---

## 2. Nástroje IDE, Git a verzování

- **Tool Usage Preferences:**
  - Always prefer native/built-in IDE file tools (`read_file`, `edit_file`, `write_file`, `view_file_outline`, `search_files`) over terminal commands.
  - K čtení, úpravám a vytváření souborů používej nativní nástroje `read_file`, `edit_file` a `write_file`.
  - Pro vyhledávání na internetu a dohledávání dokumentace používej `search_web`.
  - Do not run shell commands (`cat`, `Get-Content`, `type`, `head`, `tail`, `grep`) solely to read or inspect file contents or search directory structures.
  - Reserve terminal commands strictly for building, running test suites, git operations, or executing environment-specific scripts.
  - When reading large files, inspect targeted line slices rather than fetching entire documents.
  - Nepoužívej terminálové zkratky typu `node -e` ani ad-hoc bash skripty pro inspekci souborů.
- **Grounding v biologických a myrmekologických tématech:**
  - For grounding in ant related topics, always use **antscout MCP** tools.
- **Commit message (Conventional Commits):**
  - Na konci každé zprávy v chatu připrav výstižný návrh commit zprávy v angličtině podle konvence (`feat(...)`, `fix(...)`, `chore(...)`, `docs(...)`, `refactor(...)`, `style(...)`).
  - Vypiš **pouze čistý text zprávy** bez balení do `git commit -m` a bez uvozovek.
  - Asistent **nikdy nespouští commit ani push automaticky**, pouze navrhne zprávu a čeká na potvrzení uživatelem.
- **Sémantické verzování (SemVer):**
  - Při aktualizaci verzí (`manifest.json`, `package.json` apod.) dodržuj:
    - **PATCH (`x.y.Z+1`):** Opravy chyb, drobný refaktoring, styly, textové úpravy.
    - **MINOR (`x.Y+1.0`):** Nové funkce, nové UI komponenty, rozšíření API.
    - **MAJOR (`X+1.0.0`):** Zlomové změny architektury, nekompatibilní přepracování.
- **Udržování dokumentace:**
  - Při zásadní změně struktury kódu nebo novém prvku adekvátně aktualizuj soubory dokumentace (`README.md`, `STRUKTURA_PROJEKTU.md`).

---

## 3. UI, UX a frontendový design

- **Jednoduchost UI prvků:**
  - Tlačítka, položky navigace a nadpisy formuluj stručně a výstižně (ideálně 1–2 slova).
  - Do UI nepřidávej svévolné dekorativní štítky a odznaky (badges jako „Novinka“, „Báze znalostí“), pokud nebyly vyžádány.
- **Parita světlého a tmavého režimu (Dark / Light mode):**
  - Při jakékoliv úpravě barev nebo motivů zajisti adekvátní kontrast a funkčnost v obou režimech současně.
- **Staging a produkční synchronizace:**
  - Pokud projekt využívá trvalé testovací či staging stránky (`temp-*`), udržuj jejich kód v synchronizovaném stavu s odpovídajícími ostrými verzemi.

---

## 4. Jazyková pravidla a česká mikrotypografie

- **Výchozí jazyk:** Řiď se jazykem promptu (typicky čeština; u webových projektů s `/en/` verzí udržuj překladovou paritu).
- **Zásady české mikrotypografie:**
  - **Pevné mezery (`&nbsp;` / `\u00A0`):** Neslabičné a jednopísmenné předložky a spojky (*k, s, v, z, o, u, a, i*) nesmí zůstávat na konci řádku. Vždy je navaž pevnou mezerou k následujícímu slovu.
  - **České uvozovky:** V českých textech používej výhradně uvozovky 99–66: dole `„` (U+201E) a nahoře `“` (U+201C). Zákaz rovných `""` i anglických `“...”`.
  - **Pomlčka vs. spojovník:**
    - Půlčtverčíková pomlčka (`–`, en-dash): Pro číselné rozsahy bez mezer (*10–15 mm*, *20–25 °C*) a jako větná pomlčka oddělená mezerami po obou stranách.
    - Spojovník (`-`, hyphen): Pouze pro těsně vázaná složená slova (*česko-slovenský*) nebo dělení slov.
  - **Jednotky a procenta:** Mezi číslem a jednotkou je vždy pevná mezera (`12&nbsp;mm`, `24&nbsp;°C`). U procent: `10 %` = deset procent (podstatné jméno), `10%` = desetiprocentní (přídavné jméno).
  - **Desetinná čísla a řády:** V českém textu piš desetinnou čárku (*1,5 mm*), v anglickém tečku (*1.5 mm*). Tisíce a vyšší řády odděluj mezerou (*10 000*).
- **Japonský jazyk (pokud je v projektu použit):**
  - Přepis do latinky uváděj vždy **českým přepisem** (vhodně doplněný o Hepburn). U kandži vždy doplň přepis do kany.

---

# ČÁST III: SPECIALIZOVANÝ MODUL – BIOLOGIE A MYRMEKOLOGIE
<!-- Tento modul je aktivní při tvorbě biologického, taxonomického a chovatelského obsahu -->

## 1. Binomická nomenklatura (Vědecké názvosloví)
- **Rod (Genus):** První písmeno velké, formátováno kurzívou (*Camponotus*, *Odontomachus*, *Lasius*, *Formica*).
- **Druhový přívlastek:** Všechna písmena malá, formátováno kurzívou (*ligniperda*, *tyrannicus*, *niger*).
- **Skupina druhů (Species group):** Pouze druhové jméno kurzívou, slovo „group“ nebo „skupina druhů“ normálním písmem bez kurzívy (*tyrannicus* group, skupina druhů *rufa*).
- **Vyšší taxony (Tribus, Podčeleď, Čeleď):** Velké počáteční písmeno, **normální písmo bez kurzívy** (Camponotini, Formicinae, Formicidae).
- **Zákaz CSS capitalize:** Nikdy nepoužívat CSS vlastnost `capitalize` ani automatické kapitálky na druhy (nepřípustné: *Camponotus Ligniperda*).

## 2. Odborná terminologie a překlady
- **Zákaz překládání molekulárních a genetických termínů:** Termíny jako *COI*, *DNA barcoding*, *Barcoding*, *Barcodes*, *PCR*, *mtDNA* apod. se **NIKDY NEPŘEKLÁDAJÍ** do češtiny (např. nepoužívat výrazy „čárové kódy DNA“). Ponechat standardní vědecké označení.
- **Přesná myrmekologická morfologie:**
  - Upřednostňuj exaktní terminologii (*gaster*, *petiolus*, *trofalaxe*, *nanitik*, *pleometróza*, *gamergát*, *ergatoid*).
  - Nepřidávat české názvy do závorek za odborné termíny, které se běžně nepřekládají.
  - **Diapauza:** Mravenci mají diapauzu; nepoužívat paušálně termíny „hibernace“ či „zimování“, pokud to není přesně fyziologicky podloženo.
  - **Subkasty a kasty:** Major je subkasta dělnice, voják (soldier) je samostatná kasta s unikátními morfologickými znaky (fragmotická hlava, šavlovitá kusadla). Množné číslo: majoři, minoři, medie (nikoliv „majory“, „minora“).

## 3. Grounding a fakta o mravencích (antscout MCP)
- **Ověřování dat:** Pro grounding in ant related topics vždy používej **antscout MCP** (nástroje jako `search_species`, `species_profile`, `genus_overview`, `get_antwiki`, `search_facts`, `search_papers`, `search_specimen_images`, `search_distribution`).