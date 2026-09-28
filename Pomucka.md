# 📚 C++ Architektura, Operátory a Algoritmy (Hlavní tahák)

Tento dokument slouží jako univerzální průvodce pro objektově orientované zkouškové projekty v C++. Shrnuje pravidla rozdělení kódu, syntaxi operátorů, algoritmy a bezpečnou práci s pamětí.

---

## 1. Zlatá pravidla pro `const` (Slib, že nic nezměním)
Klíčové slovo `const` se v třídách používá na dvou místech. **Co slíbíš v `.h`, to musíš do písmene opsat v `.cpp`.**

* **Pravidlo A: Konec metody = "Nezměním sám sebe"**
  Pokud dáš `const` na úplný konec (za závorky), slibuješ, že metoda neupraví žádný atribut aktuálního objektu.
  * *Kdy psát:* Gettery, výpisy (`vypisInfo`), analýzy, porovnávání (`==`, `<`). Tyto metody jen čtou.
  * *Kdy NEpsát:* Settery, přidávání dat, operátor `+=`. Tyto metody fyzicky upravují data.
* **Pravidlo B: Uvnitř u parametru = "Nedotknu se tvých dat"**
  Pokud předáváš složitější typ (text `std::string`, nebo jiný objekt) odkazem přes `&`, přidej `const`. Slibuješ tím, že poslanou proměnnou uvnitř funkce nepřepíšeš.

---

## 2. Pravidla zápisu: Hlavička (.h) vs. Logika (.cpp)

*   **Hlavičkový soubor (`.h`):** Slouží jako deklarace. Definuje se zde samotné tělo třídy a hlavičky metod (bez jejich vnitřní logiky).
*   **Zdrojový soubor (`.cpp`):** Obsahuje samotnou logiku. Před názvem každé metody musí být uveden název třídy s dvojitou dvojtečkou (např. `BazovaTrida::vypisInfo()`), aby překladač věděl, kam metoda patří. Uvnitř `.h` souboru se čtyřtečka nikdy nepíše!

---

## 3. Bázová třída (Základní šablona / Rodič)

Bázová třída definuje společné vlastnosti pro všechny odvozené objekty. 

**V hlavičce (`.h`):**
*   Konstruktor (přijímá základní parametry, typicky `std::string nazev`).
*   Destruktor (musí mít klíčové slovo `virtual`, aby fungovalo správné smazání u potomků z paměti).
*   Gettery (vracejí chráněná data).
*   Obyčejné virtuální metody (zapisují se bez parametrů, např. `virtual void vypisInfo() const;`).
*   Deklarace metod pro přidávání dat (normální pro jednu hodnotu a přetížená pro celý vektor).

**V implementaci (`.cpp`):**
*   Úplně na začátku souboru (mimo metody) se musí inicializovat statické počítadlo: `int BazovaTrida::citac = 0;`.
*   Konstruktor: Atributy (jako je název) se nastavují hned za závorkou přes dvojtečku: `BazovaTrida(string oznaceni) : nazev(oznaceni)`. Řeší se zde také zvýšení počítadla (`citac++`).
*   Destruktor: Řeší se snížení počítadla (`citac--`).
*   **Přidávání dat do vektoru (Normální metoda):** Předaný vlastní parametr se prostě hodí na úplný konec vektoru pomocí `push_back`.
*   **Přidávání dat do vektoru (Přetížená metoda):** Slouží k vložení více hodnot naráz. Jede se přes metodu `insert`, která má formát: `(konec hlavního vektoru, začátek vl. vektoru(para), konec vl. vektoru)`.

---

## 4. Odvozené třídy (Dědičnost / Potomek)

Specializované implementace, které rozšiřují bázovou třídu.

**V hlavičce (`.h`):**
*   Dědičnost se zapisuje rovnou k definici třídy: `class OdvozenaTrida : public BazovaTrida`.
*   U metod, které se přepisují z bázové třídy, se na konec přidává klíčové slovo `override`.

**V implementaci (`.cpp`) - Konstruktor a předávání dat:**
*   Konstruktor potomka bere v parametrech typicky `string` pro bázovou třídu a pak své vlastní proměnné.
*   V implementaci `.cpp` musí závorka obsahovat všechny parametry. Část z nich se pošle rovnou "nahoru" rodiči a zbytek si potomek uloží.
    `OdvozenaTrida::OdvozenaTrida(const std::string& jmeno, double hodnota) : BazovaTrida(jmeno), vlastniPromenna(hodnota) {}`
*   Ve vlastní upravené metodě pro výpis je dobré zavolat nejprve výpis z bázové třídy a pak vypsat specifika daného potomka.
*   Při procházení historie se využívá range-based for cyklus: `for(typ kamUlozim : coProjizdim)`.

---

## 5. Přetěžování operátorů

Umožňuje používat standardní matematické a logické znaky pro naše vlastní objekty.

**V hlavičce (`.h`):**
*   `bool operator==(const OdvozenaTrida& druhyObjekt) const;` (Porovnání - vrací ano/ne. Pozor na obě `const`!).
*   `OdvozenaTrida& operator+=(double novaHodnota);` (Přidání - upravuje aktuální objekt, vrací referenci).
*   `friend std::ostream& operator<<(std::ostream& os, const OdvozenaTrida& objektKterovyVypisuji);` (Výpis do konzole - `friend` umožňuje přístup k privátním datům).

**V implementaci (`.cpp`):**
*   **Porovnání (`==`):** Má předponu třídy (čtyřtečku). Vrací výsledek porovnání s využitím ukazatele na sebe sama: `return this->vlastniAtribut == druhyObjekt.vlastniAtribut;`.
*   **Přidání (`+=`):** Má předponu třídy. Modifikuje vnitřní data a na konci vrací referenci na aktuální objekt: `return *this;`.
*   **Výpis (`<<`):** Píše se **bez** předpony třídy (bez čtyřtečky) a **bez** slova `friend`. Formátuje se klasicky přes proud a na konci vrací proud: `return os;`.

---

## 6. Algoritmy (Volné funkce v mainu)

Algoritmy se píšou jako běžné funkce nad `main()` a jako parametr obvykle dostávají ukazatel na bázovou třídu, aby fungovaly pro všechny potomky (polymorfismus). 

*   **Pozor u mazání:** Pokud algoritmus upravuje strukturu dat (např. maže prvky a nevrací hodnotu, tedy je typu `void`), nesmí se jeho volání dávat přímo do `std::cout <<`. Zavolá se na samostatném řádku a teprve poté se vypíše nový stav objektu.

**Algoritmus 1: Procházení a vyhodnocování (např. hledání, počítání)**
*   Funkce přijímá ukazatel: `void algoritmus1(BazovaTrida* s)`.
*   Využívá range-based for cyklus: `for(double hodnota : s->getHistorie())`.

**Algoritmus 2: Podmíněné mazání (`erase`)**
1. Ve funkci se musí vytvořit přímá reference na vektor, **nikoliv kopie**! `std::vector<double>& vektor = s->getHistorie();`
2. Cyklus prochází vektor přes iterátor záměrně bez třetího parametru pro posun: `for(auto it = vektor.begin(); it != vektor.end(); /* bez posunu */)`.
3. Uvnitř se přes dereferenci (`*it`) vyhodnotí podmínka. Pokud platí, smaže se prvek přes `it = vektor.erase(it)` (tím se získá nová pozice). Pokud neplatí, provede se ruční posun `++it`.

**Algoritmus 3: Vytvoření a návrat nového vektoru (Filtrování)**
*   Na začátku těla funkce se založí prázdný vektor (např. `vysledek`).
*   Přes range-based for cyklus se projdou stará data a pokud splní podmínku, vloží se do nového přes `vysledek.push_back(hodnota)`.
*   Na úplném konci se zavolá `return vysledek;`.

---

## 7. Hlavní program (`main.cpp`) a bezpečný průchod

Celá funkce `main` musí mít chronologickou strukturu pro bezpečný test polymorfismu.

1.  **Ověření výchozího stavu:** Výpis statického čítače (měl by být 0).
2.  **Vytvoření struktury a alokace:** Vytvoří se `std::vector<BazovaTrida*>`, do kterého se přes operátor `new` vloží různé odvozené objekty.
3.  **Naplnění daty:** Přes indexy a operátor šipky se objektům přidají data (např. `kolekce[0]->pridejData(...)`).
4.  **Polymorfní průchod (Hlavní cyklus):** Range-based for cyklus pro ukazatele: `for(BazovaTrida* s : senzory)`. Zavolají se na něm virtuální metody (každý se chová dle svého typu).
5.  **Spuštění algoritmů:** Zavolají se externí funkce a předá se jim konkrétní ukazatel (např. `algoritmus(senzory[0])`).
6.  **Test operátorů na lokálních proměnných:** Vytvoří se daný druh odvozené třídy na zásobníku bez `new` (např. `LokalA("nazev", hodnota)`). Otestují se operátory `+=`, `==` a výpis `<<`.
7.  **Úklid paměti (`delete`):** Než program skončí, projde se původní vektor ukazatelů a ručně se smažou objekty (`delete s;`).
8.  **Závěrečná kontrola:** Výpis čítače musí ukázat opět 0.
