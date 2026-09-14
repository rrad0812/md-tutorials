
# Stringovi

[Nizovi, isečci i mape][0205] | [Sadržaj][00] | [Funkcije][0207]

Kao i mnogi drugi programski jezici, stringovi su takođe jedna važna vrsta tipova u Gou. Ovaj članak će navesti sve činjenice o stringovima.

## Unutrašnja struktura tipova stringova

Za standardni Go kompajler, unutrašnja struktura bilo kog tipa stringa je deklarisana ovako:

```go
type _string struct {
    elements *byte // underlying bytes
    len      int   // number of bytes
}
```

Iz deklaracije znamo da je string zapravo omotač **byte** sekvence. U stvari, string zaista možemo posmatrati kao (nepromenljivi po elementu) bajtni isečak.

Imajte na umu da je u programskom jeziku Go **byte** ugrađeni alias tipa **uint8**.

### Neke jednostavne činjenice o stringovima

Sledeće činjenice o stringovima smo naučili iz prethodnih članaka.

- String vrednosti mogu se koristiti kao konstante (zajedno sa bulovskim vrednostima i svim vrstama numeričkih vrednosti).
- Go podržava dva stila string literala, stil dvostrukih navodnika (ili interpretiranih literala) i stil povratnih navodnika (ili sirovih string literala).
- Nulte vrednosti tipova stringova su prazni stringovi, koji mogu biti predstavljeni sa "" ili `` u bukvalnom obliku.
- Stringovi se mogu spajati sa **+** i **+=** operatorima.
- Svi tipovi stringova su uporedivi (korišćenjem operatora **==** i **!=**). I kao celobrojne i vrednosti sa pokretnim zarezom, dve vrednosti istog tipa stringa takođe se mogu uporediti pomoću operatora **>**, **<**, **>=** i **<=**. Prilikom upoređivanja dva stringa, njihovi osnovni bajtovi će se upoređivati, jedan bajt po jedan bajt. Ako je jedan string prefiks drugog, a drugi je duži, onda će se drugi posmatrati kao duži.

Primer:

```go
package main

import "fmt"

func main() {
    const World = "world"
    var hello = "hello"

    // Concatenate strings.
    var helloWorld = hello + " " + World
    helloWorld += "!"
    fmt.Println(helloWorld) // hello world!

    // Compare strings.
    fmt.Println(hello == "hello")   // true
    fmt.Println(hello > helloWorld) // false
}
```

### Više činjenica o tipovima i vrednostima stringa

- Kao i u Javi, sadržaj (osnovni bajtovi) string vrednosti je nepromenljiv. Dužine string vrednosti se takođe ne mogu menjati odvojeno. Adresabilna string vrednost može se prepisati samo u celini dodeljivanjem druge string vrednosti.

- Ugrađeni **string** tip nema metode (baš kao i većina drugih ugrađenih tipova u Go-u), ali možemo koristimo funkcije koje se nalaze u **strings** standardnom paketu za sve vrste manipulacija stringovima.

- Pozovite ugrađenu **len** funkciju da biste dobili dužinu stringa (broj bajtova sačuvanih u stringa).

- Koristite sintaksu pristupa elementima **aString[i]** predstavljenu u pristupima elementima kontejnera da biste dobili **i**-tu byte vrednost sačuvanu u **aString**.

- Izraz **aString[i]** nije adresabilan. Drugim rečima, vrednost **aString[i]** se ne može menjati.

- Koristite sintaksu isecanja **aString[start:end]** da biste dobili podstring od **aString**. Ovde su **start** i **end** indeksi bajtova sačuvanih u **aString**.

Za standardni Go kompajler, promenljiva odredišnog stringa i vrednost izvornog stringa u dodeli stringa će deliti isti osnovni niz bajtova u memoriji.

Rezultat izraza podstringa **aString[start:end]** takođe deli isti osnovni niz bajtova sa osnovnim stringom **aStringu** memoriji.

Primer:

```go
package main

import (
    "fmt"
    "strings"
)

func main() {
    var helloWorld = "hello world!"

    var hello = helloWorld[:5] // substring
    // 104 is the ASCII code (and Unicode) of char 'h'.
    fmt.Println(hello[0])         // 104
    fmt.Printf("%T \n", hello[0]) // uint8

    // hello[0] is unaddressable and immutable,
    // so the following two lines fail to compile.
    /*
    hello[0] = 'H'         // error
    fmt.Println(&hello[0]) // error
    */

    // The next statement prints: 5 12 true
    fmt.Println(len(hello), len(helloWorld),
            strings.HasPrefix(helloWorld, hello))
}
```

Imajte na umu da ako su **aString** indeksi i u izrazima **aString[i]** i **aString[start:end]** konstante, onda će konstantni indeksi van opsega dovesti do neuspeha kompilacija.

Takođe, imajte na umu da su rezultati evaluacije takvih izraza uvek nekonstante ( ovo se može, ali i ne mora promeniti od kasnije verzije Go-a ). Na primer, sledeći program će ispisati 4 0.

```go
package main

import "fmt"

const s = "Go101.org" // len(s) == 9

// len(s) is a constant expression,
// whereas len(s[:]) is not.
var a byte = 1 << len(s) / 128
var b byte = 1 << len(s[:]) / 128

func main() {
    fmt.Println(a, b) // 4 0
}
```

O tome zašto se promenljive a i b izračunavaju na različite vrednosti, pročitajte pravilo određivanja posebnog tipa u operaciji operatora pomeranja po bitovima i koji pozivi funkcija se izračunavaju tokom kompajliranja.

## Kodiranje stringova i Unikod kodne tačke

Unikod standard određuje jedinstvenu vrednost za svaki znak u svim vrstama ljudskih jezika. Ali osnovna jedinica u Unikodu nije znak, već kodna tačka. Za većinu kodnih tačaka, svaka od njih odgovara jednom znaku, ali za nekoliko znakova, svaka od njih se sastoji od nekoliko kodnih tačaka.

Kodne tačke su predstavljene kao vrednosti **runa** u programu Go. U programu Go, **rune** je ugrađeni alias tipa **int32**.

U aplikacijama postoji nekoliko metoda kodiranja za predstavljanje kodnih tačaka, kao što su **UTF-8** kodiranje i **UTF-16** kodiranje. Danas je najpopularnije korišćeni metod kodiranja **UTF-8** kodiranje. U programu Go, sve string konstante se posmatraju kao **UTF-8** kodirane. Tokom kompajliranja, nedozvoljene **UTF-8** kodirane string konstante će dovesti do neuspeha kompajliranja. Međutim, tokom izvršavanja, Go runtime ne može sprečiti da neki stringovi budu nedozvoljeno **UTF-8** kodirani.

Za **UTF-8** kodiranje, svaka vrednost kodne tačke može biti sačuvana kao jedan ili više bajtova (do četiri bajta). Na primer, svaka engleska kodna tačka (koja odgovara jednom engleskom znaku) se čuva kao jedan bajt, međutim, svaka kineska kodna tačka (koja odgovara jednom kineskom znaku) se čuva kao tri bajta.

### Konverzije povezane sa stringovima

U članku konstante i promenljive, saznali smo da se celi brojevi mogu eksplicitno konvertovati u stringove (ali ne i obrnuto).

Ovde su predstavljena još dva pravila konverzije vezana za stringove u jeziku Go:

- String vrednost može se eksplicitno konvertovati u bajtni isečak i obrnuto. Bajtni isečak je isečak sa osnovnim tipom elementa **[]byte**.

- String vrednost može se eksplicitno konvertovati u runski isečak i obrnuto. Runski isečak je isečak čiji je osnovni tip elementa **[]rune**.

Prilikom konverzije iz runskog isečka u string, svaki element isečka (vrednost rune) biće kodiran u UTF-8 od jednog do četiri bajta i sačuvan u rezultujućem stringu. Ako je vrednost runskog elementa isečka van opsega važećih Unicode kodnih tačaka, onda će se posmatrati kao 0xFFFD, kodna tačka za Unicode zamenski znak. 0xFFFD biće kodiran UTF-8 kao tri bajta ( 0xef 0xbf 0xbd).

Kada se string konvertuje u runski isečak, bajtovi sačuvani u stringu će se posmatrati kao uzastopni UTF-8 kodirani niz bajtova za mnoge Unicode kodne tačke. Loši UTF-8 kodirani prikazi će biti konvertovani u runsku vrednost 0xFFFD.

Kada se string konvertuje u bajt isečak, rezultujući bajt isečak je samo duboka kopija osnovne bajt sekvence stringa. Kada se bajt isečak konvertuje u string, osnovna bajt sekvenca rezultujućeg stringa je takođe samo duboka kopija bajt isečka. Potrebna je alokacija memorije za čuvanje duboke kopije u svakoj takvoj konverziji. Razlog zašto je duboka kopija neophodna je taj što su elementi isečka promenljivi, ali su bajtovi sačuvani u stringovima nepromenljivi, tako da bajt isečak i string ne mogu deliti bajt elemente.

Imajte u vidu da za konverzije između stringova i bajtnih isečaka,

- Nedozvoljeni UTF-8 kodirani bajtovi su dozvoljeni i ostaće nepromenjeni.

- Standardni Go kompajler pravi neke optimizacije za neke posebne slučajeve takvih konverzija, tako da se ne prave duboke kopije. Takvi slučajevi će biti predstavljeni u nastavku.

Konverzije između bajtnih i runskih isečaka nisu direktno podržane u Go-u. Možemo koristiti sledeće načine da postignemo ovaj cilj:

- koristite **string** vrednosti kao prelaz. Ovaj način je zgodan, ali nije baš efikasan, jer su u procesu potrebne dve duboke kopije.

- koristite funkcije iz standardnog paketa **unicode/utf8**.

- Koristite funkciju **runes** iz standardnog paketa **bytes** za pretvaranje **[]byte** vrednosti u **[]rune** vrednost. U ovom paketu ne postoji funkcija za pretvaranje runskog isečkau isečak bajta.

Primer:

```go
package main

import (
    "bytes"
    "unicode/utf8"
)

func Runes2Bytes(rs []rune) []byte {
    n := 0
    for _, r := range rs {
        n += utf8.RuneLen(r)
    }
    n, bs := 0, make([]byte, n)
    for _, r := range rs {
        n += utf8.EncodeRune(bs[n:], r)
    }
    return bs
}

func main() {
    s := "Color Infection is a fun game."
    bs := []byte(s) // string -> []byte
    s = string(bs)  // []byte -> string
    rs := []rune(s) // string -> []rune
    s = string(rs)  // []rune -> string
    rs = bytes.Runes(bs) // []byte -> []rune
    bs = Runes2Bytes(rs) // []rune -> []byte
}
```

### Optimizacije kompajlera za konverzije između stringova i bajtnih isečaka

Molimo vas da pročitate poglavlje „Isecanje stringova i bajtova“ u knjizi Go Optimizations 101.

## for-range na stringovima

Tok **for-range** kontrole petlje primenjuje se na stringove. Ali imajte na umu da **for-range** će iterirati Unikod kodne tačke (kao rune vrednosti), umesto bajtova, u stringu. Loše UTF-8 reprezentacije kodiranja u stringu biće interpretirane kao **rune** vrednost 0xFFFD.

Primer:

```go
package main

import "fmt"

func main() {
    s := "éक्षिaπ囧"
    for i, rn := range s {
        fmt.Printf("%2v: 0x%x %v \n", i, rn, string(rn))
    }
    fmt.Println(len(s))
}
```

Izlaz gore navedenog programa:

```sh
 0: 0x65 e
 1: 0x301
 3: 0x915 क
 6: 0x94d
 9: 0x937
12: 0x93f
15: 0x61 a
16: 0x3c0 π
18: 0k56e7 囧
21
```

Iz izlaznog rezultata možemo utvrditi da

- Vrednost iterativnog indeksa možda nije kontinuirana. Razlog je taj što je indeks bajta u raspoređenom stringu i jednoj kodnoj tački može biti potrebno više od jednog bajta za predstavljanje.
- Prvi znak, é, sastoji se od dve rune (ukupno 3 bajta)
- Drugi znak, क्षि, sastoji se od četiri rune (ukupno 12 bajtova).
- Engleski znak, a, sastoji se od jedne rune (1 bajt).
- Karakter, π, je sastavljen od jedne rune (2 bajta).
- Kineski znak, 囧, sastoji se od jedne rune (3 bajta).

Kako onda iterirati bajtove u stringu? Uradite ovo:

```go
package main

import "fmt"

func main() {
    s := "éक्षिaπ囧"
    for i := 0; i < len(s); i++ {
        fmt.Printf("The byte at index %v: 0x%x \n", i, s[i])
    }
}
```

Sigurno možemo koristiti i gore pomenutu optimizaciju kompajlera za iteraciju bajtova u stringu. Za standardni Go kompajler, ovaj način je malo efikasniji od gore navedenog.

```go
package main

import "fmt"

func main() {
    s := "éक्षिaπ囧"
    // Here, the underlying bytes of s are not copied.
    for i, b := range []byte(s) {
        fmt.Printf("The byte at index %v: 0x%x \n", i, b)
    }
}
```

Iz nekoliko gore navedenih primera, znamo da će **len(s)** vratiti broj bajtova u stringu s. Vremenska složenost **len(s)** je **O(1)**.

Kako dobiti broj runa u stringu? Korišćenje **for-range** petlje za iteraciju i brojanje svih runa je jedan način, a korišćenje funkcije **RuneCountInString** iz **unicode/utf8** standardnog paketa je drugi način.
Efikasnost ova dva načina je skoro ista.

Treći način je korišćenje **len([]rune(s))** da bi se dobio broj runa u stringu s. Od Go Toolchain 1.11, standardni Go kompajler pravi optimizaciju za treći način kako bi se izbeglo nepotrebno duboko kopiranje, tako da bude jednako efikasan kao i prethodna dva načina.

Imajte na umu da su vremenske složenosti svih ovih načina O(n).

## Više metoda za spajanje stringova

Molimo vas da pročitate poglavlje „Isecanje stringova i bajtova“ u knjizi Go Optimizations 101.

Šećer: Koristite stringove kao isečke bajtova

Iz članka nizovi, kriške i mape, saznali smo da možemo koristiti ugrađene funkcije **copy** i **append** za kopiranje i dodavanje elemenata isečka. U stvari, kao poseban slučaj, ako je prvi argument poziva bilo koje od dve funkcije bajtni isečak, onda drugi argument može biti string (ako je poziv poziv **append**, onda argument stringa mora biti praćen tri tačke...). Drugim rečima, string se može koristiti kao bajtni isečak za poseban slučaj.

Primer:

```go
package main

import "fmt"

func main() {
    hello := []byte("Hello ")
    world := "world!"

    // The normal way:
    // helloWorld := append(hello, []byte(world)...)
    helloWorld := append(hello, world...) // sugar way
    fmt.Println(string(helloWorld))

    helloWorld2 := make([]byte, len(hello) + len(world))
    copy(helloWorld2, hello)
    
    // The normal way:
    // copy(helloWorld2[len(hello):], []byte(world))
    copy(helloWorld2[len(hello):], world) // sugar way
    fmt.Println(string(helloWorld2))
}
```

## Više o poređenjima nizova

Gore je pomenuto da je poređenje dva niza zapravo poređenje njihovih osnovnih bajtova. Generalno, Go kompajleri će napraviti sledeće optimizacije za poređenje nizova.

- Za **==** i **!=** poređenja, ako dužine dva upoređena niza nisu jednake, onda i ta dva niza moraju biti drugačija (nema potrebe za upoređivanjem njihovih bajtova).

- Ako su njihovi osnovni pokazivači bajt sekvence upoređenih nizova jednaki, onda je rezultat poređenja isti kao i poređenje dužina dva niza.

Dakle, za dva jednaka niza strinova, vremenska složenost njihovog poređenja zavisi od toga da li su njihovi pokazivači osnovnih bajtnih sekvenci jednaki. Ako su dva jednaka, onda je vremenska složenost O(1), u suprotnom, vremenska složenost je O(n), gde nje dužina dva niza.

Kao što je gore pomenuto, za standardni Go kompajler, prilikom dodeljivanja vrednosti stringa, vrednost odredišnog stringa i vrednost izvornog stringa će deliti isti osnovni niz bajtova u memoriji. Tako troškovi poređenja dva stringa postaju veoma mali.

Primer:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    bs := make([]byte, 1<<26)
    s0 := string(bs)
    s1 := string(bs)
    s2 := s1

    // s0, s1 and s2 are three equal strings.
    // The underlying bytes of s0 is a copy of bs.
    // The underlying bytes of s1 is also a copy of bs.
    // The underlying bytes of s0 and s1 are two
    // different copies of bs.
    // s2 shares the same underlying bytes with s1.

    startTime := time.Now()
    _ = s0 == s1
    duration := time.Now().Sub(startTime)
    fmt.Println("duration for (s0 == s1):", duration)

    startTime = time.Now()
    _ = s1 == s2
    duration = time.Now().Sub(startTime)
    fmt.Println("duration for (s1 == s2):", duration)
}
```

Izlaz:

```sh
trajanje za (s0 == s1): 10.462075ms
trajanje za (s1 == s2): 136ns
```

1ms je 1000000ns! Zato pokušajte da izbegavate poređenje dva dugačka niza ako ne dele isti osnovni bajtni niz.

[Nizovi, isečci i mape][0205] | [Sadržaj][00] | [Funkcije][0207]

[0205]: 0205_Nizovi_isečci_i_mape.md
[00]: 00_Sadrzaj.md
[0207]: 0207_Funkcije.md
