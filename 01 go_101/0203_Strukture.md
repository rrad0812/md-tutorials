
# Strukture u Gou

[Pokazivači u Gou][0202] | [Sadržaj][00] | [Delovi vrednosti][0204]

Baš kao i C, Go takođe podržava strukturne tipove. Ovaj članak će predstaviti osnovna znanja o strukturnim tipovima i vrednostima u Go-u.

## Strukturni tipovi i literali strukturnih tipova

Svaki neimenovani literal tipa strukture počinje ključnom reči **struct** nakon koje sledi niz definicija polja zatvorenih u **{}**. Generalno, svaka definicija polja se sastoji od imena i tipa. Broj polja tipa strukture može biti nula.

Sledi neimenovani literal tipa strukture:

```go
struct {
    title  string
    author string
    pages  int
}
```

Gore navedeni tip strukture ima tri polja. Tipovi oba polja *title* i *author* su **string**. Tip polja *pages* je **int**.

Neki članci takođe nazivaju polja **promenljive članovi**.

Uzastopna polja istog tipa mogu se deklarisati zajedno.

```go
struct {
    title, author string
    pages         int
}
```

Veličina strukturnog tipa je zbir veličina svih njegovih tipova polja plus broj nekih bajtova za popunjavanje. Bajtovi za popunjavanje se koriste za poravnavanje memorijskih adresa nekih polja. O popunjavanju i poravnavanju memorijskih adresa možemo saznati u kasnijem članku.

Veličina strukture tipa sa nula polja je nula.

Oznaka može biti vezana za strukturno polje kada se polje deklariše. Oznake polja su opcione, podrazumevana vrednost svake oznake polja je prazan string. Sintaksa dozvoljava bilo koji oblik string literala za oznake polja. Međutim, u praksi, oznake strukturnih polja treba da se predstavljaju kao parovi ključ-vrednost, a svaka oznaka treba da se predstavlja kao sirovi string literali (...), dok svaka vrednost u oznaci treba da se predstavlja kao interpretirani string literali ( "..."). Na primer:

struct {
    Title  string `json:"title" myfmt:"s1"`
    Author string `json:"author,omitempty" myfmt:"s2"`
    Pages  int    `json:"pages,omitempty" myfmt:"n1"`
    X, Y   bool   `myfmt:"b1"`
}

Imajte na umu da su oznake polja X i Y u gornjem primeru identične (mada je korišćenje oznaka polja na ovaj način loša praksa).

Možemo koristiti refleksivni način da bismo pregledali informacije o oznakama polja.

Svrha svake oznake polja zavisi od aplikacije. U gornjem primeru, oznake polja mogu pomoći funkcijama u **encoding/json** standardnom paketu da odrede imena polja u JSON tekstovima, u procesu kodiranja strukturnih vrednosti u JSON tekstove ili dekodiranja JSON tekstova u strukturne vrednosti. Funkcije u **encoding/json** standardnom paketu će kodirati i dekodirati samo izvezena strukturna polja, zbog čega su prva slova imena polja u gornjem primeru napisana velikim slovima.

Nije dobra ideja koristiti oznake polja kao komentare.

Za razliku od C jezika, Go strukture ne podržavaju unije.

Svi gore prikazani tipovi struktura su neimenovani. U praksi, imenovani tipovi struktura su popularniji.

Samo izvezena polja strukturnih tipova prikazana u paketu mogu se koristiti u drugim paketima uvozom paketa. Neizvezena strukturna polja možemo videti kao privatne/zaštićene promenljive članove.

Oznake polja i redosled deklaracija polja u strukturnom tipu su važni za identitet strukturnog tipa. Dva neimenovana strukturna tipa su identična samo ako imaju isti niz deklaracija polja. Dve deklaracije polja su identične samo ako su njihova odgovarajuća imena, njihovi odgovarajući tipovi i njihove odgovarajuće oznake identični i ako su oba ugrađena polja ili ne. Imajte u vidu da se dva neeksportovana imena strukturnih polja iz različitih paketa uvek posmatraju kao dva različita imena.

Strukturni tip ne može imati polje samog strukturnog tipa, ni direktno ni rekurzivno.

## Literali strukturnih vrednosti i manipulacije strukturnim vrednostima

U jeziku Go, oblik **T{...}**, gde **T** mora biti literal tipa ili ime tipa, naziva se kompozitni literal i koristi se kao vrednosni literali nekih vrsta tipova, uključujući strukturne tipove i kontejnerske tipove predstavljene kasnije.

Imajte na umu da je literal tipa **T{...}** tipska vrednost, njen tip je **T**.

Dat je tip strukture *S* čiji je osnovni tip **struct{x int; y bool}**, nulta vrednost *S* može biti predstavljena sa sledeće dve varijante oblika kompozitnih literala strukture:

- *S{0, false}*  
  U ovoj varijanti, nema imena polja, ali sve vrednosti polja moraju biti prisutne prema redosledu deklaracije polja.
- S{x: 0, y: false},  
  S{y: false, x: 0},  
  S{x: 0},  
  S{y: false} i  
  S{}.  
  U ovoj varijanti, svaka stavka polja je opcionalna i redosled stavki polja nije važan. Vrednosti odsutnih polja biće postavljene kao nulte vrednosti njihovih odgovarajućih tipova. Ali ako je stavka polja prisutna, ona mora biti predstavljena sa formom **FieldName: FieldValue**. Redosled stavki polja u ovoj formi nije važan. Forma **S{}** je najčešće korišćena reprezentacija tipa *S* sa nultom vrednošću.

Ako je *S* tip strukture uvezen iz drugog paketa, preporučuje se korišćenje drugog oblika, radi održavanja kompatibilnosti. Razmotrimo slučaj gde održavalac paketa doda novo polje za tip S, što će učiniti upotrebu prvog oblika nevažećom.

Sigurno možemo koristiti i strukturne kompozitne literale da predstavimo vrednost strukture koja nije nula.

Za vrednost *v* tipa *S*, možemo koristiti *v.x* i *v.y*, koji se nazivaju **selektori** (ili **selektorski izrazi**), da bismo predstavili vrednosti polja tipa v. v se naziva **prijemnik selektora**.

Tačku u selektoru pozivamo kao operator za izbor svojstva.

Primer:

```go
package main

import (
    "fmt"
)

type Book struct {
    title, author string
    pages         int
}

func main() {
    book := Book{"Go 101", "Tapir", 256}
    fmt.Println(book) // {Go 101 Tapir 256}

    // Create a book value with another form.
    // All of the three fields are specified.
    book = Book{author: "Tapir", pages: 256, title: "Go 101"}

    // None of the fields are specified. The title and
    // author fields are both "", pages field is 0.
    book = Book{}

    // Only specify the author field. The title field
    // is "" and the pages field is 0.
    book = Book{author: "Tapir"}

    // Initialize a struct value by using selectors.
    var book2 Book // <=> book2 := Book{}
    book2.author = "Tapir Liu"
    book2.pages = 300
    fmt.Println(book2.pages) // 300
}
```

Poslednja stavka u složenom literalu je opcionalna ako se poslednja stavka u literalu i završna stavka } nalaze u istom redu. U suprotnom, poslednja,je obavezna. Za više detalja, pročitajte pravila za "prelom reda u Go-u".

var _ = Book {
    author: "Tapir",
    pages: 256,
    title: "Go 101", // here, the "," must be present
}

// The last "," in the following line is optional.
var _ = Book{author: "Tapir", pages: 256, title: "Go 101",}

## O dodeljivanju vrednosti struktura

Kada se vrednost strukture dodeli drugoj vrednosti strukture, efekat je isti kao kada se svako polje dodeli jedno po jedno.

```go
func f() {
    book1 := Book{pages: 300}
    book2 := Book{"Go 101", "Tapir", 256}

    book2 = book1
    // The above line is equivalent to the
    // following lines.
    book2.title = book1.title
    book2.author = book1.author
    book2.pages = book1.pages
}
```

Dve strukturne vrednosti mogu se dodeliti jedna drugoj samo ako su im tipovi identični ili tipovi dve strukturne vrednosti imaju identičan osnovni tip (uzimajući u obzir oznake polja) i ako je barem jedan od dva tipa neimenovani tip.

## Adresabilnost strukturnih polja

Polja adresabilne strukture su takođe adresabilna. Polja neadresabilne strukture su takođe neadresabilna. Polja neadresabilnih struktura se ne mogu menjati. Svi kompozitni literali, uključujući i strukturne kompozitne literale, su neadresabilni.

Primer:

```go
package main

import "fmt"

func main() {
    type Book struct {
        Pages int
    }
    var book = Book{} // book is addressable
    p := &book.Pages
    *p = 123
    fmt.Println(book) // {123}

    // The following two lines fail to compile, for
    // Book{} is unaddressable, so is Book{}.Pages.
    /*
    Book{}.Pages = 123
    p = &(Book{}.Pages) // <=> p = &Book{}.Pages
    */
}
```

Imajte na umu da je prioritet operatora za izbor svojstva **.** u selektoru viši od prioriteta operatora za preuzimanje adrese **&**.

Kompozitni literali se ne mogu adresirati, ali mogu prihvatati adrese.

Generalno, samo adresabilne vrednosti mogu da primaju adrese. Ali postoji sintaksički šećer u Go-u, koji nam omogućava da primamo adrese na kompozitnim literalima. Sintaksički šećer je izuzetak u sintaksi kako bi programiranje bilo praktičnije.

Na primer,

```go
package main

func main() {
    type Book struct {
        Pages int
    }
    // Book{100} is unaddressable but can
    // be taken address.
    p := &Book{100} // <=> tmp := Book{100}; p := &tmp
    p.Pages = 200
}
```

U selektorima polja, dereferenciranje prijemnika može biti implicitno.

U sledećem primeru, radi jednostavnosti, *(\*bookN).page* smože se napisati kao *bookN.pages*. Drugim rečima, *bookN* je dereferencirano u pojednostavljenim selektorima.

```go
package main

func main() {
    type Book struct {
        pages int
    }
    book1 := &Book{100} // book1 is a struct pointer
    book2 := new(Book)  // book2 is another struct pointer
    
    // Use struct pointers as structs.
    book2.pages = book1.pages
    
    // This last line is equivalent to the above line.
    // In other words, if the receiver is a pointer,
    // it will be implicitly dereferenced.
    (*book2).pages = (*book1).pages
}
```

## O poređenjima strukturnih vrednosti

Strukturni tip je uporediv samo ako nijedan od tipova njegovih polja (uključujući polja čija su imena prazan identifikator _) nije neuporediv.

Dve strukturne vrednosti su uporedive samo ako se mogu dodeliti jedna drugoj i ako su njihovi tipovi uporedivi. Drugim rečima, dve strukturne vrednosti mogu se uporediti jedna sa drugom samo ako su (uporedivi) tipovi dve strukturne vrednosti identični ili su njihovi osnovni tipovi identični (uzimajući u obzir oznake polja) i barem jedan od dva tipa je neimenovan.

Prilikom poređenja dve strukturne vrednosti istog tipa, svaki par njihovih odgovarajućih polja će biti upoređen (redosledom prikazanim u izvornom kodu). Dve strukturne vrednosti su jednake samo ako su sva njihova odgovarajuća polja jednaka. Poređenje se unapred zaustavlja kada se utvrdi da je par polja nejednak ili kada dođe do panike. Prilikom poređenja, polja sa imenima kao praznim identifikatorom _biće ignorisana.

## O konverzijama strukturnih vrednosti

Vrednosti dva strukturna tipa S1 i S2 mogu se konvertovati u međusobne tipove, ako S1 i S2 dele identičan osnovni tip (ignorišući oznake polja). Posebno ako je S1 ili S2 neimenovani tip i njihovi osnovni tipovi su identični (uzimajući u obzir oznake polja), onda konverzije između njihovih vrednosti mogu biti implicitne.

Dati su tipovi struktura S0, S1, S2, S3 i S4 u sledećem isečku koda,

- Vrednosti tipa S0 ne mogu se konvertovati u ostala četiri tipa i obrnuto, jer su odgovarajuća imena polja različita.

- Dve vrednosti dva različita tipa među S1, S2, S3 i S4 mogu se konvertovati u vrednosti jedna druge.

Posebno,

- Vrednosti tipa označenog sa S2 mogu se implicitno konvertovati u tip S3, i obrnuto.

- Vrednosti tipa označenog sa S2 mogu se implicitno konvertovati u tip S4, i obrnuto.

Ali,

- Vrednosti tipa označenog sa S2 moraju se eksplicitno konvertovati u tip S1, i obrnuto.

- Vrednosti tipa S3 moraju biti eksplicitno konvertovane u vrednosti tipa S4, i obrnuto.

```go
package main

type S0 struct {
    y int "foo"
    x bool
}

// S1 is an alias of an unnamed type.
type S1 = struct {
    x int "foo"
    y bool
}

// S2 is also an alias of an unnamed type.
type S2 = struct {
    x int "bar"
    y bool
}

// If field tags are ignored, the underlying
// types of S3(S4) and S1 are same. If field
// tags are considered, the underlying types
// of S3(S4) and S1 are different.
type S3 S2 // S3 is a defined (so named) type
type S4 S3 // S4 is a defined (so named) type

var v0, v1, v2, v3, v4 = S0{}, S1{}, S2{}, S3{}, S4{}

func f() {
    v1 = S1(v2); v2 = S2(v1)
    v1 = S1(v3); v3 = S3(v1)
    v1 = S1(v4); v4 = S4(v1)
    v2 = v3; v3 = v2 // the conversions can be implicit
    v2 = v4; v4 = v2 // the conversions can be implicit
    v3 = S3(v4); v4 = S4(v3)
}
```

U stvari, dve strukturne vrednosti mogu biti dodeljene (ili upoređene) jedna drugoj samo ako se jedna od njih može implicitno konvertovati u tip druge.

## Anonimni tipovi struktura mogu se koristiti u deklaracijama polja

Anonimni tipovi struktura mogu se koristiti kao tipovi polja drugog tipa strukture. Anonimni literali tipa strukture mogu se koristiti i u kompozitnim literalima.

Primer:

```go
var aBook = struct {
    // The type of the author field is
    // an anonymous struct type.
    author struct {
        firstName, lastName string
        gender              bool
    }
    title string
    pages int
}{
    author: struct { // an anonymous struct type
        firstName, lastName string
        gender              bool
    }{
        firstName: "Mark",
        lastName: "Twain",
    },
    title: "The Million Pound Note",
    pages: 96,
}
```

Generalno, radi bolje čitljivosti, ne preporučuje se korišćenje anonimnih literala tipa strukture u kompozitnim literalima.

## Više o tipovima struktura

Postoje neke napredne teme koje se odnose na strukturne tipove. One će biti objašnjene kasnije u odeljcima o ugrađivanju tipova i rasporedu memorije.

[Pokazivači u Gou][0202] | [Sadržaj][00] | [Delovi vrednosti][0204]

[0202]: 0202_Pokazivači.md
[00]: 00_Sadrzaj.md
[0204]: 0204_Delovi_vrednosti.md
