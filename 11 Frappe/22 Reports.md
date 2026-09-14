# Frappe Framework tutorijal

[Sadrzaj][00]

## 22 Reports

U ovom poglavlju pravimo prve izveštaje nad podacima koje vec imas (`wmTask` + `wmTimeEntry`).

Cilj:

1. da vidiš osnovne tipove report-a u Frappe-u,
2. da napraviš prvi korisni agregat (`ukupno sati po task-u`),
3. da dodaš drugi report (`sati po zaposlenom u periodu`).

### Tri tipa reporta

1. **Report Builder**

   - najbrži za pregled podataka,
   - bez koda,
   - dobar za ad-hoc filtere i listanje.

2. **Query Report**

   - SQL upit,
   - brz i jasan kada imas agregacije (`SUM`, `GROUP BY`),
   - idealan za operativne izvestaje.

3. **Script Report**

   - Python `execute(filters)` + opciono SQL,
   - najfleksibilniji,
   - najbolji kada hoces validaciju filtera i malo poslovne logike.

Za tvoj nivo sada: kreni od Query/Script report-a sa jednostavnim agregacijama.

### Kako se ovo gradi u konkretnoj aplikaciji

U jednoj realnoj aplikaciji to ide ovako:

1. prvo napraviš osnovne DocType-e, na primer `wmTask` i `wmTimeEntry`,
2. zatim napraviš report koji sumira sate po task-u ili po zaposlenom,
3. taj report povežeš sa Workspace-om ili sa stranice za menadžera,
4. na kraju ga koristiš da vidiš ko je koliko radio i koji taskovi su najopterećeniji.

U praksi, report nije samo "SQL izlaz". On je deo poslovnog alata koji pomaže korisniku da donese odluku.

### QueryReport

#### Ukupno sati po Task-u

- Kreiranje QueryReporta u Desk-u

  - Otvori `Report List` -> `New`.
  - Report Name: `wm Task Hours Summary`.
  - Ref DocType: `wmTask`.
  - Report Type: `Query Report` (ili `Script Report`, vidi dole).
  - Module: `Work Manager`.
  - Is Standard: `Yes` (ako si u developer mode-u).
  
  Ako biras `Query Report`, unesi SQL ispod.

  **SQL primer (child tabela wmTimeEntry pod wmTask)**
  
  **Napomena**: ako si DocType nazvao sa razmakom (`wmTime Entry`), tabela moze biti drugacija. U tom slucaju proveri tacan naziv u bazi/metadata.

  ```sql
  SELECT
      t.name AS task,
      t.task_title AS task_title,
      t.status AS task_status,
      COALESCE(SUM(te.hours), 0) AS total_hours
  FROM `tabwmTask` t
  LEFT JOIN `tabwmTimeEntry` te
      ON te.parent = t.name
      AND te.parenttype = 'wmTask'
  GROUP BY t.name, t.task_title, t.status
  ORDER BY total_hours DESC, t.modified DESC
  ```

### Script Report

#### Sati po zaposlenom u periodu

Ovaj report je koristan za timesheet pregled.

**Predlog filtera**:

1. `from_date` (Date, obavezno)
2. `to_date` (Date, obavezno)
3. `employee` (Link -> wmEmployee, opciono)

Napravićemo ga iz sourcea:

```py
import frappe
from frappe.utils import getdate

def execute(filters=None):
    filters = filters or {}

    # Izlazne kolone izveštaja
    columns = [
        {
            "label": "Employee",
            "fieldname": "employee",
            "fieldtype": "Link",
            "options": "wmEmployee",
            "width": 180,
        },
        {
            "label": "Entries",
            "fieldname": "entries",
            "fieldtype": "Int",
            "width": 100,
        },
        {
            "label": "Total Hours",
            "fieldname": "total_hours",
            "fieldtype": "Float",
            "width": 120,
        },
    ]

    # vrednosti za filtere iz filter forme
    from_date = filters.get("from_date")
    to_date = filters.get("to_date")
    employee = filters.get("employee")

    # Prvo ucitavanje report-a: bez greske, samo prazan rezultat.
    if not from_date or not to_date:
        return columns, []

    from_date_obj = getdate(from_date)
    to_date_obj = getdate(to_date)

    if from_date_obj > to_date_obj:
        frappe.throw("From date ne moze biti posle To date.")

    where_employee = ""
    params = {
        "from_date": from_date_obj,
        "to_date": to_date_obj,
    }

    if employee:
        where_employee = " AND te.employee = %(employee)s"
        params["employee"] = employee

    data = frappe.db.sql(
        """
        SELECT
            te.employee AS employee,
            COUNT(*) AS entries,
            COALESCE(SUM(te.hours), 0) AS total_hours
        FROM `tabwmTimeEntry` te
        WHERE te.entry_date BETWEEN %(from_date)s AND %(to_date)s
        """
        + where_employee + 
        """
        GROUP BY te.employee
        ORDER BY total_hours DESC
        """,
        params,
        as_dict=True,
    )

    return columns, data
```

I treba nam .js fajl

```js
frappe.query_reports["Hours by Employee and Time period"] = {
    filters: [
        {        
            fieldname: "from_date",
            label: "From date",
            fieldtype: "Date",
            reqd: 1,
            default: frappe.datetime.month_start()
        },
        {
            fieldname: "to_date",
            label: "To date",
            fieldtype: "Date",
            reqd: 1,
            default: frappe.datetime.get_today()
        },
        {
            fieldname: "employee",
            label: "Employee",
            fieldtype: "Link",
            options: "wmEmployee"
        }
    ]
};
```

### Brza validacija report-a

Nakon snimanja report-a proveri:

1. prikazuje li i task-ove bez time entry zapisa (`total_hours = 0`),
2. da li zbir sati odgovara manualnoj proveri jednog task-a,
3. da li se report otvara bez SQL error-a,
4. da li role koje treba imaju pristup report-u.

### Najčcećce greške

1. Pogrešan naziv SQL tabele (`tabwmTimeEntry` vs `tabwm Time Entry`).
2. Zaboravljen `parenttype = 'wmTask'` uslov kod child tabele.
3. Null vrednosti bez `COALESCE`, pa dobiješ prazno umesto `0`.
4. Filter parametri nisu poslati ili nisu istog imena kao u SQL-u.

[Sadrzaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
