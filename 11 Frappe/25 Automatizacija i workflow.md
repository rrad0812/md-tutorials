# Automatizacija i workflow u Frappe-u

Nakon Print Formata, logičan sledeći korak je prelazak sa prikaza podataka na poslovnu automaciju.

Za ovaj deo bih preporučio sledeći redosled:

1. Scheduled Job
2. Server Script
3. Workflow

- **Scheduled Job**

  Scheduled Job je najbolja početna tačka jer ti pokazuje da aplikacija može da radi i bez toga da korisnik nešto klikne.
  
  Ovo je dobra tema kada želiš da:
  
  - izvršavaš posao u pozadini
  - periodično proveravaš stanje
  - šalješ podsetnike ili obaveštenja
  - vršiš čisteće ili kontrolne akcije
  
  Na primer:
  
  - proveri taskove kojima je istekao rok
  - označi ih kao "zakasneli"
  - pošalji obaveštenje korisniku ili timu
  - ažuriraj neki status svakih nekoliko minuta ili svakog dana

- **Server Script**

  Kada razumeš Scheduled Job, prelaziš na Server Script.
  
  Server Script ti daje mogućnost da dodaješ logiku direktno na serveru, na primer kada se:
  
  - kreira dokument
  - menja dokument
  - briše dokument
  
  Zašto je ovo važno? Jer ti pokazuje kako se aplikacija ponaša kada se nešto dogodi.
  
  To je važan korak ka realnoj aplikaciji, jer više ne radiš samo "forme i tabele", nego praviš reagovanje sistema.
  
  Na primer:
  
  - kada se kreira Task, automatski postavi defaultni status
  - kada se Time Entry sačuva, izračunaj ukupno vreme
  - kada se promeni status, pošalji obaveštenje
  - kada se dokument zaključa, blokiraj dalju izmene
  
- **Workflow**

Workflow dolazi posle automatizacije jer tada već imaš bolju sliku o tome šta se dešava u aplikaciji.

Workflow ti pomaže da modeluješ poslovni proces:

- koji status ima dokument
- kada se menja status
- ko može da izvrši sledeću akciju
- šta se dešava kada se pređe u sledeći korak

Primeri ideja:

- Task ide od Draft → Open → In Progress → Completed
- Time Entry prolazi kroz status za odobrenje
- dokument može da se premesti samo uz određene dozvole
- neki korak može da otvori novu akciju ili da pošalje obaveštenje

## Server Script

Počni ovako:

1. Otvori jedan postojeći Task dokument.
2. Izaberi jedan mali primer, na primer:
   - kada se task kreira, status postaje Open!
   - kada je rok prošao, task dobija oznaku Late!
3. Zadrži se samo na tom jednom primeru i ne dodaj ništa drugo.

To je dovoljan prvi korak.

**Kako to da uradiš u Frappeu?**

U praksi, ovo je najjednostavniji put:

1. Otvori Desk.
2. Idi na modul gde imaš Task.
3. Otvori Task DocType ili modul za Task.
4. Uđi u Server Script `new Server Script`.
5. Napravi novi Server Script za događaj `before_insert`.
6. U polje za kod upiši jednostavnu logiku:

   ```python
   if doc.status != "Open":
     doc.status = "Open"
   ```

7. Sačuvaj i testiraj.
8. Napravi novi Task i vidi da li se status automatski promenio u Open.

Ako u sistemu postoji task koji nije završen:

```python
existing = frappe.db.exists("wmTask", {
    "status": ["!=", "Done"]
})

if existing:
    frappe.throw(f"Ne možeš kreirati novi zapis dok postoji aktivan dokument: {existing}")
```

Ako imaš više "zatvorenih" statusa, npr. "Done" i "Cancelled", onda je bolje:

```python
existing = frappe.db.exists("wmTask", {
    "status": ["not in", ["Done", "Cancelled"]]
})

if existing:
    frappe.throw(f"Ne možeš kreirati novi zapis dok postoji aktivan dokument: {existing}")
```

### Samo jedan aktivan task u celom sistemu

Ovo znači:

- ako postoji bilo koji task koji nije `Done` ili `Cancelled`
- ne dozvoljavaš novi

```python
existing = frappe.db.exists("wmTask", {
    "status": ["not in", ["Done", "Cancelled"]]
})

if existing:
    frappe.throw(f"Ne možeš kreirati novi task dok postoji aktivan task: {existing}")
```

Kada ovo ima smisla:

- ako je tvoj proces striktno linearan
- npr. sistem sme da ima samo jedan “trenutno otvoren” glavni posao

### Samo jedan aktivan task po korisniku

Ovo znači:

- jedan korisnik ne može imati više aktivnih taskova u isto vreme

Ako je polje `assigned_to`, onda:

```python
existing = frappe.db.exists("wmTask", {
    "assigned_to": doc.assigned_to,
    "status": ["not in", ["Done", "Cancelled"]]
})

if existing:
    frappe.throw(f"Korisnik već ima aktivan task: {existing}")
```

Kada ovo ima smisla:

- ako želiš fokus po korisniku
- jedan korisnik radi samo jedan aktivan task u datom trenutku

Važno:

- moraš zameniti `assigned_to` stvarnim imenom polja iz tvog DocType-a

### Samo jedan aktivan task po organizaciji

Ovo znači:

- svaka organizacija može imati samo jedan aktivan task

Ako imaš polje npr. `organization`, onda:

```python
existing = frappe.db.exists("wmTask", {
    "organization": doc.organization,
    "status": ["not in", ["Done", "Cancelled"]]
})

if existing:
    frappe.throw(f"Organizacija već ima aktivan task: {existing}")
```

Kada ovo ima smisla:

- ako svaka organizacija prolazi kroz jedan glavni proces
- i ne želiš paralelne aktivne taskove po organizaciji

### Kako da biraš između ove tri

1. Ako želiš globalno pravilo za ceo sistem:
   - varijanta 1

2. Ako želiš ograničenje po čoveku:
   - varijanta 2

3. Ako želiš ograničenje po poslovnom entitetu:
   - varijanta 3

### Važna napomena

Sve ove provere idu u: `Server Script`, `DocType Event`, `Before Insert`. I opet: bez `doc.save()`.

Ako menjaš postojeći dokument, onda `Before Insert` nije dovoljno, jer on važi samo za novi zapis.

## Scheduled Job

Za Scheduled Job u Frappe-u, najjednostavnije:

- ideš u Desk
- tražiš Scheduled Job
- kreiraš novi
- biraš:
  - naziv
  - funkciju koju će da izvrši
  - interval kada će da se pokreće

Osnovna ideja je:

- Scheduled Job ne reaguje na klik ili promenu dokumenta
- on "radi u pozadini" po rasporedu

**Prvi Scheduled Job primer:**

Najlakši prvi korak je ovaj:

- proveri sve Taskove koji su prošli rok
- ako su još otvoreni, promeni njihov status u Late

To znači da će Scheduled Job raditi sledeće:

1. uzme sve Taskove iz baze
2. proveri koji imaju due date u prošlosti
3. ako još nisu završeni, promeni status u Late

**Kako to napraviš u Frappe-u:**

1. Otvori Desk.
2. Idi na Scheduled Job.
3. Kreiraj novi Scheduled Job.
4. Daj mu naziv, npr. "Mark overdue tasks as late".
5. U polje za funkciju ili kod unesi jednostavnu logiku koja prolazi kroz Taskove.
6. Postavi interval, npr. svakih 5 minuta ili jednom dnevno.
7. Sačuvaj i testiraj.

**Ideja koda:**

U osnovi, logika bi bila:

```python
from frappe.utils import nowdate

tasks = frappe.get_all("wmTask", filters={"status": ["not in", ["Completed", "Cancelled"]]})

for task in tasks:
    doc = frappe.get_doc("wmTask", task.name)
    if doc.due_date and doc.due_date < nowdate():
        doc.status = "Late"
        doc.save()
```

Najvažnije je ovo:

- Scheduled Job ne reaguje na klik
- on radi po rasporedu
- koristi se kada želiš da se nešto dešava automatski u nekom vremenskom intervalu.

## Kada se šta koristi

Moglo bi se reći:

1. **Server Script**

   - kada se nešto dogodi sa dokumentom, npr. kreiranje, promena, čuvanje, brisanje.
   - koristiš ga za automatsku promenu dokumenta ili neku logiku na osnovu uslova.

2. **Scheduled Job**

   - kada nešto treba da se desi u određenom trenutku ili periodično
   - npr. svakog jutra proveri taskove, svakih 5 minuta proveri stanje, pošalji podsetnik

3. **Workflow**

   - kada želiš da dokument prolazi kroz više koraka/stanja
   - npr. Draft → Open → In Progress → Completed

## Različiti db pristupi

Evo male praktične tabele za najčešće Frappe pozive.

| Poziv | Šta vraća | Za šta služi | Pravi DocType objekat |
| --- | --- | --- | --- |
| `frappe.get_doc("wmTask", name)` | jedan dokument | čitanje i menjanje konkretnog zapisa | da |
| `frappe.new_doc("wmTask")` | novi prazan dokument | pravljenje novog zapisa u memoriji | da |
| `frappe.get_last_doc("wmTask")` | poslednji dokument | brzo uzimanje poslednjeg zapisa | da |
| `frappe.get_all("wmTask", ...)` | lista rezultata upita | brzo čitanje više zapisa | ne |
| `frappe.get_list("wmTask", ...)` | lista rezultata upita | čitanje više zapisa uz permission check | ne |
| `frappe.db.get_value("wmTask", name, "status")` | jedna vrednost ili tuple/dict | brzo čitanje jednog polja | ne |
| `frappe.db.get_single_value("System Settings", "time_zone")` | jedna vrednost | čitanje polja iz Single DocType | ne |
| `frappe.db.exists("wmTask", filters)` | ime dokumenta ili `None` | provera da li zapis postoji | ne |
| `frappe.db.set_value("wmTask", name, "status", "Late")` | obično ime ili potvrda update-a | direktan update bez punog lifecycle-a | ne |
| `frappe.get_cached_doc("wmTask", name)` | dokument iz cache-a ili baze | brže čitanje kada je cache koristan | da |

Najvažnija podela je:

1. Vraćaju pravi dokument:

   - `get_doc`
   - `new_doc`
   - `get_last_doc`
   - `get_cached_doc`

   Na njima ima smisla:

   - `doc.save()`
   - `doc.insert()`
   - `doc.submit()`
   - `doc.cancel()`

2. Vraćaju rezultat upita:

   - `get_all`
   - `get_list`
   - `db.get_value`
   - `db.exists`

   Na njima nema smisla:

   - `save()`
   - `insert()`

3. Direktno rade nad bazom:

   - `db.set_value`

   To je korisno kad hoćeš brz update, ali bez punog document lifecycle-a.

Najkraće pravilo za pamćenje:

- hoćeš listu ili proveru: `get_all`, `get_list`, `db.get_value`, `db.exists`
- hoćeš pravi dokument: `get_doc`
- hoćeš brz direktan update: `db.set_value`

Za tvoj slučaj sa scheduler-om:

1. `get_all(...)` da pronađeš kandidate
2. `get_doc(...)` ako želiš “pravi” update kroz lifecycle
3. `db.set_value(...)` ako želiš jednostavan i brz update
