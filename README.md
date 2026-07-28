# 📚 C++ Tahák a Architektura Projektu

Tento dokument slouží jako průvodce zkouškovým projektem. Kopíruje strukturu zadání a vysvětluje klíčové koncepty, které je nutné u zkoušky dodržet a chápat.

## 1. Pravidla zápisu: Hlavička (.h) vs. Logika (.cpp)

| Vlastnost | Hlavičkový soubor (`.h`) | Zdrojový soubor (`.cpp`) |
| :--- | :--- | :--- |
| **Účel** | Deklarace (co třída umí). "Jídelní lístek". | Implementace (jak to dělá). Samotný kód. |
| **Základ** | Vždy začíná direktivou `#pragma once`. | Vždy includuje svou hlavičku `#include "Trida.h"`. |
| **Tělo** | `class Nazev { ... };` (pozor na středník!). | Metody mají předponu: `void Nazev::metoda()`. |
| **Klíčová slova** | Píše se sem `virtual`, `override`, `friend`, `static`. | Tato slova se sem už **nepíší**! |

---

## 2. Bázová třída (Abstraktní)
Představuje obecný základ programu (např. `Senzor`). Nelze z ní vytvořit přímý objekt, slouží pouze jako šablona pro potomky.

* **Zapouzdření (`protected`):** Proměnné (jako označení nebo vektor s historií) musí být `protected`, nikoliv `private`, aby k nim měly odvozené třídy přímý přístup.
* **Statické členy (`static`):** Proměnná (např. čítač) existuje v paměti jen jednou a všechny objekty ji sdílejí. V hlavičce se pouze deklaruje, fyzicky se musí inicializovat v `.cpp` souboru mimo jakoukoliv funkci (zápisem `int Senzor::citac = 0;`).
* **Virtuální metody (`virtual`):** Umožňují polymorfismus. Překladač díky nim za běhu pozná, že má u potomka zavolat jeho upravenou verzi metody a ne tu základní.
* **Čistě virtuální metoda (`= 0`):** Píše se jako `virtual void analyzuj() const = 0;`. Definuje, že třída je abstraktní a každý potomek tuto metodu **musí** povinně naprogramovat.
* **Virtuální destruktor:** Bázová třída s dědičností musí mít `virtual ~Senzor();`, jinak při mazání pole neproběhne uvolnění dat potomků a vznikne únik paměti (memory leak).

---

## 3. Odvozené třídy (Dědičnost)
Konkrétní specializované implementace (např. `SenzorTeploty`, `SenzorVlhkosti`).

* **Zápis dědičnosti:** Píše se rovnou za název třídy v hlavičce: `class SenzorTeploty : public Senzor`.
* **Konstruktor:** Musí v inicializační listině vždy nejdříve předat parametry konstruktoru rodiče. Příklad v `.cpp`: `SenzorTeploty(string id) : Senzor(id) {}`.
* **Přepisování metod (`override`):** Pokud třída upravuje chování virtuální metody rodiče, doplňuje se na konec deklarace v hlavičce slovo `override`. Překladač díky tomu ohlídá překlepy.

---

## 4. Přetěžování operátorů
Učí jazyk C++ pracovat s našimi vlastními objekty pomocí běžných znaků.

* **Operátor `==` (Porovnání):** Patří do třídy (člen). Vrací `bool` a nemění objekt (na konci má `const`). Porovnává vnitřní data s daty druhého objektu předaného přes referenci.
* **Operátor `+=` (Změna objektu):** Patří do třídy (člen). Vrací referenci na sebe sama (`return *this;`), aby šly operace v kódu řetězit.
* **Operátor `<<` (Výpis do proudu):** **Nepatří** třídě, protože levá strana (např. `std::cout`) je typu `ostream`. Musí být deklarován jako `friend` uvnitř `.h` souboru. V `.cpp` se u něj nepíše čtyřtečka s názvem třídy. Vrací proud (`return os;`).

---

## 5. Algoritmy a Iterátory
Zásady pro bezpečnou práci s daty uvnitř kolekce (`std::vector`).

* **Reference (`&`):** Pokud má algoritmus upravovat původní data objektu, getter musí vracet přímý přístup do paměti, nikoliv kopii: `std::vector<double>& getHistorie()`.
* **Range-based for cyklus:** Zápis `for(double hodnota : data)` je bezpečný a nejrychlejší způsob, jak pouze přečíst celou historii od začátku do konce bez nutnosti hlídat indexy.
* **Mazání prvků (`.erase`):** Při mazání prvků během procházení se nesmí použít klasický for cyklus, přeskočily by se položky. Používá se iterátor (`auto it = data.begin()`). Pokud dojde ke smazání, metoda `.erase(it)` bezpečně zalepí díru a sama vrátí nový, správně posunutý iterátor.

---

## 6. Hlavní program a Správa paměti (`main.cpp`)
Propojení všech pilířů dohromady přes polymorfismus.

* **Vektor ukazatelů:** Vytváří se typem `std::vector<Senzor*> senzory;`. Uchovává pouze adresy, nikoliv celé objekty. Jen díky tomu do něj lze vkládat rozdílné potomky.
* **Dynamická alokace (`new`):** Objekty se vytvářejí dynamicky na haldě: `senzory.push_back(new SenzorTeploty(...));`.
* **Operátor šipky (`->`):** Při procházení vektoru ukazatelů se nepoužívá klasická tečka, ale šipka (např. `s->vypisInfo()`). Je to zkratka pro dereferenci adresy a zavolání metody zároveň.
* **Uvolnění paměti (`delete`):** Vektor ukazatelů sám o sobě vytvořené objekty nevymaže. Na konci programu musí být vždy cyklus, který zavolá `delete` pro každý jednotlivý prvek.
