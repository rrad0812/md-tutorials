
# 24 Anonimne funkcije i zatvaranja

[23 Greške][23] | [00 Sadržaj][00] | [25 JSON i XML][25]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je anonimna funkcija?
- Kako kreirati anonimnu funkciju.
- Kako pozvati anonimnu funkciju.
- Šta je zatvaranje?
- Šta je tip funkcije?

**Obrađeni tehnički koncepti!**

- Anonimna funkcija
- Funkcijski literal
- Leksički kontekst
- Obim

## Anonimne funkcije

Anonimna funkcija je slična tradicionalnoj funkciji, osim što programer toj funkciji ne daje ime.
Da bismo kreirali anonimnu funkciju, koristimo literal funkcije:

```go
// anonymous/anon/main.go
package main

import "fmt"

func main() {
    // an anonymous function
    func() {
        // the instructions of the anonymous function
        fmt.Println("I am anonymous function")
    }()
}
```

Ovde definišemo funkciju bez imena koristeći sintaksu `func(){..}` i izvršavamo je dodavanjem zagrada `()`.

Dakle, da biste definisali anonimnu funkciju i izvršili je odmah nakon toga, možete koristiti sledeću sintaksu:

```go
func(){...}()
```

- Da biste definisali funkciju, koristite sintaksu: `func(){..}`
- Da biste je definisali i pokrenuli: `func(){..}()`

Uzmimo još jedan primer:

```go
// anonymous/example-2/main.go
package main

import (
    "fmt"
    "reflect"
)

func main() {

    /**
     * func literal not executed
     */
    myFunc := func() int {
        fmt.Println("I am a func litteral")
        return 42
    }
    // Output : nothing
    // the function is not executed

    fmt.Println(reflect.TypeOf(myFunc))
    // Output : func() int

    /**
     * func literal invoked
     */
    funcValue := func() int {
        fmt.Println("I am a func literal invoked")
        return 42

    }()
    // Output: I am a func literal invoked
    // the func is executed!
    fmt.Println(reflect.TypeOf(funcValue))
    // Output : int

}
```

Šta je ovde najvažnije?

- Samo zapamtite da funkcija ne može imati ime;
- Možete sačuvati funkciju (ne njen rezultat) već samu funkciju u promenljivoj.

Ovo poslednje stvara mnogo problema mnogim programerima. Zbunjeni smo jer smo navikli da čuvamo vrednosti, a ne funkcije unutar promenljivih!

### Tip funkcija

Možete definisati tip na osnovu potpisa funkcije. Evo primera preuzetog iz modula gin (<https://github.com/gin-gonic/gin>)

```go
package gin

//...

type HandlerFunc func(*Context)

//...
```

A evo primera preuzetog iz standardnog http paketa:

```go
package http
//...
type HandlerFunc func(ResponseWriter, *Request)

//...
```

Tip funkcije označava sve funkcije sa istim parametrima i rezultatima.  Kada definišete svoj tip, možete kreirati promenljivu koja ima tip funkcije. Uzmimo primer:

```go
type Funky func(string)

var f Funky
f = func(s string){
    // my function defined
    log.Printf("Funky %s",s)
}
f("Groovy")
```

- Definišemo novi izvezeni tip Funky. Funky označava sve funkcije sa jednim parametrom tipa string i bez rezultata (func(string)).
- Kreiramo promenljivu f tipa Funky.
- Zatim promenljivoj dodeljujemo f anonimnu funkciju.
- A onda, izvršavamo f sa f("Groovy").

### Funkcije su "građani prvog reda"

U nekim programskim jezicima  funkcije su "građani prvog reda"  što im omogućava da budu:

- Korišćeni kao parametri
- Korišćene kao rezultati
- Dodeljene promenljivoj

Go funkcije se smatraju objektima prve klase. To znači da:

- Funkcije se mogu prosleđivati kao argumenti drugim funkcijama
- Možete vratiti funkcije iz drugih funkcija
- Možete povezati funkcije sa promenljivim

## Zatvaranja

Anonimna funkcija definisana u drugoj funkciji F može koristiti elemente koji su definisani u opsegu funkcije F.

Anonimne funkcije mogu zadržati reference na svoje okolno stanje. Zatvaranje se formira kada funkcija referencira promenljive koje nisu definisane u njenom sopstvenom opsegu važenja.

### Primer zatvaranja

```go
// anonymous/first-closure/main.go
package main

import "fmt"

func printer() func() {
    k := 1
    return func() {
        fmt.Printf("Print n. %d\n", k)
        k++
    }
}

func main() {

    p := printer()
    p() // Print n. 1
    p() // Print n. 2
    p() // Print n. 3
}
```

Imamo funkciju koja se zove printer. Signatura ove funkcije je `printer() func()`. To znači da će ova funkcija vratiti `func()` kao rezultat, što znači da vraćamo funkciju koja ništa ne vraća. Mogli bismo imati sledeću sintaksu kao tip povratka: `func() int`, što bi značilo da će naša vraćena funkcija vratiti ceo broj. Hajde da pređemo na deklaraciju funkcije printer. Prvo, definišemo promenljivu `k` koja je inicijalizovana vrednošću 1. A zatim vraćamo anonimnu funkciju. U ovoj funkciji koristimo `fmt.Printf("Print n. %d\n", k)`, da prikažemo na standardnom izlazu gde je `%d` zamenjeno vrednošću `k`.

Ovde mi:

- definišemo promenljivu `k` u spoljašnjoj funkciji
- vraćamo novu anonimnu funkciju koja koristi `k`.

Stoga smo napravili zatvaranje.

- Zatim, u funkciji `main`, dodelimo povratnu vrednost funkcije `printer` (što je funkcija) promenljivoj `p`.
- Nakon toga ćemo koristiti `p`.
- Zapamtite da je `p` funkcija, pa da bismo je izvršili, samo treba da dodamo zagrade na kraj:
  
  ```go
  `p()` // we execute p, the closure
  ```

- Tip `p` je `func()`.

### Opsezi

- Kažemo da anonimne funkcija imaju svoj leksički kontekst.
- Funkcija čuva memoriju promenljivih koje se koriste u njenom okruženju.
- Opseg funkcije (ili leksičko okruženje) je region gde možete koristiti sve promenljive (tipove, konstante) koje ste u njoj deklarisali.
- Funkcija koja obuhvata definiše spoljašnji opseg važenja. U našem slučaju, definišemo promenljivu `k` u spoljašnjem opsegu funkcije koja obuhvata. Zatim se k koristi u vraćenom opsegu funkcije (unutrašnji opseg važenja).
- Koristimo promenljivu u unutrašnjem opsegu važenja koju smo deklarisali u spoljašnjem opsegu važenja.
- Zatvaranja čuvaju referencu na promenljivu koju smo definisali u spoljašnjem opsegu važenja.
- Zato se vrednost `k`  povećava kada nekoliko puta izvršimo našu closure. Closure sadrži referencu i svaki put kada ga izvršimo, menjaće osnovnu vrednost.

### Neki uobičajeni slučajevi upotrebe zatvaranja

#### Parametri funkcije

Možete proslediti funkciju kao parametar druge funkcije. U paketu `sort` (deo standardne biblioteke), nekoliko funkcija prihvata funkcije kao parametre:

```go
// sort package

// Search for an index of a specific value using binary search
func Search(n int, f func(int) bool) int

// sort a slice by using the provided function (less)
func Slice(slice interface{}, less func(i, j int) bool)
```

U prethodna dva potpisa, vidimo da funkcije `sort.Search` i `sort.Slice` zahtevaju po jednu funkciju da bi radile. Funkcija se koristi interno za obavljanje posla sortiranja.

Uzmimo primer:

```go
// anonymous/input-parameters/main.go
package main

import (
    "fmt"
    "sort"
)

func main() {
    scores := []int{10, 89, 76, 3, 20, 12}
    // ascending
    sort.Slice(scores, func(i, j int) bool { return scores[i] < scores[j] })
    fmt.Println(scores)
    // Output : [3 10 12 20 76 89]

    // descending
    sort.Slice(scores, func(i, j int) bool { return scores[i] > scores[j] })
    fmt.Println(scores)
    // Output : [89 76 20 12 10 3]
}
```

- Definišemo promenljivu "scores", koja je isečak celih brojeva. Oni predstavljaju pobednički rezultat za 6 igara "gofing" (novi sport koji praktikuju samo goferi).
- Scores nisu sortirani. Cilj je sortirati ih po određenom redosledu.
- Koristićemo funkciju `sort.Slice`, iz paketa `sort`.
- Njen potpis zahteva isečak i funkciju. Funkcija će definisati kako će izvršno okruženje izvršiti sortiranje. Funkcije uzimaju dva parametra (cele brojeve); oni predstavljaju dva indeksa. Vraća logičku vrednost.
- Ovde imamo zatvaranje jer funkcija koristi promenljivu "scores", čak i ako ona nije definisana u svom unutrašnjem opsegu važenja.

U našem prvom primeru, funkcija less je:

```go
func(i, j int) bool { return scores[i] < scores[j] }
```

Element na indeksu "i" će biti postavljen ispred elementa na indeksu "j" ako je element na indeksu "i" manji od elementa na indeksu "j".

Kao rezultat toga, naš isečak će biti sortiran u rastućem redosledu:

```sh
[3 10 12 20 76 89]
```

U drugom pozivu funkcije `sort.Slice` dajemo kao ulaz sledeću funkciju:

```go
func(i, j int) bool { return scores[i] > scores[j] }
```

Element na indeksu "i" će biti postavljen ispred elementa na indeksu "j" ako je element na indeksu "i" veći od elementa na indeksu "j". Rezultat je opadajući redosled:

```sh
[89 76 20 12 10 3]
```

#### Omotači - http.HandleFunc

Možemo koristiti zatvarače da omotavanje druge funkcije.
Pre nego što objasnimo tehniku, napravićemo jednostavan veb server (možete preskočiti pasus ako već znate kako se to radi)

**Osnovni veb server**:

U sledećem bloku koda, navodimo da će se funkcija `homepageHandler` izvršiti kada se zahtev pošalje krajnjoj tački "/homepage".

```go
// anonymous/wrappers/main.go 

func homepageHandler(writer http.ResponseWriter, request *http.Request) {
  fmt.Fprintln(writer, "Welcome to my homepage")
  fmt.Fprintln(writer, "I am max")
}
```

Funkcija `homepageHandler` uzima parametar za pisanje odgovora ( `http.ResponseWriter` ) (za pripremu odgovora) i zahtev ( `*http.Request` ).
Evo primera minimalnog veb servera:

```go
// anonymous/wrappers/main.go

package main

import (
    "fmt"
    "net/http"
)

func main() {
    http.HandleFunc("/homepage", homepageHandler)
    err := http.ListenAndServe(":3000", nil)
    if err != nil {
        log.Fatal(err)
    }
}

func homepageHandler(writer http.ResponseWriter, request *http.Request) {
    fmt.Fprintln(writer, "Welcome to my homepage")
    fmt.Fprintln(writer, "I am max")
}
```

**Tehnika omotača**:

Zamislite da vaš veb server ima nekoliko krajnjih tačaka i da želite da pratite posete na svakoj od njih. Brutalni način da se ovo uradi bio bi da se doda brojač u svaki od vaših obrađivača. Ali postoji pametan način: možete kreirati generički omotač koji svi vaši obrađivači mogu da koriste. Vaš omotač treba da prihvati kao ulaz obrađivač i da vrati obrađivač. Na taj način možete imati sledeću sintaksu:

```go
http.HandleFunc("/homepage", trackVisits(homepageHandler))
```

Ovo je elegantno! Razumemo da ovde pratimo posete, a zatim izvršavamo obrađivač. Hajde da implementiramo funkciju `trackVisits`:

```go
func trackVisits(handler func(http.ResponseWriter, *http.Request)) 
    func(http.ResponseWriter, *http.Request) { // return anonim. func from trackVisits
        return func(writer http.ResponseWriter, request *http.Request) {
            // track the visit
            fmt.Println("one visit !")

            // call the original handler
            handler(writer, request)
    }
}
```

- Funkcija `trackVisits` uzima handler kao parametar i vraća handler.
- Tip povratka `trackVisits` mora imati isti potpis kao onaj koji očekuje drugi ulazni parametar `http.HandleFunc`.
- U telu funkcije vraćamo anonimnu funkciju. Ova anonimna funkcija će prvo ispisati da imamo jednu posetu, a zatim će pokrenuti izvršenje originalnog obrađivača.

Ova tehnika je veoma moćna. Daje nam moć da kreiramo generičke omotače koje možemo koristiti na nekoliko krajnjih tačaka: izbegava dupliranje koda.

```go
http.HandleFunc("/homepage", trackVisits(homepageHandler))
http.HandleFunc("/cv", trackVisits(cvHandler))
http.HandleFunc("/projects", trackVisits(projectsHandler))
```

Da li je moguće i lančano povezati omotače:

```go
http.HandleFunc("/projects", authenticate(trackVisits(projectsHandler)))
```

definisanjem drugih omotača:

```go
func authenticate(handler func(http.ResponseWriter, *http.Request)) 
    func(http.ResponseWriter, *http.Request) {
    return func(writer http.ResponseWriter, request *http.Request) {
        // check that request is authenticated
        fmt.Println("authenticated !")
        // call the original handler
        handler(writer, request)
    }
}
```

#### Obrazac funkcionalnih opcija

Dejv Čejni je opisao ovaj obrazac.  To je obrazac koji vam omogućava da poboljšate API vaših biblioteka.

Obično biblioteka nudi nekoliko "opcija". Kod ovog obrasca, opcije su obezbeđene sa funkcijama.

**Primer funkcionalnih opcija**:

Uzmimo primer biblioteke za lažnu autentifikaciju:

```go
auth := authenticator.New("test", authenticator.UnverifiedEmailAllowed(),
    authenticator.WithCustomJwtDuration(time.Second))

if auth.IsValidJWT("invalid") {
    log.Println("mmmm... maybe we should use another lib.")
}
```

- Pozivamo authenticator.New da inicijalizujemo novi autentifikator
- Funkciji predajemo string "test" i još dva parametra, koji su
  - `authenticator.UnverifiedEmailAllowed()`
  - `authenticator.WithCustomJwtDuration(time.Second)`
- Ta dva dodatna parametra su pozivi funkcija koji će vratiti vrednost.

Ovde vidite da su opcije podešene pomoću funkcija.

**Kako je napravljeno iznutra?**

```go
// anonymous/functionalOptions/authenticator/authenticator.go
package authenticator

import (
    "time"
)

type options struct {
    jwtDuration          time.Duration
    allowUnverifiedEmail bool
    allowUnverifiedPhone bool
}

type Option func(*options)

func WithCustomJwtDuration(duration time.Duration) Option {
    return func(options *options) {
        options.jwtDuration = duration
    }
}

func UnverifiedEmailAllowed() Option {
    return func(options *options) {
        options.allowUnverifiedEmail = true
    }
}

func UnverifiedPhoneAllowed() Option {
    return func(options *options) {
        options.allowUnverifiedEmail = true
    }
}

type Authenticator struct {
    name    string
    options options
}

func New(name string, opts...Option) *Authenticator {
    options := options{
        jwtDuration: time.Hour,
    }
    for _, opt := range opts {
        opt(&options)
    }
    return &Authenticator{name: name, options: options}
}

func (a Authenticator) IsValidJWT(jwt string) bool {
    return false
}
```

- Prvo se kreira options struktura
  - Ova struktura će pratiti podešene opcije.
  - Svaka opcija je polje (neizvezeno).
- Tip funkcije je kreirantype Option func(*options)
- Za svaku opciju kreiramo izvezenu funkciju koja će vratitiOption
  - Za opciju "dozvoli neproverenu e-poštu" imamo sledeću funkciju:

    ```go
    func UnverifiedEmailAllowed() Option {
        return func(options \*options) {
            options.allowUnverifiedEmail = true
        }
    }
    ```

`New` će uzeti kao drugi parametar `Option`

- Ova funkcija je varijabilna, jer se može pozvati sa nula ili više argumenata tipa Option.
  Unutra `New`:
  - Prvo, podešavamo podrazumevane vrednosti.
  - Zatim iteriramo preko `opts` (što je isečak `Option`)
  - U svakoj iteraciji dobijamo funkciju koja postavlja opciju
  - Jednostavno izvršavamo funkciju sa `opt(&options)`

## Testirajte sebe

1. Tačno ili netačno? Ne možete dodeliti anonimnu funkciju promenljivoj.
    1. Netačno
    2. Možete dodeliti anonimnu funkciju promenljivoj
       f35 := func()uint{ return 35}
2. Tačno ili netačno? Ne možete definisati imenovanu funkciju unutar funkcije
    1. Ovo je istina
    2. Sledeći fragmenti se neće kompajlirati:

       ```go
       func main(){
           func insideFunction(){
           }
       }

       func main(){
           f := func(s string){
               func insideFunction(){
               }
               test()
           }
       }
       ```

3. Tačno ili netačno? Ne možete definisati anonimnu funkciju unutar funkcije.
    1. Netačno
4. Popunite prazninu: "______ se može deklarisati sa tipom ____."
    1. Anonimna funkcija može biti deklarisana pomoću literala tipa.
5. Navedite jedan uobičajeni slučaj upotrebe zatvaranja.
    1. Rukovaoci za omotavanje
6. Može li anonimna funkcija koristiti promenljivu koja nije definisana u njenom
   telu ili listi parametara?
    1. Da
    2. Anonimna funkcija može da koristi elemente definisane u svojoj okružujućoj funkciji.

## Ključno

- Da biste definisali anonimnu funkciju, koristite literal funkcije
  `func(){ fmt.Println("my anonymous function")}`
- Funkcijski literal može biti dodeljen promenljivoj
  `myFunc := func(){ fmt.Println("my anonymous function")}`
- Da biste definisali i odmah pozvali anonimnu funkciju, koristite zagrade:
  `func(){ fmt.Println("my anonymous function")}()`
  - Ovde definišemo i izvršavamo ovu anonimnu funkciju
- Tip funkcije označava sve funkcije sa istim parametrima i rezultatima
  - Nulta vrednost tipa funkcije je nil
    Npr.: `type Option func(*options)`
- Funkcije su "građani prvog reda".
  - Mogu se proslediti kao parametri
  - Mogu se vratiti kao rezultat
  - Mogu se dodeliti promenljivoj
- Anonimna funkcija može da koristi elemente (promenljive, tipove, konstante) koji su definisani u njenom okružujućem opsegu važenja
  - Kažemo da anonimna funkcija obuhvata leksički kontekst okolne funkcije.

[23 Greške][23] | [00 Sadržaj][00] | [25 JSON i XML][25]

[23]: 23_Greške.md
[00]: 00_Sadržaj.md
[25]: 25_JSON_i_XML.md
