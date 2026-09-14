
# Kanali u Go-u

[Funkcije][0207] | [Sadržaj][00] | [Metode][0209]

Kanal je važna ugrađena funkcija u Gou. To je jedna od karakteristika koje Go čine jedinstvenim. Zajedno sa još jednom jedinstvenom funkcijom, goroutine , kanal čini istovremeno programiranje praktičnim, zabavnim i smanjuje teškoće istovremenog programiranja.

Kanal uglavnom deluje kao tehnika sinhronizacije konkurentnosti . Ovaj članak će navesti sve koncepte, sintaksu i pravila vezana za kanale. Da bi se bolje razumeli kanali, jednostavno su opisani i unutrašnja struktura kanala i neki detalji implementacije od strane standardnog Go kompajlera/okruženja za izvršavanje.

Informacije u ovom članku mogu biti pomalo izazovne za nove gofere. Neke delove ovog članka možda će biti potrebno pročitati nekoliko puta da bi se u potpunosti razumeli.

## Uvod u kanal

Jedan predlog (koji je dao Rob Pajk ) za konkurentno programiranje je da ne (dozvoljavate izračunavanjima) komuniciraju deljenjem memorije, već da (dozvoljavate im) da dele memoriju komunikacijom (kroz kanale) . (Svako izračunavanje možemo posmatrati kao gorutinu u Go programiranju.)

Komunikacija deljenjem memorije i deljenje memorije komunikacijom su dva načina programiranja u konkurentnom programiranju. Kada gorutine komuniciraju deljenjem memorije, koristimo tradicionalne tehnike sinhronizacije konkurentnosti, kao što su mutex brave, da bismo zaštitili deljenu memoriju i sprečili trke podataka. Možemo koristiti kanale za implementaciju deljenja memorije komunikacijom.

Go pruža jedinstvenu tehniku sinhronizacije konkurentnosti, kanal. Kanali omogućavaju gorutinama da dele memoriju komunikacijom. Kanal možemo posmatrati kao interni FIFO (prvi uđe, prvi izađe) red unutar programa. Neke gorutine šalju vrednosti u red (kanal), a neke druge gorutine primaju vrednosti iz reda.

Uz prenos vrednosti (kroz kanale), vlasništvo nad nekim vrednostima može se prenositi i između gorutina. Kada gorutina pošalje vrednost kanalu, možemo videti da gorutina oslobađa vlasništvo nad nekim vrednostima (kojima se može pristupiti preko poslate vrednosti). Kada gorutina primi vrednost iz kanala, možemo videti da gorutina stiče vlasništvo nad nekim vrednostima (kojima se može pristupiti preko primljene vrednosti).

Sigurno je da se vlasništvo neće prenositi zajedno sa komunikacijom kanala.

Vrednosti (čija se vlasništva prenose) se često referenciraju (ali nije potrebno da budu referencirane) prenetom vrednošću. Imajte na umu da ovde, kada govorimo o vlasništvu, mislimo na vlasništvo sa logičkog stanovišta. Go kanali mogu pomoći programerima da lako pišu kod bez trka podataka, ali Go kanali ne mogu sprečiti programere da pišu loš konkurentni kod sa nivoa sintakse.

Iako Go podržava i tradicionalne tehnike sinhronizacije konkurentnosti, samo je kanal (channel) građanin prve klase u Gou. Kanal je jedna vrsta tipova u Gou, tako da možemo koristiti kanale bez uvoza bilo kakvih paketa. S druge strane, te tradicionalne tehnike sinhronizacije konkurentnosti su obezbeđene u standardnim paketima synci .sync/atomic

Iskreno, svaka tehnika sinhronizacije konkurentnosti ima svoje najbolje scenarije upotrebe. Ali kanal ima širi opseg primene i ima veću raznolikost u korišćenju . Jedan problem kanala je taj što je iskustvo programiranja sa kanalima toliko prijatno i zabavno da programeri često čak više vole da koriste kanale za scenarije za koje kanali nisu najbolji.

## Tipovi i vrednosti kanala

Kao i niz, srez i mapa, svaki tip kanala ima tip elementa. Kanal može da prenosi samo vrednosti tipa elementa kanala.

Tipovi kanala mogu biti dvosmerni ili jednosmerni. Pretpostavimo Tda je proizvoljan tip,

- chan Toznačava dvosmerni tip kanala. Kompilatori omogućavaju i prijem vrednosti iz i slanje vrednosti ka dvosmernim kanalima.

- chan<- Toznačava tip kanala samo za slanje. Kompilatori ne dozvoljavaju primanje vrednosti iz kanala samo za slanje.

- <-chan Toznačava tip kanala samo za prijem. Kompilatori ne dozvoljavaju slanje vrednosti kanalima samo za prijem.

T se naziva tip elementa ovih tipova kanala.

Vrednosti tipa dvosmernog kanala chan Tmogu se implicitno konvertovati i u tip samo za slanje chan<- Ti u tip samo za prijem <-chan T, ali ne i obrnuto (čak i ako su eksplicitno). Vrednosti tipa samo za slanje chan<- Tne mogu se konvertovati u tip samo za prijem <-chan Ti obrnuto. Imajte na umu da <-su znakovi u literalima tipa kanala modifikatori.

Svaka vrednost kanala ima kapacitet, što će biti objašnjeno u sledećem odeljku. Vrednost kanala sa nultim kapacitetom naziva se nebaferovani kanal, a vrednost kanala sa kapacitetom različitim od nule naziva se baferovani kanal.

Nulte vrednosti tipova kanala su predstavljene unapred deklarisanim identifikatorom nil. Vrednost kanala koja nije nula mora se kreirati korišćenjem ugrađene makefunkcije. Na primer, make(chan int, 10)će kreirati kanal čiji je tip elementa int. Drugi argument makepoziva funkcije određuje kapacitet novokreiranog kanala. Drugi parametar je opcionalan i njegova podrazumevana vrednost je nula.

## Poređenja vrednosti kanala

Svi tipovi kanala su uporedivi tipovi.

Iz članka o delovima vrednosti , znamo da su vrednosti kanala koje nisu nula višedelne vrednosti. Ako je jedna vrednost kanala dodeljena drugoj, dva kanala dele iste osnovne delove. Drugim rečima, ta dva kanala predstavljaju isti interni objekat kanala. Rezultat njihovog poređenja je true.

## Operacije kanala

Postoji pet operacija određenih kanalom. Pretpostavimo da je kanal ch, njihova sintaksa i pozivi funkcija ovih operacija su navedeni ovde.

- Zatvorite kanal koristeći sledeći poziv funkcije

  close(ch)

  gde je close ugrađena funkcija. Argument closepoziva funkcije mora biti vrednost kanala, a kanal chne sme biti kanal samo za prijem.

- Pošaljite vrednost, v, kanalu koristeći sledeću sintaksu

  ch <- v

  gde vmora biti vrednost koja se može dodeliti tipu elementa channel ch, i kanal chne sme biti kanal samo za prijem. Imajte na umu da <-je ovde operator slanja kanala.

- Primite vrednost iz kanala koristeći sledeću sintaksu

  <-ch

  Operacija prijema kanala uvek vraća barem jedan rezultat, koji je vrednost tipa elementa kanala, a kanal chne sme biti kanal samo za slanje. Imajte na umu da <-je ovde operator prijema kanala. Da, njegova reprezentacija je ista kao i operator slanja kanala.

  U većini scenarija, operacija prijema kanala se posmatra kao izraz sa jednom vrednošću. Međutim, kada se operacija kanala koristi kao jedini izraz izvorne vrednosti u dodeli, može rezultirati drugom opcionom netipizovanom bulovskom vrednošću i postati izraz sa više vrednosti. Netipizovana bulovska vrednost označava da li se prvi rezultat šalje pre nego što se kanal zatvori. (U nastavku ćemo saznati da možemo primiti neograničen broj vrednosti iz zatvorenog kanala.)

  Dvokanalne operacije prijema koje se koriste kao izvorne vrednosti u dodelama:

  v = <-ch
  v, sentBeforeClosed = <-ch

- Upitajte kapacitet bafera vrednosti kanala koristeći sledeći poziv funkcije

  cap(ch)

  gde capje ugrađena funkcija koja je uvedena u kontejnere u Go-u . Vraćeni rezultat cappoziva funkcije je intvrednost.

- Upitajte trenutni broj vrednosti u baferu vrednosti (ili dužinu) kanala koristeći sledeći poziv funkcije

  len(ch)

  gde lenje ugrađena funkcija koja je takođe već predstavljena. Povratna vrednost lenpoziva funkcije je intvrednost. Dužina rezultata je broj elemenata koji su već uspešno poslati na kanal koji se traži, ali još nisu primljeni (izbačeni).

Većina osnovnih operacija u jeziku Go nisu sinhronizovane. Drugim rečima, nisu bezbedne za konkurentnost. Ove operacije uključuju dodeljivanje vrednosti, prosleđivanje argumenata i manipulacije elementima kontejnera itd. Međutim, sve upravo predstavljene operacije kanala su već sinhronizovane, tako da nisu potrebne dalje sinhronizacije da bi se ove operacije bezbedno izvršile.

Kao i većina drugih operacija u programu Go, dodeljivanje vrednosti kanala nije sinhronizovano. Slično tome, dodeljivanje primljene vrednosti drugoj vrednosti takođe nije sinhronizovano, iako je svaka operacija prijema kanala sinhronizovana.

Ako je upitani kanal kanal nil, obe ugrađene funkcije capi lenvraćaju nulu. Dve operacije upita su toliko jednostavne da kasnije neće biti daljih objašnjenja. U stvari, ove dve operacije se retko koriste u praksi.

Operacije slanja, primanja i zatvaranja kanala biće detaljno objašnjene u sledećem odeljku.

## Detaljna objašnjenja za rad kanala

Da bi objašnjenja za rad kanala bila jednostavna i jasna, u ostatku ovog članka, kanali će biti klasifikovani u tri kategorije:

- nulti kanali.
- ne-nulti zatvoreni kanali.
- ne-nulti nezatvoreni kanali.

Sledeća tabela jednostavno sumira ponašanja za sve vrste operacija koje se primenjuju na nil, zatvorene i nezatvorene ne-nil kanale.

| Operacija | Nulti kanal | Zatvoreni kanal | Nezatvoreni nenulti kanal |
| --------- | ----------- | --------------- | ------------------------- |
| Zatvori | panika | panika | uspeti zatvoriti (C) |
| Pošalji vrednost | blokirati zauvek | panika | blokirati ili uspeti slanje (B) |
| Primite vrednost od | blokirati zauvek | nikada ne blokiraj (D) | blokirati ili uspeti da primi (A) |

Za pet slučajeva prikazanih bez nadpisa, ponašanja su veoma jasna.

- Zatvaranje kanala koji je nulti ili već zatvoren proizvodi paniku u trenutnoj gorutini.

- Slanje vrednosti zatvorenom kanalu takođe proizvodi paniku u trenutnoj gorutini.

- Slanje vrednosti na ili primanje vrednosti sa nil kanala dovodi do toga da trenutna gorutina uđe i ostane u blokiranom stanju zauvek.

U nastavku će biti data dodatna objašnjenja za četiri slučaja prikazana gornjim indeksima (A, B, C i D).

Da biste bolje razumeli tipove i vrednosti kanala i olakšali neka objašnjenja, veoma je korisno pogledati sirove unutrašnje strukture unutrašnjih objekata kanala.

Možemo zamisliti svaki kanal koji se sastoji od tri reda (svi se mogu posmatrati kao FIFO redovi) interno:

- red čekanja za prijem gorutina (generalno FIFO). Red je povezana lista bez ograničenja veličine. Gorutine u ovom redu su sve u blokiranom stanju i čekaju da prime vrednosti sa tog kanala.

- red slanja gorutine (generalno FIFO). Red je takođe povezana lista bez ograničenja veličine. Gorutine u ovom redu su sve u blokiranom stanju i čekaju da pošalju vrednosti tom kanalu. Vrednost (ili adresa vrednosti, u zavisnosti od implementacije kompajlera) koju svaka gorutina pokušava da pošalje takođe se čuva u redu zajedno sa tom gorutinom.

- red bafera vrednosti (apsolutno FIFO). Ovo je kružni red. NJegova veličina je jednaka kapacitetu kanala. Tipovi vrednosti sačuvanih u ovom redu bafera su svi tipovi elemenata tog kanala. Ako trenutni broj vrednosti sačuvanih u redu bafera vrednosti kanala dostigne kapacitet kanala, kanal se poziva u statusu punog kapaciteta. Ako trenutno nema vrednosti sačuvanih u redu bafera vrednosti kanala, kanal se poziva u statusu praznog kapaciteta. Za kanal nultog kapaciteta (nebaferovan), on je uvek i u statusu punog i u statusu praznog kapaciteta.

Svaki kanal interno sadrži mutex bravu koja se koristi da bi se izbegle trke podataka u svim vrstama operacija.

Slučaj A rada kanala : kada gorutina Rpokuša da primi vrednost iz kanala koji nije zatvoren i nije nula, gorutina Rće prvo steći zaključavanje povezano sa kanalom, a zatim će izvršiti sledeće korake dok se ne zadovolji jedan uslov.

- Ako red bafera vrednosti kanala nije prazan, u kom slučaju red prijemne gorutine kanala mora biti prazan, gorutina Rće primiti (pomeranjem) vrednost iz reda bafera vrednosti. Ako red slanja gorutine kanala takođe nije prazan, gorutina slanja će biti pomerena iz reda slanja gorutine i ponovo će biti vraćena u stanje izvršavanja. Vrednost koju upravo pomerena gorutina slanja pokušava da pošalje biće gurnuta u red bafera vrednosti kanala. Prijemna gorutina Rnastavlja da se izvršava. U ovom scenariju, operacija prijema kanala se naziva neblokirajuća operacija .

- U suprotnom slučaju (red bafera vrednosti kanala je prazan), ako red slanja gorutine kanala nije prazan, u kom slučaju kanal mora biti nebaferovani kanal, prijemna gorutina Rće pomeriti slanje gorutine iz reda slanja gorutine kanala i primiti vrednost koju upravo pomerena slanje gorutina pokušava da pošalje. Upravo pomerena slanje gorutina će biti deblokirana i ponovo će biti vraćena u stanje izvršavanja. Prijemna gorutina Rnastavlja sa izvršavanjem. U ovom scenariju, operacija prijema kanala se naziva operacija bez blokiranja .

- Ako su red za praćenje vrednosti i red za slanje gorutine kanala prazni, gorutina Rće biti gurnuta u red za prijem gorutine kanala i ući (i ostati u) stanju blokiranja. Može se nastaviti u stanje izvršavanja kada neka druga gorutina kasnije pošalje vrednost kanalu. U ovom scenariju, operacija prijema kanala se naziva operacija blokiranja .

Slučaj pravila kanala B : kada gorutina Spokuša da pošalje vrednost na kanal koji nije zatvoren i nije nula, gorutina Sće prvo steći zaključavanje povezano sa kanalom, a zatim će izvršiti sledeće korake dok se ne zadovolji uslov jednog koraka.

- Ako red prijemnih gorutina kanala nije prazan, u kom slučaju red bafera vrednosti kanala mora biti prazan, gorutina slanja Sće pomeriti prijemnu gorutinu iz reda prijemnih gorutina kanala i poslati vrednost u upravo pomerenu prijemnu gorutinu. Upravo pomerena prijemna gorutina će se deblokirati i ponovo preći u stanje izvršavanja. Gorutina slanja Snastavlja sa izvršavanjem. U ovom scenariju, operacija slanja kanala se naziva operacija bez blokiranja.

- U suprotnom slučaju (red gorutina za prijem je prazan), ako red bafera vrednosti kanala nije pun, u kom slučaju red gorutina za slanje takođe mora biti prazan, vrednost koju gorutina za slanje Spokušava da pošalje biće gurnuta u red bafera vrednosti, a gorutina za slanje Sće nastaviti da se izvršava. U ovom scenariju, operacija slanja kanala se naziva operacija bez blokiranja.

- Ako je red za prijem gorutine prazan, a red za skladištenje vrednosti kanala je već pun, gorutina slanja Sće biti gurnuta u red za slanje gorutine kanala i ući će (i ostati u) stanju blokiranja. Može se nastaviti u stanje izvršavanja kada neka druga gorutina kasnije primi vrednost iz kanala. U ovom scenariju, operacija slanja kanala se naziva operacija blokiranja.

Gore pomenuto, kada se zatvori kanal koji nije nula, slanje vrednosti kanalu će izazvati paniku tokom izvršavanja u trenutnoj gorutini. Imajte na umu da se slanje podataka zatvorenom kanalu smatra operacijom koja ne blokira .

Slučaj C rada kanala : kada gorutina pokuša da zatvori kanal koji nije zatvoren i nije nula, nakon što gorutina zaključa kanal, oba sledeća koraka će biti izvršena sledećim redosledom.

- Ako red prijemnih gorutina kanala nije prazan, u kom slučaju bafer vrednosti kanala mora biti prazan, sve gorutine u redu prijemnih gorutina kanala će biti pomerene jedna po jedna, svaka od njih će dobiti nultu vrednost tipa elementa kanala i biće vraćena u stanje izvršavanja.

- Ako red slanja gorutina kanala nije prazan, sve gorutine u redu slanja gorutina kanala će biti pomerene jedna po jedna i svaka od njih će proizvesti paniku zbog slanja na zatvorenom kanalu. To je razlog zašto bi trebalo da izbegavamo istovremene operacije slanja i zatvaranja na istom kanalu. Zapravo, kada je opcija detektora trke podataka ( ) komande go-race omogućena, slučajevi istovremenih operacija slanja i zatvaranja mogu biti otkriveni tokom izvršavanja i biće izbačena panika tokom izvršavanja.

**Napomena**: nakon što se kanal zatvori, vrednosti koje su već bile ubačene u bafer vrednosti kanala su i dalje tamo. Molimo vas da pažljivo pročitate sledeća objašnjenja za slučaj D za detalje.

Slučaj operacije kanala D : nakon što se zatvori kanal koji nije nula, operacije prijema kanala na kanalu se nikada neće blokirati. Vrednosti u baferu vrednosti kanala se i dalje mogu primati. Prateće druge opcione bulove povratne vrednosti su i dalje true. Kada se sve vrednosti u baferu vrednosti izvade i prime, beskonačne nulte vrednosti tipa elementa kanala biće primljene bilo kojom od sledećih operacija prijema na kanalu. Kao što je gore pomenuto, opcioni drugi povratni rezultat operacije prijema kanala je netipizovana bulova vrednost koja pokazuje da li je prvi rezultat (primljena vrednost) poslat pre nego što se kanal zatvori. Ako je drugi povratni rezultat false, onda prvi povratni rezultat (primljena vrednost) mora biti nulta vrednost tipa elementa kanala.

Poznavanje šta su blokirajuće i neblokirajuće operacije slanja ili primanja kanala važno je za razumevanje mehanizma selectblokova kontrolnog toka koji će biti predstavljen u kasnijem odeljku.

U gore navedenim objašnjenjima, ako je gorutina pomerena iz reda čekanja (bilo reda čekanja za slanje ili reda čekanja za prijem gorutina) kanala, i ako je gorutina blokirana zbog stavljanja u red čekanja u selectbloku koda toka kontrole , onda će gorutina biti vraćena u stanje izvršavanja u koraku 9 selectizvršavanja bloka koda toka kontrole . Može biti izbačena iz odgovarajućeg reda čekanja gorutina nekoliko kanala uključenih u selectblok koda toka kontrole.

Prema gore navedenim objašnjenjima, možemo dobiti neke činjenice o unutrašnjim redovima čekanja kanala.

- Ako je kanal zatvoren, i njegov red čekanja za slanje gorutine i red čekanja za prijem gorutine moraju biti prazni, ali njegov red čekanja za bafer vrednosti ne sme biti prazan.

- U bilo kom trenutku, ako bafer vrednosti nije prazan, onda njegov red čekanja za prijem gorutine mora biti prazan.

- U bilo kom trenutku, ako bafer vrednosti nije pun, onda njegov red za slanje gorutine mora biti prazan.

- Ako je kanal baferovan, onda u bilo kom trenutku, barem jedan od redova gorutina kanala mora biti prazan (slanje, prijem ili oba).

- Ako kanal nije baferovan, većinu vremena jedan od njegovih redova gorutina za slanje i red za prijem gorutina moraju biti prazni, sa jednim izuzetkom. Izuzetak je da gorutina može biti ubačena u oba reda prilikom izvršavanja selectbloka koda toka kontrole .

**Neki primeri korišćenja kanala**:

Sada kada ste pročitali gornji odeljak, hajde da pogledamo neke primere koji koriste kanale kako bismo vam bolje razumeli.

Jednostavan primer zahteva/odgovora. Dve gorutine u ovom primeru komuniciraju jedna sa drugom preko nebaferovanog kanala.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    c := make(chan int) // an unbuffered channel
    go func(ch chan<- int, x int) {
        time.Sleep(time.Second)
        // <-ch    // fails to compile
        // Send the value and block until the result is received.
        ch <- x*x // 9 is sent
    }(c, 3)
    done := make(chan struct{})
    go func(ch <-chan int) {
        // Block until 9 is received.
        n := <-ch
        fmt.Println(n) // 9
        // ch <- 123   // fails to compile
        time.Sleep(time.Second)
        done <- struct{}{}
    }(c)
    // Block here until a value is received by
    // the channel "done".
    <-done
    fmt.Println("bye")
}
```

Izlaz:

```sh
9
ćao
```

Demonstracija korišćenja baferovanog kanala. Ovaj program nije konkurentan, već samo pokazuje kako se koriste baferovani kanali.

```go
package main

import "fmt"

func main() {
    c := make(chan int, 2) // a buffered channel
    c <- 3
    c <- 5
    close(c)
    fmt.Println(len(c), cap(c)) // 2 2
    x, ok := <-c
    fmt.Println(x, ok) // 3 true
    fmt.Println(len(c), cap(c)) // 1 2
    x, ok = <-c
    fmt.Println(x, ok) // 5 true
    fmt.Println(len(c), cap(c)) // 0 2
    x, ok = <-c
    fmt.Println(x, ok) // 0 false
    x, ok = <-c
    fmt.Println(x, ok) // 0 false
    fmt.Println(len(c), cap(c)) // 0 2
    close(c) // panic!
    // The send will also panic if the above
    // close call is removed.
    c <- 7
}
```

Beskonačna fudbalska utakmica.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    var ball = make(chan string)
    kickBall := func(playerName string) {
        for {
            fmt.Println(<-ball, "kicked the ball.")
            time.Sleep(time.Second)
            ball <- playerName
        }
    }
    go kickBall("John")
    go kickBall("Alice")
    go kickBall("Bob")
    go kickBall("Emily")
    ball <- "referee" // kick off
    var c chan bool   // nil
    <-c               // blocking here for ever
}
```

Molimo vas da pročitate slučajeve upotrebe kanala za više primera upotrebe kanala.

## Vrednosti elemenata kanala se prenose kopiranjem

Kada se vrednost prenosi iz jedne gorutine u drugu gorutinu, vrednost će biti kopirana barem jednom. Ako je preneta vrednost ikada ostala u baferu vrednosti kanala, onda će se tokom procesa prenosa dogoditi dve kopije. Jedna kopija se dešava kada se vrednost kopira iz gorutine pošiljaoca u bafer vrednosti, a druga se dešava kada se vrednost kopira iz bafera vrednosti u gorutinu primaoca. Kao i kod dodeljivanja vrednosti i prosleđivanja argumenata funkcija, kada se vrednost prenosi, kopira se samo njen direktan deo .

Za standardni Go kompajler, veličina tipova elemenata kanala mora biti manja od 65536. Međutim, generalno, ne bi trebalo da kreiramo kanale sa tipovima elemenata velike veličine, kako bismo izbegli prevelike troškove kopiranja u procesu prenosa vrednosti između gorutina. Dakle, ako je veličina prosleđene vrednosti prevelika, najbolje je umesto toga koristiti pokazivački tip elementa, kako bismo izbegli velike troškove kopiranja vrednosti.

## O sakupljanju smeća kanala i Goroutine-a

Imajte na umu da se na kanal pozivaju sve gorutine u redu za slanje ili prijem gorutina kanala, tako da ako nijedan od redova kanala nije prazan, kanal ne može biti sakupljen kao otpad. S druge strane, ako je gorutina blokirana i ostaje u redu za slanje ili prijem gorutina kanala, onda se gorutina takođe ne može sakupiti kao otpad, čak i ako se na kanal poziva samo ova gorutina. U stvari, gorutina može biti sakupljena kao otpad samo kada je već izašla iz režima sakupljanja.

Operacije slanja i primanja kanala su jednostavne izjave

Operacije slanja i prijema kanala su jednostavne izjave . Operacija prijema kanala može se uvek koristiti kao izraz sa jednom vrednošću. Jednostavne izjave i izrazi mogu se koristiti u određenim delovima osnovnih blokova toka upravljanja .

Primer u kome se operacije slanja i primanja kanala pojavljuju kao jednostavne izjave u dva forbloka toka upravljanja.

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    fibonacci := func() chan uint64 {
        c := make(chan uint64)
        go func() {
            var x, y uint64 = 0, 1
            for ; y < (1 << 63); c <- y { // here
                x, y = y, x+y
            }
            close(c)
        }()
        return c
    }
    c := fibonacci()
    for x, ok := <-c; ok; x, ok = <-c { // here
        time.Sleep(time.Second)
        fmt.Println(x)
    }
}
```

## for-range na kanalima

Blok for-rangekoda toka upravljanja primenjuje se na kanale. Petlja će pokušati iterativno da primi vrednosti poslate kanalu, sve dok se kanal ne zatvori i njegov red za zadržavanje vrednosti ne postane prazan. Sa for-rangesintaksom na nizovima, kriškama i mapama, dozvoljene su višestruke iterativne promenljive. Međutim, za for-rangeblokove primenjene na kanale, možete koristiti najviše jednu iterativnu promenljivu, koja se koristi za čuvanje primljenih vrednosti.

```go
for v := range aChannel {
    // use v
}

je ekvivalentno

for {
    v, ok = <-aChannel
    if !ok {
        break
    }
    // use v
}
```

Sigurno je da ovde aChannelvrednost ne sme biti kanal samo za slanje. Ako je kanal nil, petlja će se tamo zauvek blokirati.

Na primer, drugi forblok petlje u primeru prikazanom u poslednjem odeljku može se pojednostaviti na

```go
    for x := range c {
        time.Sleep(time.Second)
        fmt.Println(x)
    }
```

## select-case blokovi koda toka kontrole

Postoji select-casesintaksa bloka koda koja je posebno dizajnirana za kanale. Sintaksa je veoma slična switch-casesintaksi bloka. Na primer, može biti više casegrana, a najviše jedna defaultgrana u select-casebloku koda. Ali postoje i neke očigledne razlike između njih.

- Nije dozvoljeno da se izrazi i iskazi nalaze iza selectključne reči (pre {).

- Nije fallthroughdozvoljeno korišćenje izjava u casegranama.

- Svaka naredba koja sledi caseključnu reč u select-casebloku koda mora biti ili naredba za prijem kanala ili naredba za slanje kanala. Operacija prijema kanala može se pojaviti kao izvorna vrednost jednostavne naredbe za dodelu. Kasnije caseće se operacija kanala koja sledi ključnu reč nazivati caseoperacijom.

- Ako postoji jedna ili više neblokirajućih caseoperacija, Go runtime će nasumično izabrati jednu od ovih neblokirajućih operacija za izvršavanje , a zatim će nastaviti sa izvršavanjem odgovarajuće casegrane.

- Ako su sve caseoperacije u select-casebloku koda blokirajuće operacije, defaultgrana će biti izabrana za izvršavanje ako defaultje grana prisutna. Ako grana defaultnije prisutna, trenutna gorutina će biti potisnuta u odgovarajući red gorutina za slanje ili red gorutina za prijem svakog kanala koji je uključen u sve caseoperacije, a zatim će ući u stanje blokiranja.

Po pravilima, select-caseblok koda bez ikakvih grana, select{}, učiniće da trenutna gorutina zauvek ostane u blokiranom stanju.

Sledeći program će defaultsigurno ući u granu.

```go
package main

import "fmt"

func main() {
    var c chan struct{} // nil
    select {
    case <-c:             // blocking operation
    case c <- struct{}{}: // blocking operation
    default:
        fmt.Println("Go here.")
    }
}
```

Primer koji pokazuje kako se koriste pokušaji slanja i pokušaji primanja:

```go
package main

import "fmt"

func main() {
    c := make(chan string, 2)
    trySend := func(v string) {
        select {
        case c <- v:
        default: // go here if c is full.
        }
    }
    tryReceive := func() string {
        select {
        case v := <-c: return v
        default: return "-" // go here if c is empty
        }
    }
    trySend("Hello!") // succeed to send
    trySend("Hi!")    // succeed to send
    // Fail to send, but will not block.
    trySend("Bye!")
    // The following two lines will
    // both succeed to receive.
    fmt.Println(tryReceive()) // Hello!
    fmt.Println(tryReceive()) // Hi!
    // The following line fails to receive.
    fmt.Println(tryReceive()) // -
}
```

Sledeći primer ima 50% mogućnost panike. Obe caseoperacije u ovom primeru nisu blokirajuće.

```go
package main

func main() {
    c := make(chan struct{})
    close(c)
    select {
    case c <- struct{}{}:
        // Panic if the first case is selected.
    case <-c:
    }
}
```

## Implementacija mehanizma odabira

Mehanizam selekcije u Gou je važna i jedinstvena karakteristika. Ovde su navedeni koraci implementacije mehanizma selekcije od strane zvaničnog Go izvršnog okruženja.

Postoji nekoliko koraka za izvršavanje select-casebloka:

- procenite sve uključene izraze kanala i izraze vrednosti koji se potencijalno šalju u caseoperacijama, odozgo nadole i sleva nadesno. Vrednosti odredišta za operacije prijema (kao izvorne vrednosti) u dodelama ne moraju se procenjivati u ovom trenutku.

- Nasumično rasporedite redosled grananja za anketiranje u koraku 5. Grana defaultse uvek stavlja na poslednju poziciju u redosledu rezultata. Kanali mogu biti duplirani u caseoperacijama.

- Sortiraj sve kanale uključene u caseoperacije kako bi se izbegla blokada (sa drugim gorutinama) u sledećem koraku. Nijedan duplirani kanal ne ostaje u prvim Nkanalima sortiranog rezultata, gde Nje broj kanala uključenih u caseoperacije. Ispod je redosled zaključavanja kanala koncept za prve Nkanale u sortiranom rezultatu.

- zaključati (tj. preuzeti zaključavanje) svih uključenih kanala pomoću redosleda zaključavanja kanala proizvedenog u poslednjem koraku.

- anketirajte svaku granu u bloku za odabir po randomizovanom redosledu proizvedenom u koraku 2 :

  - Ako je ovo casegrana i odgovarajuća operacija kanala je operacija slanja vrednosti u zatvoreni kanal, otključajte sve kanale inverznim redosledom zaključavanja kanala i učinite da trenutna gorutina panikuje. Idite na korak 12 .

  - Ako je ovo casegrana i odgovarajuća operacija kanala nije blokirajuća, izvršite operaciju kanala i otključajte sve kanale inverznim redosledom zaključavanja kanala, a zatim izvršite casetelo odgovarajuće grane. Operacija kanala može probuditi drugu gorutinu u blokiranom stanju. Idite na korak 12 .

  - Ako je ovo defaultgrana, onda otključajte sve kanale inverznim redosledom zaključavanja kanala i izvršite defaulttelo grane. Idite na korak 12 .

- (Do ove tačke, defaultgrana je odsutna i sve caseoperacije su blokirajuće operacije.)

- gurnuti (staviti u red) trenutnu gorutinu (zajedno sa informacijama odgovarajuće casegrane) u red za prijem ili slanje gorutina uključenog kanala u svakoj caseoperaciji. Trenutna gorutina može biti gurnuta u redove kanala više puta, jer uključeni kanali u više slučajeva mogu biti isti.

- Naterajte trenutnu gorutinu da uđe u stanje blokiranja i otključa sve kanale inverznim redosledom zaključavanja kanala.

- čekati u blokiranom stanju dok druge operacije kanala ne probude trenutnu gorutinu, ...

- Trenutna gorutina se budi drugom operacijom kanala u drugoj gorutini. Druga operacija može biti operacija zatvaranja kanala ili operacija slanja/primanja kanala. Ako je u pitanju operacija slanja/primanja kanala, mora postojati caseoperacija primanja/slanja kanala (u trenutno objašnjenom select-casebloku) koja sarađuje sa njom (prenošenjem vrednosti). U toj saradnji, trenutna gorutina će biti izbačena iz reda čekanja za prijem/slanje gorutina kanala.

- zaključajte sve uključene kanale prema nalogu za zaključavanje kanala.

- izbaciti trenutnu gorutinu iz reda čekanja gorutina za prijem ili reda čekanja gorutina za slanje uključenog kanala u svakoj caseoperaciji,

- Ako se trenutna gorutina probudi operacijom zatvaranja kanala, pređite na korak 5 .
  - Ako se trenutna gorutina probudi operacijom slanja/primanja kanala, odgovarajuća casegrana sarađujuće operacije primanja/slanja je već pronađena u procesu uklanjanja iz reda, pa samo otključajte sve kanale inverznim redosledom zaključavanja kanala i izvršite odgovarajuću casegranu.

- gotovo.

Iz implementacije znamo da

- Gorutina može istovremeno da ostane u redovima gorutina za slanje i redovima gorutina za prijem više kanala. Čak može istovremeno da ostane i u redu gorutina za slanje i u redu gorutina za prijem istog kanala.

- Kada se gorutina koja je trenutno blokirana u select-casebloku koda kasnije nastavi, ona će biti uklonjena iz svih redova gorutina za slanje i redova redova gorutina za prijem svih kanala uključenih u operacije kanala nakon caseključnih reči u select-casebloku koda.

## Više o kanalima

Više slučajeva upotrebe kanala možemo pronaći u ovom članku.

Iako nam kanali mogu pomoći da lako napišemo ispravan konkurentni kod, kao i druge tehnike sinhronizacije podataka, kanali nas neće sprečiti da napišemo nepravilan konkurentni kod.

Kanal možda nije uvek najbolje rešenje za sve slučajeve upotrebe za sinhronizaciju podataka. Molimo vas da pročitate ovaj članak i ovaj članak za više tehnika sinhronizacije u Go-u.

[Funkcije][0207] | [Sadržaj][00] | [Metode][0209]

[0207]: 0207_Funkcije.md
[00]: 00_Sadrzaj.md
[0209]: 0209_Metode.md
