
# 16 Interfejsi

[15 Pokazivači][15] | [00 Sadržaj][00] | [17 Go moduli][17]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je tipski interfejs?
- Kako definisati interfejs.
- Šta znači "implementirati interfejs".
- Prednosti interfejsa

**Obrađeni tehnički koncepti!**

- Interfejs
- Konkretna implementacija
- Da biste implementirali interfejs
- Skup metoda interfejsa.

## Uvod

Interfejsi deluju teško za razumevanje kada tek počnete da programirate. Često novi programeri ne razumeju u potpunosti potencijal interfejsa. Ovaj odeljak ima za cilj da objasni šta je interfejs, zašto je zanimljiv i kako se kreiraju interfejsi.

## Osnovna definicija interfejsa

- Interfejs je ugovor koji definiše skup ponašanja.
- Interfejsi su čisti objekat dizajna, oni samo definišu skup ponašanja (tj. metode) bez davanja
  bilo kakve implementacije tih ponašanja.
- Interfejs je vrsta tipa koja definiše skup metoda bez njihove implementacije.

"Implementirati" = "Napisati kod metode".

Evo primera tipa interfejsa (iz standardnog paketa `io`):

```go
type Reader interface {
    Read(p []byte) (n int, err error)
}
```

Ovde imamo tip interfejsa pod nazivom "Reader". On određuje jednu metodu pod nazivom "Read". Metoda nema telo, nema implementaciju. Jedino što je navedeno je ime metode i njen potpis (parametri i rezultati).

### Nulta vrednost tipa interfejsa

Nulta vrednost tipa interfejsa je `nil`. Primer:

```go
var r io.Reader
log.Println(r)
<nil>
```

**Osnovni primer**:

```go
type Human struct {
    Firstname string
    Lastname string
    Age int
    Country string
}

type DomesticAnimal interface {
    ReceiveAffection(from Human)
    GiveAffection(to Human)
}
```

- Prvo, deklarišemo tip pod nazivom "Human"
- Zatim deklarišemo novi tip interfejsa pod nazivom "DomesticAnimal".
- Ovaj tip interfejsa ima skup metoda sastavljen od dve metode: "ReceiveAffection"i "GiveAffection".

"DomesticAnimal" je ugovor.

To signalizira programeru da, da bismo bili "DomesticAnimal" moramo imati najmanje dva ponašanja:

- "ReceiveAffection" i
- "GiveAffection"

Hajde da napravimo dva tipa:

```go
type Cat struct {
    Name string
}

type Dog struct {
    Name string
}
```

Imamo dva nova tipa. Da bi poštovali ugovor našeg interfejsa DomesticAnimal, moramo za svaki tip definisati metode koje diktira interfejs.

Počnimo sa tipom "Cat" :

```go
func (c Cat) ReceiveAffection(from Human) {
    fmt.Printf("The cat named %s has received affection from Human named %s\n", c.Name, from.Firstname)
}

func (c Cat) GiveAffection(to Human) {
    fmt.Printf("The cat named %s has given affection to Human named %s\n", c.Name, to.Firstname)
}
```

Sada tip "Cat" implementira interfejs "DomesticAnimal". Isto radimo sada za tip: "Dog":

```go
func (d Dog) ReceiveAffection(from Human) {
    fmt.Printf("The dog named %s has received affection from Human named %s\n", d.Name, from.Firstname)
}

func (d Dog) GiveAffection(to Human) {
    fmt.Printf("The dog named %s has given affection to Human named %s\n", d.Name, to.Firstname)
}
```

Naš "Dog" tip sada ispravno implementira interfejs DomesticAnimal.

Sada možemo da kreiramo funkciju koja prihvata interfejs kao parametar:

```go
func Pet(animal DomesticAnimal, human Human) {
    animal.GiveAffection(human)
    animal.ReceiveAffection(human)
}
```

Ova funkcija može da prihvati bilo koji tip koji implementira interfejs tipa DomesticAnimal kao parametar. Stoga ne moramo da kreiramo posebne funkcije za mačke, pse, zmije, pacove… Ova funkcija je generička i može se izvršiti za različite tipove:

Funkcija "Pet" uzima "DomesticAnimal" tip interfejsa kao prvi parametar i "Human" kao drugi parametar.

Unutar funkcije pozivamo dve funkcije interfejsa.

Hajde da koristimo ovu funkciju:

```go
// interfaces/first-example/main.go 
//...

func main() {

    // Create the Human
    var john Human
    john.Firstname = "John"


    // Create a Cat
    var c Cat
    c.Name = "Maru"

    // then a dog
    var d Dog
    d.Name = "Medor"

    Pet(c, john)
    Pet(d,john)
}
```

- Tipovi "Dog" i "Cat" implementiraju metode DomesticAnimal interfejsa.
  => bilo koja promenljiva tipa Dog i Cat može se posmatrati kaot DomesticAnimal tip.

### Kompilator vas posmatra

Poštovanje ugovora interfejsa za tip T znači implementaciju svih metoda interfejsa. Hajde da pokušamo da prevarimo kompajler da vidimo šta će se desiti:

```go
//...
// let's create a concrete type Snake
type Snake struct {
    Name string
}
// we do not implement the methods ReceiveAffection and GiveAffection intentionally
//...

func main(){

    var snake Snake
    snake.Name = "Joe"

    Pet(snake, john)
}
```

- Stvaramo novi tip "Snake"
- Ovaj tip ne implementira metode "DomesticAnimal" interfejsa
- U funkciji "main" kreiramo novu promenljivu tipa "Snake"
- Zatim pozivamo "Pet" funkciju sa ovom promenljivom kao prvim parametrom.

Ne kompajlira se:

```sh
./main.go:70:5: cannot use snake (type Snake) as type DomesticAnimal in argument to Pet:
    Snake does not implement DomesticAnimal (missing GiveAffection method)
```

Kompilator se zaustavlja na prvoj metodi po abecednom redu koja nije implementirana.

**Primer:database/sql/driver.Driver**

Hajde da pogledamo interfejs "Driver" (iz paketa "database/sql/driver")

```go
type Driver interface {
    Open(name string) (Conn, error)
}
```

- Postoje različite vrste SQL baza podataka, tako da postoji nekoliko implementacija metode "Open".
- Zašto? Zato što nećete koristiti isti kod za pokretanje veze sa "MySQL" bazom podataka i "Oracle" bazom podataka.
- Izgradnjom interfejsa definišete ugovor koji može da koristi nekoliko implementacija.

### Ugrađivanje interfejsa

Možete ugraditi interfejse u druge interfejse.

Uzmimo primer:

```go
// the Stringer type interface from the standard library
type Stringer interface {
    String() string
}
// A homemade interface
type DomesticAnimal interface {
    ReceiveAffection(from Human)
    GiveAffection(to Human)

    // embed the interface Stringer into the DomesticAnimal interface
    Stringer
}
```

U prethodnom navedenom primeru, ugrađujemo interfejs "Stringer" u ​​interfejs "DomesticAnimal".

Kao posledica toga, ostali tipovi koji već implementiraju "DomesticAnimal" moraju da implementiraju metode interfejsa "Stringer".

- Ugrađivanjem interfejsa možete dodati funkcionalnost svojim interfejsima bez dupliranja.
- Takođe dolazi sa svojom cenom; ako ugradite interfejs iz drugog modula, vaš kod će biti povezan sa njim
  - Promena u drugom interfejsu modula će vas primorati da prepišete svoj kod.
  - Imajte na umu da je ova opasnost ublažena ako zavisni modul prati šemu semantičkog verzionisanja - Možete koristiti interfejse iz standardne biblioteke bez straha

**Neki korisni (i poznati) interfejsi iz standardne biblioteke**:

- Interfejs **error**

  ```go
  type error interface {
      Error() string
  }
  ```

  Ovaj tip interfejsa se intenzivno koristi. Funkcije ili metode koje mogu da ne uspeju vraćaju interfejs "error" tipa:
  
  ```go
  func (c *Communicator) SendEmailAsynchronously(email *Email) error {
      //...
  }
  ```

  Da bismo kreirali grešku, obično pozivamo funkciju : `fmt.Errorf()` koja vraća rezultat tipa `error` ili `errors.New()`.
  
  Naravno, moguće je kreirati sopstveni tip koji implementira interfejs za greške.

- Interfejs **Stringer**

  ```go
  type Stringer interface {
      String() string
  }
  ```
  
  Pomoću `Stringer` interfejsa možete definisati kako će se tip štampati kao string, kada se pozove metod štampanja ( `fmt.Errorf()`, `fmt.Println`, `fmt.Printf`, `fmt.Sprintf`,... )
  
  Evo primera implementacije
  
  ```go
  type Human struct {
      Firstname string
      Lastname string
      Age int
      Country string
  }
  
  func (h Human) String() string {
      return fmt.Sprintf("human named %s %s of age %d living in %s",
        h.Firstname, h.Lastname, h.Age, h.Country)
  }
  ```
  
  Potom koristimo "Stringer" kroz poziv "Println":
  
  ```go
  // interfaces/stringer/main.go 
  package main 
  
  func main() {
      var john Human
      john.Firstname = "John"
      john.Lastname = "Doe"
      john.Country = "USA"
      john.Age = 45
  
      fmt.Println(john)
  }
  ```

  Izlazi:
  
  ```sh
  human named John Doe of age 45 living in the US
  ```

- Interfejs **sort**

Implementacijom ovog interfejsa na tip, možete sortirati elemente tipa (obično je osnovni tip isečak)

```go
type Interface interface {
    Len() int
    Less(i, j int) bool
    Swap(i, j int)
}
```

Evo primera upotrebe (izvor: sort/example_interface_test.go):

```go
type Person struct {
    Age int
}

// ByAge implements sort.Interface for []Person based on the Age field.
type ByAge []Person

func (a ByAge) Len() int           { return len(a) }
func (a ByAge) Swap(i, j int)      { a[i], a[j] = a[j], a[i] }
func (a ByAge) Less(i, j int) bool { return a[i].Age < a[j].Age }
```

- Tip "ByAge" implementira "Interface" interfejs.
  - Osnovni tip je "isečak Person".

- Interfejs se sastoji od tri metode:
  - **Len() int**: treba da vrati broj elemenata unutar kolekcije
  - **Less(i, j int) bool**: vraća true ako element na indeks "i" treba da bude sortiran pre elementa
    na indeksu "j".
  - **Swap(i, j int)**: trebalo bi da zamenimo elemente na indeksu "i" i "j"; drugim rečima, trebalo
    bi da stavimo element na indeks "j" na indeks "i", a element na indeksu "i" treba da bude stavljen na indeks "j".

Zatim možemo sortirati promenljivu tipa ByAge pomoću funkcije "sort".

```go
// interfaces/sort/main.go

func main() {
    people := []Person{
        {"Bob", 31},
        {"John", 42},
        {"Michael", 17},
        {"Jenny", 26},
    }

    Sort(ByAge(people))
}
```

## Implicitna implementacija interfejsa

Interfejsi su implicitno implementirani. Kada deklarišete tip, ne morate da navodite koje interfejse implementira.

**Slučajevi PHP i JAVA**:

U drugim jezicima, morate navesti implementacije interfejsa.

Evo jednog primera na Javi:

```go
// JAVA
public class Cat implements DomesticAnimal{
    public void receiveAffection(){
        //...
    }
    public void giveAffection(){
        //..
    }
}
```

Evo još jednog primera u PHP-u

```go
//PHP
<?php

class Cat implements DomesticAnimal {
    public function receiveAffection():void {
        //...
    }
    public function giveAffection():void {
        //...
    }
}
?>
```

U ta dva jezika imamo klase. Možete videti da morate dodati termin "implements" kada deklarišete klasu koja implementira interfejs.

Možda se pitate kako Go runtime rukuje tim implicitnim implementacijama interfejsa. Pokušaćemo da objasnimo mehanizam vrednosti interfejsa.

### Prazan interfejs

Prazan interfejs u ​​programu Go je najjednostavniji i najmanji interfejs koji možete napisati. NJegov skup metoda se sastoji od tačno 0 metoda.

```go
interface {}
```

Uz to rečeno, svaki tip implementira prazan interfejs. Možda se pitate zašto je tako dosadan interfejs zanimljiv. Po definiciji, prazna vrednost interfejsa može da sadrži vrednosti bilo kog tipa. Korisno je ako želite da napravite metodu koja prihvata bilo koji tip.

Uzmimo nekoliko primera iz standardne biblioteke.

- U "log" paketu imate "Fatal" metodu koja kao ulazne promenljive uzima bilo kog tipa:

  ```go
  func (l *Logger) Fatal(v...interface{}) { }
  ```

  - U fmt paketu takođe imamo mnogo metoda koje kao ulaz uzimaju prazan interfejs. Na primer, "Printf" funkcija:

  ```go
  func Printf(format string, a...interface{}) (n int, err error) { }
  ```go

### Switch tipa

Funkcija koja prihvata prazan interfejs kao parametar generalno mora da zna efektivni tip svog ulaznog parametra.

Da bi to uradila, funkcija može da koristi "switch tipa", to je slučaj prekidača koji će uporediti tip umesto vrednosti.

Evo primera preuzetog iz standardne biblioteke (datoteka , paket "runtime/error.goruntime"):

```go
// printany prints an argument passed to panic.
// If panic is called with a value that has a String or Error method,
// it has already been converted into a string by preprintpanics.
func printany(i interface{}) {
    switch v := i.(type) { // switch type
    case nil:
        print("nil")
    case bool:
        print(v)
    case int:
        print(v)
    case int8:
        print(v)
    case int16:
        print(v)
    case int32:
        print(v)
    case int64:
        print(v)
    case uint:
        print(v)
    case uint8:
        print(v)
    case uint16:
        print(v)
    case uint32:
        print(v)
    case uint64:
        print(v)
    case uintptr:
        print(v)
    case float32:
        print(v)
    case float64:
        print(v)
    case complex64:
        print(v)
    case complex128:
        print(v)
    case string:
        print(v)
    default:
        printanycustomtype(i)
    }
}
```

**O korišćenju praznog interfejsa**:

- Trebalo bi da koristite prazan interfejs veoma pažljivo.
- Koristite prazan interfejs kada nemate drugih izbora.
- Prazan interfejs ne daje nikakve informacije osobi koja će koristiti vaše funkcije ili metode, pa
  će morati da se pozove na dokumentaciju, što može biti frustrirajuće.
- Ako prihvatite prazan interfejs, vaša funkcija/metoda će morati da proveri tip ulaza, što će kod
  učiniti složenijim.

Koji metod preferirate?

```go
func (c Cart) ApplyCoupon(coupon Coupon) error  {
    //...
}

func (c Cart) ApplyCoupon2(coupon interface{}) (interface{},interface{}) {
    //...
}
```

Metoda "ApplyCoupon" strogo određuje tip koji će prihvatiti i vratiti. Dok "ApplyCoupon2" ne određuje svoje tipove na ulazu i izlazu. Kao pozivalac, teža je za korišćenje "ApplyCoupon2" u  poređenju sa "ApplyCoupon".

**Primena - Skladištenje korpi kupaca**:

Pravite veb lokaciju za e-trgovinu; morate da čuvate i preuzimate korpe kupaca. Moraju se razviti sledeća ponašanja:

- Preuzimate korpu po njenom ID-u
- Dodate korpu u bazu podataka

Predložite interfejs za ta dva ponašanja.

Takođe kreirajte tip koji implementira ta dva interfejsa. (Ne implementirajte logiku u metodi.)

**Rešenje**:

Evo predloženog interfejsa:

```go
// interfaces/application/main.go

type CartStore interface {
    GetById(ID string) (*cart.Cart, error)
    Put(cart *cart.Cart) (*cart.Cart, error)
}
```

Tip koji implementira interfejs:

```go
type CartStoreMySQL struct{}

func (c *CartStoreMySQL) GetById(ID string) (*cart.Cart, error) {
    // implement me
}

func (c *CartStoreMySQL) Put(cart *cart.Cart) (*cart.Cart, error) {
    // implement me
}
```

I još jedan tip koji implementira interfejs:

```go
type CartStorePostgres struct{}

func (c *CartStorePostgres) GetById(ID string) (*cart.Cart, error) {
    // implement me
}

func (c *CartStorePostgres) Put(cart *cart.Cart) (*cart.Cart, error) {
    // implement me
}
```

- Možete kreirati specifičnu implementaciju za svaki model baze podataka koji koristite
- Dodavanje podrške za novi sistem baze podataka je jednostavno! Samo treba da kreirate novi tip
  koji implementira interfejs.

## Zašto bi trebalo da koristite interfejse

### Evolutivnost

Kada koristite interfejse kao ulaz u svoje metode ili funkcije, dizajnirate svoj program da bude evolutivan. Budući programeri (ili budući vi) mogu kreirati nove implementacije bez menjanja velikih delova koda.

Zamislite da ste napravili aplikaciju koja vrši čitanje, unos i ažuriranje baze podataka. Možete koristiti dva pristupa dizajniranju:

- Napravite tip i metode koji su usko povezani sa sistemom baze podataka koji trenutno koristite.
- Napravite interfejs koji navodi sve operacije i konkretnu implementaciju za vaš sistem baze podataka.

U prvom pristupu, kreirate metode koje će kao parametar uzimati određenu implementaciju.
Na taj način zaključavate program na jednu implementaciju

U drugom pristupu, kreirate metode koje prihvataju interfejs.
Promena implementacije je jednostavna kao kreiranje novog tipa koji implementira interfejs

### Poboljšanje timskog rada

Timovi takođe mogu imati koristi od interfejsa.

Prilikom izgradnje funkcionalnosti, često je potrebno više od jednog programera da obave posao. Ako posao zahteva kod koji su napisala dva tima za interakciju, oni se mogu dogovoriti o jednom ili više interfejsa.

Dva tima programera zatim mogu da rade na svom kodu i koriste dogovoreni interfejs. Čak mogu i da ismevaju rad drugog tima. Na taj način, tim nije blokiran od strane drugog tima.

### Iskoristite prednosti niza rutina

Kada implementirate interfejse na svom prilagođenom tipu, možete koristiti dodatne funkcionalnosti koje nije potrebno razvijati. Uzmimo primer iz standardne biblioteke: "sort" paket. Ovo nije iznenađenje; ovaj paket se koristi za sortiranje stvari. Evo izvoda izvornog koda za go:

```go
// go v.1.10.1
package sort
//..

type Interface interface {
    // Len is the number of elements in the collection.
    Len() int
    // Less reports whether the element with
    // index i should sort before the element with index j.
    Less(i, j int) bool
    // Swap swaps the elements with indexes i and j.
    Swap(i, j int)
}

// Sort sorts data.
// It makes one call to data.Len to determine n, and O(n*log(n)) calls to
// data.Less and data.Swap. The sort is not guaranteed to be stable.
func Sort(data Interface) {
    n := data.Len()
    quickSort(data, 0, n, maxDepth(n))
}
```

U prvom redu deklarišemo trenutni paket: sort. U sledećim redovima, programeri su deklarisali interfejs koji se jednostavno zove "Interface". Ovaj interfejs specificira tri metode: "Len", "Less", "Swap".

U sledećim redovima, funkcija "Sort" je deklarisana. Ona uzima kao parametar data tipa "Interface". Ovo je veoma korisna funkcija koja će sortirati date podatke.

Kako možemo da koristimo ovu funkciju na jednom od naših tipova?

### Implementacija interfejs

Zamislite da imate tip "User":

```go
type User struct {
    firstname string
    lastname string
    totalTurnover float64
}
```

I tip "Users" koji je isečak User instanci:

```go
type Users []User
```

Hajde da kreiramo instancu Users i popunimo je sa tri promenljive tipa User.

```go
user0 := User{firstname:"John", lastname:"Doe", totalTurnover:1000}
user1 := User{firstname:"Dany", lastname:"Boyu", totalTurnover:20000}
user2 := User{firstname:"Elisa", lastname:"Smith Brown", totalTurnover:70}

users := make([]Users, 3)
users[0] = user0
users[1] = user1
users[2] = user2
```

Šta ako želimo da sortiramo po "totalTurnover"? Možemo razviti od nule algoritam za sortiranje koji odgovara našim specifikacijama. Ili možemo jednostavno implementirati interfejs potreban za korišćenje ugrađene funkcije "Sort" iz "sort" paketa. Hajde da to uradimo:

```go
// Compute the length of the array. Easy...
func (users Users) Len() int {
  return len(users)
}

// decide which instance is bigger than the other one
func (users Users) Less(i, j int) bool {
  return users[i].totalTurnover < users[j].totalTurnover
}

// swap two elements of the array
func (users Users) Swap(i, j int) {
    users[i], users[j] = users[j], users[i]
}
```

Deklarisanjem tih funkcija, možemo jednostavno koristiti funkciju "Sort":

```go
sort.Sort(users)
fmt.Println(users)
```

```sh
// will output :
[{Elisa Smith Brown 70} {John Doe 1000} {Dany Boyu 20000}]
```

## Saveti

- Koristite interfejse koje pruža standardna biblioteka
- Interfejs sa previše metoda je teško implementirati (jer zahteva pisanje mnogo metoda).

## Testirajte sebe

### Pitanja i odgovori

1. Navedite primer interfejsa koji ugrađuje drugi interfejs.  
   To je ReadWriter interfejs.

   ```go
   type ReadWriter interface {
       Reader
       Writer
   }
   ```

2. Tačno ili netačno? Metode navedene u ugrađenim interfejsima nisu deo skupa metoda interfejsa.  
   Netačno.
   Skup metoda interfejsa sastoji se od:
    - Metode direktno navedene u interfejsu
    - Metode iz ugrađenog interfejsa.

3. Navedite dve prednosti korišćenja interfejsa.  
   Lako podelite posao između programera:
   - Definišite tip interfejsa
   - Jedna osoba razvija implementaciju interfejsa
   - Druga osoba može da koristi tip interfejsa u njegovoj funkciji
   - Dve osobe mogu da rade bez međusobnog ometanja.
   - Evolutivnost
     - Kada kreirate interfejs, kreirate ugovor.
     - Različite implementacije mogu ispuniti ovaj ugovor.
     - Na početku projekta obično postoji jedna implementacija
     - Ali vremenom, možda će biti potrebna još jedna implementacija.

4. Koja je nulta vrednost tipa interfejsa?  
   nil

### Ključno

- Interfejs je ugovor
- On specificira metode (ponašanja) bez njihove implementacije.
  
  ```go
  type Cart interface {
      GetById(ID string) (*cart.Cart, error)
      Put(cart *cart.Cart) (*cart.Cart, error)
  }
  ```

- Interfejs je tip (kao strukture, nizovi, mape,...)
- Metode navedene u interfejsu nazivamo skupom metoda interfejsa.
- Tip može da implementira više interfejsa.
- Nema potrebe eksplicitno navoditi da tip implementira interfejs, za razliku od drugih jezika koji
  zahtevaju deklarisanje (PHP, Java, ...)
- Interfejs može biti ugrađen u drugi interfejs; u tom slučaju, metode ugrađenog interfejsa se
  dodaju interfejsu.
- Tipovi interfejsa mogu se koristiti kao i bilo koji drugi tip.
- Nulta vrednost tipa interfejsa je nil.
- Bilo koji tip implementira prazan interfejs: `interface{}`.
- Prazan interfejs određuje 0 metoda.
- Da biste dobili konkretan tip praznog interfejsa, možete koristiti `switch tipa`:

  ```go
  switch v := i.(type) {
      case nil:
          print("nil")
      case bool:
          print(v)
      case int:
          print(v)
  }
  ```

- Kada možemo da implementiramo ponašanje na različite načine, verovatno možemo da kreiramo
  interfejs.  
  Npr: Skladištenje (možemo koristiti MySQL, Postgres, DynamoDB, Redis bazu podataka za skladištenje istih podataka).

[15 Pokazivači][15] | [00 Sadržaj][00] | [17 Go moduli][17]

[15]: 15_Pokazivači.md
[00]: 00_Sadržaj.md
[17]: 17_Go_moduli.md
