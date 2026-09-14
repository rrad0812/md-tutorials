
# Osnovni tokovi upravljanja

[Izrazi i iskazi][0108] | [Sadržaj][00] | [Gorutine,Odloženi funkcijski pozivi i Panic/Recover][0110]

Blokovi koda za kontrolu toka u jeziku Go su veoma slični drugim popularnim programskim jezicima, ali postoje i mnoge razlike. Ovaj članak će pokazati te sličnosti i razlike.

## Uvod u kontrolne tokove u Go-u

Postoje tri vrste osnovnih blokova koda za kontrolu toka u jeziku Go:

- **if-else** dvosmerni blok uslovnog izvršavanja.
- **for** blok petlje
- **switch-case** višesmerni uslovni izvršni blok.

Takođe postoje neki blokovi koda za kontrolu toka koji su povezani sa određenim vrstama tipova u Go jeziku.

- **for-range** blok petlje za iteraciju celih brojeva, svih vrsta kontejnera, kanala i nekih funkcija.
- **type-switch** višesmerni uslovni izvršni blok za interfejse.
- **select-case** blok za kanale.

Kao i mnogi drugi popularni jezici, Go takođe podržava:

- **break**,
- **continue** i
- **goto**

naredbe za skok pri izvršavanju koda. Pored njih, u Go-u postoji posebna naredba za skok koda,

- **fallthrough**.

Među šest vrsta blokova kontrole toka, osim samog **if-else** kontrole toka, ostalih pet se nazivaju lomljivi (**breakable**) blokovi kontrole toka. Možemo koristiti **break** naredbe da bi izvršenja iskakala iz lomljivih blokova kontrole toka.

**for-range** blokovi petlje se nazivaju blokovi toka kontrole petlje. Možemo koristiti **continue** iskaze da unapred završimo iteraciju petlje u bloku toka kontrole petlje, tj. da nastavimo sa sledećom iteracijom petlje.

Imajte na umu da je svaki od gore pomenutih blokova kontrole toka iskaz i može sadržati mnoge druge podiskaze.

Gore pomenuti iskazi toka upravljanja su svi oni u užem smislu. Mehanizmi predstavljeni u sledećem članku, **gorutine**, **odloženi pozivi funkcija** i **panic/recover**, i **tehnike sinhronizacije konkurentnosti** predstavljene u kasnijem članku pregled sinhronizacije konkurentnosti mogu se posmatrati kao iskazi toka upravljanja u širem smislu.

Trenutni članak uglavnom objašnjava osnovne blokove koda toka upravljanja i naredbe za skok koda. Takođe će biti dotaknute **for range anInteger {...}** petlje koje je uveo Go 1.22. Ostali blokovi koda toka upravljanja biće objašnjeni u mnogim drugim narednim Go 101 člancima.

### if-else Blokovi kontrole toka

Pun oblik **if-else** bloka koda je ovakav:

```go
if InitSimpleStatement; Condition {
    // do something
} else {
    // do something
}
```

**if** i **else** su ključne reči. Kao i kod mnogih drugih programskih jezika, **else** grana je opcionalna.

Deo **InitSimpleStatement** je takođe opcionalan. Mora biti jednostavan iskaz ako je prisutan. Ako ga nema, možemo ga posmatrati kao prazan iskaz (jedna vrsta jednostavnih iskaza). U praksi, **InitSimpleStatement** je često kratka deklaracija promenljive ili čisto dodeljivanje. **Condition** mora biti izraz koji rezultira bulovom vrednošću. **Condition** deo može biti zatvoren u paru **()** ili, ali ne može biti zatvoren zajedno sa **InitSimpleStatement** delom.

Ako je **InitSimpleStatement** u **if-else** bloku prisutan, biće izvršen pre izvršavanja drugih naredbi u **if-else** bloku. Ako **InitSimpleStatement** nije prisutan, **tačka-zarez** koji sledi iza njega je opcionalan.

Svaki **if-else** tok upravljanja formira jedan implicitni blok koda, granu eksplicitnog **if** blok koda i granu opcioniog **else** blok koda. Dva bloka koda su ugnežđena u implicitni blok koda. Nakon izvršavanja, ako **Condition** izraz rezultira sa **true**, onda će će se izvršiti **if** grana bloka koda, u suprotnom će se izvršiti grana **else** bloka koda.

Primer:

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    if n := rand.Int(); n%2 == 0 {
        fmt.Println(n, "is an even number.")
    } else {
        fmt.Println(n, "is an odd number.")
    }

    n := rand.Int() % 2 // this n is not the above n.
    if n % 2 == 0 {
        fmt.Println("An even number.")
    }

    if ; n % 2 != 0 {
        fmt.Println("An odd number.")
    }
}
```

Ako je promenljiva **InitSimpleStatement** u **if-else** bloku koda kratka deklaracija, onda će se deklarisane promenljive posmatrati kao da su deklarisane u najvišem ugnežđenom implicitnom bloku koda **if-else** bloka koda.

Blok koda grane **else** može biti implicitan ako iza odgovarajućeg **else** sledi drugi **if-else** blok koda, u suprotnom mora biti eksplicitan.

Primer:

```go
package main

import (
    "fmt"
    "time"
)

func main() {
    if h := time.Now().Hour(); h < 12 {
        fmt.Println("Now is AM time.")
    } else if h > 19 {
            fmt.Println("Now is evening time.")
    } else {
        fmt.Println("Now is afternoon time.")
        h := h // the right one is declared above
        // The just new declared "h" variable
        // shadows the above same-name one.
        _ = h
    }
    // h is not visible here.
}
```

### for Blokovi toka upravljanja petljom

Pun oblik bloka forpetlje je:

```go
for InitSimpleStatement; Condition; PostSimpleStatement {
    // do something
}
```

**for** je ključna reč. Delovi **InitSimpleStatement** i **PostSimpleStatement** moraju biti jednostavni iskazi, a **PostSimpleStatement** deo ne sme biti kratka deklaracija promenljive. **Condition** mora biti izraz čiji je rezultat bulova vrednost. Sva tri dela su opcionalna.

Za razliku od mnogih drugih programskih jezika, upravo pomenuta tri dela koja slede **for** ključnu reč ne mogu biti zatvorena u par **()**.

Svaki **for** tok upravljanja formira najmanje dva bloka koda, jedan je implicitni, a jedan eksplicitni. Eksplicitni je ugnežđen u implicitnom.

Blok **InitSimpleStatement** u **for** petlji će biti izvršen (samo jednom) pre izvršenja ostalih naredbi u **for** bloku petlje.

Izraz **Condition** će biti izračunat u svakoj iteraciji petlje. Ako je rezultat izračunavanja false, petlja će se završiti. U suprotnom, telo (tj. eksplicitni blok koda) petlje će se izvršiti.

Biće **PostSimpleStatement** izvršeno na kraju svake iteracije petlje.

Primer petlje **for**. Primer će ispisati cele brojeve od 0 do 9.

```go
for i := 0; i < 10; i++ {
    fmt.Println(i)
}
```

Ako oba dela **InitSimpleStatement** i **PostSimpleStatement** nedostaju (samo ih posmatrajte kao prazne izraze), njihove dve tačke-zareze u blizini mogu se izostaviti. Ovakav oblik se naziva **for** petlja samo sa uslovom. Ista je kao i **while** petlja u drugim jezicima.

```go
var i = 0
for ; i < 10; {
    fmt.Println(i)
    i++
}
for i < 20 {
    fmt.Println(i)
    i++
}
```

Ako **Condition** deo nedostaje, kompajleri će ga videti kao **true**.

```go
for i := 0; ; i++ {
    if i >= 10 {
        // "break" statement will be explained below.
        break
    }
    fmt.Println(i)
}

// The following 4 endless loops are
// equivalent to each other (for most cases).
for ; true; {
}

for true {
}

for ; ; {
}

for {
}
```

Ako je **InitSimpleStatement** **for** bloka kratki iskaz za deklaraciju promenljive, onda će se deklarisane promenljive petlje posmatrati kao da su deklarisane u najvišem ugnežđenom implicitnom bloku koda bloka **for**. Na primer, sledeći isečak koda se ispisuje 012 umesto 0.

```go
for i := 0; i < 3; i++ {
    fmt.Print(i)
    // The left i is a new declared variable,
    // and the right i is the loop variable.
    i := i
    // The new declared variable is modified, but
    // the old one (the loop variable) is not yet.
    i = 10
    _ = i
}
```

> [!Note]
> Go 1.22 je izmenio semantiku for blokova petlji.
>
> - Pre verzije Go 1.22, svaka deklarisana promenljiva petlje korišćena u for
> bloku petlje bila je deljena od strane svih iteracija tokom izvršavanja bloka
> petlje.
> - Od verzije Go 1.22, svaka deklarisana promenljiva petlje koja se koristi u
> for petlji biće instancirana kao posebna instanca na početku svake iteracije.
>
> U većini slučajeva, semantička promena ne menja ponašanje koda. Ali ponekad
> menja. Dakle, semantička promena narušava kompatibilnost sa prethodnim
> verzijama. Od Go verzije 1.22, svaka Go izvorna datoteka treba da ima navedenu
> Go verziju kako bi se šteta što više smanjila.

**break** iskaza se može koristiti da bi se prevremeno izašlo iz kontrole toka petlje, ako je blok kontrole toka petlje najdublji prekinuti blok kontrole toka koji sadrži iskazu. Na primer, sledeći kod se takođe ispisuje od 0 do 9.

i := 0
for {
    if i >= 10 {
        break
    }
    fmt.Println(i)
    i++
}

**continue** iskaz se može koristiti za revremeni završetak tekuće iteracije petlje ( **PostSimpleStatement** i dalje će se izvršiti ), Ako je forje blok toka kontrole petlje najdublji blok toka kontrole petlje koji sadrži **continue** iskaz. Na primer, sledeći isečak koda će ispisati 13579.

for i := 0; i < 10; i++ {
    if i % 2 == 0 {
        continue
    }
    fmt.Print(i)
}

### for-range blokovi kontrole toka

**for-range** blokovi petlji mogu se koristiti za iteraciju celih brojeva, svih vrsta kontejnera, kanala i nekih funkcija. Trenutni članak objašnjava samo kako se koriste **for-range** blokovi petlji za iteraciju celih brojeva.

**Napomena**: korišćenje **for-range** blokova petlji za iteraciju celih brojeva je podržano tek od Go 1.22.

Sledeći kod:

```go
// i is declared earlier.
for i = range anInteger {
    ...
}
```

je zapravo skraćeni oblik od

```go
for i = 0; i < anInteger; i++ {
    ...
}
```

Slično tome:

```go
for i := range anInteger {
    ...
}
```

je zapravo skraćeni oblik od

```go
for i := 0; i < anInteger; i++ {
    ...
}
```

sa semantikom Go 1.22+.

Na primer, poslednji primer u poslednjem odeljku je ekvivalentan

```go
for i := range 10 {
    if i % 2 == 0 {
        continue
    }
    fmt.Print(i)
}
```

### switch-case blokovi kontrole toka

**switch-case** blok kontrole toka je jedna vrsta bloka kontrole toka uslovnog izvršavanja.

Puni oblik **switch-case** bloka je

```go
switch InitSimpleStatement; CompareOperand0 {
    case CompareOperandList1:
        // do something
    case CompareOperandList2:
        // do something
    ...
    case CompareOperandListN:
        // do something
    default:
        // do something
}
```

U punom obliku,

- **switch**, **case** i **default** su tri ključne reči.

- Deo **InitSimpleStatement** mora biti jednostavan iskaz. **CompareOperand0** deo je izraz koji se posmatra kao tipizovana vrednost (ako je netipizovana vrednost, onda se posmatra kao vrednost tipa njenog podrazumevanog tipa), stoga ne može biti netipizovan **nil**. **CompareOperand0** u Go specifikaciji se naziva **switch** izraz.

- Svaki od delova **CompareOperandListX** ( Xmože predstavljati od 1 do N) mora biti lista izraza odvojenih zarezima. Svaki od ovih izraza mora biti uporediv sa **CompareOperand0**. Svaki od ovih izraza se naziva izrazom slučaja u Go specifikaciji. Ako je izraz slučaja netipizovana vrednost, onda mora biti implicitno konvertibilan u tip izraza switch u istom **switch-case** toku upravljanja. Ako je konverzija nemoguća, kompilacija ne uspeva.

- Svaki case **CompareOperandListX** ili **default** otvara (i prati ga) implicitni blok koda. Implicitni blok koda i taj **case CompareOperandListX:** ili **default:** formiraju granu. Prisustvo svake takve grane je opciono. Implicitni blok koda u takvoj grani kasnije ćemo nazvati blokom koda grane.

- U bloku kontrole toka može biti najviše jedna **default** grana **switch-case**.

- Pored blokova koda grananja, svaki **switch-case** kontrole toka formira dva bloka koda, jedan je implicitni i jedan eksplicitni. Eksplicitni je ugnežđen u implicitni. Svi blokovi koda grananja su ugnežđeni u eksplicitni (i indirektno ugnežđeni u implicitni).

- **switch-case** blokovi kontrole toka su lomljivi, tako da se **break** iskazi mogu koristiti i u bilo kom bloku koda grane u bloku kontrole toka kako bi se izvršenje **switch-case** prevremeno napustilo iz bloka kontrole toka.

- **switch-case** će prvo izvršiti **InitSimpleStatement** kada **switch-case** izvrši kontrolni tok. Izvršiće se samo jednom tokom izvršavanja **switch-case** kontrole toka. Nakon što **InitSimpleStatement** se izvrši, izraz **switch CompareOperand0** će biti procenjen i procenjen samo jednom. Rezultat procene je uvek tipizirana vrednost. Rezultat procene će biti upoređen (korišćenjem operatora ==) sa rezultatom procene svakog izraza **case** u **CompareOperandListX** listama izraza, odozgo nadole i s leva na desno. Ako se utvrdi da je izraz case jednak **CompareOperand0**, proces poređenja se zaustavlja i odgovarajući blok koda grane izraza će biti izvršen. Ako se ne utvrdi da je nijedan izraz case jednak **CompareOperand0**, izvršiće se podrazumevani blok koda grane (ako je prisutan).

Primer kontrole **switch-case** kontrole toka:

```go
package main

import (
    "fmt"
    "math/rand"
)

func main() {
    switch n := rand.Intn(100); n%9 {
        case 0:
            fmt.Println(n, "is a multiple of 9.")
    
            // Different from many other languages,
            // in Go, the execution will automatically
            // jumps out of the switch-case block at
            // the end of each branch block.
            // No "break" statement is needed here.
        
        case 1, 2, 3:
            fmt.Println(n, "mod 9 is 1, 2 or 3.")
            // Here, this "break" statement is nonsense.
            break
        case 4, 5, 6:
            fmt.Println(n, "mod 9 is 4, 5 or 6.")
        // case 6, 7, 8:
            // The above case line might fail to compile,
            // for 6 is duplicate with the 6 in the last
            // case. The behavior is compiler dependent.
        default:
            fmt.Println(n, "mod 9 is 7 or 8.")
    }
}
```

Funkcija **rand.Intn** vraća nenegativnu **int** slučajnu vrednost koja je manja od navedenog argumenta (100).

Imajte na umu da ako se bilo koja dva izraza sa veličinom slova u switch-casekontrolnom toku mogu detektovati kao jednaka tokom kompajliranja, onda kompajler može odbaciti ovaj drugi. Na primer, standardni Go kompajler smatra da case 6, 7, 8je red u gornjem primeru nevažeći ako taj red nije komentarisan. Ali drugi kompajleri mogu misliti da je taj red u redu. U stvari, trenutni standardni Go kompajler (verzija 1.25.n) dozvoljava duplirane bulove izraze sa veličinom slova, a gccgo (v8.2) dozvoljava i duplirane bulove i string izraze sa veličinom slova.

Kao što komentari u gornjem primeru opisuju, za razliku od mnogih drugih jezika, u programskom jeziku Go, na kraju svakog bloka grana koda, izvršenje će se automatski prekinuti izvan odgovarajućeg switch-casekontrole bloka. Kako onda dozvoliti da izvršenje pređe u sledeći blok grana koda? Go pruža fallthroughključnu reč za obavljanje ovog zadatka. Na primer, u sledećem primeru, svaki blok grana koda će se izvršiti, po redosledu, od vrha do dna.

```go
switch n := rand.Intn(100) % 5; n {
    case 0, 1, 2, 3, 4:
        fmt.Println("n =", n)
        // The "fallthrough" statement makes the
        // execution slip into the next branch.
        fallthrough
    case 5, 6, 7, 8:
        // A new declared variable also called "n",
        // it is only visible in the current
        // branch code block.
        n := 99
        fmt.Println("n =", n) // 99
        fallthrough
    default:
        // This "n" is the switch expression "n".
        fmt.Println("n =", n)
}
```

Molimo vas da obratite pažnju,

- iskaz **fallthrough** mora biti poslednji iskaz u grani.

- iskaza **fallthrough** se ne može pojaviti u poslednjoj grani u **switch-case** bloku kontrole toka.

- Na primer, sledeće **fallthrough** upotrebe su sve nedozvoljene.

  ```go
  switch n := rand.Intn(100) % 5; n {
      case 0, 1, 2, 3, 4:
          fmt.Println("n =", n)
          // The if-block, not the fallthrough statement,
          // is the final statement in this branch.
          if true {
              fallthrough // error: not the final statement
          }
      case 5, 6, 7, 8:
          n := 99
          fallthrough // error: not the final statement
          _ = n
      default:
          fmt.Println(n)
          fallthrough // error: show up in the final branch
  }
  ```

- Delovi **InitSimpleStatement** i **CompareOperand0** u **switch-case** toku upravljanja su opcioni. Ako **CompareOperand0** deo i nedostaje, posmatraće se kao **true**, tipizirana vrednost ugrađenog tipa **bool**.

- Ako **InitSimpleStatement** deo nedostaje, tačka-zarez koji sledi iza njega može se izostaviti.

- I kao što je gore pomenuto, sve grane su opcione. Dakle, sledeći blokovi koda su svi legalni, svi se mogu smatrati blokovima bez operacija.

  ```go
  switch n := 5; n {
  }
  
  switch 5 {
  }
  
  switch _ = 5; {
  }
  
  switch {
  }
  ```

Za poslednja dva **switch-case** bloka kontrole toka u poslednjem primeru, kao što je gore pomenuto, svaki od odsutnih **CompareOperand0** delova se posmatra kao tipizirana vrednost **true** ugrađenog tipa **bool**.

Dakle, sledeći isečak koda će ispisati "hello".

```go
switch {
    case true: 
        fmt.Println("hello")
    default: 
        fmt.Println("bye")
}
```

Još jedna očigledna razlika u odnosu na mnoge druge jezike je redosled grananja.  **default** u **switch-case** bloku kontrole toka koji može biti proizvoljan. Na primer, sledeća tri **switch-case** bloka kontrole toka su ekvivalentna jedan drugom.

```go
switch n := rand.Intn(3); n {
    case 0: 
        fmt.Println("n == 0")
    case 1: 
        fmt.Println("n == 1")
    default: 
        fmt.Println("n == 2")
}

switch n := rand.Intn(3); n {
    default: 
        fmt.Println("n == 2")
    case 0: 
        fmt.Println("n == 0")
    case 1: 
        fmt.Println("n == 1")
}

switch n := rand.Intn(3); n {
    case 0: 
        fmt.Println("n == 0")
    default: 
        fmt.Println("n == 2")
    case 1: 
        fmt.Println("n == 1")
}
```

### goto deklaracija iskazi i oznake

Kao i mnogi drugi jezici, Go takođe podržava **goto** iskaz. **goto** ključnu reč mora pratiti oznaka da bi se formirao iskaz. Oznaka se deklariše u obliku **LabelName:**, gde **LabelName** mora biti identifikator. Oznaka čije ime nije prazan identifikator mora se koristiti barem jednom.

Iskaza goto će učiniti da izvršenje skoči na sledeći iskaz nakon deklaracije oznake koja se koristi u **goto** iskazu. Dakle, deklaraciju oznake mora pratiti jedna iskaza.

Oznaka mora biti deklarisana unutar tela funkcije. Upotreba oznake može se pojaviti pre ili posle deklaracije oznake. Međutim, oznaka nije vidljiva (i ne može se pojaviti) izvan najdubljeg bloka koda u kome je oznaka deklarisana.

Sledeći primer koristi **goto** iskaz i oznaku za implementaciju toka upravljanja petljom.

```go
package main

import "fmt"

func main() {
    i := 0

Next: // here, a label is declared.
    fmt.Println(i)
    i++
    if i < 5 {
        goto Next // execution jumps
    }
}
```

Kao što je gore pomenuto, oznaka nije vidljiva (i ne može se pojaviti) izvan najdubljeg bloka koda u kome je oznaka deklarisana. Stoga se sledeći primer ne kompajlira.

```go
package main

func main() {
goto Label1 // error
    {
        Label1:
        goto Label2 // error
    }
    {
        Label2:
    }
}
```

Imajte na umu da, ako je oznaka deklarisana unutar opsega važenja promenljive, onda se upotreba oznake ne može pojaviti pre deklaracije promenljive. Opsezi identifikatora biće objašnjeni kasnije u članku o blokovima i opsezima u Go-u.

Sledeći primer se takođe ne kompajlira.

```go
package main

import "fmt"

func main() {
    i := 0
Next:
    if i >= 5 {
        // error: jumps over declaration of k
        goto Exit
    }

    k := i + i
    fmt.Println(k)
    i++
    goto Next

// This label is declared in the scope of k,
// but its use is outside of the scope of k.
Exit:
}
```

Upravo pomenuto pravilo se može kasnije promeniti. Trenutno, da bi se gornji kod ispravno kompajlirao, moramo podesiti opseg promenljive k. Postoje dva načina da se reši problem u poslednjem primeru.

Jedan od načina je smanjivanje obima promenljive k.

```go
func main() {
    i := 0
Next:
    if i >= 5 {
        goto Exit
    }
    // Create an explicit code block to
    // shrink the scope of k.
    {
        k := i + i
        fmt.Println(k)
    }
    i++
    goto Next
Exit:
}
```

Drugi način je proširenje opsega promenljive k.

```go
func main() {
    var k int // move the declaration of k here.
    i := 0
Next:
    if i >= 5 {
        goto Exit
    }

    k = i + i
    fmt.Println(k)
    i++
    goto Next
Exit:
}
```

### break i continue iskazi sa oznakama

iskaz **goto** mora da sadrži oznaku. Iskaz **break** ili **continue** takođe može da sadrži oznaku, ali je oznaka opcionalna. Generalno, **break** oznake koje sadrže oznake se koriste u ugnežđenim blokovima toka upravljanja koji se mogu prekinuti, a **continue** iskaze koje sadrže oznake se koriste u ugnežđenim blokovima toka upravljanja petljom.

Ako **break** naredba sadrži oznaku, oznaka mora biti deklarisana neposredno pre prekidnog bloka kontrole toka koji sadrži naredbu **break**. Ime oznake možemo posmatrati kao ime prekidnog bloka kontrole toka. Naredba **break** će učiniti da izvršenje iskoči izvan prekidnog bloka kontrole toka, čak i ako prekidni blok kontrole toka nije najdublji prekidni blok kontrole toka koji sadrži **break** naredbu.

Ako **continue** iskaz sadrži oznaku, oznaka mora biti deklarisana neposredno pre bloka toka kontrole petlje koji sadrži continueiskaz. Ime oznake možemo posmatrati kao ime bloka toka kontrole petlje. Iskaz **continue** će prevremeno završiti trenutnu iteraciju bloka toka kontrole petlje, čak i ako blok toka kontrole petlje nije najdublji blok toka kontrole petlje koji sadrži iskaz **continue**.

Sledi primer korišćenja **break** i **continue** naredbi sa oznakama.

```go
package main

import "fmt"

func FindSmallestPrimeLargerThan(n int) int {
Outer:
    for n++; ; n++{
        for i := 2; ; i++ {
            switch {
            case i * i > n:
                break Outer
            case n % i == 0:
                continue Outer
            }
        }
    }
    return n
}

func main() {
    for i := 90; i < 100; i++ {
        n := FindSmallestPrimeLargerThan(i)
        fmt.Print("The smallest prime number larger than ")
        fmt.Println(i, "is", n)
    }
}
```

[Izrazi i iskazi][0108] | [Sadržaj][00] | [Gorutine,Odloženi funkcijski pozivi i Panic/Recover][0110]

[0108]: 0108_Izrazi_i_iskazi.md
[00]: 00_Sadrzaj.md
[0110]: 0110_Goroutine_Odloženi_Funkcijski_pozivi_i_Panic-Recover.md
