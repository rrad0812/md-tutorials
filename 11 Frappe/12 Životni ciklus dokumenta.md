# Frappe framework tutorijal

[Sadržaj][00]

## 12 Životni ciklus dokumenta

Rekao bih da je životni ciklus dokumenta za Frappe isto ono što je HTTP request lifecycle za Django ili middleware pipeline za ASP.NET.

Ako ovo razumeš, razumećeš gde ide gotovo sav poslovni kod.

### insert()

Ne zanima nas (za sada) kako `insert()` radi iznutra.

Nego: Ako napišem:

```python
doc.insert()
```

šta će Frappe redom da pozove?

Jer od toga zavisi gde ćeš pisati svoj kod.

Hajde da krenemo od programera. Zamisli da imaš:

```python
doc = frappe.new_doc("Service Order")
doc.customer = "ABC"
doc.insert()
```

Pitanje je: **Šta bi framework morao da uradi?**

Ja bih očekivao nešto ovako:
  
```text
  1. Da li je dokument ispravan? ->
  2. Validacija ->
  3. Da li treba nešto automatski popuniti? ->
  4. Upis u bazu ->
  5. Obavesti ostale delove sistema ->
  6. Kraj
```

To je intuitivno.

Ali... Koliko događaja postoji? Kojim redom? Koji se izvršavaju samo kod `insert()`, a koji i kod `save()`?

To sada treba da otkrijemo.

Hajde da pronađemo `insert()`.
  
```sh
grep -n "def insert" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```sh
261:    def insert(
```

```sh
sed -n '261,380p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
def insert(
    self,
        ignore_permissions=None,
        ignore_links=None,
        ignore_if_duplicate=False,
        ignore_mandatory=None,
        set_name=None,
        set_child_names=True,
    ) -> "Document":
        """Insert the document in the database (as a new document).
        This will check for user permissions and execute `before_insert`,
        `validate`, `on_update`, `after_insert` methods if they are written.

        :param ignore_permissions: Do not check permissions if True.
        :param ignore_links: Do not check validity of links if True.
        :param ignore_if_duplicate: Do not raise error if a duplicate entry 
            exists.
        :param ignore_mandatory: Do not check missing mandatory fields if 
            True.
        :param set_name: Name to set for the document, if valid.
        :param set_child_names: Whether to set names for the child documents.
        """
        if self.flags.in_print:
            return self

        self.flags.notifications_executed = []

        if ignore_permissions is not None:
            self.flags.ignore_permissions = ignore_permissions

        if ignore_links is not None:
            self.flags.ignore_links = ignore_links

        if ignore_mandatory is not None:
            self.flags.ignore_mandatory = ignore_mandatory

        self.set("__islocal", True)

        self._set_defaults()
        self.set_user_and_timestamp()
        self.set_docstatus()
        self.check_permission("create")
        self.check_if_latest()
        self._validate_links()
        self.run_method("before_insert")
        self.set_new_name(set_name=set_name, set_child_names=set_child_names)
        self.set_parent_in_children()
        self.validate_higher_perm_levels()

        self.flags.in_insert = True
        self.run_before_save_methods()
        self._validate()
        self.set_docstatus()
        self.flags.in_insert = False

        # run validate, on update etc.

        # parent
        if getattr(self.meta, "issingle", 0):
            self.update_single(self.get_valid_dict())
        else:
            self.db_insert(ignore_if_duplicate=ignore_if_duplicate)

        # children
        for d in self.get_all_children():
            d.db_insert()

        self.run_method("after_insert")
        self.flags.in_insert = True

        if self.get("amended_from"):
            self.validate_amended_from()
            self.copy_attachments_from_amended_from()

        relink_mismatched_files(self)
        self.run_post_save_methods()
        self.flags.in_insert = False

        # delete __islocal
        if hasattr(self, "__islocal"):
            delattr(self, "__islocal")

        # clear unsaved flag
        if hasattr(self, "__unsaved"):
            delattr(self, "__unsaved")

        if not (frappe.flags.in_migrate or frappe.local.flags.in_install or 
            frappe.flags.in_setup_wizard):
            if frappe.get_cached_value("User", frappe.session.user, 
                "follow_created_documents"):
                follow_document(self.doctype, self.name, frappe.session.user)
        return self

    def save(self, *args, **kwargs):
        """Wrapper for _save"""
        return self._save(*args, **kwargs)

    def _save(self, ignore_permissions=None, ignore_version=None) -> 
        "Document":
```
  
Mislim da se `insert()` može podeliti u četiri logične faze.

#### 1 Priprema

```python
self._set_defaults()
self.set_user_and_timestamp()
self.set_docstatus()
self.check_permission("create")
self.check_if_latest()
self._validate_links()
```

Ovo nisu događaji koje ti pišeš. Ovo framework radi da bi dokument bio spreman. Ja ovo zovem: "Infrastructure phase".  

Tu Frappe postavlja temelje.

#### 2 before_insert()

Ovde se prvi put pojavljuje nešto što programer piše.

```python
self.run_method("before_insert")
```

Ovo znači:

Ako u svom kontroleru napišeš:

```python
class ServiceOrder(Document):

    def before_insert(self):
        ...
```

ovde će biti pozvano.

I odmah jedna važna stvar. `before_insert()` se izvršava samo jednom u životu dokumenta. Nikada više.

To ga razlikuje od mnogih drugih metoda.

##### 2.1 set_new_name

```python
self.set_new_name(...)
```

Framework dodeljuje ime.

Na primer:

```py
SO-00001
```

ili

```py
CUST-00034
```

ili šta god `Naming Rule` kaže.

##### 2.2 set_parent_in_children()

Posle toga:

```python
self.set_parent_in_children()
```

To smo praktično već videli kada smo proučavali `_init_child()`.

#### 3 Najvažniji deo

Po meni je ovo srce celog životnog ciklusa.

```python
self.run_before_save_methods()
self._validate()
self.set_docstatus()
```
  
##### 3.1 run_before_save_methods()
  
To je, po mom mišljenju, sledeća funkcija koju treba otvoriti.

Zašto?

Jer se iz njenog imena vidi da ona verovatno poziva:

- validate
- before_save
- možda još nešto

Drugim rečima, mislim da se **pravi redosled događaja krije upravo tu**.

#### 4 Upis u bazu

Ovde više nema filozofije.

```python
self.db_insert()
```

i zatim

```python
child.db_insert()
```

Dakle: Prvo roditelj. Pa sva deca.

To ima smisla.

#### 5 Posle baze

Sada dolaze događaji koji se izvršavaju **nakon INSERT-a**.

Prvo:

```python
self.run_method("after_insert")
```

Dakle:

```python
class ServiceOrder(Document):

    def after_insert(self):
        ...
```

će se izvršiti ovde.

A onda:

```python
self.run_post_save_methods()
```

I evo opet funkcije koju moramo otvoriti.

Po imenu očekujem da upravo ona poziva:

- on_update
- notify
- hook-ove
- možda Version
- možda Assignment
- možda Timeline

#### Dijagram poziva

Već imamo prvu verziju dijagrama.

```text
insert()
   ├── priprema
   ├── before_insert()
   ├── naming
   ├── run_before_save_methods()
   ├── _validate()
   ├── db_insert()
   ├── child.db_insert()
   ├── after_insert()
   └── run_post_save_methods()
```

##### Framework ne poziva direktno

Pogledaj:

```python
self.run_method("before_insert")
```

Framework **ne poziva direktno** `before_insert` nego:

```python
run_method(...)
```

To znači da `run_method()` verovatno radi mnogo više od običnog poziva metode.

Moja pretpostavka je da:

- poziva metodu kontrolera (ako postoji),
- poziva `doc_events` hook-ove,
- možda Server Script,
- možda još neke ekstenzije.

Ako je to tačno, onda je **run_method()** centralna tačka sistema događaja u Frappe-u.

#### run_pre_save_methods()

Iz `insert()` se vidi da je redosled:

```text
before_insert() -> run_before_save_methods() -> _validate() -> db_insert() -> after_insert() -> run_post_save_methods()
```

Ovde postoje dve "kutije":

- `run_before_save_methods()`
- `run_post_save_methods()`

Po mom iskustvu sa frameworcima, upravo su takve funkcije **orkestratori**. One ne rade mnogo same, nego pozivaju desetak drugih metoda odgovarajućim redosledom.

```sh
grep -R -n "def run_before_save_methods" ~/frappe-bench/apps/frappe/frappe
```

```sh
/home/radosav/frappe-bench/apps/frappe/frappe/model/document.py:1125:    def run_before_save_methods(self):
```

Ovo (`grep -R`) ćeš koristiti stalno kada budeš proučavao Frappe ili bilo koji veliki projekat.

```sh
sed -n '1125,1195p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def run_before_save_methods(self):
        """Run standard methods before    `INSERT` or `UPDATE`. Standard Methods are:

        - `validate`, `before_save` for **Save**.
        - `validate`, `before_submit` for **Submit**.
        - `before_cancel` for **Cancel**
        - `before_update_after_submit` for **Update after Submit**

        Will also update title_field if set"""

        self.reset_seen()

        # before_validate method should be executed before ignoring validations
        if self._action in ("save", "submit"):
            self.run_method("before_validate")

        if self.flags.ignore_validate:
            return

        if self._action == "save":
            self.run_method("validate")
            self.run_method("before_save")
        elif self._action == "submit":
            self.run_method("validate")
            self.run_method("before_submit")
        elif self._action == "cancel":
            self.run_method("before_cancel")
        elif self._action == "update_after_submit":
            self.run_method("before_update_after_submit")

        self.set_title_field()

    def run_post_save_methods(self):
        """Run standard methods after `INSERT` or `UPDATE`. Standard Methods are:

        - `on_update` for **Save**.
        - `on_update`, `on_submit` for **Submit**.
        - `on_cancel` for **Cancel**
        - `update_after_submit` for **Update after Submit**"""

        if self._action == "save":
            self.run_method("on_update")
        elif self._action == "submit":
            self.run_method("on_update")
            self.run_method("on_submit")
        elif self._action == "cancel":
            self.run_method("on_cancel")
            self.check_no_back_links_exist()
        elif self._action == "update_after_submit":
            self.run_method("on_update_after_submit")

        self.clear_cache()

        if self.flags.get("notify_update", True):
            self.notify_update()
```

##### run_method()

Prava slika je:

```text
insert() -> run_before_save_methods() -> run_method(...)
```

i

```text
run_post_save_methods() -> run_method(...)
```

Dakle, **run_before_save_methods()** i **run_post_save_methods()** su orkestratori životnog ciklusa.

Pogledaj komentar.

To nije običan komentar. To je praktično specifikacija Frappe-a.

```python
Save
    validate
    before_save

Submit
    validate
    before_submit

Cancel
    before_cancel

Update after submit
    before_update_after_submit
```

Odmah ispod toga vidiš implementaciju.

##### before_validate

Pogledaj ovo.

```python
if self._action in ("save", "submit"):
    self.run_method("before_validate")
```

Dakle postoji:

```text
before_validate
```

Mi ga do sada uopšte nismo pominjali.

To znači da je redosled:

```text
before_validate -> validate -> before_save -> 
```

**Zašto postoji before_validate?**

Recimo da korisnik nije uneo nešto što može automatski da se izračuna.

Na primer:

```text
full_name
```

iz:

```text
first_name
last_name
```

To možeš da uradiš ovde.

```python
def before_validate(self):

    self.full_name = ...
```

A onda:

```python
validate()
```

već proverava gotove podatke.

##### before_save

```python
validate() -> before_save()
```

Ovo je zanimljivo. Zašto prvo validate? Zato što `before_save()` više nije mesto za proveru podataka.

Po meni:

```text
validate = da li je dokument ispravan?
```

a

```text
before_save = sada kada znam da jeste, uradi završne pripreme
```

##### before_submit

Ovde mi se posebno sviđa dizajn.

```text
validate() -> before_submit()
```

Dakle... Submit nije posebna planeta. On takođe prolazi kroz validaciju.
To znači da pravila koja važe za dokument važe i prilikom submit-a.

To je veoma elegantno.

#### run_post_save_methods()

Posle baze. Ovde je sve mnogo jasnije.
  
##### save
  
```text
on_update
```
  
##### submit
  
```text
on_update -> on_submit
```

E ovo je interesantno.

`submit()` poziva i:

```text
on_update
```

i:

```text
on_submit
```

Dakle `submit()` je zapravo specijalan slučaj update-a. To mi ranije nije bilo očigledno.

##### cancel

Posle cancel-a:

```text
on_cancel
```

I odmah:

```python
check_no_back_links_exist()
```

To znači da framework odmah proverava referencijalni integritet.

Mislim da sada konačno možemo napraviti tabelu koju ćeš zaista koristiti.

| Događaj           | Kada se poziva                           | Tipična upotreba                                                      |
| ----------------- | ---------------------------------------- | --------------------------------------------------------------------- |
| `before_validate` | Pre svake validacije (`save`, `submit`)  | Popunjavanje ili normalizacija podataka pre provere.                  |
| `validate`        | Pre čuvanja i pre submit-a               | Provera poslovnih pravila, bacanje greške ako dokument nije ispravan. |
| `before_save`     | Samo kod običnog `save`                  | Završne pripreme pre upisa dokumenta.                                 |
| `before_submit`   | Neposredno pre `submit`                  | Provere koje važe samo pri predaji dokumenta.                         |
| `on_update`       | Posle uspešnog `save` (i tokom `submit`) | Reakcija na uspešno snimanje dokumenta.                               |
| `on_submit`       | Posle uspešnog `submit`                  | Radnje koje se izvršavaju samo jednom pri potvrđivanju dokumenta.     |
| `before_cancel`   | Pre otkazivanja                          | Provere pre otkazivanja.                                              |
| `on_cancel`       | Posle otkazivanja                        | Čišćenje, oslobađanje resursa, dodatne akcije.                        |

#### Kada se koristi

Controller ne pišeš zato što postoji `validate()`. Controller pišeš zato što je to prirodno mesto gde dokument definiše svoje ponašanje.

Na primer:

```python
class ServiceOrder(Document):

    def validate(self):
        ...

    def before_submit(self):
        ...

    def on_submit(self):
        ...
```

To je životni ciklus tog dokumenta.

Hook dolazi tek posle. Hook kaže: "Kada ovaj događaj nastupi, moja aplikacija želi još nešto da uradi."

Drugim rečima:

```text
Document -> Controller metoda -> run_method() -> Hook-ovi
```

#### run_method

```sh
grep -R -n "def run_method" ~/frappe-bench/apps/frappe/frappe/
```

```sh
/home/radosav/frappe-bench/apps/frappe/frappe/model/document.py:1002:    def run_method(self, method: str, *args, **kwargs):
```

```sh
sed -n '1002,1102p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def run_method(self, method: str, *args, **kwargs):
        """run standard triggers, plus those in hooks"""

        assert not method.startswith("__"), "Run method is for hooks, avoid usage on internal methods"

        def fn(self, *args, **kwargs):
            method_object = getattr(self, method, None)

            # Cannot have a field with same name as method
            # If method found in __dict__, expect it to be callable
            if method in self.__dict__ or callable(method_object):
                return method_object(*args, **kwargs)

        fn.__name__ = str(method)
        out = Document.hook(fn)(self, *args, **kwargs)

        self.run_notifications(method)
        run_webhooks(self, method)
        run_server_script_for_doc_event(self, method)

        return out

    def run_trigger(self, method, *args, **kwargs):
        return self.run_method(method, *args, **kwargs)

    def run_notifications(self, method):
        """Run notifications for this method"""
        if (
            (frappe.flags.in_import and frappe.flags.mute_emails)
            or frappe.flags.in_patch
            or frappe.flags.in_install
        ):
            return

        if self.flags.notifications_executed is None:
            self.flags.notifications_executed = []

        from frappe.email.doctype.notification.notification import evaluate_alert

        if self.flags.notifications is None:

            def _get_notifications():
                """returns enabled notifications for the current doctype"""

                return frappe.get_all(
                    "Notification",
                    fields=["name", "event", "method"],
                    filters={"enabled": 1, "document_type": self.doctype},
                )

            self.flags.notifications = frappe.cache.hget("notifications", self.doctype, _get_notifications)

        if not self.flags.notifications:
            return

        def _evaluate_alert(alert):
            if alert.name in self.flags.notifications_executed:
                return

            evaluate_alert(self, alert.name, alert.event)
            self.flags.notifications_executed.append(alert.name)

        event_map = {
            "on_update": "Save",
            "after_insert": "New",
            "on_submit": "Submit",
            "on_cancel": "Cancel",
        }

        if not self.flags.in_insert and not self.flags.in_delete:
            # value change is not applicable in insert
            event_map["on_change"] = "Value Change"

        for alert in self.flags.notifications:
            event = event_map.get(method, None)
            if event and alert.event == event:
                _evaluate_alert(alert)
            elif alert.event == "Method" and method == alert.method:
                _evaluate_alert(alert)

    def _submit(self):
        """Submit the document. Sets `docstatus` = 1, then saves."""
        self.docstatus = DocStatus.SUBMITTED
        return self.save()

    def _cancel(self):
        """Cancel the document. Sets `docstatus` = 2, then saves."""
        self.docstatus = DocStatus.CANCELLED
        return self.save()

    def _rename(self, name: str, merge: bool = False, force: bool = False, 
      validate_rename: bool = True):
         """Rename the document. Triggers frappe.rename_doc, then reloads."""
         from frappe.model.rename_doc import rename_doc
 
         self.name = rename_doc(doc=self, new=name, merge=merge, force=force, validate=validate_rename)
         self.reload()
 
    @frappe.whitelist()
    def submit(self):
        """Submit the document. Sets `docstatus` = 1, then saves."""
        return self._submit()
```

Hajde da je rastavimo.

##### run_method iznutra

Pogledaj koliko je `run_method()` zapravo jednostavan.

Suština je praktično ovo:

```python
method_object = getattr(self, method, None)

...

out = Document.hook(fn)(self, *args, **kwargs)

self.run_notifications(method)
run_webhooks(self, method)
run_server_script_for_doc_event(self, method)
```

Cela magija je u ove četiri linije.

- **Controller**

  ```python
  method_object = getattr(self, method, None)
  ```
  
  Ako si napisao:
  
  ```python
  class ServiceOrder(Document):
  
      def validate(self):
          ...
  ```
  
  onda će:
  
  ```python
  self.run_method("validate")
  ```
  
  naći upravo tu metodu.
  
  To smo i očekivali.

- **Hook**

  ```python
  Document.hook(fn)
  ```
  
  To znači da se ne poziva direktno
  
  ```python
  fn(self)
  ```
  
  nego:
  
  ```python
  Document.hook(...)
  ```
  
  Dakle... Ovde se krije ceo sistem hook-ova.
  
  Ja sam očekivao da će `run_method()` biti mesto gde se izvršavaju hook-ovi.
  Ali nije. On ih samo prosleđuje:
  
  ```text
  run_method() -> Document.hook()
  ```
  
  Po meni je to sledeća "crna kutija".
  
- **Posle Controllera**

  Tek kada Controller završi:
  
  ```python
  self.run_notifications(method)
  ```
  
  Dakle Notification nije deo Controller-a, nego dodatna usluga framework-a.

- **Webhook**

  Posle toga:
  
  ```python
  run_webhooks(...)
  ```
  
  Dakle potpuno isti događaj može automatski da ode na HTTP endpoint.
  
  Lepo.

- **Server Script**

  Na kraju:
  
  ```python
  run_server_script_for_doc_event(...)
  ```
  
  I ovo mi je bilo iznenađenje. Server Script nije zamena za Controller. Nije ni zamena za Hook.
  
  On je **još jedan potrošač događaja.**
  
  Drugim rečima: isti događaj može da pokrene:
  
  - Controller
  - Hook
  - Notification
  - Webhook
  - Server Script
  
  To je već ozbiljna arhitektura.

#### Pravi sled događaja

Po meni ona izgleda ovako.

```txt
insert() -> run_method("validate") -> Controller.validate() -> Hook (Document.hook) -> Notifications -> Webhooks -> Server Script
```

Ovde dolazimo do nečega što je meni promenilo razmišljanje.

**Hook nije konkurencija Controller-u**:

Ranije sam ih zamišljao ovako:

```text
Controller ili Hook
```

Sada vidim da nije tako.

Prava slika je:

```text
Controller -> Hook -> Notification -> Webhook -> Server Script
  ```
  
Svi oni učestvuju u obradi **istog događaja**.
  
U `run_method()` smo videli:

```python
out = Document.hook(fn)(self, *args, **kwargs)
```

To znači da je `hook` gotovo sigurno **dekorator**. On "umotava" poziv metode i oko njega dodaje izvršavanje hook-ova.

**Širu sliku toka izvršavanja**!

Mislim da je korisno da već sada znamo gde se nalazimo na "mapi" Frappe-a.

```txt
1. get_doc()                  ✔
2. get_controller()           ✔
3. import_controller()        ✔
4. BaseDocument               ✔
5. insert()/save()            ✔
6. run_before_save_methods()  ✔
7. run_post_save_methods()    ✔
8. run_method()               ✔
9. Document.hook()            ← sada
----------------------------------------
10. hooks.py (doc_events)
11. Pisanje sopstvenog Controller-a
12. Pisanje sopstvenog Hook-a
13. Životni ciklus na jednom realnom DocType-u
```

#### Document.hook

```sh
grep -n "def hook" ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```sh
1337:    def hook(f):
```

```sh
sed -n '1337,1378p' ~/frappe-bench/apps/frappe/frappe/model/document.py
```

```py
    def hook(f):
        """Decorator: Make method `hookable` (i.e. extensible by another app).

        Note: If each hooked method returns a value (dict), then all returns are
        collated in one dict and returned. Ideally, don't return values in hookable
        methods, set properties in the document."""

        def add_to_return_value(self, new_return_value):
            if new_return_value is None:
                self._return_value = self.get("_return_value")
                return

            if isinstance(new_return_value, dict):
                if not self.get("_return_value"):
                    self._return_value = {}
                self._return_value.update(new_return_value)
            else:
                self._return_value = new_return_value

        def compose(fn, *hooks):
            def runner(self, method, *args, **kwargs):
                add_to_return_value(self, fn(self, *args, **kwargs))
                for f in hooks:
                    add_to_return_value(self, f(self, method, *args, **kwargs))

                return self.__dict__.pop("_return_value", None)

            return runner

        def composer(self, *args, **kwargs):
            hooks = []
            method = f.__name__
            doc_events = frappe.get_doc_hooks()
            for handler in doc_events.get(self.doctype, {}).get(method, []) + doc_events.get("*", {}).get(
                method, []
            ):
                hooks.append(frappe.get_attr(handler))

            composed = compose(f, *hooks)
            return composed(self, method, *args, **kwargs)

        return composer
```

Ovde je ceo sistem događaja u 40 linija koda.

Pogledaj poslednji deo:

```python
doc_events = frappe.get_doc_hooks()

for handler in doc_events.get(self.doctype, {}).get(method, []) \
           + doc_events.get("*", {}).get(method, []):

    hooks.append(frappe.get_attr(handler))
```

Ovo je praktično odgovor na pitanje: Kako Frappe pronađe hook-ove?

Odgovor:

1. učita sve `doc_events`
2. pogleda da li postoje hook-ovi za taj DocType
3. pogleda da li postoje globalni (`"*"` )
4. importuje funkcije
5. izvrši ih

To je to.

##### Redosled izvršavanja

Ovo je najvažniji deo cele funkcije.

```python
add_to_return_value(
    fn(self, *args, **kwargs)
)

for f in hooks:

    add_to_return_value(
        f(self, method, *args, **kwargs)
    )
```

Dakle:

```text
Controller -> Hook 1 -> Hook 2 -> Hook 3
```

To više nije pretpostavka. To piše u kodu.

Šta to znači u praksi? Recimo da imaš:

```python
class SalesOrder(Document):

    def validate(self):

        print("Controller")
```

i u drugoj aplikaciji:

```python
doc_events = {

    "Sales Order": {
        "validate": "my_app.events.validate"
    }
}
```

onda će redosled biti:

```text
Controller -> my_app.events.validate
```

Nikada obrnuto.

##### Kada se koristi Controller a kada Hook

**Controller**:

Controller koristiš kada pišeš ponašanje samog DocType-a.

Na primer:

```python
class Invoice(Document):

    def validate(self):
        ...
```

To je logika koja pripada tom dokumentu.

Bez nje dokument nije potpun.

**Hook**:

Hook koristiš kada želiš da: "Dodaš ponašanje postojećem dokumentu, a da ga ne menjaš."

Na primer:

ERPNext već ima:

```txt
Sales Order
```

Ti nećeš menjati ERPNext.

Napisaćeš:

```python
doc_events
```

i zakačiti svoju logiku.

To je upravo razlog postojanja hook-ova.

Primeti da hook uopšte ne zna ništa o ERPNext-u.

On zna samo:

```python
doctype

method
```

Sve ostalo dolazi iz:

```python
hooks.py
```

To znači da je ceo sistem potpuno otvoren za proširenja.

#### Kompletna slika poziva

Ako pogledam unazad, krenuli smo od jedne jedine linije:

```python
doc = frappe.get_doc(...)
```

A završili smo sa kompletnom slikom:

```text
    get_doc() ->
    get_controller() ->
    import_controller() ->
    Controller ->
    insert()/save() ->
    run_before_save_methods() ->
    run_method() ->
    Controller metoda ->
    doc_events Hook ->
    Notification ->
    Webhook ->
    Server Script ->
    db_insert() ->
    run_post_save_methods()
```

To je ogromna količina znanja.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
