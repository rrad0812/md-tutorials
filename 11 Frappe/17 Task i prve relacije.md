# Frappe Framework tutorijal

[Sadržaj][00]

## 17 Task i prve relacije

U prethodnom delu smo stabilizovali `wmProject`. Sada prelazimo na prvi DocType koji uvodi relaciju: `wmTask`.

Cilj ovog poglavlja je da dobiješ:

1. Relaciju `Task -> Project`.
2. Prvu realnu poslovnu validaciju između dva DocType-a.
3. Osnovu za kasnije `Time Entry` i izveštaje.

`Task` je idealan sledeći korak jer uvodi:

- `Link` polje,
- assignment logiku,
- statusni tok,
- filtere po projektu.

To je dovoljno složeno da nešto naučimo, a dovoljno malo da ostanemo fokusirani.

### Minimalna specifikacija za wmTask

Predlog za prvu verziju:

| Polje         | Tip        | Obavezno | Napomena                                  |
| ------------- | ---------- | -------- | ----------------------------------------- |
| task_name     | Data       | Da       | Kratak naziv zadatka                      |
| project       | Link       | Da       | Link ka `wmProject`                       |
| status        | Select     | Da       | Draft, Open, In Progress, Done, Cancelled |
| priority      | Select     | Ne       | Low, Medium, High                         |
| assigned_to   | Link       | Ne       | Link ka `wmEmployee`                      |
| due_date      | Date       | Ne       | Planirani rok                             |
| description   | Small Text | Ne       | Kratak opis                               |

**Napomena**: nemoj koristiti Child Table za task, već zaseban DocType.

### Kratko o preduslovu: `wmEmployee`

Ako ti nedostaje zaposleni za dodelu taska, nemoj direktno graditi relaciju ka internom `User` u poslovnoj logici. Praktičnije je da napraviš svoj DocType `wmEmployee`, pa da on ima vezu ka `User`.

Minimalni `wmEmployee`:

| Polje | Tip | Obavezno | Napomena |
| ----- | --- | -------- | -------- |
| employee_name | Data | Da | Ime zaposlenog |
| user | Link | Da | Link ka sistemskom DocType `User` |
| is_active | Check | Ne | Aktivni/inaktivni zaposleni |

Time dobijaš čist model:

- `wmTask.assigned_to -> wmEmployee`
- `wmEmployee.user -> User`

Ovako kasnije lako dodaješ odeljenje, satnicu, tim i druge HR atribute bez menjanja `wmTask` strukture.

### Kreiranje DocType-a

U Desk-u kreiraj novi DocType:

- Name: `wmTask`
- Module: `Work Manager`
- Is Submittable: `No`
- Is Child Table: `No`

Dodaj navedena polja i za `status` odmah upiši opcije:

```text
Draft
Open
In Progress
Done
Cancelled
```

Posle čuvanja uradi export i proveri da je JSON nastao na očekivanoj lokaciji.

### Validacija u wmtask.py

Dodaj osnovnu validaciju datuma i zaštitu statusa:

```python
import frappe
from frappe.model.document import Document

class wmTask(Document):
    def validate(self):
        self.validate_due_date()
        self.validate_project_state()

    def validate_due_date(self):
        if self.due_date and self.project:
            project_start, project_end = frappe.db.get_value(
                "wmProject", self.project, ["start_date", "end_date"]
            )
            if project_start and self.due_date < project_start:
                frappe.throw("Due Date cannot be before Project Start Date")
            if project_end and self.due_date > project_end:
                frappe.throw("Due Date cannot be after Project End Date")

    def validate_project_state(self):
        if not self.project:
            return

        project_status = frappe.db.get_value("wmProject", self.project, "status")
        if project_status in {"Completed", "Cancelled"} and self.status in {
            "Open",
            "In Progress",
        }:
            frappe.throw(
                "Task cannot be Open/In Progress when project is Completed or Cancelled"
            )
```

Ovo je dobra početna logika: mala, jasna i odmah korisna.

### List View podešavanje

U `wmtask.json` postavi da se u listi vide:

- `task_name`
- `project`
- `status`
- `priority`
- `due_date`

Ako želiš još čitljiviji prikaz, dodaj i `wmtask_list.js` sa indikatorom po statusu.

Primer:

```javascript
frappe.listview_settings['wmTask'] = {
  get_indicator: function (doc) {
    if (doc.status === 'Done') return ['Done', 'green', 'status,=,Done'];
    if (doc.status === 'In Progress') return ['In Progress', 'orange', 'status,=,In Progress'];
    if (doc.status === 'Cancelled') return ['Cancelled', 'red', 'status,=,Cancelled'];
    if (doc.status === 'Late') return ['Late', 'red', 'status,=,Late'];

    // fallback: prikazi stvarni status, ne hardkodiran Open
    return [doc.status || 'Open', 'blue', `status,=,${doc.status || 'Open'}`];
  }
};
```

### Test scenario

Napravi 4 testa ručno kroz Desk:

1. Task bez projekta -> treba da padne (project je obavezan).
2. Task sa `due_date` pre početka projekta -> treba da padne.
3. Task sa validnim datumom i statusom `Open` -> treba da prođe.
4. Task `In Progress` na projektu `Completed` -> treba da padne.

Ako ova 4 slučaja rade, model je zdrav i spreman za sledeći korak.

### Nedostaje wmEmployee DocType

Tačno si prepoznao problem: za poslovni model je bolje da ne kačiš zadatak direktno na User, nego na Employee sloj.

Šta je izmenjeno:

1. Polje `assigned_to` u `wmTask` sada je definisano kao `Link` ka `wmEmployee` (umesto `User`).
2. Dodata je sekcija sa minimalnim preduslovom za `wmEmployee`.
3. Jasno je postavljen lanac relacija:  
   `wmTask.assigned_to` -> `wmEmployee`  
   `wmEmployee.user` -> `User`

Praktično u Frappe Desk sada uradi:

1. Kreiraj DocType `wmEmployee` sa poljima
   - `employee_name` (Data),
   - `user` (Link -> User),
   - `is_active` (Check).
2. U `wmTask` polju `assigned_to` postavi `Options` na `wmEmployee`.
3. Napravi bar jedan `wmEmployee` zapis povezan na postojeći sistemski `User`.
4. Testiraj kreiranje taska i dodelu preko `assigned_to`.

**Napomena**: `User` DocType je sistemski i već postoji u Frappe-u, zato ga ne praviš kao custom DocType. Custom praviš `wmEmployee`.

### Automatski testovi (primer za wmTask)

Da ne ostane samo na ručnom klikanju kroz Desk, ispod je minimalan test primer za `wmTask`.

Predložena lokacija fajla:

`apps/work_manager/work_manager/work_manager/doctype/wmtask/test_wmtask.py`

```python
import frappe
from frappe.tests.utils import FrappeTestCase
from frappe.utils import add_days, nowdate


class TestWMTask(FrappeTestCase):
    
    def setUp(self):
        self.customer = self._make_customer()
        self.project_open = self._make_project(status="Open")
        self.project_completed = self._make_project(status="Completed")

    def _make_customer(self):
        return frappe.get_doc(
            {
                "doctype": "wmCustomer",
                "customer_name": f"Test Customer {frappe.generate_hash(length=6)}",
            }
        ).insert(ignore_permissions=True)

    def _make_project(self, status="Open"):
        return frappe.get_doc(
            {
                "doctype": "wmProject",
                "project_name": f"Test Project {frappe.generate_hash(length=6)}",
                "customer": self.customer.name,
                "status": status,
                "start_date": nowdate(),
                "end_date": add_days(nowdate(), 7),
            }
        ).insert(ignore_permissions=True)

    def test_valid_task_is_created(self):
        task = frappe.get_doc(
            {
                "doctype": "wmTask",
                "task_name": "Valid Task",
                "project": self.project_open.name,
                "status": "Open",
                "due_date": nowdate(),
            }
        ).insert(ignore_permissions=True)

        self.assertEqual(task.project, self.project_open.name)
        self.assertEqual(task.status, "Open")

    def test_due_date_cannot_be_before_project_start(self):
        with self.assertRaises(frappe.ValidationError):
            frappe.get_doc(
                {
                    "doctype": "wmTask",
                    "task_name": "Too Early",
                    "project": self.project_open.name,
                    "status": "Open",
                    "due_date": add_days(self.project_open.start_date, -1),
                }
            ).insert(ignore_permissions=True)

    def test_due_date_cannot_be_after_project_end(self):
        with self.assertRaises(frappe.ValidationError):
            frappe.get_doc(
                {
                    "doctype": "wmTask",
                    "task_name": "Too Late",
                    "project": self.project_open.name,
                    "status": "Open",
                    "due_date": add_days(self.project_open.end_date, 1),
                }
            ).insert(ignore_permissions=True)

    def test_open_task_not_allowed_on_completed_project(self):
        with self.assertRaises(frappe.ValidationError):
            frappe.get_doc(
                {
                    "doctype": "wmTask",
                    "task_name": "Blocked by project state",
                    "project": self.project_completed.name,
                    "status": "Open",
                    "due_date": nowdate(),
                }
            ).insert(ignore_permissions=True)
```

Napomena za ovaj primer:

1. Name se ne postavlja ručno; sistem ga generiše po šablonu tipa `WMX-.#####`.
2. U test podacima popunjavaš title polja: `customer_name`, `project_name`, `task_name`.
3. Link polja i dalje vezuješ preko `.name` vrednosti (npr. `self.customer.name`).

Pokretanje testova (kada app struktura postoji):

```bash
bench --site <tvoj_sajt> run-tests --module work_manager.work_manager.doctype.wmtask.test_wmtask
```

### Kako čitaš grešku kada test padne

Kada test padne, gledaj redom ove 3 stvari:

1. Naziv testa koji je pao (npr. `test_due_date_cannot_be_after_project_end`).
2. Tip greške (`AssertionError`, `frappe.ValidationError`, itd).
3. Poslednju poruku u traceback-u (to je obično prava poruka iz `frappe.throw(...)`).

Brzi workflow za debug:

1. Pokreni test modul.
2. Nađi prvi pali test (ne rešavaj sve odjednom).
3. Uporedi očekivanje iz testa sa porukom/ponašanjem iz validacije.
4. Ispravi ili test ili poslovno pravilo (ako je pravilo menjano).
5. Ponovo pokreni isti modul dok ne prođe.

Najčešći uzroci u ovom poglavlju:

1. `wmProject` ima dodatna obavezna polja koja nisu popunjena u `_make_project`.
2. Status vrednosti nisu iste kao u Select opcijama (npr. `Completed` vs `Complete`).
3. Datum u testu nije u opsegu `start_date` - `end_date`.

Ako želiš detaljniji izlaz testa, koristi:

```bash
bench --site <tvoj_sajt> run-tests --module work_manager.work_manager.doctype.wmtask.test_wmtask --verbose
```

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
