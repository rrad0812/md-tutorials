
# Funkcije u programu Go

[Stringovi][0206] | [Sadržaj][00] | [Kanali][0208]

Deklaracije i pozivi funkcija su već objašnjeni. U ovom članku ćemo se više baviti konceptima i detaljima vezanim za funkcije u Go jeziku.

U stvari, funkcija je jedna vrsta prvoklasnih građanskih tipova u Gou. Drugim rečima, funkcije možemo koristiti kao vrednosti. Iako je Go statički jezik, Go funkcije su veoma fleksibilne. Osećaj korišćenja Go funkcija je sličan korišćenju kod mnogih dinamičkih jezika.

U jeziku Go postoje neke ugrađene funkcije. Ove funkcije su dokumentovane u **builtin** i **unsafe** standardnim paketima koda. Ugrađene funkcije imaju nekoliko razlika u odnosu na prilagođene funkcije. Ove razlike će biti pomenute u nastavku.

## Potpisi funkcija i tipovi funkcija

Literal funkcionalnog tipa sastoji se od **func** ključne reči i literala potpisa funkcije. Potpis funkcije sastoji se od dve liste tipova,

- jedna je lista tipova ulaznih parametara,
- druga je lista tipova izlaznih rezultata.

Imena parametara i rezultata mogu se pojaviti u literalima tipa funkcije i potpisa, ali imena nisu važna.

U praksi, **func** ključna reč može biti predstavljena u literalima potpisa, ili ne. Iz tog razloga, tip funkcije i potpis funkcije možemo smatrati istim konceptom.

Evo literala funkcionalnog tipa:

```go
func (a int, b string, c string) (x int, y int, z bool)
```

Iz članka o deklaracijama i pozivima funkcija, saznali smo da se uzastopni parametri i rezultati istog tipa mogu deklarisati zajedno. Dakle, gore navedeni literal je ekvivalentan sa:

```go
func (a int, b, c string) (x, y int, z bool)
```

Pošto imena parametara i imena rezultata nisu važna u literalima (sve dok nema duplih imena koja nisu prazna), gore navedena su ekvivalentna sledećem.

```go
func (x int, y, z string) (a, b int, c bool)
```

Imena promenljivih (parametara i rezultata) mogu biti prazni identifikatori _. Gore navedena su ekvivalentna sledećem.

```go
func (_ int, _, _ string) (_, _ int, _ bool)
```

Nazivi parametara moraju biti ili svi prisutni ili svi odsutni (anonimni). Isto pravilo važi i za nazive rezultata. Gore navedeni su ekvivalentni sledećim.

```go
func (int, string, string) (int, int, bool) // the standard form
func (a int, b string, c string) (int, int, bool)
func (x int, _ string, z string) (int, int, bool)
func (int, string, string) (x int, y int, z bool)
func (int, string, string) (a int, b int, _ bool)
```

Svi gore navedeni literali tipa funkcije označavaju isti (neimenovani) tip funkcije.

Svaka lista parametara mora biti zatvorena u **()** literalu, čak i ako je lista parametara prazna. Ako je lista rezultata tipa funkcije prazna, onda se može izostaviti iz literala tog tipa funkcije. Kada lista rezultata ima najviše jedan rezultat, onda lista rezultata ne mora biti zatvorena u literalu () ako literal liste rezultata ne sadrži imena rezultata.

```go
// The following three function types are identical.
func () (x int)
func () (int)
func () int

// The following two function types are identical.
func (a int, b string) ()
func (a int, b string)
```

## Varijadični parametri i tipovi varijadičnih funkcija

Poslednji parametar funkcije može biti varijadički parametar. Svaka funkcija može imati najviše jedan varijadički parametar. Tip varijadičkog parametra je uvek tip isečka. Da biste naznačili da je poslednji parametar varijadički, samo dodajte tri tačke ... ispred tipa elementa njegovog tipa (isečka) u njegovoj deklaraciji. Primer:

```go
func (values ...int64) (sum int64)
func (sep string, tokens ...string) string
```

Funkcionalni tip sa varijadičkim parametrom može se nazvati varijadičkim funkcijskim tipom. Varijadički funkcijski tip i nevarijadički funkcijski tip apsolutno nisu identični.

Neki primeri varijadnih funkcija biće prikazani u odeljku ispod.

## Tipovi funkcija su neuporedivi tipovi

U Go 101 je nekoliko puta pomenuto da su tipovi funkcija neuporedivi tipovi. Ali kao i vrednosti mape i isečka, vrednosti funkcija mogu se uporediti sa netipizovanim golim **nil** identifikatorom. (Vrednosti funkcija će biti objašnjene u poslednjem odeljku ovog članka.)

Pošto su tipovi funkcija neuporedivi tipovi, ne mogu se koristiti kao ključni tipovi tipova mapa.

## Prototipovi funkcija

Prototip funkcije se sastoji od imena funkcije i tipa funkcije (ili potpisa). NJen literal se sastoji od **func** ključne reči, imena funkcije i literala potpisa funkcije.

**Primer literala prototipa funkcije:**

```go
func Double(n int) (result int)
```

Drugim rečima, prototip funkcije je deklaracija funkcije bez dela tela. Deklaracija funkcije se sastoji od prototipa funkcije i tela funkcije.

### Deklaracije varijadičkih funkcija i pozivi varijadičkih funkcija

Opšte deklaracije i pozivi funkcija objašnjeni su u deklaracijama i pozivima funkcija. Ovde je predstavljeno kako se deklarišu i pozivaju varijabilne funkcije.

#### Deklaracije varijadičkih funkcija

Deklaracije varijadičkih funkcija su slične opštim deklaracijama funkcija. Razlika je u tome što poslednji parametar varijadičke funkcije mora biti varijadički parametar. Treba napomenuti da će varijadički parametar varijadičke funkcije biti tretiran kao isečak unutar tela varijadičke funkcije.

```go
// Sum and return the input numbers.
func Sum(values ...int64) (sum int64) {
    // The type of values is []int64.
    sum = 0
    for _, v := range values {
        sum += v
    }
    return
}

// An inefficient string concatenation function.
func Concat(sep string, tokens ...string) string {
    // The type of tokens is []string.
    r := ""
    for i, t := range tokens {
        if i != 0 {
            r += sep
        }
        r += t
    }
    return r
}
```

Iz gornje dve deklaracije varijadičnih funkcija, možemo videti da ako je varijadični parametar deklarisan sa tipom kao **...T**, onda je tip parametra **[]T** zapravo .

U stvari, funkcije **Print**, **Println** i **Printf** iz **fmt** standardnog paketa su sve varijadičke funkcije.

```go
func Print(a ...interface{}) (n int, err error)
func Printf(format string, a ...interface{}) (n int, err error)
func Println(a ...interface{}) (n int, err error)
```

Varijadički tipovi parametara ove tri funkcije su svi **[]interface{}**, čiji tip elementa **interface{}** tip interfejsa. Tipovi interfejsa i vrednosti biće objašnjeni kasnije u Go-u.

### Pozivi varijadičkih funkcija

Postoje dva načina za prosleđivanje argumenata varijadičkom parametru tipa **[]T**:

- prosledite vrednost isečka kao jedini argument. Isečak mora biti dodeljiv vrednostima tipa **[]T**, a iza isečka moraju biti sastavljene tri tačke **...**. Prosleđeni isečak se poziva kao varijadički argument.

- prosledite nula ili više argumenata koji se mogu dodeliti vrednostima tipa **T**. Ovi argumenti će biti kopirani (ili konvertovani) kao elementi nove dodeljene vrednosti isečka tipa **[]T**, a zatim će novi dodeljeni osečak biti prosleđen parametru variadic.

Imajte na umu da se ova dva načina ne mogu mešati u istom pozivu varijadičke funkcije.

Primer programa koji koristi neke pozive varijadičkih funkcija:

```go
package main

import "fmt"

func Sum(values ...int64) (sum int64) {
    sum = 0
    for _, v := range values {
        sum += v
    }
    return
}

func main() {
    a0 := Sum()
    a1 := Sum(2)
    a3 := Sum(2, 3, 5)

    // The above three lines are equivalent to
    // the following three respective lines.
    b0 := Sum([]int64{}...) // <=> Sum(nil...)
    b1 := Sum([]int64{2}...)
    b3 := Sum([]int64{2, 3, 5}...)
    fmt.Println(a0, a1, a3) // 0 2 10
    fmt.Println(b0, b1, b3) // 0 2 10
}
```

Još jedan primer:

```go
package main

import "fmt"

func Concat(sep string, tokens ...string) (r string) {
    for i, t := range tokens {
        if i != 0 {
            r += sep
        }
        r += t
    }
    return
}

func main() {
    tokens := []string{"Go", "C", "Rust"}
    // manner 1
    langsA := Concat(",", tokens...)
    // manner 2
    langsB := Concat(",", "Go", "C","Rust")
    fmt.Println(langsA == langsB) // true
}
```

Sledeći primer se ne kompajlira, jer su dva načina pozivanja varijabilnih funkcija pomešana.

```go
package main

// See above examples for the full declarations
// of the following two functions.
func Sum(values ...int64) (sum int64)
func Concat(sep string, tokens ...string) string

func main() {
    // The following two lines both fail
    // to compile, for the same error:
    // too many arguments in call.
    _ = Sum(2, []int64{3, 5}...)
    _ = Concat(",", "Go", []string{"C", "Rust"}...)
}
```

## Više o deklaracijama i pozivima funkcija

### Funkcije čija imena mogu biti duplirana

Generalno, imena funkcija deklarisanih u istom paketu koda ne mogu biti duplirana. Ali postoje dva izuzetka.

- Jedan izuzetak je što svaki paket koda može deklarisati nekoliko funkcija sa istim imenom **init** i istim tipom **func ()**.
- Drugi izuzetak je to što se više funkcija može deklarisati sa imenima kao praznim identifikatorom **_**, u kom slučaju se deklarisane funkcije nikada ne mogu pozvati.

### Neki pozivi funkcija se evaluiraju tokom kompajliranja

Većina poziva funkcija se izvršava u vreme izvršavanja. Ali pozivi funkcija standardnog **unsafe** paketa se uvek izvršavaju u vreme kompajliranja. Pozivi nekih drugih ugrađenih funkcija, kao što su **len** i **cap**, mogu se izvršavati ili u vreme kompajliranja ili u vreme izvršavanja, u zavisnosti od prosleđenih argumenata.

Rezultati poziva funkcija izvršenih u vreme kompajliranja mogu se dodeliti konstantama.

### Svi argumenti funkcije se prosleđuju kopijom

Ponovimo ponovo, kao i kod svih dodeljivanja vrednosti u programu Go, svi argumenti funkcije se prenose pomoću funkcije **copy** u programu Go. Kada se vrednost kopira, kopira se samo njen direktan deo (tj. plitka kopija).

### Deklaracije funkcija bez tela

Možemo implementirati funkciju u Go asemblerskom programu. Izvorni fajlovi Go asemblera se čuvaju u **\*.a** datotekama. Funkcija implementirana u Go asembleru i dalje mora biti deklarisana u datoteci *.go, ali je potrebno da bude prisutan samo prototip funkcije. Deo tela deklaracije funkcije mora biti izostavljen u datoteci **\*.go**.

### Neke funkcije sa rezultatima ne moraju da vraćaju

Ako funkcija ima rezultate vraćanja, onda poslednja naredba u telu njene deklaracije mora biti završna naredba. Osim **return** završne naredbe, postoje i neke druge vrste završnih naredbi. Dakle, telo funkcije ne mora da sadrži naredbu vraćanja. Na primer.

func fa() int {
    a:
    goto a
}

func fb() bool {
    for{}
}

### Rezultati poziva prilagođene funkcije mogu biti odbačeni, što ne važi za pozive nekih ugrađenih funkcija

Rezultati povratka poziva prilagođene funkcije mogu se svi odbaciti zajedno. Međutim, rezultati povratka poziva ugrađenih funkcija, osim **recover** i **copy**, ne mogu se odbaciti, iako se mogu ignorisati dodeljivanjem nekim praznim identifikatorima. Pozivi funkcija čiji se rezultati ne mogu odbaciti ne mogu se koristiti kao odloženi pozivi funkcija ili pozivi goroutine.

### Koristite pozive funkcija kao izraze

Poziv funkcije sa jednim povratnim rezultatom uvek se može koristiti kao jedna vrednost. Na primer, može se ugnjezditi u poziv druge funkcije kao argument, a može se koristiti i kao jedna vrednost koja se pojavljuje u bilo kojim drugim izrazima i izjavama.

Ako se vraćeni rezultati poziva funkcije sa više rezultata ne odbace, onda se poziv može koristiti kao izraz sa više vrednosti samo u dva scenarija.

- Poziv se može koristiti u dodeli kao izvorne vrednosti. Ali poziv se ne može mešati sa drugim izvornim vrednostima u dodeli.
- Poziv se može ugnezditi u drugi poziv funkcije kao argument. Ali poziv se ne može mešati sa drugim argumentima.

Primer:

```go
package main

func HalfAndNegative(n int) (int, int) {
    return n/2, -n
}

func AddSub(a, b int) (int, int) {
    return a+b, a-b
}

func Dummy(values ...int) {}

func main() {
    // These lines compile okay.
    AddSub(HalfAndNegative(6))
    AddSub(AddSub(AddSub(7, 5)))
    AddSub(AddSub(HalfAndNegative(6)))
    Dummy(HalfAndNegative(6))
    _, _ = AddSub(7, 5)

    // The following lines fail to compile.
    /*
    _, _, _ = 6, AddSub(7, 5)
    Dummy(AddSub(7, 5), 9)
    Dummy(AddSub(7, 5), HalfAndNegative(6))
    */
}
```

Imajte na umu da za standardni Go kompajler neke ugrađene funkcije krše univerzalnost gore opisanih pravila.

## Vrednosti funkcija

Kao što je gore pomenuto, funkcionalni tipovi su jedna vrsta tipova u jeziku Go. Vrednost funkcionalnog tipa naziva se vrednost funkcije. Nulte vrednosti funkcionalnih tipova su predstavljene unapred deklarisanim **nil** identifikatorom.

Kada deklarišemo prilagođenu funkciju, zapravo deklarišemo i nepromenljivu vrednost funkcije. Vrednost funkcije je identifikovana imenom funkcije. Tip vrednosti funkcije je predstavljen kao literal izostavljanjem imena funkcije iz literala prototipa funkcije.

Imajte na umu da se ugrađene funkcije ne mogu koristiti kao vrednosti. **init** funkcije se takođe ne mogu koristiti kao vrednosti.

Bilo koja vrednost funkcije može se pozvati baš kao i deklarisana funkcija. Fatalna greška je pozvati **nil** funkciju za pokretanje nove gorutine. Fatalna greška se ne može popraviti i dovešće do pada celog programa. U drugim situacijama, pozivi **nil** funkcija će proizvesti popravljive panike, uključujući odložene pozive funkcija.

Iz članka o delovima vrednosti, znamo da su vrednosti funkcija koje nisu **nil** višedelne vrednosti. Nakon što se jedna vrednost funkcije dodeli drugoj, dve funkcije dele iste osnovne delove. Drugim rečima, dve funkcije predstavljaju isti interni objekat funkcije. Efekti pozivanja dve funkcije su isti.

Primer:

```go
package main

import "fmt"

func Double(n int) int {
    return n + n
}

func Apply(n int, f func(int) int) int {
    return f(n) // the type of f is "func(int) int"
}

func main() {
    fmt.Printf("%T\n", Double) // func(int) int
    // Double = nil // error: Double is immutable.

    var f func(n int) int // default value is nil.
    f = Double
    g := Apply // let compile deduce the type of g
    fmt.Printf("%T\n", g) // func(int, func(int) int) int

    fmt.Println(f(9))         // 18
    fmt.Println(g(6, Double)) // 12
    fmt.Println(Apply(6, f))  // 12
}
```

U gornjem primeru, g(6, Double) i Apply(6, f) su ekvivalentni.

U praksi, često dodeljujemo anonimne funkcije funkcionalnim promenljivim, tako da možemo pozivati anonimne funkcije više puta.

```go
package main

import "fmt"

func main() {
    // This function returns a function (a closure).
    isMultipleOfX := func (x int) func(int) bool {
        return func(n int) bool {
            return n%x == 0
        }
    }

    var isMultipleOf3 = isMultipleOfX(3)
    var isMultipleOf5 = isMultipleOfX(5)
    fmt.Println(isMultipleOf3(6))  // true
    fmt.Println(isMultipleOf3(8))  // false
    fmt.Println(isMultipleOf5(10)) // true
    fmt.Println(isMultipleOf5(12)) // false

    isMultipleOf15 := func(n int) bool {
        return isMultipleOf3(n) && isMultipleOf5(n)
    }
    fmt.Println(isMultipleOf15(32)) // false
    fmt.Println(isMultipleOf15(60)) // true
}
```

Sve funkcije u jeziku Go mogu se posmatrati kao zatvaranja. To je jedan važan razlog zašto je Go, kao statički jezik, fleksibilan kao i mnogi dinamički skriptni jezici.

## Opseg nad funkcijama (od Go 1.23)

Od verzije Go 1.23, vrednosti tipova funkcija čiji su osnovni tipovi bilo koji od sledećih mogu se menjati pomoću **for-range** petlji.

```go
// K and V are some types.

func(yield func() bool)
func(yield func(V) bool)
func(yield func(K, V) bool)
```

Takve vrednosti se nazivaju **push iteratori**, a često i skraćeno iteratori.

Kada se koristi **for-range** petlja koja operiše na takvoj vrednosti funkcije iteratora, vrednost funkcije iteratora će biti pozvana (jednom) sa implicitno konstruisanom **yield** funkcijom povratnog poziva. **yield** funkcija povratnog poziva vraća **bool** rezultat. Kada vrati false, funkcija iteratora **yield** više ne bi trebalo da poziva funkciju.

Slede neki primeri korišćenja takvih iteratora.

```go
package main

import "fmt"

func Loop3(yield func() bool) {
    if (!yield()) {
        return
    }
    if (!yield()) {
        return
    }
    yield()
}

func OneDigitNumbers(onValue func(int) bool) {
    for i := range 10 {
        if (!onValue(i)) {
            return
        }
    }
}

func SquareLessThan50(onKeyValue func(int, int) bool) {
    for i := range 8 {
        if (!onKeyValue(i, i*i)) {
            return
        }
    }
}

func main() {
    var n = 0
    for range Loop3 {
        fmt.Print(n)
        n++
    }
    fmt.Println()
    // Output: 012

    for i := range OneDigitNumbers {
        fmt.Print(i)
    }
    fmt.Println()
    // Output: 0123456789
    
    for i, ii := range SquareLessThan50 {
        fmt.Printf("%v:%v ", i, ii)
    }
    fmt.Println()
    // Output: 0:0 1:1 2:4 3:9 4:16 5:25 6:36 7:49 
}
```

Za navedene slučajeve prikazane u gornjem primeru, **for-range** petlje su ekvivalentne sledećim pozivima funkcija, respektivno.

```go
func main() {
    var n = 0
    Loop3(func() bool {
        fmt.Print(n)
        n++
        return true
    })
    fmt.Println()

    OneDigitNumbers(func(i int) bool {
        fmt.Print(i)
        return true
    })
    fmt.Println()
    
    SquareLessThan50(func(i, ii int) bool {
        fmt.Printf("%v:%v ", i, ii)
        return true
    })
    fmt.Println()
}
```

Međutim, imajte na umu da u nekim drugim slučajevima takva ekvivalencija možda neće važiti.

Molimo vas da pročitate ovaj zvanični članak na blogu i ovaj članak za više informacija o iteratorima u Go jeziku.

[Stringovi][0206] | [Sadržaj][00] | [Kanali][0208]

[0206]: 0206_Stringovi.md
[00]: 00_Sadrzaj.md
[0208]: 0208_Kanali.md
