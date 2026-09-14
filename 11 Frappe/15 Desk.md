# Frappe Framework tutorijal

[Sadržaj][00]

## 15 Desk

Najkraće: Desk je razvojno i poslovno okruženje izgrađeno na Frappe-u.

Kada odeš na:

```text
http://site1.local/app
```

ti nisi samo u "admin panelu". Ti si u jednoj kompletnoj aplikaciji.

### Glavni objekti

Ovo je mapa koju bih voleo da zapamtiš.

```text
App
│
├── Module
│     │
│     ├── DocType
│     ├── Report
│     ├── Print Format
│     ├── Workspace
│     ├── Page
│     ├── Dashboard
│     ├── Web Form
│     └── ...
│
└── hooks.py
```

To je praktično ceo Frappe.

#### DocType

Njega već znaš.

On predstavlja:

* tabelu,
* model,
* formu,
* REST resurs,
* dozvole.

Sve u jednom.

#### Workspace

Ono što vidiš na početnoj strani.

Na primer:

* Accounting
* HR
* CRM
* Website

To su Workspace-i.

Svaki Workspace može imati:

* prečice,
* grafikone,
* brojače,
* liste,
* linkove.

To je početna strana za korisnika.

#### Report

Ne pišeš SQL svaki put.

Možeš napraviti:

* Report Builder
* Query Report
* Script Report

#### Page

Ako želiš potpuno svoju stranicu.

Na primer:

* Dashboard proizvodnje
* Mapa
* Kanban
* Calendar

To nije DocType.

To je posebna stranica.

#### Dashboard

Dashboard je način da korisnik brzo vidi najvažnije podatke bez otvaranja svakog dokumenta jedan po jedan.

U praksi to znači:

* brojčane vrednosti (KPI), na primer: ukupno narudžbi, otvorenih zadataka, prodaja ovog meseca,
* grafikone, na primer: trend prodaje po danima ili mesecima,
* tabele i liste, na primer: poslednjih 10 aktivnih naloga,
* linkove ka važnim stranicama, na primer: svi otvoreni zadaci, svi korisnici, svi dokumenti.

Najvažnija ideja je sledeća:

* Workspace je početna stranica aplikacije,
* Dashboard je pogled na ključne metrike i stanje sistema.

Drugim rečima, Workspace je "mesto gde ulaziš", a Dashboard je "mesto gde vidiš šta je bitno".

U Frappe-u je dashboard često sastavljen od nekoliko blokova:

* Number Card — brojčana vrednost,
* Chart — grafikon,
* List — lista podataka,
* Link/Card — prečica ka drugoj stranici.

Zato dashboard ne mora biti samo "lep ekran". On treba da pomogne korisniku da odmah razume stanje posla.

### Kako se to gradi u konkretnoj aplikaciji

Recimo da praviš aplikaciju za timski rad, na primer Work Manager.

U praksi bi to izgledalo ovako:

1. napraviš Workspace pod nazivom "Work Manager",
2. dodaješ Number Card za broj otvorenih taskova,
3. dodaješ Chart za broj taskova po statusu,
4. dodaješ List sa poslednjih 10 aktivnih taskova,
5. dodaješ link ka stranici za Time Entry ili Report-e.

Tako dashboard postaje prvi ekran na kojem korisnik vidi stanje sistema, a ne samo praznu početnu stranicu.

#### Print Format

* PDF.
* Fakture.
* Otpremnice.
* Nalepnice.
* Sve preko Jinja template-a.

#### Web Form

Ako želiš da neko spolja popuni formu.

Na primer:

* Prijava za posao
* Kontakt forma
* Rezervacija

bez logovanja.

#### Website

Frappe može biti i CMS.

Možeš napraviti:

* blog,
* dokumentaciju,
* landing page,
* portal.

### Izgradnja poslovne aplikacije

Po meni je sada najvažnije pitanje: Kako od ovoga nastaje gotova poslovna aplikacija?

Jer tu većina početnika izgubi nit.

U stvarnosti, proces izgleda ovako:

```txt
    new-app
       ↓
    Workspace
       ↓
    DocType
       ↓
    Module
       ↓
    Permissions
       ↓
    Report
       ↓
    Print Format
       ↓
    Deploy
```

I to je zapravo ceo razvojni ciklus.

### Mentalna mapa jedne aplikacije

Kada sam prvi put učio Frappe, imao sam utisak da postoje stotine nepovezanih pojmova:

* DocType
* Page
* Workspace
* Report
* Workflow
* Dashboard
* Print Format
* Role
* Hook
* Scheduler
* ...

Sada ih više ne vidim tako. Sada ih vidim kao kutiju sa alatima.

Za svaku poslovnu potrebu uzmeš odgovarajući alat:

* treba ti podatak → **DocType**,
* treba pregled → **Workspace**,
* treba analiza → **Report**,
* treba PDF → **Print Format**,
* treba automatizacija → **Scheduler**,
* treba proširenje → **Hook**.

I to je, po mom mišljenju, pravi mentalni model Frappe-a.

### Deploy

Recimo da si napravio:

```text
library
```

na svom laptopu.

Kako ona završi kod korisnika?

#### Kod

Tvoja aplikacija je običan Git projekat:

```text
apps/
    library/
```

Nema magije.

Objaviš je na GitHub, GitLab ili privatni Git server.

#### Server

Na serveru napraviš Bench:

```bash
bench init production-bench
```

Zatim:

```bash
bench get-app https://github.com/tvoje_ime/library.git
```

ili iz lokalnog repozitorijuma.

#### Site

Napraviš novi sajt:

```bash
bench new-site firma.local
```

#### Instalacija aplikacije

```bash
bench --site firma.local install-app library
```

#### Migracija

Kada napraviš novi DocType ili dodaš polje:

```bash
bench --site firma.local migrate
```

To uradi:

* SQL ALTER TABLE
* sinhronizaciju DocType metadata
* patch-eve
* fixture-e
* scheduler

i još dosta toga.

Ti ne pišeš SQL migracije ručno.

#### Nova verzija

Na serveru:

```bash
cd apps/library
git pull
```

zatim:

```bash
bench --site firma.local migrate
```

Gotovo.

#### Gde je ovde Docker

Može. Može i bez njega. Može Kubernetes. Može običan Ubuntu. To nije odluka Frappe-a.  

#### Nginx

U development-u:

```bash
bench start
```

imaš ugrađeni server.

U produkciji:

```txt
   Internet
      │
   nginx
      |
   gunicorn
      |
   frappe
      │
   postgres/mariadb
```

i još:

* Redis
* worker procesi
* scheduler
* websocket servis

Bench ume da generiše većinu potrebne konfiguracije.

Bench nije samo CLI.

On upravlja:

* Python okruženjem,
* Node build-om,
* migracijama,
* asset-ima,
* servisima,
* aplikacijama,
* site-ovima.

To je praktično "devops alat" za Frappe.

#### App i Site nisu isto

Jedna aplikacija:

```text
library
```

može biti instalirana na:

```text
library_beograd

library_novi_sad

library_nis
```

Tri različita sajta.

Svaki ima:

* svoju bazu,
* svoje korisnike,
* svoje podatke,

ali koriste isti kod aplikacije.

To je veoma elegantan model za SaaS ili za više klijenata.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
