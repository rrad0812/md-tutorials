
# Frappe framework tutorijal

[Sadržaj][00]

## 09 Tok izvršenja u Frappeu

Hajde da pratimo jedan konkretan tok izvršavanja.

Gde se nalazi `get_doc`? Ja bih rekao: `frappe/__init__.py`

Zašto? Zato što je korisnik poziva kao:

```python
frappe.get_doc(...)
```

To znači da mora biti izložena na nivou paketa `frappe`.

```sh
grep -n "def get_doc" ~/frappe-bench/apps/frappe/frappe/__init__.py
```

```py
1223:def get_document_cache_key(doctype: str, name: str):
1269:def get_doc(document: "Document", /) -> "Document":
1274:def get_doc(doctype: str, /) -> _SingleDocument:
1280:def get_doc(doctype: str, name: str, /, *, for_update: bool | None = None) -> "Document":
1286:def get_doc(**kwargs: dict) -> "_NewDocument":
1293:def get_doc(documentdict: dict) -> "_NewDocument":
1299:def get_doc(*args, **kwargs):
1571:def get_doc_hooks():
2299:def get_doctype_app(doctype):
```

I sada smo naišli na nešto veoma zanimljivo. Pogledaj ovo:

```text
1269:def get_doc(document: "Document", /) -> "Document":
1274:def get_doc(doctype: str, /) -> _SingleDocument:
1280:def get_doc(doctype: str, name: str, /, *, for_update: bool | None = None) -> "Document":
1286:def get_doc(**kwargs: dict) -> "_NewDocument":
1293:def get_doc(documentdict: dict) -> "_NewDocument":
1299:def get_doc(*args, **kwargs):
```

Prva reakcija je obično: "Kako može Python da ima šest funkcija sa istim imenom?".

Odgovor je: ne može. Ovo nisu različite implementacije.

One služe za **type hinting**.

Pogledaj pažljivo poslednju:

```python
def get_doc(*args, **kwargs):
```

To je jedina prava implementacija.

Sve iznad su opisi za:

- IDE (PyCharm, VS Code),
- type checker-e (`mypy`, `pyright`),
- autocomplete,
- dokumentaciju.

To je obrazac koji se često koristi u modernom Pythonu.

Dakle, ovo nije "preopterećenje funkcija" kao u C++ ili Javi, već način da se jednom implementacijom opiše više načina korišćenja.

Python podržava samo:

- jednu implementaciju,
- više deklaracija za potrebe tipova.

Sada nas zanima samo jedna stvar

Hoćemo da vidimo pravu implementaciju.

```sh
sed -n '1299,1325p' ~/frappe-bench/apps/frappe/frappe/__init__.py
```

```py
def get_doc(*args, **kwargs):
  """Return a `frappe.model.document.Document` object of the given type and name.

  :param arg1: DocType name as string **or** document JSON.
  :param arg2: [optional] Document name as string.

  Examples:
            # insert a new document
          todo = frappe.get_doc({"doctype":"ToDo", "description": "test"})
          todo.insert()
            # open an existing document
          todo = frappe.get_doc("ToDo", "TD0001")
  """
  import frappe.model.document
  return frappe.model.document.get_doc(*args, **kwargs)
```

I sada mogu da ti pokažem jedan obrazac koji se ponavlja kroz ceo Frappe.

Pogledaj koliko je funkcija mala:

```python
def get_doc(*args, **kwargs):
    ...
    import frappe.model.document
    return frappe.model.document.get_doc(*args, **kwargs)
```

To je praktično sve.

Dakle... `frappe.get_doc()` ne radi gotovo ništa. Njegov posao je samo da kaže: "Idi u `frappe.model.document` i tamo odradi pravi posao." Zašto ovako?. Ovo je veoma lepo projektovanje. Zamisli da nije ovako. Morali bismo da pišemo:
  
```python
from frappe.model.document import get_doc

doc = get_doc(...)
```
  
ili

```python
import frappe.model.document

doc = frappe.model.document.get_doc(...)
```

Umesto toga možeš jednostavno:

```python
doc = frappe.get_doc(...)
```

To je mnogo prijatniji javni API.

> [!Note]
>
> **Facade**
>
> Ne znam da li si ranije nailazio na ovaj termin. U objektno orijentisanom projektovanju postoji
> obrazac:
>
> ```txt
> Korisnik -> Facade -> Komplikovan sistem
> ```
>
> Ovde je:
>
> ```txt
> Programer -> frappe.get_doc() -> frappe.model.document.get_doc() -> Document -> Database
> ```
>
> Programer vidi samo jednu funkciju. Sve ostalo je sakriveno.

A zašto `import` nije na vrhu fajla?

Ovo je još zanimljivije.

Pogledaj:

```python
def get_doc(...):
    import frappe.model.document
```

Većina Python programa radi import na vrhu pyjton modula.

```python
import ...
import ...
import ...
```

Ovde ne. Zašto?

- Izbegavanje kružnih zavisnosti

  Zamisli:

  ```sh
  __init__.py -> document.py -> base_document.py -> frappe -> __init__.py
  ```

  I odjednom:

  ```py
  ImportError
  ```

  Lokalni `import` često rešava ovakve probleme.

- Brže pokretanje
  
  Ako nikad ne pozoveš `get_doc()`, nikad se neće importovati `document.py`.
  To može malo ubrzati startovanje.

Već sada imamo malu mapu.

```txt
frappe.get_doc() -> frappe.model.document.get_doc() -> ??? -> Document
```

Sada treba da otkrijemo ono **???**

Više nema razloga da ostajemo u `__init__.py`. Pravi posao je ovde:

```sh
frappe/model/document.py
```

Hajde da pronađemo sledeću funkciju.

```sh
grep -n "^def get_doc" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
36:def get_doc(*args, **kwargs):
```

Sada više nije dovoljno da znamo da postoji `get_doc()`.

Moramo videti šta radi.

```sh
sed -n '36,90p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
def get_doc(*args, **kwargs):
  """returns a frappe.model.Document object.

  :param arg1: Document dict or DocType name.
  :param arg2: [optional] document name.
  :param for_update: [optional] select document for update.

  There are multiple ways to call `get_doc`

          # will fetch the latest user object (with child table) from the database
          user = get_doc("User", "test@example.com")

          # create a new object
          user = get_doc({
                  "doctype":"User"
                  "email_id": "test@example.com",
                  "roles: [
                          {"role": "System Manager"}
                  ]
          })

          # create new object with keyword arguments
          user = get_doc(doctype='User', email_id='test@example.com')

          # select a document for update
          user = get_doc("User", "test@example.com", for_update=True)
  """
  if args:
    if isinstance(args[0], BaseDocument):
      # already a document
      return args[0]
    elif isinstance(args[0], str):
      doctype = args[0]

    elif isinstance(args[0], dict):
      # passed a dict
      kwargs = args[0]

    else:
      raise ValueError("First non keyword argument must be a string or dict")

  if len(args) < 2 and kwargs:
    if "doctype" in kwargs:
      doctype = kwargs["doctype"]
    else:
      raise ValueError('"doctype" is a required key')

  controller = get_controller(doctype)
  if controller:
    return controller(*args, **kwargs)

  raise ImportError(doctype)
```

`get_doc()` uopšte ne učitava dokument iz baze.

To znači da ova funkcija ima sasvim drugu ulogu.

Šta ona zapravo radi?

Ja bih je nazvao:

**Fabrika (Factory)**:

Ona ne pravi SQL.  Ona odlučuje: "Koju klasu treba napraviti?"

Hajdemo redom

- **Prvi slučaj**

  ```py
  if isinstance(args[0], BaseDocument):
    return args[0]
  ```

  Ako joj pošalješ već gotov dokument:

  ```py
  doc = frappe.get_doc(existing_doc)
  ˙˙˙
  ```

  ona kaže: "Već imaš Document." i vrati ga.

- **Drugi slučaj**
  
  ```py
  elif isinstance(args[0], str):
      doctype = args[0]
  ```

  Ovo je najčešći poziv.

  Na primer:

  ```py
  frappe.get_doc("Customer", "CUST-0001")
  ```

  Ovde samo zapamti:

  ```py
  doctype = "Customer"
  ```

  Još ništa nije učitano.

- **Treći slučaj**
  
  ```py
  elif isinstance(args[0], dict):
  ```

  Ovde praviš novi dokument.

  Na primer:

  ```py
  frappe.get_doc({
      "doctype": "Customer",
      "customer_name": "Marko"
  })
  ```

  Dakle ista funkcija podržava i:

  - otvaranje postojećeg,

  - pravljenje novog dokumenta.

A onda... Dolazimo do najvažnije linije.

```py
controller = get_controller(doctype)
```

Po meni, ovo je ključ cele funkcije.

Jer odjednom više nije važno:

- da li je dokument novi,
- da li dolazi iz baze,
- da li je Customer,
- da li je Item.

Sve se svodi na jedno pitanje: Ko upravlja ovim DocType-om?
  
### Controller
  
"Controller" je zapravo Python klasa koja predstavlja DocType.

Na primer:

```txt
Customer
Customer(Document)
```

Ako takva klasa postoji. Ako ne... videćemo šta Frappe radi.

Pogledaj završetak funkcije.

```py  
return controller(*args, **kwargs)
```

Ovo je fantastično. Ne piše:

- `Customer(...)`.
- `User(...)`.
- `Task(...)`.

Nego: `controller(...)`.

Drugim rečima:

```py
get_doc() -> get_controller() -> ??? -> pozovi klasu
```

To je čisti **Factory Pattern**.

Sećaš se kada smo govorili:

```txt
DocType -> Meta -> Document
```

Sada možemo malo da dopunimo dijagram.

```txt
frappe.get_doc() -> document.get_doc() -> get_controller() -> ??? -> Document(...)
```

Odgovor na ono veliko ??? krije se upravo u `get_controller()`.
  
I evo pitanja koje sada možemo postaviti:

"Ako napišem: `frappe.get_doc("Customer")` kako `get_controller()` zna da vrati baš: "Customer" a ne: "User" ili: "Task".

I još važnije... Šta ako "Customer.py" uopšte ne postoji?

Pratimo ovu jednu liniju:

```py
controller = get_controller(doctype)
```

Pošto smo je već videli u `base_document.py`, sada ćemo konačno razumeti njenu ulogu.

```sh
grep -A 50 "^def get_controller" ~/frappe-bench/apps/frappe/frappe/model/
base_document.py
```

```py
def get_controller(doctype):
    """
    Returns the locally cached **class** object of the given DocType.
    For `custom` type, returns `frappe.model.document.Document`.
    :param doctype: DocType name as string.
    """
    
    if frappe.local.dev_server or frappe.flags.in_migrate:
        return import_controller(doctype)
    
    site_controllers = frappe.controllers.setdefault(frappe.local.site, {})
    
    if doctype not in site_controllers:
        site_controllers[doctype] = import_controller(doctype)
    
    return site_controllers[doctype]
```

**Prvo**:

Pogledaj docstring:

```python
Returns the locally cached **class** object of the given DocType.
```

Obrati pažnju na reči: **class object**. To znači da ova funkcija ne vraća Customer objekat.
Ona vraća odnosno sam objekat klase

```py
Customer
```

Dakle...

```py
controller = get_controller("Customer")
```

Sada znamo da je rezultat nešto poput:

```py
controller == Customer # Klasa tipa Customer
```

ili možda:

```py
controller == Document # Klasa tipa Document, jer je Customer nasleđen iz Document
```

zavisno od DocType-a.

Posle toga:

```py
return controller(*args, **kwargs)
```

znači:

```py
Customer(*args, **kwargs)
```

ili

```py
Document(*args, **kwargs)
```
  
**Drugo**:

Pogledaj ovaj deo koda:

```python
if frappe.local.dev_server or frappe.flags.in_migrate:
    return import_controller(doctype)
```

Ovde se vidi da Frappe razlikuje dva režima rada.

- **Normalan rad**
  Koristi keš.

- **Development**  
  Ne koristi keš.  
  Zašto? Jer u development-u menjaš kod. Ako bi klasa ostala keširana...  menjaš fajl... a Frappe i dalje koristi staru klasu. To bi bilo veoma frustrirajuće.

**Treće**:

```python
site_controllers = frappe.controllers.setdefault(
    frappe.local.site,
    {}
)
```

Ovde prvi put vidimo nešto veoma važno.

Frappe nije zamišljen kao:

```txt
jedan server
jedna baza
```

nego kao:

```txt
jedan Bench
    │
    ├── site1.local
    ├── site2.local
    ├── companyA
    └── companyB
```

Dakle... **keš nije globalan.**. Keš je: **po sajtu**, jer dva sajta mogu imati različite aplikacije.

Evo kako ja to zamišljam:
  
- Pogledaj ovo:
  
  ```python
  if doctype not in site_controllers:
  ```
  
  Dakle prvi put:
  - Customer
  - učitaj klasu
  - stavi u keš
  
  Drugi put:
  - Customer
  - vrati iz keša
  
  Bez ponovnog importa.

**import_controller**:

Videli smo:

```python
return import_controller(doctype)
```

To je sada "crna kutija". Tu se krije odgovor na pitanje: Kako string `"Customer"` postaje Python klasa? To je upravo ono što smo tražili još od početka.

Voleo bih da obratiš pažnju na nešto.

Pre desetak koraka imali smo dijagram:

```txt
    get_doc() -> Document
```

Sada je on postao mnogo precizniji.

```txt
    get_doc() -> get_controller() -> import_controller() -> Python class -> Document instance
```

Pogledaj kako je funkcija napisana. Nema 200 linija. Nema ogromnog `if`. Nema SQL-a. Radi samo jednu stvar: "Pronađi odgovarajuću klasu i keširaj je."

To je odličan primer principa **Single Responsibility Principle (SRP)** iz SOLID-a.

Mislim da smo sada došli do mesta gde treba otvoriti poslednju "crnu kutiju":

```python
import_controller(doctype)
```

Kompletna arhitektura:

```txt
frappe.get_doc(...) -> frappe.model.document.get_doc(...) -> get_controller(doctype) -> import_controller(doctype) -> Python klasa -> Instanca Document
```

```sh
grep -A 80 "^def import_controller" ~/frappe-bench/apps/frappe/frappe/model/
base_document.py
```

```py
def import_controller(doctype):
  from frappe.model.document import Document
  from frappe.utils.nestedset import NestedSet

  module_name = "Core"
  if doctype not in DOCTYPES_FOR_DOCTYPE:
    doctype_info = frappe.db.get_value("DocType", doctype, ("module", 
       "custom", "is_tree"), as_dict=True)
    if doctype_info:
      if doctype_info.custom:
        return NestedSet if doctype_info.is_tree else Document
      module_name = doctype_info.module

  module_path = None
  class_overrides = frappe.get_hooks("override_doctype_class")
  if class_overrides and class_overrides.get(doctype):
    import_path = class_overrides[doctype][-1]
    module_path, classname = import_path.rsplit(".", 1)
    module = frappe.get_module(module_path)

  else:
    module = load_doctype_module(doctype, module_name)
    classname = doctype.replace(" ", "").replace("-", "")
```

Moja hipoteza od prošlog puta bila je skoro tačna, ali ne potpuno. Ja sam očekivao nešto poput:
  
```text
DocType -> nađi modul -> import -> vrati klasu
```
  
Ali Frappe radi još nekoliko veoma zanimljivih stvari usput.
  
- **Prvo**

  Definiše podrazumevane klase:

  ```python
  from frappe.model.document import Document
  from frappe.utils.nestedset import NestedSet
  ```
  
  Već na početku vidimo dve moguće "osnovne" klase. To znači da Frappe već zna da postoje najmanje dve porodice DocType-ova:
  
  ```txt
  Document
  NestedSet
  ```
  
  `NestedSet` ćemo ostaviti za kasnije (koristi se za hijerarhijske strukture), ali je zanimljivo da se već ovde pojavljuje.

- **Drugo**
  
  Prvi odlazak u bazu. Ovo je prvi put da naš tok izvršavanja zaista odlazi u bazu.
  
  ```python
  doctype_info = frappe.db.get_value(
      "DocType",
      doctype,
      ("module", "custom", "is_tree"),
      as_dict=True
  )
  ```

  Primeti šta čita. Ne čita:
  
  - Customer
  - User
  - Sales Invoice
  
  nego čita:
  
  ```text
  DocType
  ```
  
  Drugim rečima: Prvo se učitavaju metapodaci o DocType-u.
  
- **Treće**  
  
  Document ili NestedSet.
  
  Pogledaj ovo:
  
  ```python
  if doctype_info.custom:
      return NestedSet if doctype_info.is_tree else Document
  ```
  
  Provera da li je DoxType custom, ako jeste proverava da li je stablo i tada NestedSet inače vraća Document.

- **Četvrto**

  Override DocType.

  Dolazimo do nečega što mi se posebno dopada.

  ```python
  class_overrides = frappe.get_hooks("override_doctype_class")
  ```

  Ovo pokazuje koliko je Frappe otvoren za proširenja. Ne kaže: "Koristi ovu klasu." Nego prvo pita: "Da li je neko u hook-ovima rekao da želi drugu klasu?" To znači da aplikacija može da zameni implementaciju nekog DocType-a bez menjanja izvornog koda Frappe-a.
  To je veoma moćan mehanizam.

- **import**  
  
  Ako nema override-a:
  
  ```py
  module = load_doctype_module(...)
  ```
  
  Dakle moja prethodna hipoteza:
  
  ```py
  import modul
  ```
  
  jeste tačna. Ali nije prvi korak.

**Kako pronalazi pravu klasu?**  
Veoma jednostavno.

```python
classname = doctype.replace(" ", "").replace("-", "")
```

Dakle:

```txt
Sales Invoice
```

postaje

```txt
SalesInvoice
```

a zatim:

```python
getattr(module, classname)
```

To je potpuno standardan Python.

**Provera**:

Još jedna stvar koja mi se dopada.

```python
issubclass(class_, BaseDocument)
```

Drugim rečima, ako napišeš:

```python
class Customer:
    ...
```

Frappe će reći: Ne.  

Mora da bude:

```python
class Customer(Document):
```

ili neka druga izvedena klasa `BaseDocument`.

To čuva konzistentnost celog framework-a.

**Najzanimljiviji deo**:

Ako pogledamo ceo tok koji smo do sada ispratili, to izgleda ovako:

```text
frappe.get_doc(...) -> document.get_doc(...) -> get_controller(doctype) -> (import iz keša ili import_controller) -> DocType tabela  -> prvi odlazak u bazu -> 

    | -> custom -> Document          |\
    | -> hook overide? -> druga klasa |  ->
    | -> load_doctype_module         |/

getattr(...) -> Python klasa -> controller(*args)
```

Ovo je jedan od najvažnijih dijagrama koje smo do sada napravili.

Frappe kaže:

- za **custom DocType** → meta model je dovoljan (`Document`),
- za **standardne ili naprednije DocType-ove** → možeš dodati Python kontroler,
- a čak možeš i **zameniti kontroler** preko hook-ova.

To nije bolji ili lošiji pristup — to je drugačiji kompromis između fleksibilnosti i jednostavnosti.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
