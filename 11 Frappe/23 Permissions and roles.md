# Frappe Framework tutorijal

[Sadržaj][00]

## 23 Permissions i role

Nakon modela, validacija i report-a, sledeći korak je da razumeš da Frappe ne radi samo „model -> dokument“, nego i „ko sme šta da radi“.

Cilj ovog poglavlja je da ti objasni osnovu za dozvole i role u Work Manager projektu.

### Osnovni koncept

1. Korisnik ima jednu ili više rola.
2. Rola ima dozvole za DocType.
3. Dozvole određuju da li korisnik može da:
   - pravi dokumente,
   - čita ih,
   - menja ih,
   - briše ih.

U praksi to znači da `wmTask` i `wmTimeEntry` nisu samo tabele, nego i sigurnosni sloj.

### ASCII pogled: kako se veze skladaju

#### Role + Permissions

```text
Korisnik
└── Rola
    └── Dozvola za DocType
        ├── Read
        ├── Write
        ├── Create
        └── Delete
```

Primer:

```text
Employee User
└── Work Manager Employee
    └── Permission za wmTimeEntry
        ├── Read
        ├── Create
        └── Write
```

#### User Permissions (dodatno ograničenje)

Ovo je sledeći nivo. Role daje osnovna prava, a User Permission kaže:
„čak i sa tim pravima, ovaj korisnik može da vidi samo određene podatke“.

```text
Korisnik
├── Rola
│   └── Osnovna dozvola na DocType
└── User Permission
    └── Ograničenje po podatku / vrednosti / filter-u
```

Primer:

```text
Employee User
├── Rola: Employee
└── User Permission
    └── Može da vidi samo svoje wmTimeEntry zapise
```

#### Dublji model: šta se zapravo dešava

```text
User
├── Role
│   └── Permission
│       └── DocType
│           ├── Read
│           ├── Write
│           ├── Create
│           └── Delete
└── User Permission
    └── Record / Field / Value restriction
```

U praksi to znači:

- Role = ko je korisnik
- Permission = šta može da radi
- User Permission = na koje konkretne podatke može da utiče
- Custom logic / has_permission = ako treba još precizniji kontrolni sloj

#### Kako se ovo gradi u konkretnoj aplikaciji

U jednoj konkretnoj aplikaciji, na primer Work Manager, proces izgleda ovako:

1. napraviš dve role: `Work Manager Manager` i `Work Manager Employee`,
2. dodeliš role korisnicima u Desk-u,
3. u Role Permission Manager-u postaviš osnovne dozvole za `wmTask` i `wmTimeEntry`,
4. za zaposlenog dodaješ dodatno ograničenje da može da vidi i menja samo svoje zapise,
5. zatim proveriš rad sistema kao drugi korisnik.

Tako se iz teorije stvara prava pristupa u stvarnoj aplikaciji: ko može šta, nad kojim dokumentima i pod kojim uslovima.

### Praktičan plan za Work Manager

Za tvoj primer bih preporučio ove role:

1. `Work Manager Manager`
   - može sve da vidi i menja,
   - može da uređuje projekte, taskove i time entries.

2. `Work Manager Employee`
   - može da vidi taskove i projekte,
   - može da kreira i menja svoje `wmTimeEntry` zapise,
   - ne treba da menja sve taskove.

### Kako to postaviti u Desk-u

1. Otvori `User List` i izaberi korisnika.
2. U polju `Roles` dodaj odgovarajuću rolu.
3. Otvori `Role Permission Manager`.
4. Izaberi DocType, na primer `wmTask` ili `wmTimeEntry`.
5. Za svaku rolu postavi dozvole:
   - `Read`
   - `Write`
   - `Create`
   - `Delete` (ako je potrebno)

### Minimalni primer

Za `wmTask`:

- `Work Manager Manager`: Read, Write, Create, Delete
- `Work Manager Employee`: Read, Create, Write (opciono ograničeno)

Za `wmTimeEntry`:

- `Work Manager Manager`: Read, Write, Create, Delete
- `Work Manager Employee`: Read, Write, Create

### Brzi test

Nakon podešavanja:

1. Prijavi se kao korisnik sa rola `Work Manager Employee`.
2. Pokušaj da napraviš novi `wmTimeEntry`.
3. Proveri da li može da vidi taskove i projekte.
4. Proveri da li mu je zabranjeno da menja dokumente koje ne treba da menja.

### Važna napomena

Dozvole su različite od validacija.

- Validacija kaže: „Ovo nije ispravno“.
- Permission kaže: „Nije ti dozvoljeno da ovo uradiš“.

### Komentar

Da, to je sasvim tačno.

Moglo bi se reći ovako:

- User -> Role: jedan korisnik može imati više rola
- Role -> DocType: jedna rola može imati dozvole za više DocType-ova
- DocType -> Permission: svaki DocType ima svoje dozvole (read/write/create/delete)

U obliku relacije:

- User m:n Role
- Role m:n DocType

Isto tako, u praksi:

- korisnik dobija prava preko rola,
- rola donosi dozvole,
- DocType je objekat nad kojim se dozvole primenjuju.

Najkraće:

User → Role → DocType

To je ta Frappe mentalna mapa.

Što znači „opciono ograničeno“:

- to nije posebna Frappe dozvola,
- to je samo tvoj način da kažeš: „neću da dam svim employee-ima potpuno isto pravo, nego ću ih ograničiti na neki podskup“.
- npr. Employee može da vidi taskove, ali ne sme da menja sve taskove ili da briše ih.

U praksi, najčešće to znači:

- Employee može da kreira svoj `wmTimeEntry`
- ne može da menja tuđe time entries
- ne može da briše taskove
- ne može da menja projekat ako to nije deo njegove uloge

Najjednostavnija verzija za početak:

- Manager: sve
- Employee: samo read/create/write za `wmTimeEntry`, read za `wmTask`

Ograničenje može da bude i preko role i preko dozvola, zavisno od toga šta želiš da postigneš.

1. Rola

   - Rola je „grupa prava“.
   - Npr. `Employee` rola znači: ovaj korisnik ima određeni skup dozvola.

2. Dozvola

   - Dozvola je konkretno pravo na DocType-u.
   - Npr. `Read` na `wmTask`, `Write` na `wmTimeEntry`.

3. Ograničenje

   - Ako hoćeš da kažeš „Employee može da vidi samo svoje time entries“, to već nije samo obična dozvola.
   - To je dodatno ograničenje nad podacima.

U Frappe-u to najčešće znači:

- osnovna dozvola: role + permission
- dodatno ograničenje: custom logic ili user permissions

Najjednostavnije pravilo:

- role = ko je
- permission = šta može
- ograničenje = na šta tačno može

Primer:

- `Employee` rola
- dozvola: `Read/Write/Create` za `wmTimeEntry`
- ograničenje: može da vidi samo svoje zapise, ne i tuđe

To je često:

- role + permission + user permission / custom logic

Za tvoj početni Work Manager:

- ne moraš odmah da ides u složena ograničenja,
- dovoljno je da imaš:
  - role,
  - dozvole,
  - i razumeš da „ograničeno“ znači „nešto više od obične dozvole“.
  
Pa daj da ti dam jedan vrlo praktičan primer, bez komplikacija.

Pretpostavimo da imaš:

- `Manager`
- `Employee`

I da želiš da `Employee` može:

- da vidi taskove,
- da pravi svoje `wmTimeEntry`,
- ali ne da vidi ili menja tuđe `wmTimeEntry`.

To znači:

1. Rola
   - `Employee` je rola.

2. Dozvole
   - za `wmTask`: Read
   - za `wmTimeEntry`: Read, Create, Write

3. Ograničenje

   - Employee može da vidi samo svoje zapise u `wmTimeEntry`
   - to je već dodatno ograničenje nad podacima

U Frappe svetu to se obično rešava na dva nivoa:

- lakši nivo:

  - role + permissions

- napredniji nivo:

  - user permissions ili custom logic u `has_permission` / `validate`

Najjednostavnije pravilo za tebe:

- ako želiš samo "da se ne prevrće sve", startuj sa role + permissions
- ako želiš "samo moj zapis", onda ideš na ograničenje nad podacima

Za tvoj Work Manager:

- za početak je sasvim dovoljno da Manager sve može, a Employee samo svoje time entries.
- to je odličan prvi korak, i nije potrebno odmah ništa “tehnički komplikovano”.

Ako hoćeš, mogu odmah da ti napišem i konkretan primer “Manager vs Employee” kao mini tabela, baš kao da si to hteo da napraviš u Desk-u.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
