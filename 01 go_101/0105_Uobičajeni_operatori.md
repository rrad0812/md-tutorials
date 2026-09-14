
# Uobičajeni operatori

[Konstante i promenljive][0104] | [Sadržaj][00] | [Deklaracije i pozivi funkcija][0106]

Operatorske operacije su operacije koje koriste sve vrste operatora. Ovaj članak će predstaviti uobičajene operatore u jeziku Go. Više operatora biće predstavljeno u drugim člancima kasnije.

## O nekim opisima u objašnjenjima operatora

Ovaj članak će predstaviti samo aritmetičke operatore, bitske operatore, operatore poređenja, bulove operatore i operator spajanja stringova. Ovi operatori su ili binarni operatori ili unarni operatori. Binarna operatorska operacija uzima dva operanda, a unarna operatorska operacija uzima samo jedan operand.

Sve operatorske operacije predstavljene u ovom članku vraćaju po jedan rezultat.

Ovaj članak ne teži tačnosti nekih opisa. Na primer, kada se kaže da binarni operator zahteva da tipovi njegova dva operanda moraju biti isti, to znači:

- Ako su oba operanda tipizirane vrednosti, onda njihovi tipovi moraju biti isti, ili se jedan operand može implicitno konvertovati u tip drugog.
- Ako je samo jedan od dva operanda tipiziran, onda drugi (netipizovani) operand mora biti predstavljiv kao vrednost tipa tipiziranog operanda, ili vrednosti podrazumevanog tipa drugog (netipiziranog) operanda mogu biti implicitno konvertovane u tipizirani tip tipiziranog operanda.
- Ako su oba operanda netipizirane vrednosti, onda moraju biti obe bulove vrednosti, obe string vrednosti ili obe osnovne numeričke vrednosti.

Slično tome, kada se kaže da operator, bilo binarni ili unarni operator, zahteva da tip jednog od njegovih operanda bude određenog tipa, to znači:

- Ako je operand tipiziran, onda njegov tip mora biti, ili se može implicitno konvertovati u, taj određeni tip.
- Ako operand nije tipiziran, onda netipizirana vrednost mora biti predstavljiva kao vrednost tog određenog tipa, ili vrednosti podrazumevanog tipa operanda mogu biti implicitno konvertovane u taj određeni tip.

## Konstantni izrazi

Pre nego što uvedemo sve vrste operatora, trebalo bi da znamo šta su konstantni izrazi i činjenicu u izračunavanju konstantnih izraza. Izrazi će biti objašnjeni u kasnijem članku, pod nazivom "Izrazi i iskazi". Trenutno, trebalo bi da znamo samo da je većina operacija pomenutih u ovom članku izraz.

Ako su svi operandi uključeni u izraz konstante, onda se taj izraz naziva **konstantni izraz**. Svi konstantni izrazi se izračunavaju tokom kompajliranja. Rezultat izračunavanja konstantnog izraza je i dalje konstanta.

Ako samo jedan operand u izrazu nije konstanta, izraz se naziva **nekonstantnim izrazom**.

## Aritmetički operatori

Go podržava pet osnovnih binarnih aritmetičkih operatora:

| Operator | Ime | Zahtevi za dva operanda |
| -------- | --- | ----- |
| + | sabiranje | [1] |
| - | oduzimanje | [1] |
| * | množenje | [1] |
| / | divizija | [1] |
| % | ostatak | [2] |

> [!Note]
> [1] - Oba operanda moraju biti vrednosti istog osnovnog numeričkog tipa.  
> [2] - Oba operanda moraju biti vrednosti istog osnovnog celobrojnog tipa.

Pet operatora se često nazivaju i operatori **zbira**. **razlike**. **proizvoda**. **količnika** i **modula**. respektivno. U Go 101 neće se detaljno objašnjavati kako ove operacije sa operatorima funkcionišu.

Go podržava šest binarnih aritmetičkih operatora na više bitova:

| Operator | Ime | Zahtevi za dva operanda | Objašnjenja mehanizma |
| -------- | --- | ------- | ----- |
| & | bitski AND | [1] | `1100 & 1010` rezultira `1000` |
| \| | bitsko OR | [1] | `1100 \| 1010` rezultira `1110` |
| ^ | bitsko XOR | [1] | `1100 ^ 1010` rezultira `0110` |
| &^ | bitsko brisanje | [1] | `1100 &^ 1010` rezultira `0100` |
| << | bitsko pomeranje ulevo | [2] | `1100 << 3` rezultira `1100000` |
| >> | bitsko pomeranje udesno | [2] | `1100 >> 3`  rezultira `1` |

> [!Note]
> [1] Oba operanda moraju biti vrednosti istog celobrojnog tipa.  
> [2] Levi operand mora biti ceo broj, a desni operand takođe mora biti ceo broj
> (ako je konstanta, onda mora biti nenegativna), njihovi tipovi ne moraju biti
> identični.

**Napomena**:  
Pre verzije 1.13, desni operand mora biti neoznačeni ceo broj ili netipizovana celobrojna konstanta koja se može predstaviti kao uintvrednost. Negativan desni operand (mora biti nekonstantan) će izazvati paniku tokom izvršavanja.

**Napomena**:  
Kod operacije pomeranja bitova udesno, svi oslobođeni bitovi sa leve strane se popunjavaju bitom znaka (najvišim bitom) levog operanda. Na primer, ako je levi operand vrednost **int8**, *-128* ili *10000000* u binarnom literalu, onda je `10000000 >> 2` rezultat `11100000`, odnosno `-32`.

Go takođe podržava tri unarna aritmetička operatora:

| Operator | Ime | Objašnjenja |
| -------- | --- | ----------- |
| `+` | pozitivno | `+n` je ekvivalentno sa `0 + n` |
| `-` | negativno | `-n` je ekvivalentno sa `0 - n` |
| `^` | bitski komplement (bitsko not) | `^n` je ekvivalentno sa `m^n`, gde je m vrednost čiji su svi bitovi jednaki 1. </br> Na primer, ako je tip n, `int8` onda je m jednako -1, a ako je tip n, `uint8` onda je m jednako 0xFF. |

**Napomena**:

- U mnogim drugim jezicima, operator komplementa po bitovima se označava kao `~`.
- Kao i mnogi drugi jezici, binarni operator sabiranja `+` može se koristiti i kao operator spajanja stringova. što će biti predstavljeno u nastavku.
- Kao i u jezicima C i C++, binarni operator množenja `*` može se koristiti i kao operator dereferenciranja pokazivača. a bitski operator `&` može se koristiti i kao operator uzimanja adrese. Molimo vas da kasnije pročitate pokazivače u Go-u za detalje.
- Za razliku od Java jezika, Go podržava neoznačene celobrojne tipove, tako da neoznačeni operator pomeranja `>>>` ne postoji u Gou.
- U programu Go nema operatora potencije, umesto toga koristite `Powf` unkciju iz `math` standardnog paketa. Paketi i uvoz paketa biće predstavljeni u sledećem članku paketi i uvozi.
- Operator za brisanje bitova `&^` je jedinstveni operator u jeziku Go. `m &^ n` ekvivalentan je `m & (^n)`.

Primer:

```go
func main() {
    var (
        a, b float32 = 12.0, 3.14
        c, d int16   = 15, -6
        e    uint8   = 7
    )

    // The ones compile okay.
    _ = 12 + 'A' // two numeric untyped operands
    _ = 12 - a   // one untyped and one typed operand
    _ = a * b    // two typed operands
    _ = c % d
    _, _ = c + int16(e), uint8(c) + e
    _, _, _, _ = a / b, c / d, -100 / -9, 1.23 / 1.2
    _, _, _, _ = c | d, c & d, c ^ d, c &^ d
    _, _, _, _ = d << e, 123 >> e, e >> 3, 0xF << 0
    _, _, _, _ = -b, +c, ^e, ^-1

    // The following ones fail to compile.
    _ = a % b   // error: a and b are not integers
    _ = a | b   // error: a and b are not integers
    _ = c + e   // error: type mismatching
    _ = b >> 5  // error: b is not an integer
    _ = c >> -5 // error: -5 is not representable as uint

    _ = e << uint(c) // compiles ok
    _ = e << c       // only compiles ok since Go 1.13
    _ = e << -c      // only compiles ok since Go 1.13,
                     // will cause a panic at run time.
    _ = e << -1      // error: right operand is negative
}
```

## O prekoračenjima

Prekoračenja nisu dozvoljena za tipizirane konstantne vrednosti, ali su dozvoljeni za nekonstantne i netipizirane konstantne vrednosti, bilo da su vrednosti međurezultati ili konačni rezultati. Prekoračenja će biti skraćeni (ili obmotani) za nekonstantne vrednosti, ali prekoračenja (za podrazumevane tipove) na netipiziranoj konstantnoj vrednosti neće biti skraćeni (ili obmotani).

Primer:

```go
// Results are non-constants.
var a, b uint8 = 255, 1

// Compiles ok, higher overflowed bits are truncated.
var c = a + b  // c == 0

// Compiles ok, higher overflowed bits are truncated.
var d = a << b // d == 254

// Results are untyped constants.
const X = 0x1FFFFFFFF * 0x1FFFFFFFF // overflows int
const R = 'a' + 0x7FFFFFFF          // overflows rune

// The above two lines both compile ok, though the
// two untyped value X and R both overflow their
// respective default types.

// Operation results or conversion results are
// typed values. These lines all fail to compile.
var e = X // error: untyped constant X overflows int
var h = R // error: constant 2147483744 overflows rune
const Y = 128 - int8(1)  // error: 128 overflows int8
const Z = uint8(255) + 1 // error: 256 overflow uint8
```

## O rezultatima aritmetičkih operatorskih operacija

Osim operacija pomeranja po bitovima, rezultat operacije binarnog aritmetičkog operatora:

- je tipizirana vrednost istog tipa kao i dva operanda ako su oba operanda tipizirane vrednosti istog tipa.
- je tipizirana vrednost istog tipa tipiziranog operanda ako je samo jedan od dva operanda tipizirana vrednost. U izračunavanju, druga (netipizirana) vrednost će biti izvedena kao vrednost tipa tipiziranog operanda. Drugim rečima, netipizirani operand će biti implicitno konvertovan u tip tipiziranog operanda.
- je i dalje netipizovana vrednost ako su oba operanda netipizovana. Podrazumevani tip rezultujuće vrednosti je jedan od dva podrazumevana tipa i to je onaj koji se pojavljuje poslednji na ovoj listi: int, rune, float64, complex128. Na primer, ako je podrazumevani tip jednog netipizovanog operanda int, a drugog je rune, onda je podrazumevani tip rezultujuće netipizovane vrednosti rune.

Pravila za rezultat operacije pomeranja po bitovima su malo komplikovana. Prvo, vrednost rezultata je uvek celobrojna vrednost. Da li je tipizovana ili netipizovana zavisi od specifičnih scenarija.

- Ako je levi operand tipizirana vrednost (celobrojna vrednost), onda je tip rezultata isti kao i tip levog operanda.
- Ako je levi operand netipizovana vrednost, a desni operand konstanta, onda će levi operand uvek biti tretiran kao celobrojna vrednost. Ako njegov podrazumevani tip nije celobrojni tip, mora biti predstavljiv kao netipizovani ceo broj, a njegov podrazumevani tip će biti posmatran kao int. U takvim slučajevima, rezultat je takođe netipizovana vrednost, a podrazumevani tip rezultata je isti kao i levi operand.
- Ako je levi operand netipizovana vrednost, a desni operand je nekonstantni ceo broj, onda će levi operand prvo biti konvertovan u tip koji bi pretpostavio kada bi se operacija pomeranja po bitovima zamenila samo njegovim levim operandom. Rezultat je tipizovana vrednost čiji je tip pretpostavljenog tipa.

Primer:

```go
func main() {
    // Three untyped values. Their default
    // types are: int, rune(int32), complex64.
    const X, Y, Z = 2, 'A', 3i

    var a, b int = X, Y // two typed values.

    // The type of d is the default type of Y: rune.
    d := X + Y
    
    // The type of e is the type of a: int.
    e := Y - a
    
    // The type of f is the types of a and b: int.
    f := a * b
    // The type of g is Z's default type: complex64.
    g := Z * Y

    // Output: 2 65 (+0.000000e+000+3.000000e+000i)
    println(X, Y, Z)

    // Output: 67 63 130 (+0.000000e+000+1.950000e+002i)
    println(d, e, f, g)
}
```

Još jedan primer (operacije pomeranja po bitovima):

```go
const N = 2

// A is an untyped value (default type as int).
const A = 3.0 << N // A == 12

// B is typed value (type is int8).
const B = int8(3.0) << N // B == 12

var m = uint(32)
// The following three lines are equivalent to
// each other. In the following two lines, the
// types of the two "1" are both deduced as
// int64, instead of int.
var x int64 = 1 << m
var y = int64(1 << m)
var z = int64(1) << m

// The following line fails to compile.
/*
var _ = 1.23 << m // error: shift of type float64
*/
```

Poslednje pravilo za operacije operatora pomeranja po bitovima jeste izbegavanje slučajeva da neke operacije pomeranja po bitovima vraćaju različite rezultate na različitim arhitekturama, ali razlike neće biti otkrivene na vreme. Na primer, ako se operand izvede kao int umesto int64, operacija po bitovima u liniji 13 (ili liniji 12 ) vratiće različite rezultate između 32-bitnih arhitektura (0) i 64-bitnih arhitektura (2 32 ), što može dovesti do nekih grešaka koje je teško otkriti na vreme.

Jedna zanimljiva posledica poslednjeg pravila za operaciju operatora pomeranja po bitovima prikazana je u sledećem isečku koda:

```go
const n = uint(2)
var m = uint(2)

// The following two lines compile okay.
var _ float64 = 1 << n
var _ = float64(1 << n)

// The following two lines fail to compile.
var _ float64 = 1 << m
var _ = float64(1 << m)
```

Razlog zašto se poslednja dva reda ne kompajliraju je taj što su oba ekvivalentna sledećim dvama redovima:

```go
var _ = float64(1) << m
var _ = 1.0 << m // error: shift of type float64

Još jedan primer:

package main

const n = uint(8)
var m = uint(8)

func main() {
    println(a, b) // 2 0
}

var a byte = 1 << n / 128
var b byte = 1 << m / 128
```

Gornji program ispisuje 2 0, jer su poslednja dva reda ekvivalentna

```go
var a = byte(int(1) << n / 128)
var b = byte(1) << m / 128
```

## O celobrojnom deljenju i operacijama sa ostatkom brojeva

Pretpostavimo da su *x* i *y* dva operanda istog celobrojnog tipa, celobrojni količnik *q = x / y* i ostatak *r = x % y* zadovoljavaju *x == q*y + r*, gde je |r| < |y|. Ako *r* nije nula, njegov znak je isti kao *x* (deljenik). Rezultat *x / y* je skraćen prema nuli.

Ako je delilac *y* konstanta, ne sme biti nula. Ako je delilac nula u vreme izvršavanja i on je ceo broj, javlja se panika u vreme izvršavanja. Panike su poput izuzetaka u nekim drugim jezicima. Više o panikama možemo saznati u ovom članku.

Primer:

```go
println( 5/3,   5%3)  // 1 2
println( 5/-3,  5%-3) // -1 2
println(-5/3,  -5%3)  // -1 -2
println(-5/-3, -5%-3) // 1 -2

println(5.0 / 3.0)     // 1.666667
println((1-1i)/(1+1i)) // -1i

var a, b = 1.0, 0.0
println(a/b, b/b) // +Inf NaN

_ = int(a)/int(b) // compiles okay but panics at run time.

// The following two lines fail to compile.
println(1.0/0.0) // error: division by zero
println(0.0/0.0) // error: division by zero
```

## Korišćenje op= za binarne aritmetičke operatore

Za binarni aritmetički operator **op**, **x = x op y** može se skratiti na **x op= y**. U skraćenom obliku, **x** će se izračunati samo jednom.

Primer:

```go
var a, b int8 = 3, 5
a += b
println(a) // 8

a *= a
println(a) // 64

a /= b
println(a) // 12

a %= b
println(a) // 2

b <<= uint(a)
println(b) // 20
```

## Operatori inkrementa ++ i dekrementa --

Kao i mnogi drugi popularni jezici, Go takođe podržava operatore inkrementa **++** i dekrementa **--**. Međutim, operacije koje koriste ova dva operatora ne vraćaju nikakve rezultate, tako da se takve operacije ne mogu koristiti kao izrazi. Jedini operand uključen u takvu operaciju mora biti numerička vrednost, numerička vrednost ne sme biti konstanta, a operator **++** ili **--** mora **slediti** operand.

Primer:

```go
package main

func main() {
    a, b, c := 12, 1.2, 1+2i
    a++ // ok. <=> a += 1 <=> a = a + 1
    b-- // ok. <=> b -= 1 <=> b = b - 1
    c++ // ok

    // The following lines fail to compile.
    // _ = a++
    // _ = b--
    // _ = c++
    // ++a
    // --b
    // ++c
}
```

## Operator za spajanje stringova

Kao što je gore pomenuto, operator sabiranja se takođe može koristiti za  spajanje stringova.

| Operator | Ime | Zahtevi za dva operanda |
| :------: | --- | ----------------------- |
| + | spajanje stringova | Oba operanda moraju biti vrednosti istog tipa stringa |

Oblik op= se takođe primenjuje i na operator spajanja srtingova.

Primer:

```go
println("Go" + "lang") // Golang
var a = "Go"
a += "lang"
println(a) // Golang
```

Ako je jedan od dva operanda operacije spajanja stringova tipizirani string, onda je tip rezultujućeg niza isti kao i tip tipiziranog tringa. Ako su oba operanda netipizirani (konstantni) stringovi, rezultat je takođe netipizirana vrednost stringa.

## Bulovi (tj. logički) operatori

Go podržava dva bulova binarna operatora i jedan bulov unarni operator:

| Operator | Ime | Zahtevi za dva operanda |
| :------: | --- | ----------------------- |
| && | bulov AND (binarni) - poznati kao uslovni AND | [1] |
| \|\| | bulov OR (binarni) - poznati kao uslovni OR | [1] |
| ! | bulov not (unarni) | [2] |

> [!Note]
> [1] - Oba operanda moraju biti vrednosti istog bulovog tipa.  
> [2] - Tip jedinog operanda mora biti bulov tip.

Možemo koristiti **!=** operator predstavljen u sledećem pododeljku kao bulov **xor** operator.

Objašnjenja mehanizma:

```go
// x    y       x && y   x || y   !x      !y
true    true    true     true     false   false
true    false   false    true     false   true
false   true    false    true     true    false
false   false   false    false    true    true
```

Ako je jedan od dva operanda tipizirana bulova vrednost, onda je tip rezultujuće bulove vrednosti isti kao i tip tipizirane bulove vrednosti. Ako su oba operanda netipizovane bulove vrednosti, rezultat je takođe netipizirana bulova vrednost.

## Operatori poređenja

Go podržava šest binarnih operatora za poređenje:

| Operator | Ime | Zahtevi za dva operanda |
| :------: | --- | ----------------------- |
| == | jednako | [1] |
| != | nije jednako | [1] |
| < | manje od | [2]] |
| <= | manje ili jednako | [2] |
| > | veći od | [2] |
| >= | veće ili jednako | [2] |

> [!Note]
> [1] - Generalno, tipovi njegova dva operanda moraju biti isti. Za detaljna
> pravila, pročitajte pravila poređenja u Go-u.  
> [2] - Oba operanda moraju biti vrednosti istog celobrojnog tipa, tipa sa
> pokretnim zarezom ili tipa stringa.

Tip rezultata bilo koje operacije poređenja je uvek netipizovana bulova vrednost. Ako su oba operanda operacije poređenja konstantna, rezultat je takođe konstantna (bulova) vrednost.

Kasnije, ako kažemo da su dve vrednosti uporedive, mislimo da se mogu uporediti pomoću operatora `==` i `!=`. Kasnije ćemo naučiti koje vrednosti kojih tipova nisu uporedive. Vrednosti osnovnih tipova su sve uporedive.

Imajte na umu da se ne mogu svi realni brojevi tačno predstaviti u memoriji, tako da poređenje dve vrednosti sa pokretnim zarezom (ili kompleksne) možda nije pouzdano. Trebalo bi da proverimo da li je apsolutna razlika dve vrednosti sa pokretnim zarezom manja od malog praga da bismo procenili da li su dve vrednosti sa pokretnim zarezom jednake.

## Prioritet operatora

Sledi prioritet operatera u jeziku Go. Operatori sa najvišeg mesta imaju veći prioritet. Operatori u istom redu imaju isti prioritet. Kao i mnogi drugi jezici, () se mogu se koristiti za unapređenje prioriteta.

| Prioritet | . | . | . | . | . | . | . |
| :-------: | :-: | :-: | :-: | :-: | :-: | :-: | :-: |
| I | * | / | % | << | >> | & | &^ |
| II | + | - | \| | ^ | | | |
| III | == | != | < | <= | > | >= | |
| IV | && | | | | | | |
| V | \|\| | | | | | | |

Jedna očigledna razlika u odnosu na neke druge popularne jezike je da je prioritet jezika `<<` i  `>>` veći od `+` i `-` u Go-u.

## Više o konstantnim izrazima

Sledeća deklarisana promenljiva će biti inicijalizovana kao 2.2 umesto 2.7. Razlog je taj što je prioritet operacije deljenja viši od operacije sabiranja, a u operaciji deljenja, oba 3 i 2 se posmatraju kao celi brojevi. Rezultat izračunavanja 3/2 je 1.

var x = 1.2 + 3/2

Dve imenovane konstante deklarisane u sledećem programu nisu jednake. U prvoj deklaraciji, oba 3i 2se posmatraju kao celi brojevi. Međutim, u drugoj deklaraciji, oba se posmatraju kao brojevi sa pokretnim zarezom.

```go
package main

const x = 3/2*0.1
const y = 0.1*3/2

func main() {
    println(x) // +1.000000e-001
    println(y) // +1.500000e-001
}
```

## Više operatora

Isto kao i u C/C++, postoje dva operatora vezana za pokazivače, `*` i `&`. `&` se koristi za uzimanje adrese adresabilne vrednosti a `*` koristi se za dereferenciranje vrednosti pokazivača. Za razliku od C/C++, u Go-u, vrednosti tipova pokazivača ne podržavaju aritmetičke operacije. Za više detalja, pročitajte o pokazivačima u Go-u kasnije.

Postoje i neki drugi operatori u Gou. Oni će biti predstavljeni i objašnjeni u drugim Go 101 člancima.

[Konstante i promenljive][0104] | [Sadržaj][00] | [Deklaracije i pozivi funkcija][0106]

[0104]: 0104_Konstante_i_promenljive.md
[00]: 00_Sadrzaj.md
[0106]: 0106_Deklaracije_i_pozivi_funkcija.md
