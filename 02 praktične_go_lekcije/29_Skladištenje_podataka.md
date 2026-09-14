
# 29 Skladištenje podataka - Datoteke i baze podataka

[28 Datumi i vreme][28] | [00 Sadržaj][00] | [30 Konkurentnost][30]

**Šta ćete naučiti u ovom poglavlju?**

- Kako kreirati datoteku.
- Kako upisati podatke u datoteku.
- Kako kreirati CSV datoteku.
- Kako se upituje u sledećim bazama podataka:
  - MySQL
  - PostgreSQL
  - MongoDB
  - Elastična saerch

**Obrađeni tehnički koncepti!**

- Dozvola za datoteku
- Oktalno
- SQL injekcije
- Pripremljeni upiti
- Bez šeme
- RDBMS

## Uvod

Ovaj odeljak će proći kroz različite tehnike za čuvanje podataka pomoću Go-a.

## Napravite datoteku

Metoda `os.Create` će kreirati novu datoteku:

```go
// data-storage/file-save/simple/main.go 
package main

import (
    "fmt"
    "os"
)

func main() {
    f, err := os.Create("test.csv")
    if err != nil {
        fmt.Println(err)
        return
    }
    //... success
}
```

Kreiramo datoteku pod nazivom "test.csv". Ako postoji greška, ispisujemo grešku i vraćamo rezultat. Ovaj program je ekvivalentan sledećem:

```go
f, err := os.OpenFile("test.csv", os.O_RDWR|os.O_CREATE|os.O_TRUNC, 0666)
if err != nil {
    fmt.Println(err)
    return
}
```

`os.Create` interno poziva `os.OpenFile` sa sledećim parametrima:

- Ime datoteke (ovde: "test.csv")
- Set zastavice (ovde: `os.O_RDWR|os.O_CREATE|os.O_TRUNC`)
- Režim datoteke (ovde: `0666`)

U naredna dva odeljka ćemo videti šta su te zastavice i režimi. Možete preskočiti te odeljke ako ste već upoznati sa tim pojmovima.

## Zastavice datoteka

Kada otvorite datoteku pomoću, `os.OpenFile` vršite sistemski poziv. Ovaj sistemski poziv treba da zna putanju do datoteke i dodatne informacije o vašim namerama. Te informacije su sažete u listi zastavica. To su različite vrste zastavica:

- **Zastavice režima pristupa**: one će definisati na koji način ćemo kreirati datoteku. Uvek morate dati jednu od ovih zastavica.
- **Zastavice kreiranja**: one će uticati na to kako ćemo izvršiti sistemske operacije
- **Zastavice statusa datoteke**: one će uticati na sve operacije koje ćemo izvršiti nakon otvaranja datoteke.

| Naziv zastavice | Opis |
| --------------- | ---- |
| os.O_RDONLY | Otvori u režimu samo za čitanje. Možete samo da pročitate datoteku |
| os.O_WRONLY | Otvori u režimu samo za pisanje. Možete samo da pišete |
| os.O_RDWR | Otvori u režimu čitanja i pisanja. |
| os.O_CREATE | Kreiraće datoteku ako ona ne postoji. |
| os.O_EXCL | Ako se koristi sa os.O_CREATE dovešće do greške ako datoteka već postoji. |
| os.O_TRUNC | Datoteka će biti skraćena prilikom otvaranja. Ako je nešto napisano u datoteci, biće obrisano. |
| os.O_APPEND | Ako pišete u datoteku, ono što napišete biće dodato. |
| os.O_SYNC | Otvorite datoteku u sinhronom režimu. Sistem će sačekati da se svi podaci zapišu. |

Svaka zastavica je ceo broj. Da biste koristili te zastavice, morate koristiti bitske operatore. Kada pozovete os.Create datoteka će biti otvorena sa zastavicama:

```go
os.O_RDWR|os.O_CREATE|os.O_TRUNC
```

To znači da će datoteka biti otvorena u režimu čitanja i pisanja. Biće kreirana ako ne postoji. Pored toga, sistem će je skratiti ako je moguće.

Na primer, da biste bili sigurni da je kreirana nova datoteka, možete dodati zastavicu: `os.O_EXCL`

```go
f, err := os.OpenFile("test.csv", os.O_RDWR|os.O_CREATE|os.O_TRUNC|os.O_EXCL, 0666)
if err != nil {
    fmt.Println(err)
    return
}
fmt.Println(f)
```

Izvršavanje prethodnog programa će ispisati:

```sh
open test.csv: file exists
```

što je očekivano ponašanje.

## Režimi datoteke

Na UNIX sistemima, svaka datoteka ima skup dozvola:

- Dozvole dodeljene vlasniku datoteke
- Dozvole za korisnike koji su članovi grupe kojoj datoteka pripada
- Dozvole za sve ostale korisnike

Ako želite da vizualizujete dozvole datoteke, otvorite terminal i unesite `ls -l`.

```sh
$ ls -al
-rw-r--r--  1 maximilienandile  staff  242 25 nov 19:47 main.go
```

- `-rw-r--r--` : režim datoteke
  - 1 : broj linkova
  - maximilienandile : vlasnik datoteke
  - staf : grupa datoteke
  - 242 : broj bajtova datoteke
  - 25.novembar 19:47 : datum poslednje izmene

### Simbolička notacija

Režim datoteke je strukturiran u tri bloka (dozvole za korisnike, grupu i ostatak sveta).

U ta tri bloka imate tri slova koja definišu dozvole. Ona su uvek istim redosledom:

- `r` : čitanje
- `w` : pisanje
- `x` : izvršavanje

Prvi znak je crtica `-`, ako je u pitanju datoteka, inače je `d` u slučaju direktorijuma. Ako jedno od slova nedostaje i napisana je samo crtica, to znači da dozvola nije dostupna.

Uzmimo primer sledećeg režima datoteke:

```sh
-rw-r--r--
```

- Prvi znak - : to je datoteka
- Korisnik (rw-) : može da čita, može da piše, ne može da izvršava
- Grupa (r--) : može da čita, ne može da piše niti da izvršava
- Ostatak sveta (r--) : može da čita, ne može da piše niti da izvršava

Hajde da dobijemo režim datoteke sa go :

```go
// data-storage/file-save/mode/main.go 

// open the file
f, err := os.Open("myFile.test")
if err != nil {
    fmt.Println(err)
    return
}

// read the file info
info, err := f.Stat()
if err != nil {
    fmt.Println(err)
    return
}

fmt.Println(info.Mode())
```

Prethodni program će ispisati -rw-r--r--.

### Numerička notacija

Mod se takođe može konvertovati u oktalni broj:

```go
fmt.Printf("Mode Numeric : %o\n", info.Mode())
fmt.Printf("Mode Symbolic : %s\n", info.Mode())
// Mode Numeric : 644
// Mode Symbolic : -rw-r--r--
```

Numeričku notaciju koristi Go kada pozovete `os.OpenFile` (treći parametar). Ova notacija se sastoji od 3 cifre.

Cifre (oktalne) predstavljaju skup dozvola. Svaka oktalna cifra (od 0 do 7) ima specifično značenje.

| Dozvola | Simbolično | Oktalno |
| ------- | ---------- | ------- |
| Nijedna | --- | 0 |
| Samo izvršavanje | --x | 1 |
| Samo pisanje | -w- | 2 |
| Pisanje i izvršavanje | -wx | 3 |
| Samo čitanje | r-- | 4 |
| Čitanje i izvršavanje | r-x | 5 |
| Čitanje i pisanje | rw- | 6 |
| Čitanje i pisanje i izvršavanje | rwx | 7 |

Uzmimo primer: 644. Hajde da ga razložimo:

- Vlasnik: 6 što znači: čitanje + pisanje
- Grupa: 4 što znači: čitanje
- Ostali: 4 što znači: čitanje

Još jedan primer 777 :

- Vlasnik: 7 što znači: čitanje + pisanje + izvršavanje
- Grupa: 7 što znači: čitanje + pisanje + izvršavanje
- Ostalo: 7 što znači: čitanje + pisanje + izvršavanje

### Promenite režim datoteke pomoću Go-a

Na datoteci možete koristiti metodu `Chmod` za promenu režima datoteke:

```go
// data-storage/file-save/mode/main.go 

err = f.Chmod(0777)
if err != nil {
    fmt.Println(err)
}
```

Ovde smo promenili režim datoteke na 0777. Što znači da sva dozvole date vlasniku, grupi i ostalima.

Zašto smo dodali nulu u 0777? To signalizira Gou da broj nije "decimalni" već "oktalni".

## Pisanje u datoteku: primer CSV datoteke

### Šta je CSV datoteka

CSV znači vrednosti razdvojene zarezima. To je format datoteke dizajniran za čuvanje podataka.

- CSV datoteka se sastoji od nekoliko redova
- Svaki novi red je zapis podataka
- Svaki zapis podataka sastoji se od "polja"
- Zarez odvaja svako polje
- Prvi red se često koristi kao zaglavlje za opisivanje onoga što se može naći u poljima.

Evo primera CSV datoteke:

```sh
age,genre,name
23,M,Hendrick
65,F,Stephany
```

### Kodeks

U ovom odeljku ćemo videti kako se podaci zapisuju u datoteku. Da bismo odmah primenili naše znanje, koristićemo stvarni slučaj upotrebe: kreiranje CSV datoteke iz isečka stringova.

```go
// data-storage/file-save/writing/main.go 
package main

import (
    "bytes"
    "fmt"
    "os"
)

func main() {
    s := [][]string{
        {"age", "genre", "name"},
        {"23", "M", "Hendrick"},
        {"65", "F", "Stephany"},
    }

    // Open the file or create it
    f, err := os.Create("myFile.csv")
    defer f.Close()
    if err != nil {
        fmt.Println(err)
        return
    }
    
    var buffer bytes.Buffer
    
    // iterate over the slice s
    for _, data := range s {
        buffer.WriteString(fmt.Sprintf("%s,%s,%s\n", data[0], data[1], data[2]))
    }

    n, err := f.Write(buffer.Bytes())
    fmt.Printf("%d bytes written\n", n)
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

Prvo kreiramo isečak od isečaka "s". To su podaci koje želimo da zapišemo u CSV datoteku.

Kreiramo promenljivu tipa `bytes.Buffer` pod nazivom "buffer".

A zatim iteriramo "s" da bismo dodali podatke u "buffer". Svaka vrednost je odvojena zarezom. Označavamo da želimo da dodamo novi red pomoću karaktera "\n".

Kada je "buffer" spreman, upisujemo ga u datoteku pomoću metode "Write". Ova metoda uzima isečak bajtova kao parametar i vraća dve vrednosti:

- broj bajtova zapisanih u datoteku
- eventualnu grešku

### bytes.Buffer

U prethodnom programu smo koristili bafer podataka. Bafer "je region fizičke memorije koji se koristi za privremeno čuvanje podataka dok se oni premeštaju sa jednog mesta na drugo".

Ovaj tip `bytes.Buffer` može biti veoma koristan kada je potrebno efikasno spajanje stringova ili manipulisanje isečcima bajtova.

## Primer slučaja upotrebe

U sledećem odeljku ćemo koristiti sledeći primer upotrebe:

Škola te je zaposlila, da izradiš program za upravljanje učenicima i nastavnicima:

- preuzmi, dodaj, ažuriraj, obriši nastavnika
- preuzmi, dodaj, ažuriraj, obriši učenika

U sledećim odeljcima ćemo videti kako to uraditi sa različitim bazama podataka:

## MySQL baza podataka

MySQL je sistem za upravljanje relacionim bazama podataka otvorenog koda. U ovom odeljku ćemo videti kako se povezati sa MySQL bazom podataka i kako izvršiti osnovne CRUD operacije pomoću Go-a.

> [!Note]
> **Pažnja**: Go se ne isporučuje sa MySQL drajverom.

Standardna biblioteka definiše interfejse za manipulaciju SQL bazom podataka.

U vreme pisanja ovog teksta, na Go vikiju su predložena dva drajvera.

Izabrao sam najpopularniji na GitHub-u <https://github.com/go-sql-driver/mysql>, koji je pod Mozilla Public License verzije 2.0.

### Povezivanje na MySQL

```go
// data-storage/mysql/create/main.go 
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/go-sql-driver/mysql"
)

func main() {
    db, err := sql.Open("mysql", "root:root@tcp(localhost:8889)/school")
    if err != nil {
        fmt.Println(err)
        return
    }
    
    err = db.Ping()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

Ovde uvozimo standardni paket `database/sql` koji definiše sve funkcije potrebne za obavljanje operacija na SQL bazi podataka.

Zatim koristimo prazan uvoz da registrujemo drajver "github.com/go-sql-driver/mysql".

Ako ne uvezete drajver, može doći do sledeće greške:

```go
sql: unknown driver "mysql" (forgotten import?)
```

Funkcija `sql.Open` prihvata dva parametra:

- Ime drajvera
- Naziv izvora podataka (često se naziva DSN)

Ovaj DSN parametar je string gde definišete:

- korisničko ime: root
- lozinka: root
- protokol koji se koristi za povezivanje sa bazom podataka: TCP
- domaćin: localhost
- port: 8889
- naziv baze podataka: school

DSN je u sledećem obliku:

```go
username:password@protocol(address)/dbname?param=value
```

Funkcija `sql.Open` vraća `*sql.DB`.

Metoda `Ping` će proveriti da li je veza dostupna; ako nije, kreiraće novu.

### Kreiranje tabele u MySQL

Prvo što biste možda želeli da uradite je da kreirate tabelu u našoj bazi podataka. Hajde da napišemo naš SQL skript:

```sql
CREATE TABLE `teacher` (
    `id` INT(11) NOT NULL AUTO_INCREMENT,
    `create_time` TIMESTAMP DEFAULT NULL,
    `update_time` TIMESTAMP DEFAULT NULL ON UPDATE CURRENT_TIMESTAMP,
    `firstname` VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
    `lastname` VARCHAR(255) CHARACTER SET utf8 COLLATE utf8_general_ci DEFAULT NULL,
    PRIMARY KEY (`id`)
) ENGINE=InnoDB;
```

Kreiramo tabelu pod nazivom "teacher" koja će sadržati podatke o svakom nastavniku u školi:

- NJihovo ime
- NJihovo prezime

Svaki nastavnik će imati "ID" (primarni ključ naše tabele). Takođe dodajemo dve tehničke kolone koje će čuvati datum kreiranja (create_time) reda i datum poslednjeg ažuriranja (update_time).

Hajde da pokrenemo ovaj skript.

Prvo, sačuvaćemo ovaj SQL skript u datoteku pod nazivom *"create_table.sql"*. Naš skript će prvo učitati skriptu:

```go
f, err := os.Open("create_table.sql")
if err != nil {
    fmt.Println(err)
    return
}

b, err := ioutil.ReadAll(f)
if err != nil {
    fmt.Println(err)
    return
}
```

U b, imamo isečak bajtova koji predstavlja naš upit. Hajde da ga izvršimo:

```go
res, err := db.Exec(string(b))
if err != nil {
    fmt.Println(err)
    return
}
```

Koristimo `string(b)` za pretvaranje isečka bajta u string jer metoda `Exec` uzima string kao parametar.

Obratite pažnju da se vraćaju dve vrednosti, ili rezultat ( tip `sql.Result` ) ili greška. `sql.Result` je interfejs tipa sa dve metode:

```go
LastInsertId() (int64, error)
RowsAffected() (int64, error)
```

Samo ćemo proveriti da nema grešaka; kreiranje tabele ne utiče na postojeće redove niti na umetanje podataka.

### Umetanje u MySQL

Da bismo dodali nastavnika, koristićemo "INSERT" naredbu. Koristićemo `Exec` metod:

```go
// data-storage/mysql/insert/main.go 

func createTeacher(firstname string, lastname string, db *sql.DB) (int64, error) {
    res, err := db.Exec("INSERT INTO `teacher` (`create_time`, `firstname`, `lastname`) VALUES (NOW(), ?, ?)", firstname, lastname)
    if err != nil {
        return 0, err
    }

    id, err := res.LastInsertId()
    if err != nil {
        return 0, err
    }

    return id, nil
}
```

Ovde imamo funkciju "createTeacher", koja kao parametre prihvata dva stringa "firstname" i "lastname" i pokazivač na "sql.DB".

Metodi Exec prosleđujemo tri argumenta.

- SQL upit (sa ? kao rezervisanim mestom za vrednosti) i vrednosti koje želimo da ubrizgamo u
  bazu podataka.
- db.Exec() gradi pripremljeni iskaz.

#### Pripremljeni iskazi

Prateći dokumentaciju MySQL, pripremljeni iskaz vrši dve različite operacije:

- **Faza pripreme**: ovde klijent (naš Go program) šalje šablon serveru baze podataka. Server će
  izvršiti proveru sintakse zahteva. Takođe će inicijalizovati resurse za sledeće korake.

- **Faza povezivanja i izvršavanja**: klijent zatim šalje vrednosti serveru (u našem slučaju
  "John" i "Doe"). Zatim će server izgraditi upit sa tim vrednostima i konačno izvršiti upit.

#### Reč o SQL injekcijama

SQL injekcija je uobičajena ranjivost (SQL injekcije čine 20% ranjivosti visoke ozbiljnosti od 1988. do 2012. godine).

SQL injekcija se dešava "kada napadač može da ubaci string SQL izraza u 'upit' manipulišući unosom podataka u aplikaciju."

Jedno rešenje (ne jedino) je korišćenje pripremljenih iskaza u vašem kodu kako biste sprečili ovu vrstu napada. Toplo vam savetujem da koristite pripremljene iskaze čak i ako one uzrokuju dva poziva serveru.

### Pročitajte jedan red iz MySQL

Da bismo čitali podatke, koristićemo "SELECT" izraz.

Da biste pročitali jedan red u bazi podataka, možete koristiti metodu `QueryRow`. Ova metoda će vratiti pokazivač na instancu tipa `sql.Row`. Tip `sql.Row` ima `Scan` metodu koju možemo koristiti za ubrizgavanje podataka preuzetih iz baze podataka u Go promenljive:

```go
// data-storage/mysql/select/main.go 

func teacher(id int, db *sql.DB) (*Teacher, error) {
    teacher := Teacher{id: id}
    err := db.QueryRow("SELECT firstname, lastname FROM teacher WHERE id = ?", id).Scan(&teacher.firstname, &teacher.lastname)
    if err != nil {
        return &Teacher{}, err
    }
    return &teacher, nil
}
```

- Funkcija teacher će prihvatiti jedan `int` i "*sql.DB", kao parametar.
- Ceo broj "id" je ID nastavnika u bazi podataka.
- Ova funkcija će napraviti upit bazi podataka, sa `QueryRow`.
- Rezultat se zatim vraća putem `*sql.Row`.
- Onda koristimo `Scan` metod.

Ova metoda će izdvojiti podatke iz rezultata upita i kopirati ih u promenljive koje su prosleđene kao parametri. Imajte na umu da će Go umesto vas izvršiti konverziju tipova.

U našem primeru, kreirali smo tip "Teacher":

```go
type Teacher struct {
    id        int
    firstname string
    lastname  string
}
```

Ne možete direktno proslediti "Teacher" promenljivu metodi `Scan`, morate proslediti pokazivače na polja:

```go
Scan(&teacher.firstname, &teacher.lastname)
```

Molimo vas da obratite pažnju na dva uobičajena slučaja kada koristite `QueryRow`:

#### 0 rezultata

Metoda `Scan` će vratiti grešku:

```go
greška sql.ErrNoRows:

var ErrNoRows = errors.New("sql: no rows in result set")
```

#### 1+ rezultata

Metoda `Scan` će uzeti prvi red koji će biti vraćen.

### Pročitajte nekoliko redova iz MySQL

Kada imate upit koji će vratiti više od jednog reda, morate koristiti metodu `Query` i iterirati kroz vraćene redove.

```go
// data-storage/mysql/selectMultiple/main.go 

rows, err := db.Query("SELECT id, firstname, lastname FROM teacher ")
if err != nil {
    return nil, err
}
```

`db.Query` će vratiti element tipa `*sql.Rows`. On predstavlja tok rezultata koji šalje server baze podataka. Kada završite sa čitanjem rezultata upita, pozovite metodu `Close`. Koristićemo naredbu `defer` da je izvršimo na kraju naše funkcije.

```go
defer rows.Close()
teachers := make([]Teacher, 0)
for rows.Next() {
    // iterate over each row of the result set
}
```

Možete videti idiomatski metod za iteraciju kroz redove u prethodnom kodu.

Koristimo `for` petlju sa `rows.Next()`. Ideja je da se kreira prazan isečak tipa Teacher, a zatim da se iterira kroz svaki red:

```go
for rows.Next() {
    teacher := Teacher{}
    if err := rows.Scan(&teacher.id, &teacher.firstname, &teacher.lastname); err != nil {
        return nil, err
    }
    teachers = append(teachers, teacher)
}
```

Kada više nema redova, petlja for će se zaustaviti (jer će `rows.Next()` vratiti false). `Scan` se koristi za dobijanje podataka reda i podešavanje naših strukturnih polja.

- Poslednja instrukcija for petlje će dodati trenutnog nastavnika u "teachers" isečak.
- Na kraju izvršavanja ove petlje, imamo isečak "techers"!

Evo izvornog koda kompletne funkcije koju moramo da dizajniramo:

```go
func teachers(db *sql.DB) (*[]Teacher, error) {
    rows, err := db.Query("SELECT id, firstname, lastname FROM teacher ")
    if err != nil {
        return nil, err
    }
    defer rows.Close()
    teachers := make([]Teacher, 0)
    for rows.Next() {
        teacher := Teacher{}
        if err := rows.Scan(&teacher.id, &teacher.firstname, &teacher.lastname); err != nil {
            return nil, err
        }
        teachers = append(teachers, teacher)
    }
    return &teachers, nil
}
```

### Ažuriranje redova MySQL

Bez iznenađenja, ako želite da ažurirate red (ili više redova odjednom), morate da koristite SQL UPDATE izraz. `Exec` metod je optimalan za ovu vrstu operacije.

`Exec` vraća `sql.Result` što je tip interfejsa sa dve metode: `LastInsertId()` i `RowsAffected()`.

Poslednja metoda vam omogućava da proverite koliko redova u tabeli ste promenili.

Uzmimo primer gde želite da ažurirate ime nastavnika sa ID-om 1:

```go
// data-storage/mysql/update/main.go 

res, err := db.Exec("UPDATE teacher SET firstname = ? WHERE id = ?", "Daniel", 1)
if err != nil {
    fmt.Println(err)
    // query was not a success; something went wrong
}
```

Sa našom `sql.Result` vrednošću možemo koristiti metodu `RowsAffected()`, da proverimo da li je pogođen samo jedan red:

```go
affected, err := res.RowsAffected()
if err != nil {
    fmt.Println(err)
    return
}

if affected != 1 {
    fmt.Printf("Something went wrong %d rows were affected expected 1\n", affected)
} else {
    fmt.Println("Update is a success")
}
```

`RowsAffected()` će vratiti ili int64 ili grešku.

### Konkurencija

Tip `sql.DB` predstavlja skup od jedne ili više veza sa bazom podataka. Možemo ga bezbedno koristiti u konkurentnim programima.

## PostgreSQL baza podataka

Kao i MySQL, PostgreSQL je sistem za upravljanje relacionim bazama podataka otvorenog koda. Objavljen je 1996. godine.

### Drajver

Da bismo se povezali sa bazom podataka, moramo da koristimo PostgreSQL drajver.

Ovaj primer će koristiti drajver <https://github.com/lib/pq> koji preporučuje wiki za Go repozitorijum. Takođe izgleda da je to veoma popularan repozitorijum koji ima više od 4.000 zvezdica i 70 saradnika.

Dokumentacija drajvera može se naći na godoc-u: <https://godoc.org/github.com/lib/pq>.

### Povezivanje na PostgreSQL

Koristićemo funkciju `Open` iz `sql` paketa:

```go
// data-storage/postgresql/connection/main.go
package main

import (
    "database/sql"
    "fmt"

    _ "github.com/lib/pq"
)

func main() {
    db, err := sql.Open("postgres", "host=localhost port=5432 user=postgres dbname=school password=pg123 sslmode=disable")
    if err != nil {
        fmt.Println(err)
        return
    }
    err = db.Ping()
    if err != nil {
        fmt.Println(err)
        return
    }
}
```

- `sql.Open` - Možete primetiti da je prvi parametar "postgres".
- Interno Go će izabrati i koristiti odgovarajući drajver.
- Zatim, drugi parametar je DSN (naziv izvora podataka).
- Ovo je string koji definiše kako se povezati sa bazom podataka. Malo se razlikuje od MySQL
  verzije i mislim da je izražajniji:
- `"host=localhost port=5432 user=postgres dbname=school password=pg123 sslmode=disable"`

U stvarnoj aplikaciji, aktiviraćete `ssl` mode (ovde smo ga onemogućili u svrhu testiranja), a pored toga, ne preporučuje se čuvanje tih parametara u izvornom kodu. Trebalo bi da napravite DSN stringove od vrednosti konfiguracione datoteke, na primer, ili od promenljivih okruženja.

Dostupni su i drugi parametri:

- `connect_timeout` - Ovo je vreme čekanja na otvaranje veze. Ako ga podesite na nulu ili ništa,
  čekaće se neograničeno.
- `sslcert`, `sslkey`, `sslrootcert`: se koriste za SSL podršku.

### Kreiranje tabele u PostgrSQL

PostgreSQL koristi SQL jezik, ali u poređenju sa MySQL-om, sintaksa `CREATE TABLE` nije ista. Postgres tipovi nisu isti kao MySQL tipovi.

Evo skripte za kreiranje slične tabele:

```sql
CREATE TABLE public.teacher
(
    id serial,
    create_time timestamp without time zone,
    update_time timestamp without time zone,
    firstname character varying(255),
    lastname character varying(255),
    PRIMARY KEY (id)
);
```

Pojavljuju se neke specifičnosti:

- Tip `serial` se koristi za primarni ključ
- Možete dodati vremenske zone vremenskim oznakama (u MySQL-u možete odrediti vremensku zonu za
  svaku konekciju, podrazumevano će to biti vremenska zona servera).
- Navedeni tip `character varying(255)` može se koristiti za čuvanje stringova.

Da bismo kreirali tabelu, izvršićemo skriptu sa `Exec` metodom:

```go
// load SQL script
// use os.Open("create_table.sql") to open the script
// then ioutil.ReadAll to load it's content
// execute the script
res, err := db.Exec(string(b))
if err != nil {
    fmt.Println(err)
    return
}
// success, table created !
```

Sintaksa je slična onoj koja se koristi sa MySQL bazom podataka.

### Umetanje u PostgreSQL

Za umetanje (ili ažuriranje), odgovarajuća metoda koju treba koristiti je `Exec` za MySQL baze podataka. Možemo koristiti ovu metodu za PostgreSQL baze podataka, ali nije optimalna. `Exec` će vratiti `Row`, što definiše metod `LastInsertId`. Ova metoda će izvršiti zahtev

```sql
SELECT LAST_INSERT_ID();
```

Sa PostgreSQL-om, ova metoda će vratiti grešku (ova funkcija ne postoji). Umesto toga, možemo dodati SQL INSERT upitu: `RETURNING id`.

Kompletan SQL INSERT zahtev je:

```sql
INSERT INTO public.teacher (create_time, firstname, lastname)
VALUES (NOW(),$1, $2)
RETURNING id;
```

PostgreSQL ne podržava "?" znak, umesto toga morate koristiti rezervisana mesta ( \$1, \$2,..., \$n ).

Moraćemo da koristimo metodu, `QueryRow` zatim i poziv metode `Scan` da bismo ubacili identifikator:

```go
// data-storage/postgresql/insert/main.go 

func createTeacher(firstname string, lastname string, db *sql.DB) (int, error) {
    insertedId := 0
    err := db.QueryRow("INSERT INTO public.teacher (create_time, firstname, lastname) VALUES (NOW(),$1, $2) RETURNING id;", firstname, lastname).Scan(&insertedId)
    if err != nil {
        return 0, err
    }
    if insertedId == 0 {
        return 0, errors.New("something went wrong id inserted is equal to zero")
    }
    return insertedId, nil
}
```

- Funkcija "createTeacher" prima tri argumenta ("firstname", "lastname" i pokazivač na "sql.DB").
- Pokazivač na promenljivu `insertedId` se prosleđuje kao argument metodi `Scan`.
- Go će ažurirati vrednost `insertedId` rezultatom SQL upita (što je umetnuti identifikator).

Poziv `db.QueryRow(..).Scan(...)` vraća grešku ako nešto pođe po zlu.

### Pročitaj jedan red u PostgreSQL

Kreirali smo strukturu tipa Teacher koja će čuvati podatke vezane za jednog nastavnika u našoj bazi podataka.

```go
type Teacher struct {
    id        int
    firstname string
    lastname  string
}
```

Evo primera funkcije za preuzimanje nastavnika u bazi podataka

```go
// data-storage/postgresql/select/main.go 

func teacher(id int, db *sql.DB) (*Teacher, error) {
    teacher := Teacher{}
    err := db.QueryRow("SELECT id, firstname, lastname FROM teacher WHERE id > $1 ", id).Scan(&teacher.id, &teacher.firstname, &teacher.lastname)
    if err != nil {
        return &teacher, err
    }
    return &teacher, nil
}
```

`QueryRow` će vratiti jedan red iz baze podataka.

- Ako vaš upit vrati nekoliko redova, biće obrađen samo prvi red.
- Funkcija `Scan` će izvući vrednosti iz vraćenog reda.

### Pročitajte nekoliko redova iz postgreSQL

Da biste pročitali više redova, trebalo bi da koristite `Query` metod:

```go
// data-storage/postgresql/select-multiple/main.go 

rows, err := db.Query("SELECT id, firstname, lastname FROM teacher")

Upit vraća ili pokazivač na `sql.Rows` ili `error`.

Morate pozvati `rows.Close()` da biste prekinuli zatvorili `rows`:

// close the connexion at the end of the function
defer rows.Close()
```

Zatim, iteriramo kroz redove sa `rows.Next()`:

```go
for rows.Next() {
    // treat one row
}
```

Unutar `for` petlje, izvući ćemo podatke iz trenutnog reda pomoću metode `Scan`:

```go
err := rows.Scan(&id, &firstname, &lastname)
```

### Metod skeniranja

Funkcija `convertAssign` (u paketu `sql`) će biti pozvana:

```go
func convertAssign(dest, src interface{}) error {
    //...
}
```

- Argument `dest` je pokazivač na našu odredišnu promenljivu.
- `src` je izvor, ono što je drajver baze podataka vratio.
- Funkcija će preuzeti tip argumenata `dest` i `src`. Funkcija ima preko 200 linija koda. Ovde
  nećemo  reprodukovati ceo sadržaj, već samo prve linije.

  ```go
  // extract of function convertAssign (line 209, src/database/sql.go)
  func convertAssign(dest, src interface{}) error {
      // Common cases, without reflect.
      switch s := src.(type) {
      case string:
          switch d := dest.(type) {
          case *string:
              if d == nil {
                  return errNilPtr
              }
              *d = s
              return nil
          case *[]byte:
              if d == nil {
                  return errNilPtr
              }
              *d = []byte(s)
              return nil
          case *RawBytes:
              if d == nil {
                  return errNilPtr
              }
              *d = append((*d)[:0], s...)
              return nil
          }
          //...
  }
    ```

Funkcija se sastoji od velike `switch` naredbe. Prebacujemo tip `src`. Tip se preuzima naredbom :

```go
s := src.(type)
```

Ako je `src` promenljiva `string` tipa onda ćemo izdvojiti tip argumenta `dest`:

```go
d := dest.(type)
```

Pokrenuta je još jedan `switch` iskaz.

Ako je `src` `string`, `dest` je `*[]byte`. Izvršava se sledeći isečak koda:

```go
if d == nil {
    return errNilPtr
}
*d = []byte(s)
return nil
```

Funkcija će testirati da li `dest` pokazivač nije nil. Ako nije, vrednost `dest` će se promeniti u `[]byte(s)`:

```go
*d = []byte(s)
```

### Ažuriranje redova PostgreSQL

Kao i za MySQL, odgovarajuća metoda je `Exec`. Ova metoda će vratiti tip koji implementira `sql.Result`. `res.RowsAffected()` vratiće broj redova u tabeli na koje je uticala izjava UPDATE. Možete kontrolisati uticaj vašeg zahteva na tabelu kontrolisanjem ove vrednosti:

```go
// data-storage/postgresql/update/main.go 

res, err := db.Exec("UPDATE teacher SET firstname = $1 WHERE id = $2", "Daniel", 1)
// check that there is no error
// get the number of affected rows
affected, err := res.RowsAffected()
//...
```

Jedina razlika u odnosu na MySQL je format rezervisanih mesta. Umesto "?" trebalo bi da koristite "$n" (gde je n redni broj rezervisanog mesta).

## Testirajte sebe

### Pitanja i Odgovori

1. Šta je režim datoteke?
   - Režim datoteke predstavlja skup dozvola povezanih sa datotekom.
2. Kako kreirati vezu sa SQL bazom podataka?
   - Instalirajte odgovarajući drajver
   - Pozovi sql.Open
3. Koju metodu možete koristiti za izvršavanje INSERT i UPDATE upita na
   SQL bazi podataka?
   - Exec
4. Koju metodu možete koristiti za analizu rezultata SQL zahteva?
   - Scan

### Ključno

- Da biste kreirali datoteku, koristite os.Create
- Da biste kreirali datoteku i direktno u nju upisali podatke, možete koristiti ioutil.WriteFile
- Na UNIX sistemima, svaka datoteka ima skup dozvola
- Možete promeniti dozvole datoteke pomoću funkcije os.Chmod
- Nakon instaliranja drajvera, možete koristiti SQL baze podataka u svom programu
- Ne zaboravite da uvezete svoj drajver sa praznom datotekom za uvoz.  
  sql.Open može se koristiti za otvaranje nove veze sa bazom podataka
  - Vratiće pokazivač na sql.DB koji ima sledeće metode  
    Exec: za izvršavanje sirovih upita (umetanje, ažuriranje,...)  
    QueryRow: da biste izabrali jedan red  
    Query: za izbor više redova  

  - Rezultate upita možete pretvoriti u promenljive pomoću metode Scan.

[28 Datumi i vreme][28] | [00 Sadržaj][00] | [30 Konkurentnost][30]

[28]: 28_Datumi_i_vreme.md
[00]: 00_Sadržaj.md
[30]: 30_Konkurentnost.md
