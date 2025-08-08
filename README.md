Cílem skriptu je vytvořit plně dynamickou webovou příručku. Veškerý obsah a struktura aplikace (tlačítka, obrázky, parametry) nejsou napevno zapsány v HTML, ale jsou načítány a skládány za běhu z externích .csv souborů. Tím je zajištěno, že pro jakoukoliv změnu nebo přidání dat stačí upravit pouze textové soubory .csv, bez nutnosti sahat do samotného kódu.

Struktura kódu a jeho průběh
Kód lze rozdělit do 4 hlavních fází: Konfigurace, Načítání dat, Zpracování dat a Vykreslení a interaktivita.

1. Konfigurace a globální proměnné
Na samém začátku skriptu najdete:

config: Malý objekt, který obsahuje základní nastavení.

categoriesFile: Název souboru, který funguje jako "rozcestník" pro celou aplikaci (kategorie.csv).

imageBasePath: Cesta ke složce s obrázky (images/).

appData = {}: Prázdný objekt, který je nejdůležitější proměnnou v celé aplikaci. Během načítání se postupně naplní veškerými daty ze všech CSV souborů ve strukturované, vnořené podobě (kategorie -> typy -> varianty).

activeCategory a activeType: Proměnné, které si "pamatují", na jaké tlačítko kategorie a typu uživatel zrovna kliknul. Řídí, co se má na stránce zobrazovat.

Proměnné pro elementy: Odkazy na jednotlivé části (kontejnery) v HTML, se kterými JavaScript pracuje (např. categorySelector pro tlačítka kategorií).

2. Spuštění a načítání dat (funkce loadApplicationData)
Toto je hlavní mozek celé operace, který se spustí ihned po načtení stránky.

Fáze 1: Načtení seznamu kategorií

Skript se pomocí funkce fetch pokusí stáhnout soubor definovaný v config.categoriesFile (tedy kategorie.csv).

Pokud se to nepodaří (např. soubor neexistuje), zobrazí fatální chybu a skončí.

Pokud se soubor podaří načíst, jeho textový obsah se převede na pole objektů (každý řádek CSV je jeden objekt) a uloží do dočasné proměnné categoryList.

Fáze 2: Načtení dat pro každou kategorii

Skript projde categoryList a pro každou položku (kategorii) v něm se pokusí stáhnout odpovídající datový soubor. Název souboru se dynamicky sestaví z názvu kategorie (např. Lisovací šrouby -> Lisovací šrouby.csv).

Používá se zde Promise.all, což znamená, že se skript souběžně pokusí stáhnout všechny datové soubory najednou, což je velmi efektivní.

Pokud se některý z těchto datových souborů nepodaří najít, zobrazí se na stránce konkrétní chyba (díky vylepšenému hlášení chyb), ale proces pokračuje dál s ostatními soubory.

3. Zpracování dat (funkce processCategoryData)
Jakmile jsou data pro konkrétní kategorii stažena, tato funkce je vezme a správně zařadí do hlavní datové struktury appData.

Určení klíče pro varianty: Podívá se do informací načtených z kategorie.csv. Pokud je ve sloupci Sloučení číslo (např. 3), zjistí si název třetího sloupce v datovém souboru (např. L) a uloží si ho jako klíč pro budoucí seskupování variant. Pokud je Sloučení prázdné, klíč pro varianty nebude existovat.

Vytvoření struktury: Vytvoří v appData záznam pro danou kategorii a uloží si k ní ikonu, obrázek a zjištěný klíč pro varianty.

Seskupení podle typu: Projde všechny řádky z datového souboru a seskupí je podle hodnoty ve sloupci Typ. Vzniknou tak podskupiny jako M3, M4 atd.

Uložení variant: Každý jednotlivý řádek z datového CSV se uloží jako jedna "varianta" do příslušné podskupiny daného typu.

Po dokončení tohoto procesu pro všechny kategorie je objekt appData kompletní a připravený k použití.

4. Vykreslení a interaktivita
Tato část se stará o to, aby se data z appData zobrazila uživateli a aby stránka reagovala na kliknutí.

initializeApp(): Spustí se po úspěšném načtení všech dat. Skryje načítací animaci a spustí první vykreslení tlačítek kategorií.

render...() funkce (renderCategoryButtons, renderTypeButtons): Tyto funkce čtou data z appData a dynamicky generují HTML kód pro tlačítka. Zároveň kontrolují, které tlačítko má být zrovna aktivní (zvýrazněné).

select...() funkce (selectCategory, selectType): Jsou to "posluchači událostí", kteří se zavolají, když uživatel na něco klikne. Jejich úkolem je:

Aktualizovat globální proměnné (activeCategory, activeType).

Znovu zavolat render...() funkce, aby se překreslila tlačítka a zvýraznilo se to aktivní.

Zavolat funkci showDetails() nebo zobrazit výchozí text.

showDetails(): Tato funkce je zodpovědná za zobrazení finální karty produktu.

Podle aktivní kategorie a typu najde v appData příslušná data.

Podívá se, zda má daná kategorie definovaný klíč pro varianty.

Pokud ano, vygeneruje tlačítka pro varianty (např. délky).

Vygeneruje HTML pro obrázek, popisek a tabulku s technickými parametry pro aktuálně zvolenou variantu.

Vloží celý vygenerovaný HTML kód do stránky.
