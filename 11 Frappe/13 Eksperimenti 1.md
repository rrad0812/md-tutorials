# Frappe Framework tutorijal

[Sadržaj][00]

## 13 Eksperimenti - I

Sada je mnogo korisnije da krenemo "odozgo nadole":

* prvo napišemo kod kao Frappe programeri,
* posmatramo šta se dešava,
* pa tek onda, ako nešto nije jasno, vratimo se u izvorni kod Frappea.

### Developer mode

Imamo nekoliko mogućnosti za praćenje izvršavanja:

1. **`print()` u konzoli** – radi samo ako je proces koji izvršava kod pokrenut iz terminala i stdout nije preusmeren. Nije najpouzdanije.
2. **`frappe.logger()`** – profesionalniji pristup, zapis ide u log fajlove.
3. **`frappe.log_error()`** – koristan za pojedinačne događaje, ali nije namenjen za ovakvo praćenje.

Ja bih koristio **`frappe.logger()`**, jer je to način koji ćeš koristiti i kasnije u realnim aplikacijama.

Međutim, tu postoji jedna važna stvar kod Frappe-a: "developer mode" nije osobina Bench-a, već pojedinačnog Site-a.

Dakle, ne prebacuješ bench u developer mode, već `site1.local`.

Najpre proveri trenutno stanje:

```bash
cd ~/frappe-bench

bench --site site1.local console
```

U Python konzoli:

```python
frappe.conf.developer_mode
```

Ako dobiješ `0` ili `False` onda developer mod nije uključen.

Izađi iz konzole i zatim uključi developer mode:

```bash
bench --site site1.local set-config developer_mode 1
```

To će upisati u:

```sh
sites/site1.local/site_config.json
```

stavku:

```json
{
    "developer_mode": 1
}
```

Posle toga je dobro da restartuješ procese Bench-a.

Ako koristiš:

```bash
bench start
```

dovoljno je da prekineš (`Ctrl+C`) i ponovo pokreneš:

```bash
bench start
```

Sada možemo da radimo:

* kreiranje i izmenu Custom DocType-ova i drugih razvojnih objekata na očekivan
  način,
* izvoz promena u aplikaciju kada budemo imali svoju aplikaciju (`bench
  export-fixtures` i druge razvojne tokove),
* razvoj koji je usklađen sa načinom na koji Frappe očekuje da se razvijaju
  aplikacije.

### Nova aplikacija

Ja ne bih odmah pravio DocType. Pre toga bih voleo da upoznaš strukturu aplikacije koju je Bench napravio.

#### Struktura aplikacije

```sh
tree -L 2 ~/frappe-bench/apps/frappe_lab
```

```sh
/home/radosav/frappe-bench/apps/frappe_lab
├── frappe_lab
│   ├── config
│   ├── frappe_lab
│   ├── hooks.py
│   ├── __init__.py
│   ├── modules.txt
│   ├── patches
│   ├── patches.txt
│   ├── public
│   ├── __pycache__
│   ├── templates
│   └── www
├── license.txt
├── pyproject.toml
└── README.md

9 directories, 7 files
```

Ovo je sasvim standardna struktura. Hajde da označimo šta nam je **bitno sada**, a šta **kasnije**.

```sh
frappe_lab/
├── pyproject.toml        ⭐
├── README.md             😴
├── license.txt           😴
└── frappe_lab/
    ├── __init__.py       ⭐
    ├── hooks.py          ⭐⭐⭐⭐⭐
    ├── modules.txt       ⭐⭐⭐
    ├── config/           ⭐⭐
    ├── frappe_lab/       ⭐⭐⭐⭐
    ├── patches/          ⏳
    ├── patches.txt       ⏳
    ├── public/           ⏳
    ├── templates/        ⏳
    └── www/              ⏳
```

Za sada su nam važna samo četiri mesta

* **`hooks.py`**

  Srce aplikacije. Ovde se registruje skoro sve:
  
  * `doc_events`
  * scheduler
  * override-i
  * fixtures
  * Jinja
  * itd.
  
  Tu ćemo provesti dosta vremena.

* **`modules.txt`**

  Kaže Frappe-u koje module aplikacija ima. Kasnije će ovde stajati, recimo:
  
  ```sh
  Frappe Lab
  ```
  
  i taj modul će se pojaviti na Desk-u.

* **`frappe_lab/frappe_lab/`**

  Ovo je glavni Python paket tvoje aplikacije. Ovde će nastajati:
  
  * DocType-ovi
  * API funkcije
  * Business logika
  * itd.

* **`config/`**

  Za sada samo zapamti da je vezan za Desk/module konfiguraciju.
  Ne diramo još.

Primećuješ li nešto zanimljivo ovde?

```sh
apps/
└── frappe_lab/
    └── frappe_lab/
        └── frappe_lab/
```

Dakle imamo **tri puta** `frappe_lab`.

```sh
~/frappe-bench/apps/
└── frappe_lab/          ← 1. Git projekat (repozitorijum aplikacije)
    └── frappe_lab/      ← 2. Glavni Python paket aplikacije
        └── frappe_lab/  ← 3. Modul (business module) u Frappe-u
```

* **`apps/frappe_lab`**

  Ovo je običan Git projekat.
  
  Tu su:
  
  * `README.md`
  * `pyproject.toml`
  * `.git`
  * itd.
  
  To Python uopšte ne zanima.

* **`apps/frappe_lab/frappe_lab`**

  Ovo je **Python paket**.
  
  Zato tu postoji:
  
  ```sh
  __init__.py
  hooks.py
  modules.txt
  ...
  ```
  
  Kada Python izvrši:
  
  ```python
  import frappe_lab
  ```
  
  uvozi upravo ovaj paket.

* **`apps/frappe_lab/frappe_lab/frappe_lab`**

  Ovo nije novi Python paket zbog Pythona, već **Frappe modul**.
  
  U njemu će kasnije biti nešto poput:
  
  ```sh
  doctype/
  page/
  report/
  dashboard/
  ...
  ```
  
  Drugim rečima, ovo predstavlja jedan **Module Def** u Frappe-u.
  
  Kada jednog dana napraviš još jedan modul, dobio bi:
  
  ```sh
  frappe_lab/
      accounting/
      crm/
      warehouse/
  ```
  
  Svi oni bi bili moduli iste aplikacije.

  **Zašto je ovo urađeno ovako?**

  Zato što jedna aplikacija može imati više Frappe modula.
  
  Na primer ERPNext je jedna aplikacija, ali ima module:
  
  * Accounts
  * Buying
  * Selling
  * HR
  * CRM
  * Manufacturing
  * ...
  
  Svaki od njih izgleda upravo kao ovaj tvoj treći direktorijum.

#### Frappe modul

U Frappe svetu reč **module** se koristi za dve različite stvari, pa ljudi često pomešaju.

Da razdvojimo:

```sh
Aplikacija (App)
    |
    +-- Modul (Module)
            |
            +-- DocType
            +-- Report
            +-- Page
            +-- ...
```

**App (aplikacija)**:

To je ono što smo upravo napravili:

```text
frappe_lab
```

App je Python/Git projekat.

Pravi se:

```sh
bench new-app frappe_lab
```

App može da se instalira na site:

```bash
bench --site site1.local install-app frappe_lab
```

Jedan site može imati:

```sh
frappe
erpnext
frappe_lab
moja_druga_app
```

više aplikacija.

**Module (Frappe modul)**:

Modul je **organizacija sadržaja unutar aplikacije**.

Na primer ERPNext:

```sh
erpnext (App)
    |
    +-- Accounts (Module)
    |
    +-- Selling (Module)
    |
    +-- Buying (Module)
    |
    +-- Stock (Module)
```

Dakle: ERPNext nije 10 aplikacija. To je **jedna aplikacija sa mnogo modula**.

##### Kako se pravi novi modul
  
Modul se pravi implicitno kroz izgradnu DocTypea!

* **App** = naša aplikacija za učenje
* **Module** = oblast unutar aplikacije
* **DocType** = konkretan poslovni objekat

Samo Frappe koristi istu reč za oba koncepta, Frappe modul i Python modul.
  
**Aplikacija**:

Aplikacija dobija vezu kroz sadržaj koji napraviš.

Na primer:

```sh
frappe_lab (app)
|
+-- frappe_lab (python paket)
    |
    +-- frappe_lab 
        |
        +-- doctype
            |
            +-- Customer (module = frappe_lab)
```

Znači:

```sh
DocType
    |
    +-- module: frappe_lab
```

govori Frappe-u: "Ovaj DocType pripada modulu frappe_lab."

##### `modules.txt`

U tvojoj aplikaciji imaš:

```sh
frappe_lab/modules.txt
```

On sadrži listu modula koje aplikacija obezbeđuje.

Na primer:

```sh
Frappe Lab
```

To nije Python import lista. To je više metadata za Frappe.

##### Praktično za nas

Mi ćemo verovatno uraditi ovako:

* Imamo app:

  ```txt
  frappe_lab
  ```

* Kreiramo DocType:

  ```txt
  Book
  ```

  i on će imati:
  
  ```json
  {
      "module": "frappe_lab"
  }
  ```

* Taj DocType se čuva u aplikaciji:

  ```sh
  frappe_lab/
      frappe_lab/
          frappe_lab/
              doctype/
                  book/
  ```

```sh
cat ~/frappe-bench/apps/frappe_lab/frappe_lab/modules.txt
```

```sh
Frappe Lab
```

##### Lista relavantni modula i aplikacija

```sh
bench --site site1.local console
Apps in this namespace:
frappe, frappe_lab
```

```py
In [1]: frappe.get_all("Module Def", fields=["name", "app_name"])
Out[1]: 
[{'name': 'Frappe Lab', 'app_name': 'frappe_lab'},
 {'name': 'Automation', 'app_name': 'frappe'},
 {'name': 'Social', 'app_name': 'frappe'},
 {'name': 'Contacts', 'app_name': 'frappe'},
 {'name': 'Printing', 'app_name': 'frappe'},
 {'name': 'Integrations', 'app_name': 'frappe'},
 {'name': 'Desk', 'app_name': 'frappe'},
 {'name': 'Geo', 'app_name': 'frappe'},
 {'name': 'Custom', 'app_name': 'frappe'},
 {'name': 'Email', 'app_name': 'frappe'},
 {'name': 'Workflow', 'app_name': 'frappe'},
 {'name': 'Website', 'app_name': 'frappe'},
 {'name': 'Core', 'app_name': 'frappe'}]
```

Dakle, Module Def je logička organizacija funkcionalnosti unutar aplikacije.
Ona služi Frappe-u i korisniku.

Na primer:

```txt
ERPNext (app)

Accounts (Module Def)
    Journal Entry 
    Payment Entry
    GL Entry

Selling (Module Def)
    Sales Order
    Sales Invoice
    Customer

Buying (Module Def)
    Purchase Order
    Supplier
```

Korisnik ne razmišlja: "Ovo je u aplikaciji ERPNext." Nego razmišlja: "Radim u modulu Selling." To je organizacija **poslovnih funkcionalnosti**.

I pogledaj kako se lepo slažu podaci koje si izvukao:

| Module Def | App Name |
| ------ | --- |
| Frappe Lab | frappe_lab |
| Core | frappe |
| Desk | frappe |
| Email | frappe |
| Website | frappe |

Svaki Module Def zna:

* kome pripada (`app_name`)
* kako se zove (`name`)

#### Proverimo Module Def

Pošto u `modules.txt` imaš:

```text
Frappe Lab
```

proverimo da li postoji i na sajtu (što si već potvrdio):

```python
frappe.get_all("Module Def", fields=["name", "app_name"])
```

Postoji.

To znači da možemo da koristimo modul **Frappe Lab**.

#### Napravi prvi DocType

Uradi u Desk-u:

***Developer → DocType → New***

Popuni:

* **Module:** `Frappe Lab` -  Da bi DocType bio u tom modulu
* **Name:** `Book`
* **Custom:** ❌ (isključeno)
* **Is Submittable:** ❌
* **Is Single:** ❌

Dodaj samo dva polja:

| Label | Type     |
| ----- | -------- |
| Title | Data     |
| Price | Currency |

Sačuvaj.

```sh
tree -L 5 ~/frappe-bench/apps/frappe_lab/frappe_lab
```

```sh
/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab
├── config
│   └── __init__.py
├── frappe_lab
│   ├── doctype
│   │   ├── book
│   │   │   ├── book.js
│   │   │   ├── book.json
│   │   │   ├── book.py
│   │   │   ├── __init__.py
│   │   │   ├── __pycache__
│   │   │   │   ├── book.cpython-312.pyc
│   │   │   │   └── __init__.cpython-312.pyc
│   │   │   └── test_book.py
│   │   ├── __init__.py
│   │   └── __pycache__
│   │       └── __init__.cpython-312.pyc
│   ├── __init__.py
│   └── __pycache__
│       └── __init__.cpython-312.pyc
├── hooks.py
├── __init__.py
├── modules.txt
├── patches
│   └── __init__.py
├── patches.txt
├── public
│   ├── css
│   └── js
├── __pycache__
│   ├── hooks.cpython-312.pyc
│   └── __init__.cpython-312.pyc
├── templates
│   ├── includes
│   ├── __init__.py
│   └── pages
│       └── __init__.py
└── www

17 directories, 21 files
```

Sada smo konačno na "živom" Frappe-u. I odmah možeš da primetiš jednu vrlo zanimljivu stvar. Ranije smo pričali o `Document` klasi, a sada vidi šta je Frappe napravio:

```text
book/
├── book.json
├── book.py
├── book.js
└── test_book.py
```

Svaki od ovih fajlova ima svoju ulogu.

* `book.json` → **model** (definicija DocType-a: polja, dozvole, opcije...)
* `book.py` → **server-side kontroler** (naslednik `Document`)
* `book.js` → **client-side logika** (forma u browseru)
* `test_book.py` → testovi

Već sada vidiš kako se spajaju baza, Python i JavaScript.

### Prvi mali eksperiment

```bash
nano ~/frappe-bench/apps/frappe_lab/frappe_lab/frappe_lab/doctype/book/book.py
```

```py
# Copyright (c) 2026, rrad and contributors
# For license information, please see license.txt

# import frappe
from frappe.model.document import Document


class Book(Document):
        pass
```

Pogledaj ovo:

```python
from frappe.model.document import Document

class Book(Document):
    pass
```

Drugim rečima:

```python
doc = frappe.new_doc("Book")
```

napraviće:

```python
Book(Document)
```

a kada pozoveš:

```python
doc.insert()
```

izvršava se isti `Document.insert()` koji smo detaljno analizirali, samo što će usput pozivati metode iz tvoje klase `Book`.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
