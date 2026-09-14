
# Konstante i promenljive

[Osnovni tipovi i osnovni vrednosni literali][0103] | [Sadržaj][00] | [Uobičajeni operatori][0105]

Ovaj članak će predstaviti deklaracije konstanti i promenljivih u jeziku Go. Takođe će biti predstavljen koncept netipizovanih vrednosti i eksplicitnih konverzija.

Literali predstavljeni u prethodnom članku nazivaju se neimenovane konstante (ili literalne konstante), osim **false** i **true**, koje su dve unapred deklarisane (ugrađene) imenovane konstante. Prilagođene deklaracije imenovanih konstanti biće predstavljene u nastavku ovog članka.

## Netipizovane i tipizovane vrednosti

U jeziku Go, neke vrednosti nisu tipizovane. **Netipizovana vrednost** znači da tip vrednosti još nije određen. Naprotiv, tip **tipizovane vrednosti** je određen.

Za većinu netipizovanih vrednosti, svaka od njih ima jedan podrazumevani tip. Predeklarisana vrednost **nil** je jedina netipizovana vrednost koja nema podrazumevani tip. Više o tome ćemo saznati kasnije u drugim Go 101 člancima.

Sve literalne konstante (neimenovane konstante) su netipizovane vrednosti. U stvari, u jeziku Go, većina netipizovanih vrednosti su literalne konstante i imenovane konstante (koje će biti predstavljene u nastavku ovog članka). Ostale netipizovane vrednosti uključuju upravo pomenute **nil** i neke **bulove** rezultate koje vraćaju neke operacije, a koji će biti predstavljeni kasnije u drugim člancima.

Podrazumevani tip literalne konstante određen je njenim literalnim oblikom.

- Podrazumevani tip string literala je **string**.
- Podrazumevani tip bulovog literala je **bool**.
- Podrazumevani tip celobrojnog literala je **int**.
- Podrazumevani tip runskog literala je **rune** (takođe poznat kao, **int32**).
- Podrazumevani tip literala sa pokretnim zarezom je **float64**.
- Ako literal sadrži imaginarni deo, onda je njegov podrazumevani tip **complex128**.

## Eksplicitne konverzije netipizovanih konstanti

Kao i mnogi drugi jezici, Go takođe podržava konverzije vrednosti. Možemo koristiti oblik **T(v)** da konvertujemo vrednost **v** u tip označen sa **T**(ili jednostavno rečeno, tip T). Ako je konverzija **T(v)** legalna, Go kompajleri vide **T(v)** kao tipizovanu vrednost tipa **T**. Svakako, za određeni tip **T**, da bi konverzija bila **T(v)** legalna, vrednost **v** ne može biti proizvoljna.

Sledeća pomenuta pravila važe i za literalne konstante predstavljene u prethodnom članku i za netipizovane imenovane konstante koje će uskoro biti predstavljene.

Za netipizovanu konstantnu vrednost **v**, postoje dva scenarija gde **T(v)** je legalno.

- **v** (ili literal označen sa v) se može predstaviti kao vrednost osnovnog tipa **T**. Rezultatna vrednost je tipizirana konstanta tipa **T**.
- Podrazumevani tip **v** je ceo broj ( **int** ili **rune**) i **T** je string tipa. Rezultat **T(v)** je string tipa **T** i sadrži UTF-8 reprezentaciju celog broja kao Unikod kodnu tačku. Celobrojne vrednosti van opsega važećih Unikod kodnih tačaka rezultiraju stringovima predstavljenim sa "\uFFFD" (takođe poznatim kao, "\xef\xbf\xbd"). **0xFFFD** je kodna tačka za Unikod zamenski karakter. Rezultatni string konverzije iz celog broja uvek sadrži jednu i samo jednu runu. (Napomena, novije verzije Goa mogu dozvoliti samo konvertovanje runskih ili bajtnih celih brojeva u stringove. Od Go Toolčejna 1.15, **go vet** komanda upozorava na konverzije iz nerunskih i nebajtnih celih brojeva u stringove.)

U stvari, drugi scenario ne mora vbiti konstanta. Ako je **v** konstanta, onda je rezultat konverzije takođe konstanta; u suprotnom, rezultat nije konstanta.

Na primer, sledeće konverzije su sve legalne.

```go
// Rounding happens in the following 3 lines.
complex128(1 + -1e-1000i)  // 1.0+0.0i
float32(0.49999999)        // 0.5
float32(17000000000000000)

// No rounding in the these lines.
float32(123)
uint(1.0)
int8(-123)
int16(6+0i)
complex128(789)

string(65)          // "A"
string('A')         // "A"
string('\u68ee')    // "森"
string(-1)          // "\uFFFD"
string(0xFFFD)      // "\uFFFD"
string(0x2FFFFFFFF) // "\uFFFD"
```

I sledeće konverzije su sve ilegalne.

```go
// 1.23 is not representable as a value of int.
int(1.23)

// -1 is not representable as a value of uint8.
uint8(-1)

// 1+2i is not representable as a value of float64.
float64(1+2i)

// Constant -1e+1000 overflows float64.
float64(-1e1000)

// Constant 0x10000000000000000 overflows int.
int(0x10000000000000000)

// The default type of 65.0 is float64,
// which is not an integer type.
string(65.0)

// The default type of 66+0i is complex128,
// which is not an integer type.
string(66+0i)
```

Iz gornjih primera znamo da netipizovana konstanta, (na primer -1e1000i 0x10000000000000000), možda čak neće moći da se predstavi kao vrednost svog podrazumevanog tipa.

Imajte na umu da ponekad oblik eksplicitnih konverzija mora biti napisan kako bi **(T)(v)** se izbegle dvosmislenosti. Takve situacije se često dešavaju u slučaju da **T** nije identifikator.

Više eksplicitnih pravila konverzije ćemo saznati kasnije u drugim člancima o Go 101.

## Zaključivanje tipova u Go-u

Go podržava zaključivanje tipova. Drugim rečima, u mnogim okolnostima, programeri ne moraju eksplicitno da navode tipove nekih vrednosti u kodu. Go kompajleri će izvesti tipove za ove vrednosti na osnovu konteksta.

Dedukcija tipa se takođe često naziva **zaključivanje** tipa.

U Go kodu, ako je poziciji potrebna vrednost određenog tipa i netipizovana vrednost (često konstanta) se može predstaviti kao vrednost određenog tipa, onda se netipizovana vrednost može koristiti na toj poziciji. Go kompajleri će netipizovanu vrednost videti kao tipizovanu vrednost određenog tipa. Takva pozicije uključuju operand u operatorskoj operaciji, argument u pozivu funkcije, odredišnu vrednost ili izvornu vrednost u dodeli, itd.

U nekim okolnostima nema zahteva u pogledu tipova korišćenih vrednosti. Ako se u takvoj okolnosti koristi netipizovana vrednost, Go kompajleri će tretirati netipizovanu vrednost kao tipizovanu vrednost njenog podrazumevanog tipa.

Dva tipa slučaja odbitka mogu se posmatrati kao implicitne konverzije.

Dole navedeni odeljci o deklaracijama konstanti i promenljivih prikazaće neke slučajeve izvođenja tipa. Više pravila i slučajeva izvođenja tipa biće predstavljeno u drugim člancima.

## Konstantne deklaracije

Neimenovane konstante su sve **bulove**, **numeričke** i **string** vrednosti. Kao i neimenovane konstante, imenovane konstante takođe mogu biti samo **bulove**, **numeričke** i **string** vrednosti. Ključna reč **const** se koristi za deklarisanje imenovanih konstanti. Sledeći program sadrži neke deklaracije konstanti.

```go
package main

// Declare two individual constants. Yes,
// non-ASCII letters can be used in identifiers.
const π = 3.1416
const Pi = π // <=> const Pi = 3.1416

// Declare multiple constants in a group.
const (
    No         = !Yes
    Yes        = true
    MaxDegrees = 360
    Unit       = "radian"
)

func main() {
    // Declare multiple constants in one line.
    const TwoPi, HalfPi, Unit2 = π * 2, π * 0.5, "degree"
}
```

Go specifikacija poziva svaki od redova koji sadrže =simbol u gornjoj grupi deklaracija konstanti kao specifikaciju konstante.

U gornjem primeru, `*` simbol je operator množenja, a `!` simbol je bulov operator **not**. Operatori će biti predstavljeni u sledećem članku,

## Uobičajeni operatori

Simbol **=** znači **vezi** umesto **dodeli**. Trebalo bi da tumačimo svaku specifikaciju konstante kao da je deklarisani identifikator vezan za odgovarajući osnovni literal vrednosti. Molimo vas da pročitate poslednji odeljak u ovom članku za više objašnjenja.

U gornjem primeru, konstante sa imenom **π** i **Pi** su obe vezane za literal 3.1416. Dve imenovane konstante mogu se koristiti na mnogim mestima u kodu. Bez deklaracija konstanti, literal 3.1416 bi bio na tim mestima. Ako želimo da promenimo literal u 3.14 nešto kasnije, potrebno je izmeniti mnoga mesta. Uz pomoć deklaracija konstanti, literal 3.141 6će se pojaviti samo u jednoj deklaraciji konstante, tako da je potrebno izmeniti samo jedno mesto. To je glavna svrha deklaracija konstanti.

Kasnije ćemo koristiti termin **nekonstantne** vrednosti da bismo označili vrednosti koje nisu konstante. **Promenljive** koje će biti uvedene u nastavku, sve pripadaju jednoj vrsti **nekonstantnih** vrednosti.

Imajte u vidu da se konstante mogu deklarisati i na nivou paketa (van tela bilo koje funkcije) i u telima funkcija. Konstante deklarisane u telima funkcija nazivaju se **lokalne konstante**. Konstante deklarisane van tela bilo koje funkcije nazivaju se **konstante na nivou paketa**. Konstante na nivou paketa često nazivamo i **globalnim konstantama**.

Redosled deklaracije dve konstante na nivou paketa nije važan. U gornjem primeru, redosled deklaracije *No* i *Yes* mogu se zameniti.

Sve konstante deklarisane u poslednjem primeru su **netipizovane**. Podrazumevani tip imenovane netipizovane konstante je isti kao i literal koji je vezan za nju.

## Tipizovane imenovane konstante

Možemo deklarisati tipizirane konstante, sve tipizirane konstante su imenovane. U sledećem primeru, sve četiri deklarisane konstante su tipizirane vrednosti. Tipovi *X* i *Y* su su oba **float32**, a tipovi *A* i *B* su oba **int64**.

```go
const X float32 = 3.14

const (
    A, B int64   = -3, 5
    Y    float32 = 2.718
)
```

Ako je više tipiziranih konstanti deklarisano u istoj specifikaciji konstante, onda njihovi tipovi moraju biti isti, baš kao konstante *A* i *B* u gornjem primeru.

Takođe možemo koristiti eksplicitne konverzije da bismo pružili dovoljno informacija Go kompajlerima da zaključe tipove tipiziranih imenovanih konstanti. Gore navedeni isečak koda je ekvivalentan sledećem, u kome su. *X*, *Y*, *A* i *B* sve tipizirane konstante.

```go
const X = float32(3.14)

const (
    A, B = int64(-3), int64(5)
    Y    = float32(2.718)
)
```

Ako je osnovni vrednosni literal povezan sa tipizovanom konstantom, osnovni vrednosni literal mora biti predstavljiv kao vrednost tipa konstante. Sledeće deklaracije tipizovanih konstanti su nevažeće.

```go
// error: 256 overflows uint8
const a uint8 = 256

// error: 256 overflows uint8
const b = uint8(255) + uint8(1)

// error: 128 overflows int8
const c = int8(-128) / int8(-1)

// error: -1 overflows uint
const MaxUint_a = uint(^0)

// error: -1 overflows uint
const MaxUint_b uint = ^0
```

U gornjem i sledećim primerima **^** je **bitwise not** operator.

Sledeća deklaracija tipizirane konstante je važeća na 64-bitnim operativnim sistemima, ali nevažeća na 32-bitnim operativnim sistemima. Jer svaka **uint** vrednost ima samo 32 bita na 32-bitnim operativnim sistemima. **(1 << 64) - 1** nije predstavljiva kao 32-bitna vrednost. (Ovde **<<** je operator pomeranja ulevo na više bitova.)

```go
const MaxUint uint = (1 << 64) - 1
```

Kako onda deklarisati tipizovanu **uint** konstantu i povezati najveću **uint** vrednost sa njom? Umesto toga koristite sledeći način.

```go
const MaxUint = ^uint(0)
```

Slično, možemo deklarisati tipizovanu **int** konstantu i povezati najveću **int** vrednost sa njom. (Ovde **>>** je operator pomeranja udesno po bitu.)

```go
const MaxInt = int(^uint(0) >> 1)
```

Slična metoda se može koristiti za dobijanje broja bitova izvorne reči i proveru da li je trenutni OS 32-bitni ili 64-bitni.

```go
// NativeWordBits is 64 or 32.
const NativeWordBits = 32 << (^uint(0) >> 63)
const Is64bitOS = ^uint(0) >> 63 != 0
const Is32bitOS = ^uint(0) >> 32 == 0
```

Ovde su **!=** i **==** operatori nejednakosti i jednakosti.

## Automatsko dovršavanje u deklaracijama konstanti

U deklaraciji konstante grupnog stila, osim prve specifikacije konstante, ostale specifikacije konstanti mogu biti nepotpune. Nepotpuna specifikacija konstante sadrži samo listu identifikatora. Kompilatori će automatski dopuniti nepotpune redove umesto nas kopiranjem nedostajućeg dela iz prve prethodne potpune specifikacije konstante. Na primer, tokom kompajliranja, kompajleri će automatski dopuniti sledeći kod

```go
const (
    X float32 = 3.14
    Y           // here must be one identifier
    Z           // here must be one identifier
    A, B = "Go", "language"
    C, _
    // In the above line, the blank identifier
    // is required to be present.
)
```

kao

```go
const (
    X float32 = 3.14
    Y float32 = 3.14
    Z float32 = 3.14
    A, B = "Go", "language"
    C, _ = "Go", "language"
)
```

## iota u stalnim deklaracijama

Funkcija automatskog dovršavanja plus **iota** funkcija generatora konstanti donosi mnogo pogodnosti Go programiranju. **iota** je unapred deklarisana konstanta koja se može koristiti samo u drugim deklaracijama konstanti. Deklarisana je kao

```go
const iota = 0
```

Ali vrednost konstante **iota** u kodu ne mora uvek biti 0. Kada se unapred deklarisana **iota** konstanta koristi u deklaraciji prilagođene konstante, tokom kompajliranja, unutar deklaracije prilagođene konstante, njena vrednost će biti resetovana na vrednost 0 prve konstantne specifikacije svake grupe konstanti i povećavaće se za 1 specifikacija konstante za jednu specifikaciju konstante. Drugim rečima, u n-toj konstantnoj specifikaciji deklaracije konstante, vrednost **iota** je n (počevši od nule). Dakle **iota**, korisna je samo u deklaracijama konstanti u grupnom stilu.

Evo primera koji koristi **iota** funkcije automatskog dovršavanja i generatora konstanti. Molimo vas da pročitate komentare da biste razumeli šta će se desiti tokom kompajliranja. Simbol **+** u ovom primeru je operator sabiranja.

```go
package main

func main() {
    const (
        k = 3 // now, iota == 0

        m float32 = iota + .5  // m float32 = 1 +.5
        n                      // n float32 = 2 +.5

        p = 9             // now, iota == 3
        q = iota * 2      // q = 4 * 2
        _                 // _ = 5 * 2
        r                 // r = 6 * 2
        s, t = iota, iota // s, t = 7, 7
        u, v              // u, v = 8, 8
        _, w              // _, w = 9, 9
    )

    const x = iota // x = 0
    const (
        y = iota // y = 0
        z        // z = 1
    )

    println(m)             // +1.500000e+000
    println(n)             // +2.500000e+000
    println(q, r)          // 8 12
    println(s, t, u, v, w) // 7 7 8 8 9
    println(x, y, z)       // 0 0 1
}
```

Gore navedeni primer je samo da bi se demonstrirala pravila iotafunkcije generatora konstanti. Sigurno bi u praksi trebalo da je koristimo na smislenije načine. Na primer,

```go
const (
    Failed = iota - 1 // == -1
    Unknown           // == 0
    Succeeded         // == 1
)

const (
    Readable = 1 << iota // == 1
    Writable             // == 2
    Executable           // == 4
)
```

Ovde je **-** simbol **operator oduzimanja**, a **<<** simbol je **operator pomeranja ulevo**. Oba ova operatora biće predstavljena u sledećem članku.

## Promenljive, deklaracije promenljivih i dodeljivanje vrednosti

**Promenljive** su **imenovane vrednosti**. Promenljive se čuvaju u memoriji tokom izvršavanja. Vrednost koju predstavlja promenljiva može se izmeniti tokom izvršavanja.

Sve promenljive su **tipizirane vrednosti**. Prilikom deklarisanja promenljive, mora biti dato dovoljno informacija da bi kompajleri mogli da zaključe tip promenljive.

Promenljive deklarisane unutar tela funkcija nazivaju se **lokalne promenljive**. Promenljive deklarisane van bilo kog tela funkcije nazivaju se **promenljive nivoa paketa**. Promenljive nivoa paketa često nazivamo i **globalnim promenljivim**.

Postoje dva osnovna oblika deklaracije promenljivih, **standardni** i **kratki**. Kratki oblik se može koristiti samo za deklarisanje lokalnih promenljivih.

## Standardni obrasci za deklaraciju promenljivih

Svaka standardna deklaracija počinje ključnom **var** reči, nakon čega sledi deklarisani naziv promenljive. Nazivi promenljivih moraju biti identifikatori.

U nastavku su navedeni neki kompletni standardni obrasci deklaracije. U ovim deklaracijama su navedeni tipovi i početne vrednosti deklarisanih promenljivih.

```go
var lang, website string = "Go", "https://golang.org"
var compiled, dynamic bool = true, false
var announceYear int = 2009
```

Kao što smo otkrili, više promenljivih može biti deklarisano zajedno u jednoj deklaraciji promenljivih. Imajte na umu da u deklaraciji promenljive može biti naveden samo jedan tip. Dakle, tipovi više promenljivih deklarisanih u istoj deklaracijskoj liniji moraju biti identični.

Potpuni standardni obrasci deklaracije promenljivih se retko koriste u praksi, jer su opširni. U praksi se češće koriste dva varijantna oblika deklaracije standardnih promenljivih predstavljena u nastavku. U te dve varijante, odsutni su ili tipovi ili početne vrednosti deklarisanih promenljivih.

Slede neke standardne deklaracije promenljivih bez navođenja tipova promenljivih. Kompilatori će dedukovati tipove deklarisanih promenljivih kao tipove (ili podrazumevane tipove) njihovih odgovarajućih početnih vrednosti.

Sledeće deklaracije su zapravo ekvivalentne gore navedenim. Imajte u vidu da u sledećim deklaracijama tipovi više promenljivih deklarisanih u istoj deklaracijskoj liniji mogu biti različiti.

```go
// The types of the lang and dynamic variables
// will be deduced as built-in types "string"
// and "bool" by compilers, respectively.
var lang, dynamic = "Go", false

// The types of the compiled and announceYear
// variables will be deduced as built-in
// types "bool" and "int", respectively.
var compiled, announceYear = true, 2009

// The types of the website variable will be
// deduced as the built-in type "string".
var website = "https://golang.org"
```

Izvođenje tipova u gornjem primeru može se posmatrati kao implicitne konverzije.

Slede neke standardne deklaracije bez navođenja početnih vrednosti promenljivih. U ovim deklaracijama, sve deklarisane promenljive su inicijalizovane kao nulte vrednosti njihovih odgovarajućih tipova.

```go
// Both are initialized as blank strings.
var lang, website string

// Both are initialized as false.
var interpreted, dynamic bool

// n is initialized as 0.
var n int
```

Više promenljivih može biti grupisano u jednu standardnu deklaraciju oblika korišćenjem (). Na primer:

```go
var (
    lang, bornYear, compiled     = "Go", 2007, true
    announceAt, releaseAt    int = 2009, 2012
    createdBy, website       string
)
```

Gore navedeni primer je formatiran korišćenjem **go fmt** komande date u Go Toolchain-u. U gornjem primeru, svaka od tri linije je obuhvaćena, () što se naziva specifikacija promenljive.

Generalno, deklarisanje povezanih promenljivih zajedno će učiniti kod čitljivijim.

## Dodele čiste vrednosti

U gore navedenim deklaracijama promenljivih, znak **=** označava dodelu. Kada se promenljiva deklariše, možemo izmeniti njenu vrednost korišćenjem čistih dodela vrednosti. Kao i deklaracije promenljivih, više vrednosti može biti dodeljeno u čistoj dodeli.

Stavke izraza levo od **=** simbola u čistoj dodeli nazivaju se odredišne ili ciljne vrednosti. One moraju biti adresabilne vrednosti, izrazi indeksa mape ili prazan identifikator. Adrese vrednosti i mape biće predstavljene u kasnijim člancima.

Konstante su nepromenljive, tako da se konstanta ne može pojaviti na levoj strani čistog dodeljivanja kao odredišna vrednost, već samo na desnoj strani kao izvorna vrednost. Promenljive se mogu koristiti i kao izvorne i kao odredišne vrednosti, tako da se mogu pojaviti na obe strane čistog dodeljivanja vrednosti.

Prazni identifikatori se takođe mogu pojaviti na levoj strani dodela čistih vrednosti kao odredišne vrednosti, u kom slučaju to znači da ignorišemo odredišne vrednosti. Prazni identifikatori se ne mogu koristiti kao izvorne vrednosti u dodelama.

Primer:

```go
const N = 123
var x int
var y, z float32

N = 9 // error: constant N is not modifiable
y = N // ok: N is deduced as a float32 value
x = y // error: type mismatch
x = N // ok: N is deduced as an int value
y = x // error: type mismatch
z = y // ok
_ = y // ok

x, y = y, x // error: type mismatch
x, y = int(y), float32(x) // ok
z, y = y, z               // ok
_, y = y, z               // ok
z, _ = y, z               // ok
_, _ = y, z               // ok
x, y = 69, 1.23           // ok
```

Kod u poslednjem redu gornjeg primera koristi eksplicitne konverzije da bi se odgovarajuće vrednosti odredišta i izvora podudarale. Eksplicitna pravila konverzije za numeričke vrednosti koje nisu konstante predstavljena su u nastavku.

Go ne podržava lanac dodele. Na primer, sledeći kod je nedozvoljen.

var a, b int
a = b = 123 // syntax error

## Kratki obrasci deklaracije promenljivih

Takođe možemo koristiti kratke deklaracije promenljivih za deklarisanje promenljivih. Kratke deklaracije promenljivih mogu se koristiti samo za deklarisanje **lokalnih** promenljivih. Pogledajmo primer koji koristi neke kratke deklaracije promenljivih.

```go
package main

func main() {
    // Both lang and year are newly declared.
    lang, year := "Go language", 2007

    // Only createdBy is a new declared variable.
    // The year variable has already been
    // declared before, so here its value is just
    // modified, or we can say it is redeclared.
    year, createdBy := 2009, "Google Research"

    // This is a pure assignment.
    lang, year = "Go", 2012

    print(lang, " is created by ", createdBy)
    println(", and released at year", year)
}
```

Svaka kratka deklaracija promenljive mora deklarisati barem jednu novu promenljivu.

Postoji nekoliko razlika između kratkih i standardnih deklaracija promenljivih.

- U kratkom deklaracionom obliku, var ključne reči i tipovi promenljivih moraju biti izostavljeni.
- Znak dodele mora biti **:=** umesto **=**.
- U kratkoj deklaraciji promenljive, stare i nove promenljive mogu se mešati levo od :=. Ali mora postojati barem jedna nova promenljiva levo.

Imajte u vidu da, u poređenju sa čistim dodelama, postoji ograničenje za kratke deklaracije promenljivih. U kratkoj deklaraciji promenljive, sve stavke levo od **:=** znaka moraju biti čisti identifikatori. To znači da neke druge stavke kojima se može dodeliti vrednost, a koje će biti predstavljene u drugim člancima, ne mogu se pojaviti levo od **:=**. Ove stavke uključuju kvalifikovane identifikatore, elemente kontejnera, dereference pokazivača i selektore strukturnih polja. Čiste dodele nemaju takvo ograničenje.

## O terminologiji "dodele"

Kasnije, kada se pomene reč "dodela", ona može značiti

- čistu dodelu,
- kratku deklaraciju promenljive ili
- specifikaciju promenljive sa početnim vrednostima u standardnoj deklaraciji promenljive.
- U stvari, opštija definicija takođe uključuje prosleđivanje argumenata funkcije predstavljeno u nastavku članka.

Kažemo da je *x* dodelivo tipu *y* ako je *y = x* validan iskaz (kompajlira se bez problema). Pretpostavimo da je tip *y* *Ty*, ponekad, radi lakšeg opisa, možemo reći i da je *x* dodelivo tip u *Ty*.

Generalno, ako se *x* može dodeliti *y*, onda bi *y* trebalo da bude promenljivo, a tipovi *x* i *y* su identični ili *x* se može implicitno konvertovati u tip *y*. Sigurno, *y* može biti i prazan identifikator _.

## Svaka lokalno deklarisana promenljiva mora biti efektivno korišćena barem jednom

Imajte u vidu da standardni Go kompajler gc i gccgo ne dozvoljavaju lokalne promenljive koje se deklarišu, ali se ne koriste. Promenljive na nivou paketa nemaju takvo ograničenje.

Ako se lokalna promenljiva koristi samo kao odredišne vrednosti, ona će se takođe smatrati neiskorišćenom. Na primer, u sledećem programu, *r* se koristi samo kao odredište.

```go
package main

// Some package-level variables.
var x, y, z = 123, true, "foo"

func main() {
    var q, r = 789, false
    r, s := true, "bar"
    r = y // r is unused.
    x = q // q is used.
}
```

Kompilacija gornjeg programa će rezultirati sledećim greškama pri kompilaciji (pretpostavimo da je izvorna datoteka imena *example-unused.go*):

```go
./example-unused.go:6:6: r je deklarisan i nije korišćen
./example-unused.go:7:16: s je deklarisano i nije korišćeno
```

Rešenje je jednostavno, možemo dodeliti r praznim identifikatorima kako bismo izbegli greške pri kompajlaciji.

```go
package main

var x, y, z = 123, true, "foo"

func main() {
    var q, r = 789, false
    r, s := true, "bar"
    r = y
    x = q

    _, _ = r, s // make r and s used.
}
```

Generalno, gore navedena ispravka se ne preporučuje za upotrebu u produkcijskom kodu. Treba je koristiti samo u fazi razvoja/debagovanja. Nije dobra navika ostavljati neiskorišćene lokalne promenljive u kodu, jer neiskorišćene lokalne promenljive imaju negativne efekte i na čitljivost koda i na performanse izvršavanja programa.

## Zavisni odnosi promenljivih na nivou paketa utiču na njihov redosled inicijalizacije

Za sledeći primer,

var x, y = a+1, 5         // 8 5
var a, b, c = b+1, c+1, y // 7 6 5

Redosled inicijalizacije promenljivih na nivou paketa je: y = 5, c = y, b = c+1, a = b+1, i x = a+1.

Ovde je simbol **+** operator sabiranja, koji će biti predstavljen u sledećem članku.

Promenljive na nivou paketa ne mogu biti ciklično zavisne u svojoj deklaraciji. Sledeći kod se ne kompajlira.

var x, y = y, x

## Adresabilnost vrednosti

U programu Go, neke vrednosti su adresabilne (postoji adresa za njihovo pronalaženje). Sve promenljive su adresabilne, a sve konstante su neadresabilne. Više o adresama i pokazivačima možemo saznati iz članka pokazivači u programu Go. a kasnije i o drugim adresabilnim i neadresabilnim vrednostima iz drugih članaka.

## Eksplicitne konverzije na nekonstantnim numeričkim vrednostima

U jeziku Go, dve tipizirane vrednosti dva različita osnovna tipa ne mogu se dodeliti jedna drugoj. Drugim rečima, tipovi odredišne i izvorne vrednosti u dodeli moraju biti identični ako su obe vrednosti osnovne vrednosti. Ako tip izvorne osnovne vrednosti nije isti kao tip odredišne osnovne vrednosti, onda se izvorna vrednost mora eksplicitno konvertovati u tip odredišne vrednosti.

Kao što je gore pomenuto, nekonstantne celobrojne vrednosti mogu se konvertovati u stringove. Ovde predstavljamo još dva legalna slučaja konverzije vezana za nekonstantne numeričke vrednosti.

- Nekonstantne vrednosti sa pokretnim zarezom i celobrojne vrednosti mogu se eksplicitno konvertovati u bilo koje druge tipove sa pokretnim zarezom i celobrojne vrednosti.
- Nekonstantne kompleksne vrednosti mogu se eksplicitno konvertovati u bilo koje druge kompleksne tipove.

Za razliku od konverzija konstantnih brojeva, prekoračenja su dozvoljena kod konverzija nekonstantnih brojeva. A prilikom konvertovanja nekonstantne vrednosti sa pokretnim zarezom u ceo broj, zaokruživanje je takođe dozvoljeno. Ako nekonstantna vrednost sa pokretnim zarezom ne prekorači celobrojni tip, razlomljeni deo vrednosti sa pokretnim zarezom će biti odbačen (prema nuli) kada se konvertuje u celobrojni tip.

U svim konverzijama koje nisu konstante i koje uključuju vrednosti sa pokretnim zarezom ili kompleksne vrednosti, ako tip rezultata ne može da predstavi vrednost, onda konverzija uspeva, ali vrednost rezultata zavisi od implementacije.

U sledećem primeru, nameravane implicitne konverzije u redu 7 i redu 18 ne funkcionišu. Eksplicitne konverzije u redu 5 i redu 16 takođe nisu dozvoljene.

```go
const a = -1.23

// The type of b is deduced as float64.
var b = a                                               [4]

// error: constant 1.23 truncated to integer.
var x = int32(a)

// error: cannot assign float64 to int32.
var y int32 = b

// okay: z == -1, and the type of z is int32.
//       The fraction part of b is discarded.
var z = int32(b)

const k int16 = 255

// The type of n is deduced as int16.
var n = k

// error: constant 256 overflows uint8.
var f = uint8(k + 1)

// error: cannot assign int16 to uint8.
var g uint8 = n + 1

// okay: h == 0, and the type of h is uint8.
//       n+1 overflows uint8 and is truncated.
var h = uint8(n + 1)
```

Možemo pretpostaviti da je vrednost *a* u liniji 4 implicitno konvertovana u svoj podrazumevani tip ( **float64** ), tako da se tip b zaključuje kao **float64**. Više implicitnih pravila konverzije biće predstavljeno u drugim člancima kasnije.

## Opsezi promenljivih i imenovanih konstanti

U programu Go, možemo koristiti par `{` i `}` da bismo formirali blok koda. Blok koda može da ugnezdi druge blokove koda. Promenljiva ili imenovana konstanta deklarisana u unutrašnjem bloku koda će zakloniti promenljive i konstante deklarisane sa istim imenom u spoljašnjim blokovima koda. Na primer, sledeći program deklariše tri različite promenljive, sve se zovu *x*. Unutrašnja *x* promenljiva zakloniće spoljašnju.

```go
package main

const y = 789
var x int = 123

func main() {
    
    // The x variable shadows the above declared
    // package-level variable x.
    var x = true

    // A nested code block.
    {
        // Here, the left x and y are both
        // new declared variable. The right
        // ones are declared in outer blocks.
        x, y := x, y

        // In this code block, the just new
        // declared x and y shadow the outer
        // declared same-name identifiers.
        x, z := !x, y/10 // only z is new declared
        y /= 100
        println(x, y, z) // false 7 78
    }
    
    println(x)          // true
    println(z)          // error: z is undefined.
}
```

Opseg (opseg vidljivosti u kodu) promenljive na nivou paketa (ili imenovane konstante) je ceo paket u kome je promenljiva (ili imenovana konstanta) deklarisana. Opseg lokalne promenljive (ili imenovane konstante) počinje na kraju njene deklaracije i završava se na kraju njenog najdubljeg bloka koda. Zbog toga se poslednji red u funkciji *main* iz gornjeg primera ne kompajlira.

Blokovi koda i opsezi identifikatora biće detaljno objašnjeni kasnije u odeljku o blokovima i opsezima.

## Više o deklaracijama konstanti

Vrednost označena netipizovanom konstantom može preći svoj podrazumevani tip

Na primer, sledeći kod se kompajlira bez problema.

```go
// 3 untyped named constants. Their bound
// values all overflow their respective
// default types. This is allowed.
const n = 1 << 64          // overflows int
const r = 'a' + 0x7FFFFFFF // overflows rune
const x = 2e+308           // overflows float64

func main() {
    _ = n >> 2
    _ = r - 0x7FFFFFFF
    _ = x / 2
}
```

Ali sledeći kod se ne kompajlira, jer su sve konstante tipizirane.

```go
// 3 typed named constants. Their bound
// values are not allowed to overflow their
// respective default types. The 3 lines
// all fail to compile.
const n int = 1 << 64           // overflows int
const r rune = 'a' + 0x7FFFFFFF // overflows rune
const x float64 = 2e+308        // overflows float64
```

## Svaki imenovani konstantni identifikator biće zamenjen svojom vezanom literalnom vrednošću tokom kompajliranja

Deklaracije konstanti mogu se posmatrati kao poboljšani **#define** makroi u C-u. Deklaracija konstante definiše imenovanu konstantu koja predstavlja literal. Sva pojavljivanja imenovane konstante biće zamenjena literalom koji ona predstavlja tokom kompajliranja.

Ako su oba operanda operatora konstante, onda će operacija biti izvršena tokom kompajliranja. Molimo vas da pročitate sledeći članak o uobičajenim operatorima za detalje.

Na primer, tokom kompajliranja, sledeći kod

```go
package main

const X = 3
const Y = X + X
var a = X

func main() {
    b := Y
    println(a, b, X, Y)
}
```

biće posmatrano kao

```go
package main

var a = 3

func main() {
    b := 6
    println(a, b, 3, 6)
}
```

[Osnovni tipovi i osnovni vrednosni literali][0103] | [Sadržaj][00] | [Uobičajeni operatori][0105]

[0103]: 0103_Osnovni_tipovi_%20i_osnovni_vrednosni_literali.md
[00]: 00_Sadrzaj.md
[0105]: 0105_Uobičajeni_operatori.md
