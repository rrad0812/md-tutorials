
# Generici

[Nebezbedni pokazivači][0212] | [Sadržaj][00] | [Refleksije][0214]

Pre verzije Go 1.18, Go je podržavao samo ugrađene generike. Od verzije Go 1.18, Go takođe podržava prilagođene generike. Ovaj članak predstavlja samo ugrađene generike.

Ugrađeni generički tipovi u Go-u su podržani kroz prvoklasne kompozitne tipove. Možemo koristiti kompozitne tipove za kreiranje beskonačnih prilagođenih tipova korišćenjem kompozitnih tipova. Ovaj članak će pokazati neke primere kompozicije tipova i objasniti kako čitati ove kompozitne tipove.

**Primeri kompozitnih tipova**
Kompozicije tipova u Go jeziku su dizajnirane veoma intuitivno i lako za tumačenje. Teško je izgubiti se u razumevanju kompozitnih tipova u Gou, čak i kada su u pitanju neki veoma složeni. U nastavku će biti navedeno nekoliko primera kompozicija tipova, od jednostavnijih do složenijih.

Hajde da pogledamo jednostavan literal kompozitnog tipa.

```go
[3][4]int
```

Prilikom tumačenja kompozitnog tipa, trebalo bi da ga posmatramo sleva nadesno. [3] sa leve strane u gornjem literalu tipa označava da je ovaj tip tip niza. Čitav desni deo nakon je [4]int drugi tip niza, koji je tip elementa prvog tipa niza. Tip elementa tipa elementa (tip niza) prvog tipa niza je ugrađeni tip int. Prvi tip niza može se posmatrati kao dvodimenzionalni tip niza.

Primer korišćenja ovog dvodimenzionalnog tipa niza.

```go
package main

import (
    "fmt"
)

func main() {
    matrix := [3][4]int{
        {1, 0, 0, 1},
        {0, 1, 0, 1},
        {0, 0, 1, 1},
    }

    matrix[1][1] = 3
    a := matrix[1] // type of a is [4]int
    fmt.Println(a) // [0 3 0 1]
}
```

Slično tome,

- **[][]string** je tip isečka čiji je tip elementa drugi tip isečka []string.

- **\*\*bool** je tip pokazivača čiji je osnovni tip drugi tip pokazivača *bool.

- **chan chan int** je tip kanala čiji je tip elementa drugi tip kanala chan int.

- **map[int]map[int]string** je tip mape čiji je tip elementa drugi tip mape map[int]string. Ključni tipovi oba tipa mape su int.

- **func(int32) func(int32)** je tip funkcije čiji je jedini tip povratnog rezultata drugi tip funkcije func(int32). Oba tipa funkcija imaju samo jedan ulazni parametar tipa int32.

Hajde da pogledamo drugi tip.

```go
chan *[16]byte
```

- Ključna reč **chan** skroz levo označava da je ovaj tip, tip kanala.
- Čitav desni deo *[16]byte, koji je tip pokazivača, označava tip elementa ovog tipa kanala.
- Osnovni tip tipa pokazivača je [16]byte, koji je tip niza.
- Tip elementa tipa niza je byte.

**Primer korišćenja ovog tipa kanala**:

```go
package main

import (
    "fmt"
    "time"
    "crypto/rand"
)

func main() {
    c := make(chan *[16]byte)

    go func() {
        // Use two arrays to avoid data races.
        var dataA, dataB = new([16]byte), new([16]byte)
        for {
            _, err := rand.Read(dataA[:])
            if err != nil {
                close(c)
            } else {
                c <- dataA
                dataA, dataB = dataB, dataA
            }
        }
    }()

    for data := range c {
        fmt.Println((*data)[:])
        time.Sleep(time.Second / 2)
    }
}
```

- Slično tome, tip **map[string] []func(int) int** je tip mape.
- Tip ključa ovog tipa mape je **string**.
- Preostali desni deo **[]func(int) int** značava tip elementa tipa mape, označava da je tip elementa mape tip isečka,
- čiji je tip elementa tip funkcije **func(int) int**.

**Primer korišćenja upravo objašnjenog tipa mape**:

```go
package main

import "fmt"

func main() {
    addone := func(x int) int {return x + 1}
    square := func(x int) int {return x * x}
    double := func(x int) int {return x + x}

    transforms := map[string][]func(int) int {
        "inc,inc,inc": {addone, addone, addone},
        "sqr,inc,dbl": {square, addone, double},
        "dbl,sqr,sqr": {double, double, square},
    }

    for _, n := range []int{2, 3, 5, 7} {
        fmt.Println(">>>", n)
        for name, transfers := range transforms {
            result := n
            for _, xfer := range transfers {
                result = xfer(result)
            }
            fmt.Printf(" %v: %v \n", name, result)
        }
    }
}
```

Ispod je tip koji izgleda pomalo složeno.

```go
[]map[struct {
    a int
    b struct {
        x string
        y bool
    }
}]interface {
    Build([]byte, struct {x string; y bool}) error
    Update(dt float64)
    Destroy()
}
```

Hajde da pročitamo s leva na desno:

- Početak **[]** krajnje leve strane ukazuje da je ovaj tip tip isečka.
- Sledeća ključna reč **map** pokazuje da je tip elementa tipa isečka tip **map**.
- Tip **struct** označen literalom strukture u sledećoj ključnoj je **key** tip tipa **map**.
- Tip elementa **map** tipa je tip **interfejsa** koji određuju tri metode.
- Key tip, tip struct, ima dva polja, jedno polje a je int tipa, a drugo polje b je struct tipa  **struct {x string; y bool}**.

Imajte na umu da se drugi tip strukture takođe koristi kao jedan tip parametra jedne metode koju određuje upravo pomenuti tip interfejsa.

Radi bolje čitljivosti, često razlažemo takav tip na više deklaracija tipa. Aliasi tipa deklarisani u sledećem kodu i upravo objašnjeni tip iznad označavaju identičan tip.

```go
type B = struct {
    x string
    y bool
}

type X = struct {
    a int
    b B
}

type E = interface {
    Build([]byte, B) error
    Update(dt float64)
    Destroy()
}

type T = []map[K]E
```

## Ugrađene generičke funkcije

Pored ugrađenih generičkih funkcija za kompozitne tipove, postoji nekoliko ugrađenih funkcija koje takođe podržavaju generike, kao što je ugrađena **len** funkcija, može se koristiti za dobijanje  vrednosti dužine nizova, isečaka, mapa, stringova i kanala. Generalno, funkcije u **unsafe** standardnom paketu se takođe smatraju ugrađenim funkcijama. Ugrađene generičke funkcije su predstavljene u prethodnim člancima,

## Prilagođeni generički proizvodi

Od verzije 1.18, Go već podržava prilagođene generike. Molimo vas da pročitate knjigu „Go Generics 101“ da biste saznali kako se koriste prilagođeni generici.

[Nebezbedni pokazivači][0212] | [Sadržaj][00] | [Refleksije][0214]

[0212]: 0212_Nebezbedni_pokazivači.md
[00]: 00_Sadrzaj.md
[0214]: 0214_Refleksije.md
