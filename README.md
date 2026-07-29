# 📚 C++  Architektura, Operátory a Algoritmy

Tento dokument slouží jako komplexní průvodce pro zkouškový projekt. Shrnuje pravidla rozdělení kódu, syntaxi operátorů a bezpečnou práci s pamětí a vektory.

---

## 1. Pravidla zápisu: Hlavička (.h) vs. Logika (.cpp)

Při psaní objektově orientovaného kódu se třídy rozdělují do dvou souborů:
*   **Hlavičkový soubor (`.h`):** Slouží jako deklarace (co třída umí). Definuje se zde samotné tělo třídy a hlavičky metod.
*   **Zdrojový soubor (`.cpp`):** Obsahuje samotnou logiku a implementaci metod. Před názvem každé metody musí být uveden název třídy s dvojitou dvojtečkou (např. `Senzor::vypisInfo()`).

---

## 2. Bázová třída (Základní šablona)

Bázová třída definuje společné vlastnosti pro všechny odvozené objekty. 

**V hlavičce (`.h`):**
*   Tělo konstruktoru (přijímá parametr jména a případně další vlastní proměnné).
*   Destruktor (musí být `virtual`, aby fungovalo správné smazání u potomků).
*   Gettery (např. pro získání čítače nebo historie).
*   Obyčejné virtuální metody (zapisují se bez parametrů, např. `virtual void vypisInfo() const;`).

**V implementaci (`.cpp`):**
*   Úplně na začátku souboru (mimo metody) se musí implementovat/inicializovat statické počítadlo: `int Senzor::citac = 0;`.
*   V samotném konstruktoru se řeší zvýšení počítadla, v destruktoru jeho snížení.
*   Přidávací metoda na konec vektoru využívá `push_back(hodnota)`.
*   Přetížená verze přidávací metody (pro celý vektor hodnot) využívá `insert` a prochází vlastní proměnnou od začátku do konce.
*   Výpis počtu prvků historie se řeší přes metodu `size()`.

---

## 3. Odvozené třídy (Dědičnost)

Konkrétní specializované implementace, které rozšiřují bázovou třídu.

**V hlavičce (`.h`):**
*   Dědičnost se zapisuje rovnou k definici třídy: `class SpecifickySenzor : public Senzor`.
*   U metod, které se přepisují z bázové třídy, se na konec přidává klíčové slovo `override`.

**V implementaci (`.cpp`):**
*   Konstruktor musí nejprve zavolat bázovou třídu a předat jí název, a až poté inicializovat svou vlastní proměnnou: 
    `SpecifickySenzor(string nazev, double hodnota) : Senzor(nazev), vlastniPromenna(hodnota) {}`
*   Ve vlastní upravené metodě pro výpis je dobré zavolat nejprve výpis z bázové třídy (např. `Senzor::vypisInfo();`) a pak vypsat specifika daného potomka.
*   Při procházení historie se využívá range-based for cyklus: `for(typ kamUlozim : coProjizdim)`.

---

## 4. Přetěžování operátorů

Umožňuje používat standardní matematické a logické znaky pro naše vlastní objekty.

**V hlavičce (`.h`):**
*   `bool operator==(SenzorTeploty& A) const;` (Porovnání - vrací ano/ne).
*   `SenzorTeploty& operator+=(double hodnota);` (Přidání - upravuje objekt).
*   `friend std::ostream& operator<<(std::ostream& os, const SenzorTeploty& promena);` (Výpis do konzole).

**V implementaci (`.cpp`):**
*   **Porovnání (`==`):** Má předponu třídy. Vrací výsledek porovnání s využitím ukazatele na sebe sama: `return this->podminka == A.podminka;`
*   **Přidání (`+=`):** Má předponu třídy. Zavolá standardní přidávací metodu a na konci vrací referenci na aktuální objekt: `return *this;`
*   **Výpis (`<<`):** Píše se **bez** předpony třídy a **bez** slova friend. Formátuje se klasicky přes proud: `os << ... << promena.jmeno << ...`. Na konci vrací proud: `return os;`

---

## 5. Knihovna `std::vector` (Základní příkazy)

Přehled metod pro práci s kolekcemi (např. historií měření):
*   `size()` - Vrátí celkový počet prvků uvnitř vektoru.
*   `begin()` - Vrátí iterátor (ukazatel) na úplně první prvek.
*   `end()` - Vrátí iterátor na konec (za poslední prvek).
*   `push_back(X)` - Vloží hodnotu X na úplný konec vektoru.
*   `insert(...)` - Vloží více prvků na specifikované místo.
*   `erase(it)` - Bezpečně smaže prvek na pozici iterátoru a posune zbytek dat.

---

## 6. Algoritmy

Algoritmy se volají z mainu přímo na konkrétní prvek vektoru: `nazevVektoru[index]->funkce()`.

**Algoritmus 1: Procházení (např. hledání nejdelší řady)**
*   Funkce přijímá ukazatel: `void algoritmus1(Senzor* s)`
*   Využívá klasický for cyklus `for(double hodnota : s->getHistorie())` a podmínkami porovnává hodnoty.

**Algoritmus 2: Podmíněné mazání (`erase`)**
1. Načtení dat před mazáním: `nazev[0]->getHistorie().size()` a výpis.
2. Ve funkci se musí vytvořit přímá reference na vektor, **nikoliv kopie**! 
   `std::vector<double>& vektor = s->getHistorie();`
3. Cyklus prochází vektor přes iterátor:
   `for(auto it = vektor.begin(); it != vektor.end(); /* bez posunu */)`
4. Uvnitř se řeší dereference `*it` a samotné mazání:
   ```cpp
   if(*it < 0 && *it > -5) {
       it = vektor.erase(it); // Smaže a vrátí novou pozici
   } else {
       ++it; // Posune se dál, jen pokud se nemazalo
   }

## 7. Hlavní program (`main.cpp`) a Projetí mainu

Celá funkce `main` musí mít jasnou chronologickou strukturu, aby správně otestovala všechny vlastnosti (polymorfismus, operátory, algoritmy) a zamezila úniku paměti. 

**Chronologický postup (kostra mainu):**

1.  **Ověření výchozího stavu:** Výpis statického čítače (měl by být 0).
2.  **Vytvoření struktury a alokace:** Vytvoří se `std::vector<Senzor*>`, do kterého se přes `new` vloží různé odvozené objekty.
3.  **Naplnění daty:** Přes indexy a operátor šipky se zavolá vkládací metoda (např. `vektor[0]->pridejHodnotu(...)`).
4.  **Polymorfní průchod (Hlavní cyklus):** Klasický for cyklus projde vektor a zavolá virtuální metody. Díky polymorfismu se každý prvek zachová podle svého skutečného typu.
5.  **Spuštění algoritmů:** Zavolají se externí funkce (např. pro mazání) a předá se jim konkrétní ukazatel z vektoru (např. `algoritmus(vektor[0])`).
6.  **Test operátorů:** Vytvoří se lokální proměnné (na zásobníku), provede se test operátorů `+=` a `==` a jejich výpis `<<`.
7.  **Úklid paměti (`delete`):** Než program skončí, musí se projít původní vektor ukazatelů a ručně smazat alokovaná data.
8.  **Závěrečná kontrola:** Výpis čítače musí ukázat opět 0.
