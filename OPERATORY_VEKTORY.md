# ⚡ Vektory a Operátory

## 1. Práce s vektory (`std::vector`)
Základní operace pro rychlé použití v algoritmech a metodách:

*   `vektor.push_back(hodnota)` – Vloží prvek úplně na konec.
*   `vektor.insert(vektor.end(), jiny.begin(), jiny.end())` – Vloží celý jiný vektor na konec aktuálního.
*   `vektor.size()` – Vrátí aktuální počet prvků (hodí se pro výpisy a cykly).
*   `vektor.back()` – Vrátí hodnotu úplně posledního prvku ve vektoru. *(Pozor: Před zavoláním by měl mít vektor velikost alespoň 1, jinak program spadne).*
*   `vektor.erase(iterator)` – Smaže prvek, na který ukazuje iterátor (typicky se používá uvnitř cyklu).
*   `vektor.empty()` – Vrátí `true` (1), pokud je vektor prázdný. Nutné používat jako pojistku na začátku algoritmů nebo před čtením prvků (např. před `.back()`).

---
## 2. Přetěžování operátorů (Přehled a .h deklarace)

| Operátor | K čemu slouží v praxi | Pravidla pro `const` (Deklarace v `.h`) |
| :--- | :--- | :--- |
| `==` | **Porovnání na shodu:** Určuje, podle čeho poznáme, že jsou dva objekty totožné. | Čte obě strany, `const` je všude:<br>`bool operator==(const Trida& pravy) const;` |
| `<` | **Menší než:** Definuje, který objekt je menší (nutné pro `std::sort`). | Čte obě strany, `const` je všude:<br>`bool operator<(const Trida& pravy) const;` |
| `+=` | **Přidání / Sloučení:** Přičte hodnotu k aktuálnímu objektu. Vrací referenci na `*this` pro řetězení. | Mění sebe, čte pravou stranu:<br>`Trida& operator+=(const Trida& pravy);` (nebo např. `double v`) |
| `<<` | **Výpis do proudu:** Učí objekt, jak se má sám vypsat přes `std::cout`. | Externí funkce (`friend`), jen čte objekt:<br>`friend std::ostream& operator<<(std::ostream& os, const Trida& obj);` |
| `()` | **Funktor:** Dovoluje zavolat objekt jako funkci (např. `mujObjekt(15)`). | Záleží na tom, zda volání mění stav objektu. |
| `=` | **Přiřazení:** Řeší hlubokou kopii při `A = B` (nutné, pokud jsou v paměti ukazatele). | Mění levou stranu, čte pravou:<br>`Trida& operator=(const Trida& pravy);` |
| `[]` | **Indexace:** Přístup k datům jako k poli (např. `objekt[3]`). | Vrací referenci, často má verzi s `const` i bez. |

---
## 3. Implementace a použití v praxi

| Operátor | Deklarace v `.h` | Implementace v `.cpp` | Použití v `main()` |
| :---: | :--- | :--- | :--- |
| **`==`** | `bool operator==(const Stanice& pravy) const;` | `bool Stanice::operator==(const Stanice& pravy) const {`<br>&nbsp;&nbsp;`return this->smog == pravy.smog;`<br>`}` | `if (s1 == s2) {`<br>&nbsp;&nbsp;`std::cout << "Shoda";`<br>`}` |
| **`+=`** | `Stanice& operator+=(double novaTeplota);` | `Stanice& Stanice::operator+=(double novaTeplota) {`<br>&nbsp;&nbsp;`this->historie.push_back(novaTeplota);`<br>&nbsp;&nbsp;`return *this;`<br>`}` | `Stanice s("Brno");`<br>`s += 25.5;`<br>`s += -3.0;` |
| **`<<`** | `friend std::ostream& operator<<(std::ostream& os, const Stanice& obj);` | `std::ostream& operator<<(std::ostream& os, const Stanice& obj) {`<br>&nbsp;&nbsp;`os << "Stanice: " << obj.oznaceni;`<br>&nbsp;&nbsp;`return os;`<br>`}` | `Stanice s("Praha");`<br>`std::cout << s;` |
| **`<`** | `bool operator<(const Stanice& pravy) const;` | `bool Stanice::operator<(const Stanice& pravy) const {`<br>&nbsp;&nbsp;`return this->smog < pravy.smog;`<br>`}` | `if (s1 < s2) {`<br>&nbsp;&nbsp;`std::cout << "S1 je cistsi";`<br>`}` |
| **`=`** | `Stanice& operator=(const Stanice& pravy);` | `Stanice& Stanice::operator=(const Stanice& pravy) {`<br>&nbsp;&nbsp;`if (this != &pravy) {`<br>&nbsp;&nbsp;&nbsp;&nbsp;`this->smog = pravy.smog;`<br>&nbsp;&nbsp;`<br>&nbsp;&nbsp;`return *this;`<br>`}` | `Stanice s1("Brno");`<br>`Stanice s2("Praha");`<br>`s1 = s2;` |
| **`[]`** | `double& operator[](int index);` | `double& Stanice::operator[](int index) {`<br>&nbsp;&nbsp;`return this->historie[index];`<br>`}` | `Stanice s("Ostrava");`<br>`s += 25.5;`<br>`double t = s[0];` |
| **`()`** | `void operator()(double zmena);` | `void Stanice::operator()(double zmena) {`<br>&nbsp;&nbsp;`this->smog += zmena;`<br>`}` | `Stanice s("Opava");`<br>`s(1.5);` |

---
## Funktor && Indexor

**1. Operátor volání funkce `()` (Funktor)**
*   **`.h`:** `int operator()(double zmena);`[cite: 1]
*   **`.cpp`:** `int Trida::operator()(double zmena) { /* vnitřní logika a výpočet */ return vysledek; }`[cite: 1]
*   **`main()`:** `int vysledek = objekt(15.5);` (Zavolá se jako funkce přímo na objektu)[cite: 1]

**2. Operátor indexace `[]`**
*   **`.h`:** `double& operator[](int index);`[cite: 1]
*   **`.cpp`:** `double& Trida::operator[](int index) { return this->historie[index]; }`[cite: 1]
*   **`main()`:** `double t = objekt[0];` (Čtení) nebo `objekt[0] = 50.0;` (Zápis)[cite: 1]

**3. Inkrementace `++` (Prefix a Postfix)**
*   **`.h`:** `Trida& operator++();` (Prefix) a `Trida operator++(int);` (Postfix)[cite: 1]
*   **`.cpp`:**
    ```cpp
    Trida& Trida::operator++() {
        this->hodnota += 1.0;
        return *this;
    }
    ```[cite: 1]
*   **`main()`:** `++objekt;` nebo `objekt++;`[cite: 1]

---
