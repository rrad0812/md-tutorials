
# 10 Funkcije

[09 Kontrolni iskazi][09] | [00 Sadržaj][00] | [11 Paketi i uvoz][11]

**Šta ćete naučiti u ovom poglavlju?**

- Zašto korišćenje funkcija može poboljšati vaš kod.
- Kako napisati funkciju.

**Obrađeni tehnički koncepti!**

- Potpis funkcije, telo, parametri, rezultati
- Izjava o povraćaju
- Poziv funkcije

## Šta je funkcija?

**Definicija**:
Funkcija je blok koda koji se poziva određenim imenom. Može primati ulazne vrednosti. Takođe može imati izlazne vrednosti.

**Rečnik**:
Hajde da uvedemo važne pojmove:

- Kada kreiramo funkciju, kažemo da je definišemo.
- Ulazne vrednosti su parametri funkcije.
- Izlazne vrednosti su rezultati funkcije.
- Kada koristimo funkciju, kažemo da je pozivamo.
- Korišćenje funkcije se naziva poziv funkcije.

**Primer**:
Evo jedne funkcije preuzete iz standardne biblioteke.

```go
// TrimSpace returns a slice of the string s, with all leading
// and trailing white space removed, as defined by Unicode.
func TrimSpace(s string) string {
   // Fast path for ASCII: look for the first ASCII non-space byte
   start := 0
   for ; start < len(s); start++ {
      c := s[start]
      if c >= utf8.RuneSelf {
         // If we run into a non-ASCII byte, fall back to the
         // slower Unicode-aware method on the remaining bytes
         return TrimFunc(s[start:], unicode.IsSpace)
      }
      if asciiSpace[c] == 0 {
         break
      }
   }

   // Now look for the first ASCII non-space byte from the end
   stop := len(s)
   for ; stop > start; stop-- {
      c := s[stop-1]
      if c >= utf8.RuneSelf {
         return TrimFunc(s[start:stop], unicode.IsSpace)
      }
      if asciiSpace[c] == 0 {
         break
      }
   }

   // At this point s[start:stop] starts and ends with an ASCII
   // non-space bytes, so we're done. Non-ASCII cases have already
   // been handled above.
   return s[start:stop]
}
```

Strukturu funkcije ćemo detaljno opisati u narednim odeljcima.

### Zašto su nam potrebne funkcije

#### Ponovna upotreba

Pomoću funkcija možete jednom napisati blok složenog koda, a zatim ga koristiti nekoliko puta u svojoj aplikaciji.

**Koristite kod drugih programera**:

U vašem go programu možete koristiti funkcije koje su napisali drugi programeri:

- Iz standardne biblioteke Go
- Iz projekata koje su objavili Go programeri

Videćemo kako da uradimo oba u narednim odeljcima.

#### Apstrakcija

Ovo smo radili u prethodnim blokovima koda koje smo napisali kada smo pozvali funkciju `Println` iz `fmt` paketa. Kada pozovete `Println`, pozivate ogroman blok koda. Ovaj blok koda nije vidljiv pozivaocu funkcije. Skriven je. I to je odlična stvar! Ne moramo znati unutrašnjost funkcije da bismo je koristili. Funkcije donose određeni nivo apstrakcije.

Funkcije su poput crnih kutija; detalji implementacije su skriveni od pozivaoca.

## Parametri i rezultati funkcija

Funkcija se može posmatrati kao mašina; ona prima ulaz, izvršava akciju i nešto izbacuje.

**Broj parametara, broj rezultata**:

- Broj parametara može da varira od 0 do N. Tehnički gledano, N može biti 100 ili čak 1000.
- Nije dobra praksa definisati mnogo parametara.  
  Po mom mišljenju, funkcija ne bi trebalo da ima više od tri parametra.
- Broj rezultata takođe može da varira od 0 do N.  
  Moja preporuka je da se ograničite na 2 rezultata.

## return iskaz

Iskaz `return` počinje ključnom reči `return`. Iskaz `return` se koristi u funkciji za:

- Prekidanje izvršavanja funkcije
- Opciono vraćanje rezultata funkcije

Videćemo na konkretnim primerima kako se koristi naredba za vraćanje.

## Funkcija sa 2 parametra, 1 rezultat

Funkcija se sastoji od:

- Imena (koje je identifikator)
- Liste parametara zatvorenih zagradama.
  - Svaki parametar ima ime i tip.
- Nakon liste parametara, nalazi se lista rezultata

**Primer deklaracije funkcije**:

```go
// functions/declaration-example/main.go
package main

// compute the price of a room based on its rate per night
// and the number of night
func computePrice(rate float32, nights int) float32 {
    return rate * float32(nights)
}
```

Hajde da detaljno objasnimo našu funkciju:

- Naziv funkcije jecomputePrice
- Potpis funkcije je `(rate float32, nights int) float32`
- Prvi parametar je imenovan "rate" i tipa je `float32`.
- Drugi parametar je imenovan "nights" i predstavlja `int`.
- Postoji jedan rezultat, njegov tip je `float32`
- Telo funkcije je
  
  ```go
  return rate * float32(nights)
  ```
  
- `return` je ključna reč: ona ukazuje Go-u da je ono što sledi naš rezultat.
- Rezultat je množenje `float32` između "rate" i "nights".  

Koja je svrha funkcije?

- Komentar ispred funkcije objašnjava šta funkcija radi
- Ime nam takođe daje naznaku; izračunava cenu.
- Potpis takođe daje informacije
  - Potrebna je cena i određeni broj noćenja
  - Vraća float32, cenu.

**Primer upotrebe**:

```go
// functions/usage-example/main.go
package main

import "fmt"

func main() {
    johnPrice := computePrice(145.90, 3) //*\label{funcEx1Us1}
    paulPrice := computePrice(26.32, 10) //*\label{funcEx1Us2}
    robPrice := computePrice(189.21, 2)  //*\label{funcEx1Us3}

    total := johnPrice + paulPrice + robPrice
    fmt.Printf("TOTAL : %0.2f $", total)
}

func computePrice(rate float32, nights int) float32 { //*\label{funcEx1Dec}
    return rate * float32(nights)
}
```

Rezultat "computePrice(145.90, 3)" se čuva u promenljivoj "johnPrice".
Rezultat "computePrice(26.32, 10)" se čuva u promenljivoj "paulPrice".
Rezultat "computePrice(189.21, 2)" se čuva u promenljivoj "robPrice".

Zatim kreiramo promenljivu "total" i dodeljujemo joj zbir "johnPrice", "paulPrice" i "robPrice".

**Brza pitanja**:

- Koji je tip "johnPrice", "paulPrice", "robPrice" i "total"?  
- U funkciji, zašto treba da napišemo "float32(nights)", zašto ne bismo jednostavno napisali nights?
- Možemo li pisati "computePrice(189.21, "two")" ?

**Odgovori**:

- `float32`  
  Funkcija vraća `float32`; dodeljujemo promenljivim `johnPrice`. `paulPrice` i `robPrice` povratnu vrednost ove funkcije. Kao posledica toga, to su `float32`.  
  Što se tiče `total`, sabiranje 3 `float32` vrednosti je `float32`.

- Promenljiva "nights" je ceo broj.  
  Potrebno je da konvertujemo "nights" u `float32` da bismo je pomnožili sa `float32`.  
  Kada množimo dve vrednosti, one treba da budu istog tipa.

- Drugi parametar funkcije "computePrice" je ceo broj. "two" je string.  
  Dajete funkciji parametar pogrešnog tipa.

## Opseg funkcije

Svaka funkcija ima opseg važenja. Opseg funkcije počinje početnom vitičastom zagradom `{` i završava se zatvarajućom vitičastom zagradom `}`. Opseg je deo izvornog koda. To je ekstent izvornog koda.

Unutar ovog bloka, definisani su parametri funkcije. Nismo morali da inicijalizujemo i dodeljujemo nijednu promenljivu: "rate" i "nights" se može slobodno koristiti:

```go
func computePrice(rate float32, nights int) float32 {
   return rate * float32(nights)
}
```

Mogu se koristiti samo u okviru funkcije. Nakon zatvarajuće vitičaste zagrade, oni ne postoje. Parametri postoje samo u okviru funkcije.

```go
// functions/scope/main.go
package main

import "fmt"

func main() {
    johnPrice := computePrice(145.90, 3)
    fmt.Println("John:", johnPrice, "rate:", rate)
}

func computePrice(rate float32, nights int) float32 {
    return rate * float32(nights)
}
```

Da li se prethodni kod kompajlira? Pa, ne kompajlira, pokušavamo da ispišemo promenljivu "rate" koja ne postoji u opsegu važenja funkcije `main`. Ali "rate" postoji u opsegu važenja funkcije "computePrice".

Ako pokušamo da kompajliramo prethodni program, dobićemo ovu grešku:

```sh
./main.go:7:43: undefined: rate
```

Opseg je ograničenje koje omogućava savršeno obuhvatanje funkcije. Spolja se ne može manipulisati unutrašnjošću. Opseg je poput metalnog štita mašine. Korisnik ne može manipulisati unutrašnjošću mašine.

Naravno, možete definisati i nove promenljive unutar funkcije:

```go
// functions/scope2/main.go
package main

import "fmt"

func main() {
    fmt.Println(computePrice(145.90, 3))
}

func computePrice(rate float32, nights int) float32 {
    n := float32(nights)
    return rate * n
}
```

Kreirali smo novu promenljivu n, koja je inicijalizovana vrednošću `float32` "nights". Ova promenljiva postoji samo u okviru funkcije `computePrice`.

## Funkcija sa 2 parametra, 1 imenovani rezultat

Go vam daje mogućnost da date ime rezultatima. U prethodnom odeljku, imali smo anonimni rezultat. Znali smo samo tip. Ovde navodimo njegovo ime i tip.

Zagrade koje okružuju rezultat i tip rezultata su obavezne.

**Primer deklaracije**:

```go
func computePrice(rate float32, nights int) (price float32) {
   price = rate * float32(nights)
   return
}
```

- U telu funkcije, dodeljujemo vrednost funkcijskom rezultat "price" = "rate * "nights", i on je `float32` tipa.

- Imajte na umu da promenljiva "price" već postoji; ne moramo je deklarisati

- Promenljiva "price" je inicijalizovana na nultu vrednost svog tipa (ovde float32 => 0)

- Takođe imajte na umu da jednostavno pozivamo samo return. Mogli smo da napišemo `return "price"`, ali to nije neophodno kada imenujete parametre.

- Kada se pozove funkcijski `return`, funkcija će vratiti trenutnu vrednost "price".

**Primer upotrebe**:

Sintaksa upotrebe je i dalje ista:

```go
johnPrice := computePrice(145.90, 3)
```

**Brza pitanja**:

```go
// functions/usage-example-2/main.go
package main

import "fmt"

func main() {
    johnPrice := computePrice(145.90, 3)
    fmt.Printf("TOTAL : %0.2f $", johnPrice)
}

// compute the price with a 200% margin
func computePrice(rate float32, nights int) (price float32) {
    p := rate * float32(nights)
    p = p * 2
    return
}
```

- Da li se ovaj kod kompajlira?
- Šta je izlaz ovog programa

**Odgovori**:

- Da
- TOTAL: 0,00 $

Zašto?
Funkcija "computePrice" ima imenovani rezultat "price". U telu funkcije nikada ne dodeljujemo "price" vrednost. Kada se `return` pozove, vraća se "price" vrednost. Vrednost se automatski inicijalizuje nultom vrednošću `float32` čija je vrednost 0.00. Funkcija vraća 0.00.

## Funkcija sa 0 parametara, 1 rezultat

Ovde imamo 0 parametara; funkcija ne prihvata nikakve ulazne podatke. Ali vraća nešto.

**Primer**:

```go
// functions/usage-example-3/main.go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    vacant := vacantRooms()
    fmt.Println("Vacant rooms:", vacant)
}

func vacantRooms() int {
    rand.Seed(time.Now().UTC().UnixNano())
    return rand.Intn(100)
}
```

- Definišemo funkciju: vacantRooms.
- Ova funkcija se poziva u glavnoj funkciji. Vraćena vrednost se čuva u promenljivoj pod nazivom "vacant".

**Brzo pitanje**:

- Da li je promenljiva "vacno" neophodna?

**Odgovor**:

- Ne. Promenljiva je deklarisana, ali se koristi samo jednom. Nije neophodno. Možemo direktno napisati:

  ```go
  func main() {
     fmt.Println("Vacant rooms:", vacantRooms())
  }
  ```
  
  Deklarisanje promenljive ima smisla kada treba da sačuvate rezultat funkcije da biste ga koristili na drugoj lokaciji u vašem programu.
  
## Funkcija sa 0 parametara, 0 rezultata

**Primer**:

```go
// functions/usage-example-4/main.go
package main

import (
    "fmt"
)

func main() {
    printHeader()
}

func printHeader() {
    fmt.Println("Hotel Golang")
    fmt.Println("San Francisco, CA")
}
```

Funkcija "printHeader" će ispisati "Hotel Golang", a zatim u drugom redu će ispisati "San Francisco, CA". U funkciji `main` (koja takođe uzima 0 parametara i ima 0 rezultata) pozivamo "printHeader()".

**Brza pitanja**:

- Možemo li dodati naredbu za vraćanje u funkciju printHeader?
- Sledeći kod definiše konstantu pod nazivom "HotelName". Konstanta se koristi unutar "printHeader" funkcije. Šta mislite o ovom kodu?

```go
// functions/quick-question/main.go
package main

import (
    "fmt"
)

func main() {
    const HotelName = "Golang"
    printHeader()
}

func printHeader() {
    fmt.Println("Hotel", HotelName)
    fmt.Println("San Francisco, CA")
}
```

12.1.0.2 Odgovori

- Funkcija nema rezultat. Dodavanje naredbe `return` funkciji je opciono. Kada program dođe do kraja funkcije, prekinuće njeno izvršavanje.

- Konstanta "HotelNameje" definisana u opsegu važenja funkcije `main`. Ova konstanta ne postoji u opsegu važenja funkcije "printHeader". Program se neće kompajlirati!

## Funkcija main

Prva funkcija koju smo koristili bila je `main` function. `main` funkcija (iz `main` paketa) nema parametre niti rezultat. Ovo je ulazna tačka programa.

```go
package main

func main() {

}
```

## Praktična primena

Nastavićemo da razvijamo aplikaciju za upravljanje hotelom. Na slici 1 možete videti žičane okvire aplikacije:

Bacite pogled na sledeći kod:

```go
package main

import (
    "fmt"
    "math/rand"
    "time"
)

func main() {
    const hotelName = "Gopher Paris Inn"
    const totalRooms = 134
    const firstRoomNumber = 110

    rand.Seed(time.Now().UTC().UnixNano())
    roomsOccupied := rand.Intn(totalRooms)
    roomsAvailable := totalRooms - roomsOccupied

    occupancyRate := (float32(roomsOccupied) / float32(totalRooms)) * 100
    var occupancyLevel string
    if occupancyRate > 70 {
        occupancyLevel = "High"
    } else if occupancyRate > 20 {
        occupancyLevel = "Medium"
    } else {
        occupancyLevel = "Low"
    }

    fmt.Println("Hotel:", hotelName)
    fmt.Println("Number of rooms", totalRooms)
    fmt.Println("Rooms available", roomsAvailable)
    fmt.Println("                  Occupancy Level:", occupancyLevel)
    fmt.Printf("                  Occupancy Rate: %0.2f %%\n", occupancyRate)

    if roomsAvailable > 0 {
        fmt.Println("Rooms:")
        for i := 0; roomsAvailable > i; i++ {
            roomNumber := firstRoomNumber + i
            size := rand.Intn(6) + 1
            nights := rand.Intn(10) + 1
            fmt.Println(roomNumber, ":", size, "people /", nights, " nights ")
        }
    } else {
        fmt.Println("No rooms available for tonight")
    }

}
```

Refaktorišite ovaj kod i kreirajte funkcije koje će:

- Izračunati nivo popunjenosti
- Izračunati stopu popunjenosti
- Odštampati detalje određene sobe.

**Rešenje**:

Tri nove funkcije:

```go
//....

// display information about a room
func printRoomDetails(roomNumber, size, nights int) {
   fmt.Println(roomNumber, ":", size, "people /", nights, " nights ")
}

// retrieve occupancyLevel from an occupancyRate
// From 0% to 30% occupancy rate return Low
// From 30% to 60% occupancy rate return Medium
// From 60% to 100% occupancy rate return High
func occupancyLevel(occupancyRate float32) string {
   if occupancyRate > 70 {
      return "High"
   } else if occupancyRate > 20 {
      return "Medium"
   } else {
      return "Low"
   }
}

// compute the hotel occupancy rate
// return a percentage
// ex : 14,43 => 14,43%
func occupancyRate(roomsOccupied int, totalRooms int) float32 {
   return (float32(roomsOccupied) / float32(totalRooms)) * 100
}
```

**main funkcija (ulazna tačka programa)**:

```go
package main

import (
   "fmt"
   "math/rand"
   "time"
)

func main() {
   const hotelName = "Gopher Paris Inn"
   const totalRooms = 134
   const firstRoomNumber = 110

   rand.Seed(time.Now().UTC().UnixNano())
   roomsOccupied := rand.Intn(totalRooms)
   roomsAvailable := totalRooms - roomsOccupied

   occupancyRate := occupancyRate(roomsOccupied, totalRooms)
   occupancyLevel := occupancyLevel(occupancyRate)

   fmt.Println("Hotel:", hotelName)
   fmt.Println("Number of rooms", totalRooms)
   fmt.Println("Rooms available", roomsAvailable)
   fmt.Println("                  Occupancy Level:", occupancyLevel)
   fmt.Printf("                  Occupancy Rate: %0.2f %%\n", occupancyRate)

   if roomsAvailable > 0 {
      fmt.Println("Rooms:")
      for i := 0; roomsAvailable > i; i++ {
         roomNumber := firstRoomNumber + i
         size := rand.Intn(6) + 1
         nights := rand.Intn(10) + 1
         printRoomDetails(roomNumber, size, nights)
      }
   } else {
      fmt.Println("No rooms available for tonight")
   }
}
```

**Objašnjenja**:

Hajde da detaljno razmotrimo konstrukciju svake funkcije:

- **printRoomDetails**

  ```go
  // display information about a room
  func printRoomDetails(roomNumber, size, nights int) {
     fmt.Println(roomNumber, ":", size, "people /", nights, " nights ")
  }
  ```

  Ova funkcija prima tri parametra: "roomNumber", "size" i "nights". Ta tri parametra su tipa `int`. Funkcija ne vraća ništa.
  
  Kada su parametri istog tipa, možemo napisati:
  
  ```go
  func printRoomDetails(roomNumber, size, nights int)
  ```
  
  umesto:
  
  ```go
  func printRoomDetails(roomNumber int, size int, nights int)
  ```
  
  Ova poslednja notacija je sasvim legalna, ali zahteva više znakova i više truda čitaoca da razume.
  
  Telo funkcije je poziv funkcije `fmt.Println`. Ispisujemo:
  
  - vrednost "roomNumber"
  - ":"
  - vrednost "size"
  - "people /"
  - "nights"
  - " nights "
  
  Kada je vrednost roomNumber 169, vrednost size četiri, a vrednost nights 5: funkcija će ispisati:
  
  ```sh
  169 : 4 people / 5  nights
  ```
  
  Kada je vrednost broja noći jednaka 1 (i ostale promenljive imaju istu vrednost), rezultat je:
  
  ```sh
  169 : 4 people / 1  nights
  ```
  
  To je greška! Mogli smo da dizajniramo naš program da ispisuje "night" bez "s" ako je broj noći jednak 1.
  
  **Izazov - množina naspram jednine**:
  
  - Napišite drugu verziju funkcije "printRoomDetails" koja ispravlja grešku u množini i jednini.
  - Tipovi ulaznih parametara su sumnjivi. Zašto?
  
- **Nivo popunjenosti**

  ```go
  // retrieve occupancyLevel from an occupancyRate
  // From 0% to 30% occupancy rate return Low
  // From 30% to 60% occupancy rate return Medium
  // From 60% to 100% occupancy rate return High
  func occupancyLevel(occupancyRate float32) string {
     if occupancyRate > 70 {
        return "High"
     } else if occupancyRate > 20 {
        return "Medium"
     } else {
        return "Low"
     }
  }
  ```
  
  Ovde funkcija prima float32 parametar kao ulaz. Naziv ovog parametra je "occupancyRate". Funkcija vraća `string`.
  
  Imamo konstrukciju `if / else if / else`. Ona je prilagođena u ovom slučaju gde imamo tri moguća ishoda.

  **Izazov**:  
  Napišite drugu verziju funkcije "occupancyLevel" koja koristi `switch/case` umesto `if / else if / else` strukture.

- **Stopa popunjenosti**

  ```go
  // compute the hotel occupancy rate
  // return a percentage
  // ex : 14,43 => 14,43%
  func occupancyRate(roomsOccupied int, totalRooms int) float32 {
     return (float32(roomsOccupied) / float32(totalRooms)) * 100
  }
  ```

  Uzimamo dva cela broja kao ulaz. Pošto su dva parametra istog tipa, mogli smo napisati:
  
  ```go
  func occupancyRate(roomsOccupied, totalRooms int) float32
  ```
  
  Telo funkcije se sastoji od `return` iskaza. Vraća se rezultat izračunavanja.(`float32`)
  "(roomsOccupied) / float32(totalRooms)) * 100"
  
  **float32(X)**:
  
  `float32(X)` će konvertovati promenljivu "X" u `float32` tip. Možemo konvertovati samo numeričke tipove u `float32`:
  
  ```go
  // try to convert a string to a float32!
  float32("test")
  ```
  
  Neće se kompajlirati! Poruka o grešci je:
  
  ```sh
  "cannot convert test (type untyped string) to type float32"
  ```
  
- **Rešenja izazova**

  - Množina naspram jednine

    Da biste otkrili da li je količina u jednini, morate je uporediti sa 1. Ako je količina 1, onda će vaša reč biti u jednini.

    - Prvo rešenje

      ```go
      // functions/plural-singular-sol-1/main.go
      package main
      
      import "fmt"
      
      func main() {
          printRoomDetails(112, 2, 2)
      }
      
      // display information about a room
      func printRoomDetails(roomNumber, size, nights int) {
          nightText := "nights"
          if nights == 1 {
              nightText = "night"
          }
          fmt.Println(roomNumber, ":", size, "people /", nights, nightText)
      }
      ```

      Ovde definišemo promenljivu "nightText". Inicijalizujemo je vrednošću "nights". Zatim dodajemo `if`. Ako je "nights" jednako 1, onda se vrednost promenljive "nightText" prepisuje na "night"

      Zatim se promenljiva `nightText` koristi kao parametar `fmt.Println`.

    - Drugo rešenje (nije najbolje)

      ```go
      // functions/plural-singular-sol-2/main.go
      package main
      
      import "fmt"
      
      func main() {
          printRoomDetails2(112, 3, 4)
      }
      
      // display information about a room
      // This alternative works, but the code is duplicated...
      func printRoomDetails2(roomNumber, size, nights int) {
          if nights == 1 {
              fmt.Println(roomNumber, ":", size, "people /", nights, "night")
          } else {
              // practically the same line as before!
              // avoid this
              fmt.Println(roomNumber, ":", size, "people /", nights, "nights")
          }
      }
      ```

      Ovde ne definišemo nijednu promenljivu. Umesto toga imamo `if / else` konstrukciju. Ovaj kod će raditi. Ali nije optimalan:

      - Konstrukcija `if` je lakša za čitanje od konstrukcije if/else. U konstrukciji `if-else`, čitalac vašeg koda će morati da razmisli o suprotnom od jednakosti 1.
      - Imamo dupliranje koda. Poziv funkcije `fmt.Println` se ponavlja, i ta dva poziva su praktično ista.
      - Šta ako nightsje jednako 0?
  
  - O tipu ulaznih parametara:

    Mogli bismo zameniti ceo broj neoznačenim celim brojem. Broj sobe (roomNumber), veličina (size), broj noćenja (nights) su uvek pozitivni... Ako zadržimo tip int, mogli bismo imati negativan broj noćenja, što je nemoguće i može prouzrokovati ozbiljne greške u sistemu. Mogli bismo generisati negativne cene, kao da dajemo novac mušteriji da ostane u našem hotelu!

  - `Swich/case`

    Kada imate `if / else if / els`e, možete ga zameniti sa `switch case`-om.

    ```go
    func occupancyLevel(occupancyRate float32) string {
       switch {
       case occupancyRate > 70:
          return "High"
       case occupancyRate > 20:
          return "Medium"
       default:
          return "Low"
    
       }
    }
    ```

    Ovde nije dat izraz u zaglavlju switch case-a. Ovo je ekvivalentnosa :

    ```go
    switch true{
       case occupancyRate > 70:
       //..
       }
    ```

    Program će:

    - Izračunati bulov izraz "occupancyRate > 70". Ako je true, vratiće string "High". Ako nije, izvršava se sledeći korak.
    - Izračunati bulov izraz occupancyRate > 20. Ako je true, onda će vratiti string "Medium".
    - Kada su dva prethodna dva izraza false, izvršava se default blok, koji vraća string "Low".

    Imajte na umu da će naredba `return` prekinuti izvršavanje funkcije.

## Saveti

Imena funkcija treba jasno da odražavaju operaciju koju će funkcija izvršiti

- Izbegavajte imena sa jednim slovom
- Koristite camilCase pri imenovanju funkcija
- Ne bi trebalo da ime funkcije sadrži više od dve reči  
  "sendEmail" je dobro, "sendEmailToTheOperationalTeam" nije baš dobro.
- Nazivi parametara i rezultata takođe treba da prenose informacije. Pratite isti savet koji je dat ranije.
- Izbegavajte pominjanje tipa u identifikatoru.  
  "descriptionString" je loše, "description" je bolje.
- Pažljivo birajte tipove parametara i tipove povratnih vrednosti. Dobro odabrani tipovi poboljšavaju čitljivost vašeg koda.  
  Npr: "nights" je neoznačeni ceo broj (uint), a ne int. Negativan broj noći ne postoji (barem u našoj ljudskoj dimenziji)

## Testirajte sebe

### Pitanja i odgovori

1. Funkcija treba da ima barem jedan parametar. Tačno ili netačno?
   - Netačno. Funkcija može imati 0 parametara.
2. U funkciji, koja je svrha naredbe return?
   - Iskaz `return` zaustavlja izvršavanje funkcije.  
   - Takođe opciono pruža jednu ili više vrednosti rezultata
3. Koja je razlika između liste rezultata i liste parametara?
   - Obe su liste identifikatora i tipova  
   - Lista rezultata je izlaz funkcije, ono što će funkcija vratiti kada
     se pozove  
   - Lista parametara je ulaz funkcije. Unutrašnja logika funkcije će  
     koristiti ulazne parametre.
4. Možemo li pisati Go programe bez funkcija?
   - Možete, ali će vam biti potrebna barem main funkcija da biste
     pokrenuli program.
   - Funkcije su odličan alat za izradu programa; možete ih posmatrati
     kao mašine koje se mogu pozivati svuda.
5. Popunite prazninu: Da bih pokrenuo izvršavanje funkcije, potrebno je
   da je ______.
   - pozovem
6. Kada definišem funkciju, ona će biti pokrenuta na početku mog
   programa. Tačno ili netačno?
   - Ovo je netačno. Čin definisanja funkcije se razlikuje od pozivanja
     funkcije. Funkcija će biti izvršena kada se pozove.

### Ključno

- Funkcije su praktični gradivni blokovi programa
- Funkcija je kao "mašina":
- Projektovanje / Izrada mašine => definisanje funkcije
- Pokrenite mašinu => pozovite funkciju
- Dajte svojim funkcijama smislena i kratka imena
- Funkcije mogu imati
  - Parametre: ulaz
  - Rezultate: izlaz
- Naredba return prekida izvršavanje funkcije i opciono dostavlja rezultate pozivaocu.

[09 Kontrolni iskazi][09] | [00 Sadržaj][00] | [11 Paketi i uvoz][11]

[09]: 09_Kontrolni_izrazi.md
[00]: 00_Sadržaj.md
[11]: 11_Paketi_i_uvoz.md
