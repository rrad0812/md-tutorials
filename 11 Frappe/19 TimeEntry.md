# Frappe Framework tutorijal

[Sadržaj][00]

## 19 Time Entry

Sve relacije iz prethodnog poglavlja su spremne, pa sada uvodimo `wmTime Entry` kao evidenciju rada po zadatku.

Cilj je da dobiješ:

1. vezu ka `wmTask`, `wmEmployee` i `wmProject`,
2. jasnu validaciju sati i statusa,
3. bazu za kasnije agregacije i izveštaje.

### Minimalna specifikacija za wmTime Entry

| Polje | Tip | Obavezno | Napomena |
| ----- | --- | -------- | -------- |
| task | Link | Da | Options = `wmTask` |
| employee | Link | Da | Options = `wmEmployee` |
| project | Link | Da | Options = `wmProject`, read-only |
| entry_date | Date | Da | Podrazumevano danas |
| hours | Float | Da | Broj sati rada |
| billable | Check | Ne | Naplativo vreme |
| description | Small Text | Ne | Kratak opis rada |

### Kreiranje DocType-a

U Desk-u kreiraj novi DocType:

1. Name: `wmTime Entry`
2. Module: `Work Manager`
3. Is Submittable: `No`
4. Is Child Table: `No`

### Kompletan primer validacije u wmtime_entry.py

```python
import frappe
from frappe.model.document import Document


class wmTimeEntry(Document):
   def validate(self):
      self.validate_required_links()
      self.set_project_from_task()
      self.validate_hours()
      self.validate_employee_active()
      self.validate_task_and_project_state()

   def validate_required_links(self):
      if not self.task:
         frappe.throw("Task is required")
      if not self.employee:
         frappe.throw("Employee is required")

   def set_project_from_task(self):
      task_project = frappe.db.get_value("wmTask", self.task, "project")

      if not task_project:
         frappe.throw("Selected Task must be linked to a Project")

      if self.project and self.project != task_project:
         frappe.throw("Project must match Task's Project")

      self.project = task_project

   def validate_hours(self):
      if self.hours is None:
         frappe.throw("Hours is required")

      if self.hours <= 0:
         frappe.throw("Hours must be greater than 0")

      if self.hours > 24:
         frappe.throw("Hours cannot be greater than 24 for one day")

   def validate_employee_active(self):
      is_active = frappe.db.get_value("wmEmployee", self.employee, "is_active")

      if not is_active:
         frappe.throw("Cannot log time for inactive Employee")

   def validate_task_and_project_state(self):
      task_status = frappe.db.get_value("wmTask", self.task, "status")
      if task_status == "Cancelled":
         frappe.throw("Cannot log time on Cancelled Task")

      project_status, project_end_date = frappe.db.get_value(
         "wmProject", self.project, ["status", "end_date"]
      )

      if project_status == "Cancelled":
         frappe.throw("Cannot log time on Cancelled Project")

      # Soft rule for completed projects: entry date can exist, but not after end date.
      if (
         project_status == "Completed"
         and self.entry_date
         and project_end_date
         and self.entry_date > project_end_date
      ):
         frappe.throw("Entry Date cannot be after Project End Date for Completed Project")
```

### List View (preporuka)

U listi prikaži:

1. `entry_date`
2. `employee`
3. `task`
4. `project`
5. `hours`
6. `billable`

Opcioni indikator po satima:

```javascript
frappe.listview_settings['wmTime Entry'] = {
  get_indicator: function (doc) {
   if (doc.hours > 8) return ['Heavy', 'red', 'hours,>,8'];
   if (doc.hours >= 4) return ['Medium', 'orange', 'hours,between,[4,8]'];
   return ['Light', 'green', 'hours,<,4'];
  }
};
```

### Ručni test scenario

1. Validan unos (`task`, `employee`, `hours=2.5`) -> prolazi.
2. `hours=0` -> pada.
3. `hours=25` -> pada.
4. Task bez projekta -> pada.
5. Cancelled task -> pada.
6. Inactive employee -> pada.
7. Completed project + `entry_date` posle `end_date` -> pada.

### Automatski testovi (primer za wmTime Entry)

Ako do sada nisi radio testove, ovo je najjednostavniji obrazac koji možeš da kopiraš i prilagodiš.

Predložena lokacija fajla:

`apps/work_manager/work_manager/work_manager/doctype/wmtimeentry/test_wmtimeentry.py`

```python
import frappe
from frappe.tests.utils import FrappeTestCase
from frappe.utils import add_days, nowdate


class TestWMTimeEntry(FrappeTestCase):
   def setUp(self):
      self.customer = self._make_customer()
      self.project = self._make_project(status="Open")
      self.employee = self._make_employee(is_active=1)
      self.task = self._make_task(project=self.project.name, status="Open")

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
            "end_date": add_days(nowdate(), 10),
         }
      ).insert(ignore_permissions=True)

   def _make_user(self):
      email = f"test-employee-{frappe.generate_hash(length=8)}@example.com"
      return frappe.get_doc(
         {
            "doctype": "User",
            "email": email,
            "first_name": "Test",
            "enabled": 1,
         }
      ).insert(ignore_permissions=True)

   def _make_employee(self, is_active=1):
      user = self._make_user()
      return frappe.get_doc(
         {
            "doctype": "wmEmployee",
            "employee_name": "Test Employee",
            "user": user.name,
            "is_active": is_active,
         }
      ).insert(ignore_permissions=True)

   def _make_task(self, project, status="Open"):
      return frappe.get_doc(
         {
            "doctype": "wmTask",
            "task_name": "Test Task",
            "project": project,
            "status": status,
         }
      ).insert(ignore_permissions=True)

   def test_valid_time_entry_is_created(self):
      entry = frappe.get_doc(
         {
            "doctype": "wmTime Entry",
            "task": self.task.name,
            "employee": self.employee.name,
            "entry_date": nowdate(),
            "hours": 2.5,
            "billable": 1,
         }
      ).insert(ignore_permissions=True)

      self.assertEqual(entry.project, self.project.name)

   def test_hours_must_be_greater_than_zero(self):
      with self.assertRaises(frappe.ValidationError):
         frappe.get_doc(
            {
               "doctype": "wmTime Entry",
               "task": self.task.name,
               "employee": self.employee.name,
               "entry_date": nowdate(),
               "hours": 0,
            }
         ).insert(ignore_permissions=True)

   def test_hours_cannot_exceed_24(self):
      with self.assertRaises(frappe.ValidationError):
         frappe.get_doc(
            {
               "doctype": "wmTime Entry",
               "task": self.task.name,
               "employee": self.employee.name,
               "entry_date": nowdate(),
               "hours": 25,
            }
         ).insert(ignore_permissions=True)

   def test_cancelled_task_blocks_time_entry(self):
      cancelled_task = self._make_task(project=self.project.name, status="Cancelled")

      with self.assertRaises(frappe.ValidationError):
         frappe.get_doc(
            {
               "doctype": "wmTime Entry",
               "task": cancelled_task.name,
               "employee": self.employee.name,
               "entry_date": nowdate(),
               "hours": 1,
            }
         ).insert(ignore_permissions=True)

   def test_inactive_employee_blocks_time_entry(self):
      inactive_employee = self._make_employee(is_active=0)

      with self.assertRaises(frappe.ValidationError):
         frappe.get_doc(
            {
               "doctype": "wmTime Entry",
               "task": self.task.name,
               "employee": inactive_employee.name,
               "entry_date": nowdate(),
               "hours": 1,
            }
         ).insert(ignore_permissions=True)
```

Pokretanje testova (kada app struktura postoji):

```bash
bench --site <tvoj_sajt> run-tests --module work_manager.work_manager.doctype.wmtimeentry.test_wmtimeentry
```

Ako želiš da proveriš sve testove iz app:

```bash
bench --site <tvoj_sajt> run-tests --app work_manager
```

Ovim dobijaš šablon koji samo preslikaš na `wmTask`, `wmEmployee` i `wmProject`.

### Kada chil_table a kada standalone DocType

Evo 5 kratkih pravila baš za tvoj Work Manager:

1. Koristi child table kada su stavke “detalji parent-a”
   Primer: checklist na task-u, mali dnevnik promena, redovi koji nemaju život van task-a.

2. Koristi standalone DocType kada želiš ozbiljan rad sa podacima
   Ako za `Time Entry` planiraš pretragu, izveštaje, dozvole, workflow, import/export, audit, idi na poseban DocType.

3. Ako zapis može da postoji samostalno, ne treba da bude child
   `Time Entry` obično ima svoj identitet (ko, kad, koliko sati), pa je najčešće bolje da bude standalone.

4. Izbegni dupliranje podataka osim ako donosi jasnu korist
   Ako `project` može da se izvede iz `task`, drži ga samo na jednom mestu. Ako ga ipak držiš i na `Time Entry`, auto-popuni iz `task` i zaključaj edit da ne dođe do mismatch-a.

5. Razmišljaj unapred o skali
   Za 10-50 unosa child table je ok.
   Za stotine/hiljade sati i mesečne izveštaje: standalone `wmTime Entry` + Link ka `wmTask` je dugoročno bolje.

Za tvoj slučaj: krenuo bih sa standalone `wmTime Entry` i ostavio child varijantu samo ako ti treba brz unos direktno iz task forme.  

Ako želiš, sledeće mogu da ti dam “migracioni plan” iz child u standalone bez gubitka podataka, korak po korak.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
