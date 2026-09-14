
# Interfejsi

[Metode][0209] | [Sadržaj][00] | [Ugradjeni tipovi][0211]

Tipovi interfejsa su jedna posebna vrsta tipova u Gou. Interfejsi igraju nekoliko važnih uloga u Gou. U osnovi, tipovi interfejsa omogućavaju Gou da podrži „vrednosno boksovanje“. Shodno tome, kroz „vrednosno boksovanje“ se podržavaju refleksija i polimorfizam.

Od verzije 1.18, Go već podržava prilagođene generike. U prilagođenim genericima, tip interfejsa se (uvek) može koristiti i kao ograničenje tipa. U stvari, sva ograničenja tipa su zapravo tipovi interfejsa. Pre Go 1.18, svi tipovi interfejsa mogu se koristiti kao tipovi vrednosti. Ali od Go 1.18, neki tipovi interfejsa mogu se koristiti samo kao ograničenja tipa. Tipovi interfejsa koji se mogu koristiti kao tipovi vrednosti nazivaju se osnovni tipovi interfejsa.

Ovaj članak je uglavnom napisan pre nego što je Go počeo da podržava prilagođene generike, tako da se uglavnom bavi osnovnim interfejsima. O tipovima interfejsa samo sa ograničenjima, pročitajte knjigu Go generics 101 za detalje.

Tipovi interfejsa i skupovi tipova

Tip interfejsa definiše neke (tipske) zahteve. Svi tipovi koji nisu interfejsi i zadovoljavaju ove zahteve formiraju skup tipova, koji se naziva skup tipova tipa interfejsa.

Zahtevi definisani za tip interfejsa izražavaju se ugrađivanjem nekih elemenata interfejsa u tip interfejsa. Trenutno (verzija 1.25) postoje dve vrste elemenata interfejsa, elementi metode i elementi tipa.

- Element metode se predstavlja kao specifikacija metode. Specifikacija metode ugrađena u tip interfejsa ne sme da koristi prazan identifikator _kao svoje ime.

- Element tipa može biti ime tipa, literal tipa, aproksimacioni tip ili unija tipova. Trenutni članak ne govori mnogo o poslednja dva i govori samo o imenima tipova i literalima koji označavaju tipove interfejsa.

Na primer, unapred deklarisani errortip interfejsa , čija je definicija prikazana ispod, ugrađuje specifikaciju metode Error() string. U definiciji interface{...} se naziva literal tipa interfejsa , a reč interfaceje ključna reč u jeziku Go.

```go
type error interface {
        Error() string
}
```

Takođe možemo reći da errortip interfejsa (direktno) specificira metod Error() string. NJegov skup tipova se sastoji od svih tipova koji nisu interfejsi, a koji imaju metod sa specifikacijom Error() string. U teoriji, skup tipova je beskonačan. Sigurno je, za određeni Go projekat, konačan.

U nastavku su navedene neke druge definicije tipova interfejsa i deklaracije aliasa.

```go
// This interface directly specifies two methods and
// embeds two other interface types, one of which
// is a type name and the other is a type literal.
type ReadWriteCloser = interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    error                      // a type name
    interface{ Close() error } // a type literal
}

// This interface embeds an approximation type. Its type
// set includes all types whose underlying type is []byte.
type AnyByteSlice = interface {
    ~[]byte
}

// This interface embeds a type union. Its type set includes
// 6 types: uint, uint8, uint16, uint32, uint64 and uintptr.
type Unsigned interface {
    uint | uint8 | uint16 | uint32 | uint64 | uintptr
}
```

Ugrađivanje tipa interfejsa (označenog ili imenom tipa ili literalom tipa) u drugi je ekvivalentno (rekurzivnom) proširivanju elemenata iz prvog u drugi. Na primer, tip interfejsa označen alijasom tipa ReadWriteCloserje ekvivalentan tipu interfejsa označenom sledećim literalom, koji direktno određuje četiri metode.

```go
interface {
    Read(buf []byte) (n int, err error)
    Write(buf []byte) (n int, err error)
    Error() string
    Close() error
}
```

Skup tipova gore navedenog tipa interfejsa sastoji se od svih tipova koji nisu interfejsi, a koji imaju barem četiri metode navedene tipom interfejsa. Skup tipova je takođe beskonačan. On je definitivno podskup skupa tipova errortipa interfejsa.

Imajte na umu da, pre verzije Go 1.18, samo imena tipova interfejsa mogu biti ugrađena u tipove interfejsa.

Tipovi interfejsa prikazani u sledećem kodu nazivaju se prazni tipovi interfejsa, koji ne ugrađuju ništa i ne određuju nikakve metode.

```go
// The unnamed blank interface type.
interface{}
    
// Nothing is a defined blank interface type.
type Nothing interface{}
```

U stvari, Go 1.18 je uveo unapred deklarisani alias, any, koji označava prazan tip interfejsa interface{}.

Skup tipova praznog tipa interfejsa sastoji se od svih tipova koji nisu interfejsi.

## Skupovi metoda tipova

Svaki tip ima skup metoda koji je sa njim povezan.

- Za tip koji nije interfejs, njegov skup metoda se sastoji od specifikacija svih metoda (eksplicitnih ili implicitnih) deklarisanih za njega.

- Za tip interfejsa, njegov skup metoda se sastoji od svih specifikacija metoda koje on navodi, direktno ili indirektno kroz ugrađivanje drugih tipova.

U primerima prikazanim u poslednjem odeljku,

- Skup metoda tipa interfejsa označenog sa ReadWriteClosersadrži četiri metode.

- Skup metoda unapred deklarisanog tipa interfejsa errorsadrži samo jednu metodu.

- Skup metoda praznog tipa interfejsa je prazan.

Radi lakšeg snalaženja, skup metoda nekog tipa se često naziva i skup metoda bilo koje vrednosti tog tipa.

## Osnovni tipovi interfejsa

Osnovni tipovi interfejsa su tipovi interfejsa koji se mogu koristiti kao tipovi vrednosti. Neosnovni tip interfejsa se naziva i tip interfejsa samo sa ograničenjima.

Trenutno (Go 1.25), svaki osnovni tip interfejsa može biti u potpunosti definisan skupom metoda (može biti prazan). Drugim rečima, osnovni tip interfejsa ne zahteva definisanje elemenata tipa.

U primerima prikazanim u pretposlednjem odeljku, tip interfejsa označen sa alias ReadWriteCloserje osnovni tip, ali Unsignedtip interfejsa i tip označen sa alias AnyByteSlicenisu. Poslednja dva su tipovi interfejsa samo sa ograničenjima.

Prazni tipovi interfejsa i unapred deklarisani errortip interfejsa su takođe osnovni tipovi interfejsa.

Dva neimenovana osnovna tipa interfejsa su identična ako su im skupovi metoda identični. Imajte u vidu da se neeksportovane metode (čija imena su neeksportovani identifikatori) iz različitih paketa uvek posmatraju kao dve različite metode, čak i ako su im imena identična.

## Implementacije

Ako se neinterfejsni tip nalazi u skupu tipova interfejsnog tipa, onda kažemo da neinterfejsni tip implementira taj interfejsni tip. Ako je skup tipova interfejsnog tipa podskup drugog interfejsnog tipa, onda kažemo da prvi implementira drugi.

Tip interfejsa uvek implementira samog sebe, kao što je skup tipova uvek podskup (ili nadskup) samog sebe. Slično tome, dva tipa interfejsa sa istim skupom metoda implementiraju jedan drugog. U stvari, dva neimenovana tipa interfejsa su identična ako su njihovi skupovi tipova identični.

Ako tip Timplementira tip interfejsa X, tada skup metoda Tmora biti nadskup od X, bez obzira da li Tje tip interfejsa ili tip koji nije interfejs. Na primer, u primerima datim u prethodnom odeljku, tip interfejsa označen sa ReadWriteCloserimplementira errortip interfejsa.

Sve implementacije su implicitne u Go jeziku. Kompilator ne zahteva da se implementacioni odnosi eksplicitno navedu u kodu. implementsU Go jeziku ne postoji ključna reč. Go kompilatori će automatski proveravati implementacione odnose po potrebi.

Na primer, u sledećem primeru, skupovi metoda tipa struct pointer *Book, integer tipa MyInti pointer tipa \*MyInt sadrže specifikaciju metode About() string, tako da svi implementiraju gore pomenuti tip interfejsa Aboutable.

```go
type Aboutable interface {
    About() string
}

type Book struct {
    name string
    // more other fields...
}

func (book *Book) About() string {
    return "Book: " + book.name
}

type MyInt int

func (MyInt) About() string {
    return "I'm a custom integer value"
}
```

Implicitni dizajn implementacije omogućava da tipovi definisani u drugim bibliotečkim paketima, kao što su standardni paketi, pasivno implementiraju neke tipove interfejsa deklarisane u korisničkim paketima. Na primer, ako deklarišemo tip interfejsa kao sledeći, onda će i tip DBi tip Txdeklarisani u standardnom database/sqlpaketu automatski implementirati tip interfejsa, jer oba imaju tri odgovarajuće metode navedene u interfejsu.

```go
import "database/sql"

...

type DatabaseStorer interface {
    Exec(query string, args...interface{}) (sql.Result, error)
    Prepare(query string) (*sql.Stmt, error)
    Query(query string, args...interface{}) (*sql.Rows, error)
}
```

Treba napomenuti da, pošto je skup tipova praznog tipa interfejsa sastavljen od svih tipova koji nisu interfejsi, svi tipovi implementiraju bilo koji prazan tip interfejsa. Ovo je važna činjenica u jeziku Go.

## Vrednosni boks

Ponovo, trenutno (verzija 1.25), tipovi vrednosti interfejsa moraju biti osnovni tipovi interfejsa. U preostalom sadržaju trenutnog članka, kada se pominje tip vrednosti, tip vrednosti može biti tip koji nije interfejs ili osnovni tip interfejsa. Nikada nije tip interfejsa samo sa ograničenjima.

Svaku vrednost interfejsa možemo posmatrati kao kutiju za kapsuliranje vrednosti koja nije interfejs. Da bismo kutijali/kapsulirali vrednost koja nije interfejs u ​​vrednost interfejsa, tip vrednosti koja nije interfejs mora da implementira tip vrednosti interfejsa.

Ako tip Timplementira (osnovni) tip interfejsa I, onda se bilo koja vrednost tipa Tmože implicitno konvertovati u tip I. Drugim rečima, bilo koja vrednost tipa Tje dodeljiva (promenljivim) vrednostima tipa I. Kada Tse vrednost konvertuje (dodeli) u Ivrednost,

- Ako Tje tip tip koji nije interfejs, onda se kopija vrednosti Tpakuje (ili kapsulira) u rezultujuću (ili odredišnu) Ivrednost. Vremenska složenost kopije je O(n), gde nje veličina kopirane Tvrednosti. (Standardni Go kompajler pravi nekoliko optimizacija kako bi smanjio troškove pakovanja vrednosti koje zadovoljavaju određene uslove. Molimo vas da pročitate poglavlje „Interfejsi“ u knjizi Go Optimizations 101 za takve optimizacije.)

- Ako Tje tip takođe tip interfejsa, onda se kopija vrednosti uokvirene u vrednosti Tuokviruje (ili inkapsulira) u rezultatsku (ili odredišnu) Ivrednost. Standardni Go kompajler ovde pravi optimizaciju, tako da je vremenska složenost kopije O(1), umesto O(n).

Informacije o tipu vrednosti u kutiji se takođe čuvaju u rezultujućoj (ili odredišnoj) vrednosti interfejsa. (Ovo će biti detaljnije objašnjeno u nastavku.)

Kada je vrednost uokvirena u vrednosti interfejsa, ta vrednost se naziva dinamička vrednost vrednosti interfejsa. Tip dinamičke vrednosti se naziva dinamički tip vrednosti interfejsa.

Direktni deo dinamičke vrednosti vrednosti interfejsa je nepromenljiv, mada možemo zameniti dinamičku vrednost vrednosti interfejsa drugom dinamičkom vrednošću.

U programskom jeziku Go, nulte vrednosti bilo kog tipa interfejsa su predstavljene unapred deklarisanim identifikatorom nil. Ništa nije uokvireno u vrednost interfejsa nil. Dodeljivanje netipizovane nilvrednosti interfejsu će obrisati dinamičku vrednost uokvirenu u vrednosti interfejsa.

(Napomena: nulte vrednosti mnogih tipova koji nisu interfejsi u Go jeziku su takođe predstavljene sa nil. Vrednosti nil koje nisu interfejsi takođe mogu biti uokvirene u vrednostima interfejsa. Vrednost interfejsa koja uokviruje vrednost nil koja nije interfejs i dalje uokviruje nešto, tako da to nije vrednost interfejsa nil.)

Kako bilo koji tip implementira sve prazne tipove interfejsa, tako se svaka vrednost koja nije interfejs može uokviriti (ili dodeliti) praznoj vrednosti interfejsa. Iz tog razloga, prazni tipovi interfejsa mogu se posmatrati kao anytip u mnogim drugim jezicima.

Kada se praznoj vrednosti interfejsa dodeli netipizovana vrednost (osim netipizovanih nilvrednosti), netipizovana vrednost će prvo biti konvertovana u svoj podrazumevani tip. (Drugim rečima, možemo smatrati da se netipizovana vrednost izvodi kao vrednost svog podrazumevanog tipa).

Pogledajmo primer koji demonstrira neka dodeljivanja sa vrednostima interfejsa kao odredištima.

```go
package main

import "fmt"

type Aboutable interface {
    About() string
}

// Type *Book implements Aboutable.
type Book struct {
    name string
}
func (book *Book) About() string {
    return "Book: " + book.name
}

func main() {
    // A *Book value is boxed into an
    // interface value of type Aboutable.
    var a Aboutable = &Book{"Go 101"}
    fmt.Println(a) // &{Go 101}

    // i is a blank interface value.
    var i interface{} = &Book{"Rust 101"}
    fmt.Println(i) // &{Rust 101}

    // Aboutable implements interface{}.
    i = a
    fmt.Println(i) // &{Go 101}
}
```

Imajte na umu da je prototip funkcije fmt.Printlnkoja je mnogo puta korišćena u prethodnim člancima

```go
func Println(a...interface{}) (n int, err error)
```

Zbog toga fmt.Printlnpozivi funkcija mogu primati argumente bilo kog tipa.

Sledi još jedan primer koji pokazuje kako se prazna vrednost interfejsa koristi za uokvirivanje vrednosti bilo kog tipa koji nije interfejs.

```go
package main

import "fmt"

func main() {
    var i interface{}
    i = []int{1, 2, 3}
    fmt.Println(i) // [1 2 3]
    i = map[string]int{"Go": 2012}
    fmt.Println(i) // map[Go:2012]
    i = true
    fmt.Println(i) // true
    i = 1
    fmt.Println(i) // 1
    i = "abc"
    fmt.Println(i) // abc

    // Clear the boxed value in interface value i.
    i = nil
    fmt.Println(i) // <nil>
}
```

Go kompajleri će napraviti globalnu tabelu koja sadrži informacije o svakom tipu tokom kompajliranja. Informacije uključuju kakva je vrsta tipa, koje metode i polja tip poseduje, koji je tip elementa kontejnerskog tipa, veličine tipova itd. Globalna tabela će biti učitana u memoriju kada se program pokrene.

Tokom izvršavanja, kada se vrednost koja nije interfejs upakuje u vrednost interfejsa, Go runtime (barem za standardni Go runtime) će analizirati i izgraditi informacije o implementaciji za par tipova dve vrednosti i sačuvati informacije o implementaciji u vrednosti interfejsa. Informacije o implementaciji za svaki par tipa koji nije interfejs i tipa interfejsa biće izgrađene samo jednom i keširane u globalnoj mapi radi razmatranja efikasnosti izvršavanja. Broj unosa globalne mape se nikada ne smanjuje. U stvari, vrednost interfejsa koja nije nula samo koristi interno polje pokazivača koje referencira keširani unos informacija o implementaciji.

Informacije o implementaciji za svaki par (tip interfejsa, dinamički tip) uključuju dve informacije:

- informacije dinamičkog tipa (tip koji nije interfejs)

- i tabelu metoda (sekciju) koja čuva sve odgovarajuće metode određene tipom interfejsa i deklarisane za tip koji nije interfejs (dinamički tip).

Ove dve informacije su neophodne za implementaciju dve važne funkcije u Gou:

- Informacije o dinamičkom tipu su ključne za implementaciju refleksije u Gou.

- Informacije iz tabele metoda su ključne za implementaciju polimorfizma (polimorfizam će biti objašnjen u sledećem odeljku).

## Polimorfizam

Polimorfizam je jedna ključna funkcionalnost koju pružaju interfejsi i važna je karakteristika Go jezika.

tKada je vrednost tipa koja nije interfejs Tuokvirena u vrednost interfejsa itipa I, pozivanje metode određene tipom interfejsa Ina vrednosti interfejsa iće zapravo pozvati odgovarajuću metodu deklarisanu za tip koji nije interfejs Tna vrednosti koja nije interfejs t. Drugim rečima, pozivanje metode vrednosti interfejsa će zapravo pozvati odgovarajuću metodu dinamičke vrednosti vrednosti interfejsa. Na primer, pozivanje metode i.mće zapravo pozvati metodu t.m. Sa različitim dinamičkim vrednostima različitih dinamičkih tipova uokvirenim u vrednost interfejsa, vrednost interfejsa se ponaša drugačije. Ovo se naziva polimorfizam.

Kada se pozove metoda , pretražiće se i.mtabela metoda u informacijama o implementaciji koje su sačuvane u da bi se pronašla i pozvala odgovarajuća metoda. Tabela metoda je sloj, a pretraga je samo indeksiranje elemenata sloja, tako da je ovo brzo. it.m

(Napomena: pozivanje metoda na interfejs vrednosti nil će izazvati paniku tokom izvršavanja, jer nema dostupnih deklarisanih metoda za pozivanje.)

Primer:

```go
package main

import "fmt"

type Filter interface {
    About() string
    Process([]int) []int
}

// UniqueFilter is used to remove duplicate numbers.
type UniqueFilter struct{}
func (UniqueFilter) About() string {
    return "remove duplicate numbers"
}
func (UniqueFilter) Process(inputs []int) []int {
    outs := make([]int, 0, len(inputs))
    pusheds := make(map[int]bool)
    for _, n := range inputs {
        if !pusheds[n] {
            pusheds[n] = true
            outs = append(outs, n)
        }
    }
    return outs
}

// MultipleFilter is used to keep only
// the numbers which are multiples of
// the MultipleFilter as an int value.
type MultipleFilter int
func (mf MultipleFilter) About() string {
    return fmt.Sprintf("keep multiples of %v", mf)
}
func (mf MultipleFilter) Process(inputs []int) []int {
    var outs = make([]int, 0, len(inputs))
    for _, n := range inputs {
        if n % int(mf) == 0 {
            outs = append(outs, n)
        }
    }
    return outs
}

// With the help of polymorphism, only one
// "filterAndPrint" function is needed.
func filterAndPrint(fltr Filter, unfiltered []int) []int {
    // Calling the methods of "fltr" will call the
    // methods of the value boxed in "fltr" actually.
    filtered := fltr.Process(unfiltered)
    fmt.Println(fltr.About() + ":\n\t", filtered)
    return filtered
}

func main() {
    numbers := []int{12, 7, 21, 12, 12, 26, 25, 21, 30}
    fmt.Println("before filtering:\n\t", numbers)

    // Three non-interface values are boxed into
    // three Filter interface slice element values.
    filters := []Filter{
        UniqueFilter{},
        MultipleFilter(2),
        MultipleFilter(3),
    }

    // Each slice element will be assigned to the
    // local variable "fltr" (of interface type
    // Filter) one by one. The value boxed in each
    // element will also be copied into "fltr".
    for _, fltr := range filters {
        numbers = filterAndPrint(fltr, numbers)
    }
}
```

Izlaz:

```sh
pre filtriranja:
    [12 7 21 12 12 26 25 21 30]
uklonite duplirane brojeve:
    [12 7 21 26 25 30]
zadržite višekratnike broja 2:
    [12 26 30]
zadržite višekratnike broja 3:
    [12 30]
```

U gornjem primeru, polimorfizam čini nepotrebnim pisanje jedne filterAndPrintfunkcije za svaki tip filtera.

Pored gore navedene prednosti, polimorfizam takođe omogućava programerima paketa bibliotečkog koda da deklarišu izvezeni tip interfejsa i deklarišu funkciju (ili metod) koja ima parametar tipa interfejsa, tako da korisnik paketa može da deklariše tip, koji implementira tip interfejsa, u korisničkom kodu i da prosledi argumente tipa korisnika pozivima funkcije (ili metode). Programeri paketa koda ne moraju da brinu o tome kako je tip korisnika deklarisan, sve dok tip korisnika zadovoljava ponašanja koja su određena tipom interfejsa deklarisanim u paketu bibliotečkog koda.

U stvari, polimorfizam nije suštinska karakteristika jezika. Postoje alternativni načini da se postigne isti posao, kao što su funkcije povratnog poziva. Ali način sa polimorfizmom je čistiji i elegantniji.

## Odraz

Informacije o dinamičkom tipu sačuvane u vrednosti interfejsa mogu se koristiti za ispitivanje dinamičke vrednosti vrednosti interfejsa i manipulaciju vrednostima na koje se referencira dinamička vrednost. Ovo se u programiranju naziva refleksija.

Ovaj članak neće objasniti funkcionalnosti koje pruža standardni reflectpaket. Molimo vas da pročitate refleksije u programskom jeziku Go da biste saznali kako da koristite taj paket. U nastavku ćemo predstaviti samo ugrađene funkcionalnosti refleksije u programskom jeziku Go. U programskom jeziku Go, ugrađene refleksije se postižu pomoću tvrdnji tipa i type-switchblokova koda za kontrolu toka.

Tvrdnja tipa

U Go jeziku postoje četiri vrste slučajeva konverzije vrednosti koje uključuju vrednosti interfejsa:

- konvertovati vrednost koja nije interfejs u ​​vrednost interfejsa, gde tip vrednosti koja nije interfejs mora da implementira tip vrednosti interfejsa.
- konvertovati vrednost interfejsa u vrednost interfejsa, gde tip vrednosti izvornog interfejsa mora da implementira tip vrednosti odredišnog interfejsa.
- konvertovati vrednost interfejsa u vrednost koja nije interfejs, gde tip vrednosti koja nije interfejs mora da implementira tip vrednosti interfejsa.
- konvertovati vrednost interfejsa u vrednost interfejsa, gde tip vrednosti izvornog interfejsa ne implementira tip odredišnog interfejsa, ali dinamički tip vrednosti izvornog interfejsa može implementirati tip odredišnog interfejsa.

Već smo objasnili prve dve vrste slučajeva. Oba zahtevaju da tip izvorne vrednosti implementira tip odredišnog interfejsa. Konvertibilnost za prva dva se proverava tokom kompajliranja.

Ovde će biti objašnjene ove dve vrste slučajeva. Konvertibilnost za ove dve se proverava u vreme izvršavanja, korišćenjem sintakse koja se zove tvrdnja tipa. U stvari, sintaksa se primenjuje i na drugu vrstu konverzije na našoj gornjoj listi.

Oblik izraza za tvrdnju tipa je i.(T), gde ije vrednost interfejsa, a Tje ime tipa ili literal tipa. Tip Tmora biti

- ili proizvoljni tip koji nije interfejs,
- ili proizvoljni tip interfejsa.

U tvrdnji tipa i.(T), ise naziva tvrdnjena vrednost i Tnaziva se tvrdnjeni tip. Tvrdnja tipa može biti uspešna ili neuspešna.

- U slučaju Tda je tip ne-interfejs, ako dinamički tip ipostoji i identičan je sa T, onda će tvrdnja biti uspešna, u suprotnom, tvrdnja će neuspešno proći. Kada tvrdnja bude uspešna, rezultat evaluacije tvrdnje je kopija dinamičke vrednosti i. Tvrdnje ove vrste možemo posmatrati kao pokušaje raspakivanja vrednosti.

- U slučaju Tda je tip interfejsa, ako dinamički tip ipostoji i implementira T, onda će tvrdnja biti uspešna, u suprotnom, tvrdnja će neuspešno proći. Kada tvrdnja bude uspešna, kopija dinamičke vrednosti ibiće upakovana u Tvrednost, a Tvrednost će biti korišćena kao rezultat evaluacije tvrdnje.

Kada tvrdnja tipa ne uspe, rezultat njene evaluacije je nulta vrednost tvrđenog tipa.

Prema gore opisanim pravilima, ako je navedena vrednost u tvrdnji tipa nil vrednost interfejsa, onda će tvrdnja uvek neuspešna.

U većini scenarija, tvrdnja tipa se koristi kao izraz sa jednom vrednošću. Međutim, kada se tvrdnja tipa koristi kao jedini izraz izvorne vrednosti u dodeli, može rezultirati drugom opcionom netipizovanom bulovskom vrednošću i posmatrati se kao izraz sa više vrednosti. Druga opciona netipizovana bulovska vrednost pokazuje da li je tvrdnja tipa uspešna ili ne.

Imajte na umu da ako tvrdnja tipa ne uspe i tvrdnja tipa se koristi kao izraz sa jednom vrednošću (drugi opcioni bool rezultat je odsutan), onda će doći do panike.

Primer koji pokazuje kako se koriste tvrdnje tipa (tvrdnjeni tipovi su tipovi koji nisu interfejsi):

```go
package main

import "fmt"

func main() {
    // Compiler will deduce the type of 123 as int.
    var x interface{} = 123

    // Case 1:
    n, ok := x.(int)
    fmt.Println(n, ok) // 123 true
    n = x.(int)
    fmt.Println(n) // 123

    // Case 2:
    a, ok := x.(float64)
    fmt.Println(a, ok) // 0 false

    // Case 3:
    a = x.(float64) // will panic
}
```

Još jedan primer koji pokazuje kako se koriste tvrdnje o tipu (tvrdnjeni tipovi su tipovi interfejsa):

```go
package main

import "fmt"

type Writer interface {
    Write(buf []byte) (int, error)
}

type DummyWriter struct{}
func (DummyWriter) Write(buf []byte) (int, error) {
    return len(buf), nil
}

func main() {
    var x interface{} = DummyWriter{}
    var y interface{} = "abc"
    // Now the dynamic type of y is "string".
    var w Writer
    var ok bool

    // Type DummyWriter implements both
    // Writer and interface{}.
    w, ok = x.(Writer)
    fmt.Println(w, ok) // {} true
    x, ok = w.(interface{})
    fmt.Println(x, ok) // {} true

    // The dynamic type of y is "string",
    // which doesn't implement Writer.
    w, ok = y.(Writer)
    fmt.Println(w, ok) // <nil> false
    w = y.(Writer)     // will panic
}
```

U stvari, za vrednost interfejsa isa dinamičkim tipom T, poziv metode i.m(...)je ekvivalentan pozivu metode i.(T).m(...).

```go
type-switch blok kontrolnog toka

Sintaksa bloka koda type-switchje možda najčudnija sintaksa u Go jeziku. Može se posmatrati kao poboljšana verzija tvrdnje tipa. type-switchBlok koda je na neki način sličan switch-casebloku koda za kontrolu toka. Izgleda ovako:

switch aSimpleStatement; v := x.(type) {
case TypeA:
   ...
case TypeB, TypeC:
   ...
case nil:
   ...
default:
   ...
}
```

Deo aSimpleStatement;je opcionalan u type-switchbloku koda. aSimpleStatementmora biti jednostavna izjava. xmora biti vrednost interfejsa i naziva se tvrdnja (asserted value). vnaziva se rezultat tvrdnje (assert result), mora biti prisutan u obliku kratke deklaracije promenljive.

Svaka caseključna reč u type-switchbloku može biti praćena unapred deklarisanim nilidentifikatorom ili listom odvojenom zarezima sastavljenom od najmanje jednog imena tipa i literala tipa. Nijedna od takvih stavki ( nil, imena tipova i literali tipova) ne sme biti duplirana u istom type-switchbloku koda.

Ako tip označen imenom tipa ili literalom tipa koji sledi caseključnu reč u type-switchbloku koda nije tip interfejsa, onda mora da implementira tip interfejsa navedene vrednosti.

Evo primera u kome type-switchse koristi blok koda za kontrolu toka.

```go
package main

import "fmt"

func main() {
    values := []interface{}{
        456, "abc", true, 0.33, int32(789),
        []int{1, 2, 3}, map[int]bool{}, nil,
    }
    for _, x := range values {
        // Here, v is declared once, but it denotes
        // different variables in different branches.
        switch v := x.(type) {
        case []int: // a type literal
            // The type of v is "[]int" in this branch.
            fmt.Println("int slice:", v)
        case string: // one type name
            // The type of v is "string" in this branch.
            fmt.Println("string:", v)
        case int, float64, int32: // multiple type names
            // The type of v is "interface{}",
            // the same as x in this branch.
            fmt.Println("number:", v)
        case nil:
            // The type of v is "interface{}",
            // the same as x in this branch.
            fmt.Println(v)
        default:
            // The type of v is "interface{}",
            // the same as x in this branch.
            fmt.Println("others:", v)
        }
        // Note, each variable denoted by v in the
        // last three branches is a copy of x.
    }
}
```

Izlaz:

```sh
broj: 456
string: abc
drugi: tačno
broj: 0,33
broj: 789
ceo broj: [1 2 3]
ostali: mapa[]
<nula>
```

Gore navedeni primer je ekvivalentan sledećem u logici:

```go
package main

import "fmt"

func main() {
    values := []interface{}{
        456, "abc", true, 0.33, int32(789),
        []int{1, 2, 3}, map[int]bool{}, nil,
    }
    for _, x := range values {
        if v, ok := x.([]int); ok {
            fmt.Println("int slice:", v)
        } else if v, ok := x.(string); ok {
            fmt.Println("string:", v)
        } else if x == nil {
            v := x
            fmt.Println(v)
        } else {
            _, isInt := x.(int)
            _, isFloat64 := x.(float64)
            _, isInt32 := x.(int32)
            if isInt || isFloat64 || isInt32 {
                v := x
                fmt.Println("number:", v)
            } else {
                v := x
                fmt.Println("others:", v)
            }
        }
    }
}
```

type-switchBlokovi koda su slični switch-caseblokovima koda u nekim aspektima.

- Kao i switch-casekod blokova, u type-switchbloku koda može postojati najviše jedna defaultgrana.

- Kao i switch-casekod blokova, u type-switchbloku koda, ako defaultje grana prisutna, to može biti poslednja grana, prva grana ili srednja grana.

- Kao i switch-caseblokovi, type-switchblok koda možda ne sadrži grane, već će se smatrati zabranjenim.

Ali, za razliku od switch-caseblokova koda, fallthroughizjave se ne mogu koristiti unutar blokova grana type-switchbloka koda.

## Više o interfejsima u Go-u

Poređenja koja uključuju vrednosti interfejsa

Postoje dva slučaja poređenja koja uključuju vrednosti interfejsa:

- poređenja između vrednosti koja nije interfejs i vrednosti interfejsa.

- poređenja između dve vrednosti interfejsa.

U prvom slučaju, tip vrednosti koja nije interfejs mora da implementira tip (pretpostavimo da je I) vrednosti interfejsa, tako da se vrednost koja nije interfejs može konvertovati u (uokviriti u) vrednost interfejsa od I. To znači da se poređenje između vrednosti koja nije interfejs i vrednosti interfejsa može prevesti u poređenje između dve vrednosti interfejsa. Stoga će u nastavku biti objašnjena samo poređenja između dve vrednosti interfejsa.

Poređenje dve vrednosti interfejsa je zapravo poređenje njihovih odgovarajućih dinamičkih tipova i dinamičkih vrednosti.

Koraci poređenja dve vrednosti interfejsa (pomoću ==operatora):

- Ako je jedna od dve vrednosti interfejsa vrednost interfejsa nil, rezultat poređenja je da li je i druga vrednost interfejsa vrednost nil.
- Ako su dinamički tipovi dve vrednosti interfejsa dva različita tipa, onda je rezultat poređenja false.
- u slučaju kada su dinamički tipovi dve vrednosti interfejsa istog tipa,
  - Ako je isti dinamički tip neuporedivog tipa , doći će do panike.
  - u suprotnom, rezultat poređenja je rezultat poređenja dinamičkih vrednosti dve vrednosti interfejsa.

Ukratko, dve vrednosti interfejsa su jednake samo ako je ispunjen jedan od sledećih uslova.

- Obe su vrednosti interfejsa nil.

- NJihovi dinamički tipovi su identični i uporedivi, a njihove dinamičke vrednosti su jednake jedna drugoj.

Po pravilima, dve vrednosti interfejsa koje su obe dinamičke vrednosti nilmogu biti različite. Primer:

```go
package main

import "fmt"

func main() {
    var a, b, c interface{} = "abc", 123, "a"+"b"+"c"
    // A case of step 2.
    fmt.Println(a == b) // false
    // A case of step 3.
    fmt.Println(a == c) // true

    var x *int = nil
    var y *bool = nil
    var ix, iy interface{} = x, y
    var i interface{} = nil
    // A case of step 2.
    fmt.Println(ix == iy) // false
    // A case of step 1.
    fmt.Println(ix == i) // false
    // A case of step 1.
    fmt.Println(iy == i) // false

    // []int is an incomparable type
    var s []int = nil
    i = s
    // A case of step 1.
    fmt.Println(i == nil) // false
    // A case of step 3.
    fmt.Println(i == i) // will panic
}
```

## Unutrašnja struktura vrednosti interfejsa

Za zvanični Go kompajler/okruženje za izvršavanje, prazne vrednosti interfejsa i vrednosti interfejsa koje nisu prazne su predstavljene sa dve različite interne strukture. Molimo vas da pročitate delove vrednosti za detalje.

Vrednosti []Tne mogu se direktno konvertovati u []I, čak i ako type Timplementira interfejs type I.

Na primer, ponekad nam je potrebno da konvertujemo []stringvrednost u []interface{}tip. Za razliku od nekih drugih jezika, ne postoji direktan način da se izvrši konverzija. Konverziju moramo izvršiti ručno u petlji:

```go
package main

import "fmt"

func main() {
    words := []string{
        "Go", "is", "a", "high",
        "efficient", "language.",
    }

    // The prototype of fmt.Println function is
    // func Println(a...interface{}) (n int, err error).
    // So words... can't be passed to it as the argument.

    // fmt.Println(words...) // not compile

    // Convert the []string value to []interface{}.
    iw := make([]interface{}, 0, len(words))
    for _, w := range words {
        iw = append(iw, w)
    }
    fmt.Println(iw...) // compiles okay
}
```

## Svaka metoda navedena u tipu interfejsa odgovara implicitnoj funkciji

Za svaku metodu sa imenom mu skupu metoda definisanom tipom interfejsa I, kompajleri će implicitno deklarisati funkciju pod nazivom I.m, koja ima jedan ulazni parametar više, tipa I, od metode m. Dodatni parametar je prvi ulazni parametar funkcije I.m. Ako pretpostavimo ida je vrednost interfejsa I, onda je poziv metode i.m(...)ekvivalentan pozivu funkcije I.m(i,...).

Primer:

```go
package main

import "fmt"

type I interface {
    m(int)bool
}

type T string
func (t T) m(n int) bool {
    return len(t) > n
}

func main() {
    var i I = T("gopher")
    fmt.Println(i.m(5))                        // true
    fmt.Println(I.m(i, 5))                     // true
    fmt.Println(interface{m(int)bool}.m(i, 5)) // true

    // The following lines compile okay,
    // but will panic at run time.
    I(nil).m(5)
    I.m(nil, 5)
    interface {m(int) bool}.m(nil, 5)
}
```

[Metode][0209] | [Sadržaj][00] | [Ugradjeni tipovi][0211]

[0209]: 0209_Metode.md
[00]: 00_Sadrzaj.md
[0211]: 0211_Ugradjivanje_tipa.md
