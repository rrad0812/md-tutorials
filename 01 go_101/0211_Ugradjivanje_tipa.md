
# Ugrađivanje tipa

[Interfejsi][0210] | [Sadržaj][00] | [Nebezbedni pokazivači][0212]

Iz članka o strukturama u Go-u , znamo da strukturni tip može imati mnogo polja. Svako polje se sastoji od jednog imena polja i jednog tipa polja. U stvari, ponekad, strukturno polje može biti sastavljeno samo od jednog tipa polja. Način deklarisanja strukturnih polja naziva se ugrađivanje tipa.

Ovaj članak će objasniti svrhu ugrađivanja tipova i sve vrste detalja o ugrađivanju tipova.

## Kako izgleda ugrađivanje tipova

Evo primera koji demonstrira ugrađivanje tipova:

```go
package main

import "net/http"

func main() {
    type P = *bool
    type M = map[int]int
    var x struct {
        string // a named non-pointer type
        error  // a named interface type
        *int   // an unnamed pointer type
        P      // an alias of an unnamed pointer type
        M      // an alias of an unnamed type

        http.Header // a named map type
    }
    x.string = "Go"
    x.error = nil
    x.int = new(int)
    x.P = new(bool)
    x.M = make(M)
    x.Header = http.Header{}
}
```

U gornjem primeru, šest tipova je ugrađeno u tip strukture. Svako ugrađivanje tipa formira ugrađeno polje.

Ugrađena polja se nazivaju i anonimnim poljima. Međutim, svako ugrađeno polje ima implicitno navedeno ime. Nekvalifikovano ime tipa ugrađenog polja služi kao ime polja. Na primer, imena šest ugrađenih polja u gornjim primerima su string, error, int, P, M, i Header, respektivno.

## Koji tipovi se mogu ugraditi

Trenutna Go specifikacija (verzija 1.25) kaže

Ugrađeno polje mora biti navedeno kao ime tipa Tili kao pokazivač na ime tipa koji nije interfejs *T, i Tsamo po sebi ne sme biti tip pokazivača.

Gore navedeni opis je bio tačan pre Go 1.9. Međutim, sa uvođenjem alijasa tipova u Go 1.9, opis je postao malo zastareo i netačan . Na primer, opis ne uključuje razlikovanje velikih i malih slova Ppolja u primeru u poslednjem odeljku.

Ovde članak pokušava da pruži preciznije opise.

- Ime tipa Tmože biti ugrađeno kao ugrađeno polje, osim ako Tne označava imenovani tip pokazivača ili tip pokazivača čiji je osnovni tip ili pokazivač ili tip interfejsa.

- Tip pokazivača *T, gde Tje ime tipa koje označava osnovni tip tipa pokazivača, može biti ugrađen kao ugrađeno polje, osim ako ime tipa Tne označava tip pokazivača ili interfejsa.

U nastavku su navedeni neki primeri tipova koji mogu i ne mogu biti ugrađeni:

```go
type Encoder interface {Encode([]byte) []byte}
type Person struct {name string; age int}
type Alias = struct {name string; age int}
type AliasPtr = *struct {name string; age int}
type IntPtr *int
type AliasPP = *IntPtr

// These types and aliases can be embedded.
Encoder
Person
*Person
Alias
*Alias
AliasPtr
int
*int

// These types and aliases can't be embedded.
AliasPP          // base type is a pointer type
*Encoder         // base type is an interface type
*AliasPtr        // base type is a pointer type
IntPtr           // named pointer type
*IntPtr          // base type is a pointer type
*chan int        // base type is an unnamed type
struct {age int} // unnamed non-pointer type
map[string]int   // unnamed non-pointer type
[]int64          // unnamed non-pointer type
func()           // unnamed non-pointer type
```

Nije dozvoljeno da dva polja imaju isto ime u strukturi, nema izuzetaka za anonimna polja struktura. Prema pravilima imenovanja ugrađenih polja, neimenovani tip pokazivača ne može biti ugrađen zajedno sa svojim osnovnim tipom u isti tip strukture. Na primer, inti *intne mogu biti ugrađeni u isti tip strukture.

Strukturni tip ne može rekurzivno ugraditi sebe ili svoje alijase.

Generalno, smisleno je ugrađivati samo tipove koji imaju polja ili metode (u narednim odeljcima će biti objašnjeno zašto), mada se mogu ugraditi i neki tipovi bez ikakvog polja i metode.

## Šta je smisleno ugrađivanje tipova

Glavna svrha ugrađivanja tipova je proširenje funkcionalnosti ugrađenih tipova u tip ugrađivanja, tako da ne moramo ponovo implementirati funkcionalnosti ugrađenih tipova za tip ugrađivanja.

Mnogi drugi objektno orijentisani programski jezici koriste nasleđivanje da bi postigli isti cilj ugrađivanja tipova. Oba mehanizma imaju svoje prednosti i mane . Ovde se u ovom članku neće razmatrati koji je bolji. Treba samo da znamo da je Go izabrao mehanizam ugrađivanja tipova i da postoji velika razlika između njih dvojice:

- Ako tip Tnasleđuje drugi tip, onda tip Tdobija sposobnosti drugog tipa. Istovremeno, svaka vrednost tipa Tmože se posmatrati i kao vrednost drugog tipa.

- Ako tip Tugrađuje drugi tip, tada tip `other type` postaje deo tipa `type` T, a tip `T dobija mogućnosti drugog tipa`, ali nijedna vrednost tipa `type` ne Tmože se posmatrati kao vrednost drugog tipa.

Evo primera koji pokazuje kako tip ugrađivanja proširuje funkcionalnosti ugrađenog tipa.

```go
package main

import "fmt"

type Person struct {
    Name string
    Age  int
}
func (p Person) PrintName() {
    fmt.Println("Name:", p.Name)
}
func (p *Person) SetAge(age int) {
    p.Age = age
}

type Singer struct {
    Person // extends Person by embedding it
    works  []string
}

func main() {
    var gaga = Singer{Person: Person{"Gaga", 30}}
    gaga.PrintName() // Name: Gaga
    gaga.Name = "Lady Gaga"
    (&gaga).SetAge(31)
    (&gaga).PrintName()   // Name: Lady Gaga
    fmt.Println(gaga.Age) // 31
}
```

Iz gornjeg primera, izgleda da, nakon ugrađivanja tipa Person, tip Singerdobija sve metode i polja tipa Person, a tip *Singerdobija sve metode tipa \*Person. Da li su zaključci tačni? Sledeći odeljci će odgovoriti na ovo pitanje.

Imajte na umu da Singervrednost nije Personvrednost, sledeći kod se ne kompajlira:

```go
var gaga = Singer{}
var _ Person = gaga
```

## Da li tip ugrađivanja dobija polja i metode ugrađivanih tipova

Hajde da navedemo sva polja i metode tipa Singeri metode tipa *Singerkorišćene u poslednjem primeru koristeći funkcionalnosti refleksije koje su date u reflectstandardnom paketu.

```go
package main

import (
    "fmt"
    "reflect"
)

... // the types declared in the last example

func main() {
    t := reflect.TypeOf(Singer{}) // the Singer type
    fmt.Println(t, "has", t.NumField(), "fields:")
    for i := 0; i < t.NumField(); i++ {
        fmt.Print(" field#", i, ": ", t.Field(i).Name, "\n")
    }
    fmt.Println(t, "has", t.NumMethod(), "methods:")
    for i := 0; i < t.NumMethod(); i++ {
        fmt.Print(" method#", i, ": ", t.Method(i).Name, "\n")
    }

    pt := reflect.TypeOf(&Singer{}) // the *Singer type
    fmt.Println(pt, "has", pt.NumMethod(), "methods:")
    for i := 0; i < pt.NumMethod(); i++ {
        fmt.Print(" method#", i, ": ", pt.Method(i).Name, "\n")
    }
}
```

Rezultat:

```sh
main.Singer ima 2 polja:
 polje br. 0: Osoba
 polje br. 1: radovi
main.Singer ima 1 metodu:
 metod br. 0: ŠtampajNaziv
*main.Singer ima 2 metode:
 metod br. 0: ŠtampajNaziv
 metod br. 1: PostaviStarost
```

Iz rezultata znamo da tip Singerzapravo poseduje PrintNamemetodu, i da tip *Singerzapravo poseduje dve metode, PrintNamei SetAge. Ali tip Singerne poseduje Namepolje. Zašto je onda selektorski izraz gaga.Namelegalan za Singervrednost gaga? Molimo vas da pročitate sledeći odeljak da biste razumeli razlog.

## Skraćenice za selektore

Iz članaka o strukturama u programu Go i metodama u programu Go , saznali smo da se, za vrednost x, x.ynaziva selektor, gde yje ili ime polja ili ime metode. Ako yje ime polja, onda xmora biti vrednost strukture ili vrednost pokazivača strukture. Selektor je izraz koji predstavlja vrednost. Ako selektor x.yoznačava polje, on takođe može imati svoja polja (ako x.yje vrednost strukture) i metode. Na primer x.y.z, gde ztakođe može biti ili ime polja ili ime metode.

U jeziku Go (bez razmatranja kolizija selektora i senčenja objašnjenih u kasnijem odeljku), ako srednje ime u selektoru odgovara ugrađenom polju, onda se to ime može izostaviti iz selektora . Zbog toga se ugrađena polja nazivaju i anonimnim poljima.

Na primer:

```go
package main

type A struct {
    FieldX int
}

func (a A) MethodA() {}

type B struct {
    *A
}

type C struct {
    B
}

func main() {
    var c = &C{B: B{A: &A{FieldX: 5}}}

    // The following 4 lines are equivalent.
    _ = c.B.A.FieldX
    _ = c.B.FieldX
    _ = c.A.FieldX // A is a promoted field of C
    _ = c.FieldX   // FieldX is a promoted field

    // The following 4 lines are equivalent.
    c.B.A.MethodA()
    c.B.MethodA()
    c.A.MethodA()
    c.MethodA() // MethodA is a promoted method of C
}
```

Zato gaga.Name je izraz legalan u primeru u poslednjem odeljku. Jer je to samo skraćenica od gaga.Person.Name.

Slično tome, selektor gaga.PrintNamese može posmatrati kao skraćenica od gaga.Person.PrintName. Ali, takođe je u redu ako mislimo da nije skraćenica. Na kraju krajeva, tip Singerzaista ima PrintNamemetodu, iako je metoda implicitno deklarisana (molimo vas da pročitate odeljak posle sledećeg za detalje). Iz istog razloga, selektor (&gaga).PrintNamei (&gaga).SetAgese takođe mogu posmatrati kao, ili ne kao, skraćenice od (&gaga.Person).PrintNamei (&gaga.Person).SetAge.

Namenaziva se unapređeno polje tipa Singer. PrintNamenaziva se unapređena metoda tipa Singer.

Imajte na umu da možemo koristiti i selektor gaga.SetAge, samo ako gagaje adresabilna vrednost tipa Singer. To je samo sintaksički šećer od (&gaga).SetAge. Molimo vas da pročitate pozive metoda za detalje.

U gornjim primerima, c.B.A.FieldXse naziva punim oblikom selektora c.FieldX, c.B.FieldXi c.A.FieldX. Slično tome, c.B.A.MethodAse naziva punim oblikom selektora c.MethodA, c.B.MethodAi c.A.MethodA.

Ako svako srednje ime u punom obliku selektora odgovara ugrađenom polju, onda se broj srednjih imena u selektoru naziva dubina selektora. Na primer, dubina selektora c.MethodAkorišćenog u gornjem primeru je 2 , za puni oblik selektora je c.B.A.MethodA.

Senkiranje i sudaranje selektora

Za vrednost x(uvek treba pretpostaviti da je adresabilna, čak i ako nije), moguće je da mnogi njeni selektori punog oblika imaju isti poslednji element yi svako srednje ime ovih selektora predstavlja ugrađeno polje. U takvim slučajevima,

- Samo selektor punog oblika sa najmanjom dubinom (pretpostavimo da je jedini) može se skratiti kao x.y. Drugim rečima, x.yoznačava selektor punog oblika sa najmanjom dubinom. Ostali selektori punog oblika su zasenjeni onim sa najmanjom dubinom.

- Ako postoji više od jednog selektora punog oblika sa najmanjom dubinom, onda se nijedan od tih selektora punog oblika ne može skratiti kao x.y. Kažemo da se ti selektori punog oblika sa najmanjom dubinom sudaraju jedni sa drugima.

Ako je selektor metode zasenjen drugim selektorom metode, a dva odgovarajuća potpisa metode su identična, kažemo da je prva metoda nadjačana drugom.

Na primer, pretpostavimo da su A, Bi Ctri definisana tipa .

```go
type A struct {
    x string
}
func (A) y(int) bool {
    return false
}

type B struct {
    y bool
}
func (B) x(string) {}

type C struct {
    B
}
```

Sledeći kod se ne kompajlira. Razlog je što su dubine selektora v1.A.xi v1.B.xjednake, tako da se dva selektora sudaraju jedan sa drugim i nijedan od njih ne može biti skraćen na v1.x. Ista situacija je i za selektore v1.A.yi v1.B.y.

```go
var v1 struct {
    A
    B
}

func f1() {
    _ = v1.x // error: ambiguous selector v1.x
    _ = v1.y // error: ambiguous selector v1.y
}
```

Sledeći kod se kompajlira bez problema. Selektor v2.C.B.xje zasenjen sa v2.A.x, tako da je selektor v2.xskraćeni oblik od v2.A.xactually. Iz istog razloga, selektor v2.yje skraćeni oblik od v2.A.y, a ne od v2.C.B.y.

```go
var v2 struct {
    A
    C
}

func f2() {
    fmt.Printf("%T \n", v2.x) // string
    fmt.Printf("%T \n", v2.y) // func(int) bool
}
```

Kolizioni ili zasenčeni selektori ne sprečavaju unapređenje njihovih dubljih selektora. Na primer, selektori `i` .Mi `i` .zse i dalje unapređuju u sledećem primeru.

```go
package main

type x string
func (x) M() {}

type y struct {
    z byte
}

type A struct {
    x
}
func (A) y(int) bool {
    return false
}

type B struct {
    y
}
func (B) x(string) {}

func main() {
    var v struct {
        A
        B
    }
    //_ = v.x // error: ambiguous selector v.x
    //_ = v.y // error: ambiguous selector v.y
    _ = v.M // ok. <=> v.A.x.M
    _ = v.z // ok. <=> v.B.y.z
}
```

Jedan detalj koji je neobičan, ali treba napomenuti, jeste da se dve neeksportovane metode (ili polja) iz dva različita paketa uvek posmatraju kao dva različita identifikatora, čak i ako su im imena identična. Dakle, nikada se neće sudarati ili zasenčiti kada su njihovi tipovi vlasnika ugrađeni u isti tip strukture. Na primer, program koji se sastoji od dva paketa, kao što je prikazano u nastavku, kompajliraće se i pokrenuti bez problema. Ali ako m()se sva pojavljivanja zamene sa M(), onda program neće uspeti da se kompajlira za A.Mi B.Msudaraće se jedan sa drugim, tako da c.Mnije validan selektor.

```go
package foo // import "x.y/foo"

import "fmt"

type A struct {
    n int
}

func (a A) m() {
    fmt.Println("A", a.n)
}

type I interface {
    m()
}

func Bar(i I) {
    i.m()
}

package main

import "fmt"
import "x.y/foo"

type B struct {
    n bool
}

func (b B) m() {
    fmt.Println("B", b.n)
}

type C struct{
    foo.A
    B
}

func main() {
    var c C
    c.m()      // B false
    foo.Bar(c) // A 0
}
```

## Implicitne metode za ugrađivanje tipova

Kao što je gore pomenuto, i tip Singeri tip *Singerimaju PrintNamemetodu, a i tip \*Singer ima SetAgemetodu. Međutim, nikada eksplicitno ne deklarišemo ove metode za ova dva tipa. Odakle dolaze ove metode?

U stvari, pretpostavimo da strukturni tip Sugrađuje tip (ili alias tipa) Ti da je ugrađivanje legalno,

- Za svaku metodu ugrađenog tipa T, ako selektori te metode ne kolidiraju niti su zasenjeni drugim selektorima, onda će kompajleri implicitno deklarisati odgovarajuću metodu sa istom specifikacijom za tip ugrađene strukture S. I shodno tome, kompajleri će takođe implicitno deklarisati odgovarajuću metodu za tip pokazivača *S.

- Za svaku metodu tipa pokazivača *T, ako selektori te metode nisu ni u koliziji sa drugim selektorima niti su im zasenčeni, onda će kompajleri implicitno deklarisati odgovarajuću metodu sa istom specifikacijom za tip pokazivača \*S.

Jednostavno govoreći,

- tip struct{Embedded}i tip *struct{Embedded}(ako su validni) dobijaju sve metode tipa označenog sa Embedded.

- tip *struct{Embedded}, tip struct{*Embedded}i tip *struct{*Embedded}(ako su validni) svi dobijaju sve metode tipa *Embedded.

Sledeće (promovisane) metode su implicitno deklarisane od strane kompajlera za tip Singeri tip *Singer.

```go
// Note: these declarations are not legal Go syntax.
// They are shown here just for explanation purpose.
// They indicate how implicit method values are
// evaluated (see the next section for more).
func (s Singer) PrintName = s.Person.PrintName
func (s *Singer) PrintName = (*s).Person.PrintName
func (s *Singer) SetAge = (&(*s).Person).SetAge
```

Desni delovi su odgovarajući selektori punog oblika.

Iz članka o metodama u Go jeziku , znamo da ne možemo eksplicitno deklarisati metode za neimenovane strukturne tipove i neimenovane pokazivačke tipove čiji su osnovni tipovi neimenovani strukturni tipovi. Ali, putem ugrađivanja tipova, takvi neimenovani tipovi takođe mogu posedovati metode.

Ako tip strukture ugrađuje tip koji implementira tip interfejsa (ugrađeni tip može biti sam tip interfejsa), onda generalno tip strukture takođe implementira tip interfejsa, izuzetak je metoda koju određuje tip interfejsa, a koja je zasenjena ili se kolizira sa drugim metodama ili poljima. Na primer, u gornjem primeru programa, i tip ugrađivane strukture i tip pokazivača čiji je osnovni tip tip ugrađivane strukture implementiraju tip interfejsa I.

Imajte u vidu da će tip direktno ili indirektno dobiti samo metode tipova koje ugrađuje. Drugim rečima, skup metoda tipa sastoji se od metoda deklarisanih direktno (eksplicitno ili implicitno) za taj tip i skupa metoda osnovnog tipa tipa. Na primer, u sledećem kodu,

- Tip Agenema metode, jer ne ugrađuje nijedan tip.
- Tip Xima dve metode, IsOdda Double. IsOddse dobija ugrađivanjem tipa MyInt.
- Tip Ynema metode, jer je ugrađeni tip Agenema metode.
- Tip Zima samo jednu metodu, IsOdd, koja se dobija ugrađivanjem tipa MyInt. Ne dobija metodu Doubleiz tipa X, jer ne ugrađuje tip X.

```go
type MyInt int
func (mi MyInt) IsOdd() bool {
    return mi%2 == 1
}

type Age MyInt

type X struct {
    MyInt
}
func (x X) Double() MyInt {
    return x.MyInt + x.MyInt
}

type Y struct {
    Age
}

type Z X
```

## Normalizacija i evaluacija vrednosti promovisanih metoda

Pretpostavimo v.mda je izraz vrednosti promovisane metode legalan, kompajleri će ga normalizovati kao rezultat promene implicitnog uzimanja adresa i operacija dereferenciranja pokazivača u eksplicitne u odgovarajućem selektoru punog oblika v.m.

Isto kao i kod bilo koje druge evaluacije vrednosti metode , za normalizovani izraz vrednosti metode v.m, tokom izvršavanja, kada v.mse vrednost metode evaluira, argument prijemnika vse evaluira i kopija rezultata evaluacije se čuva i koristi u kasnijim pozivima vrednosti metode.

Na primer, u sledećem kodu

- Selektor punog oblika izraza unapređene metode s.M1je s.T.X.M1. Nakon promene implicitnih operacija uzimanja adrese i dereferenciranja pokazivača u njemu, on postaje (*s.T).X.M1. Tokom izvršavanja, argument prijemnika (*s.T).Xse procenjuje i kopija rezultata procene se čuva i koristi u kasnijim pozivima vrednosti unapređene metode. Rezultat procene je 1, zato poziv f()uvek ispisuje 1.

- Selektor punog oblika izraza promovisane metode s.M2je s.T.X.M2. Nakon promene implicitnih operacija uzimanja adrese i dereferenciranja pokazivača u njemu, on postaje (&(*s.T).X).M2. Tokom izvršavanja, argument prijemnika &(*s.T).Xse procenjuje i kopija rezultata procene se čuva i koristi u kasnijim pozivima vrednosti promovisane metode. Rezultat procene je adresa polja s.X(tj. (*s.T).X). Svaka promena vrednosti s.Xće se odraziti kroz dereferenciranje adrese, ali promene vrednosti s.Tnemaju uticaja na rezultat procene, zato oba g()poziva ispisuju 2.

```go
package main

import "fmt"

type X int

func (x X) M1() {
    fmt.Println(x)
}

func (x *X) M2() {
    fmt.Println(*x)
}

type T struct { X }

type S struct { *T }

func main() {
    var t = &T{X: 1}
    var s = S{T: t}
    var f = s.M1 // <=> (*s.T).X.M1
    var g = s.M2 // <=> (&(*s.T).X).M2
    s.X = 2
    f() // 1
    g() // 2
    s.T = &T{X: 3}
    f() // 1
    g() // 2
}
```

## Tipovi interfejsa Ugrađuju sve vrste tipova

Tipovi interfejsa mogu da ugrađuju sve vrste tipova. Molimo vas da pročitate o interfejsima u Go-u za detalje.

**Zanimljiv primer ugrađivanja tipova**:

Na kraju, pogledajmo jedan zanimljiv primer. Primer programa će dovesti do mrtve petlje i prelivanja steka. Ako ste razumeli gore navedeni sadržaj i polimorfizam i ugrađivanje tipova, lako je razumeti zašto će doći do mrtve petlje.

[Interfejsi][0210] | [Sadržaj][00] | [Nebezbedni pokazivači][0212]

[0210]: 0210_Interfejsi.md
[00]: 0001_Uvod.md
[0212]: 0212_Nebezbedni_pokazivači.md
