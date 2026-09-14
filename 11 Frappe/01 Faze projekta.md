# Frappe Framework tutorijal

[Sadržaj][00]

## 01 Faze projekta

### Priprema okruženja

- Izabrati verziju Ubuntu-a (24.04 LTS)
- Kreirati novu VM
- Instalirati Ubuntu Server
- Ažurirati sistem
- Instalirati osnovne alate
- Napraviti snapshot Clean System

  Cilj: "Stabilna razvojna mašina.
  
### Instalacija Frappe okruženja

- Python
- Node.js
- Redis
- PostgreSQL
- Bench
- Kreirati prvi Bench

  Ovde nećemo još praviti nijedan DocType.  
  Cilj je da razumemo: "Šta je Bench?"

### Anatomija Bencha

  Ovde ne pišemo kod. Samo istražujemo.
  
- Struktura direktorijuma
- apps/
- sites/
- env/
- logs/
- config/
  
  Na kraju ove faze treba da možeš da pogledaš Bench i kažeš: "Znam čemu služi svaki direktorijum."

### Prvi Site

- napraviti Site
- pokrenuti ga
- otvoriti Desk
- prijaviti se
- pogledati šta je nastalo u PostgreSQL-u
  
  Ovde ćemo prvi put zaviriti u bazu.

### Anatomija Site-a

  Opet bez programiranja.
  
- site_config.json
- private/
- public/
- assets/
- logs/
  
  Na kraju: "Znam šta je Site."

### Prva aplikacija

  Tek ovde,
  
- new-app
- instalacija aplikacije
- modul
- prvi DocType

### Desk

  Ovo je deo koji si već pomenuo da ti je bio nejasan.
  
  Ovde ćemo odgovoriti na pitanja:
  
- Kako Desk vidi moj DocType?
- Kako se pojavljuje u Workspace-u?
- Zašto ga nekad nema?
- Kako ga organizovati?
  
  Mislim da će ova faza biti jedna od najzanimljivijih.

### DocType

  Ovde konačno ulazimo u razvoj.
  
- polja
- validacija
- child table
- link
- select
- naming

### ORM

- Python.  
- Ne JavaScript.  
- Ne REST.  
- Samo ORM.

### Hook-ovi

  Šta se događa kada:
  
- sačuvaš dokument
- obrišeš dokument
- submit
- cancel
  
### JavaScript

- Client Script.
- Form Script.
- List Script.

### REST API

  Ovde ćemo ga uporediti sa uAdmin-om.  
  Mislim da će to biti veoma zanimljivo.

### Bezbednost

- Users
- Roles
- Permissions

### Deploy

  Tek na kraju.
  
- production
- nginx
- supervisor
- backup

### Završna faza

  Napravićemo jednu ozbiljniju aplikaciju. Ne "ToDo", ne "Student". Nego nešto što ima smisla.
  
[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
