
# Više o odloženim pozivima funkcija

[Pravila za prelom reda][0301] | [Sadržaj][00] | [Panic i restore][0303]

Odloženi pozivi funkcija su već predstavljeni. Zbog ograničenog znanja o jeziku Go u to vreme, neki detalji i slučajevi upotrebe odloženih poziva funkcija nisu obrađeni u tom članku. Ovi detalji i slučajevi upotrebe biće obrađeni u ostatku ovog članka.

## Pozivi mnogih ugrađenih funkcija sa povratnim rezultatima ne mogu biti odloženi

U programskom jeziku Go, vrednosti rezultata poziva prilagođenih funkcija mogu biti sve odsutne (odbačene). Međutim, za ugrađene funkcije sa listama rezultata koje nisu prazne, vrednosti rezultata njihovih poziva ne smeju biti odsutne, osim poziva ugrađenih funkcija **copy** i **recover**. S druge strane, naučili smo da vrednosti rezultata odloženog poziva funkcije moraju biti odbačene, tako da pozivi mnogih ugrađenih funkcija ne mogu biti odloženi.

Srećom, potrebe za odlaganjem poziva ugrađenih funkcija (sa listama rezultata koje nisu prazne) su retke u praksi. Koliko ja znam, ponekad je potrebno odložiti samo pozive ugrađene funkcije **append**. U ovom slučaju, možemo odložiti poziv anonimne funkcije koja završava poziv **append**.

```go
package main
import "fmt"

func main() {
    s := []string{"a", "b", "c", "d"}
    
    defer fmt.Println(s) // [a x y d]
    // defer append(s[:1], "x", "y") // error
    
    defer func() {
        _ = append(s[:1], "x", "y")
    }()
}
```

## Moment evaluacije odloženih vrednosti funkcije

Pozvana funkcija (vrednost) u odloženom pozivu funkcije se izračunava kada se poziv ubaci u stek odloženih poziva trenutne gorutine. Na primer, sledeći program će ispisati false.

```go
package main
import "fmt"

func main() {
    var f = func () {
        fmt.Println(false)
    }
    defer f()
    f = func () {
        fmt.Println(true)
    }
}
```

Pozvana funkcija u odloženom pozivu funkcije može biti vrednost funkcije **nil**. U takvom slučaju, panika će se javiti kada se pozove poziv funkcije nil, umesto kada se poziv pošalje na stek odloženih poziva trenutne gorutine. Primer:

```go
package main
import "fmt"

func main() {
    defer fmt.Println("reachable 1")
    
    var f func() // f is nil by default
    defer f()    // panic here
    // The following lines are also reachable.
    
    fmt.Println("reachable 2")
    f = func() {} // useless to avoid panicking
}
```

## Moment evaluacije argumenata prijemnika odloženih poziva metoda

Kao što je već objašnjeno, argumenti odloženog poziva funkcije se takođe izračunavaju kada se odloženi poziv ubaci u stek odloženih poziva trenutne gorutine.

Argumenti prijemnika metode takođe nisu izuzeci. Na primer, sledeći program ispisuje 1342.

```go
package main

type T int

func (t T) M(n int) T {
  print(n)
  return t
}

func main() {
    var t T
    // "t.M(1)" is the receiver argument of the method
    // call ".M(2)", so it is evaluated when the
    // ".M(2)" call is pushed into defer-call stack.
    defer t.M(1).M(2)
    t.M(3).M(4)
}
```

Odloženi pozivi čine kod čistijim i manje podložnim greškama

Primer:

```go
import "os"

func withoutDefers(filepath string, head, body []byte) error {
    f, err := os.Open(filepath)
    if err != nil {
        return err
    }

    _, err = f.Seek(16, 0)
    if err != nil {
        f.Close()
        return err
    }

    _, err = f.Write(head)
    if err != nil {
        f.Close()
        return err
    }

    _, err = f.Write(body)
    if err != nil {
        f.Close()
        return err
    }

    err = f.Sync()
    f.Close()
    return err
}

func withDefers(filepath string, head, body []byte) error {
    f, err := os.Open(filepath)
    if err != nil {
        return err
    }
    defer f.Close()

    _, err = f.Seek(16, 0)
    if err != nil {
        return err
    }

    _, err = f.Write(head)
    if err != nil {
        return err
    }

    _, err = f.Write(body)
    if err != nil {
        return err
    }

    return f.Sync()
}
```

Koji izgleda čistije? Izgleda da je to onaj sa odloženim pozivima, mada malo. I manje je sklonan greškama, jer **f.Close()** u funkciji ima toliko poziva bez odloženih poziva da je veća verovatnoća da će se jedan od njih propustiti.

Sledi još jedan primer koji pokazuje da odloženi pozivi mogu učiniti kod manje podložnim greškama. Ako pozivi *doSomething* u sledećem primeru izazovu paniku, funkcija *f2* će se završiti bez otključavanja *Mutex* vrednosti. Dakle, funkcija *f1* je manje podložna greškama.

```go
var m sync.Mutex

func f1() {
    m.Lock()
    defer m.Unlock()
    doSomething()
}

func f2() {
    m.Lock()
    doSomething()
    m.Unlock()
}
```

## Gubici performansi uzrokovani odlaganjem poziva funkcija

Nije uvek dobro koristiti odložene pozive funkcija. Za zvanični Go kompajler, pre verzije 1.13, odloženi pozivi funkcija će izazvati nekoliko gubitaka performansi tokom izvršavanja. Od Go Toolchain 1.13, neki uobičajeni slučajevi upotrebe odlaganja su znatno optimizovani, tako da generalno ne moramo da brinemo o problemu gubitka performansi izazvanog odloženim pozivima. Hvala Denu Skejlsu na odličnim optimizacijama.

### Vrsta curenja resursa odlaganjem poziva funkcija

Veoma veliki stek odloženih poziva takođe može trošiti mnogo memorije, a neki resursi se možda neće osloboditi na vreme ako se neki pozivi previše odlože.

Na primer, ako je potrebno obraditi mnogo datoteka u pozivu sledeće funkcije, onda veliki broj obrađivača datoteka neće biti oslobođen pre nego što se funkcija završi.

```go
func writeManyFiles(files []File) error {
    for _, file := range files {
        f, err := os.Open(file.path)
        if err != nil {
            return err
        }
        defer f.Close()

        _, err = f.WriteString(file.content)
        if err != nil {
            return err
        }

        err = f.Sync()
        if err != nil {
            return err
        }
    }

    return nil
}
```

U takvim slučajevima, možemo koristiti anonimnu funkciju da bismo obuhvatili odložene pozive tako da se odloženi pozivi funkcija izvršavaju ranije. Na primer, gornja funkcija se može prepisati i poboljšati kao

```go
func writeManyFiles(files []File) error {
    for _, file := range files {
        if err := func() error {
            f, err := os.Open(file.path)
            if err != nil {
                return err
            }
            // The close method will be called at
            // the end of the current loop step.
            defer f.Close()

            _, err = f.WriteString(file.content)
            if err != nil {
                return err
            }

            return f.Sync()
        }(); err != nil {
            return err
        }
    }
    return nil
}
```

[Pravila za prelom reda][0301] | [Sadržaj][00] | [Panic i restore][0303]

[0301]: 0301_Pravila%20za%20prelom%20reda.md
[00]: 00_Sadrzaj.md
[0303]: 0303_Panic_Restore.md
