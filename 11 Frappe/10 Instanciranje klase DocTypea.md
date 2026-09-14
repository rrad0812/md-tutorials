
# Frappe framework tutorijal

[Sadržaj][00]

## 10 Instaciranje klase DocTypea

Hajde da pratimo šta se dešava kada se klasa instancira, odnosno kako `Document.__init__()` (i ono što nasleđuje iz `BaseDocument`) pretvara prosleđene argumente u pravi objekat. Time ćemo zatvoriti ceo životni ciklus `frappe.get_doc()` pre nego što pređemo na čuvanje u bazu.

Došli smo do ove linije:

```python
controller(*args, **kwargs)
```

Pretpostavimo da je `controller` zapravo:

```python
Customer
```

koji nasleđuje:

```text
Customer -> Document -> BaseDocument
```

Pošto `Document` nema svoj `__init__()`, Python će pozvati:

```python
BaseDocument.__init__()
```

```python
def __init__(self, d):
    if d.get("doctype"):
        self.doctype = d["doctype"]

    self._table_fieldnames = {
        df.fieldname for df in self._get_table_fields()
    }

    self.update(d)
    self.dont_update_if_missing = []

    if hasattr(self, "__setup__"):
        self.__setup__()
```

**Oderđivanje DocTypea**:

Hajde da ga čitamo veoma pažljivo:

```python
if d.get("doctype"):
    self.doctype = d["doctype"]
```

Ovde se dešava nešto sasvim očekivano.

Ako si napravio:

```python
frappe.get_doc({
    "doctype": "Customer",
    "customer_name": "Marko"
})
```

objekat odmah dobija:

```python
self.doctype = "Customer"
```

Dakle doctype je identitet objekta.

**Čitanje table_fieldnames**:

```python
self._table_fieldnames = {
    for df in self._get_table_fields()
}
```

Ovde se ne čitaju obična polja. Čitaju se **Table** polja. Frappe odmah želi da zna: Koja polja predstavljaju child tabele?
  
- **Čitanje običnih polja**

  ```python
  self.update(d)
  ```
  
  Ovde sam skoro siguran da počinje pravo "punjenje" objekta.
  
  Drugim rečima:
  
  iz ovog:
  
  ```python
  {
      "doctype": "Customer",
      "customer_name": "Marko"
  }
  ```
  
  nastaje:
  
  ```python
  doc.customer_name
  ```
  
- **Dodatna inicijalizacija**

  ```python
  if hasattr(self, "__setup__"):
      self.__setup__()
  ```
  
  To znači: ako kontroler želi dodatnu inicijalizaciju... ne mora da dira `__init__()`. Samo napiše:
  
  ```python
  def __setup__(self):
      ...
  ```
  
  To je lep obrazac.

Po meni se sada pojavljuje nova "crna kutija".
  
```python
self.update(d)
```
  
```py
grep -A 60 "^ def update" ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

```py
  def update(self, d):
    """Update multiple fields of a doctype using a dictionary of key-value pairs.

    Example:
            doc.update({
                    "user": "admin",
                    "balance": 42000
            })
    """

    # set name first, as it is used a reference in child document
    if "name" in d:
      self.name = d["name"]

    as_value = not self._table_fieldnames or self.flags.get("ignore_children", False)
    for key, value in d.items():
      self.set(key, value, as_value=as_value)

    return self

  def update_if_missing(self, d):
    """Set default values for fields without existing values"""
    if isinstance(d, BaseDocument):
      d = d.get_valid_dict()

    for key, value in d.items():
      if (
        value is not None
        and self.get(key) is None
        # dont_update_if_missing is a list of fieldnames
        # for which you don't want to set default value
        and key not in self.dont_update_if_missing
      ):
        self.set(key, value)

  def set(self, key, value, as_value=False):
    if key in self._reserved_keywords:
      return

    if not as_value and key in self._table_fieldnames:
      self.__dict__[key] = []

      # if value is falsy, just init to an empty list
      if value:
        self.extend(key, value)

      return

    self.__dict__[key] = value
```

- **Prvo**

  `update()` metoda. radi samo tri stvari.
  
  ```python
  if "name" in d:
      self.name = d["name"]
  
  as_value = ...
  
  for key, value in d.items():
      self.set(key, value, as_value=as_value)
  ```
  
  I to je sve. `update()` ne zna kako se postavlja polje. Ona samo obilazi rečnik.
  
  Pravi posao prebacuje na:
  
  ```python
  self.set(...)
  ```

  **Zašto prvo postavlja `name`?**

  Komentar kaže:
  
  ```python
  # set name first, as it is used a reference in child document
  ```
  
  To je veoma logično.
  
  Zamisli:
  
  ```python
  Sales Invoice
  ```
  
  ima child tabelu:
  
  ```text
  items
  ```
  
  Svaki child red ima:
  
  ```text
  parent = "SINV-00001"
  ```
  
  Ako roditelj još nema `name`... onda child dokumenti ne znaju kome pripadaju.
  
  Dakle:
  
  ```text
  name -> child rows
  ```

- **Drugo**

  ```python
  as_value = not self._table_fieldnames ...
  ```
  
  Postoje dva načina postavljanja polja.

  - **Obično polje**

    ```python
    customer_name -> samo upiši vrednost.
    ```

  - **Table polje**

    ```python
    items 
    ```

    ne sme samo:

    ```python
    self.items = [...]
    ```

    prvo treba napraviti child objekte.

    Zato postoji:

    ```python
    self.set(...)
    ```

- **`set()`**

  - **Rezervisane reči**

    ```python
    if key in self._reserved_keywords:
        return
    ```

    Dakle korisnik ne može slučajno da uradi:

    ```python
    doc.update({
        "meta": ...
    })
    ```

    ili

    ```python
    "flags"
    ```

    To štiti internu strukturu objekta.

  - **Child tabela**

    ```python
    if key in self._table_fieldnames:
    ```

    Prvo:

    ```python
    self.__dict__[key] = []
    ```

    Dakle:

    ```text
    items = []
    ```

    pa tek onda:

    ```python
    self.extend(key, value)
    ```

    Drugim rečima... ne kopira listu. Nego je obrađuje, jer `value` verovatno izgleda ovako:

    ```python
    [
      {
        "item_code": "A"
      },
      {
        "item_code": "B"
      }
    ]
    ```

    Ali u objektu neće ostati dict.

    Verovatno će postati:

    ```text
    SalesInvoiceItem
    ```

    objekti.

    To još nismo videli. Ali gotovo sam siguran. Jer postoji:

    ```python
    append()
    extend()
    _init_child()
    ```

    Sve smo ih ranije videli u listi metoda.

  - **četvrto**

  Ako nije child tabela:
  
  ```python
  self.__dict__[key] = value
  ```
  
  To je to. Bez magije. Bez refleksije. Bez ORM čarolije.
  
  Samo:
  
  ```python
  self.__dict__
  ```
  
Znači, kada Frappe kaže, ako nije tabela:

```text
dict -> update() -> set() -> atribut
```

a ako jeste

```text
dict -> update() -> set() -> ako jeste tabela -> extend() -> child Document
```

Dakle već sada postoji razdvajanje između:

- običnih vrednosti,
- child objekata.

```py
grep -A 40 "^ def extend" ~/frappe-bench/apps/frappe/frappe/model/ 
base_document.py
```

```py
  def extend(self, key, value):
    try:
      value = iter(value)
    except TypeError:
      raise ValueError

    for v in value:
      self.append(key, v)

  def _init_child(self, value, key):
    if not isinstance(value, BaseDocument):
      if not (doctype := self.get_table_field_doctype(key)):
        raise AttributeError(key)

      value["doctype"] = doctype
      value = get_controller(doctype)(value)

    value.parent = self.name
    value.parenttype = self.doctype
    value.parentfield = key

    if value.__dict__.get("docstatus") is None:
      value.__dict__["docstatus"] = DocStatus.DRAFT

    if not getattr(value, "idx", None):
      if table := getattr(self, key, None):
        value.idx = len(table) + 1
      else:
        value.idx = 1

    if not getattr(value, "name", None):
```

Komentar:

- **Prvo`**

  Prvo pogledaj koliko je mala:
  
  ```python
  def extend(self, key, value):
      value = iter(value)
  
      for v in value:
          self.append(key, v)
  ```
  
  Ona ne zna ništa o child dokumentima. Ne zna ništa o DocType-ovima. Ne zna ništa o bazi.
  
  Radi samo:
  
  ```text
  lista -> append() -> append() -> append() ...
  ```
  
  Prava priča počinje u:

- **`_init-child()`**

  ```python
  if not isinstance(value, BaseDocument):
  ```

  Drugim rečima: Ako si već dao pravi Document... ne diraj ga.
  
  Ako si dao:

  ```python
  {
      "item_code": "ABC"
  }
  ```

  onda tek počinje obrada.

- **Drugo**

  ```python
  doctype = self.get_table_field_doctype(key)
  ```
  
  Child tabela sama po sebi nije dovoljna.
  
  Na primer:
  
  ```text
  Sales Invoice -> items
  ```
  
  Kako zna da je svaki red:
  
  ```text
  Sales Invoice -> Item
  ```
  
  Odgovor je: Meta.
  
  Opet. Ne postoji hardkodirano:
  
  ```python
  if key == "items":
  ```
  
  nego:
  
  ```text
  Meta -> items -> Sales Invoice Item
  ```
  
  To je upravo metadata-driven dizajn.
  
- **Treće**

  ```python
  value["doctype"] = doctype
  ```
  
  Ovde se događa nešto veoma važno.
  
  Ako je ulaz bio:
  
  ```python
  {
      "item_code": "ABC"
  }
  ```
  
  Sada postaje:
  
  ```python
  {
      "doctype": "Sales Invoice Item",
      "item_code": "ABC"
  }
  ```
  
  Dakle runtime dopunjava podatke.

- **Četvrto**

  I sada dolazi najlepša linija.
  
  ```python
  value = get_controller(doctype)(value)
  ```
  
  Sećaš se čime smo počeli celu priču?
  
  ```python
  frappe.get_doc(...) -> get_controller() -> import_controller()
  ```
  
  E pa... isti mehanizam se koristi i za child dokumente.
  
  To znači da Frappe nema dva različita sistema. I parent i child dokumenti nastaju na isti način.

  Posle toga:
  
  ```python
  value.parent = self.name
  ```
  
  ```python
  value.parenttype = self.doctype
  ```
  
  ```python
  value.parentfield = key
  ```
  
  Drugim rečima: child dokument dobija svoju "adresu".
  
  Na primer:
  
  ```text
  Sales Invoice
  name = SINV-00001
  ```
  
  ↓
  
  Child:
  
  ```text
  parent = SINV-00001
  parenttype = Sales Invoice
  parentfield = items
  ```
  
  To je upravo ono što omogućava Frappe-u da zna kome pripada svaki red.

- **`docstatus`**

  Zatim:

  ```python
  DocStatus.DRAFT
  ```
  
  Dakle svaki novi child red automatski kreće kao nacrt. To je očekivano.

- **`idx`**

  I onda:
  
  ```python
  value.idx = len(table) + 1
  ```
  
  Frappe automatski numeriše redove.
  
  Dakle:
  
  ```text
  Item A
  idx = 1
  
  Item B
  idx = 2
  
  Item C
  idx = 3
  ```
  
  Bez ikakvog dodatnog posla korisnika.

- **Doslednost Frappea**

  Iskreno, ovo je trenutak kada sam shvatio da je Frappe mnogo dosledniji nego što sam očekivao.
  
  Pogledaj obrazac:
  
  ```txt
  Parent -> update() -> set() -> extend() -> append() -> _init_child() -> get_controller() -> Document
  ```
  
  Dakle isti mehanizam instanciranja koristi se svuda. Nema posebnog "child engine".
  
  To je odličan dizajn.

Frappe koristi metapodatke da odgovori na pitanje: "Koji je DocType ovog reda?". A zatim koristi isti mehanizam (`get_controller`) da napravi odgovarajući Python objekat. Dakle, metapodaci i objekti nisu odvojeni svetovi, nego metapodaci vode do objekata.

Međutim... Na samom kraju izlaza vidim:

```python
if not getattr(value, "name", None):
```

i tu se kod prekida.

Do sada smo praktično dokazali sledeći tok:
  
```text
frappe.get_doc() ->
document.get_doc() ->
get_controller() ->
import_controller() ->
BaseDocument.__init__() ->
update() -> set() ->
  |  obična polja -> __dict__ ->
  |  child tabela -> extend() -> append() -> _init_child() ->
get_controller(child_doctype).
```

To je, po mom mišljenju, već kompletna priča o tome kako jedan `dict` postaje mreža povezanih `Document` objekata.

```sh
sed -n '220,280p' ~/frappe-bench/apps/frappe/frappe/model/base_document.py
```

```py
    if limit and isinstance(value, list | tuple) and len(value) > limit:
      value = value[:limit]

    return value

  def getone(self, key, filters=None):
    return self.get(key, filters=filters, limit=1)[0]

  def set(self, key, value, as_value=False):
    if key in self._reserved_keywords:
      return

    if not as_value and key in self._table_fieldnames:
      self.__dict__[key] = []

      # if value is falsy, just init to an empty list
      if value:
        self.extend(key, value)

      return

    self.__dict__[key] = value

  def delete_key(self, key):
    if key in self.__dict__:
      del self.__dict__[key]

  def append(self, key: str, value: D | dict | None = None, position: int = -1) -> D:
    """Append an item to a child table.

    Example:
            doc.append("childtable", {
                    "child_table_field": "value",
                    "child_table_int_field": 0,
                    ...
            })
    """
    if value is None:
      value = {}

    if (table := self.__dict__.get(key)) is None:
      self.__dict__[key] = table = []

    d = self._init_child(value, key)

    if position == -1:
      table.append(d)
    else:
      # insert at specific position
      table.insert(position, d)

      # re number idx
      for i, _d in enumerate(table):
        _d.idx = i + 1

    # reference parent document but with weak reference, parent_doc will be deleted if self is garbage collected.
    d.parent_doc = weakref.ref(self)

    return d

  @property
```

Hajde da završimo ovu celinu.

Ako korisnik napiše:

```python
doc = frappe.get_doc({
    "doctype": "Sales Invoice",
    "customer": "ABC",
    "items": [
        {
            "item_code": "ITEM-001",
            "qty": 2
        },
        {
            "item_code": "ITEM-002",
            "qty": 5
        }
    ]
})
```

ono što se dešava ispod haube sada možemo skoro potpuno da nacrtamo.

```text
    frappe.get_doc() -> document.get_doc() -> get_controller("Sales Invoice") ->
    import_controller() -> SalesInvoice(...) ili Document(...) -> BaseDocument.__init__() ->     update() ->
      set(customer) ->
        __dict__["customer"]="ABC" -> set(items) -> extend() -> append() -> _init_child() ->
        get_controller("Sales Invoice Item") -> SalesInvoiceItem(...)
```

Po meni je ovo već kompletan mentalni model prve faze rada Frappe-a.

- **`append()`**  
  Lep primer "jedne odgovornosti"

  Pogledaj šta radi.
  
  - **Ako ništa nije prosleđeno**
  
    ```python
    if value is None:
        value = {}
    ```

    Sasvim jednostavno.
  
  - **Ako tabela još ne postoji**
  
    ```python
    if (table := self.__dict__.get(key)) is None:
        self.__dict__[key] = table = []
    ```

    Dakle:

    ```text
    items
    ```

    postaje

    ```python
    []
    ```

  - **Najvažnija linija**

    ```python
    d = self._init_child(value, key)
    ```

    Primeti nešto zanimljivo.

    `append()` uopšte ne zna kako nastaje child dokument.

    On samo kaže: "Daj mi gotov child." To je još jedan primer veoma lepog razdvajanja odgovornosti.

    Ako je:

    ```python
    position == -1
    ```

    onda radi:

    ```python
    table.append(d)
    ```

    Inače umeće na određenu poziciju i ponovo numeriše `idx`.

  - **Posebno zanimljivo**

    ```python
    d.parent_doc = weakref.ref(self)
    ```

    Ovo je veoma zanimljivo.

    Ne čuva:

    ```python
    d.parent_doc = self
    ```

    nego:

    ```python
    weakref.ref(self)
    ```

    Zašto?  
    Zbog referenci.  
    Zamisli da imaš:

    ```text
    Parent
    ```

    koji pokazuje na:

    ```text
    Child
    ```

    a Child pokazuje nazad na:

    ```text
    Parent
    ```

    To je kružna referenca.

    Python GC ume da ih rešava, ali one mogu biti skupe i komplikovane. Frappe kaže: "Child može da zna roditelja, ali preko **slabe reference**." To znači: ako roditelj nestane... child ga neće sprečiti da bude oslobođen iz memorije.

    To je mali detalj, ali pokazuje da su autori razmišljali i o upravljanju memorijom.

Na početku sam mislio da je:

```text
Document
```

glavni objekat. Sada mislim da nije. Po meni, prava osovina sistema izgleda ovako:

```text
Meta -> BaseDocument -> Document -> Specijalizovani kontroleri
```

Drugim rečima:

- **Meta** opisuje strukturu.
- **BaseDocument** zna kako da napravi objekat.
- **Document** dodaje ponašanje.
- **Kontroleri** dodaju poslovnu logiku.

To je veoma lepo slojevita arhitektura.

Frappeova filozofija je prilično jednostavna: "Sve može da radi samo na metapodacima (`Document`), ali ako zatreba, svaki DocType može da dobije svoju Python klasu."

To znači da Frappe zadržava metadata-driven pristup, ali ostavlja "izlaz" za kompleksnu poslovnu logiku bez menjanja samog framework-a.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
