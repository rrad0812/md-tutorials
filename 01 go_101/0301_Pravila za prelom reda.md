
# Pravila za prelom reda

[Refleksije][0214] | [Sadržaj][00] | [Više o defer pozivima][0302]

Ako ste mnogo pisali Go kod, trebalo bi da znate da ne možemo koristiti proizvoljne stilove koda u Go programiranju. Konkretno, ne možemo prekinuti red koda na proizvoljnoj poziciji razmaka. U ostatku ovog članka biće navedena detaljna pravila za prelom reda u Go-u.

## Pravila umetanja tačke-zareza

Jedno pravilo kojeg se često pridržavamo u praksi jeste da ne bi trebalo da stavljamo početnu vitičastu zagradu ( **{** ) bilo kog eksplicitnog bloka koda u novi red. Na primer, sledeći forblok koda petlje se ne kompajlira.

```go
for i := 5; i > 0; i--
{ // unexpected newline, expecting { after for clause
}
```

Da bi se kompajliralo ispravno, početna vitičasta zagrada ne sme biti stavljena u novi red, kao što je prikazano u nastavku:

```go
for i := 5; i > 0; i-- {
}
```

Međutim, postoje neki izuzeci od gore pomenutog pravila. Na primer, sledeći for blok gole petlje se kompajlira bez problema.

```go
for
{
// do something...
}
```

Koja su onda osnovna pravila za prelom reda u Go programiranju? Pre nego što odgovorimo na ovo pitanje, trebalo bi da znamo činjenicu da formalna Go gramatika koristi tačka-zarez ( **;** ) kao završetak redova koda. Međutim, retko koristimo tačka-zarez u našem Go kodu. Zašto? Razlog je taj što je većina tačka-zareza opcionalna i može se izostaviti. Go kompajleri će automatski umetnuti izostavljene tačka-zareze prilikom kompajliranja.

Na primer, deset tačka-zareza u sledećem programu su opcione.

```go
package main;
import "fmt";

func main() {
    var (
        i   int;
        sum int;
    );
    for i < 6 {
        sum += i;
        i++;
    };
    fmt.Println(sum);
};
```

Pretpostavimo da je gornji program sačuvan u datoteci pod nazivom `semicolons.go`, možemo pokrenuti komandu `go fmt semicolons.go` da bismo uklonili sve nepotrebne tačke-zareze iz te datoteke. Kompilatori će automatski ubaciti uklonjene tačke-zareze nazad (u memoriju) prilikom kompajliranja izvornog koda.

Koja su pravila za umetanje tačka-zareza u Gou? Hajde da pročitamo pravila za tačka-zarez navedena u Go specifikaciji.

Formalna gramatika koristi tačka-zarez ";" kao terminatore u brojnim produkcijama. Go programi mogu izostaviti većinu ovih tačka-zareza koristeći sledeća dva pravila:

Kada se ulaz razbije na tokene, tačka-zarez se automatski ubacuje u tok tokena odmah nakon poslednjeg tokena linije ako je taj token

- identifikator​
- ceo broj, broj sa pokretnim zarezom, imaginarni broj, runski broj ili string literal
  jedna od ključnih reči break, continue, fallthrough, ili return
- jedan od operatora i interpunkcijskih znakova ++, --, ), ], ili}

Da bi složeni iskazi zauzeli jedan red, tačka-zarez se može izostaviti ispred završnog znaka ) ili }.

Za scenarije navedene u prvom pravilu, sigurno možemo ručno umetnuti tačka-zareze, baš kao tačka-zareze u poslednjem primeru koda. Drugim rečima, ove tačka-zareze su opcione u programiranju.

Drugo pravilo znači da su poslednja tačka-zarez u deklaraciji sa više stavki pre znaka zatvaranja ) i poslednja tačka-zarez unutar bloka koda ili deklaracije tipa (strukture ili interfejsa) pre znaka zatvaranja } opcioni. Ako poslednja tačka-zarez nedostaje, kompajleri će je automatski vratiti.

Drugo pravilo nam omogućava da napišemo sledeći validan kod.

```go
import (_ "math"; "fmt")

var (a int; b string)
const (M = iota; N)
type (MyInt int; T struct{x bool; y int32})
type I interface{m1(int) int; m2() string}
func f() {print("a"); panic(nil)}
```

Kompilatori će automatski umetnuti izostavljene tačke-zareze umesto nas, kao što je prikazano u sledećem kodu.

```go
var (a int; b string;);
const (M = iota; N;);
type (MyInt int; T struct{x bool; y int32;};);
type I interface{m1(int) int; m2() string;};
func f() {print("a"); panic(nil);};
```

Kompilatori neće umetati tačka-zareze ni za jedan drugi scenario. Moramo ručno umetnuti tačka-zareze po potrebi za ostale scenarije. Na primer, prva tačka-zarez u svakom redu u poslednjem primeru koda je obavezna. Tačka-zarezi u sledećem primeru su takođe obavezni.

```go
var a = 1; var b = true
a++; b = !b
print(a); print(b)
```

Iz ova dva pravila znamo da se tačka-zarez nikada neće umetnuti odmah posle fork ljučne reči. Zato je primer for gole petlje prikazan gore validan.

Jedna od posledica pravila umetanja tačke-zareza je da operacije inkrementa i dekrementa moraju da se pojavljuju kao iskazi. Ne mogu se koristiti kao izrazi. Na primer, sledeći kod je nevažeći.

```go
func f() {
    a := 0
    // The following two lines both fail to compile.
    println(a++) // unexpected ++, expecting comma or )
    println(a--) // unexpected --, expecting comma or )
}
```

Razlog zašto je gornji kod nevažeći je taj što će ga kompajleri videti kao

```go
func f() {
    a := 0;
    println(a++;);
    println(a--;);
}
```

Još jedna posledica pravila umetanja tačke i zareza je da ne možemo prekinuti red pre tačke. u selektoru. Možemo prekinuti red samo posle tačke, kao što pokazuje sledeći kod.

```go
anObject.
    MethodA().
    MethodB().
    MethodC()
```

dok sledeći kod ne uspeva da se kompajlira.

```go
anObject
    .MethodA()
    .MethodB()
    .MethodC()
```

Kompilatori će u modifikovanoj verziji ubaciti tačku-zarez na kraj svakog reda, tako da je gornji kod ekvivalentan sledećem kodu koji je očigledno nevažeći.

```go
anObject;
   .MethodA();
   .MethodB();
   .MethodC();
```

Pravila umetanja tačke-zareza nam omogućavaju da pišemo čistiji kod. Takođe omogućavaju pisanje validnog, ali pomalo čudnog koda. Na primer

```go
package main
import "fmt"

func alwaysFalse() bool {return false}

func main() {
    for
        i := 0
        i < 6
        i++ {
            // use i...
        }

    if x := alwaysFalse()
    !x {
        // do something...
    }

    switch alwaysFalse()
    {
        case true: fmt.Println("true")
        case false: fmt.Println("false")
    }
}
```

Sva tri bloka kontrolnog toka su validna. Kompilatori će umetnuti tačku-zarez na kraj svakog od redova 9 , 10 , 15 i 20.

Imajte na umu da će **switch-case** blok u gornjem primeru ispisati **true** umesto **false**. Razlikuje se od:

```go
switch alwaysFalse() {
    case true: fmt.Println("true")
    case false: fmt.Println("false")
}
```

Ako koristimo **go fmt** komandu za formatiranje prethodne, tačka-zarez će se automatski dodati nakon *alwaysFalse()* poziva, tako da će postati

```go
switch alwaysFalse();
{
    case true: fmt.Println("true")
    case false: fmt.Println("false")
}
```

Modifikovana verzija je ekvivalentna sledećoj.

```go
switch alwaysFalse(); true {
    case true: fmt.Println("true")
    case false: fmt.Println("false")
}
```

Zato će odštampati true.

Dobra je navika da se to često radi `go fmt` i `go vet` za vaš kod.

U retkim slučajevima, pravila umetanja tačke-zareza takođe čine da neki kod izgleda validno, ali je zapravo nevažeći. Na primer, sledeći isečak koda se ne kompajlira.

```go
func f(x int) {
    switch x {
        case 1:
        {
            goto A
            A: // compiles okay
        }
        case 2:
            goto B
            B: // syntax error: missing statement after label
        case 0:
            goto C
            C: // compiles okay
    }
}
```

Poruka o grešci pri kompajlaciji ukazuje da mora postojati naredba nakon deklaracije oznake. Ali izgleda da nijedna oznaka u gornjem primeru nije praćena naredbom. Zašto je samo B: deklaracija oznake nevažeća? Razlog je taj što će, prema drugom pravilu umetanja tačke-zareza pomenutom gore, kompajleri umetnuti tačku-zarez ispred svakog karaktera } koji sledi deklaracije oznake A: i C:. Kao što je prikazano u sledećem kodu.

```go
func f(x int) {
    switch x {
    case 1:
    {
        goto A
        A:
    ;} // a semicolon is inserted here
    case 2:
        goto B
        B: // syntax error: missing statement after label
    case 0:
        goto C
        C:
    ;} // a semicolon is inserted here
}
```

Samo tačka-zarez zapravo predstavlja prazan izraz, zbog čega su deklaracije A:i C: labele validne. S druge strane, B: deklaraciju oznake prati case 0:, što nije izraz, pa je B: deklaracija oznake nevažeća.

Možemo ručno umetnuti tačku-zarez (praznu izjavu) na kraj svake B: deklaracije oznake kako bi se kompajlirala ispravno.

## Zarez ( **,** ) se neće automatski umetnuti

U nekim sintaksnim oblicima koji sadrže više sličnih elemenata, zarezi se koriste kao separatori, kao što su složeni literali, liste argumenata funkcija, liste parametara funkcija i liste rezultata funkcija. U takvom sintaksnom obliku, poslednju stavku uvek može pratiti zarez. Ako je sledeći zarez poslednji efektivni znak u odgovarajućoj liniji koda, onda je zarez obavezan, u suprotnom je opcionalna. Kompilatori neće automatski umetati zapetaje ni u jednom slučaju.

Na primer, sledeći isečak koda je validan.

```go
func f1(a int, b string,) (x bool, y int,) {
    return true, 789
}

var f2 func (a int, b string) (x bool, y int)
var f3 func (a int, b string, // the last comma is required
) (x bool, y int,             // the last comma is required
)

var _ = []int{2, 3, 5, 7, 9,} // the last comma is optional
var _ = []int{2, 3, 5, 7, 9,  // the last comma is required
}

var _ = []int{2, 3, 5, 7, 9}
var _, _ = f1(123, "Go",) // the last comma is optional
var _, _ = f1(123, "Go",  // the last comma is required
)
var _, _ = f1(123, "Go")

// The same for explicit conversions.
var _ = string(65,) // the last comma is optional
var _ = string(65,  // the last comma is required
)
```

Međutim, sledeći isečak koda je nevažeći, jer će kompajleri umetnuti tačku-zarez za svaki red u kodu, osim drugog reda. Postoje tri reda koja će izazvati unexpected newlinesintaksičke greške.

```go
func f1(a int, b string,) (x bool, y int // error
) {
    return true, 789
}
var _ = []int{2, 3, 5, 7 // error: unexpected newline
}
var _, _ = f1(123, "Go" // error: unexpected newline
)
```

## Završne reči

Na kraju, opišimo pravila preloma reda u Go jeziku prema gore navedenim objašnjenjima.

U programu Go, prelom reda je u redu (neće uticati na ponašanje koda) ako:

- dešava se odmah nakon ključne reči koja nije break, continuei return, ili nakon bilo koje od tri ključne reči koje nisu praćene oznakama ili rezultatima vraćanja;
- to se dešava odmah posle tačke-zareza, bez obzira da li je tačka-zarez umetnut eksplicitno ili implicitno;
- to ne dovodi do implicitnog umetanja tačke-zareza.

Kao i neki drugi detalji dizajna u Go jeziku, postoje i pohvale i kritike za pravila umetanja tačke-zareza. Nekim programerima se ne sviđaju pravila, jer misle da ograničavaju slobodu stilova koda. Hvalitelji smatraju da pravila ubrzavaju kompajliranje koda i da kod koji su napisali različiti programeri izgleda slično, tako da je lako razumeti kod koji su napisali jedni drugi.

[Refleksije][0214] | [Sadržaj][00] | [Više o defer pozivima][0302]  

[0214]: 0214_Refleksije.md  
[00]: 00_Sadrzaj.md  
[0302]: 0302_Više_o_odloženim_funkcijskim_pozivima.md
