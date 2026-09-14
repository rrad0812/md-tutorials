# Frappe Framework tutorijal

[Sadržaj][00]

## 16 Izrada aplikacije 1

Faze izrade aplikacaije bi bile:

1. **Projektovanje**

   - Analiza zahteva
   - Model podataka (DocType-ovi i veze)

2. **Izrada**

   - App
   - Module
   - Workspace
   - DocType-ovi
   - Child tabele

3. **Poslovna logika**

   - Controller
   - Hook-ovi (samo gde imaju smisla)
   - Validacije

4. **Bezbednost**

   - Role
   - Permission
   - User

5. **Korisnički deo**

   - Report-i
   - Dashboard
   - Print Format

6. **Deployment**

   - Git
   - Migracije
   - Instalacija na novi site

### Izrada konkretne Frappe aplikacije

Bilo bi interesantno hajde da napravimo nešto poput: **Project & Work Management**

#### Analiza

Aplikacija treba da:

- bude dovoljno mala da je završimo,
- bude dovoljno velika da iskoristi većinu mogućnosti Frappe-a,
- ima realan poslovni tok,
- koristi više povezanih DocType-ova,
- ima izveštaje,
- ima dozvole,
- ima workflow,
- ima dashboard,
- ima štampu,
- ima API,
- ima pozadinske (background) zadatke.

Odmah imamo:

- master podatke,
- transakcione podatke,
- relacije 1:N,
- relacije N:M,
- Child Table,
- Link polja,
- Workflow,
- Dashboard,
- Reports.

Ali ja ne bih koristio postojeće ERPNext DocType-ove (`Customer`, `Project`, `Task`...). Napravili bismo sve svoje DocType-ove.

Zašto? Zato što želimo da učimo Frape framework, a ne ERPNext.Ako koristimo postojeće ERPNext DocType-ove, stalno ćemo se pitati: "Da li je ovo naše ili ERPNext radi nešto iza kulisa?"

Ako napravimo sve od nule:

- znaćemo svako polje,
- svaku validaciju,
- svaki hook,
- svaku tabelu u bazi.

To će nam dati mnogo jasniju sliku kako Frappe funkcioniše.

Dakle, predlažem da krenemo ovim redosledom

- Definišemo funkcionalni obim aplikacije.
- Napravimo konceptualni model (entiteti i njihove veze).
- Isplaniramo strukturu aplikacije (Workspace, Module, navigacija).
- Tek onda otvorimo Bench i počnemo sa implementacijom.

Mislim da će nam ovaj pristup dati ne samo funkcionalnu aplikaciju, već i dobru osnovu za razumevanje kako se u Frappe-u projektuju ozbiljni poslovni sistemi.

#### Definisanje domena

Prvo moramo da odlučimo šta je **predmet poslovanja** naše aplikacije. Ja bih predložio da aplikaciju nazovemo radnim imenom: **Work Manager**, (ime nije bitno, kasnije ga možemo promeniti).

Njena svrha bi bila:

Evidencija

- projekata,
- poslova,
- angažovanja ljudi i
- utrošenog vremena.

To je dovoljno jednostavno da ne zalutamo, a dovoljno bogato da iskoristimo gotovo sve mogućnosti Frappe-a.

#### Ko su korisnici

Ja vidim četiri tipa korisnika:

- Administrator
- Menadžer projekta
- Zaposleni
- Klijent (opciono, kroz Portal)

Već ovde imamo razlog da kasnije učimo:

- Role
- Permission Manager
- User Permissions
- Portal

#### Osnovni poslovni objekti

Ja bih krenuo od ovih:

```text
Customer
Project
Task
Employee
Time Entry
Expense
```

To je sasvim dovoljno za prvu verziju.

Kasnije možemo dodati:

- Invoice
- Milestone
- Sprint
- Attachment
- Comment

ali sada ne.

#### Model

Prva verzija bi izgledala ovako:

```txt
Project
--------
ID
Name
Description
Status
Start Date
End Date

Task
--------
ID
Project
Title
Description
Status
Priority
Assigned To

Time Entry
--------
Task
Employee
Hours
Date

Employee
--------
Name
Email
Department
```

Ovo je sasvim dovoljan model za početak. Zašto baš ovako? Zato što svaki novi DocType uvodi novi Frappe koncept.

**Da li koristiti Child Table za Task?**

Mnogi početnici naprave:

```text
Project
  |
  └── Task (Child Table)
```

To deluje logično. Ali dugoročno pravi mnogo problema, jer Task tada:

- nema sopstveni URL,
- nema sopstvene dozvole,
- nema workflow,
- nema REST endpoint,
- nema dashboard,
- nema svoje izveštaje,
- ne može lako da se pretražuje.

Zbog toga bih od početka napravio `Project` i `Task` kao dva potpuno nezavisna DocType-a, povezana `Link` poljem. To je i način na koji je Frappe projektovan za ovakve entitete.

### Projektovanje strukture aplikacije

Pre nego što otvorimo terminal, želim da razdvojimo četiri pojma koja se u Frappe-u često mešaju:

```txt
Bench
   │
   ├── Apps
   │      │
   │      └── Modules
   │               │
   │               └── DocTypes
   │
   └── Sites
          |
          └── Workspaces
```

Ovo je jedna od stvari koja početnicima pravi najveću zabunu.

#### Apps

Aplikacija je ono što se distribuira.

Aplikacija sadrži Python kod, JavaScript, hookove, DocType-ove, izveštaje... Ukratko, predstavlja jedan logički proizvod.

#### Modules

Modul služi samo za organizaciju unutar aplikacije.

To nema veze sa Python modulima (`import`), već sa organizacijom funkcionalnosti u Frappe-u.

#### DocTypes

DocType predstavlja poslovni objekat.

Svaki od njih će imati svoju tabelu u bazi, kontroler, formu, listu itd.

#### Workspace

Workspace je početna stranica za korisnika.

Dakle:

- *Workspace* - je ulazna tačka za korisnika.
- *Module* - grupiše funkcionalnosti unutar aplikacije.
- *DocType* - predstavlja podatke.

#### Organizacija

Za početak bih bio vrlo konzervativan.

Jedan modul: Work Manager, a u njemu Project, Task, Employee, Time Entry, Expense DocTypes.

Tek kasnije, ako aplikacija poraste, možemo razdvojiti na više modula.

#### Naziv aplikacije

Ovde bih voleo da donesemo jednu odluku.

Ranije smo napravili aplikaciju:

```txt
frappe_lab
```

Ona nam je služila za eksperimente i mislim da tako treba i da ostane. Tu možemo i dalje isprobavati ideje bez bojazni da ćemo pokvariti "pravu" aplikaciju.

Za projekat koji sada počinjemo predložio bih novu aplikaciju, na primer:

```txt
work_manager
```

ili nešto drugo što zajedno odaberemo.

Zašto nova aplikacija?

- `frappe_lab` ostaje "laboratorija" za eksperimente.
- Nova aplikacija predstavlja stvaran proizvod.
- Lakše je kasnije verzionisati, objaviti ili čak koristiti na drugom sajtu.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
