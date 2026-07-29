# 📚 C++ Architektura, Operátory a Algoritmy (Obecný tahák)

Tento dokument slouží jako univerzální průvodce pro objektově orientované zkouškové projekty v C++. Shrnuje pravidla rozdělení kódu, syntaxi operátorů a bezpečnou práci s pamětí.

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

Při psaní kódu se třídy rozdělují do dvou souborů:
*   **Hlavičkový soubor (`.h`):** Slouží jako deklarace. Definuje se zde samotné tělo třídy a hlavičky metod (bez jejich vnitřní logiky).
*   **Zdrojový soubor (`.cpp`):** Obsahuje samotnou logiku. Před názvem každé metody musí být uveden název třídy s dvojitou dvojtečkou (např. `BazovaTrida::vypisInfo()`), aby překladač věděl, kam metoda patří. Uvnitř `.h` souboru se čtyřtečka nikdy nepíše!

---

## 3. Bázová třída (Základní šablona / Rodič)

Bázová třída definuje společné vlastnosti pro všechny odvozené objekty. 

**V hlavičce (`.h`):**
*   Konstruktor (přijímá základní parametry, typicky jméno/označení).
*   Destruktor (musí mít klíčové slovo `virtual`, aby fungovalo správné smazání u potomků z paměti).
*   Gettery (vracejí chráněná data).
*   Obyčejné virtuální metody (zapisují se bez parametrů, např. `virtual void vypisInfo() const;`).

**V implementaci (`.cpp`):**
*   Úplně na začátku souboru (mimo metody) se musí inicializovat statické počítadlo: `int BazovaTrida::citac = 0;`.
*   V samotném konstruktoru se řeší zvýšení počítadla (`citac++`), v destruktoru jeho snížení (`citac--`).

---

## 4. Odvozené třídy (Dědičnost / Potomek)

Specializované implementace, které rozšiřují bázovou třídu.

**V hlavičce (`.h`):**
*   Dědičnost se zapisuje rovnou k definici třídy: `class OdvozenaTrida : public BazovaTrida`.
*   U metod, které se přepisují z bázové třídy, se na konec přidává klíčové slovo `override`.

**V implementaci (`.cpp`) - Konstruktor a předávání dat:**
*   Konstruktor potomka dostane parametry od uživatele. Část z nich (např. jméno) musí poslat rovnou "nahoru" rodiči do bázové třídy, a zbytek si uloží do svých vlastních proměnných.
    ```cpp
    // 'hodnotaProRodice' se jen předává dál (do BazovaTrida),
    // 'vlastniHodnota' je nová proměnná, kterou si ukládáme do atributu 'vlastniAtribut'.
    OdvozenaTrida::OdvozenaTrida(const std::string& hodnotaProRodice, double vlastniHodnota) 
        : BazovaTrida(hodnotaProRodice), vlastniAtribut(vlastniHodnota) { 
        // Tělo konstruktoru zůstává většinou prázdné
    }
    ```
*   Ve vlastní upravené metodě pro výpis je dobré zavolat nejprve výpis z bázové třídy (např. `BazovaTrida::vypisInfo();`) a pak vypsat specifika daného potomka.

---

## 5. Přetěžování operátorů

Umožňuje používat standardní matematické a logické znaky pro naše vlastní objekty.

**V hlavičce (`.h`):**
*   `bool operator==(const OdvozenaTrida& druhyObjekt) const;` (Porovnání - vrací ano/ne. Pozor na obě `const`!).
*   `OdvozenaTrida& operator+=(double novaHodnota);` (Přidání - upravuje aktuální objekt, vrací referenci).
*   `friend std::ostream& operator<<(std::ostream& os, const OdvozenaTrida& objektKterovyVypisuji);` (Výpis do konzole - `friend` umožňuje přístup k privátním datům).

**V implementaci (`.cpp`):**
*   **Porovnání (`==`):** Má předponu třídy (čtyřtečku). Vrací výsledek porovnání s využitím ukazatele na sebe sama: 
    `return this->vlastniAtribut == druhyObjekt.vlastniAtribut;`
*   **Přidání (`+=`):** Má předponu třídy. Modifikuje vnitřní data a na konci vrací referenci na aktuální objekt: 
    `return *this;`
*   **Výpis (`<<`):** Píše se **bez** předpony třídy (bez čtyřtečky) a **bez** slova `friend`. Formátuje se klasicky přes proud: 
    `os << "Text " << objektKterovyVypisuji.vlastniAtribut;` a na konci vrací proud: `return os;`

---

## 6. Algoritmy (Volné funkce v mainu)

Algoritmy se píšou jako běžné funkce nad `main()` a jako parametr obvykle dostávají ukazatel na bázovou třídu, aby fungovaly pro všechny potomky díky polymorfismu.

*   Funkce přijímá ukazatel: `typNavratu nazevAlgoritmu(BazovaTrida* parametrBaze)`
*   Uvnitř funkce se přes parametr přistupuje k datům objektu (pomocí šipky `->`, např. `parametrBaze->getData()`).
*   **Pozor u mazání:** Pokud algoritmus upravuje strukturu dat (např. maže prvky a nevrací hodnotu, tedy je typu `void`), nesmí se jeho volání dávat přímo do `std::cout <<`. Zavolá se na samostatném řádku a teprve poté se vypíše nový stav objektu.

---

## 7. Hlavní program (`main.cpp`) a bezpečný průchod

Celá funkce `main` musí mít chronologickou strukturu, aby správně otestovala polymorfismus a zamezila úniku paměti. 

1.  **Ověření výchozího stavu:** Výpis statického čítače (měl by být 0).
2.  **Vytvoření struktury a alokace:** Vytvoří se pole/kolekce ukazatelů na bázovou třídu. Přes klíčové slovo `new` se do něj vloží různé odvozené objekty.
3.  **Naplnění daty:** Přes indexy a operátor šipky se objektům přidají data (např. `kolekce[0]->pridejData(...)`).
4.  **Polymorfní průchod (Hlavní cyklus):** Klasický cyklus projde celou kolekci a zavolá virtuální metody. Díky polymorfismu se každý prvek zachová podle svého skutečného typu (potomka), ačkoliv cyklus vidí jen bázovou třídu.
5.  **Spuštění algoritmů:** Zavolají se externí funkce (algoritmy) a předá se jim konkrétní prvek z kolekce.
6.  **Test operátorů:** Vytvoří se nezávislé lokální proměnné zabalené v bloku `{ ... }` na zásobníku, provede se test operátorů `+=` a `==` a jejich výpis `<<`.
7.  **Úklid paměti (`delete`):** Než program skončí, musí se projít alokovaná kolekce a ručně smazat objekty (volání `delete`).
8.  **Závěrečná kontrola:** Výpis čítače musí ukázat opět 0, což potvrzuje, že proběhly všechny destruktory.
