
# Uvod u elemente izvornog koda

[Go alati][0002] | [Sadržaj][00] | [Ključne reči i indentifikatori][0102]

Go je poznat po svojoj jednostavnoj i čistoj sintaksi. Ovaj članak predstavlja uobičajene elemente izvornog koda u programiranju kroz jednostavan primer. Ovo će pomoći novim "goferima" (Go programerima) da steknu osnovnu predstavu o upotrebi Go elemenata.

## Elementi programiranja i izvornog koda

Programiranje se može posmatrati kao manipulisanje operacijama na sve vrste načina kako bi se postigli određeni ciljevi. Operacije zapisuju podatke na i čitaju podatke sa hardverskih uređaja radi završetka zadataka. Za moderne računare, osnovne operacije su instrukcije niskog nivoa procesora i grafičke kartice. Uobičajeni hardverski uređaji uključuju memoriju, disk, mrežnu karticu, grafičku karticu, monitor, tastaturu i miš itd.

Programiranje direktnim manipulisanjem instrukcijama niskog nivoa je zamorno i sklono greškama. Programski jezici visokog nivoa prave neke enkapsulacije za operacije niskog nivoa i neke apstrakte za podatke, kako bi programiranje bilo intuitivnije i prilagođenije ljudima.

U popularnim programskim jezicima visokog nivoa, operacije se uglavnom postižu pozivanjem funkcija i korišćenjem operatora. Većina popularnih programskih jezika visokog nivoa podržava nekoliko vrsta uslovnih i petljičnih kontrolnih tokova, možemo ih smatrati specijalnim operacijama. Sintaksa ovih kontrolnih tokova je bliska ljudskom jeziku tako da je kod koji pišu programeri lak za razumevanje.

Podaci se apstrahuju kao tipovi i vrednosti u većini programskih jezika visokog nivoa. Tipovi se mogu posmatrati kao šabloni vrednosti, a vrednosti se mogu posmatrati kao instance tipova. Većina jezika podržava nekoliko ugrađenih tipova, a takođe podržava i prilagođene tipove. Sistem tipova programskog jezika je duh jezika.

U programiranju se može koristiti veliki broj vrednosti. Neke od njih se mogu direktno predstaviti svojim literalima (tekstualnim reprezentacijama), ali druge ne. Da bi programiranje bilo fleksibilno i manje podložno greškama, mnoge vrednosti su imenovane. Takve vrednosti uključuju promenljive i imenovane konstante.

Imenovane funkcije, imenovane vrednosti (uključujući promenljive i imenovane konstante), definisani tipovi i alijasi tipova nazivaju se elementi koda. Imena elemenata koda moraju biti identifikatori. Imena paketa i imena uvoza paketa takođe treba da budu identifikatori.

Kompajleri će prevesti programski kod visokog nivoa u instrukcije procesora niskog nivoa radi izvršavanja. Da bi se kompajlerima pomoglo u parsiranju programskog koda visokog nivoa, mnoge reči su rezervisane kako bi se sprečilo njihovo korišćenje kao identifikatora. Takve reči se nazivaju ključne reči.

Mnogi moderni programski jezici visokog nivoa koriste pakete za organizovanje koda. Paket mora da uveze drugi paket da bi koristio izvezene (javne) elemente koda u drugom paketu. Imena paketa i imena uvezenih paketa takođe treba da budu identifikatori.

Iako je kod napisan u programskim jezicima visokog nivoa razumljiviji od mašinskih jezika niskog nivoa, i dalje su nam potrebni neki komentari za neki kod kako bismo objasnili logiku. Primer programa u sledećem odeljku sadrži mnogo komentara.

## Jednostavan Go demo program

Hajde da pogledamo kratak demo program za Go da bismo se upoznali sa svim vrstama elemenata koda u Gou. Kao i u nekim drugim jezicima, u Gou, komentari redova počinju sa //, a svaki komentar bloka je zatvoren parom slovnih znakova /*i*/.

Ispod je demo Go program. Molimo vas da pročitate komentare za objašnjenja. Više objašnjenja sledi nakon programa.

```go
package main // specify the source file's package                   [1]

import "math/rand" // import a standard package                     [3]

const MaxRnd = 16 // a named constant declaration                   [5]

// A function declaration
/*
 StatRandomNumbers produces a certain number of
 non-negative random integers which are less than
 MaxRnd, then counts and returns the numbers of
 small and large ones among the produced randoms.
 n specifies how many randoms to be produced.
*/
func StatRandomNumbers(n int) (int, int) {                          [15]
    
    // Declare two variables (both as 0).
    var a, b int                                                    [18]
    
    // A for-loop control flow.
    for i := 0; i < n; i++ {                                        [21]
    
        // An if-else control flow.
        if rand.Intn(MaxRnd) < MaxRnd/2 {                           [24]
            a = a + 1                                               [25]
        } else {                                                    
            b++ // same as: b = b + 1                               [27]
        }
    }
    return a, b // this function return two results                 [30]
}

// "main" function is the entry function of a program.
func main() {                                                       [35]
    var num = 100                                                   [36]
    
    // Call the declared StatRandomNumbers function.
    x, y := StatRandomNumbers(num)                                  [37]
    
    // Call two built-in functions (print and println).
    print("Result: ", x, " + ", y, " = ", num, "? ")                [40]
    println(x+y == num)                                             [41]
}
```

Sačuvajte gornji izvorni kod u datoteku pod nazivom *basic-code-element-demo.go* i pokrenite ovaj program. Dobićemo sledeći sličan izlaz:

```go
go run basic-code-element-demo.go
Rezultat: 46 + 54 = 100? true
```

U gornjem programu, **package**, **import**, **const**, **func**, **var**, **for**, **if**, **else**, i **return** su ključne reči. Većina ostalih reči u programu su **identifikatori**. Molimo vas da pročitate ključne reči i identifikatori za više informacija o ključnim rečima i identifikatorima.

**int** reči označavaju ugrađeni **int** tip, jedan od mnogih vrsta celobrojnih tipova u Gou. Reči **16**, **0**, **2**, **1** i **100** su **celobrojni literali**. Reč **"Result: "** je **string literal**. Molimo vas da pročitate osnovne tipove i njihove vrednosne literale za više informacija o gore navedenim ugrađenim osnovnim tipovima i njihovim vrednosnim literalima. Neki drugi tipovi (kompozitni tipovi) biće predstavljeni kasnije u drugim člancima.

**a = a + 1** je **dodela vrednosti**. **const MaxRand = 16** deklariše **imenovanu konstantu**, *MaxRnd*. Imamo deklaraciju tri **promenljive** *a*, *b* i  *num**, koristeći standardni oblik deklaracije promenljivih. Promenljive *i*,  *x* i *y* edu 37 su deklarisane pomoću **kratkog oblika deklaracije promenljivih**. Naveli smo tip za promenljive *a* i *b* kao **int**. Go kompajler će zaključiti da su tipovi *i*, *num*, *x* i *y* svi *int*, jer su inicijalizovani celobrojnim literalima. Molimo vas da pročitate konstante i promenljive za više informacija o netipizovanim vrednostima, određivanju tipa, dodeljivanju vrednosti i kako deklarisati promenljive i imenovane konstante.

U programu se koristi mnogo operatora, kao što su operator poređenja *manje od* **<**, operator *jednako* **==** i operator *sabiranja*  **+**2. Vrednosti uvedene u operatorskoj operaciji se nazivaju **operandi**. Izraz **" + "** nije operator, to je jedan znak u u literalu stringa. Molimo vas da pročitate uobičajene operatore za više informacija. Više operatora biće predstavljeno u drugim člancima kasnije.

Imamo pozive dve ugrađene funkcije, **print** i **println**. Prilagođena funkcija *StatRandomNumbers* je deklarisana i definisanaa. Nadaje pozivamo funkciju, **Intn**, koja je funkcija deklarisana u **math/rands** standardnom paketu. Poziv funkcije je **operacija** funkcije. Ulazne vrednosti koje se koriste u pozivu funkcije nazivaju se **argumenti**. Molimo vas da pročitate deklaracije i pozive funkcija za više informacija.

> [!Note]
> Ugrađene funkcije **print** i **println** se ne preporučuju za upotrebu u
> formalnom Go programiranju. Umesto toga, u formalnim Go projektima treba
> koristiti odgovarajuće funkcije iz **fmt** standardnog paketa. U Go 101, ove
> dve funkcije se koriste samo u nekoliko početnih članaka.

Na početku navodi se ime paketa trenutne izvorne datoteke. Funkcija **main** unosa mora biti deklarisana u paketu koji se takođe zove **main**. Nadalje ide uvoz paketa, **math/rand** standardni paket koda. Njegovo ime za uvoz je **rand**. Funkcija **Intn** deklarisana u ovom standardnom paketu poziva se kao **rand.intn**. Molimo vas da pročitate pakete koda i uvoz paketa za više informacija o tome kako organizovati pakete koda i uvoziti pakete.

Članak o izrazima, iskazima i jednostavnim iskazima će predstaviti šta su izrazi i iskazi. Posebno su navedene sve vrste jednostavnih iskaza, koji su specijalni iskazi. Neki delovi svih vrsta tokova upravljanja moraju biti jednostavni iskazi, a neki delovi moraju biti izrazi.

U telu **StatRandomNumbers** funkcije koriste se dve kontrole toka. Jedna je **for** kontrola toka petlje, koji ugnežđuje drugi, **if-else** uslovni kontrolni tok. Molimo vas da pročitate osnovne kontrolne tokove za više informacija o svim vrstama osnovnih kontrolnih tokova. Neki drugi posebni kontrolni tokovi biće predstavljeni u drugim člancima kasnije.

U gornjem programu su korišćeni prazni redovi radi poboljšanja čitljivosti koda. A pošto je ovaj program namenjen uvodu u elemente koda, u njemu postoji mnogo komentara. Osim komentara u dokumentaciji za funkciju **StatRandomNumbers**, ostali komentari su samo u svrhu demonstracije. Trebalo bi da pokušamo da kod bude samorazumljiv i da koristimo samo neophodne komentare u formalnim projektima.

## O prelomima redova

Kao i mnogi drugi jezici, Go takođe koristi par vitičastih zagrada **{** i **}** da bi formirao eksplicitni blok koda. Međutim, u Go programiranju, stil kodiranja ne može biti proizvoljan. Na primer, mnoge početne vitičaste zagrade **{** ne mogu se staviti u sledeći red. Ako izmenimo **StatRandomNumbers** deklaraciju funkcije u gornjem programu na sledeći način, program neće uspeti da se kompajlira.

```go
func StatRandomNumbers(n int) (int, int)
{                                                   // syntax error
    var a, b int
    for i := 0; i < n; i++
    {                                               // syntax error
        if rand.Intn(MaxRnd) < MaxRnd/2
        {                                           // syntax error
            a = a + 1
        } else {
            b++
        }
    }
    return a, b
}
```

Nekim programerima se možda neće sviđati ograničenja preloma reda. Ali ograničenja imaju dve prednosti:

- Oni ubrzavaju kompilacije koda.
- Oni čine da stilovi kodiranja koje su napisali različiti goferi izgledaju slično, tako da je lakše čitanje i razumevanje koda koji su napisali drugi goferi.

Više o pravilima preloma reda možemo saznati u kasnijem članku. Trenutno bi trebalo da izbegavamo stavljanje početne vitičaste zagrade u novi red. Drugim rečima, generalno, prvi neprazan znak u liniji koda ne bi trebalo da bude početni znak vitičaste zagrade.

> [!Note]
> Ali, imajte na umu da ovo nije univerzalno pravilo.

[Go alati][0002] | [Sadržaj][00] | [Ključne reči i indentifikatori][0102]

[0002]: 0002_Go_alati.md
[00]: 00_Sadrzaj.md
[0102]: 0102_Ključne_reči_i_identifikatori.md
