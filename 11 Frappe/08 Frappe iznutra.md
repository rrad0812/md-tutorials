
# Frappe Framework tutorijal

[Sadržaj][00]

## 08 Frappe iznutra

```sh
grep "^class " ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```sh
class Document(BaseDocument):
```

To znači: "Postoje stvari koje važe za svaki dokument, i postoje stvari koje važe samo za spec. Frappe Document."

Drugim rečima:

```txt <- Document
```

Ovo je veoma uobičajen obrazac u velikim frejmvorcima.

Drugim rečima:
  
```txt
BaseDocument
    │
    ├── čuva podatke
    ├── ponaša se kao objekat
    └── pomoćne metode

Document
    │
    ├── ORM
    ├── SQL
    ├── Hooks
    ├── Permissions
    └── Business Logic
```

Zašto je ovo dobro projektovanje? Zamisli da sutra Frappe želi da uvede još jednu vrstu dokumenta.

Na primer:

```txt
BaseDocument
        │
   ┌────┴─────┐
   │          │
Document   VirtualDocument
```

Oba mogu koristiti iste osnovne mehanizme. To je jedan od razloga zašto postoji `virtual_doctype.py`, koji si video u spisku fajlova.

Dakle, `BaseDocument` nije napravljen slučajno.

E sada želimo da razdvojimo dva sloja:

- Mehaniku Python objekta (`BaseDocument`),
- Frappe ORM ponašanje (`Document`).

```sh
grep "def " ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

```py
def get_controller(doctype):
def import_controller(doctype):
def __init__(self, d):
def meta(self):
def permitted_fieldnames(self) -> set[str]:
def __getstate__(self):
def remove_unpicklable_values(self, state):
def update(self, d):
def update_if_missing(self, d):
def get_db_value(self, key):
def get(self, key, filters=None, limit=None, default=None):
def getone(self, key, filters=None):
def set(self, key, value, as_value=False):
def delete_key(self, key):
def append(self, key: str, value: D | dict | None = None, position: int = -1) -> D:
def parent_doc(self):
def parent_doc(self, value):
def parent_doc(self):
def extend(self, key, value):
def remove(self, doc):
def _init_child(self, value, key):
def _get_table_fields(self):
def get_valid_dict(
def init_child_tables(self):
def init_valid_columns(self):
def get_valid_columns(self) -> list[str]:
def is_new(self) -> bool:
def docstatus(self) -> DocStatus:
def docstatus(self, value) -> None:
def as_dict(
def as_json(self):
def get_table_field_doctype(self, fieldname):
def get_parentfield_of_doctype(self, doctype):
def db_insert(self, ignore_if_duplicate=False):
def db_update(self):
def db_update_all(self):
def show_unique_validation_message(self, e):
def get_field_name_by_key_name(self, key_name):
def get_label_from_fieldname(self, fieldname):
def update_modified(self):
def _fix_numeric_types(self):
def _get_missing_mandatory_fields(self):
def get_msg(df):
def has_content(df):
def get_invalid_links(self, is_submittable=False):
def get_msg(df, docname):
def set_fetch_from_value(self, doctype, df, values):
def _validate_selects(self):
def _validate_data_fields(self):
def _validate_constants(self):
def _validate_length(self):
def _validate_code_fields(self):
def _sync_autoname_field(self):
def throw_length_exceeded_error(self, df, max_length, value):
def _validate_update_after_submit(self):
def _sanitize_content(self):
def _save_passwords(self):
def get_password(self, fieldname="password", raise_exception=True):
def is_dummy_password(self, pwd):
def precision(self, fieldname, parentfield=None) -> int | None:
def get_formatted(
def is_print_hide(self, fieldname, df=None, for_print=True):
def __setup__(self):
def in_format_data(self, fieldname):
def reset_values_if_no_permlevel_access(self, has_access_to, high_permlevel_fields):
def get_value(self, fieldname):
def cast(self, value, df):
def _extract_images_from_text_editor(self):
def _filter(data, filters, limit=None):
```

Medju naslovima poziva vidimo:

```text
db_insert()
db_update()
db_update_all()
```

Dakle, imamo ovde i rad sa bazom.

### Grupisanje metoda

Ovaj spisak od 60+ metoda izgleda zastrašujuće, ali u stvari može da se podeli u nekoliko logičkih grupa.

- **Konstrukcija objekta**:

  ```py
  __init__()
  update()
  update_if_missing()
  ```

  Ove metode služe da naprave i popune objekat.

- **Pristup podacima**:

  ```py
  get()
  getone()
  set()
  append()
  extend()
  remove()
  delete_key()
  get_value()
  ```
  
  Ovo je interfejs za rad sa poljima dokumenta.

- **Serijalizacija**:
  
  ```text
  as_dict()
  as_json()
  __getstate__()
  ```

  Pretvaranje objekta u rečnik, JSON ili stanje za pickle.

- **Meta informacije**:

  ```text
  meta()
  get_valid_columns()
  get_table_field_doctype()
  ```
  
  Ovde se vidi veza sa `Meta` objektom o kome smo pričali.

- **Validacija**:

  Ovde ih ima mnogo:
  
  ```text
  _validate_length()
  _validate_selects()
  _validate_data_fields()
  _validate_constants()
  ...
  ```
  
  To znači da se dosta validacije dešava na nivou samog dokumenta.

- **Baza**:

  ```text
  db_insert()
  db_update()
  db_update_all()
  ```
  
  Ovo je zanimljivo. Obrati pažnju na prefiks `db_`. To mi govori da Frappe pravi razliku između:
  
  ```py
  save()
  i
  db_update()
  ```
  
  To je veoma dobra praksa. Zašto? Zato što `save()` verovatno radi mnogo više od običnog SQL `UPDATE`.
  
  Na primer:
  
  ```txt
  save() -> permissions -> hooks -> validate -> before_save -> db_update() -> after_save 
  ```
  
  Dok:
  
  ```py
  db_update()
  ```
  
  verovatno radi samo posao sa bazom.
  
  To ćemo kasnije proveriti.

Pogledaj ove metode:

```py
meta()
get_valid_columns()
permitted_fieldnames()
```

One mi odmah govore da `BaseDocument` zna šta je njegov DocType.

Drugim rečima:

```txt
Document -> Meta -> DocField
```

nije samo teorija.

Već sada vidimo tragove te veze u samom kodu.

Po meni, najvažnija metoda u celom fajlu nije:

- `db_update()` ili
- `append()` ili
- `get()`.
  
Ja mislim da je to:

```python
@property
def meta(...)
```

Zašto? Jer bez `Meta` objekta `BaseDocument` ne zna:

- koja polja postoje,
- koji je tip svakog polja,
- koja su obavezna,
- koja predstavljaju child tabele,
- koja su `Link` polja,
- koja su `Date`, `Currency`, `Check`, itd.

Drugim rečima, `Meta` je "uputstvo za upotrebu" svakog dokumenta.

### Meta property

```sh
grep -A 8 "def meta" ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

```py
def meta(self):
  return frappe.get_meta(self.doctype)

@cached_property
def permitted_fieldnames(self) -> set[str]:
  return set(get_permitted_fields(doctype=self.doctype, parenttype=getattr(self,  
    "parenttype", None)))

def __getstate__(self):
```

Na prvi pogled deluje gotovo razočaravajuće jednostavno.

Ali u stvari, ovde se krije jedna od najvažnijih ideja Frappe-a.

**`self.doctype`**

Zamisli da imaš objekat:

```python
customer = frappe.get_doc("Customer", "CUST-0001")
```

Taj objekat u sebi ima nešto poput:

```python
customer.doctype == "Customer"
```

Dakle, `doctype` nije Python klasa. To je običan string.

**Šta se dešava kada pozoveš `customer.meta`**

Izvrši se:

```python
frappe.get_meta("Customer")
```

Dakle:

```txt
Customer Document -> doctype = "Customer" -> frappe.get_meta("Customer") -> Meta objekat
```

I sad dolazimo do ključne ideje.

**Šta je zapravo Meta?**

Nemoj ga zamišljati kao "fajl". Zamisli ga kao opis dokumenta u memoriji.

Na primer:

```txt
Meta
  doctype = Customer
    fields
      name
      email
      phone
      country
      customer_group
      territory
```

To nije red iz baze. To je opis strukture.

Zašto je ovo genijalno? Pogledaj šta nigde ne postoji.
  
Ne postoji:

```python
class Customer:
```

Frappe ne mora da ima Python klasu za svaki DocType.

Umesto toga radi:

```txt
Customer -> get_meta() -> Meta -> Document
```

To je ogromna razlika u odnosu na Django.

U Django-u:

```python
class Customer(models.Model):
    email = ...
    phone = ...
```

Klasa opisuje bazu.

U Frappe-u:

```txt
DocType -> Meta -> Document
```

Klasa je ista za sve.

Menja se samo Meta.

To znači... Ako napraviš novi DocType u Desk-u:

```txt
Vehicle
```

Frappe ne generiše:

```py
class Vehicle(Document):
```

Nego samo napravi novi Meta zapis.

Kasnije:

```py
frappe.get_doc("Vehicle")
```

koristi isti `Document`.

To je veoma elegantan dizajn.

**A šta radi `frappe.get_meta()`?**

Ova metoda:

```python
frappe.get_meta("Customer")
```

Mora negde da:

- pročita DocType,
- pročita DocField,
- napravi Meta objekat,
- kešira ga,
- vrati ga.

Drugim rečima:

```txt
Database -> DocType -> DocField -> Meta() -> cache -> return
```

Metoda nije:

```py
def get_meta(self):
```

nego

```py
@property
def meta(self):
```

To znači da pišeš:

```py
doc.meta
```

a ne

```python
doc.get_meta()
```

Zašto? Zato što se autor trudi da `meta` izgleda kao osobina dokumenta, a ne kao neka operacija.

To lepo čitaš:

```python
doc.meta.fields
```

umesto:

```python
doc.get_meta().fields
```

To je mala stvar, ali doprinosi čitljivosti.

A odmah ispod... Vidimo još nešto zanimljivo:
  
```python
@cached_property
def permitted_fieldnames(...)
```

Ovo je odličan primer modernog Pythona.

Prvi put kada pristupiš:

```python
doc.permitted_fieldnames
```

izračunaće skup dozvoljenih polja.

Drugi put... neće ponovo računati. Vratiće vrednost iz keša. To je važno jer se dozvole mogu proveravati mnogo puta tokom jednog zahteva.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
