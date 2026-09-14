
# Paketi i uvoz paketa

[Deklaracije i pozivi funkcija][0106] | [Sadržaj][00] | [Izrazi i iskazi][0108]

Kao i mnogi moderni programski jezici, Go kod je takođe organizovan kroz pakete. Da biste koristili izvezene elemente koda (funkcije, tipove, promenljive i imenovane konstante itd.) u određenom paketu, paket prvo mora biti uvezen, osim **builtin** standardnog paketa koda (koji je univerzalni paket). Ovaj članak će objasniti pakete i uvoz paketa u Go-u.

## Uvoz paketa

Pogledajmo mali program koji uvozi standardni paket koda. (Pretpostavimo da je izvorni kod ovog programa sačuvan u datoteci pod nazivom "simple-import-demo.go.")

```go
package main

import "fmt"

func main() {
    fmt.Println("Go has", 25, "keywords.")
}
```

Neka objašnjenja:

- Prvi red navodi ime paketa koji sadrži izvorni fajl "simple-import-demo.go". **main** funkcija unosa programa mora biti smeštena u paket pod nazivom **main**.

- Treći red uvozi **fmt** standardni paket koristeći ključnu reč **import**. Identifikator **fmt** je naziv paketa. Takođe se koristi kao naziv uvoza i predstavlja ovaj standardni paket u opsegu koji sadrži izvornu datoteku. (Nazivi uvoza biće objašnjeni u odeljku ispod.) Postoji mnogo funkcija formatiranja deklarisanih u ovom standardnom paketu koje drugi paketi mogu da koriste. Funkcija **Println** je jedna od njih. Ona će ispisati string reprezentacije proizvoljnog broja argumenata na standardni izlaz.

- Šesti red poziva funkciju. Imajte na umu da ime funkcije u pozivu **Println** ima prefiks **fmt.**, gde je **fmt** ime paketa koji sadrži pozvanu funkciju. Oblik **aImportName.AnExportedIdentifier** se naziva kvalifikovani identifikator. **AnExportedIdentifier** se naziva nekvalifikovani identifikator.

- Poziv funkcije **fmt.Println** nema zahteve za svoje argumente, tako da će u ovom programu njegova tri argumenta biti izvedena kao vrednosti njihovih odgovarajućih podrazumevanih tipova **string**, **int** i **string**.

- Za svaki **fmt.Println** poziv, između svake dve uzastopne reprezentacije Stringova se ubacuje razmak, a na kraju se ispisuje znak za novi red.

Pokretanjem ovog programa, dobićete sledeći izlaz:

```sh
go run simple-import-demo.go
Go ima 25 ključnih reči.
```

Imajte u vidu da se samo izvezeni elementi koda u paketu mogu koristiti u izvornoj datoteci koja uvozi paket. Izvezeni element koda koristi izvezeni identifikator kao svoje ime. Na primer, prvi znak identifikatora **Println** je veliko slovo (dakle, **Println** funkcija se izvozi), zbog čega se **Println** funkcija deklarisana u **fmt** standardnom paketu može koristiti u gornjem primeru programa.

Ugrađene funkcije, **print** i **println**, imaju slične funkcionalnosti kao odgovarajuće funkcije u **fmt** standardnom paketu. Ugrađene funkcije se mogu koristiti bez uvoza bilo kojih paketa.

Imajte na umu da se dve ugrađene funkcije, **print** i **println**, ne preporučuju za upotrebu u produkcijskom okruženju, jer nije garantovano da će ostati u budućim verzijama Go-a.

Svi standardni paketi su navedeni ovde (zvanični) ili ovde (nezvanični, koji održava Go 101). Takođe možemo pokrenuti lokalni server da bismo videli Go dokumentaciju.

Uvoz paketa se u jeziku Go formalno naziva i deklaracija uvoza. Deklaracija uvoza je vidljiva samo izvornoj datoteci koja sadrži deklaraciju uvoza. Nije vidljiva drugim izvornim datotekama u istom paketu.

Pogledajmo još jedan primer:

```go
package main

import "fmt"
import "math/rand"

func main() {
    fmt.Printf("Next pseudo-random number is %v.\n", rand.Uint32())
}
```

Ovaj primer uvozi još jedan standardni paket, **math/rand**, koji je podpaket standardnog **math** paketa. Ovaj paket pruža neke funkcije za generisanje pseudoslučajnih brojeva.

Neka objašnjenja:

- U ovom primeru, naziv paketa **rand** se koristi kao naziv za uvoz uvezenog **math/rand** standardnog paketa. **rand.Uint32()** poziv će vratiti slučajni **uint32** ceo broj.

- **Printf** je još jedna često korišćena funkcija u fmtstandardnom paketu. Poziv funkcije **Printf** mora da primi barem jedan argument. Prvi argument **Printf** poziva funkcije mora biti string vrednost koja određuje format ispisanog rezultata. **%v** u prvom argumentu se naziva glagol formatiranja i biće zamenjen string reprezentacijom drugog argumenta. Kao što smo saznali u članku Osnovni tipovi i njihovi literali, **\n** u dvostruko navedenom string literalu biće izbegnut kao karakter za novi red.

Gore navedeni program će uvek ispisati:

```sh
Sledeći pseudo-slučajni broj je 2596996162.
```

**Napomena**: pre Go 1.20, ako očekujemo da gornji program proizvede drugačiji slučajni broj pri svakom pokretanju, trebalo bi da postavimo drugačije početno stanje pozivanjem funkcije **rand.Seed** kada se program tek pokrene.

Ako se više paketa uveze u izvorni fajl, možemo ih grupisati u jednu deklaraciju uvoza tako što ćemo ih zatvoriti u **()**.

Primer:

```go
package main

// Multiple packages can be imported together.
import (
    "fmt"
    "math/rand"
)

func main() {
    fmt.Printf("Next pseudo-random number is %v.\n", rand.Uint32())
}
```

Neka objašnjenja:

- Ovaj primer uvozi još jedan paket, **time** standardni paket, koji pruža mnoge uslužne programe vezane za vreme.

- Funkcija **time.Now()** vraća trenutno vreme, kao vrednost tipa **time.Time**.

## Više o fmt.Printf formatirajućim glagolima

Kao što je gore pomenuto, ako postoji jedan glagol formatiranja u prvom argumentu poziva fmt.Printf, on će biti zamenjen string reprezentacijom drugog argumenta. U stvari, u prvom stringargumentu može biti više glagola formatiranja. Drugi glagol formatiranja biće zamenjen string reprezentacijom trećeg argumenta, i tako dalje.

U Go 101, koristiće se samo sledeći navedeni glagoli formatiranja.

- **%v**, koji će biti zamenjen opštom string reprezentacijom odgovarajućeg argumenta.

- **%T**, koji će biti zamenjen imenom tipa ili literalom tipa odgovarajućeg argumenta.

- **%x**, koji će biti zamenjen heksadecimalnom reprezentacijom odgovarajućeg argumenta. Imajte na umu da heksadecimalne reprezentacije vrednosti nekih vrsta tipova nisu definisane. Generalno, odgovarajući argumenti **%x** treba da budu stringovi, celi brojevi, celobrojni nizovi ili celobrojni kriške (nizovi i kriške će biti objašnjeni u kasnijem članku).

- **%s**, koji će biti zamenjen string reprezentacijom odgovarajućeg argumenta. Odgovarajući argument treba da bude string ili bajtni isečak.

- Glagol formatiranja **//** predstavlja znak procenta.

Primer:

```go
package main

import "fmt"

func main() {
    a, b := 123, "Go"
    fmt.Printf("a == %v == 0x%x, b == %s\n", a, a, b)
    fmt.Printf("type of a: %T, type of b: %T\n", a, b)
    fmt.Printf("1// 50// 99//\n")
}
```

Izlaz:

```sh
a == 123 == 0x7b, b == Idi
tip a: int, tip b: string
1% 50% 99%
```

Za više **Printf** glagola formatiranja, pročitajte onlajn fmtdokumentaciju paketa ili pogledajte istu dokumentaciju pokretanjem lokalnog servera dokumentacije. Takođe možemo pokrenuti komandu **go doc fmt** da bismo videli dokumentaciju standardnog fmtpaketa i **go doc fmt.Printf** da bismo videli dokumentaciju funkcije **fmt.Printf** u terminalu.

## Direktorijum paketa, putanja uvoza paketa i zavisnosti paketa

Paket koda može da se sastoji od nekoliko izvornih datoteka. Ove izvorne datoteke se nalaze u istoj fascikli. Izvorne datoteke u fascikli (ne uključujući podfascikle) moraju pripadati istom paketu. Dakle, fascikla odgovara paketu koda i obrnuto. Fascikla koja sadrži izvorne datoteke paketa koda naziva se fascikla paketa.

Za Go Toolchain, paket čija putanja uvoza sadrži internalime fascikle se posmatra kao poseban paket. Mogu ga uvesti samo paketi unutar i ispod direktnog nadređenog direktorijuma fascikle internal. Na primer, paket .../a/b/c/internal/d/e/fi .../a/b/c/internalmogu ga uvesti samo paketi čije putanje uvoza imaju .../a/b/cprefiks.

Kada jedna izvorna datoteka u paketu uveze drugi paket, kažemo da paket koji se uvozi zavisi od paketa koji se uvozi.

Go ne podržava kružne zavisnosti paketa. Ako paket azavisi od paketa bi paket bzavisi od paketa c, onda izvorne datoteke u paketu cne mogu da uvezu paket ai b, a izvorne datoteke u paketu bne mogu da uvezu paket a.

Sigurno je da izvorne datoteke u paketu ne mogu, i ne moraju, da uvoze sam paket.

Kasnije ćemo pakete sa imenom maini koji sadrže mainfunkcije za unos nazivati programskim paketima (ili paketima komandi ), a ostale pakete nazivati bibliotečkim paketima . Programski paketi se ne mogu uvoziti. Svaki Go program treba da sadrži jedan i samo jedan programski paket.

Naziv fascikle paketa ne mora biti isti kao i naziv paketa. Međutim, za bibliotečki paket, korisnici paketa će biti zbunjeni ako se naziv paketa razlikuje od naziva fascikle paketa. Uzrok zabune je taj što je podrazumevana putanja uvoza paketa naziv paketa, ali ono što je sadržano u putanji uvoza paketa je naziv fascikle paketa. Zato pokušajte da napravite dva imena identična za svaki bibliotečki paket.

S druge strane, preporučuje se da se svakoj fascikli programskog paketa da smisleno ime, drugačije od imena paketa, main.

## Funkcija init​

U paketu može biti više funkcija sa nazivom **init** "kao što je deklarisano", čak i u datoteci izvornog koda. Funkcije sa nazivom **init** ne smeju imati ni ulazne parametre niti vraćati rezultate.

Imajte na umu da se na najvišem bloku na nivou paketa, **init** identifikator može koristiti samo u deklaracijama funkcija. Ne možemo deklarisati promenljive/konstante/tipove na nivou paketa čija su imena **init**.

Tokom izvršavanja, svaka initfunkcija će biti (sekvencijalno) pozvana jednom i samo jednom (pre pozivanja **main** funkcije za unos). Dakle, značenje funkcija initje veoma slično statičkim blokovima inicijalizatora u Javi.

Evo jednostavnog primera koji sadrži dve initfunkcije:

```go
package main

import "fmt"

func init() {
    fmt.Println("hi,", bob)
}

func main() {
    fmt.Println("bye")
}

func init() {
    fmt.Println("hello,", smith)
}

func titledName(who string) string {
    return "Mr. " + who
}

var bob, smith = titledName("Bob"), titledName("Smith")
```

Izlaz ovog programa:

```sh
Zdravo, gospodine Bob
Zdravo, gospodine Smit
ćao
```

## Redosled inicijalizacije resursa

Tokom izvršavanja, paket će biti učitan nakon svih paketa zavisnosti. Svaki paket će biti učitan jednom i samo jednom.

Sve **init** funkcije u svim uključenim paketima u programu biće pozivane sekvencijalno. **init**  funkcija u paketu za uvoz biće pozvana nakon što su sve initfunkcije deklarisane u paketima zavisnosti paketa za uvoz sigurno. Sve **init** funkcije će biti pozvane pre pozivanja **main** funkcije za unos.

Redosled pozivanja funkcija initu istoj izvornoj datoteci je od vrha do dna. Go specifikacija preporučuje, ali ne zahteva, pozivanje funkcija initu različitim izvornim datotekama istog paketa po abecednom redosledu imena datoteka koje ih sadrže. Stoga nije dobra ideja imati zavisne odnose između dve initfunkcije u dve različite izvorne datoteke.

Sve promenljive na nivou paketa deklarisane u paketu se inicijalizuju pre nego što initse pozove bilo koja funkcija deklarisana u istom paketu.

Go runtime će pokušati da inicijalizuje promenljive na nivou paketa u paketu prema njihovom redosledu deklaracije, ali promenljiva na nivou paketa će sigurno biti inicijalizovana nakon svih njenih zavisnih promenljivih. Na primer, u sledećem isečku koda, inicijalizacije četiri promenljive na nivou paketa se dešavaju redosledom *y*, *z*, *x* i *w*.

```go
func f() int {
    return z + y
}

func g() int {
    return y/2
}

var (
    w       = x
    x, y, z = f(), 123, g()
)
```

O detaljnijem pravilu redosleda inicijalizacije promenljivih na nivou paketa, pročitajte članak:
"Redosled evaluacije izraza".

## Obrasci za uvoz kompletnih paketa

U stvari, puni obrazac uvozne deklaracije je

```go
import importname "path/to/package"
```

gde **importname** je opciono, njegova podrazumevana vrednost je ime (ne ime dira) uvezenog paketa.

U stvari, u gore navedenim deklaracijama uvoza, importnamesvi delovi su izostavljeni, jer su identični odgovarajućim imenima paketa. Ove deklaracije uvoza su ekvivalentne sledećim:

```go
import fmt "fmt"        // <=> import "fmt"
import rand "math/rand" // <=> import "math/rand"
import time "time"      // <=> import "time"
```

Ako importnamese deo nalazi u deklaraciji uvoza, onda prefiksni tokeni koji se koriste u kvalifikovanim identifikatorima moraju biti importnameumesto imena uvezenog paketa.

Potpuni obrazac deklaracije za uvoz se ne koristi široko. Međutim, ponekad ga moramo koristiti. Na primer, ako izvorna datoteka uvozi dva paketa sa istim imenom, da bismo izbegli zabunu kompajlera, moramo koristiti potpuni obrazac za uvoz da bismo podesili prilagođenu promenu importnameza barem jedan od ta dva paketa.

Evo primera korišćenja kompletnih obrazaca za uvoznu deklaraciju.

```go
package main

import (
    format "fmt"
    random "math/rand"
)

func main() {
    format.Print("A random number: ", random.Uint32(), "\n")

    // The following line fails to compile,
    // for "rand" is not identified.
    /*
    fmt.Print("A random number: ", rand.Uint32(), "\n")
    */
}
```

Neka objašnjenja:

- Moramo koristiti formati **random** kao prefiksni token u kvalifikovanim identifikatorima, umesto stvarnih imena paketa **fmt** i **rand**.

- **Print** je još jedna funkcija u **fmt** standardnom paketu. Kao i **Println** pozivi funkcija, **Print** poziv funkcije može prihvatiti proizvoljan broj argumenata. Ispisaće string reprezentacije prosleđenih argumenata, jedan po jedan. Ako dva uzastopna argumenta nisu string vrednosti, onda će se između njih u rezultatu ispisa automatski umetnuti razmak.

U **importname** punom obliku deklaracije uvoza može biti tačka (**.**). Takav uvoz se naziva tačkasti uvoz. Da biste koristili izvezene elemente u paketima koji se uvoze sa tačkom, prefiksni deo u kvalifikovanim identifikatorima mora biti izostavljen.

Primer:

```go
package main

import (
    . "fmt"
    . "time"
)

func main() {
    Println("Current time:", Now())
}
```

U gornjem primeru, **Println** umesto **fmt.Println**, i **Now** umesto **time.Now** mora se koristiti.

Generalno, uvoz tačaka smanjuje čitljivost koda, tako da se ne preporučuje za upotrebu u formalnim projektima.

U punom **importname** obliku deklaracija uvoza može biti prazan identifikator (**_**). Takav uvoz se naziva anonimni uvoz (neki članci ih nazivaju i prazni uvozi). Izvorne datoteke koje se uvoze ne mogu koristiti izvezene elemente koda u anonimno uvezenim paketima. Svrha anonimnog uvoza je inicijalizacija uvezenih paketa (svaka od initfunkcija u anonimno uvezenim paketima biće pozvana jednom).

U sledećem primeru, sve initfunkcije deklarisane u standardnom **net/http/pprof** paketu biće pozvane pre nego što **main** se pozove funkcija unosa.

```go
package main

import _ "net/http/pprof"

func main() {
    ... // do some things
}
```

Svaki neanonimni uvoz mora biti korišćen barem jednom.

Osim anonimnog uvoza, ostali uvozi moraju se koristiti barem jednom. Na primer, sledeći primer se ne kompajlira.

```go
package main

import (
    "net/http" // error: imported and not used
    . "time"   // error: imported and not used
)

import (
    format "fmt"  // okay: it is used once below
    _ "math/rand" // okay: it is not required to be used
)

func main() {
    format.Println() // use the imported "fmt" package
}
```

## Moduli

Modul je kolekcija nekoliko paketa. Nakon preuzimanja na lokalnu mapu, svi ovi paketi se nalaze u istoj fascikli, koja se naziva korenska fascikla modula. Modul može imati mnogo verzija, koje prate specifikaciju semantičkog verzionisanja. Za više koncepata vezanih za module i kako upravljati i koristiti module, pročitajte zvaničnu dokumentaciju.

[Deklaracije i pozivi funkcija][0106] | [Sadržaj][00] | [Izrazi i iskazi][0108]

[0106]: 0106_Deklaracije_i_pozivi_funkcija.md
[00]: 00_Sadrzaj.md
[0108]: 0108_Izrazi_i_iskazi.md
