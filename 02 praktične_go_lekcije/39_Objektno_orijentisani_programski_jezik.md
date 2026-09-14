
# 39 Objektno orijentisani programski jezik

[38 Generici][38] | [00 Sadržaj][00] | [40 Nadogradnja ili vraćanje na stariju verziju][40]

**Šta ćete naučiti u ovom poglavlju?**

- Šta su objektno orijentisani jezici?
- Koje su zajedničke karakteristike objektno orijentisanih jezika?

**Obrađeni tehnički koncepti**:

- apstrakcija
- polimorfizam
- nasleđivanje
- sastav
- razred
- objekat
- struktura tipa
- ugrađivanje tipa

## Uvod

Ako ste novi u programiranju, možda niste bili izloženi "objektno orijentisanom programiranju". Ako ste iskusan programer, velika je verovatnoća da ste razvijali programe pomoću objektno orijentisanog jezika: znate šta su objekti, šta znači enkapsulacija... itd.

Svrha ovog odeljka je da pregleda sve karakteristike objektno orijentisanog jezika. Na taj način ćemo pokušati da razumemo zašto se Go razlikuje od tih tradicionalnih jezika.

Većina mojih studenata se plaši termina i pojmova povezanih sa objektno orijentisanim programiranjem. Nemojte da vas impresioniraju ti komplikovani termini.

## Šta je objektno orijentisan jezik

Koncept nije nov.

Davne 1966. godine, SIMULA jezik je bio objektno orijentisan, zatim SMALLTALK-80, prvi put objavljen 1980. godine. Ovaj poslednji jezik se mnogo koristio u školama da bi se učenici naučili osnovama dizajna objekata.

Koja je definicija ove vrste jezika? Objektno-orijentisani jezik je "programski jezik koji omogućava korisniku da izrazi program u smislu objekata i poruka između tih objekata" (IEEE definicija).

Definicija ističe da je to kategorija programskih jezika koja korisnicima nudi objekte i način za razmenu poruka između objekata.

Objektno orijentisani jezici dele neke zajedničke karakteristike:

- Klase i objekti su gradivni blokovi programa.
- Nudi se određeni nivo apstrakcije
- Obezbeđuju se mehanizmi enkapsulacije
- Polimorfizam je moguć
- Klase mogu nasleđivati jedna od druge

Hajde da ponovimo svaku od tih karakteristika kako bismo utvrdili da li se Go može smatrati objektno orijentisanim programskim jezikom.

### Klasa, objekat i instanca

Ovaj odeljak će dati neke osnovne definicije klase, objekta i instance. To su opšte definicije koje se primenjuju na većinu objektno orijentisanih programskih jezika.

### Klasa

- Ovo je korisnički definisan programski entitet
- Definiše skup atributa (svaki atribut ima tip)
- Atributi se nazivaju i svojstva, polja, članovi podataka
- Definiše metode (ponašanja i operacije) i njihovu implementaciju.
- Metode i atributi imaju specifičnu "vidljivost". Atributi i metode se mogu pozivati samo pod
  određenim uslovima.
- Često, klase imaju "konstruktor", to je metoda koja je dizajnirana da kreira objekat.
  - Konstruktori se koriste za inicijalizaciju objekta "stanje", tj. inicijalizaciju vrednosti
    atributa na određene vrednosti.

Evo primera klase u C++:

```cpp
// C++
#include <iostream>
using namespace std;

class Teacher {
  public:
    string firstname; // Attribute
    string lastname;  // Attribute
    void sayHello() {  // Method
      cout << "Hello World!";
    }
};

int main() {
  Teacher myTeacher;
  myTeacher.sayHello();
  return 0;
}
```

### Objekti

- Objekti se kreiraju tokom izvršavanja programa. Oni su "entiteti tokom izvršavanja".
- Poziv konstruktora klase kreira objekat.
- Objekat se naziva i "instanca".
- Proces kreiranja objekta naziva se "instanciranje".

### Go klase

Go ima "tipove", ali nema klase. Tip određuje: "skup vrednosti operacije i metode specifične za te vrednosti". Strukturni tipovi vam omogućavaju da definišete skup polja sa imenom i tipom. Uzmimo primer objekta nastavnika:

```go
// object-oriented/classes/main.go 

type Teacher struct {
    id        int
    firstname string
    lastname  string
}
```

Definišemo tip "Theacher" koji ima tri polja: "id" tipa `int`, "firstname" i "lastname" tipa `string`.

Takođe možemo da prikačimo metode ovom tipu:

```go
func (t *Teacher) sayHiToClass(){
    fmt.Printf("Hi class my name is %s %s", t.firstname, t.lastname)
}
```

Definisali smo strukturu tipa sa strukturom podataka i priloženim metodama. Hajde da kreiramo instancu "t" tog tipa:

```go
t := Teacher{12, "John", "Doe"}
```

Onda možemo koristiti metode definisane na objektu sa tom konkretnom instancom:

```go
t.sayHiToClass()
```

Takođe možete kreirati tip na osnovu drugog tipa:

```go
type StatusCode int
```

I ovaj tip može imati i metode:

```go
func (s *StatusCode) String() string {
    return fmt.Sprintf("Status Code %d",s)
}
```

U ovom slučaju, skup vrednosti StatusCode je ograničen na cele brojeve.

- Tip strukture u Gou su bliske klasama
- Tip Struktura imaju polja
- Možete definisati metode vezane za tipove struktura
- U poređenju sa drugim jezicima, nema "konstruktora", podrazumevano.
- Polja tipa struktura se automatski inicijalizuju Go-om (na nultu vrednost tipa polja).
- Umesto konstruktora, neke biblioteke definišu "New" funkciju:

Evo jedne "New" funkcije iz modula `github.com/gin-gonic/gin`:

```go
package gin

// New returns a new blank Engine instance without any middleware attached.
// ...
func New() *Engine {
    //...
}
```

Evo jedne `New` funkcije definisane u standardnom paketu `math/rand`:

```go
package rand

// New returns a new Rand that uses random values from src
// to generate other random values.
func New(src Source) *Rand {
    //..
}
```

I još jedne iz `doc` paketa:

```go
package doc

//...

func New(pkg *ast.Package, importPath string, mode Mode) *Package {
    //...
}
```

- Možete primetiti da, generalno, New funkcija vraća pokazivač,
- Takođe je uobičajeno imati New funkciju koja vraća pokazivač i `error`.

## Apstrakcija

Još jedan važan koncept objektno orijentisanog programiranja je apstrakcija.

Kada gradimo naš kod, pozivamo metode ili funkcije da bismo izvršili određenu operaciju:

```go
u.store()
```

Ovde govorimo našem programu da pozove metodu "store". Ne zahteva da znate kako je metoda implementirana (kako je kodirana unutra).

- Termin apstrakcija potiče od latinskog `abstractio`. Ovaj termin prenosi ideju odvajanja,
  oduzimanja nečega.
- Pozivi funkcija i metoda oduzimaju konkretnu implementaciju, složene detalje.
- Glavna prednost skrivanja detalja implementacije je to što ih možete promeniti bez uticaja na
  poziv funkcije (ako ne promenite potpis funkcije).

## Enkapsulacija

Enkapsulacija potiče od latinskog izraza "capsula" što znači "mala kutija". U francuskom jeziku, termin "kapsula" se odnosi na nešto što je zatvoreno u kutiju. U biologiji, ovaj termin se koristi za označavanje ćelijske membrane.

Kapsula je kontejner u koji možete staviti predmete. Inkapsuliranje nečega je ekvivalentno zatvaranju nečega unutar kapsule.

Ako se vratimo na računarsku nauku, enkapsulacija je tehnika dizajniranja koja se koristi za grupisanje operacija i podataka u izolovane kapsule kodova. Pristup tim operacijama i podacima je regulisan, a te "kapsule" definišu spoljne interfejse za interakciju sa njima.

Objektno orijentisan programski jezik daje programeru mogućnost enkapsuliranja objekata. Da li je to slučaj sa Go jezikom? Možemo li enkapsulirati stvari? Možemo li definisati stroge eksterne interfejse na paketima?

Odgovor je da! Koncept paketa vam omogućava da definišete strogi eksterni interfejs.

### Vidljivost

Uzmimo primer "packageA". Definišemo "packageA":

```go
// object-oriented/encapsulation/packageA/packageA.go
package packageA

import "fmt"

func DoSomething() {
    fmt.Println("do something")
}

func onePrivateFunction() {
    //...
}

func anotherPrivateFunction() {
    //...
}

func nothingToSeeHere() {
    //..
}
```

U "packageA" imamo samo jednu javnu funkciju - "DoSomething". Spoljašnji svet može da koristi ovu funkciju, ali se neće kompajlirati ako pokušate da pristupite privatnoj funkciji ( "onePrivateFunction", "anotherPrivateFunction", ...). U jeziku ne postoji ključna reč public ili private; vidljivost je definisana velikim i malim slovom prvog slova definisanih elemenata (metoda, tipova, konstanti ...).

### Prednosti enkapsulacije

- Sakriva implementaciju od korisnika paketa.
- Omogućava vam da promenite implementaciju bez brige da ćete pokvariti nešto u korisničkom kodu.
  Možete pokvariti nešto samo ako promenite jedan od izvezenih (javnih) elemenata vašeg paketa.
- Održavanje i refaktorisanje je lakše izvoditi. Interne promene retko utiču na klijente.

## Polimorfizam

Polimorfizam je sposobnost nečega da ima više oblika. Reči potiču od termina `poli` (što znači nekoliko) i `morfizam` koji dizajnira forme, oblike nečega.

Kako to primeniti na programski jezik? Da biste razumeli polimorfizam, morate se fokusirati na operacije, tj. metode.

Sistem se naziva polimorfnim ako se "ista operacija može ponašati različito u različitim klasama".

Hajde da usavršimo naš primer sa nastavnikom. Školski nastavnik nije isto što i univerzitetski nastavnik; stoga možemo definisati dve različite "klase", dva različita tipa strukture. Tip "SchoolTeacher" strukture:

```go
type SchoolTeacher struct {
    id        int
    firstname string
    lastname  string
}
```

I "UniversityTeacher" tip strukture:

```go
type UniversityTeacher struct {
    id         int
    firstname  string
    lastname   string
    department string
    specialty string
}
```

Druga vrsta nastavnika ima više polja za svoju specijalnost i odeljenje.

Zatim ćemo definisati istu operaciju za dve vrste nastavnika:

```go
func (t SchoolTeacher) SayHiToClass() {
    fmt.Printf("Hi class my name is %s %s\n", t.firstname, t.lastname)
}
```

A za drugi tip, implementacija je malo drugačija:

```go
func (t UniversityTeacher) SayHiToClass() {
    fmt.Printf("Hi dear students my name is %s %s I will teach you %s\n", t.firstname, t.lastname, t.specialty)
}
```

Univerzitetski nastavnik uvek navodi svoju oblast i odsek kada pozdravlja razred. Dok školski nastavnik to ne čini.

Ovde imamo istu operaciju definisanu na dva različita tipa; možemo definisati interfejs Teacher:

```go
type Teacher interface {
    SayHiToClass()
}
```

Automatski će tipovi "SchoolTeacher" i "UniversityTeacher" implementirati interfejs (ne morate ga navoditi u izvornom kodu). Hajde da napravimo isečak sastavljen od dva objekta:

```go
john := UniversityTeacher{id:12,firstname:"John",lastname:"Doe",department:"Science",specialty:"Biology"}
marc := SchoolTeacher{id:23,firstname:"Marc",lastname:"Chair"}
```

DŽon je univerzitetski profesor biologije; Mark obrazuje decu. NJihov posao nije isti, ali dele zajedničku operaciju: operaciju pozdravljanja razreda. Stoga implementiraju tip interfejsa `Teacher`. Možemo grupisati ta dva objekta u isečak sastavljen od objekata `Teacher`:

```go
teachers := []Teacher{john,marc}
```

Zatim možemo da iteriramo preko isečka i nateramo ih da pozdrave svoju klasu na svoj specifičan način:

```go
// object-oriented/polymorphism/main.go 
func main() {
    //...
    teachers := []Teacher{john, marc}
    for _, t := range teachers {
        t.SayHiToClass()
    }
}
```

U ovom poslednjem kodu, otkrili smo moć polimorfizma:

- Go može da pronađe koju metodu da pozove na osnovu tipa t.
- Isti tip interfejsa dizajnira različite implementacije Teacher.

## Nasleđivanje

Dete može naslediti genetska svojstva svojih predaka. Ali ne možemo svesti decu na njihove pretke. Kada odrastu, izgradiće svoja specifična svojstva.

Isti koncept nasleđivanja primenjuje se na klase u objektno orijentisanom programiranju. Kada kreirate složenu aplikaciju, često identifikovani objekti dele neka zajednička svojstva i operacije.

Na primer, svaki nastavnik ima ime, prezime, datum rođenja, adresu itd. Školski nastavnici i univerzitetski nastavnici imaju oba ta svojstva. Zašto ne kreirati super objekat koji će sadržati ta zajednička svojstva i deliti ih sa podklasama?

To je ideja nasleđivanja. Objekti će definisati svoja jedinstvena svojstva i operacije i nasleđivati svog pretka. Ovo će smanjiti ponavljanje koda. Deljena svojstva se definišu samo u nad-objektu, samo jednom.

Da li je to moguće sa Go-om? Odgovor je ne. Videćemo važan koncept dizajna: ugrađivanje tipova. Glavni cilj ovog odeljka je da vidimo da li ugrađivanje tipova možemo posmatrati kao oblik nasleđivanja.

### Ugrađivanje tipa

Možete ugraditi tip u drugi tip. Uzmimo naš prethodni primer. Imamo strukturu tipa human sa dva polja:

```go
type Human struct {
    firstname string
    lastname  string
}
```

Ovo predstavlja čoveka. Dodajemo mu sposobnost hodanja:

```go
func (t Human) Walk() {
    fmt.Printf("I walk...\n")
}
```

Školski nastavnik i univerzitetski nastavnik su ljudi; možemo ugraditi tip "Human" u ta dva tipa:

```go
type SchoolTeacher struct {
    Human
}

type UniversityTeacher struct {
    Human
    department string
    specialty string
}
```

Da bismo to uradili, samo dodajemo ime tipa kao novo polje (bez imena).

Na taj način možemo definisati naša dva nastavnika:

```go
john := UniversityTeacher{Human: Human{ firstname: "John", lastname: "Doe"}, department: "Science", specialty: "Biology"}
marc := SchoolTeacher{Human: Human{firstname:"Marc",lastname:"Chair"}}
```

Metodu možemo nazvati "Walk on john and marc":

```go
// object-oriented/inheritance/single-type-embedding/main.go 
//...
func main() {
    john.Human.Walk()
    marc.Human.Walk()
    // or
    john.Walk()
    marc.Walk()
}
```

Što će izvesti:

```sh
I walk...
I walk...
```

### Ugrađivanje višestrukih tipova

Niste ograničeni na jedan tip ugrađen u strukturu tipa. Možete ugraditi nekoliko tipova. Hajde da predstavimo tip "Researcher":

```go
type Researcher struct {
    fieldOfResearch string
    numberOfPhdStudents int
}
```

"Researcher" ima oblast istraživanja i određeni broj doktoranata koje mora da prati. Univerzitetski profesori su takođe istraživači. Možemo ugraditi tip "Researcher" u tip "UniversityTeacher":

```go
// object-oriented/inheritance/multiple-type-Embedding/main.go 

type UniversityTeacher struct {
    Human
    Researcher
    department string
    specialty string
}
```

## Sukobi imena

Ali postoji važno ograničenje za ugrađivanje tipova: sukobi imena. Ako želite da ugradite dva tipa koja dolaze iz različitih paketa, ali dele isto ime, vaš program se neće kompajlirati. Uzmimo primer. Imate dva paketa packageA i packageB, svaki od njih deklariše tip pod nazivom MyType:

```go
package packageA

type MyType struct {
    id int
}

package packageB

type MyType struct {
    id int
}
```

Sada u vašem glavnom paketu definišete paket "MyOtherType" koji sadrži oba tipa:

```go
type MyOtherType struct {
    packageA.MyType
    packageB.MyType
}
```

Postoji dupliranje imena datoteke. Prva dva polja su ugrađena polja; stoga su oba imena MyType u strukturi tipa "MyOtherType". Nije moguće imati dva polja sa istim imenom. Vaš program se neće kompajlirati:

```sh
typeEmbedding/nameClash/main.go:10:10: duplicate field MyType

Compilation finished with exit code 2
```

### Direktan pristup imovini i operacijama

U drugim objektno orijentisanim jezicima, klasa B koja nasleđuje od A može direktno pristupiti svim svojstvima i operacijama definisanim u A.

Šta to znači? Uzmimo primer PHP-a. Definišemo klasu "A" koja ima metodu "sayHi" (public znači da joj spoljašnji svet može pristupiti, to je kao ime metode koje počinje velikim slovom):

```php
// PHP Code
class A
{
    public function sayHi()
    {
        echo 'Hi !';
    }

}
```

Zatim u istom skriptu definišemo klasu "B" koja nasleđuje od klase "A". Ključna reč u PHP-u je `extends`:

```php
// PHP Code
class B extends A
{
    public function sayGoodbye()
    {
        echo 'Goodbye !';
    }
}
```

Klasa "B" je naslednik klase A. Stoga možemo direktno pristupiti metodama klase "A" preko objekta tipa "B":

```php
// PHP Code
// $b is a new instance of class B
$b = new B();
// we call the method sayHi, which is defined in class A
$b->sayHi();
// We call the method sayGoodbye defined in the class B
$b->sayGoodbye();
```

Ovde imamo direktan pristup svojstvima i operacijama definisanim na klase "A" iz klase "B".

Možemo direktno pristupiti svojstvima ugrađenog tipa pomoću ugrađivanja tipa. Možemo napisati:

```go
john.Walk()
```

Takođe možemo napisati:

```go
john.Human.Walk()
```

### Kompozicija pre nasleđivanja

Ugrađivanje tipova i nasleđivanje imaju isti cilj: izbegavanje ponavljanja koda i maksimiziranje ponovne upotrebe koda. Ali kompozicija i nasleđivanje su dva veoma različita dizajna. Go pristup je kompozicija pre nasleđivanja.

Napisaću razliku koju su napravili autori veoma poznate knjige "Obrasci dizajna: Elementi objektno-orijentisanog softvera za višekratnu upotrebu":

- Nasleđivanje je strategija "ponovne upotrebe bele kutije": klase naslednici imaju pristup internim svojstvima svojih predaka; interna svojstva i operacije mogu biti vidljive klasi naslednici.

- Kompozicija je strategija "ponovne upotrebe crne kutije": Klase se komponuju zajedno, tip A koji ugrađuje tip B nema pristup svim internim svojstvima i operacijama. Javna polja i metode su dostupni; privatna nisu.

Uzmimo primer. Imamo struct "Cart" koji definiše tip struct "Cart":

```go
package cart

type Cart struct {
    ID     string
    locked bool
}
```

Ova struktura tipa ima dva polja sa imenima "ID" i "locked". Polje "ID" je izvezeno. Polje sa imenom "locked" nije dostupno iz drugog paketa.

U našem glavnom programu kreiramo još jedan strukturni tip pod nazivom "User":

```go
package main

type User struct {
    cart.Cart
}
```

Tip cart.Cart je ugrađen u User. Ako kreiramo promenljivu tipa User možemo pristupiti samo izvezenim poljima iz cart.Cart:

```go
func main() {
    u := User{}
    fmt.Println(u.Cart.ID)
}
```

Ako pozovemo

```go
fmt.Println(u.Cart.locked)
```

Naš program se neće kompajlirati sa sledećom porukom o grešci:

```sh
typeEmbedding/blackBoxReuse/main.go:17:27: u.Cart.locked undefined (cannot refer to unexported field or method private)
```

### Prednosti kompozicije u odnosu na nasleđivanje

- U modelu nasleđivanja, klasa naslednik može pristupiti unutrašnjim delovima klase pretka.
  Unutrašnji delovi klase pretka mogu biti vidljivi nasledniku; pravilo enkapsulacije je donekle prekršeno. Dok kod kompozicije ne možete pristupiti skrivenim delovima klase pretka. Ova tehnika osigurava da će korisnici vaših struktura koristiti samo izvezene metode i polja.

- Kompozicija je generalno jednostavnija od nasleđivanja. Ako ste iskusan programer, možda ste se
  suočili sa objektno orijentisanim dizajnom gde se nasleđivanje prekomerno koristi. U takvim slučajevima, projekat je teže razumeti, a početnici moraju da provode sate sa iskusnim programerima da bi razumeli graf klasa.

## Testirajte sebe

1. Generalno, koji je (su) tip (tipovi) povratka New funkcija?

   - Pokazivač
   - Opciono greška

2. Šta je enkapsulacija? Navedite primer.

   - Enkapsulacija je tehnika dizajniranja koja se koristi za grupisanje operacija i podataka u
     izolovane "kapsule" kodova.
   - Izolovano = pristup podacima i metodama je ograničen.
   - Promenljiva, konstanta, tip, funkcija ili metod definisan u Go paketu je dostupan u drugom
     paketu samo ako je izvezen.
     Npr.: pogledajte odlomak pitanja 4 ispod.
   - Tip Cart je izvezen. Funkcija total nije izvezena. Funkcija New je izvezena.
   - Iz drugog paketa, dostupni su samo CartiNew

3. Šta je polimorfizam? Objasnite to svojim rečima.

   - Reč polimorfan označava entitet koji može imati različite oblike.
   - Polimorfizam se postiže kada jedan simbol može predstavljati različite tipove.
   - Tip interfejsa može predstavljati različite tipove (implementacije).

4. Tačno ili netačno. Tip Go može nasleđivati od drugog tipa.

   - Netačno.
   - U Go-u ne postoji mehanizam nasleđivanja.
   - Međutim, Go dozvoljava sastavljanje tipova
   - Možete ugraditi tip u drugi tip.

   ```go
   // question 4 snippet
   package cart
   
   //...
   
   type Cart struct {
       ID string
       Items
   }
   
   func total() {
       //...
   }
   
   func New() *Cart {
       //...
   }
   ```

## Ključno

- Objektno orijentisan jezik je "programski jezik koji omogućava korisniku da izrazi program u
  smislu objekata i poruka između tih objekata". Objekti su instance klasa.
  - Klasa je korisnički definisani programski entitet sa ponašanjima i podacima.
  - U nekim jezicima, moraćete da definišete konstruktore. To su specifične metode namenjene
    inicijalizaciji objekta.
  - Objekat obuhvata podatke i ponašanja (metode)
- Objektno orijentisan jezik ima sledeće karakteristike:
  - Klase i objekti su gradivni blokovi programa.
  - Nudi određeni nivo apstrakcije
  - Obezbeđuje mehanizme enkapsulacije
  - Polimorfizam je moguć
  - Klase mogu nasleđivati jedna od druge
- Go ima tipove, ali nema klase.
  - Tip određuje "skup vrednosti" zajedno sa "metodama specifičnim za te vrednosti"
  - Neke biblioteke definišu `New` funkcije koje možemo asimilirati kao "konstruktore".
- Go nudi "apstrakciju" programerima. Na primer: ne morate znati stvarnu implementaciju metode da
  biste je pozvali.
- "Enkapsulacija" je karakteristika objektno orijentisanih jezika
- Go tipovi, promenljive i konstante mogu se eksportovati, što ih čini dostupnim iz drugog paketa.
  - Zahvaljujući ovom mehanizmu, kod može biti "enkapsuliran". Pristup detaljima implementacije
    može biti zabranjen pozivaocu.
- Polimorfizam se postiže kada jedan simbol može predstavljati različite tipove.
  - Tip interfejsa može predstavljati različite tipove (implementacije).
- U Go-u ne postoji mehanizam nasleđivanja. Međutim, tipovi se mogu sastavljati zajedno putem
  ugrađivanja tipova.
  - Nasleđivanje je strategija "ponovne upotrebe bele kutije" : Klase naslednici imaju pristup
    svim internim svojstvima svojih predaka, sva interna svojstva i operacije su vidljive klasi naslednici.
  - Kompozicija je strategija "ponovne upotrebe crne kutije" : Klase se komponuju zajedno, tip
    A koji ugrađuje tip B nema pristup svim internim svojstvima i operacijama. Javna polja i metode su dostupni, privatna nisu.

```go
package main

type User struct {
    // Cart is embedded into type User
    // this is called composition
    cart.Cart
}
func main() {
    u := User{}
    fmt.Println(u.Cart.ID)
}
```

[38 Generici][38] | [00 Sadržaj][00] | [40 Nadogradnja ili vraćanje na stariju verziju][40]  

[38]: 38_Generici.md
[00]: 00_Sadržaj.md
[40]: 40_Nadogradnja_ili_vraćanje_na%20staru_verziju.md
