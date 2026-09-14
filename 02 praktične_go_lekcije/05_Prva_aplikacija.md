
# 5 Aplikacija za prvi pokušaj

[04 Podesite svoje okruženje][04] | [00 Sadržaj][00] | [06 Binarni i decimalni][06]

**Šta ćete naučiti u ovom poglavlju?**

- Napišite jednostavan program koristeći Go
- Kompajlirajte program

**Obrađeni tehnički koncepti!**

- Specifikacije
- Kompilacija
- Binarna / izvršna datoteka

## Uvod

U ovom poglavlju ćemo napisati naše dve prve Go aplikacije.

## Prva Go aplikacija

### Specifikacija primene

Pre nego što napišemo bilo koji kod, moramo da odlučimo šta će naša aplikacija raditi. Većina projekata počinje ovom fazom. Ovo se naziva faza specifikacije. Cilj joj je da se proizvedu precizni zahtevi koje aplikacija treba da ispuni. Ti zahtevi su specifikacije (ili tzv. specs).

Specifikacija naše aplikacije je jednostavna: kada se pokrene, aplikacija treba da prikaže datum i vreme i da se zatvori.

### Direktorijum projekata

Go aplikacija je sastavljena od datoteka. U tim datotekama ćemo pisati Go kod. Te datoteke nazivamo "izvorne datoteke". Aplikacija se čuva u glavnom direktorijumu. Ovaj glavni direktorijum može da sadrži samo jednu izvornu datoteku. Većinu vremena je sastavljen od nekoliko poddirektorijuma.

Napravićemo ovaj glavni direktorijum za našu aplikaciju. To možete uraditi pomoću komandne linije (ili pomoću grafičkog interfejsa vašeg sistema):

```sh
cd Documents/code
mkdir dateAndTime
```

### IDE

Samo nekoliko minuta nas deli od pisanja koda. Da li vam je potreban poseban softver za pisanje Go koda? Teoretski, možete pisati kod pomoću standardnog uređivača teksta. Na tržištu postoji namenski softver posebno razvijen za programere. Zovu se IDE: Integrated Development Environment (Integrisano razvojno okruženje).

IDE pruža funkcionalnosti kao što su:

- Automatsko obojavanje rezervisanih reči (isticanje sintakse)
- Automatsko dovršavanje
- Mogućnosti refaktorisanja, itd.

Na tržištu postoji mnogo IDE-a. Možete pretražiti Google da biste pronašli ono koje vam najbolje odgovara. Ja koristim Goland, koji razvija IntelliJ. Ovaj softver nije besplatan (baziran je na pretplati), ali mi se čini lakim za korišćenje.  

### Izvorna datoteka

Hajde da kreiramo naš izvorni fajl. Nazvaćemo ga `main.go`.

```go
// first-go-application/first/main.go
package main

import (
    "fmt"
    "time"
)

func main() {
    now := time.Now()
    fmt.Println(now)
}
```

#### Objašnjenje koda

- Prvi red je obavezan u programskom jeziku Go. U svim datotekama morate dodati deklaraciju paketa. Takva deklaracija se sastoji od ključne reči `package`, a zatim imena paketa.

- U drugom redu možete pročitati još jednu ključnu reč: `import`. Obično je praćena otvorenom zagradom i listom uvezenih paketa programa. Svaki paket se piše u novom redu. Svaki paket ima ime razdvojeno dvostrukim navodnicima. Ovde naša aplikacija zavisi od dva paketa:

  - `fmt`
  - `time`

  Ti paketi su deo standardne biblioteke.

- Zatim pronalazite deklaraciju funkcije pod nazivom `main`. Kasnije ćemo detaljnije razmotriti sintaksu funkcija.  
  - Deklaracija funkcije je zatvorena vitičastim zagradama: `{` i `}`.  
  - Unutar deklaracije funkcije imamo dve izjave:  
    - Prva instrukcija je afektacija. Inicijalizujemo promenljivu "now" i dajemo joj vrednost koju vraća poziv funkcije `Now()` iz paketa `time`.
    - Druga instrukcija je poziv funkcije `Println` iz paketa `fmt`.

    UPOZORENJE! Uverite se da koristite samo pakete koje zapravo uvozite. U suprotnom, vaš program se neće kompajlirati... Kada koristite paket koji nije uvezen, vaš Go program se neće kompajlirati.

    Uverite se da koristite uvezene pakete. U sledećem primeru koda uvezem paket fmt, time ali ga ne koristim u svojoj main funkciji:

    ```go
    // DO NOT COMPILE
    // first-go-application/import-issue/main.go
    package main
    
    import (
        "fmt"
        "time"
    )
    
    func main(){
    
    }
    ```

#### O `main` funkciji

`Main` funkcija je ulazna tačka programa. U svakoj aplikaciji imate barem jednu `main` funkciju. Program će početi sa prvom izjavom ove funkcije. (Imajte na umu da u C, C++, Javi postoji koncept glavne funkcije).

### Kompilacija

Izvorni fajl je spreman za transformaciju u binarni (ili izvršni) fajl. Da bismo to uradili, koristićemo Go alatku. Otvorite terminal:

```sh
cd Documents/code/dateAndTime
go build main.go
```

Prva instrukcija ( cd ) nalaže šelu da promeni trenutni direktorijum u `Documents/code/dateAndTime`. Druga komanda će kompajlirati program u izvršnu datoteku. Izvršna datoteka se zove `main` (isto ime kao i izvorna datoteka, bez ekstenzije.go). Pogledajmo datoteke koje se sada nalaze u našem direktorijumu dateAndTime:

```sh
$ ls -lh
total 4160
-rwxr-xr-x  1 maximilienandile  staff   2.0M Aug 16 11:27 main
-rw-r--r--  1 maximilienandile  staff    94B Aug 16 11:00 main.go
```

Koristimo komandu `ls`. (za korisnike Windows-a, možete koristiti komandu `dir` ). Možete videti da imamo dve datoteke:

- `main` koja teži 2,0 M (megabajta) i koja je izvršna.
- `main.go` koji teži samo 94 bajta. (ovo je naš izvorni fajl).

Sada je vreme da pokrenemo našu aplikaciju:

```sh
$./main
2019-08-16 11:45:44.435637 +0200 CEST m=+0.000263533
```

Čestitamo na vašoj prvoj Go aplikaciji!

## Testirajte sebe

### Pitanja i odgovori

1. Kako kompajlirati Go aplikaciju?
   - Otvorite terminal
   - Idite u direktorijum vaše aplikacije.
   - Pod uslovom da postoji datoteka pod nazivom main.go koja sadrži glavnu  
     funkciju, izdajte sledeće komande:

     ```sh
     cd /code/myApp
     go build main.go
     ```

2. Kako se naziva rezultat kompilacije?
   - Zove se izvršna ili binarna datoteka
3. Kako se zove funkcija ulazne tačke Go aplikacije?
   - main
4. Koja je upotreba naredbe za uvoz?
   - Koristi se za uvoz paketa (iz standardne biblioteke ili drugih izvora) u
     program.
   - Uvezeni paket se zatim može koristiti unutar koda.

### Vežba

#### Specifikacije

Napravite go aplikaciju koja prikazuje string „Hello World“ na ekranu i zatvara se.

```go
// first-go-application/hello-world/main.go
package main

import "fmt"

func main() {
    fmt.Println("Hello World")
}
```

Napomene:  

- Ovde možete videti da koristimo samo jednu zavisnost. Deo programa za uvoz je promenjen; zagrade nisu potrebne kada se uvozi samo jedan paket.
- Imamo main funkciju. I jednu izjavu unutra. Ovde se poziva funkcija Println. Ova funkcija je deo paketa fmt (fmt je skraćenica od „formatiranje“).
- Paket fmt je deo standardne biblioteke.

### Ključno

- Da biste napravili jednostavan program
  - Napravite datoteku
  - Nazovite ga main.go
  - Evo osnovnog skeleta programa (koji ne radi ništa)

    ```go
    // first-go-application/skeleton/main.go
    package main
    
    func main() {
        
    }
    ```

  - Ova datoteka je označena kao "izvorna datoteka".  
  - Iz ove izvorne datoteke možemo kreirati izvršni program (koji se može
    pokrenuti).
  - Kreiranje izvršne datoteke naziva se „kompilacija“.
  - Da biste kompajlirali program, unesite sledeću komandu u terminal

    ```sh
    go build main.go
    ```

    Da biste pokrenuli kompajlirani program, unesite sledeće u terminal:

    ```sh
    ./main
    ```

[04 Podesite svoje okruženje][04] | [00 Sadržaj][00] | [06 Binarni i decimalni][06]

[04]: 04_Podesite_svoje_okružnje.md
[00]: 00_Sadržaj.md
[06]: 06_Binarni_i_decimalni.md
