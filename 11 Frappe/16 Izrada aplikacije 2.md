
# Frappe Framework tutorijal

[Sadržaj][00]

## Izrada aplikacije 2

### Praktični rad

#### Nova aplikacija

```bash
cd ~/frappe-bench
bench new-app work_manager
```

Tokom kreiranja možeš ostaviti podrazumevane vrednosti ili uneti svoje (opis, autora, licencu...). To kasnije može da se promeni.

#### Instalacija na sajt

Kada se aplikacija napravi:

```bash
bench --site site1.local install-app work_manager
```

Ovime se aplikacija registruje na tom sajtu i izvršavaju se njene inicijalne migracije (ako ih ima).

#### Provera

Proveri da li Bench vidi aplikaciju:

```bash
bench --site site1.local list-apps
```

Trebalo bi da vidiš nešto poput:

```text
frappe
work_manager
frappe_lab
```

#### Kako Frappe vidi aplikaciju

Kada se korisnik prijavi, Frappe ne "skenira" proizvoljno direktorijume. Postoji prilično jasan lanac:

```text
    App
     │ 
     ▼ 
  Module
     │
     ▼
  Workspace
     │
     ▼
  Shortcut
     │
     ▼
  DocType / Report / Page
```

Drugim rečima:

- *App* - paket koji si instalirao (`work_manager`).
- *Module*- grupisanje funkcionalnosti unutar te aplikacije.
- *Workspace* - početna stranica koju korisnik vidi za taj modul.
  Na Workspace-u se nalaze prečice (Shortcut-i) ka DocType-ovima, izveštajima, stranicama i drugim sadržajima.

### wmProject DocType

Hajde sada da napravimo prvo jedan jednostavan DocType: `wmProject`.

Primetio si da je Frappe napravio:

```text
work_manager/
└── work_manager/
    └── doctype/
        └── wmproject/
            ├── wmproject.json
            ├── wmproject.py
            ├── wmproject.js
            └── test_wmproject.py
```

E ovo je trenutak kada se prazna struktura aplikacije počinje puniti. Svaki novi DocType će dobiti svoj direktorijum sa sopstvenim fajlovima.

Hajde da analiziramo ova četiri fajla, jer se upravo u njima nalazi filozofija Frappe-a.

**`wmproject.json`**

Ovo je najvažniji fajl. Mnogi početnici misle da je Python glavni. Nije.
Glavni je upravo JSON.

On opisuje:

- polja,
- tipove,
- dozvole,
- prikaz forme,
- listu,
- naming,
- itd.

Python je samo logika. Drugim rečima: DocType je prvenstveno metadata. To je jedna od najvećih razlika između Frappe-a i klasičnih Django/Flask aplikacija.

**`wmproject.py`**

Ovo je kontroler. Ovde smo već eksperimentisali sa:

- `validate`
- `before_insert`
- `after_insert`
- `on_update`

Sada ćemo ga koristiti za stvarnu poslovnu logiku.

**`wmproject.js`**

Ovo je klijentska logika. Do sada smo ga praktično ignorisali.  
Kasnije ćemo ovde raditi:

- događaje na formi,
- automatsko popunjavanje polja,
- pozive serveru (`frappe.call`),
- prilagođavanje UI-ja.

**`test_wmproject.py`**

Frappe ga pravi odmah. To je lep podsetnik da je svaki DocType zamišljen da ima i testove.

Pre nego što dodamo i jedno jedino polje, hajde da pogledamo `wmproject.json`, ne ceo fajl (ume da bude dug), već me zanimaju početni deo i osnovni atributi.

```sh
head -60 apps/work_manager/work_manager/work_manager/doctype/wmproject/wmproject.json
```

```json
{
 "actions": [],
 "allow_rename": 1,
 "creation": "2026-07-22 23:12:42.032714",
 "doctype": "DocType",
 "engine": "InnoDB",
 "field_order": [
  "section_break_a6e4"
 ],
 "fields": [
  {
   "fieldname": "section_break_a6e4",
   "fieldtype": "Section Break"
  }
 ],
 "grid_page_length": 50,
 "index_web_pages_for_search": 1,
 "links": [],
 "modified": "2026-07-22 23:12:42.032714",
 "modified_by": "Administrator",
 "module": "Work Manager",
 "name": "wmProject",
 "owner": "Administrator",
 "permissions": [
  {
   "create": 1,
   "delete": 1,
   "email": 1,
   "export": 1,
   "print": 1,
   "read": 1,
   "report": 1,
   "role": "System Manager",
   "share": 1,
   "write": 1
  }
 ],
 "row_format": "Dynamic",
 "rows_threshold_for_grid_search": 20,
 "sort_field": "modified",
 "sort_order": "DESC",
 "states": []
}
```

Ovo je kao definicija klase u klasičnom ORM-u (Django, SQLAlchemy...) napisao bi nešto poput:

```python
class Project(Model):
    name = CharField(...)
    ...
```

U Frappe-u je ekvivalent tome upravo ovaj JSON. Zato sam ranije rekao da je DocType prvenstveno metadata.

Hajde da ga čitamo.

- *"name": "wmProject"* je ime tipa dokumenta. Drugim rečima: *wmProject*
  je ekvivalent Python klasi.

- *"module": "Work Manager"* govori Frappe-u kom modulw ovaj DocType pripada.

- *"permissions": [...]*  je izuzetno zanimljivo. Primeti: Nismo napisali nijednu liniju Python-a. A već imamo ACL. To je jedna od najvećih prednosti Frappe-a.

- "fields": [ { "fieldname":"section_break..." } ] govori da trenutno praktično
  nema forme. Samo jedan `Section Break` koji je Desk automatski ubacio.

- *"doctype": "DocType"* - Mi trenutno uređujemo jedan Document tipa `DocType`.
  
Znači, `wmProject` nije "specijalan fajl". To je običan Document u tabeli `tabDocType`. To je razlog zašto možeš da ga otvoriš kroz Desk, menjaš, eksportuješ i verzionišeš. To je jedna od najlepših ideja u Frappe-u: sistem opisuje sam sebe pomoću sopstvenih dokumenata (meta-model).

#### Šta je zapravo `wmProject`

Da li je to:

- tabela?
- forma?
- Python klasa?
- REST resurs?
- API?
- JSON?
- UI?

Odgovor je: *Sve to istovremeno.*

Jedna definicija (`DocType`) postaje:

- tabela u bazi,
- Python objekat,
- forma,
- lista,
- REST endpoint,
- dozvole,
- pretraga,
- import/export,
- itd.

To znači da će svako polje koje dodamo imati posledice na više slojeva sistema odjednom.

#### wmProject kao poslovni objekat

Mislim da je došao trenutak da počnemo da projektujemo `wmProject` kao poslovni objekat, a ne kao tehnički DocType.

Ne bih odmah dodavao desetine polja.

Na primer, za prvu verziju možda nam je dovoljno ovakva mini specifikacija:

| Polje       | Tip        | Obavezno | Zašto postoji              |
| ----------- | ---------- | -------- | -------------------------- |
| Code        | Data       | Da       | Jedinstvena šifra projekta |
| Name        | Data       | Da       | Naziv projekta             |
| Description | Small Text | Ne       | Kratak opis                |
| Status      | Select     | Da       | Životni ciklus projekta    |
| Start Date  | Date       | Ne       | Planirani početak          |
| End Date    | Date       | Ne       | Planirani završetak        |

```sh
cat wmproject.json
```

```json
{
 "actions": [],
 "allow_rename": 1,
 "creation": "2026-07-22 23:12:42.032714",
 "doctype": "DocType",
 "engine": "InnoDB",
 "field_order": [
  "code",
  "project_name",
  "description",
  "status",
  "start_date",
  "end_date"
 ],
 "fields": [
  {
   "fieldname": "code",
   "fieldtype": "Data",
   "in_list_view": 1,
   "label": "Code",
   "reqd": 1
  },
  {
   "fieldname": "project_name",
   "fieldtype": "Data",
   "in_list_view": 1,
   "label": "Name",
   "reqd": 1
  },
  {
   "fieldname": "description",
   "fieldtype": "Small Text",
   "label": "Description"
  },
  {
   "fieldname": "status",
   "fieldtype": "Select",
   "in_list_view": 1,
   "label": "Status",
   "reqd": 1
  },
  {
   "fieldname": "start_date",
   "fieldtype": "Date",
   "label": "Start Date"
  },
  {
   "fieldname": "end_date",
   "fieldtype": "Date",
   "label": "End Date"
  }
 ],
 "grid_page_length": 50,
 "index_web_pages_for_search": 1,
 "links": [],
 "modified": "2026-07-23 01:38:09.753352",
 "modified_by": "Administrator",
 "module": "Work Manager",
 "name": "wmProject",
 "owner": "Administrator",
 "permissions": [
  {
   "create": 1,
   "delete": 1,
   "email": 1,
   "export": 1,
   "print": 1,
   "read": 1,
   "report": 1,
   "role": "System Manager",
   "share": 1,
   "write": 1
  }
 ],
 "row_format": "Dynamic",
 "rows_threshold_for_grid_search": 20,
 "sort_field": "modified",
 "sort_order": "DESC",
 "states": []
```

Prvo ćemo kritikovati ovaj model. To je potpuno normalan deo projektovanja.

**Prvo, tehnički**:

Sve je kako treba.

Frappe je u JSON upisao upravo ono što si uneo kroz UI. To potvrđuje ono što smo ranije pričali: UI nije "glavni". UI samo menja metapodatke, a metapodaci su sačuvani u `wmproject.json`.

To je veoma važan koncept.

**Sada da pogledamo model**:

- `code`

  Da li će korisnik unositi šifru ili će je sistem generisati? To nije tehničko pitanje nego poslovno. Na primer:
  
  ```txt
  WM-00001 ili PRJ-2026-001 ili KUPAC-001
  ```

  Ako će sistem da generiše šifru, onda nam možda ovo polje uopšte nije potrebno kao Data polje, već treba da koristimo *Naming Series* ili *autoname*.
  Ja bih za sada ipak ostavio `code`, pa ćemo kasnije odlučiti.

- `status`
  
  Ovde imamo problem.

  Video sam:
  
  ```json
  {
    "fieldtype": "Select"
  }
  ```

  Ali... nema opcija.

  Trebaće nešto poput:
  - Draft
  - Active
  - On Hold
  - Completed
  - Cancelled

  Kasnije ćemo pričati o tome da li status treba da bude običan `Select` ili da ga vodi **Workflow**.
  
  Za prvu verziju, `Select` je sasvim dobar izbor.

- **Datumi**
  
  Ovo je sasvim u redu. Ali već vidim prvu validaciju.

  Na primer:

  ```python
  if end_date < start_date:
      frappe.throw(...)
  ```

  Eto prve stvarne poslovne logike.

- Nedostaje jedno veoma važno polje.

  ```txt
  Customer
  ```

  Zašto? Zato što projekat skoro uvek postoji zbog nekoga. Ali... Pošto još nemamo `wmCustomer`, potpuno je ispravno da ga sada ne dodajemo.

  Samo beležim da će kasnije postati `Link`.

- Primetio sam:

  ```txt
  Description
  ```

  je `Small Text`. To mi se sviđa. Mnogi odmah stave `Text Editor`. Ja ne bih. `Text Editor` uvodi HTML, formatiranje, slike...

  Za opis projekta je `Small Text` često sasvim dovoljan.

#### Prvi zapis

Mislim da je došao trenutak da napravimo prvi zapis. Ne novi DocType. Nego jedan konkretan wmProject.

```txt
Code = WM-001
Name = Demo Project
Status = Draft
```

#### Primarni ključ u Frappeu

U Frappe-u je primarni ključ uvek polje `name`.

U tabeli ćeš imati nešto poput:

```text
name                ← PRIMARY KEY
owner
creation
modified
modified_by
docstatus
idx
...
code
project_name
status
start_date
...
```

Dakle:

`name` je PK. Uvek.

I to je jedna od fundamentalnih stvari u Frappe-u.

Zato mi se sviđa što si napravio `project_name`. Jer si praktično rekao: "Neću da mešam poslovni naziv projekta sa internim identifikatorom."

Predložio si: `Customer_Name.WMP.redni broj projekta.start_date`

To je već ozbiljno razmišljanje.

Na primer:

```text
ACME.WMP.00017.20260723
```

ili

```text
OPENAI.WMP.00003.20260723
```

Odmah se vidi:

- kupac
- aplikacija
- redni broj
- datum

Ali sada dolazi "zla baba". Postaviću pitanje koje bi ti postavio arhitekta na code review-u.

- Šta ako kupac promeni ime?
- Šta ako postoje dva kupca istog imena
- Šta ako kupac bude obrisan?
- Šta ako projekat promeni vlasnika?

Vidiš šta radim? Ne kritikujem ideju. Pokušavam da je "slomim". Ako preživi sva ova pitanja... onda je dobra.

Kako se ovo obično rešava?

#### Poslovni identitet

Obično postoji razlika između:

- Poslovne šifre, na primer: ACME-WMP-0017, to korisnici vide i
- Primarnog ključa (`name`), na primer: WMP-000000231. To koristi Frappe.

To ne mora biti isto.

I upravo je to jedna od stvari koju ćemo kasnije istražiti kroz *Naming Series* i *autoname*.

Za prvu verziju bih bio vrlo dosadan.

Koristio bih:

```text
name (PK):

WMP-.#####
```

što daje:

```text
WMP-00001
WMP-00002
WMP-00003
```

A kada budemo napravili `wmCustomer`, vratićemo se na tvoju ideju i videćemo kako da napravimo nešto poput:

```text
ACME.WMP.0023.20260723
```

ili još elegantnije, koristeći Frappe-ov `autoname` mehanizam.

Ne zato što je tvoja ideja loša, nego zato što ćemo tada imati sve potrebne informacije (Customer, datum, pravila imenovanja) da ga implementiramo kako treba.

Napravimo nekoliko testnih projekata, na primer:

| Code   | Name               | Status  |
| ------ | ------------------ | ------- |
| WM-001 | Izrada ERP         | Draft   |
| WM-002 | CRM migracija      | Active  |
| WM-003 | Mobilna aplikacija | On Hold |

Zašto? Jer bez podataka ne možemo ozbiljno da testiramo list view, filtere, pretragu i kasnije relacije.

#### Pogled u bazu

```sh
bench --site site1.local psql
```

```sql
_c9eb2d89e08e8728=> \d "tabwmProject"

                             Table "public.tabwmProject"
    Column    |              Type              | Collation | Nullable |    Default    
--------------+--------------------------------+-----------+----------+---------------
 name         | character varying(140)         |           | not null | 
 creation     | timestamp(6) without time zone |           |          | 
 modified     | timestamp(6) without time zone |           |          | 
 modified_by  | character varying(140)         |           |          | 
 owner        | character varying(140)         |           |          | 
 docstatus    | smallint                       |           | not null | '0'::smallint
 idx          | bigint                         |           | not null | '0'::bigint
 _user_tags   | text                           |           |          | 
 _comments    | text                           |           |          | 
 _assign      | text                           |           |          | 
 _liked_by    | text                           |           |          | 
 code         | character varying(140)         |           |          | 
 project_name | character varying(140)         |           |          | 
 description  | text                           |           |          | 
 status       | character varying(140)         |           |          | 
 start_date   | date                           |           |          | 
 end_date     | date                           |           |          | 
Indexes:
    "tabwmProject_pkey" PRIMARY KEY, btree (name)
```

#### Validacija datuma

U wmProject.py dodaj:

```py
def validate(self):
    if self.end_date and self.start_date:
        if self.end_date < self.start_date:
            frappe.throw("End Date must be after Start Date")
```

#### Podešavanje ListViewa

Kako da uklonim iz ListView-a kolonu primarnog ključa?

Imaš više načina:

- **Customize Form**

  Idi:
  
  ```txt
  Awesome Bar -> Customize Form -> wmProject
  ```
  
  Na dnu pogledaj:
  
  ```txt
  List View Settings
  ```
  
  ili:
  
  ```txt
  In List View
  ```
  
  Tu možeš da odrediš koja polja se prikazuju.
  
  Ali važno: **name* - nije normalno polje koje možeš da skloniš kao ostala polja.
  
- **DocType JSON-a (bolje za development)**
  
  U tvom slučaju, pošto radiš aplikaciju u developer modu, idi na:
  
  ```bash
  cd ~/frappe-bench
  ```
  
  i otvori:
  
  ```bash
  apps/work_manager/work_manager/work_manager/doctype/wmproject/wmproject.json
  ```
  
  Pogledaj da li imaš:
  
  ```json
  "sort_field": "modified",
  "sort_order": "DESC"
  ```
  
  i dodaj (ili promeni):
  
  ```json
  "show_name_in_global_search": 0
  ```
  
- **List View kolone**
  
  Pravi mehanizam je u JS fajlu.
  
  Napravi:
  
  ```txt
  wmproject_list.js
  ```
  
  u istom direktorijumu:
  
  ```txt
  wmproject/
   ├── wmproject.py
   ├── wmproject.json
   └── wmproject_list.js
  ```
  
  sa sledećim sadržajem:
  
  ```javascript
  frappe.listview_settings['wmProject'] = {
      hide_name_column: true
  };
  ```
  
  Posle:
  
  ```bash
  bench --site site1.local clear-cache
  ```
  
  i refresh browser.

### Zaključak

Frappe je otišao daleko u smeru **ERP platforme**, pa zato vuče dosta "tereta":

- Desk,
- Role Permission Manager,
- Workflow,
- Doc Events,
- Reports,
- Print Formats,
- Notifications,
- Background jobs,
- Multi-tenancy,
- REST API,
- Hooks,
- Apps kao paketi.

Cena toga je baš ono što si primetio: ponekad deluje kao da se framework "bori protiv tebe".

Posebno oko stvari kao što su:

- `name` kao primarni identifikator,
- način imenovanja DocType-a,
- Workspace filozofija,
- jaka konvencija strukture aplikacije.

To su odluke koje daju konzistentnost ogromnom sistemu, ali umeju da iritiraju nekoga ko dolazi iz klasičnijeg razmišljanja baze podataka.

Slažem se sa tvojom završnom procenom: Frappe ima mnogo urađenih stvari i vrlo ozbiljnu infrastrukturu, ali nosi i određene filozofske odluke koje nisu za svakoga. Kod ovakvih framework-a nije pitanje da li su "dobri" ili "loši", nego da li se njihov način razmišljanja poklapa sa problemima koje rešavaš.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
