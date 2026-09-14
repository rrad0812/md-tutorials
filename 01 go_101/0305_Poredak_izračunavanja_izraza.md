
# Poredka evaluacije izraza

[Blokovi koda i opsezi identifikatora][0304] | [Sadržaj][00]

Ovaj članak će objasniti redoslede evaluacije izraza u svim vrstama scenarija.

Izraz se izračunava nakon izraza od kojih zavisi

Ovo je lako razumeti. Očigledan primer je izraz koji se izračunava kasnije od svojih podizraza. Na primer, u pozivu funkcije f(x, y[n]),

- f()se izračunava kasnije od zavisnih izraza, uključujući f, xi y[n].
- Izračunavanje izraza y[n]je kasnije od izračunavanja ni y.

Molimo vas da pročitate redosled inicijalizacije elemenata programskog koda za još jedan primer o redosledu inicijalizacije promenljivih na nivou paketa.

Redosled inicijalizacije promenljivih na nivou paketa

Kada se paket učita tokom izvršavanja, Go runtime će pokušati da inicijalizuje neinicijalizovane promenljive na nivou paketa koje nemaju zavisnosti od neinicijalizovanih promenljivih, prema njihovom redosledu deklaracije, sve dok se nijedna promenljiva ne inicijalizuje u takvom procesu. Za uspešno kompajliran Go program, ne bi trebalo da bude neinicijalizovanih promenljivih nakon završetka svih takvih procesa.

Promenljive na nivou paketa koje se pojavljuju kao prazni identifikatori tretiraju se kao i sve ostale promenljive u procesu inicijalizacije.

Na primer, u sledećem programu, promenljiva azavisi od b, a promenljive ci _zavise od a. Dakle

- Prva inicijalizovana promenljiva je b, što je prva promenljiva na nivou paketa bez zavisnosti od drugih promenljivih na nivou paketa.
- Druga inicijalizovana promenljiva je a. Nakon što bje inicijalizovana, aje prva promenljiva na nivou paketa bez zavisnosti od neinicijalizovanih promenljivih na nivou paketa.
- Treća i četvrta inicijalizovane promenljive su _ i c. Nakon inicijalizovanih promenljivih a i b, \_ i c obe ne zavise od neinicijalizovanih promenljivih na nivou paketa.

```go
package main

import "fmt"

var (
    _ = f()
    a = b / 2
    b = 6
    c = f()
)

func f() int {
    a++
    return a
}

func main() {
    fmt.Println(a, b, c) // 5 6 5
}
```

Gore navedeni program štampa 5 6 5.

Više promenljivih na levoj strani specifikacije promenljive inicijalizovanih jednim viševrednosnim izrazom na desnoj strani inicijalizuju se zajedno. Na primer, za deklaraciju promenljive na nivou paketa var x, y = f(), promenljive xi yće biti inicijalizovane zajedno. Drugim rečima, nijedna druga promenljiva neće biti inicijalizovana između njih.

Deklaracija promenljive na nivou paketa sa više izraza izvorne vrednosti biće rastavljena na više deklaracija promenljivih sa jednom izvornom vrednošću pre inicijalizacije svih promenljivih na nivou paketa. Na primer.

```go
var m, n = expr1, expr2
```

je ekvivalentno

```go
var m = expr1
var n = expr2
```

Ako postoje skrivene zavisnosti između promenljivih, redosled inicijalizacije između tih promenljivih nije naveden. U sledećem primeru (kopiranom iz Go specifikacije),

- Promenljiva će sigurno abiti inicijalizovana nakon toga ,b
- Ali da li xje inicijalizovano pre b, između bi aili posle a, nije navedeno.
- i trenutak u kojem sideEffect()se funkcija poziva (pre ili posle xinicijalizacije) takođe nije naveden.

```go
// x has a hidden dependency on a and b
var x = I(T{}).ab()
// Assume sideEffect is unrelated to x, a, and b.
var _ = sideEffect()
var a = b
var b = 42

type I interface    { ab() []int }
type T struct{}
func (T) ab() []int { return []int{a, b} }
```

Imajte na umu da Go specifikacija ne određuje obavezno redosled kompajliranja izvornih datoteka u paketu, zato pokušajte da ne stavljate neke promenljive na nivou paketa u različite izvorne datoteke u paketu ako postoje komplikovane zavisnosti između tih promenljivih; u suprotnom, promenljiva bi mogla biti inicijalizovana na različite vrednosti od strane različitih Go kompajlera.

I imajte na umu da neke verzije Go Toolchain-a ne implementiraju pravilno gore navedena pravila opisana u ovom odeljku. Na primer:

- greška u Go Toolchain-u 1.22.0 - 1.22.4.
- greška u verzijama Go Toolchain-a pre verzije 1.23.

## Redosledi evaluacije operanda u logičkim operacijama

U bulovom izrazu a && b, izraz desnog operanda bće biti izračunat samo ako je izraz levog operanda aizračunat kao true. Dakle b, biće izračunat, ako je potrebno, nakon izračunavanja a.

U bulovom izrazu a || b, izraz desnog operanda bće biti izračunat samo ako je izraz levog operanda aizračunat kao false. Dakle b, biće izračunat, ako je potrebno, nakon izračunavanja a.

### Uobičajeni redosled

Za evaluacije unutar tela funkcije, Go specifikacija kaže

..., prilikom izračunavanja operanda izraza, dodele ili naredbe za vraćanje, svi pozivi funkcija, pozivi metoda i operacije komunikacije (kanala) se izračunavaju leksičkim redosledom sleva nadesno.

Upravo opisani redosled naziva se uobičajeni redosled.

Imajte na umu da eksplicitna konverzija vrednosti T(v)nije poziv funkcije.

Na primer, u izrazu []int{x, fa(), fb(), y}, pretpostavimo da su xi ydve promenljive, faa fbsu dve funkcije, onda je zagarantovano da će poziv fa()biti izvršen pre fb(). Međutim, sledeći redosledi izvršavanja nisu navedeni u Go specifikaciji:

- redosled evaluacije x(ili y) i fa()(ili fb()).
- redosled evaluacije x, y, fai fb.

Još jedan primer, sledeći zadatak, je demonstriran u Go specifikaciji.

y[z.f()], ok = g(h(a, b), i()+x[j()], <-c), k()

gde:

- c je izraz kanala i biće procenjen kao vrednost kanala.
- g, h, i, ji ksu izrazi funkcija.
- f je metod izražavanja z.

Takođe, uzimajući u obzir pravilo pomenuto u poslednjem odeljku, kompajleri bi trebalo da garantuju sledeće redoslede evaluacije tokom izvršavanja.

- Pozivi funkcija, pozivi metoda i operacije komunikacije kanala se dešavaju redosledom z.f()→ h()→ i()→ j()→ <-c→ g()→ k().
- h()se izračunava nakon izračunavanja izraza h, ai b.
- y[]se procenjuje nakon procene z.f().
- z.f()se izračunava nakon izračunavanja izraza z.
- x[]se procenjuje nakon procene j().

Međutim, sledeća naređenja (i još mnogo drugih) nisu navedena.

- Redosled evaluacije za y, z, g, h, a, b, x, i, j, ci k.
- Redosled evaluacije y[], x[]i <-c.

Uobičajenim redosledom, znamo da će sledeće deklarisane promenljive i x( takođe prikazano u Go specifikaciji) biti inicijalizovane dvosmislenim vrednostima. mn

```go
a := 1
f := func() int { a++; return a }
// x may be [1, 2] or [2, 2]: evaluation order
// between a and f() is not specified.
x := []int{a, f()}
// m may be {2: 1} or {2: 2}: evaluation order
// between the two map element assignments is
// not specified.
m := map[int]int{a: 1, a: 2}
// n may be {2: 3} or {3: 3}: evaluation order
// between the key and the value is unspecified.
n := map[int]int{a: f()}
```

## Redosaled evaluacije i dodela zadataka u izjavama o dodeli zadataka

Pored gore navedenih pravila, Go specifikacija više određuje o redosledu izračunavanja izraza, odnosno redosledu pojedinačnih dodela u naredbi dodele:

Dodeljivanje se odvija u dve faze. Prvo, operandi indeksnih izraza i indirekcije pokazivača (uključujući implicitnu indirekciju pokazivača u selektorima) sa leve i izraza sa desne strane se svi izračunavaju uobičajenim redosledom. Drugo, dodeljivanja se vrše redosledom sleva nadesno.

Kasnije, prvu fazu možemo nazvati fazom evaluacije, a drugu fazu fazom izvršenja.

Go specifikacija ne precizira jasno da li dodele izvršene tokom druge faze mogu uticati na rezultate evaluacije izraza dobijene u prvoj fazi, što je ikada izazvalo neke sporove . Dakle, ovde će ovaj članak više objasniti redoslede evaluacije u dodelama vrednosti.

Prvo, da razjasnimo da zadaci izvršeni tokom druge faze ne utiču na rezultate evaluacije izraza dobijene na kraju prve faze.

Da bismo sledeća objašnjenja učinili lakšim, pretpostavljamo da je vrednost kontejnera (sečka ili mape) izraza odredišta indeksa u dodeli uvek adresabilna. Ako nije, možemo smatrati da je vrednost kontejnera već sačuvana i zamenjena privremenom adresabilnom vrednošću kontejnera pre nego što izvršimo drugu fazu.

U trenutku završetka faze evaluacije i neposredno pre početka faze izvršavanja, svaki odredišni izraz sa leve strane dodele je evaluiran kao njegov elementarni oblik. Različiti odredišni izrazi imaju različite elementarne oblike.

- Ako je odredišni izraz prazan identifikator, onda je njegov elementarni oblik i dalje prazan identifikator.
- Ako je odredišni izraz indeksni izraz kontejnera (niza, isečka ili mape) `c[k]`, onda je njegova elementarni oblik `(*cAddr)[x]`, gde `cAddr` je pokazivač koji pokazuje na `c`.
- U drugim slučajevima, odredišni izraz mora rezultirati adresabilnom vrednošću, tada je njegov elementarni oblik dereferenca na adresu rezultata evaluacije odredišnog izraza.

Pretpostavimo da su ai bdve adresabilne promenljive istog tipa, sledeće dodeljivanje

```go
a, b = b, a
```

biće izvršen kao što slede sledeći koraci:

```go
// The evaluation phase:
P0 := &a; P1 := &b
R0 := b; R1 := a 

// The elementary form: *P0, *P1 = R0, R1

// The carry-out phase:
*P0 = R0
*P1 = R1
```

Evo još jednog primera, u kome je x[0] umesto x[1] modifikovano.

```go
x := []int{0, 0}
i := 0
i, x[i] = 1, 2
fmt.Println(x) // [2 0]
```

Dekomponovani koraci izvršenja za liniju 3 prikazani ispod su sledeći:

```go
// The evaluation phase:
P0 := &i; P1 := &x; T2 := i
R0 := 1; R1 := 2
// Now, T2 == 0

// The elementary form: *P0, (*P1)[T2] = R0, R1

// The carry-out phase:
*P0 = R0
(*P1)[T2] = R1
```

Primer koji je malo složeniji.

```go
package main
import "fmt"

func main() {
    m := map[string]int{"Go": 0}
    s := []int{1, 1, 1}; olds := s
    n := 2
    p := &n
    s, m["Go"], *p, s[n] = []int{2, 2, 2}, s[1], m["Go"], 5
    fmt.Println(m, s, n) // map[Go:1] [2 2 2] 0
    fmt.Println(olds)    // [1 1 5]
}
```

Dekomponovani koraci izvršenja za liniju 10 prikazani ispod su sledeći:

```go
// The evaluation phase:
P0 := &s; PM1 := &m; K1 := "Go"; P2 := p; PS3 := &s; T3 := 2
R0 := []int{2, 2, 2}; R1 := s[1]; R2 := m["Go"]; R3 := 5
// now, R1 == 1, R2 == 0

// The elementary form:
//     *P0, (*PM1)[K1], *P2, (*PS3)[T3] = R0, R1, R2, R3

// The carry-out phase:
*P0 = R0
(*PM1)[K1] = R1
*P2 = R2
(*PS3)[T3] = R3
```

Sledeći primer rotira sve elemente u sloju za jedan indeks.

```go
x := []int{2, 3, 5, 7, 11}
t := x[0]
var i int
for i, x[i] = range x {}
x[i] = t
fmt.Println(x) // [3 5 7 11 2]
```

Još jedan primer:

```go
x := []int{123}
x, x[0] = nil, 456        // will not panic
x, x[0] = []int{123}, 789 // will panic
```

Iako je legalno, ne preporučuje se korišćenje složenih viševrednosnih dodela u Go-u, jer njihova čitljivost nije dobra i imaju negativne efekte i na brzinu kompajliranja i na performanse izvršavanja.

Kao što je gore pomenuto, nisu svi redosledi navedeni u Go specifikaciji za dodeljivanje vrednosti, tako da neki loše napisan kod može proizvesti različite rezultate. U sledećem primeru, redosled izraza x+1i f(&x)nije naveden. Dakle, primer može ispisati 100 99ili 1 99.

```go
package main
import "fmt"

func main() {
    f := func (p *int) int {
        *p = 99
        return *p
    }

    x := 0
    y, z := x+1, f(&x)
    fmt.Println(y, z)
}
```

Sledi još jedan primer koji će ispisati dvosmislene rezultate. Može ispisati 1 7 2, 1 8 2ili 1 9 2, u zavisnosti od različitih implementacija kompajlera.

```go
package main
import "fmt"

func main() {
    x, y := 0, 7
    f := func() int {
        x++
        y++
        return x
    }
    fmt.Println(f(), y, f())
}
```

## Redosledi evaluacije izraza u switch-case blokovima koda

Redosled izračunavanja izraza u switch-casebloku koda je već opisan . Ovde je prikazan samo jedan primer. Jednostavno govoreći, pre nego što se uđe u blok koda grane, izrazi caseće biti izračunati i upoređeni sa izrazom switch jedan po jedan, sve dok poređenje ne rezultira sa true.

```go
package main
import "fmt"

func main() {
    f := func(n int) int {
        fmt.Printf("f(%v) is called.\n", n)
        return n
    }

    switch x := f(3); x + f(4) {
    default:
    case f(5):
    case f(6), f(7), f(8):
    case f(9), f(10):
    }
}
```

Tokom izvršavanja, f()pozivi će biti evaluirani redosledom od vrha do dna i s leva na desno, sve dok poređenje ne rezultira sa true. Dakle f(8), , f(9)i f(10)se neće evaluirati u ovom primeru.

Izlaz:

```go
Poziva se f(3).
Poziva se f(4).
Poziva se f(5).
Poziva se f(6).
Poziva se f(7).
```

## Redosledi evaluacije izraza u select-case blokovima koda

Prilikom izvršavanja select-case bloka koda, pre ulaska u blok koda grananja, svi operandi kanala operacija prijema i operandi naredbi slanja uključeni u select-case blok koda se procenjuju tačno jednom, po redosledu izvora (odozgo nadole, s leva na desno).

Imajte na umu da će ciljni izraz kojem se dodeljuje caseoperacija prijema biti izračunat samo ako se ta operacija prijema kasnije izabere.

U sledećem primeru, izraz *fptr("aaa")nikada neće biti izračunat, jer njegova odgovarajuća operacija prijema <-fchan("bbb", nil)neće biti izabrana.

```go
package main
import "fmt"

func main() {
    c := make(chan int, 1)
    c <- 0
    fchan := func(info string, c chan int) chan int {
        fmt.Println(info)
        return c
    }
    fptr := func(info string) *int {
        fmt.Println(info)
        return new(int)
    }

    select {
    case *fptr("aaa") = <-fchan("bbb", nil): // blocking
    case *fptr("ccc") = <-fchan("ddd", c):   // non-blocking
    case fchan("eee", nil) <- *fptr("fff"):  // blocking
    case fchan("ggg", nil) <- *fptr("hhh"):  // blocking
    }
}
```

Izlaz gore navedenog programa:

```go
bbb
ddd
njen
blin
ggg
hhh
ccc
```

Imajte na umu da je izraz *fptr("ccc")poslednji izračunati izraz u gornjem primeru. Izračunava se nakon što <-fchan("ddd", c)je izabrana odgovarajuća operacija prijema.

[Blokovi koda i opsezi identifikatora][0304] | [Sadržaj][00]

[0304]: 0304_Blokovi_koda_i%20_opsezi_identifikatora.md
[00]: 00_Sadrzaj.md
