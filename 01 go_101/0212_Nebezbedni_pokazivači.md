
# Pokazivači nebezbedni po tipu

[Ugrađivanje tipa][0211] | [Sadržaj][00] | [Generici][0213]

U poređenju sa C pokazivačima, postoje mnoga ograničenja za pokazivače na Go jeziku. Na primer, pokazivači na Go jeziku ne mogu da učestvuju u aritmetičkim operacijama, a za dva proizvoljna tipa pokazivača, vrlo je moguće da se njihove vrednosti ne mogu konvertovati jedna u drugu.

Pokazivači objašnjeni u tom članku se zapravo nazivaju pokazivači bezbedni po tipu. Iako nam ograničenja u vezi sa pokazivačima bezbednim po tipu zaista omogućavaju da lako pišemo bezbedan Go kod, ona takođe stvaraju neke prepreke za pisanje efikasnog koda u nekim scenarijima.

U stvari, Go takođe podržava pokazivače nesigurne po tipu, koji su pokazivači bez ograničenja namenjenih bezbednim pokazivačima. Pokazivači nebezbedni po tipu se takođe nazivaju nebezbedni pokazivači u Gou. Nebezbedni pokazivači u Gou su veoma slični C pokazivačima, moćni su, ali i opasni. U nekim slučajevima, možemo napisati efikasniji kod uz pomoć Nebezbedni pokazivača. S druge strane, korišćenjem nebezbednih pokazivača, lako je napisati loš kod koji je previše suptilan da bi se otkrio na vreme.

Još jedan veliki rizik korišćenja nebezbednih pokazivača dolazi od činjenice da nebezbedni mehanizam nije zaštićen smernicama za kompatibilnost sa Go 1. Kod koji zavisi od nebezbednih pokazivača i danas funkcioniše mogao bi da se pokvari od kasnije verzije Go-a.

Ako zaista želite efikasna poboljšanja koda korišćenjem nebezbednih pokazivača iz bilo kog razloga, ne samo da treba da znate gore pomenute rizike, već i da pratite uputstva napisana u zvaničnoj Go dokumentaciji i jasno razumete efekat svake upotrebe nebezbednog pokazivača, kako biste mogli da pišete bezbedan Go kod sa nebezbednim pokazivačima.

## O unsafe standardnom paketu

Go pruža posebnu vrstu tipova za nebezbedne pokazivače. Moramo da uvezemo standardni **unsafe** paket da bismo koristili nebezbedne pokazivače. **unsafe.PointerTip** je definisan kao:

```go
type Pointer *ArbitraryType
```

Sigurno, to nije uobičajena definicija tipa. Ovde se sa **\*ArbitraryType** samo nagoveštava da **unsafe.Pointer** se vrednost može konvertovati u bilo koju bezbednu vrednost pokazivača u programskom jeziku Go (i obrnuto). Drugim rečima, **unsafe.Pointer** je kao **void\*** u C jeziku.

Nebezbedni pokazivači označavaju tipove čiji su osnovni tipovi **unsafe.Pointer**.

Nulte vrednosti nebezbednih pokazivača su takođe predstavljene unapred deklarisanim identifikatorom **nil**.

Pre Go 1.17, unsafestandardni paket je već pružao tri funkcije.

- func Alignof(variable ArbitraryType) uintptr, koji se koristi za dobijanje poravnanja adrese vrednosti. Imajte u vidu da poravnanja za vrednosti strukturnih polja i vrednosti koje nisu polja istog tipa mogu biti različita, mada su za standardni Go kompajler uvek ista. Za gccgo kompajler mogu biti različita.

- func Offsetof(selector ArbitraryType) uintptr, koji se koristi za dobijanje pomeranja adrese polja u vrednosti strukture. Pomeranje je relativno u odnosu na adresu vrednosti strukture. Vraćeni rezultati treba uvek da budu isti za isto odgovarajuće polje vrednosti istog tipa strukture u istom programu.
  
- func Sizeof(variable ArbitraryType) uintptr, koji se koristi za dobijanje veličine vrednosti (tj. veličine tipa vrednosti). Vraćeni rezultati treba uvek da budu isti za sve vrednosti istog tipa u istom programu.

Napomena,

- Tipovi povratnih rezultata sve tri funkcije su uintptr. U nastavku ćemo saznati da se vrednosti uintptr mogu konvertovati u nebezbedne pokazivače (i obrnuto).

- Iako su povratni rezultati poziva bilo koje od tri funkcije konzistentni u istom programu, mogu biti različiti ukršteni operativni sistemi, ukrštene arhitekture, ukršteni kompajleri i ukrštene verzije kompajlera.

- Pozivi ovih triju funkcija se uvek evaluiraju tokom kompajliranja. Rezultati evaluacije su tipizirane konstante tipa uintptr.

- Argument koji se prosleđuje pozivu funkcije unsafe.Offsetofmora biti oblika selektora polja strukture value.field. Selektor može da označava ugrađeno polje, ali polje mora biti dostupno bez implicitnih indirekcija pokazivača.

Primer korišćenja tri funkcije.

```go
package main

import "fmt"
import "unsafe"

func main() {
    var x struct {
        a int64
        b bool
        c string
    }
    const M, N = unsafe.Sizeof(x.c), unsafe.Sizeof(x)
    fmt.Println(M, N) // 16 32

    fmt.Println(unsafe.Alignof(x.a)) // 8
    fmt.Println(unsafe.Alignof(x.b)) // 1
    fmt.Println(unsafe.Alignof(x.c)) // 8

    fmt.Println(unsafe.Offsetof(x.a)) // 0
    fmt.Println(unsafe.Offsetof(x.b)) // 8
    fmt.Println(unsafe.Offsetof(x.c)) // 16
}
```

Primer koji ilustruje poslednju gore pomenutu napomenu.

```go
package main

import "fmt"
import "unsafe"

func main() {
    type T struct {
        c string
    }
    type S struct {
        b bool
    }
    var x struct {
        a int64
        *S
        T
    }

    fmt.Println(unsafe.Offsetof(x.a)) // 0
    
    fmt.Println(unsafe.Offsetof(x.S)) // 8
    fmt.Println(unsafe.Offsetof(x.T)) // 16
    
    // This line compiles, for c can be reached
    // without implicit pointer indirections.
    fmt.Println(unsafe.Offsetof(x.c)) // 16
    
    // This line doesn't compile, for b must be
    // reached with the implicit pointer field S.
    //fmt.Println(unsafe.Offsetof(x.b)) // error
    
    // This line compiles. However, it prints
    // the offset of field b in the value x.S.
    fmt.Println(unsafe.Offsetof(x.S.b)) // 0
}
```

Molimo vas da imate u vidu da su rezultati štampanja prikazani u komentarima za standardni Go kompajler verzije 1.25.n na Linux AMD64 arhitekturi.

Tri funkcije koje se nalaze u **unsafe** paketu ne izgledaju mnogo opasno. Potpisi ovih funkcija je gotovo nemoguće promeniti u budućim verzijama Go 1. Rob Pajk je čak i predložio da se tri funkcije premeste negde drugde. Većina nebezbednosti paketa **unsafe** dolazi od nebezbednih pokazivača. Oni su jednako opasni kao i C pokazivači, što bezbedni pokazivači Go-a uvek pokušavaju da izbegnu.

Go 1.17 uvodi jedan novi tip i dve nove funkcije u **unsafe** paket. Novi tip je **IntegerType**, Sledi njegova definicija. Ovaj tip ne označava određeni tip. On samo predstavlja bilo koji proizvoljni celobrojni tip. Možemo ga posmatrati kao generički tip.

```go
type IntegerType int
```

Dve funkcije predstavljene u Go 1.17 su:

- **func Add(ptr Pointer, len IntegerType) Pointer**. Ova funkcija dodaje pomak adresi koju predstavlja nebezbedni pokazivač i vraća novi nebezbedni pokazivač koji predstavlja novu adresu. Ova funkcija delimično pokriva upotrebu dole predstavljenog obrasca korišćenja nebezbednog pokazivača 3.

- **func Slice(ptr \*ArbitraryType, len IntegerType) []ArbitraryType** Ova funkcija se koristi za izvođenje sloja navedene dužine iz bezbednog pokazivača, gde ArbitraryTypeje tip elementa rezultujućeg sloja.

Go 1.20 je dodatno uveo još tri funkcije:

- **func String(ptr \*byte, len IntegerType) string** Ova funkcija se koristi za izvođenje stringa navedene dužine iz bezbednog bytepokazivača.

- **func StringData(str string) \*byte** Ova funkcija se koristi za dobijanje pokazivača na osnovni bajtni niz stringa. Imajte u vidu da ovoj funkciji ne prosleđujete prazne stringove kao argumente.

- **func SliceData(slice []ArbitraryType) \*ArbitraryType** Ova funkcija se koristi za dobijanje pokazivača na osnovni niz elemenata segmenta.

Ove funkcije uvedene od verzije Go 1.17 nose određenu opasnost. Potrebno ih je koristiti sa oprezom. Sledi primer koji koristi dve funkcije uvedene u Go 1.17.

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    a := [16]int{3: 3, 9: 9, 11: 11}
    fmt.Println(a)
    eleSize := int(unsafe.Sizeof(a[0]))
    p9 := &a[9]
    up9 := unsafe.Pointer(p9)
    p3 := (*int)(unsafe.Add(up9, -6 * eleSize))
    fmt.Println(*p3) // 3
    s := unsafe.Slice(p9, 5)[:3]
    fmt.Println(s) // [9 0 11]
    fmt.Println(len(s), cap(s)) // 3 5

    t := unsafe.Slice((*int)(nil), 0)
    fmt.Println(t == nil) // true

    // The following two calls are dangerous.
    // They make the results reference
    // unknown memory blocks.
    _ = unsafe.Add(up9, 7 * eleSize)
    _ = unsafe.Slice(p9, 8)
}
```

Sledeće dve funkcije mogu se koristiti za konverzije između stringova i bajtnih isečaka, na načine koji nisu sigurni po tipu. U poređenju sa načinima koji nisu sigurni po tipu, načini koji nisu sigurni po tipu ne dupliraju osnovne bajtne sekvence stringova i bajtnih isečaka, pa su efikasniji.

```go
import "unsafe"

func String2ByteSlice(str string) []byte {
    if str == "" {
        return nil
    }
    return unsafe.Slice(unsafe.StringData(str), len(str))
}

func ByteSlice2String(bs []byte) string {
    if len(bs) == 0 {
        return ""
    }
    return unsafe.String(unsafe.SliceData(bs), len(bs))
}
```

## Pravila konverzije povezana sa nebezbednim pokazivačima

Trenutno (Go 1.25), Go kompajleri dozvoljavaju sledeće eksplicitne konverzije.

- Bezbedan pokazivač može biti eksplicitno konvertovan u nebezbedan pokazivač i obrnuto.

- Vrednost uintptr može se eksplicitno konvertovati u nesigurni pokazivač i obrnuto. Ali imajte na umu da unsafe.Pointer vrednosti nil ne treba konvertovati u uintptr i nazad aritmetikom.

Korišćenjem ovih konverzija, možemo konvertovati vrednost bezbednog pokazivača u proizvoljni tip bezbednog pokazivača.

Međutim, iako su sve ove konverzije legalne u vreme kompajliranja, nisu sve validne (bezbedne) u vreme izvršavanja. Ove konverzije narušavaju bezbednost memorije koju ceo Go sistem tipova (osim nebezbednog dela) pokušava da održi. Moramo pratiti uputstva navedena u kasnijem odeljku ispod da bismo napisali validan Go kod sa nebezbednim pokazivačima.

## Neke činjenice o Gou koje bi trebalo da znamo

Pre nego što uvedemo validne obrasce korišćenja nebezbednih pokazivača, potrebno je da znamo neke činjenice o Go-u.

- **Činjenica 1**: nebezbedni pokazivači su pokazivači, a vrednosti **uintptr** su celi brojevi

Svaki od ne-nil bezbednih i nebezbednih pokazivača upućuje na drugu vrednost. Međutim, vrednosti **uintptr**-a ne upućuju ni na kakve vrednosti, već su samo obični celi brojevi, mada često svaka od njih čuva ceo broj koji se može koristiti za predstavljanje memorijske adrese.

Go je jezik koji podržava automatsko sakupljanje smeća. Kada se Go program pokreće, Go runtime će proveriti koji memorijski blokovi se više ne koriste od strane bilo koje vrednosti i povremeno će sakupljati memoriju dodeljenu za te neiskorišćene blokove. Pokazivači igraju važnu ulogu u procesu provere. Ako je memorijski blok nedostupan (na njega se ne može referencirati) od strane bilo koje vrednosti koja je još uvek u upotrebi, onda Go runtime smatra da je to neiskorišćena vrednost i da se može bezbedno sakupiti smeće.

Pošto su vrednosti **uintptr** celi brojevi, mogu učestvovati u aritmetičkim operacijama.

Primer u sledećem pododeljku prikazuje razlike između pokazivača i vrednosti **uintptr**.

**Činjenica 2**: neiskorišćeni blokovi memorije mogu se sakupiti u bilo kom trenutku

Tokom izvršavanja, sakupljač smeća može da se pokrene u neizvesno vreme, a svaki proces sakupljanja smeća može da traje neizvesno vreme. Dakle, kada blok memorije postane neiskorišćen, može biti sakupljen u neizvesno vreme.

Na primer:

```go
import "unsafe"

// Assume createInt will not be inlined.
//go:noinline
func createInt() *int {
    return new(int)
}

func foo() {
    p0, y, z := createInt(), createInt(), createInt()
    var p1 = unsafe.Pointer(y)
    var p2 = uintptr(unsafe.Pointer(z))

    // At the time, even if the address of the int
    // value referenced by z is still stored in p2,
    // the int value has already become unused, so
    // garbage collector can collect the memory
    // allocated for it now. On the other hand, the
    // int values referenced by p0 and p1 are still
    // in use.

    // uintptr can participate arithmetic operations.
    p2 += 2; p2--; p2--

    *p0 = 1                         // okay
    *(*int)(p1) = 2                 // okay
    *(*int)(unsafe.Pointer(p2)) = 3 // dangerous!
}
```

U gornjem primeru, činjenica da je p2 vrednost još uvek u upotrebi ne može garantovati da memorijski blok koji je ikada sadržao int vrednost na koju se poziva nije još uvek sakupljen kao smeće. Drugim rečima, kada *(*int)(unsafe.Pointer(p2)) = 3 se izvrši, memorijski blok može biti sakupljen, ali i ne. Opasno je dereferencirati adresu sačuvanu u vrednosti p2 na int vrednost, jer je moguće da je memorijski blok već preraspoređen za drugu vrednost (čak i za drugi program).

**Činjenica 3**: adrese nekih vrednosti mogu se promeniti tokom izvršavanja

Molimo vas da pročitate članak o memorijskim blokovima za detalje (pogledajte kraj odeljka sa hiperlinkom). Ovde bi trebalo samo da znamo da kada se veličina steka gorutine promeni, memorijski blokovi dodeljeni na steku će biti pomereni. Drugim rečima, adrese vrednosti smeštenih na ovim memorijskim blokovima će se promeniti.

**Činjenica 4**: opseg trajanja vrednosti tokom izvršavanja možda nije toliko veliki koliko izgleda u kodu

U sledećem primeru, činjenica da tje vrednost još uvek u upotrebi ne može garantovati da su vrednosti na koje se poziva vrednost t.yjoš uvek u upotrebi.

```go
type T struct {
    x int
    y *[1<<23]byte
}

func bar() {
    t := T{y: new([1<<23]byte)}
    p := uintptr(unsafe.Pointer(&t.y[0]))

    ... // use T.x and T.y

    // A smart compiler can detect that the value
    // t.y will never be used again and think the
    // memory block hosting t.y can be collected now.

    // Using *(*byte)(unsafe.Pointer(p))) is
    // dangerous here.

    // Continue using value t, but only use its x field.
    println(t.x)
}
```

**Činjenica 5**: \*unsafe.Pointer je opšti bezbedni tip pokazivača

Da, \*unsafe.Pointer je bezbedan tip pokazivača. NJegov osnovni tip je **unsafe.Pointer**. Pošto je bezbedan pokazivač, prema pravilima konverzije navedenim gore, može se konvertovati u **unsafe.Pointer** tip i obrnuto.

Na primer:

```go
package main

import "unsafe"

func main() {
    x := 123                // of type int
    p := unsafe.Pointer(&x) // of type unsafe.Pointer
    pp := &p                // of type *unsafe.Pointer
    p = unsafe.Pointer(pp)
    pp = (*unsafe.Pointer)(p)
}
```

## Kako pravilno koristiti nebezbedne pokazivače?

Standardna **unsafe** dokumentacija paketa navodi šest nebezbednih obrazaca korišćenja pokazivača. U nastavku ćemo ih predstaviti i objasniti jedan po jedan.

**Obrazac 1**: konvertujte **\*T1** vrednost u nebezbedni pokazivač, a zatim konvertujte vrednost nebezbednog pokazivača u **\*T2**.

Kao što je gore pomenuto, korišćenjem gore navedenih pravila konverzije nebezbednih pokazivača, možemo konvertovati vrednost **\*T1** u tip **\*T2**, gde su **T1** i **T2** dva proizvoljna tipa. Međutim, takve konverzije treba da radimo samo ako veličina **T1** nije manja od **T2**, i samo ako su konverzije značajne.

Kao rezultat toga, možemo postići i konverzije između tipa **T1** i **T2** koristeći ovaj obrazac.

Jedan primer je **math.Float64bits** funkcija, koja konvertuje **float64** vrednost u **uint64** vrednost, bez promene bilo kog bita u **float64** vrednosti. **math.Float64frombits** funkcija vrši obrnute konverzije.

```go
func Float64bits(f float64) uint64 {
    return *(*uint64)(unsafe.Pointer(&f))
}

func Float64frombits(b uint64) float64 {
    return *(*float64)(unsafe.Pointer(&b))
}
```

Imajte na umu da se povratni rezultat **math.Float64bits** (aFloat64)poziva funkcije razlikuje od rezultata eksplicitne konverzije **uint64(aFloat64)**.

U sledećem primeru, koristimo ovaj obrazac za konvertovanje **[]MyString** isečka u tip **[]string** i obrnuto. Rezultujući isečak i originalni isečak dele osnovne elemente. Takve konverzije su nemoguće na bezbedne načine,

```go
package main

import (
    "fmt"
    "unsafe"
)

func main() {
    type MyString string
    ms := []MyString{"C", "C++", "Go"}
    fmt.Printf("%s\n", ms)  // [C C++ Go]
    // ss := ([]string)(ms) // compiling error
    ss := *(*[]string)(unsafe.Pointer(&ms))
    ss[1] = "Zig"
    fmt.Printf("%s\n", ms) // [C Zig Go]
    // ms = []MyString(ss) // compiling error
    ms = *(*[]MyString)(unsafe.Pointer(&ss))
    
    // Since Go 1.17, we may also use the
    // unsafe.Slice function to do the conversions.
    ss = unsafe.Slice((*string)(&ms[0]), len(ms))
    ms = unsafe.Slice((*MyString)(&ss[0]), len(ms))
}
```

Inače, od Go 1.17, možemo koristiti i funkciju **unsafe.Slice** za konverzije:

```go
func main() {
    ...
    
    ss = unsafe.Slice((*string)(&ms[0]), len(ms))
    ms = unsafe.Slice((*MyString)(&ss[0]), len(ss))
}
```

Praksa korišćenja šablona je konvertovanje bajtnog isečka, koji se neće koristiti nakon konverzije, u string, kao što je prikazano u sledećem kodu. U ovoj konverziji se izbegava dupliranje osnovnog bajtnog niza.

```go
func ByteSlice2String(bs []byte) string {
    return *(*string)(unsafe.Pointer(&bs))
}
```

Ovo je implementacija koju je usvojila metoda **String** tipa **Builder** podržanog od Go 1.10 u **strings** standardnom paketu. Veličina bajtnog isečka je veća od stringa, a njihove unutrašnje strukture su slične, tako da je konverzija validna (za glavne Go kompajlere). Međutim, uprkos tome što se implementacija sada može bezbedno koristiti u standardnim paketima, ne preporučuje se njena upotreba u opštem korisničkom kodu. Od Go 1.20, u opštem korisničkom kodu, trebalo bi da pokušamo da koristimo implementaciju koja koristi funkciju **unsafe.String** pomenutu gore u ovom članku.

Suprotno, pretvaranje stringa u bajtni isečak na sličan način, nije validno, jer je veličina stringa manja od bajtnog isečka.

```go
func String2ByteSlice(s string) []byte {
    return *(*[]byte)(unsafe.Pointer(&s)) // dangerous!
}
```

U odeljku 6 obrasca ispod, predstavljena je validna implementacija za obavljanje istog posla.

**Napomena**: kada koristite upravo predstavljeni nebezbedni način za konvertovanje bajtnog isečka u string, vodite računa da ne izmenite bajtove u bajtnom isečku ako rezultujući string i dalje postoji.

**Obrazac 2**: konvertujte nebezbedni pokazivač u uintptr, a zatim koristite vrednost uintptr.

Ovaj obrazac nije baš koristan. Obično ispisujemo rezultujuće vrednosti uintptr da bismo proverili memorijske adrese sačuvane u njima. Međutim, postoje i drugi, bezbedni i manje opširni načini za ovaj posao. Dakle, ovaj obrazac nije baš koristan.

Primer:

```go
package main

import "fmt"
import "unsafe"

func main() {
    type T struct{a int}
    var t T
    fmt.Printf("%p\n", &t)                          // 0xc6233120a8
    println(&t)                                     // 0xc6233120a8
    fmt.Printf("%x\n", uintptr(unsafe.Pointer(&t))) // c6233120a8
}
```

Izlazne adrese mogu biti različite za svako pokretanje.

**Obrazac 3**: konvertovanje nebezbednog pokazivača u **uintptr**, izvršavanje aritmetičkih operacija sa vrednošću uintptr, a zatim njeno vraćanje unazad.

U ovom obrascu, rezultujući nebezbedni pokazivač mora nastaviti da pokazuje na originalno dodeljeni blok memorije. Na primer:

```go
package main

import "fmt"
import "unsafe"

type T struct {
    x bool
    y [3]int16
}

const N = unsafe.Offsetof(T{}.y)
const M = unsafe.Sizeof(T{}.y[0])

func main() {
    t := T{y: [3]int16{123, 456, 789}}
    p := unsafe.Pointer(&t)
    // "uintptr(p)+N+M+M" is the address of t.y[2].
    ty2 := (*int16)(unsafe.Pointer(uintptr(p)+N+M+M))
    fmt.Println(*ty2) // 789
}
```

U stvari, od verzije Go 1.17, preporučljivije je koristiti gore predstavljenu unsafe.Addfunkciju za izvođenje takvih operacija pomeranja adrese.

Imajte u vidu da u ovom navedenom primeru konverzija unsafe.Pointer(uintptr(p) + N + M + M)ne bi trebalo da bude podeljena u dva reda, kao što je prikazano u sledećem kodu. Molimo vas da pročitate komentare u kodu da biste saznali razlog.

```go
func main() {
    t := T{y: [3]int16{123, 456, 789}}
    p := unsafe.Pointer(&t)
    // ty2 := (*int16)(unsafe.Pointer(uintptr(p)+N+M+M))
    addr := uintptr(p) + N + M + M
    
    // ... (some other operations)
    
    // Now the t value becomes unused, its memory may be
    // garbage collected at this time. So the following
    // use of the address of t.y[2] may become invalid
    // and dangerous! 
    // Another potential danger is, if some operations
    // make the stack grow or shrink here, then the
    // address of t might change, so that the address
    // saved in addr will become invalid (fact 3).
    ty2 := (*int16)(unsafe.Pointer(addr))
    fmt.Println(*ty2)
}
```

Takve greške su veoma suptilne i teško ih je otkriti, zbog čega je upotreba nebezbednih pokazivača opasna.

Međuvrednost uintptr može takođe učestvovati u &^operacijama brisanja po bitovima radi poravnanja adrese, sve dok rezultujući nebezbedni pokazivač i originalna vrednost ukazuju na isti dodeljeni blok memorije.

Još jedan detalj koji treba napomenuti jeste da se ne preporučuje čuvanje krajnje granice memorijskog bloka u pokazivaču (bilo bezbednom ili nebezbednom). Ovo će sprečiti da drugi memorijski blok koji odmah sledi prethodni memorijski blok bude „sakupljan smećem“ ili da se program sruši ako ta granična adresa nije važeća za bilo koji dodeljeni memorijski blok (u zavisnosti od implementacije kompajlera). Molimo vas da pročitate ovu stavku često postavljanih pitanja da biste dobili više objašnjenja.

**Obrazac 4**: konvertovanje nebezbednih pokazivača u **uintptr** vrednosti kao argumente **syscall.Syscall** poziva.

Iz objašnjenja za poslednji obrazac, znamo da je sledeća funkcija opasna.

```go
// Assume this function will not inlined.
func DoSomething(addr uintptr) {
    // read or write values at the passed address ...
}
```

Razlog zašto je gore navedena funkcija opasna je taj što sama funkcija ne može garantovati da blok memorije na adresi prosledjenog argumenta još nije sakupljen kao smeće. Ako je blok memorije sakupljen ili je preraspoređen za druge vrednosti, onda su operacije izvršene u telu funkcije opasne.

Međutim, prototip funkcije Syscallu syscallstandardnom paketu je sledeći

```go
func Syscall(trap, a1, a2, a3 uintptr) (r1, r2 uintptr, err Errno)
```

Kako ova funkcija garantuje da memorijski blokovi na prosleđenim adresama a1još uvek a2nisu a3sakupljeni „smeće“ unutar same funkcije? Funkcija to ne može garantovati. U stvari, kompajleri će to garantovati. To je privilegija poziva syscall.Syscallsličnih funkcija.

Možemo pretpostaviti da će kompajleri automatski umetnuti neke instrukcije za svaki od nebezbednih argumenata pokazivača koji se konvertuju u uintptr, kao što je treći argument u sledećem syscall.Syscallpozivu, kako bi sprečili da se blok memorije na koji se poziva taj argument sakuplja „smeće“ ili premešta.

Imajte na umu da je pre verzije Go 1.15 bilo u redu da izrazi za konverziju uintptr(anUnsafePointer)deluju kao podizrazi argumenta o kojima se govori. Od verzije Go 1.15, zahtev postaje malo strožiji: argumenti o kojima se govori moraju biti predstavljeni tačno u datom uintptr(anUnsafePointer)obliku.

Sledeći poziv je bezbedan:

```go
syscall.Syscall(SYS_READ, uintptr(fd),
            uintptr(unsafe.Pointer(p)), uintptr(n))

Ali sledeći pozivi su opasni:

u := uintptr(unsafe.Pointer(p))
// At this time, the value referenced by p might
// have become unused and been collected already,
// or the address of the value has changed.
syscall.Syscall(SYS_READ, uintptr(fd), u, uintptr(n))

// Arguments must be in the "uintptr(anUnsafePointer)"
// form. In fact, the call was safe before Go 1.15.
// But Go 1.15 changes the rule a bit.
syscall.Syscall(SYS_XXX, uintptr(uintptr(fd)),
            uint(uintptr(unsafe.Pointer(p))), uintptr(n))
```

**Napomena**: ovaj obrazac se takođe primenjuje na metode **syscall.Proc.Call** i **syscall.LazyProc.Call** na Windows-u.

Ponovo, nikada ne koristite ovaj obrazac kada pozivate druge funkcije.

**Obrazac 5**: konvertovanje uintptrrezultata poziva metode **reflect.Value.Pointer** ili **reflect.Value.UnsafeAddr** u nebezbedan pokazivač

Metode **Pointer** i **UnsafeAddr** tipa **Value** u **reflect** standardnom paketu vraćaju rezultat tipa **uintptr** umesto **unsafe.Pointer**. Ovo je namerna konstrukcija, koja služi da se izbegne konvertovanje rezultata poziva (dvema metodama) u bilo koje bezbedne tipove pokazivača bez uvoza **unsafe** standardnog paketa.

Dizajn zahteva da se povratni rezultat poziva bilo koje od dve metode mora konvertovati u nebezbedan pokazivač odmah nakon poziva. U suprotnom, postojaće mali vremenski prozor u kome bi memorijski blok dodeljen na adresi sačuvanoj u rezultatu mogao da izgubi sve reference i bude sakupljen kao smeće.

Na primer, sledeći poziv je bezbedan.

```go
p := (*int)(unsafe.Pointer(reflect.ValueOf(new(int)).Pointer()))

S druge strane, sledeći poziv je opasan.

u := reflect.ValueOf(new(int)).Pointer()
// At this moment, the memory block at the address
// stored in u might have been collected already.
p := (*int)(unsafe.Pointer(u))
```

Imajte na umu da Go 1.19 uvodi novu metodu, reflect.Value.UnsafePointer(), koja vraća unsafe.Pointervrednost i koja je poželjnija od dve upravo pomenute funkcije. To znači da se stari namerni dizajn sada smatra lošim.

Obrazac 6: konvertovanje polja tipa a reflect.SliceHeader.Dataili reflect.StringHeader.Datau nebezbedan pokazivač i obrnuto.

Iz istog razloga pomenutog za poslednji pododeljak, Datapolja tipa strukture SliceHeaderi StringHeaderu reflectstandardnom paketu su deklarisana sa tipom uintptrumesto unsafe.Pointer.

Možemo konvertovati pokazivač stringa u *reflect.StringHeadervrednost pokazivača, tako da možemo manipulisati unutrašnjošću stringa. Isto tako, možemo konvertovati pokazivač segmenta u **\*reflect.SliceHeader** vrednost pokazivača, tako da možemo manipulisati unutrašnjošću segmenta.

Primer korišćenja reflect.StringHeader:

```go
package main

import "fmt"
import "unsafe"
import "reflect"

func main() {
    a := [...]byte{'G', 'o', 'l', 'a', 'n', 'g'}
    s := "Java"
    hdr := (*reflect.StringHeader)(unsafe.Pointer(&s))
    hdr.Data = uintptr(unsafe.Pointer(&a))
    hdr.Len = len(a)
    fmt.Println(s) // Golang
    // Now s and a share the same byte sequence, which
    // makes the bytes in the string s become mutable.
    a[2], a[3], a[4], a[5] = 'o', 'g', 'l', 'e'
    fmt.Println(s) // Google
}
```

Primer korišćenja **reflect.SliceHeader**:

```go
package main

import (
    "fmt"
    "unsafe"
    "reflect"
)

func main() {
    a := [6]byte{'G', 'o', '1', '0', '1'}
    bs := []byte("Golang")
    hdr := (*reflect.SliceHeader)(unsafe.Pointer(&bs))
    hdr.Data = uintptr(unsafe.Pointer(&a))

    hdr.Len = 2
    hdr.Cap = len(a)
    fmt.Printf("%s\n", bs) // Go
    bs = bs[:cap(bs)]
    fmt.Printf("%s\n", bs) // Go101
}
```

Generalno, trebalo bi da dobijemo *reflect.StringHeadervrednost pokazivača samo iz stvarnog (već postojećeg) stringa ili da dobijemo **\*reflect.SliceHeader** vrednost pokazivača iz stvarnog (već postojećeg) segmenta. Ne bi trebalo da radimo suprotno, kao što je kreiranje stringa iz novog dodeljenog stringa StringHeaderili kreiranje segmenta iz novog dodeljenog stringa SliceHeader. Na primer, sledeći kod je opasan.

```go
var hdr reflect.StringHeader
hdr.Data = uintptr(unsafe.Pointer(new([5]byte)))
// Now the just allocated byte array has lose all
// references and it can be garbage collected now.
hdr.Len = 5
s := *(*string)(unsafe.Pointer(&hdr)) // dangerous!
```

Sledi primer koji pokazuje kako konvertovati string u bajtni segment, koristeći nebezbedan način. Za razliku od bezbedne konverzije iz stringa u bajtni segment, nebezbedan način ne dodeljuje novi osnovni bajtni niz za rezultujući segment u svakoj konverziji.

```go
package main

import (
    "fmt"
    "reflect"
    "strings"
    "unsafe"
)

func String2ByteSlice(str string) (bs []byte) {
    strHdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
    sliceHdr := (*reflect.SliceHeader)(unsafe.Pointer(&bs))
    sliceHdr.Data = strHdr.Data
    sliceHdr.Cap = strHdr.Len
    sliceHdr.Len = strHdr.Len
    return
}

func main() {
    // str := "Golang"
    // For the official standard compiler, the above
    // line will make the bytes in str allocated on
    // an immutable memory zone.
    // So we use the following line instead.
    str := strings.Join([]string{"Go", "land"}, "")
    s := String2ByteSlice(str)
    fmt.Printf("%s\n", s) // Goland
    s[5] = 'g'
    fmt.Println(str) // Golang
}
```

Imajte na umu da kada koristite upravo predstavljeni nebezbedni način za konvertovanje stringa u bajtni segment, vodite računa da ne modifikujete bajtove u rezultujućem bajtnom segmentu ako string i dalje preživi (radi demonstracije, gornji primer krši ovaj princip).

Takođe je moguće konvertovati bajtni isečak u string na sličan način, što je malo bezbednije (ali malo sporije) od načina prikazanog u obrascu 1.

```go
func ByteSlice2String(bs []byte) (str string) {
    sliceHdr := (*reflect.SliceHeader)(unsafe.Pointer(&bs))
    strHdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
    strHdr.Data = sliceHdr.Data
    strHdr.Len = sliceHdr.Len
    return
}
```

Slično tome, vodite računa da ne menjate bajtove u argumentu bajt isečka ako rezultujući string i dalje postoji.

Uzgred, pogledajmo loš primer koji krši princip obrasca 3 (primer je pozajmljen iz jednog komentara na Slack-u koji je objavio Brajan K. Mils):

```go
package main

import (
    "fmt"
    "reflect"
    "unsafe"
)

func Example_Bad() *byte {
    var str = "godoc"
    hdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
    pbyte := (*byte)(unsafe.Pointer(hdr.Data + 2))
    return pbyte // *pbyte == 'd'
}

func main() {
    fmt.Println(string(*Example_Bad()))
}

Dve ispravne implementacije:

func Example_Good1() *byte {
    var str = "godoc"
    hdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
    pbyte := (*byte)(unsafe.Pointer(
        uintptr(unsafe.Pointer(hdr.Data)) + 2))
    return pbyte
}

// Works since Go 1.17.
func Example_Good2() *byte {
    var str = "godoc"
    hdr := (*reflect.StringHeader)(unsafe.Pointer(&str))
    pbyte := (*byte)(unsafe.Add(unsafe.Pointer(hdr.Data), 2))
    return pbyte
}
```

Teško? Da.

Dokumentacija tipova **SliceHeader** i **StringHeader** u **reflect** standardnom paketu je slična po tome što kaže da se reprezentacije dva tipa struktura mogu promeniti u kasnijim izdanjima. Dakle, gore navedeni validni primeri koji koriste dva tipa mogu postati nevažeći čak i ako pravila o nesigurnosti ostanu nepromenjena. Srećom, trenutno (Go 1.25), dva dostupna glavna Go kompajlera (standardni Go kompajler i gccgo kompajler) prepoznaju reprezentacije dva tipa deklarisana u **reflect** standardnom paketu.

Tim za razvoj osnovnog jezika Go je takođe shvatio da su ova dva tipa nezgodna i sklona greškama, tako da se ova dva tipa više ne preporučuju od verzije Go 1.20 i da su zastareli od verzije Go 1.21. Umesto toga, trebalo bi da pokušamo da koristimo funkcije unsafe.String, unsafe.StringDatai opisane ranije u ovom članku. unsafe.Sliceunsafe.SliceData

## Završne reči

Iz gore navedenog sadržaja znamo da nam, u nekim slučajevima, nesigurni mehanizam može pomoći da napišemo efikasniji Go kod. Međutim, veoma je lako uvesti neke suptilne greške koje imaju veoma malu verovatnoću da se pojave kada se koristi nesigurni mehanizam. Program sa ovim greškama može dobro da radi dugo vremena, ali se iznenada ponašati nenormalno, pa čak i da se sruši kasnije. Takve greške je veoma teško otkriti i otkloniti.

Nebezbedan mehanizam treba da koristimo samo kada je to neophodno, i moramo ga koristiti sa izuzetnim oprezom. Posebno treba da sledimo gore opisana uputstva.

I ponovo, trebalo bi da budemo svesni da se gore predstavljeni nebezbedni mehanizam može promeniti, pa čak i postati potpuno nevažeći u kasnijim verzijama Go-a, iako nema dokaza da će se to uskoro dogoditi. Ako se pravila nebezbednog mehanizma promene, gore predstavljeni validni obrasci korišćenja nebezbednih pokazivača mogu postati nevažeći. Zato vas molimo da omogućite lakoću vraćanja na bezbedne implementacije za vaš kod u zavisnosti od nebezbednog mehanizma.

Na kraju, vredi napomenuti da -gcflags=all=-d=checkptrje opcija dinamičke analize kompajlera podržana od Go Toolchain 1.14 (preporučuje se korišćenje ove opcije na Windows-u sa Go Toolchain 1.15+). Kada se koristi ova opcija, neke (ali ne sve) pogrešne nebezbedne upotrebe pokazivača biće otkrivene tokom izvršavanja. Kada se takva nepravilna upotreba otkrije, doći će do panike. Hvala Metjuu Dempskom na implementaciji ove sjajne funkcije!

[Ugrađivanje tipa][0211] | [Sadržaj][00] | [Generici][0213]

[0211]: 0211_Ugradjivanje_tipa.md
[00]: 00_Sadrzaj.md
[0213]: 0213_Generici.md
