# ⚡ Vektory a Operátory

## 1. Práce s vektory (`std::vector`)
Základní operace s vektorem pro rychlé použití v algoritmech a metodách:

* **`vektor.push_back(hodnota)`** – Vloží jeden prvek úplně na konec.
* **`vektor.insert(vektor.end(), jiny.begin(), jiny.end())`** – Vloží celý jiný vektor na konec aktuálního.
* **`vektor.size()`** – Vrátí aktuální počet prvků (často se hodí pro výpisy a cykly).
* **`vektor.back()`** – Vrátí hodnotu úplně **posledního prvku** ve vektoru. *(Pozor: před zavoláním `back()` by měl mít vektor velikost alespoň 1, jinak program spadne).*
* **`vektor.erase(iterator)`** – Smaže prvek, na který ukazuje iterátor (typicky se používá uvnitř cyklu s iterátory).

---

## 2. Přetěžování operátorů (Slovníček)
Co jednotlivé operátory dělají a jak u nich funguje `const`:

| Operátor | K čemu slouží v praxi | Pravidla pro `const` (Deklarace v `.h`) |
| :--- | :--- | :--- |
| **`==`** | **Porovnání na shodu:** Určuje, podle čeho poznáme, že jsou dva objekty totožné. | Čte obě strany. `const` je všude:<br>`bool operator==(const Trida& pravy) const;` |
| **`<`** | **Menší než / Řazení:** Definuje, který objekt je menší (nutné pro `std::sort`). | Čte obě strany. `const` je všude:<br>`bool operator<(const Trida& pravy) const;` |
| **`+=`** | **Přidání / Sloučení:** Přičte hodnotu k aktuálnímu objektu. Vrací referenci na sebe `*this`, aby šlo řetězit. | Mění sebe, čte pravou stranu:<br>`Trida& operator+=(const Trida& pravy);` (nebo např. `double v`) |
| **`<<`** | **Výpis do proudu:** Učí objekt, jak se má sám vypsat přes `std::cout`. | Externí funkce (friend), jen čte objekt:<br>`friend std::ostream& operator<<(std::ostream& os, const Trida& obj);` |
| **`()`** | **Funktor:** Dovoluje zavolat samotný objekt, jako by to byla funkce (např. `mujObjekt(15)`). | Dle toho, zda volání mění stav objektu. |
| **`=`** | **Přiřazení:** Říká, co se stane při `A = B` (nutné řešit tzv. hlubokou kopii, pokud máme v paměti ukazatele). | Mění levou stranu, čte pravou:<br>`Trida& operator=(const Trida& pravy);` |
| **`[]`** | **Indexace:** Přístup k datům objektu jako k poli (např. `mojeTrida[3]`). | Vrací referenci, často má 2 verze (s `const` i bez). |
