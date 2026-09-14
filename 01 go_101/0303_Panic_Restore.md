
# Detaljno o mehanizmu panic/restore

[Više o odloženim funkcijskim pozivima][0302] | [Sadržaj][00] | [Blokovi koda i opsezi identifikatora][0304]

Mehanizam panike i oporavka je već predstavljen, a nekoliko slučajeva upotrebe panike/oporavka je prikazano u poslednjem članku. Ovaj članak će detaljno objasniti mehanizam panike/oporavka. Izlazne faze poziva funkcija će takođe biti detaljno objašnjene.

## Izlazne faze poziva funkcija

U jeziku Go, poziv funkcije može proći kroz fazu izlaska pre nego što se potpuno završi. U fazi izlaska, odloženi pozivi funkcija koji su ubačeni na stek odloženih poziva tokom izvršavanja poziva funkcije biće izvršeni (inverznim redosledom u odnosu na ubacivanje). Kada se svi odloženi pozivi potpuno završe, faza izlaska se završava i poziv funkcije se takođe potpuno završava.

Izlazne faze se takođe mogu nazvati fazama povratka negde drugde.

Poziv funkcije može ući u svoju izlaznu fazu (ili direktno izaći) na tri načina:

- nakon što se poziv vrati normalno.
- kada se u pozivu pojavi panika.
- nakon što je **runtime.Goexit** funkcija pozvana i potpuno se završi u pozivu.

Na primer, u sledećem isečku koda,

- poziv funkcije f0 ili f1 će ući u svoju postojeću fazu nakon što se normalno vrati.
- poziv funkcije f2 će ući u svoju izlaznu fazu nakon što se desi panika deljenja sa nulom.
- poziv funkcije f3 će ući u svoju fazu izlaska nakon što se **runtime.Goexit** poziv funkcije potpuno završi.

```go
import (
    "fmt"
    "runtime"
)

func f0() int {
    var x = 1
    defer fmt.Println("exits normally:", x)
    x++
    return x
}

func f1() {
    var x = 1
    defer fmt.Println("exits normally:", x)
    x++
}

func f2() {
    var x, y = 1, 0
    defer fmt.Println("exits for panicking:", x)
    x = x / y // will panic
    x++       // unreachable
}

func f3() int {
    x := 1
    defer fmt.Println("exits for Goexiting:", x)
    x++
    runtime.Goexit()
    return x+x // unreachable
}
```

Uzgred, **runtime.Goexit()** funkcija nije namenjena za poziv u glavnoj gorutini programa.

## Povezivanje signala panike i izlaza sa pozivima funkcija

Kada se panika javi direktno u pozivu funkcije, kažemo da (neoporavljena) panika počinje da se povezuje sa pozivom funkcije. Slično tome, kada runtime.Goexitse funkcija pozove u pozivu funkcije, kažemo da Goexit signal počinje da se povezuje sa pozivom funkcije nakon što runtime.Goexitse poziv potpuno završi. Kao što je objašnjeno u poslednjem odeljku, povezivanje panike ili Goexit signala sa pozivom funkcije će dovesti do toga da poziv funkcije odmah uđe u fazu izlaska.

Naučili smo da se panike mogu obnoviti. Međutim, ne postoje načini da se otkaže Goexit signal.

U bilo kom trenutku, poziv funkcije može biti povezan sa najviše jednom nepreuzrokovanom panikom. Ako je poziv povezan sa nepreuzrokovanom panikom, onda

- poziv će biti povezan sa nikakvim panikama kada se oporavi neotkrivena panika.
- kada se u pozivu funkcije pojavi nova panika, nova će zameniti staru i postati pridružena neoporavljena panika poziva funkcije.

Na primer, u sledećem programu, oporavljena panika je panika 3, što je poslednja panika povezana sa mainpozivom funkcije.

```go
package main

import "fmt"

func main() {
    defer func() {
        fmt.Println(recover()) // 3
    }()
    
    defer panic(3) // will replace panic 2
    defer panic(2) // will replace panic 1
    defer panic(1) // will replace panic 0
    panic(0)
}
```

Pošto se Goexit signali ne mogu otkazati, raspravljanje o tome da li poziv funkcije može da se poveže sa najviše jednim ili više Goexit signala je nepotrebno.

Iako je neuobičajeno, u gorutini može istovremeno postojati više neoporavljenih panika. Svaka od njih je povezana sa jednim pozivom funkcije koja još nije završena u steku poziva gorutine. Kada se ugnježdeni poziv potpuno završi, a i dalje je povezan sa neoporavljenom panikom, neoporavljena panika će se proširiti na ugnježdeni poziv (pozivaoca ugnježdenog poziva). Efekat je isti kao kada se panika pojavi direktno u ugnježdenom pozivu. To znači,

- Ako je ranije postojala stara, neoporavljena panika povezana sa ugnežđenim pozivom, stara će biti zamenjena onom sa širenjem. U ovom slučaju, ugnežđeni poziv je sigurno već morao da uđe u svoju izlaznu fazu, tako da će se pozvati sledeći odloženi poziv funkcije u njegovom steku odloženih poziva.

- Ako prethodno nije bilo neoporavljene panike povezane sa pozivom za ugnežđenje, širenje će se povezati sa pozivom za ugnežđenje. U ovom slučaju, poziv za ugnežđenje je možda ušao u svoju izlaznu fazu ili ne. Ako nije, odmah će ući u izlaznu fazu.

Dakle, kada gorutina završi sa izlaskom, može postojati najviše jedna neoporavljena panika u gorutini. Ako gorutina završi sa neoporavljenom panikom, ceo program se ruši i biće prijavljene informacije o neoporavljenoj panici. U suprotnom, gorutina se izlazi normalno (mirno).

Kada se funkcija pozove, ne postoji ni panika niti Goexit signali povezani sa njenim pozivom na početku, bez obzira da li je njen pozivalac (ugnežđeni poziv) ušao u fazu izlaska ili ne. Svakako, panike se mogu pojaviti ili funkcija runtime.Goexitmože biti pozvana kasnije u procesu izvršavanja poziva, tako da se panike i Goexit signali mogu kasnije povezati sa pozivom.

Sledeći primer programa će se srušiti ako se pokrene, jer panika 2 još uvek nije oporavljena kada se nova gorutina završi.

```go
package main

func main() {
    // The new goroutine.
    go func() {
        // This is an anonymous deferred call.
        // When it fully exits, the panic 2 will spread
        // to the entry function call of the new
        // goroutine, and replace the panic 0. The
        // panic 2 will never be recovered.
        defer func() {
            // As explained in the last example,
            // panic 2 will replace panic 1.
            defer panic(2)
            
            // When the anonymous function call fully
            // exits, panic 1 will spread to (and
            // associate with) the nesting anonymous
            // deferred call.
            func () {
                // Once the panic 1 occurs, there will
                // be two unrecovered panics coexisting
                // in the new goroutine. One (panic 0)
                // associates with the entry function
                // call of the new goroutine, the other
                // (panic 1) associates with the
                // current anonymous function call.
                panic(1)
            }()
        }()
        panic(0)
    }()
    select{}
}
```

Izlaz (kada se gornji program kompajlira standardnim Go kompajlerom v1.25.n):

```sh
panika: 0
    panika: 1
    panika: 2
...
```

Format izlaza nije savršen, sklon je da navede neke ljude da pomisle da je panika 0 poslednja neoporavljena panika, dok je poslednja neoporavljena panika zapravo panika 2.

Slično tome, kada ugnežđeni poziv potpuno izađe i povezuje se sa Goexit signalom, tada će se Goexit signal takođe proširiti na (i povezati se sa) ugnežđeni poziv. Ovo će učiniti da ugnežđeni poziv odmah uđe (ako već nije ušao) u svoju izlaznu fazu.

Kada se Goexit signal poveže sa pozivom funkcije, ako se poziv funkcije povezuje sa neotkrivenom panikom, onda će se panika oporaviti (iskreno, meni je to čudno). Na primer, sledeći program će se mirno završiti, jer byeće Goexit signal oporaviti paniku.

```go
package main
import (
    "runtime"
)

func f() {
    // The Goexit signal will disable the "bye" panic.
    defer runtime.Goexit()
    panic("bye")
}

func main() {
    go f()
    
    for runtime.NumGoroutine() > 1 {
        runtime.Gosched()
    }
}
```

## Neki recover pozivi nisu obavezni

Ugrađena recoverfunkcija mora biti pozvana na odgovarajućim mestima da bi imala efekat. U suprotnom, pozivi su neobavljajući. Na primer, nijedan od **recover** poziva u sledećem primeru ne obnavlja bye paniku.

```go
package main

func main() {
    defer func() {
        defer func() {
            recover() // no-op
        }()
    }()
    defer func() {
        func() {
            recover() // no-op
        }()
    }()
    func() {
        defer func() {
            recover() // no-op
        }()
    }()
    func() {
        defer recover() // no-op
    }()
    func() {
        recover() // no-op
    }()
    recover()       // no-op
    defer recover() // no-op
    panic("bye")
}
```

Već znamo da sledeći recover poziv stupa na snagu.

```go
package main

func main() {
    defer func() {
        recover() // take effect
    }()

    panic("bye")
}
```

Zašto onda ti **recover** pozivi u prvom primeru trenutnog odeljka ne stupaju na snagu? Hajde da pročitamo trenutnu verziju Go specifikacije :

Vraćena vrednost recoverje nilako je ispunjen bilo koji od sledećih uslova:

- panic argument je bio nikakav;
- gorutina ne paniči;
- recover nije direktno pozvana odloženom funkcijom.

U poslednjem članku je primer koji prikazuje prvi uslovni slučaj.

Većina recoverpoziva u prvom primeru trenutnog odeljka zadovoljava ili drugi ili treći uslov pomenut u Go specifikaciji, osim prvog poziva. Da, ovde trenutni opisi još uvek nisu precizni. Treći uslov treba opisati kao

- **recover** nije direktno pozvan odloženim pozivom funkcije koja je direktno pozvana pozivom funkcije koja se povezuje sa očekivanom panikom koja treba da se oporavi.

U prvom primeru trenutnog odeljka, očekivana panika koja se treba oporaviti povezana je sa mainpozivom funkcije. Prvi recoverpoziv se poziva direktno odloženim pozivom funkcije, ali odloženi poziv funkcije se ne poziva direktno mainpozivom funkcije. Zbog toga recoverje prvi poziv neizvršen.

U stvari, trenutna Go specifikacija takođe ne objašnjava dobro zašto drugi recoverpoziv (po redosledu linija koda), za koji se očekuje da oporavi paniku 1, u sledećem primeru ne stupa na snagu.

```go
// This program exits without recovering panic 1.
package main

func demo() {
    defer func() {
        defer func() {
            recover() // this one recovers panic 2
        }()

        defer recover() // no-op

        panic(2)
    }()
    panic(1)
}

func main() {
    demo()
}
```

Ono što Go specifikacija ne pominje jeste da se svaki poziv **recover** posmatra kao pokušaj oporavka najnovije neotkrivene panike u trenutnoj goroutine.

Go runtime smatra da drugi **recover** poziv u gornjem primeru pokušava da oporavi najnoviju neoporavljenu paniku, paniku 2, koja je povezana sa pozivaocem drugog **recover** poziva. Drugi **recover** poziv se ne poziva direktno odloženim pozivom funkcije koji se poziva pozivom funkcije koja ga je pozvala. Umesto toga, direktno se poziva pozivom funkcije koja ga je pozvala. Zbog toga je drugi **recover** poziv neuspešan.

## Rezime

U redu, sada, hajde da pokušamo da napravimo kratak opis koji recoverće pozivi stupiti na snagu:

Poziv **recover** stupa na snagu samo ako je direktni pozivalac poziva **recover** odloženi poziv i ako se direktni pozivalac odloženog poziva povezuje sa najnovijom neotkrivenom panikom u trenutnoj gorutini. Efektivan **recover** poziv odvaja najnoviju neotkrivenu paniku od poziva funkcije koja je sa njom povezana i vraća vrednost prosleđenu pozivu **panic** koji je proizveo najnoviju neotkrivenu paniku.

[Više o odloženim funkcijskim pozivima][0302] | [Sadržaj][00] | [Blokovi koda i opsezi identifikatora][0304]  

[0302]: 0302_Više_o_odloženim_funkcijskim_pozivima.md
[00]: 00_Sadrzaj.md
[0304]: 0304_Blokovi_koda_i%20_opsezi_identifikatora.md
