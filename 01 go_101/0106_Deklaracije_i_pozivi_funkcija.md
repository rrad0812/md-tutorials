
# Deklaracije funkcija i pozivi funkcija

[Uobičajeni operatori][0105] | [Sadržaj][00] | [Paketi i uvoz paketa][0107]

Osim operatorskih operacija predstavljenih u prethodnom članku, funkcionalne operacije su još jedna vrsta popularnih operacija u programiranju. Funkcionalne operacije se često nazivaju pozivima funkcija. Ovaj članak će predstaviti kako se deklarišu funkcije i pozivaju funkcije.

## Deklaracije funkcija

Hajde da pogledamo deklaraciju funkcije.

```go
func SquaresOfSumAndDiff(a int64, b int64) (s int64, d int64) {
    x, y := a + b, a - b
    s = x * x
    d = y * y
    return // <=> return s, d
}
```

Možemo videti da je deklaracija funkcije sastavljena od nekoliko delova. S leva na desno:

- Prvi deo je **func** ključna reč.
- Sledeći deo je **naziv funkcije**, koji mora biti identifikator. Ovde je naziv funkcije *SquareOfSumAndDiff*.
- Treći deo je **lista deklaracija ulaznih parametara**, koja je zatvorena u paru **()**.
- Četvrti deo je **izlazna (ili povratna) lista deklaracija rezultata**. Go funkcije mogu vratiti više rezultata. Za ovaj navedeni primer, lista deklaracija rezultata je takođe zatvorena u par **()**. Međutim, ako je lista prazna ili se sastoji od samo jedne anonimne deklaracije rezultata, onda je par () u listi deklaracija rezultata opcionalan (videti dole za detalje).
- Poslednji deo je **telo funkcije**, koje je zatvoreno u par **{}**. U telu funkcije, ključna reč **return** se koristi za završetak normalnog toka izvršavanja unapred i ulazak u izlaznu fazu (videti odeljak posle sledećeg) poziva funkcije.

U gornjem primeru, svaka deklaracija parametra i rezultata se sastoji od imena i tipa (tip sledi ime). Deklaracije parametara i rezultata možemo posmatrati kao standardne deklaracije promenljivih bez **var** ključnih reči. Gore deklarisana funkcija ima dva parametra, *a* i *b*, i ima dva rezultata, *s* i *d*. Svi tipovi parametara i rezultata su **int64**. Parametri i rezultati se tretiraju kao lokalne promenljive unutar njihovih odgovarajućih tela funkcija.

Imena u listi deklaracija rezultata deklaracije funkcije mogu/moraju biti prisutna ili odsutna u potpunosti. Oba slučaja se često koriste u praksi. Ako je rezultat definisan imenom, onda se rezultat naziva **named result**, u suprotnom se naziva anonimni rezultat.

Kada su svi rezultati u deklaraciji funkcije anonimni, onda, unutar tela odgovarajuće funkcije, **return** ključnu reč mora pratiti niz povratnih vrednosti, pri čemu svaka od njih odgovara deklaraciji rezultata deklaracije funkcije. Na primer, sledeća deklaracija funkcije je ekvivalentna gornjoj.

```go
func SquaresOfSumAndDiff(a int64, b int64) (int64, int64) {
    return (a+b) * (a+b), (a-b) * (a-b)
}
```

U stvari, ako se svi parametri nikada ne koriste unutar odgovarajućeg tela funkcije, imena u listi deklaracija parametara deklaracije funkcije takođe se mogu potpuno izostaviti. Međutim, anonimni parametri se retko koriste u praksi.

Iako izgleda da su parametrična i rezultatska promenljiva deklarisane izvan tela deklaracije funkcije, one se posmatraju kao opšte lokalne promenljive unutar tela funkcije. Razlika je u tome što se lokalne promenljive sa imenima koja nisu prazna, a deklarisana su unutar tela funkcije, uvek moraju koristiti u telu funkcije. Imena koja nisu prazna, lokalnih promenljivih najvišeg nivoa, parametara i rezultata u deklaraciji funkcije ne mogu se duplirati.

Go ne podržava podrazumevane vrednosti parametara. Početna vrednost svakog rezultata je nulta vrednost njegovog tipa. Na primer, sledeća funkcija će uvek ispisati (i vratiti) 0, false.

```go
func f() (x int, y bool) {
    println(x, y) // 0, false
    return
}
```

Ako su delovi tipa nekih uzastopnih parametara u listi deklaracija parametara identični, onda ovi parametri mogu da dele isti deo tipa u listi deklaracija parametara. Isto važi i za liste deklaracija rezultata. Na primer, gornje dve deklaracije funkcija sa imenom SquaresOfSumAndDiffsu ekvivalentne sa

```go
func SquaresOfSumAndDiff(a, b int64) (s, d int64) {
    return (a+b) * (a+b), (a-b) * (a-b)
    // The above line is equivalent
    // to the following line.
    /*
    s = (a+b) * (a+b); d = (a-b) * (a-b); return
    */
}
```

Imajte na umu da čak i ako su oba rezultata imenovana, **return** ključnu reč mogu pratiti vraćene vrednosti.

Ako lista deklaracija rezultata u deklaraciji funkcije sadrži samo jednu anonimnu deklaraciju rezultata, onda lista deklaracija rezultata ne mora biti zatvorena unutar karakteristike (). Ako deklaracija funkcije nema povratnih rezultata, onda se deo liste deklaracije rezultata može potpuno izostaviti. Deo liste deklaracije parametara nikada se ne može izostaviti, čak i ako je broj parametara deklarisane funkcije nula.

Evo još primera deklaracije funkcija.

```go
func CompareLower4bits(m, n uint32) (r bool) {
    r = m&0xF > n&0xF
    return
    // The above two lines is equivalent to
    // the following line.
    /*
    return m&0xF > n&0xF
    */
}

// This function has no parameters. The result
// declaration list is composed of only one
// anonymous result declaration, so it is not
// required to be enclosed within ().
func VersionString() string {
    return "go1.0"
}

// This function has no results. And all of
// its parameters are anonymous, for we don't
// care about them. Its result declaration
// list is blank (so omitted).
func doNothing(string, int) {
}
```

Jedna činjenica koju smo naučili iz ranijih članaka u Go 101 je da mainje funkcija unosa u svakom Go programu deklarisana bez parametara i rezultata.

Imajte na umu da funkcije moraju biti direktno deklarisane na nivou paketa. Drugim rečima, funkcija se ne može deklarisati unutar tela druge funkcije. U kasnijem odeljku ćemo saznati da možemo definisati anonimne funkcije u telima drugih funkcija. Ali anonimne funkcije nisu deklaracije funkcija.

## Pozivi funkcija

Deklarisana funkcija može se pozvati preko svog imena plus liste argumenata. Lista argumenata mora biti zatvorena unutar (). Često ovo nazivamo prosleđivanjem argumenata (ili prosleđivanjem parametara). Svaki argument sa jednom vrednošću odgovara (prosleđuje se) parametru.

**Napomena**: prosleđivanje argumenata je takođe dodeljivanje vrednosti.

Tip argumenta **ne mora** biti identičan odgovarajućem tipu parametra. Jedini zahtev za argument je da mora biti dodeliv (tj. implicitno konvertibilan) odgovarajućem tipu parametra.

Ako funkcija vraća rezultate, onda se svaki njen poziv posmatra kao izraz. Ako vraća više rezultata, onda se svaki njen poziv posmatra kao izraz sa više vrednosti. Izraz sa više vrednosti može biti dodeljen listi odredišnih vrednosti sa istim brojem.

Sledi potpuni primer koji pokazuje kako se pozivaju neke deklarisane funkcije.

```go
package main

func SquaresOfSumAndDiff(a int64, b int64) (int64, int64) {
    return (a+b) * (a+b), (a-b) * (a-b)
}

func CompareLower4bits(m, n uint32) (r bool) {
    r = m&0xF > n&0xF
    return
}

// Initialize a package-level variable
// with a function call.
var v = VersionString()

func main() {
    println(v) // v1.0
    x, y := SquaresOfSumAndDiff(3, 6)
    println(x, y) // 81 9
    b := CompareLower4bits(uint32(x), uint32(y))
    println(b) // false
    // "Go" is deduced as a string,
    // and 1 is deduced as an int32.
    doNothing("Go", 1)
}

func VersionString() string {
    return "v1.0"
}

func doNothing(string, int32) {
}
```

Iz gornjeg primera možemo saznati da funkcija može biti deklarisana pre ili posle bilo kog od njenih poziva.

Pozivi funkcija mogu biti odloženi ili pozvani u novim gorutinama (zelene niti) u Go-u. Molimo vas da pročitate kasniji članak za detalje.

### Izlazak (ili povratak) iz faze poziva funkcije

U jeziku Go, pored normalne faze izvršavanja unapred, poziv funkcije može proći kroz fazu izlaska (takođe nazvanu faza vraćanja). Faza izlaska poziva funkcije počinje kada se pozvana funkcija vrati. Drugim rečima, kada se poziv funkcije vrati, moguće je da još nije izašao.

Detaljnija objašnjenja za izlazak iz faza poziva funkcija mogu se naći u ovom članku .

## Anonimne funkcije

Go podržava anonimne funkcije. Definicija anonimne funkcije je skoro ista kao i deklaracija funkcije, osim što u definiciji anonimne funkcije nema dela sa imenom funkcije.

Anonimna funkcija se može pozvati odmah nakon što je definisana. Primer:

```go
package main

func main() {
    // This anonymous function has no parameters
    // but has two results.
    x, y := func() (int, int) {
        println("This function has no parameters.")
        return 3, 4
    }() // Call it. No arguments are needed.

    // The following anonymous function have no results.

    func(a, b int) {
        // The following line prints: a*a + b*b = 25
        println("a*a + b*b =", a*a + b*b)
    }(x, y) // pass argument x and y to parameter a and b.

    func(x int) {
        // The parameter x shadows the outer x.
        // The following line prints: x*x + y*y = 32
        println("x*x + y*y =", x*x + y*y)
    }(y) // pass argument y to parameter x.

    func() {
        // The following line prints: x*x + y*y = 25
        println("x*x + y*y =", x*x + y*y)
    }() // no arguments are needed.
}
```

Imajte na umu da, pošto je poslednja anonimna funkcija u opsegu važenja promenljivih xi ydeklarisanih gore, ona može direktno da koristi te dve promenljive. Takve funkcije se nazivaju zatvaranja. U stvari, sve prilagođene funkcije u programskom jeziku Go mogu se posmatrati kao zatvaranja. Zbog toga su Go funkcije fleksibilne kao i mnogi dinamički jezici.

Kasnije ćemo saznati da se anonimna funkcija može dodeliti vrednosti funkcije i da se može pozvati u bilo kom trenutku kasnije.

## Ugrađene funkcije

Postoje neke ugrađene funkcije u Gou, na primer, funkcije **println** i **print**. Možemo pozvati ove funkcije bez uvoza bilo kakvih paketa.

Možemo koristiti ugrađene funkcije **real** i **imag** da bismo dobili realni i imaginarni deo kompleksne vrednosti. Možemo koristiti ugrađenu **complex** funkciju da bismo proizveli kompleksnu vrednost. Imajte na umu da ako su bilo koji od argumenata poziva bilo koje od dve funkcije konstante, onda će poziv biti izvršen tokom kompajliranja, a rezultatska vrednost poziva je takođe konstanta. Konkretno, ako je bilo koji od argumenata netipizirana konstanta, onda je rezultatska vrednost takođe netipizirana konstanta. Poziv se posmatra kao konstantni izraz.

Primer:

```go
// c is a untyped complex constant.
const c = complex(1.6, 3.3)

// The results of real(c) and imag(c) are both
// untyped floating-point values. They are both
// deduced as values of type float32 below.
var a, b float32 = real(c), imag(c)

// d is deduced as a typed value of type complex64.
// The results of real(d) and imag(d) are both
// typed values of type float32.
var d = complex(a, b)

// e is deduced as a typed value of type complex128.
// The results of real(e) and imag(e) are both
// typed values of type float64.
var e = c
```

Više ugrađenih funkcija biće predstavljeno kasnije u drugim člancima o Go 101.

## Više o funkcijama

Postoji više o konceptima i detaljima vezanim za funkcije koji nisu obrađeni u ovom članku. Te koncepte i detalje možemo kasnije naučiti u članku o tipovima i vrednostima funkcija.

[Uobičajeni operatori][0105] | [Sadržaj][00] | [Paketi i uvoz paketa][0107]

[0105]: 0105_Uobičajeni_operatori.md
[00]: 00_Sadrzaj.md
[0107]: 0107_Paketi_i_uvoz_paketa.md
