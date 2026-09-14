
# 21 Isečci

[20 Nizovi][20] | [00 Sadržaj][00] | [22 Mape][22]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je isečak?
- Kako napraviti isečak.
- Kako iterirati preko isečka
- Kako kreirati višedimenzionalne isečke.
- Kako postaviti element na određeni indeks u segmentu.
- Kako se koriste dodavanje i kopiranje.

**Obrađeni tehnički koncepti!**

- Isečak
- Dužina
- Kapacitet
- Niz
- Kopija
- Pokazivač
- Varijadička funkcija

## Definicija

Isečak je rastuća kolekcija elemenata istog tipa. Rastući je zato što ne fiksirate veličinu isečka tokom kompajliranja; možete dodavati elemente tokom izvršavanja. Kada dodajete elemente isečku tokom izvršavanja programa, kažemo da će se isečak povećavati.

Tip isečka je označen sa `[]T`, gde je `T` naziv tipa elemenata unutar isečka.

Na primer: `[]int` je isečak celih brojeva.

Imajte na umu da ne navodimo veličinu isečka kao što to radimo za nizove. Unutar uglastih zagrada ne navodimo nikakvu dužinu.

Nulta vrednost isečka je `nil`.

## Kreiranje novog isečka

```go
// slices/creation/main.go
package main

func main() {
    s := make([]int, 3)
    s[0] = 12
    s[2] = 3
}
```

Ovde definišemo promenljivu `s` tipa isečka celih brojeva. Zatim možemo popuniti isečak. Definišemo elemente na indeksima 0 i 2.

```go
s2 = []int{10,12}
```

Definišemo s2 koji je takođe isečak celih brojeva i postavljamo njegova prva dva elementa.

### Isecanje niza, pokazivač na niz ili isečak

Isečak je deo nečega. Na primer, isečak sira nije ceo sir već samo njegov deo. U Go-u je isto, možete iseći:

- niz
- pokazivač na niz
- isečak

Rezultat ove operacije (zvane isecanje) je isečak. Da biste isekli element "e", možete koristiti sledeću sintaksu:

```go
s := e[low:high]
```

"low" i "high" će vam omogućiti da izaberete opseg elemenata u e.

**Primer 1**:

```go
// slices/slicing-array/main.go
package main

import "fmt"

func main() {
    customers := [4]string{"John Doe", "Helmuth Verein", "Dany Beril", "Oliver Lump"}
    // slice the array
    customersSlice := customers[0:1]
    fmt.Println(customersSlice)
}
```

Ovaj program će ispisati:

```sh
[John Doe]
```

Imamo niz stringova pod nazivom "customers". Ovaj isečak sadrži 4 elementa. Kreiramo isečak iz ovog niza, koji sadrži element niza od indeksa 0 do indeksa 1. Drugim rečima, uzimamo prvi element "customers[0]".

**Primer 2**:

```go
customersSlice2 := customers[2:4]
fmt.Println(customersSlice2)
```

Ovaj program će ispisati:

```sh
[Dany Beril Oliver Lump]
```

Uzimamo elemente od indeksa 2 do indeksa 4, [2, 3].

**Zapamtite**:

Kada pišete:

s := e[low:high]

dobijate od indeksa "low" do "high-1" elemente iz e.

Drugi način da se to zapamti je da uzimamo elemente od "low" do "high", isključujući.

**Da li se sečenje kopira podatke?**

Ne! Evo jednog primera:

```go
// slices/slicing-copy/main.go
package main

import "fmt"

func main() {
    customers := [4]string{"John Doe", "Helmuth Verein", "Dany Beril", "Oliver Lump"}
    customersSlice := customers[0:1]
    fmt.Println(customersSlice)
    // modify original array
    customers[0] = "John Doe Modified"
    fmt.Println("After modification of original array")
    fmt.Println(customersSlice)
}
```

Izlaz ovog programa:

```sh
[John Doe]
After modification of the original array
[John Doe Modified]
```

Kada kreiramo isečak, podaci se ne kopiraju, već se uzima referenca na originalne podatke.

### Isecanje stringa

Takođe možete iseći string! Rezultat ove operacije je string:

```go
hotelName := "Go Dev Hotel"
s := hotelName[0:6]
fmt.Println(s)
```

Ovaj program će ispisati: "Go Dev". Stringovi u programu Go su nepromenljivi. Jednom kreirani i sačuvani u memoriji, ne možete menjati string. Pogledajmo primer:

```go
// slices/slicing-string/main.go
package main

import "fmt"

func main() {
    hotelName := "Go Dev Hotel"
    s := hotelName[0:6]
    fmt.Println(s)
    hotelName = "Java Dev Hotel"
    fmt.Println(s)
}
```

U ovom isečku, prvo kreiramo string ( "hotelName" ) . Zatim ga isečemo. Isečak se zove s. Zatim menjamo vrednost "hotelName".

Mogli bismo očekivati da će s biti izmenjeno, ali nije. Program će ispisati:

```sh
Go Dev
Go Dev
```

### Dužina isečka

Dužina je broj elemenata koji se nalaze u isečku. Ugrađena funkcija `len` vraća dužinu isečka:

```go
vatRates := []float64{4.65, 4, 15, 20}
fmt.Printf("length of slice vatRates is %d", len(vatRates))
```

### Reprezentacija interne memorije

Interno, struktura isečka sadrži pokazivač na niz.

- Kada kreiramo isecanje, Go će kreirati niz. Nikada mu nećete imati pristup.
  Ovaj niz je interni.
- Kada isecamo niz, Go će uzeti pokazivač na taj postojeći niz.

Isečak je spoj tri elementa:

- Pokazivač na osnovni niz. Pokazivač pokazuje na osnovni niz gde počinje
  isečak. Pokazuje na prvi
  element isečka.
- Dužina isečka ( uint )
- Kapacitet isečka ( uint )

Kada kreiramo isečak u našem programu, biće kreiran niz. Ovaj niz će sadržati elemente isečka. Interno, Go će sačuvati pokazivač na prvi element ovog niza.

Isečak se takođe može kreirati isecanjem postojećeg niza. U ovom slučaju, kapacitet isečka nije jednak njegovoj dužini.

### Kapacitet

Kapacitet je neoznačeni ceo broj. On predstavlja "broj elemenata za koje je dodeljen prostor u osnovnom nizu". Go specifikacija: <https://golang.org/ref/spec#Length_and_capacity.> Da biste preuzeli kapacitet isečka, možete koristiti ugrađenu funkciju `cap`.

Uzmimo primer da bismo razumeli koncept kapaciteta:

```go
names := [4]string{"John", "Bob", "Claire", "Nik"}

mySlice := names[1:3]

fmt.Println("length:", len(mySlice))
fmt.Println("capacity:", cap(mySlice))
```

Ovaj program će ispisati:

```sh
length: 2
capacity: 3
```

Napravili smo imenovani isečak "mySlice" isecanjem "names" niza. Uzimamo sve između elementa na indeksu jedan i elementa na indeksu 2 (3-1). "mySlice" ima dužinu 2 (unutra se nalaze dva elementa). Ali kapacitet je jednak 3. U osnovnom nizu isečka je dodeljen prostor za tri elementa.

### Odnos dužine i kapaciteta

Kapacitet je uvek veći ili jednak dužini isečka. Nema smisla imati kapacitet koji je manji od dužine. Zamislite isečak dužine 3 i kapaciteta 2. Možemo da sačuvamo samo dva elementa u osnovnom nizu od tri elementa?!?

### Isečci u parametrima funkcije

Funkcija koja uzima isečak kao parametar može promeniti osnovni niz. Zašto? Zato što je isečak interno pokazivač na osnovni niz. Evo primera:

```go
// slices/as-fct-parameters/main.go
package main

import "fmt"

func main() {
    s := []int{1, 2, 3}
    multiply(s, 2)
    fmt.Println(s) //[2 4 6]
}

func multiply(slice []int, factor int) {
    for i := 0; i < len(slice); i++ {
        slice[i] = slice[i] * factor
    }
}
```

Ovde definišemo isečak "s" (sastavljen od celih brojeva). Kreiramo funkciju "multiply" koja uzima isečak celih brojeva i factor (int) kao parametar. Cilj ove funkcije je da pomnoži svaki element isečka sa factor. Kada se "s" kreira, Go će interno kreirati niz koji će sadržati vrednosti 1, 2 i 3. Takođe će kreirati pokazivač na taj niz. Kada prosledimo "s" funkciji "multiply", ona će izmeniti osnovni niz. Isečci su po prirodi pokazivači.

**Uobičajena greška**!

- Kada funkciji prosledite isečak, funkcija može da ga izmeni promenom vrednosti
  osnovnog niza.
- Kada dodate elemente u isečak unutar funkcije i vaš segment dostigne svoj
  maksimalni kapacitet,
  okruženje za izvršavanje dodeljuje novi osnovni niz.
- To ponašanje je normalno, međutim, na kraju izvršavanja funkcije, elementi
  dodati u isečak neće biti prisutni kao što biste očekivali.

**Primer**:

Uzmimo primer da ovo ilustrujemo:

```go
// slices/common-error-function/main.go
package main

import "fmt"

func main() {
    languages := []string{"Java", "PHP", "C"}
    fmt.Println("Capacity :", cap(languages))
    // Capacity : 3

    // call function
    addGo(languages)
    
    fmt.Println("Capacity :", cap(languages))
    // Capacity : 3
    fmt.Println(languages)
    // [Java PHP C]
    // what ! , where is Go ?????
}

func addGo(languages []string) {
    languages = append(languages, "Go")
    fmt.Println("in function, capacity", cap(languages))
}
```

**Objašnjenja**:

- Kreiramo funkciju "addGo" koja će dodati element isečku stringova.
- U funkciji main, inicijalizujemo isečak stringova pod nazivom "languages".
- Inicijalizujemo isečak sa tri elementa. NJen kapacitet i dužina su jednaki 3.
- Zatim pozivamo funkciju "addGo". Nakon poziva, ispisujemo isečak i nijedan
  element nije dodat.

Šta se dešava u funkciji?

- Kada se funkcija pozove, izvršno okruženje pravi kopiju isečka.
- Novi osnovni niz je dodeljen unutar funkcije jer je kapacitet prekoračen.
- Referenca na osnovni niz koji isečak sadrži interno je promenjena.
- Ali, promenjeno je samo na kopiji isečka.
- Kada se funkcija vrati, kopirani isečak se uništava. Isečak "languages" i
  dalje referencira stari
  osnovni niz.

**Kako to izbeći**?

- "addGo" treba da prihvati pokazivač na isečak stringova kao ulazni parametar.
- U tom slučaju, isečak se ne kopira.
- Referenca na osnovni niz se ažurira u originalnom isečku.

```go
// slices/common-error-function-fix/main.go
package main

import "fmt"

func main() {
    languages := []string{"Java", "PHP", "C"}
    fmt.Println("Capacity :", cap(languages))
    // Capacity : 3

    // call function
    addGoFixed(&languages)

    fmt.Println("Capacity :", cap(languages))
    // Capacity : 6
    fmt.Println(languages)
    // [Java PHP C Go]
}

func addGoFixed(languages *[]string) {
    *languages = append(*languages, "Go")
}
```

- `*` znači "prati adresu", ovo je operator dereferenciranja (ako to nije jasno,
  pogledajte poglavlje posvećeno pokazivačima)

## Ugrađene funkcije za rad sa isečcima

### Ugrađena funkcija `make`

Sa ugrađenom komandom `make`, možete kreirati isečak sa određenom dužinom i kapacitetom. `make` će vratiti novi isečak. Morate navesti tip elemenata koje želite da stavite u isečak, kao i njegovu dužinu i kapacitet. Imajte na umu da je kapacitet opcionalan. Ako nije naveden, kapacitet će biti jednak dužini.

```go
test := make([]int,2,10)
```

Isečak test će imati dužinu 2 i kapacitet 10. Pazite, kapacitet ne sme biti manji od dužine; u suprotnom se neće kompajlirati!.

### Ugrađena funkcija `copy`

Kopiranje je praktična funkcija koja vam omogućava da kopirate sve elemente isečka (zvane izvor) u drugi isečak (zvani odredište). Funkcija kopiranja uzima dva isečka kao parametre. Ugrađena funkcija `copy` će kopirati sve elemente drugog isečka u prvi. Prvi isečak nazivamo odredištem, a drugi izvorom.

Funkcija vraća ceo broj koji predstavlja broj uspešno kopiranih elemenata.

```go
func copy(dst, src []Type) int
```

Imajte na umu da će broj kopiranih elemenata biti minimalan između dužine izvora i dužine odredišta.

Na primer, isečak ima četiri elementa, a izvorni isečak ima 2. Tada će broj kopiranih elemenata biti 2. Ako odredište ima dužinu 1, onda će broj kopiranih elemenata biti samo jedan. Ugrađena funkcija copy ne povećava dužinu odredišnog isečka. Ako želite da imate isti isečak, morate da obezbedite odredišni isečak iste dužine kao i izvorni isečak.

Evo primera korišćenja ugrađene funkcije `copy`:

```go
// slices/copy-builtin/main.go
package main

import "fmt"

func main() {
    // destination > source
    a := []int{10, 20, 30, 40}
    b := []int{1, 1, 1, 1, 1}
    copy(b, a)
    fmt.Printf("a: %v - b : %v\n", a, b)
    // Output : a: [10 20 30 40] - b : [10 20 30 40 1]

    // source > destination
    a = []int{10, 20, 30, 40}
    b = []int{1, 1}
    copy(b, a)
    fmt.Printf("a: %v - b : %v\n", a, b)
    // Output : a: [10 20 30 40] - b : [10 20]

    // source = destination
    a = []int{10, 20, 30, 40}
    b = make([]int, 4)
    copy(b, a)
    fmt.Printf("a: %v - b : %v\n", a, b)
    // Output : a: [10 20 30 40] - b : [10 20 30 40]
}
```

Imao sam poteškoća da zapamtim da je prvi argument kopije odredište, a drugi izvor.

Evo jednog mnemotehničkog saveta koji treba zapamtiti: nazivi parametara su po abecednom redu.  
D pa S!

### Ugrađena funkcija `append`

Funkcija `append` će dodati element(e) na kraj isečka.

Funkcija `append` prihvata kao prvi parametar isečak, a zatim jedan ili više elemenata koje će mu se dodati. Kažemo da je `append` varijadička funkcija.

Funkcija se naziva varijadičkom kada ima promenljiv broj parametara. Definicija funkcije ne fiksira broj parametara. U signaturi funkcije to možete otkriti tako što ćete potražiti tri tačke "... ".

```go
func append(slice []Type, elems ...Type) []Type
```

Ovde funkcija uzima isečak kao parametar, ali i elemente tipa Type, gde tri tačke znači da možemo dati nekoliko vrednosti za drugi parametar "elems". Funkcija `append` vraća izmenjeni isečak. Uzmimo primer upotrebe:

```go
// slices/append/main.go
package main

import "fmt"

func main() {
    a := []int{10, 20, 30, 40}
    a = append(a, 50)
    fmt.Println(a)
    // [10 20 30 40 50]
}
```

Ovde prvo kreiramo isečak "a", a zatim mu dodajemo ceo broj 50. Imajte na umu da će funkcija `append` vratiti modifikovanu verziju. Stoga moramo ponovo dodeliti modifikovanu verziju promenljivoj.

Na kraju skripte, ispisujemo "a" i dobijamo isečak "a" sa novim elementima unutra. Funkcija `append` je automatski promenila veličinu našeg originalnog isečka. Ako kapacitet osnovnog niza nije dovoljan, funkcija append će dodeliti novi.

### Kako rastu isečci

Sa ugrađenom funkcijom `append` možemo dodati više podataka na kraj isečka. Interno, kada dodajemo vrednost isečku, postavljamo vrednost u osnovni niz. Osnovni niz ima određenu veličinu. Ako nema preostalog prostora u nizu, Go će kreirati novi niz sa dovoljno prostora. Go će zatim kopirati sve elemente iz prethodnog niza u novi niz. Uzmimo primer:

```go
s := []uint{10, 20, 30, 40}
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 4 - Capacity : 4

s = append(s, 50)
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 5 - Capacity : 8

s = append(s, 60)
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 6 - Capacity : 8

s = append(s, 70)
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 7 - Capacity : 8

s = append(s, 80)
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 8 - Capacity : 8

s = append(s, 90)
fmt.Printf("Length : %d - Capacity : %d\n", len(s), cap(s)) // Length : 9 - Capacity : 16
```

U ovom programu, kreiramo isečak s sa četiri početna elementa, a zatim mu dodajemo elemente. Nakon svakog koraka beležimo dužinu isečka i njegov kapacitet.

- U početku imamo četiri elementa i osnovni niz od 4 elementa.

- Kada dodamo jedan element na isečak, kapacitet se povećava na 8.
  - To znači da je kreiran novi osnovni niz od 8 elemenata.
  - Elementi prisutni u starom nizu se svi kopiraju u novi niz.
- Kada dodamo 90 na isečak length == capacity. Osnovni niz je pun.
  - Kreira se novi niz elemenata kapaciteta 16.
  - Elementi prisutni u starom nizu se svi kopiraju u novi.

### Uticaj na performanse

Kada se osnovni niz popuni, kreira se novi niz i vaši podaci se kopiraju u novi niz. Ova operacija može uticati na performanse vašeg programa. Uzmimo primer:

```go
func grow1() {
    s := []uint{10, 20, 30, 40}
    s = append(s, 50)
    s = append(s, 60)
    s = append(s, 70)
    s = append(s, 80)
    s = append(s, 90)
}

func grow2() {
    s := make([]uint, 9)
    s = append(s, 10, 20, 30, 40)
    s = append(s, 50)
    s = append(s, 60)
    s = append(s, 70)
    s = append(s, 80)
    s = append(s, 90)
}
```

Funkcija "grow2" je relativno brža od "grow1". U "grow2" kreiramo isečak koristeći ugrađenu funkciju `make`. Eksplicitno kažemo da želimo dužinu od 9 (i samim tim kapacitet od 9). Izvršno okruženje će kreirati osnovni niz jednom. U "grow1" Go će kreirati dva osnovna niza (i kopirati sve elemente iz jednog niza u drugi). Evo benčmarka koji sam pokrenuo na svom računaru Macbook Pro - 2,2 GHz Quad-Core Intel Core i7 - 16 GB 1600 MHz DDR3.

- grow1: prosek: 94,8 nanosekundi po operaciji
- grow2: prosek: 58,7 nanosekundi po operaciji

## Indeks elemenata isečka

Elementi u isečku su indeksirani. Indeksiranje počinje od 0. U svetu računarstva, brojanje počinje od 0. To je često izvor grešaka za početnike. Molimo vas da to imate na umu.

### Pristup elementu po njegovom indeksu

Dakle, ako želite da pristupite elementu na indeksu 3 iz isečka a, možete koristiti sledeći kod:

```go
elementAtIndex3 := a[3]
```

Morate biti sigurni da element na indeksu postoji. U suprotnom, možete se suočiti sa panikom tokom izvršavanja. Na primer, sledeći program će se kompajlirati (kompajler neće proveriti postojanje indeksa). Evo primera:

```go
// slices/access-element-index/main.go
package main

import "fmt"

func main() {
    a := []int{10, 20, 30, 40}
    fmt.Println(a[8])
}
```

Ovaj program će se kompajlirati. Želimo da pristupimo elementu na indeksu 8. Ali taj element ne postoji. Zbog čega program paniči:

```sh
$ go run main.go
panic: runtime error: index out of range
```

Ako pokušate da pristupite elementu niza van indeksa, program se neće kompajlirati. Nizovi imaju dužinu poznatu u vreme kompajliranja; Isečci imaju dužinu koja može da raste tokom izvršavanja programa.

## Iteriranje elemenata isečka

Da biste iterirali preko elemenata isečka, možete koristiti petlju for:

```go
names := []string{"John", "Bob", "Claire", "Nik"}
for i, name := range names {
    fmt.Println("Element at index", i, "=", name)
}
```

Ovaj program će ispisati:

```sh
Element at index 0 = John
Element at index 1 = Bob
Element at index 2 = Claire
Element at index 3 = Nik
```

U svakoj iteraciji:

- Vrednost "i" jednaka vrednosti iteracije indeksa.
- Vrednost "name" je jednaka vrednosti elementa isečka na indeksu i.

### Pažnja, opasnost - za petlje koje modifikuju isečak

Zamislite da treba da izmenite svaki element isečka. Petlja `for` sa izrazom `range` može da obavi posao. Ali postoji početnička greška koju treba izbegavati.

Uzmimo primer:

```go
// Warning : there is a trap!
names := []string{"John", "Bob", "Claire", "Nik"}
for _, name := range names {
    name = strings.ToUpper(name)
}
fmt.Println(names)
```

Ovaj program će ispisati:

```sh
[John Bob Claire Nik]
```

Originalni isečak nije izmenjen! Izraz `range` će kopirati vrednost svakog elementa u lokalnu promenljivu name. `name = strings.ToUpper(name)` izmeniće samo lokalnu promenljivu. Originalna imena isečaka ostaju netaknuta!

```go
for i := range names {
    names[i] = strings.ToUpper(names[i])
}
fmt.Println(names)
```

Program će ispisati:

```sh
[JOHN BOB CLAIRE NIK]
```

## Spajanje dva isečka

Sledeći isečak koda će se spojiti "b" i "c":

```go
b := append(b, c...)
```

Imajte na umu da "b" i "c" moraju biti istog tipa.

```go
test := append([]int{10,20}, []int{30,40,50}...)
```

Promenljiva test će biti jednaka sledećem isečku:

```sh
[10,20,30,40,50]
```

## Pronalaženje elementa u isečku

Ne postoji ugrađena funkcija za to.

Morate iterirati preko svakog elementa isečka i proveriti da li je element jednak onom koji tražite.

```go
names := []string{"John", "Bob", "Claire", "Nik"}
for i, name := range names {
    if name == "Claire" {
        fmt.Println("Claire found at index", i)
    }
}
// Output : Claire found at index 2
```

## Uklanjanje elementa na indeksu

Algoritam sa: <https://github.com/golang/go/wiki/SliceTricks>

Recimo da imamo isečak dužine 10:

[1,2,3,4,5,6,7,8,9,10]

Želimo da eliminišemo element na indeksu 8 (što je 9). Tehnika je da napravimo dva isečka. Prvi isečak će sadržati elemente od indeksa 0 do indeksa 7. Drugi isečak će sadržati samo jedan element: onaj na indeksu 9.

Evo koda za ovo pomoću Go-a:

```go
a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
a = append(a[:8], a[9:]...)
```

- Prvo, kreiramo isečak koji se zove a.
- Zatim koristite ugrađenu funkciju `append` da bismo spojili dva isečka:
- a[:8]: što je "a" svedeno na elemente sa indeksima od 0 do 7. (imajte na umu da je indeks posle ":" uvek isključen).
- a[9:]: koji sadrži sve elemente od "a" indeksa devet, uključujući.
- Zatim spajamo dva isečka da bismo dobili naš novi isečak koji ne sadrži element na indeksu 8!

Evo generalizacije algoritma. Ako imate isećak s i želite da obrišete element na indeksu i, onda možete koristiti sledeće:

```go
// Za i < len(s):
a = append(a[:i], a[i+1:]...)
```

## Stavite element na indeks isečka

Algoritam sa: <https://github.com/golang/go/wiki/SliceTricks>

Imamo isečak, žeelimo da mu dodamo element, ali na određenom indeksu. Ideja je da koristimo ugrađenu funkciju za kopiranje:

```go
// objective put "C" at index 2
// index:            0    1    2    3    4
letters := []string{"A", "B", "D", "E", "F"}

// 1) add an element to the end of the slice
letters = append(letters, "")

// 2) copy letters[i:] to letters[i+1:]
copy(letters[3:], letters[2:])

// 3) set "C" at index 2
letters[2] = "C"
```

### Generalizacija

Kada je "s" je isečak celih brojeva:

```go
s = append(s, 0)
copy(s[i+1:], s[i:])
s[i] = x
```

## Uklonite sve elemente isečka

### a[:0]

Možete ukloniti sve elemente datog isečka korišćenjem operatora isecanja. Caka je u tome da isečete naš isečak. Uzećemo 0 elemenata iz njega:

```go
a := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
a = a[:0]
fmt.Println(a)
// []

fmt.Println(len(a))
// 0

fmt.Println(cap(a))
// 10
```

Operatorom isecanja [:0] smo uklonili sve elemente isečka.

Stoga je dužina sada jednaka 0. Ali kapacitet (što je veličina osnovnog niza) je i dalje 10.

To znači da osnovni niz i dalje postoji i ima kapacitet od 10.

### Postavite isečak na nulu

Možete postaviti vrednost vašeg isečka na nil. Time uklanjamo podatke iz memorije.

b := []int{1, 2, 3, 4, 5, 6, 7, 8, 9, 10}
b = nil
fmt.Println(b)
// []

fmt.Println(len(b))
// 0

fmt.Println(cap(b))
// 0

Imajte na umu da je kapacitet sada jednak nuli, što znači da je memorija koja sadrži osnovni niz oslobođena i da će je interna go skripta sakupljati kao smeće.

## Dodavanje elementa na početak isečka

Možemo koristiti ugrađenu funkciju `append` da dodamo unos u isečak.

Ali za dodavanje na početak, izgleda da nemamo ugrađenu funkciju na raspolaganju. Ali možete koristiti ugrađenu funkciju dodavanja na početak:

```go
b := []int{2, 3, 4}
b = append([]int{1}, b...)
```

Pravimo novi isečak sa elementom koji želimo da dodamo na početak, a zatim dodajemo sve elemente originalnog isečka.

## Višedimenzionalni isečci

Višedimenzionalni isečci vam omogućavaju da kreirate složenu strukturu podataka kojom je lako manipulisati.

### Kreiranje dvodimenzionalnih isečaka

Ponekad se vaši podaci ne uklapaju u potpunosti u jednu dimenziju. Zamislite da radite u supermarketu. Želite da proučite korpe klijenata. Možda ćete želeti da analizirate raspodelu cena unutar korpi. Imate detaljan cenovnik za svaku korpu. Na primer:

Primer cenovnika

| korpa | cene |
| :---: | ---- |
| 1 | 1.23​; 1; 12.40; 3; 20 |
| 2 | 40; 2. 45​; 23 |
| ... | ... |
| 1000 | 5 |

Možemo koristiti dvodimenzionalni isečak da ga sačuvamo. Tip će biti. To je isečak koji će sadržati isečke vrednosti tipa float: `[][]float64`

```go
// slices/multidimensional/main.go
package main

import "fmt"

func main() {
    innerSlice1 := []float64{2.30, 4.01, 6.99, 8}
    innerSlice2 := []float64{1, 10.20, 30, 1.34, 23}
    my2DSlice := [][]float64{innerSlice1, innerSlice2}
    fmt.Println(my2DSlice)
}
```

Prvo definišemo dva isečka `float64` ( "innerSlice1" i "innerSlice2" ). A zatim kreiramo naš dvodimenzionalni isečak ( nazvan "my2DSlice" ) koji će sadržati "innerSlice1" i "innerSlice2".

Sledeći isečak je lakša verzija prethodnog skripta:

```go
// slices/2d-dimension/main.go
package main

import "fmt"

func main() {
    my2DSlice := [][]float64{[]float64{2.30, 4.01, 6.99, 8}, []float64{1, 10.20, 30, 1.34, 23}}
    fmt.Println(my2DSlice)
}
```

Direktno smo stavili dva unutrašnja isečka u glavni (bez korišćenja dve dodatne promenljive).

### Dinamičko kreiranje dvodimenzionalnih preseka

Možete koristiti ugrađenu funkciju appendza dinamičko konstruisanje vašeg 2D preseka:

```go
myOther2DSlice := [][]int{}
for i := 0; i < 10; i++ {
    myOther2DSlice = append(myOther2DSlice, []int{i})
}
fmt.Println(myOther2DSlice)
//[[0] [1] [2] [3] [4] [5] [6] [7] [8] [9]]
```

Prvo definišemo naš 2D isečak myOther2DSlice. Unutrašnji isečci su isečci celih brojeva.

Zatim kreiramo for-petlju. Ponavljaćemo od 0 do 9. Svaki put ćemo dodavati novi isečak na "myOther2DSlice". Ovaj isečak će sadržati samo jedan broj: trenutnu vrednost "i".

Na kraju izvršavanja, dobićemo isečak sastavljen od 10 unutrašnjih isečaka. Svaki od tih unutrašnjih isečaka sadrži jedan ceo broj.

### Više od dve dimenzije?

Rukovanje dimenzijama većim od dva nije komplikovano.

3D isečak će biti definisan korišćenjem "[][][]T" gde je "T" tip elemenata unutar isečaka.

Evo jednog primera:

```go
store12 := [][]float64{[]float64{2.30, 4.01, 6.99, 8}, []float64{1, 10.20, 30, 1.34, 23}}
store34 := [][]float64{[]float64{1, 1.25}, []float64{2.45}, []float64{1.99, 2.45}}
my3DSlice := [][][]float64{store12,store34}
fmt.Println(my3DSlice)
//[[[2.3 4.01 6.99 8] [1 10.2 30 1.34 23]] [[1 1.25] [2.45] [1.99 2.45]]]
```

Da biste kreirali isečke veće dimenzije, samo dodajte dodatni dodatak "[]" definiciji tipa:

```go
// 4-D
fourDimensions := [][][][]int

// 5-D
fiveDimensions := [][][][][]int
```

**Dimenzije se povećavaju, dok se čitljivost smanjuje**:

Imajte na umu da kada povećate dimenziju vaših isečaka, takođe povećavate vreme potrebno ostalima da razumeju vaš algoritam.

LJudski mozak nije naviknut na prostore sa više od tri dimenzije.

Po mom mišljenju, trebalo bi da izbegavamo 3+ dimenzije.

**Koristiti drugu strukturu podataka**?

U prethodnom primeru, možda možemo koristiti drugu strukturu podataka... Pogledajte poglavlje o mapi, a zatim se vratite na ovaj primer da biste videli kako možete poboljšati čitljivost koda.

## Testirajte sebe

### Pitanja i odgovori

1. Popunite praznine: "Isečak je ______ na osnovni _____".
   - Isečak je pokazivač na osnovni niz.
2. Kako pronaći indeks elementa u segmentu?
   - Možete koristiti petlju for sa izrazom opsega:for index, value := range s
3. Šta se nalazi iza isečka, interno?
   - Struktura sastavljena od tri polja:
     - Pokazivač na osnovni niz
     - Dužina (jedinica)
     - Kapacitet (jedinica)
4. Koliki je kapacitet namesu sledećem primeru:. Objasnite zašto.names := []
   string{"John", "Bob",
   "Claire", "Nik"}
   - Kapacitet je jednak 4. Kada se kreiraju imena isečaka, Go će dodeliti osnovni niz dužine i
     kapaciteta jednakog 4.
5. Koliki je kapacitet s2u sledećem primeru:. Objasnite zašto.s2 := names[0:2]
   - s2se kreira sečenjem postojećeg dela ( names).
   - Uzimamo isečak od indeksa 0 do indeksa 1 (2-1).
     - Dužina ovog isečka će biti 2 (u isečku postoje dva elementa).
     - Kapacitet je jednak "broju elemenata za koje je dodeljen prostor u osnovnom nizu".
     - U osnovnom nizu su dodeljena još dva elementa. Kapacitet je jednak 4.
6. Šta se dešava interno kada dodate element na isečak čiji je kapacitet jednak
   njegovoj dužini?
   - Kada je kapacitet segmenta jednak njegovoj dužini, to znači da nema preostalog prostora na
     osnovnom nizu.
     - Kada dodate element u isečak, izvršno okruženje će kreirati novi niz (njegova dužina će biti
       dvostruko veća od prethodne).
     - Svi podaci iz prvog niza biće kopirani u drugi niz.

### Ključno

- Kriška je rastuća kolekcija elemenata istog tipa
- Isečak je pokazivač na osnovni niz
- Dužina : broj elemenata isečka
- Kapacitet : broj elemenata za koje je dodeljen prostor u osnovnom nizu
- Da biste kreirali prazan isečak sa datom dužinom i kapacitetom, možete
  koristiti ugrađenu komandu
  `make`:

  ```go
  s := make([]int,0,10)
  ```

  kreirate isečak celih brojeva, dužine 0 i kapaciteta 10.

. Da biste dodali elemente u isečak, koristite ugrađenu funkciju append
  
  ```go
  s = append(s,2020,2021)
  ```

- Da biste pronašli element u isečku, potrebno je da iterirate kroz taj isečak
  Ako je potrebno, možda možete umesto toga koristiti mapu.

[20 Nizovi][20] | [00 Sadržaj][00] | [22 Mape][22]

[20]: 20_Nizovi.md
[00]: 00_Sadržaj.md
[22]: 22_Mape.md
