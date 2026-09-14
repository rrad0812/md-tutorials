
# 20 Nizovi

[19 Jedinični testovi][19] | [00 Sadržaj][00] | [21 Isečci][21]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je niz?
- Kako kreirati niz.
- Kako kreirati višedimenzionalne nizove.
- Kako iterirati kroz niz.
- Kako pronaći element u nizu.

**Obrađeni tehnički koncepti!**

- niz
- tip elementa
- dužina
- kapacitet
- dimenzija nizova

## Definicija

- Niz je kolekcija elemenata istog tipa.
- Tip elemenata unutar niza naziva se tip elementa.
- Ova kolekcija ima fiksni broj elemenata poznatih u vreme kompajliranja.

## Primer upotrebe

Recimo da želite da sačuvate ime meseca. Možete koristiti niz stringova. Broj meseci je poznat u vreme kompajliranja.

## Kreiranje niza

Evo primera kako se definiše niz u go programu:

```go
// array/creation/main.go
package main

import "fmt"

func main() {
    var myArray [2]int
    myArray[0] = 156
    myArray[1] = 147
    fmt.Println(myArray)
    // output : [156 147]
}
```

- Prethodni program je u prvom redu definisao promenljivu pod nazivom "myArray"
  koja je niz celih
  brojeva ( int ) i dužine 2.
- Drugim rečima, ovo je kolekcija dva elementa tipa int.
- Zatim postavljamo prvi element izrazom myArray[0] = 156. 0 (nula) predstavlja
  indeks prvog elementa niza myArray. Elementi su indeksirani od 0.
- Nakon toga, drugom elementu niza dodeljujemo vrednost 147. A zatim ispisujemo
  naš niz.

### Sažetiji način

Korišćenje prethodne metode za definisanje niza je ispravno, ali nije baš zgodno za pisanje. Koristili smo tri reda za definisanje niza. Postoji lakši metod: korišćenje literala niza.

```go
myArray := [2]int{156, 147}
fmt.Println(myArray)
// output : [156 147]
```

U prethodnim linijama koda, definišemo niz od 2 cela broja i direktno ga popunjavamo korišćenjem literala niza `[n]T{}`

Takođe možete dozvoliti kompajleru da zaključi veličinu našeg niza koristeći sintaksu `[...]T{}`. Na primer:

```go
myArray := [...]int{156, 147}
```

### Rezime

```go
// long way
var a [2]int
a[0] = 156
a[1] = 147
fmt.Println(a)

// more concise
b := [2]string{"FR", "US"}
fmt.Println(b)

// size computed by the compiler
c := [...]float64{13.2, 37.2}
fmt.Println(c)

// values not set (yet)
d := [2]int{}
fmt.Println(d)
```

## Nulta vrednost

Kada definišete niz, ne morate znati sve vrednosti koje želite da u njega stavite.

Možete kreirati prazan niz i popuniti ga tokom izvršavanja programa. Ali zapamtite da u Go-u nema nedefinisanih elemenata.

Kada definišete prazan niz, go će ga popuniti nultom vrednošću tipa elementa:

```go
// array/zero-value/main.go
package main

import "log"

func main() {
    var myA [3]int
    log.Printf("%v", myA)
}
```

Ispisaće:

```sh
2021/01/27 17:56:21 [0 0 0]
```

Ovde definišemo niz celih brojeva dužine 3. Pošto ne navodimo nikakvu vrednost unutar vitičastih zagrada, go ga popunjava nultom vrednošću celih brojeva, što je "0". Imajte na umu da je ponašanje isto ako definišete niz stringova bez navođenja vrednosti:

```go
myEmptyArray2 := [2]string{}
fmt.Println(myEmptyArray2)
// output : []
```

Go će staviti prazan string u prvi element kolekcije myEmptyArray2[0]. i u drugi element kolekcijemyEmptyArray2[1].

## Ugrađene funkcije

### Len

`len` je ugrađena funkcija u Go-u koja vraća dužinu promenljive v. Dužina niza je broj elemenata unutar njega. Označimo v kao niz tipa T, tada je dužina:

```go
len(v)
```

### Ograničenje: Kapacitet

Ugrađena `cap` funkcija vraća isto što i `len` funkcija. Nije korisna u kontekstu nizova, ali će biti veoma zgodna prilikom manipulacije isečcima.

## Pristup elementima niza

Nizovi su zgodan i efikasan način za čuvanje podataka sa dužinom poznatom u vreme kompajliranja. Ali ako nešto i čuvamo, to je za kasnije čitanje. Kada želite da pročitate određeni element niza, možete mu pristupiti direktno preko njegovog indeksa:

```go
// array/access-element/main.go
package main

import "fmt"

func main() {
    b := [2]string{"FR", "US"}
    firstElement := b[0]
    secondElement := b[1]
    fmt.Println(firstElement, secondElement)
}
```

U ovom primeru, pristupamo drugom elementu niza "b" koristeći index notaciju "b[1]".

### Indeks van granica niza

Morate biti sigurni da je indeks koji navodite (ovde 1) definisan u nizu. Ako želite da dobijete 101. element niza, nećete moći da kompajlirate svoj program. Pokušavate da pristupite vrednosti nečega što ne postoji. Evo kako kompajler prikazuje grešku:

```sh
invalid array index 100 (out of bounds for 2-element array)
```

## Iteracija kroz niz

### Petlja `for` sa klauzulom `range`

Petlja for vam omogućava da iterativno skenirate sve elemente niza:

```go
myArray := [2]int{156, 147}

for index, element := range myArray {
    fmt.Printf("element at index %d is %d\n", index, element)
}
// output :
//element at index 0 is 156
//element at index 1 is 147
```

Petlja `for` u kombinaciji sa ključnom reči `range` se koristi za iteraciju kroz elemente niza.

Trenutni indeks ćete dobiti u svakoj iteraciji pomoću sledeće sintakse:

for index, element := range myArray

### Ignorišite indeks (ili vrednost)

Ponekad vas zanima samo vrednost, a ne indeksi. Možete koristiti sledeću sintaksu da biste ignorisali indeks:

```go
for _, element := range myArray {
    fmt.Println(element)
}
```

`_` je prazan identifikator. Zahvaljujući ovom identifikatoru, program ignoriše indeks. Imajte na umu da možete učiniti isto za vrednost :

```go
for index, _ := range myArray {
    fmt.Println(index)
}
```

Ovaj slučaj upotrebe je redak.

### Petlja petlja na nizu

Prethodna metoda je zanimljiva ako želite da počnete od početne iteracije. Ali šta ako želite da počnete od određenog indeksa i zaustavite se na drugom. Recimo da imam tabelu sa pet elemenata i želim da počnem od elementa na indeksu dva i zaustavim se na elementu na indeksu 4 (peti element)?

Možemo koristiti petlju `for` sa promenljivom koja će služiti kao brojač.

```go
myArray2 := [2]int{156, 147}
for i := 0; i < len(myArray2); i++ {
    fmt.Printf("element at index %d is %d\n", i, myArray2[i])
}
// output :
//element at index 0 is 156
//element at index 1 is 147
```

Da biste iterativno pregledali niz od indeksa 0 do poslednjeg indeksa, koristite sledeću sintaksu:

```go
for i := 0; i < len(a); i++ {
    fmt.Println(i, a[i])
}
```

Da biste iterirali u opadajućem redosledu (od poslednjeg elementa do prvog):

```go
for i := len(a) - 1; i >= 0; i-- {
    fmt.Println(i, a[i])
}
```

### Kako pronaći element unutar niza pomoću for petlje

Da biste pronašli element unutar niza, nemate drugog izbora nego da proverite svaki njegov element:

```go
// getIndex will find an element (needle) inside an array (haystack)
// if not found the function returns -1
func getIndex(haystack [10]int, needle int) int {
    for index, element := range haystack {
        if element == needle {
            return index
        }
    }
    return -1
}
```

Ova funkcija nije savršena. Zamislite da se element koji tražite nalazi na kraju niza. Vaša funkcija će iterirati kroz svaki element niza pre nego što pronađe indeks.

Još jedna mana: ova funkcija je specifična za nizove od 10 elemenata!

## Poređenje nizova

Možete uporediti dva niza koja imaju:

- potpuno isti tip elementa (na primer, celi brojevi)
- potpuno iste dužine.

A jedini operatori poređenja koji su dozvoljeni za nizove su `==` (jednako) i `!=` (nije jednako):

```go
a := [2]int{1, 2}
b := [2]int{1, 2}
if a == b {
    print("equal")
} else {
    print("not equal")
}

// output : equal
```

## Kako proslediti nizove funkcijama

- Ovo je laka greška: prosleđivanje niza funkciji će kopirati niz.
- Kada želite da izmenite niz u funkciji, prosledite pokazivač na niz
- Pažnja! Ako je vaš niz veliki, pravljenje kopija može uticati na performanse vašeg programa.

### Demonstracija

Evo primera paketa:

```go
// array/demo/demo.go
package demo

const NewValue = "changedValue"

func UpdateArray1(array [2]string) {
    array[0] = NewValue
}
```

Funkcija UpdateArray1 uzima parametar tipa [2]string. Funkcija će izmeniti vrednost na indeksu 0. Hajde da ovo testiramo:

```go
// array/demo/demo_test.go
package demo

import (
    "testing"

    "github.com/stretchr/testify/assert"
)

func TestUpdateArray1(t *testing.T) {
    testArray := [2]string{"Value1", "Value2"}
    UpdateArray1(testArray)
    assert.Equal(t, NewValue, testArray[0])
}
```

U ovom jediničnom testu kreiramo novi niz dužine 2 "testArray".

Pokrećemo funkciju "UpdateArray1". Nakon poziva funkcije, proveravamo da li je element na indeksu 0 jednak konstanti "NewValue". Pokrenimo ovaj test:

```sh
$ go test./...
--- FAIL: TestUpdateArray1 (0.00s)
    demo_test.go:11:
                Error Trace:    demo_test.go:11
                Error:          Not equal:
                                expected: "changedValue"
                                actual  : "Value1"

                                Diff:
                                --- Expected
                                +++ Actual
                                @@ -1 +1 @@
                                -changedValue
                                +Value1
                Test:           TestUpdateArray1
FAIL
FAIL    maximilien-andile.com/array/copy/demo   0.016s
FAIL
```

Funkcija "UpdateArray1" će primiti kopiju "testArray". Funkcija menja kopiju, a ne originalni niz.

### Rešenje: pokazivač

Da bi izmenila originalni niz, funkcija treba da uzme element tipa *[2]string. Ovo je pokazivač tipa [2]string. On signalizira da funkcija može da uzme bilo koji pokazivač na vrednosti tipa ( što se naziva osnovni tip ). Pokazivač je adresa koja ukazuje na prostor u memoriji.

```go
// array/demo/demo.go 
package demo

const NewValue = "changedValue"

func UpdateArray2(array *[2]string) {
    array[0] = NewValue
}
```

Ovo je druga verzija funkcije koja sada prihvata tip pokazivača. Hajde da testiramo da li ova funkcija radi:

```go
// array/demo/demo_test.go 
package demo

import (
    "testing"

    "github.com/stretchr/testify/assert"
)

func TestUpdateArray2(t *testing.T) {
    testArray := [2]string{"Value1", "Value2"}
    UpdateArray2(&testArray)
    assert.Equal(t, NewValue, testArray[0])
}
```

Kreiramo novi niz stringova testArray. Inicijalizujemo dve vrednosti niza. Nakon poziva funkcije, testiramo da li je element na indeksu 0 jednak konstanti NewValue. Pokrenimo test:

```go
$ go test./...
ok      maximilien-andile.com/array/copy/demo   0.015s
```

## Kako kopirati niz

Kakav će biti rezultat ovog jediničnog testa?

```go
// array/demo/demo_test.go 
func TestArrayCopy(t *testing.T) {
    testArray := [2]string{"Value1", "Value2"}
    newCopy := testArray
    testArray[1] = "updated"
    assert.Equal(t, "updated", newCopy[1])
}
```

Ovaj test ne prolazi. Vrednost newCopy[1] nije jednaka testArray[1]. Zašto? To je zato što će se niz kopirati kada kreiramo promenljivu "newCopy". U memoriji ćemo imati dva odvojena niza "newCopy" i "TestArray". Kada modifikujete testArray, ne modifikujete newCopy.

## Kako uzeti adresu niza

U sledećem testu, ne kopiramo niz, već kreiramo novu promenljivu tipa (pokazivač na niz od 2 stringa):*[2]string

```go
// array/demo/demo_test.go 
func TestArrayReference(t *testing.T) {
    testArray := [2]string{"Value1", "Value2"}
    reference := &testArray
    reference[1] = "updated"
    assert.Equal(t, "updated", testArray[1])
}
```

"reference" je pokazivač na originalni niz "testArray". Kada pišemo interno reference[1], Go će preuzeti vrednost sačuvanu u testArray[1].

## Dvodimenzionalni nizovi

Termin "dimenzija" može izazvati anksioznost kod onih koji mrze matematiku. Ne treba da se plašite; koncept koji stoji iza njega je veoma jednostavan.

U prethodnom primeru, sačuvali smo cele brojeve u naš niz. Kada sačuvate stringove, cele brojeve, brojeve sa pokretnim decimalom, bulove vrednosti, bajtove, kompleksne brojeve u niz, dimenzija ovog niza je jedan.

Kada su vrednosti našeg niza drugi niz, imamo višedimenzionalni niz.

Tip takvog niza je:

```go
[3][2]int
```

Niz se sastoji od 3 vrednosti. Te tri vrednosti su tipa niza. Ovo su nizovi [2]int u nizu.

Da biste pristupili određenom elementu u takvom nizu, možete koristiti sledeću sintaksu:

```go
value := a[2][1]
```

a[2] preuzeće vrednost na indeksu 2. Vrednost na indeksu 2 je niz. Da preuzmemo element na indeksu 1 u ovom nizu [1].

### Primer upotrebe: lista gostiju hotela

U našem hotelu, osoblje mora imati mogućnost da dobije spisak gostiju. Naš hotel se sastoji od 10 soba, a svaka soba može imati najviše dva gosta. Da bismo predstavili ovaj spisak gostiju, možemo koristiti dvodimenzionalni niz.

Indeks prvog niza je broj sobe. Postoji deset soba, tako da je dužina ovog niza 10. Unutar svakog elementa stavljamo drugi niz sastavljen od dva niza (string). Evo koda koji će generisati ovu listu[2]string:

```go
// array/two-dimensional/guestList/guestList.go
package guestList

import (
    "github.com/Pallinder/go-randomdata"
)

func generate() [10][2]string {
    guestList := [10][2]string{}
    for index, _ := range guestList {
        guestList[index] = roomGuests(index)
    }
    return guestList
}

func roomGuests(roomId int) [2]string {
    guests := [2]string{}
    guests[0] = randomdata.SillyName()
    guests[1] = randomdata.SillyName()
    return guests
}
```

U funkciji generisaćemo inicijalizovati naš dvodimenzionalni niz. Zatim ćemo ga iterirati sa for-range petljom. Za svaki element, stavljamo element tipa (Niz dužine 2 sastavljen od dva stringa).  

Ovaj niz sadrži dva imena gostiju. "randomdata.SillyName()" koristi se za generisanje zabavnih imena. U sistemu iz stvarnog života, preuzećemo ta imena u bazu podataka.

Ako ispišemo ovaj niz, dobijamo sledeći rezultat:

[
[Cockatoorogue Liftersolstice]  
[Footclever Chargernickel]  
[Chestsatin Scarivy]  
[Jesterbush Cloaksteel]  
[Robincanyon Boltchip]  
[Bonefrill Shiftshy]  
[Skinnerflannel Looncalico]  
[Loonnova Spikemesquite]  
[Hideforest Yakstitch]  
[Cloakcoconut Minnowsnapdragon]
]

Da bismo pristupili imenu drugog gosta koji spava u sobi 7, možemo koristiti sintaksu:

```go
guestList[7][1].
```

## Višedimenzionalni nizovi

U programskom jeziku Go je moguće generisati nizove koji imaju više od dve dimenzije:

```go
// array/two-dimensional/main.go
package main

import "fmt"

func main() {
    threeD := [2][2][2]string{}
    threeD[0][0][0] = "element 0,0,0"
    threeD[0][1][0] = "element 0,1,0"
    threeD[0][0][1] = "element 0,1,1"
    fmt.Println(threeD)
}
```

Niz "threeD" tri dimenzije.

### Oprez

Nizove koji imaju više od dve dimenzije je teško vizualizovati. Ako želite da uvedete takav tip niza u svoj kod, možda bi trebalo da razmotrite drugo rešenje. Vaš kod će biti teži za čitanje i teži za razumevanje i pregled.

## Razmatranje efikasnosti

- Pristup podacima na nizu pomoću njegovog indeksa je veoma brza operacija.

- Pronalaženje elementa unutar niza može biti sporo jer morate da iterirate kroz njega. Ako nemate
  pojma o redosledu elemenata sačuvanih u vašem nizu, verovatno ćete morati da iterirate kroz ceo niz. U ovom slučaju, možda možete koristiti mapu (pogledajte posebno poglavlje).

## Ograničenja

Nizovi su efikasni, ali njihovo glavno ograničenje je fiksna veličina.

U stvarnom svetu, podaci koje želite da sačuvate retko imaju fiksnu veličinu tokom kompajliranja. Stoga su kreatori Go-a stvorili pojam isečaka. Videćemo kako da ih manipulišemo u posebnom poglavlju.

## Testirajte sebe

### Pitanja i odgovori

1. Niz je kolekcija elemenata različitih tipova. Tačno ili netačno?
   - Netačno
   - Elementi niza treba da budu istog tipa.

2. Niz se prosleđuje kao parametar funkciji; može li funkcija da izmeni niz?
   - Ne
   - Međutim, ako funkciji prosledite pokazivač na niz, ona ga može izmeniti.

3. Kolika je dužina niza?
   - Maksimalan broj elemenata koji se mogu sačuvati.

4. Kakva je veza između dužine i kapaciteta (za nizove)?
   - Za nizove `len = cap`.

### Ključno

- Niz je kolekcija elemenata istog tipa
- Veličina niza se naziva dužina.
- Dužina niza je poznata u vreme kompajliranja.
- Drugim rečima, jednom kreiran niz ne može da raste
- Da biste iterativno prelazili kroz niz, možete koristiti petlju for (sa ili
  bez klauzule range)
- Kada se niz prosledi funkciji, on se kopira, funkcija ga ne može menjati
- Kada funkciji prosledite pokazivač na niz, omogućavate funkciji da menja niz.
- Ne postoji ugrađena funkcija za pronalaženje elementa u nizu.

[19 Jedinični testovi][19] | [00 Sadržaj][00] | [21 Isečci][21]

[19]: 19_Jedinični_testovi.md
[00]: 00_Sadržaj.md
[21]: 21_Isečci.md
