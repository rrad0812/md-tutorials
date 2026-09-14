# Frappe Framework tutorijal

[Sadržaj][00]

## 14 Eksperimenti - II

### Prvi pravi eksperiment

Zameni `pass` ovim:

```python
from frappe.model.document import Document

class Book(Document):

    def before_insert(self):
        print(">>> before_insert")

    def before_validate(self):
        print(">>> before_validate")

    def validate(self):
        print(">>> validate")

    def before_save(self):
        print(">>> before_save")

    def after_insert(self):
        print(">>> after_insert")

    def on_update(self):
        print(">>> on_update")
```

Zašto baš ovih šest? Zato što smo ih sreli dok smo čitali `Document.insert()`. Sada ćemo proveriti da li se naše razumevanje poklapa sa stvarnim izvršavanjem.

- Sačuvaj izmene u `book.py`.

- U browseru otvori Book i napravi jedan novi zapis.

  - Title: `Frappe Internals`
  - Price: `100`

- Posmatraj terminal u kome radi `bench start`.

  ```py
  before_insert
  before_validate
  validate
  before_save
  after_insert
  on_update
  ```

#### Redosled pojavljivanja obrađivača događaja

Ovo nam govori dve važne stvari.

1. `book.py` je ispravno učitan.
2. Redosled koji smo očekivali se zaista dešava.

To je upravo ono što smo želeli da vidimo.

Pogledaj redosled:

```text
  before_insert
  before_validate
  validate
  before_save
  after_insert
  on_update
```

Kada vidiš u izvornom kodu:

```python
self.run_method("before_insert")
...
self.run_before_save_methods()
...
self.db_insert()
...
self.run_method("after_insert")
...
self.run_post_save_methods()
```

ti sada znaš šta će izaći na ekranu.

#### `on_update`

Da li primećuješ nešto "čudno"? Zašto se `on_update` poziva odmah posle `after_insert`? Na prvi pogled čovek bi pomislio: "Pa nisam radio update, radio sam insert."

Ali ipak se `on_update` izvršava.

To nije greška. To je dizajn Frappe-a. Iako postoji samo:

```sql
INSERT INTO tabBook (...)
VALUES (...)
```

Zašto se onda zove `on_update`?

Sećaš se da smo čitali nešto ovako (parafraziram):

```python
insert()
    ...
    db_insert()
    ...
    after_insert()
    ...
    run_post_save_methods()
```

A u `run_post_save_methods()` se nalazi poziv:

```python
self.run_method("on_update")
```

Dakle, Frappe kaže: "Dokument je sada uspešno sačuvan u bazi."

i tada poziva:

```python
on_update()
```

bez obzira da li je to bio:

- prvi `INSERT`, ili
- deseti `UPDATE`.

Zato što ogromnu većinu poslovne logike ne zanima kako je dokument stigao u trenutno stanje.

Na primer:

```python
def on_update(self):
    self.update_customer_balance()
    self.recalculate_totals()
    self.send_notifications()
```

Ta logika treba da se izvrši:

1. posle kreiranja dokumenta,
2. posle svake izmene.

Ne želiš da pišeš isto dva puta:

```python
after_insert()
    ...

on_update()
    ...
```

#### `after_insert`

On postoji baš za ono što se dešava samo jednom.

Na primer:

```python
def after_insert(self):
    create_default_tasks()
```

To ne želiš da radiš pri svakom snimanju.

### Drugi eksperiment

Predlažem da sledeći eksperiment bude da dodamo još nekoliko metoda:

```python
before_naming
autoname
before_validate
validate
before_save
before_insert
after_insert
on_update
on_change
```

i da vidimo ceo tok izvršavanja.

Proširi `Book` ovako:

```python
from frappe.model.document import Document


class Book(Document):

    def before_naming(self):
        print(">>> before_naming")

    def autoname(self):
        print(">>> autoname")

    def before_insert(self):
        print(">>> before_insert")

    def before_validate(self):
        print(">>> before_validate")

    def validate(self):
        print(">>> validate")

    def before_save(self):
        print(">>> before_save")

    def after_insert(self):
        print(">>> after_insert")

    def on_update(self):
        print(">>> on_update")

    def on_change(self):
        print(">>> on_change")
```

Zatim:

- Napravi *novi* - Book dokument (ne menjaj postojeći).
- Vidi redosled pojavljivnja obrađivača događaja

Posle toga, uradi još jedan eksperiment:

1. Otvori *isti* Book.
2. Promeni samo `Price`.
3. Klikni **Save**.

Uporedi redosled događaja sa prvim snimanjem.

Ako sve bude kako očekujem, dobićemo dva veoma lepa dijagrama:

- **Insert*- lifecycle
- **Update*- lifecycle

> [!Info]
>
> INSERT
>
> ```sh
> before_insert
> before_naming
> autoname
> before_validate
> validate
> before_save
> after_insert
> on_update
> on_change
> ```
>
> UPDATE
>
> ```sh
> before_validate
> validate
> before_save
> on_update
> on_change
> ```

Ono što si dokazao:

**INSERT**:

```text
before_insert
before_naming
autoname
before_validate
validate
before_save
------------------
INSERT u bazu
------------------
after_insert
on_update
on_change
```

**UPDATE**:

```text
before_validate
validate
before_save
------------------
UPDATE u bazu
------------------
on_update
on_change
```

To je praktično ceo životni ciklus dokumenta koji jedan Frappe programer koristi svakodnevno.

Svi ovi hendleri se izvršavaju na serveru.

Na primer:

```python
def on_update(self):
    self.recalculate_stock()
    self.create_gl_entries()
    self.update_customer_balance()
```

To je čista poslovna logika.

#### `on_change`

E tu dolazimo do zanimljivog pitanja.

Po čemu se razlikuju:

```python
on_update()
```

i

```python
on_change()
```

Kada si pogledao eksperiment, vidiš da se oba pozivaju.

Zašto onda postoje oba?

To je jedno od najčešćih pitanja kod Frappe početnika?

#### Hook

Dodaj u `~/frappe-bench/apps/frappe_lab/frappe_lab/hooks.py`:

```python
doc_events = {
    "Book": {
        "*": "frappe_lab.book_hooks.trace"
    }
}
```

Zatim napravi novi fajl, na primer `/home/radosav/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py`

sa sadržajem:

```python
def trace(doc, method):
    print(f"HOOK ---> {method}")
```

I onda ponovi oba eksperimenta:

- INSERT
- UPDATE

Sada ćemo videti kada se tačno poziva hook u odnosu na metode klase `Book`.

To znači:

```text
frappe_lab                 ← Python paket
    book_hooks.py          ← modul
        trace()            ← funkcija
```

što odgovara putanji:

```text
~/frappe-bench/apps/frappe_lab/frappe_lab/book_hooks.py
```

```sh
before_validate
validate
before_save
on_update
HOOK ---> on_update
on_change
```

E, sada smo dobili ono što smo hteli.

I pogledaj kako je lep rezultat:

```text
   before_validate
   validate
   before_save
   on_update
   HOOK ---> on_update
   on_change
```

Sada možemo da izvučemo nekoliko važnih zaključaka.

#### doc_events

`doc_events` nije zamena za metodu klase.

Imaš:

```python
class Book(Document):

    def on_update(self):
        print(">>> on_update")
```

i u `hooks.py`:

```python
doc_events = {
    "Book": {
        "on_update": "frappe_lab.book_hooks.trace"
    }
}
```

To su dve različite tačke proširenja.

Prvo se izvršava metoda dokumenta:

```text
on_update
```

a zatim spoljašnji hook:

```text
HOOK ---> on_update
```

**Zašto ovo postoji?**

- Zamisli da imaš standardni Frappe/ERPNext DocType:

  ```text
  Sales Invoice
  ```
  
  Ne želiš da menjaš:
  
  ```python
  erpnext/accounts/doctype/sales_invoice/sales_invoice.py
  ```
  
  jer će ti sledeći update pregaziti izmene.
  
  Umesto toga u svojoj aplikaciji kažeš:
  
  ```python
  doc_events = {
      "Sales Invoice": {
          "on_update": "my_app.invoice_hooks.after_update"
      }
  }
  ```
  
  i dodaš svoju logiku.
  
  To je jedan od glavnih razloga postojanja hook sistema.

- Primetio si ranije: "on_update je za sve"

I sada vidiš zašto.

I pri:

```text
INSERT
```

dobio si:

```text
after_insert
on_update
```

A pri:

```text
UPDATE
```

dobio si:

```text
on_update
```

Dakle `on_update` u Frappe terminologiji zapravo više znači: "dokument je uspešno sačuvan i sada mogu da reagujem" a ne strogo SQL UPDATE.

### Moje mišljenje o Frappeu

Frappe je pomalo "čudna zver".

Ljudi koji očekuju:

- Django
- Flask
- FastAPI

često ga ne vole.

Ljudi koji očekuju:

- Odoo
- Oracle APEX
- Power Apps
- Mendix

često ga obožavaju.

Zašto? Jer Frappe nije prvenstveno web framework. To je *application framework*- sa ugrađenim RAD (Rapid Application Development) alatima.

Zbog toga ima:

- ORM
- permissions
- workflow
- report engine
- print engine
- scheduler
- background jobs
- REST
- realtime
- Desk
- Website
- Jinja
- migrations
- metadata
- role system...

...što je zaista retko u jednom open-source projektu.

Njegova mana nije što je siromašan, nego što je *veliki*.

### Kada se koja vrsta događaja koristi

**Controller** koristiš kada pišeš logiku **svog DocType-a**.

```python
class Book(Document):
    def validate(self):
        ...
```

**doc_events** koristiš kada želiš da se "prikačiš" na:

- tuđi DocType
- ili svoj DocType bez menjanja controller-a.

  ```python
  doc_events = {
      "User": {
          "validate": "moja_app.user_hooks.validate"
      }
  }
  ```

To je cela filozofija.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
