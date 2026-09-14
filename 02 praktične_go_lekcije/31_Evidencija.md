
# 31 Evidencija

[30 Konkurentnost][30] | [00 Sadržaj][00] | [32 Šabloni][32]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je log?
- Kako se koristi standardni paket za evidentiranje?
- Kako podesiti i koristiti Logrus?
- Šta su nivoi logova?

**Obrađeni tehnički koncepti**:

- Dnevnik
- Nivo logova
- Ozbiljnost
- Sistemski log
- Demon

## Uvod

Log je strukturirana i raščlanjiva poruka koju generiše aplikacija instalirana na sistemu. Zadatak programera je da doda logove svojoj aplikaciji.

Aplikacija bez grešaka ne postoji; čak i ako primenite najbolje prakse, u određenom trenutku ćete se suočiti sa kvarom. Kvar može biti popravljiv, što znači da vaš sistem može da ga podnese i nastavi da radi. Takođe može biti nepopravljiv. U ovom poslednjem slučaju, vaša aplikacija će prestati da radi.

### Neuspeh je čest

Zamislite sledeću situaciju: vaš telefon počinje da zvoni usred noći. Generalni direktor je na drugom kraju linije. Obaveštava vas da je sajt koji ste razvili nedostupan. Mušterije ne mogu da naručuju. Morate ga sada popraviti. Povezujete se sa mašinom, ponovo pokrećete program, ali on se neće ponovo pokrenuti. Sa stresom koji dostiže visok nivo, pokušaćete da shvatite zašto.

Pomoću logova možete istražiti i otkriti uzrok greške. Bez logova to može biti mnogo teže. Ali čak ni sa logovima, vaš zadatak neće biti lak.

Teškoća će se smanjiti ako znate:

- Kada vaš sistem otkaže
- Gde je greška prijavljena
- Koji korisnik/operacija je bio uključen kada je sistem otkazao.

Najgora stvar koja može da se desi aplikaciji jeste da proizvede greške, ali te greške ostaju neprimećene, a bagovi su i dalje tu. Dobra navika evidentiranja vam omogućava da pratite svoju aplikaciju i održavate je u dobrom stanju.

### Primera logova

Evo primera logova:

```sh
2018/10/14 20:08:08 System is starting
2018/10/14 20:08:08 Retrieving user 12
2018/10/14 20:08:08 Compute the total of invoice 1262663663
```

Kao što vidite, imamo vremensku oznaku praćenu nizom, što je poruka.

## Standardni log paket

U standardnoj biblioteci `log` paket implementira veoma jednostavan, ali moćan loger.

Prvo što treba da uradite je da definišete izlaz vašeg logera. Ako ga ne definišete, to će biti standardna greška. U primeru, beležićemo podatke u datoteku:

```go
// logging/main.go 

f, err := os.OpenFile("app.log", os.O_APPEND|os.O_CREATE|os.O_WRONLY, 0644)
if err != nil {
    log.Fatalf("error opening file: %v", err)
}
defer f.Close()
log.SetOutput(f)
```

Koristimo paket `os` da otvorimo našu datoteku ( `os.OpenFile` ), poziv ove funkcije uzima sledeće parametre:

- Ime datoteke: "app.log".
- Zastavice opisuju kako sistem treba da otvori datoteku. U našem slučaju, želimo da joj dodamo podatke ( `O_APPEND` ). kreiramo je ako ne postoji ( `O_CREATE` ) i otvorimo je u režimu samo za pisanje ( `O_WRONLY` ).
- Skup dozvola `0644` znači da korisnik može da čita i piše, grupa i drugi mogu da pišu. Ako datoteka ne postoji, primeniće se skup dozvola.

Pozivamo `defer.Close()`, da zatvorimo datoteku kada se glavna funkcija vrati.

Zatim postavljamo izlaz standardnog logera na ovu otvorenu datoteku ( `log.SetOutput` uzima `io.Writer`)

Spremni smo za evidentiranje:

```go
// logging/main.go 
log.Println("System is starting")
log.Println("Retrieving user 12")
log.Println("Compute the total of invoice 1262663663")
```

Za evidentiranje, možete koristiti funkcije `Print`, `Printf`, `Println`,..., `Fatal`, `Fatalf`...

### log.Print, log.Println, log.Printf

```sh
log.Print("System failure caused by lost DB connexion")će ispisati standardni loger:
2018/10/14 20:08:08 System failure caused by lost DB connexion
```

Ako pozovete `log.Println("xyz")` rezultat izvršenja fmt.Sprintln("xyz") će biti ispisan u standardni loger:

```go
log.Println("System failure caused by lost DB connexion")
```

`log.Printf` je zanimljivo. Omogućava vam da dodate formatirane promenljive u vašu poruku. Možete koristiti specifikatore formata da kažete Go-u kako treba da prikaže promenljivu.

```go
dbIp:="192.168.210.12"
dbPort := 3306
log.Printf("System failure caused by lost DB connexion to host %s at port %d", dbIp,dbPort)
```

Izveštavaće sledeće:

```sh
2018/10/16 13:29:07 System failure caused by lost DB connexion to host 192.168.210.12 at port 3306
```

### log.Fatal, log.Fatalln, log.Fatalf

Ako pozovete jednu od tih funkcija, program će se odmah isključiti. Interno `log.Fatal(ln|f)` će se pozvati `os.Exit(1)`. Sve će biti zaustavljeno, a odložene funkcije se neće pozivati. To je neka vrsta dugmeta za hitne slučajeve. Program se ne prekida pravilno kada izvršite fatalnu grešku.

- **log.Fatal**  
  Ispisuje na standardnom logeru povratnu vrednost `fmt.Sprint` i poziva `os.Exit(1)`

- **log.Fatalf**  
  Slično kao `log.Fatal` sa mogušnošću proširenja izlaza.

- **log.Fatalln**  
  Povratna vrednost `fmt.Sprintln` će biti ispisana standardnom logeru i biće pozvana `os.Exit(1)`

### log.Panic, log.Panicln, log.Panicf

Ista logika kao gore se koristi za pozive funkcije `log.Panic[f][ln]`. nema razlike, Poziva se `fmt.Sprint`, `fmt.Sprinf` ili `fmt.Sprintln` "ispod haube".

Jedina razlika je što imamo poziv `panic` ( umesto `os.Exit(1)`)

Kada loger zove panic u vaše ime:

- Bilo koje odložene funkcije definisane u trenutnoj funkciji biće pozvane
- Trenutna funkcija će se vratiti svom pozivaocu
- U pozivaocu će biti pozvane sve definisane odložene funkcije, a pozivalac će se vratiti na funkciju koja ga je pozvala, itd...

Poziv `panic` će pozvati sve odložene funkcije koje su definisane u steku poziva goroutine.

## Logrus

Funkcije definisane na standardnom logeru su ograničene i možda ćete želeti da dodate dodatne funkcije logeru vaše aplikacije. Možete koristiti popularni modul pod nazivom `logrus` (više od 17 hiljada zvezdica na Github-u).

### Podešavanje

Prvo, potrebno je da ga instalirate (nakon inicijalizacije modula sa `go mod init your/module/path`):

```sh
go get github.com/sirupsen/logrus
```

Zatim možete konfigurisati svoj loger:

```go
file, err := os.OpenFile("app.log", os.O_CREATE|os.O_WRONLY|os.O_APPEND, 0644)
if err != nil {
    // handle error
}
log.SetOutput(file)
```

Ovde ćemo evidentirati u datoteku. Takođe možete evidentirati u standardnu grešku (stderr). U ovom slučaju, možete izostaviti poslednje redove.

Onda možete početi sa evidentiranjem:

```go
log.Print("my first log")
```

Evo rezultata:

```sh
time="2021-02-19T12:28:57+01:00" level=info msg="my first log"
```

### Nivoi

Zanimljiva karakteristika logrusa je rukovanje nivoima logova. Svaki log ima nivo kritičnosti. Sa nivoom možete primeniti filtere na svoje log datoteke. Evo primera logova za svaki dostupni nivo:

```go
// logging/main.go

log.Trace("going to mars")
log.Debug("connected, received buffer from worker")
log.Info("connection successful to db")
log.Warn("something went wrong with x")
log.Error("an error occurred in worker x")
log.Fatal("impossible to continue exec")
log.Panic("system emergency shutdown")
```

- **trace**: veoma detaljne informacije o izvršavanju programa, obično u svrhu
  otklanjanja grešaka
- **debug**: koristi se za prikazivanje veoma detaljnih informacija,
  obično u svrhu debagovanja
- **info**: informativni dnevnici
- **warn**: nekritična greška ili događaj koji treba da
  ispravimo. Aplikacija i dalje može da radi.
- **error**: greška koju treba da ispravimo. Aplikacija i dalje može da radi.
- **fatal**: kritična greška, aplikacija će biti zaustavljena
- **panic**: najviši nivo kritičnosti, aplikacija će biti zaustavljena

Zatim možete podesiti na kom nivou će vaša aplikacija proizvoditi logove.

Podrazumevano, nivoi `trace` i `debug` se ne štampaju.

Na primer, ako želite da prikažete sve nivoe, koristite sledeći red:

```go
log.SetLevel(log.TraceLevel)
```

Ako želite da prikažete samo greške (i takođe fatalne greške / greške panike):

```go
log.SetLevel(log.ErrorLevel)
```

### Polja

Možete dodati polja svojim porukama dnevnika. Polje se sastoji od ključa i vrednosti. Rešenje za praćenje može lako da analizira polja.

```go
// logging/main.go

workerLogger := log.WithFields(log.Fields{
    "source": "worker",
})
workerLogger.Info("worker has finished processed task")

mysqlLogger := log.WithFields(log.Fields{
    "source": "db",
    "dbType": "mysql",
})
mysqlLogger.Error("impossible to establish connexion")
```

Generisaće sledeće logove:

```sh
time="2021-02-19T12:49:47+01:00" level=info msg="worker has finished processed task" source=worker
time="2021-02-19T12:49:47+01:00" level=error msg="impossible to establish connexion" dbType=mysql source=db
```

## Format evidentiranja: Syslog pristup

Syslog je otvoreni standard za evidentiranje poruka. Ovaj odeljak će detaljno opisati neke najbolje prakse izvučene iz `RFC 5424`, koji određuje format poruka dnevnika. Evo liste polja koja možete dodati svom logeru da biste povećali njegovu efikasnost. RFC daje skup svih mogućih mogućnosti.

Syslog mogućnosti opisuje koji deo sistema je poslao poruku dnevnika

| Kod | Objekat |
| :-: | ------- |
| 0 | poruke jezgra |
| 1 | poruke na nivou korisnika |
| 2 | poštanski sistem |
| 3 | sistemski demoni |
| 4 | bezbednosne/autorizacione poruke |
| 5 | poruke koje interno generiše syslogd |
| 6 | podsistem linijskog štampača |
| 7 | podsistem mrežnih vesti |
| 8 | UUCP podsistem |
| 9 | demon sata |
| 10 | bezbednosne/autorizacione poruke |
| 11 | FTP demon |
| 12 | NTP podsistem |
| 13 | revizija dnevnika |
| 14 | upozorenje dnevnika |
| 15 | demon sata (napomena 2) |
| 16 | lokalna upotreba 0 |
| 17 | lokalna upotreba 1 |
| 18 | lokalna upotreba 2 |
| 19 | lokalna upotreba 3 |
| 20 | lokalna upotreba 4 |
| 21 | lokalna upotreba 5 |
| 22 | lokalna upotreba 6 |
| 23 | lokalna upotreba 7 |

Go Syslog paket već ima nabrojane mogućnosti, tako da ga ne morate ponovo razvijati:

```go
//src/log/syslog/syslog.go
//...
const (
    // Facility.

    // From /usr/include/sys/syslog.h.
    // These are the same up to LOG_FTP on Linux, BSD, and OS X.
    LOG_KERN Priority = iota << 3
    LOG_USER
    LOG_MAIL
    LOG_DAEMON
    LOG_AUTH
    LOG_SYSLOG
    LOG_LPR
    LOG_NEWS
    LOG_UUCP
    LOG_CRON
    LOG_AUTHPRIV
    LOG_FTP
    _ // unused
    _ // unused
    _ // unused
    _ // unused
    LOG_LOCAL0
    LOG_LOCAL1
    LOG_LOCAL2
    LOG_LOCAL3
    LOG_LOCAL4
    LOG_LOCAL5
    LOG_LOCAL6
    LOG_LOCAL7
)
//...
```

Svaka poruka u dnevniku nema istu ozbiljnost; neke poruke su čiste informacije, ali druge zahtevaju da preduzmete hitne mere kako biste održali uslugu u radu. Na primer, ako vam ponestaje prostora na disku, dnevnik bi trebalo da bude na visokom nivou ozbiljnosti jer ako ne preduzmete hitne mere, to će dovesti do pada usluge.

Specifikacija nam daje i listu nivoa ozbiljnosti koja je zanimljiva:

**Sistemski dnevnik nivoa ozbiljnosti**:

| Kod | Nivo |
| :-: | ---- |
| 0 | Vanredno stanje: sistem je neupotrebljiv |
| 1 | Upozorenje: moramo odmah preduzeti mere |
| 2 | Kritično: kritični uslovi |
| 3 | Greška: uslovi greške |
| 4 | Upozorenje: uslovi upozorenja |
| 5 | Obaveštenje: normalno, ali značajno stanje |
| 6 | Informativno: informativne poruke |
| 7 | Otklanjanje grešaka: poruke na nivou otklanjanja grešaka |

Gp paket Syslog takođe ima nabrajanje za ozbiljnost:

```go
//src/log/syslog/syslog.go
//...
const (
    // Severity.

    // From /usr/include/sys/syslog.h.
    // These are the same on Linux, BSD, and OS X.
    LOG_EMERG Priority = iota
    LOG_ALERT
    LOG_CRIT
    LOG_ERR
    LOG_WARNING
    LOG_NOTICE
    LOG_INFO
    LOG_DEBUG
)
//...
```

Ovo je datum, vreme i vremenska zona kada je dnevnik zapisan (i u proširenju kada se događaj desio). RFC 5424 preporučuje korišćenje određenog formata (navedenog u RFC 3339). Da biste dobili vremensku oznaku u traženom formatu, možete koristiti sledeći kod:

```go
timestamp := time.Now().Format(time.RFC3339)
```

Kada agregirate logove, oni mogu doći sa različitih mašina; morate biti u stanju da znate koja mašina je naišla na problem, jednostavan način da ga dobijete je da pozovete funkciju `os.Hostname()`. Ova poslednja instrukcija će tražiti od jezgra da odgovori na pitanje:

```go
hostname := os.Hostname()
```

Popunite ovo svojstvo imenom vašeg programa. Ovo će vam omogućiti da u logovima razlikujete koja aplikacija je proizvela logove. To može biti navika koja štedi vreme ako imate nekoliko aplikacija koje rade na istoj mašini. Primer: ruter

Ako želite da vaši logovi budu upotrebljivi, morate ih učiniti parsabilnim. Da biste to uradili, možete dodati strukturirane podatke u svoj log. Pod strukturiranim podacima podrazumevamo skup parova ključ-vrednost koji donose kontekst i dodatne informacije vašem logu.

Vrednost prioriteta (pod nazivom `PRIVAL`) je jednaka `facility * 8 + severity`

### Primer dnevnika formatiranog prema RFC 5424

```sh
<9>1 2018-10-17T20:56:58+02:00 Mx.local router help me please
```

### Pošaljite logove demonu

Možemo slati logove demonu za logove. Demon je program koji neprekidno radi u pozadini. Demon log je program koji je odgovoran za prikupljanje logova iz aplikacija:

```go
logwriter, err := syslog.New(syslog.LOG_WARNING|syslog.LOG_DAEMON, "loggingTestProgram")
if err != nil {
    log.Fatal(err)
}
logwriter.Emerg("emergency sent to syslog. TEST2")
```

Dodaćemo log u sistemske logove mašine. Pošto je u pitanju vanredna situacija, biće emitovan na otvorene terminale (pogledajte sliku).

## Testirajte sebe

1. Poziv funkcije `log.Fatal` će izvršiti odložene funkcije. Tačno ili netačno?

   - Netačno
   - Odložene funkcije se neće pokrenuti
   - Program će se završiti sa kodom 1

2. Poziv funkcije `log.Panic` će izvršiti odložene funkcije. Tačno ili netačno?
   - Tačno

3. Poređajte sledeće nivoe logova po kritičnosti (od najnižeg do najvišeg):
   fatal, error, panica, info, trace
   - trace, info, error, panic, fatal

4. Zašto se dodavanje strukturiranih podataka u vaše logove smatra dobrom
   praksom?
   - Strukturirani podaci se lako analiziraju pomoću sistema za praćenje
   - Shodno tome, alati i timovi za praćenje mogu efikasnije iskoristiti logove
     sa strukturiranim podacima.

## Ključno

- Aplikacija treba da proizvede informativne logove
- Zapisi mogu pomoći programerima da otklone greške u svojim aplikacijama kada dođe do kvara.
- Možemo pisati logove u datoteci ili u bilo kom tipu koji implementira interfejs `io.Writer`
- Paket `log` iz standardne biblioteke je dobra polazna tačka kada treba da dodate logove vašoj aplikaciji.
- Kada se `log.Fatal` poziva. odložene funkcije se ne izvršavaju
- Kada se `log.Panic` poziva. izvršavaju se odložene funkcije
- Modul `logrus` nudi zanimljive funkcije kao što su nivoi logova, opcije formatiranja i polja
- Dodavanje parsabilnih polja u vaše logove je dobra praksa; olakšava ih analizu.

[30 Konkurentnost][30] | [00 Sadržaj][00] | [32 Šabloni][32]

[30]: 30_Konkurentnost.md
[00]: 00_Sadržaj.md
[32]: 32_Šabloni.md
