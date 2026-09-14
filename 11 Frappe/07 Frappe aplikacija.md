
# Frappe framework tutorijal

[Sadržaj][00]

## 07 Frappe aplikacija

U `apps/` trenutno postoji samo:

```txt
frappe/
```

To je ogromna aplikacija.

I to nije Python paket. Vrlo verovatno videti nešto poput:

```sh
ls -alh apps/frappe/
```

```sh
total 332K
drwxrwxr-x  10 radosav radosav 4,0K jul 11 00:37 .
drwxrwxr-x   3 radosav radosav 4,0K jul 11 00:35 ..
-rw-rw-r--   1 radosav radosav 1,5K jul 11 00:35 attributions.md
-rw-rw-r--   1 radosav radosav  741 jul 11 00:35 babel_extractors.csv
-rw-rw-r--   1 radosav radosav 1,2K jul 11 00:35 codecov.yml
-rw-rw-r--   1 radosav radosav 3,2K jul 11 00:35 CODE_OF_CONDUCT.md
-rw-rw-r--   1 radosav radosav  220 jul 11 00:35 CODEOWNERS
-rw-rw-r--   1 radosav radosav  394 jul 11 00:35 commitlint.config.js
-rw-rw-r--   1 radosav radosav  386 jul 11 00:35 .coveragerc
-rw-rw-r--   1 radosav radosav  444 jul 11 00:35 crowdin.yml
drwxrwxr-x   6 radosav radosav 4,0K jul 11 00:35 cypress
-rw-rw-r--   1 radosav radosav  701 jul 11 00:35 cypress.config.js
-rw-rw-r--   1 radosav radosav  406 jul 11 00:35 .editorconfig
drwxrwxr-x   2 radosav radosav 4,0K jul 11 00:35 esbuild
-rw-rw-r--   1 radosav radosav  278 jul 11 00:35 .eslintignore
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 .eslintrc
drwxrwxr-x  36 radosav radosav 4,0K jul 11 00:37 frappe
-rw-rw-r--   1 radosav radosav  568 jul 11 00:35 generate_bootstrap_theme.js
drwxrwxr-x   8 radosav radosav 4,0K jul 11 00:35 .git
-rw-rw-r--   1 radosav radosav 1,3K jul 11 00:35 .git-blame-ignore-revs
drwxrwxr-x   5 radosav radosav 4,0K jul 11 00:35 .github
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 .gitignore
drwxrwxr-x   2 radosav radosav 4,0K jul 11 00:35 .greptile
-rw-rw-r--   1 radosav radosav  890 jul 11 00:35 hooks.md
-rw-rw-r--   1 radosav radosav 1,1K jul 11 00:35 LICENSE
-rw-rw-r--   1 radosav radosav 1,8K jul 11 00:35 .mergify.yml
drwxrwxr-x 418 radosav radosav  20K jul 11 00:37 node_modules
-rw-rw-r--   1 radosav radosav 2,0K jul 11 00:35 node_utils.js
-rw-rw-r--   1 radosav radosav 2,5K jul 11 00:35 package.json
-rw-rw-r--   1 radosav radosav 2,2K jul 11 00:35 .pre-commit-config.yaml
-rw-rw-r--   1 radosav radosav 4,4K jul 11 00:35 pyproject.toml
-rw-rw-r--   1 radosav radosav 3,2K jul 11 00:35 README.md
drwxrwxr-x   4 radosav radosav 4,0K jul 11 00:35 realtime
-rw-rw-r--   1 radosav radosav  597 jul 11 00:35 .releaserc
-rw-rw-r--   1 radosav radosav  556 jul 11 00:35 SECURITY.md
-rw-rw-r--   1 radosav radosav    0 jul 11 00:35 .semgrepignore
-rw-rw-r--   1 radosav radosav   37 jul 11 00:35 sider.yml
-rw-rw-r--   1 radosav radosav   23 jul 11 00:35 socketio.js
-rw-rw-r--   1 radosav radosav 162K jul 11 00:35 yarn.lock
```

- **Git repo**

  Zbog prisustva:
  
  ```text
  .git/
  ```
  
  Ovaj ceo dir je Git repozitorijum Frappe-a.  
  
  Dakle:
  
  ```sh
  git clone https://github.com/frappe/frappe.git
  ```
  
  bi napravio upravo ovakav direktorijum.
  
  Unutra je sve:
  
  - izvorni kod
  - dokumentacija
  - testovi
  - frontend
  - konfiguracija
  - Git istorija

- **app/frappe/frappe - Python paket**:

  Sada pogledaj ovu stavku:
  
  ```sh
  apps/frappe/frappe
  ```
  
  Ona nije isto što i spoljni frappe direktorijum.
  
  Imamo:
  
  ```text
  apps/
  └── frappe/          ← Git repozitorijum
      └── frappe/      ← Python paket
  ```
  
  Ovo je veoma čest obrazac u Python svetu. Na primer Django izgleda slično:
  
  ```text
  django/
      django/
  ```
  
  Nije ništa neobično.

**Šta je sve ostalo u app/frappe/?**:

Pogledajmo redom.

- `README.md`

  - Opis projekta.  
  - Kako se instalira.  
  - Kako se razvija.

- `pyproject.toml`
  - Ovo je veoma važan fajl.  
  - Danas skoro svaki ozbiljan Python projekat koristi upravo njega.  

  - On govori:
    - kako se paket zove
    - koje su zavisnosti
    - koji build sistem koristi
    - koje Python verzije podržava
    Drugim rečima: "Ja sam Python paket."

- `package.json`
  - E ovo je zanimljivo.  
  - Odmah vidiš da Frappe nije samo Python.  
  - To je istovremeno i Node projekat.  
  - Frontend koristi:
    - JavaScript
    - Node
    - Yarn
    - esbuild
  Zbog toga si instalirao Node.js.

- `node_modules`  
  - Ogroman direktorijum.  
  - Tu su svi frontend paketi.  
  - Nema potrebe da ga ikada ručno diraš.  

- `esbuild`
  - Frontend build.  
  - JavaScript.  
  - CSS.  
  - Bundle.  

- `realtime`
  - Odmah vidiš da postoji poseban direktorijum.  
  - To je SocketIO server.  
  - Kasnije ćemo ga otvoriti.  

- `cypress`
  - Automatski testovi.
  - Browser testovi.

Do sada smo praktično složili sledeću mentalnu mapu:

```txt
frappe-bench/
├── apps/
│   └── frappe/          ← Git repozitorijum
│       └── frappe/      ← Python paket (framework)
│
├── sites/
│   ├── common_site_config.json
│   └── site1.local/
│       └── site_config.json
│
├── env/                 ← Python virtuelno okruženje
├── logs/
└── config/
```

A takođe smo razjasnili i odnose:

- **Bench** upravlja okruženjem.
- **App** predstavlja funkcionalnost (za sada samo `frappe`).
- **Site** predstavlja jednu instalaciju sa sopstvenom bazom.
- Jedna **aplikacija** može biti instalirana na više sajtova.
- Jedan **sajt** može imati više aplikacija.
- Svaki sajt ima svoju PostgreSQL bazu (ili MariaDB, u zavisnosti od konfiguracije).

### Python paket

Zapravo, najvažniji direktorijum još nismo otvorili.

To je:

```sh
apps/frappe/frappe/
```

Ovo je možda najvažniji direktorijum u celom frameworku. To je zapravo temelj cele Frappe arhitekture.

Hajde da pogledamo samo prvi nivo frappe direktorijuma.

```sh
tree -L 1 frappe
```

```sh
frappe
├── api
├── app.py
├── apps.py
├── auth.py
├── automation
├── boot.py
├── build.py
├── cache_manager.py
├── change_log
├── client.py
├── commands
├── config
├── contacts
├── core
├── coverage.py
├── custom
├── data
├── database
├── defaults.py
├── deferred_insert.py
├── desk
├── email
├── exceptions.py
├── frappeclient.py
├── geo
├── gettext
├── handler.py
├── hooks.py
├── __init__.py
├── installer.py
├── integrations
├── locale
├── middlewares.py
├── migrate.py
├── model
├── modules
├── modules.txt
├── monitor.py
├── oauth.py
├── onboarding.py
├── parallel_test_runner.py
├── patches
├── patches.txt
├── permissions.py
├── printing
├── public
├── pulse
├── push_notification.py
├── __pycache__
├── query_builder
├── rate_limiter.py
├── realtime.py
├── recorder.py
├── search
├── sessions.py
├── share.py
├── social
├── templates
├── test_runner.py
├── tests
├── translate.py
├── translations
├── twofactor.py
├── types
├── utils
├── website
├── workflow
└── www
```

Mnogi početnici pogledaju ovaj spisak i pomisle: *"Ovo je haos."* Međutim, kada se grupiše po nameni, postaje prilično logičan.

Ovde je logika: "Grupiši kod po **funkcionalnosti (feature)**, a ne po tipu fajla."

Zbog toga vidiš foldere kao što su:

```sh
desk/
email/
printing/
workflow/
website/
oauth/
contacts/
```

Svaki od njih predstavlja jednu veću celinu sistema.

#### Podela na 7 velikih grupa

- **Core framework**

  Ovo je srce Frappe-a.

  ```sh
  model/
  database/
  query_builder/
  utils/
  permissions.py
  exceptions.py
  hooks.py
  ```

  Ovo je nešto poput motora automobila.
  Ako bi pravio svoj framework, upravo bi ovde završio najveći deo posla.

- **HTTP/Web**

  Ovde počinje svaki zahtev iz browsera.

  ```sh
  app.py
  handler.py
  middlewares.py
  api/
  www/
  website/
  templates/
  ```

  ```txt
  Browser -> Nginx -> Gunicorn -> app.py -> handler.py -> Python
  ```

  To je ceo tok.

- **Desk (ERP interfejs)**

  ```sh
  desk/
  boot.py
  client.py
  ```

  Ovo je ono što korisnik vidi nakon logovanja.
  Dakle:

  - Workspace
  - List View
  - Form View
  - Report View
  - Search
  - Notifications

  Sve to živi ovde.

- **ORM i DocType**

  Za mene je ovo najlepši deo Frappe-a.

  ```sh
  model/
  database/
  modules/
  custom/
  workflow/
  ```

  Ovde nastaje ono po čemu je Frappe poznat.

  ```txt
  DocType -> Python objekat -> ORM -> SQL
  ```

- **Poslovni moduli**

  Ovo nisu framework delovi.
  To su ugrađene funkcionalnosti.

  ```sh
  contacts/
  email/
  printing/
  geo/
  social/
  oauth/
  workflow/
  ```

  Drugim rečima:  
  Framework kaže: "Evo kako se pravi modul."  
  Ovi folderi su: "Evo modula napravljenih pomoću tog framework-a."

- **Administracija**

  ```sh
  installer.py
  migrate.py
  patches/
  commands/
  tests/
  ```

  Ovde Bench i Frappe rade:

  - instalaciju
  - migracije
  - update
  - testove

- **Statički sadržaj**

  ```sh
  public/
  locale/
  translations/
  gettext/
  ```

  - CSS  
  - JS  
  - ikonice  
  - prevodi  
  - fontovi  

- **Šta je najvažnije**

  Po meni nisu svi ovi folderi jednako važni.
  
  Ja bih ih rangirao ovako:
  
  ```sh
  ⭐⭐⭐⭐⭐
  
  model/
  database/
  desk/
  ```
  
  ```sh
  ⭐⭐⭐⭐
  
  api/
  utils/
  permissions.py
  hooks.py
  ```
  
  ```sh
  ⭐⭐⭐
  
  website/
  www/
  templates/
  ```
  
  ```sh
  ⭐⭐
  
  email/
  printing/
  workflow/
  oauth/
  ```
  
  ```sh
  ⭐
  
  social/
  geo/
  pulse/
  ```

  Njih možeš proučavati i mnogo kasnije.

Ako bismo učili Frappe kao da zajedno pravimo sopstveni framework, redosled bi bio:
  
- model/
- database/
- query_builder/
- Document
- DocType
- Meta
- Desk
- API
- Website

Zašto baš ovako? Zato što sve u Frappe-u počinje od `Document` klase. Bukvalno sve.

- Customer  
- Item  
- Sales Invoice  
- User  
- Role  
- Project  
- Task  

...sve su oni samo izvedene forme jednog osnovnog koncepta: `Document`.

#### Frappe model

Kada kažeš: "Napravio sam DocType `Customer`. šta se zapravo desilo"?

Da li je napravljen:

- SQL tabela?
- Python klasa?
- HTML forma?
- JavaScript?
- REST API?

Odgovor je - sve od navedenog. I upravo zato Frappe deluje "magično". Ali nema magije. Samo dosta automatizacije.

**Definicija modela**:

U čistom Pythonu mogao bi da napišeš:
  
```python
class Person:
    def __init__(self):
        self.first_name = ""
        self.last_name = ""
```

U Django-u bi napisao:

```python
class Person(models.Model):
    first_name = models.CharField(...)
```

U SQLAlchemy:

```python
class Person(Base):
    ...
```

A u Frappe-u ne pišeš skoro ništa.
  
Zašto? Zato što će DocType definicija postati Python objekat u vreme izvršavanja (runtime).
To je ogromna razlika. To je razlog zašto možeš da napraviš novi DocType iz Desk interfejsa, bez pisanja Python klase, a framework ipak zna kako da kreira, učita, sačuva i validira te dokumente.

```sh
tree -L 1 frappe/model
```

```sh
frappe/model
├── base_document.py
├── create_new.py
├── db_query.py
├── delete_doc.py
├── docfield.py
├── docstatus.py
├── document.py
├── dynamic_links.py
├── __init__.py
├── mapper.py
├── meta.py
├── naming.py
├── __pycache__
├── rename_doc.py
├── sync.py
├── utils
├── virtual_doctype.py
└── workflow.py
```

Nećemo ići redom po fajlovima. Umesto toga, pokušaćemo da odgovorimo na jedno pitanja:

- "Šta se desi kada napišem `frappe.get_doc(...)` ili
- kada kliknem `Save` na formi?"

Ako to ispratimo, videćemo gotovo ceo `model` direktorijum u prirodnom redosledu.

Pogledaj ova imena:

```text
base_document.py
document.py
meta.py
docfield.py
```

Da li primećuješ nešto?

Nisu nazvani:

- customer.py
- sales_invoice.py
- employee.py

nego:

- Document
- Meta
- DocField

To znači da Frappe ne razmišlja o "Customer-u". Razmišlja o bilo kom dokumentu. To je veoma važna filozofija.

##### Četiri glavna pojma  

Ja ih zamišljam ovako:

```txt
DocType -> Meta -> Document -> Database
```

Svaki od ova četiri ima svoju ulogu.

- **DocType**  
  Ovo već poznaješ.  
  Na primer:  
  Customer ili Item ili Task  
  DocType nije Python klasa. DocType je opis.
  
  Na primer:
  
  ```txt
  Customer
    Fields:
      name
      email
      phone
      country
  ```
  
  To su samo podaci definicije DocTypea.

- **Meta**
  Ovde dolazi `meta.py`. Meta odgovara na pitanje: "Kako izgleda Customer?"
  Ne: "Koji je Customer?" nego: "Šta Customer uopšte jeste?"  
  Drugim rečima:

  ```txt
  Customer
    field 1 = Data
    field 2 = Link
    field 3 = Check
    field 4 = Date
  ```

  Meta opisuje strukturu DocTypea.  
  Ne sadržaj.

- **Document**

  Tek ovde dolazi jedan konkretan zapis.  
  Na primer:
  
  ```txt
  Customer
  Name: Petar
  Phone: 12345
  ```
  
  To više nije opis.  
  To je konkretan objekat podataka.

- **Database**

  Na kraju se sve pretvori u SQL.

##### Fajlovi Frape modela

- `base_document.py` -> Osnovna funkcionalnost.  
- `document.py` -> Pravi Document.  
- `meta.py` -> Opis DocType-a.  
- `docfield.py` -> Opis jednog polja.  

Većina ORM-ova radi ovako:

```py
class Customer(Model):
    name = CharField(...)
```

Kod je opis modela. U Frappe-u je obrnuto.

**Model**:

Model se nalazi u bazi kao DocType definicija, a Python ga učitava u `Meta` objekat. To znači da framework može da radi sa DocType-om koji nije postojao kada je Frappe pokrenut.

Napraviš novi DocType u Desk-u. Ne restartuješ server. Odmah radi.

To je moguće upravo zahvaljujući ovom sloju `Meta`.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
