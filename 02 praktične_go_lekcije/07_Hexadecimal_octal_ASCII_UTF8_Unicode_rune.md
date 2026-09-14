
# 07 Heksadecimalni, oktalni, ASCII, UTF8, Unicode, rune

[06 Binarni i decimalni][06] | [00 Sadržaj][00] | [08 Promenljive, konstante i osnovni tipovi][08]

**Šta ćete naučiti u ovom poglavlju?**

- Šta su Unicode, ASCII i UTF-8?
- Kako se čuvaju znakovi.
- Koji je tip rune?

**Obrađeni tehnički koncepti!**

- ASCII
- UTF8
- Heksadecimalni
- Oktalno
- Runa
- Kodna tačka

## Uvod

U prethodnom poglavlju smo obrađivali decimalnu i binarnu notaciju. Ovo poglavlje će govoriti o heksadecimalnoj i oktalnoj notaciji. Takođe ćemo govoriti o ASCII i UTF-8.

## Baza 16: heksadecimalni prikaz

Da biste predstavili binarni broj, potrebno je da poravnate mnogo nula i jedinica. Ova notacija je opširna. Da bismo predstavili decimalni broj 1324, morali smo da koristimo 11 znakova u binarnom sistemu. Zato nam je potreban sistem numeracije koji je pogodniji za izražavanje velikih brojeva.

Heksadecimalni je takođe pozicioni brojevni sistem koji koristi 16 znakova za predstavljanje broja.

- Prefiks Hex znači 6 na latinskom
- Decimalni broj potiče od latinske reči decem što znači 10

Ti znakovi su brojevi i slova. Koristimo brojeve od 0 do 9 (10 znakova) i slova od A do F (6 znakova).

Ekvivalencija između heksadecimalnih i decimalnih znakova

| Sistem | . | . | . | . | . | . | . | . | . | . | . | . | . | . | . | . |
| ----------- | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - | - |
| Hexadecimal | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | A | B | C | D | E | F |  
| Decimal | 0 | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 | 11 | 12 | 13 | 14 | 15 |

Uzmimo primer: 1324 u decimalnom sistemu je ekvivalentno 52C u hexadecimalnom sistemu.

Cifre od 0 do 9 odgovaraju istoj vrednosti u decimalnom sistemu. Slova A odgovaraju broju 10, slovo B broju 11... itd. Ovo je specifičnost heksadecimalnog brojnog sistema; koristimo slova za predstavljanje numeričkih vrednosti.

![0701][0701]
Heksadecimalna reprezentacija decimalnog broja

Možete videti da smo u ovoj notaciji uveli slova. To je zato što od 0 do 9 imate deset znakova, deset cifara, ali sa sistemom brojanja sa osnovom 16, potrebno nam je još šest znakova. Zato smo uzeli prvih šest slova abecede. Ovo je istorijski izbor; drugi znakovi su mogli zameniti slova, sistem bi i dalje bio isti.

Metod koji možete koristiti za pretvaranje heksadecimalnog broja u decimalni broj je sličan prethodnom. Uzimamo krajnji desni znak, pronalazimo njegov decimalni ekvivalent, a zatim ga množimo sa 16 na stepen 0. U našem primeru, imamo slovo C. Ekvivalent slova C je 12.

Da biste odštampali heksadecimalni prikaz broja, možete koristiti funkcije iz paketa `fmt` :

```go
// hexadecimal-octal-ascii-utf8-unicode-runes/hexa-lower/main.go
package main

import "fmt"

func main() {
    n := 2548
    fmt.Printf("%x", n)
}
```

Ovaj program će odštampati: 9f4 (što je heksadecimalni prikaz decimalnog broja 2548). `"%x"` je glagol formatiranja za heksadecimalni sistem (sa malim slovima).

Imajte na umu da je n broj označen decimalnim sistemom.

Takođe možete koristiti `"%X"` za štampanje heksadecimalnog broja velikim slovima:

```go
// hexadecimal-octal-ascii-utf8-unicode-runes/hexa-upper/main.go
package main

import "fmt"

func main() {
    n := 2548
    fmt.Printf("%X", n)
}
```

Izlaz:

```sh
9F4
```

Ako želite da predstavite broj u heksadecimalnom obliku u svom kodu, dodajte 0x ispred broja:

```go
// hexadecimal-octal-ascii-utf8-unicode-runes/hex-number/main.go
package main

import "fmt"

func main() {
    n := 2548
    n2 := 0x9F4
    fmt.Printf("%X\n", n)
    fmt.Printf("%x\n", n2)
}
```

Izlaz:

```sh
9F4
9f4
```

Da biste odštampali broj u desetičnom sistemu, možete koristiti glagol `"%d"`:

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/decimal/main.go
package main

import "fmt"

func main() {
    n2 := 0x9F4
    fmt.Printf("Decimal : %d\n", n2)
}
```

Izlaz:

```sh
Decimal : 2548
```

## Baza 8 - oktalni prikaz

Skoro sam zaboravio još jedan brojevni sistem! Oktalni!

Koristi bazu 8, što znači osam različitih znakova. Izabrani su brojevi od 0 do 7. Konverzija iz decimalnog u oktalni sistem je slična metodama koje sam ranije predstavio. Uzmimo primer:

![0702][0702]  
Oktalna reprezentacija

Počinjemo sa krajnjim desnim znakom i množimo ga sa osam na stepen 0, što je 1. Zatim uzimamo sledeći znak: 5 da bismo ga pomnožili sa osam na stepen 1, što je 8...

Oktalni sistem se posebno koristi za predstavljanje dozvola za datoteku na Juniks operativnim sistemima.

Na isti način kao i heksadecimalni sistem, fmt paket definiše dva glagola formatiranja za oktalni sistem:

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/octal/main.go
package main

import "fmt"

func main() {
    n2 := 0x9F4
    fmt.Printf("Decimal : %d\n", n2)

    // n3 is represented using the octal numeral system
    n3 := 02454

    // alternative : n3 := 0o2454

    // convert in decimal
    fmt.Printf("decimal: %d\n", n3)

    // n4 is represented using the decimal numeral system
    n4 := 1324

// output n4 (decimal) in octal
    fmt.Printf("octal: %o\n", n4)

// output n4 (decimal) in octal (with a 0o prefix)
    fmt.Printf("octal with prefix : %O\n", n4)
}
```

Izlaz:

```sh
Decimal : 2548
decimal: 1324
octal: 2454
octal with prefix : 0o2454
```

- `"%o"` omogućavaju vam da odštampate broj u oktalnom obliku
- `"%O"`comogućavaju vam da odštampate broj u oktalnom sistemu sa `"0o"` prefiksom

### Bitovi za predstavljanje podataka, nibl, byte  i words

Bit je skraćenica za binarnu cifru. Na primer 010100101100se sastoji od 11 binarnih cifara, drugim rečima, 11 bitova. Veoma je uobičajeno grupisati bitove zajedno. Grupe postoje u različitim veličinama:

- Nibl se sastoji od 4 bita
- Byte se sastoji od 8 bitova (dva nibla)
- Word se sastoji od 16 bita (dva bajta)
- Dvostruka reč se sastoji od 32 bita (dve reči)
- Četvorostruka reč se sastoji od 16×4 = 64 bitova (četiri reči)

Sa Go-om, možete kreirati deo bajtova. Mnoge uobičajene standardne funkcije i metode paketa uzimaju kao argumente deo bajtova. Da vidimo kako možemo da kreiramo deo bajtova.

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/slice-of-byte/main.go
package main

import "fmt"

func main() {
    b := make([]byte, 0)
    b = append(b, 255)
    b = append(b, 10)
    fmt.Println(b)
}
```

U prethodnom isečku, kreirali smo deo bajtova (pomoću ugrađene komande make), a zatim smo tom delu dodali dva broja.

Tip `byte` u Golang je pseudonim za `uint8`. `Uint8` znači da možemo da čuvamo neoznačene (bez ikakvih znakova, dakle bez negativnih brojeva) cele brojeve na 8 bitova (bajt) podataka. Minimalna vrednost je 0 (binarna cifra 00000000​b) maksimalna vrednost je 255 (11111111b) ​što je ekvivalentno decimalnom broju 2^7 + 2^6 + 2^5 + 2^4 + 2^3 + 2^2 + 2^1 + 2^0 = 255d.

Zato byte isečku možemo dodavati samo brojeve od 0 do 255. Ako pokušate da dodate broj veći od 255, dobićete sledeću grešku:

```sh
dataRepresentation/bytes/main.go:7:15: constant 256 overflows byte
```

Da biste odštampali binarni prikaz broja, možete koristiti `"%b"` glagol formatiranja:

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/decimal-binary/main.go
package main

import "fmt"

func main() {
    n2 := 0x9F4
    fmt.Printf("Decimal : %d\n", n2)
    fmt.Printf("Binary : %b\n", n2)
}
```

Izlaz:

```sh
Decimal : 2548
Binary : 100111110100
```

**Šta je sa ostalim znacima**?

Šta ako želite da sačuvate nešto drugo osim brojeva? Na primer, kako bismo mogli da sačuvamo ovaj haiku od Masaokija Šikija:

```sh
spring rain:
browsing under an umbrella
at the picture-book store
```

Da li je tip bajta odgovarajući? Bajt nije ništa više od neoznačenog celog broja sačuvanog na 8 bitova. Ovaj haiku je sastavljen od slova i specijalnih znakova. Imamo ":" i "-", takođe imamo prelome reda. Kako možemo da sačuvamo te znakove?

Moramo pronaći način da svakom slovu, pa čak i specijalnim karakterima, damo jedinstveni kod. Možda ste čuli za UTF-8, ASCII, Unicode? Ovaj odeljak će objasniti šta su oni i kako funkcionišu. Kada sam počeo da programiram (to nije bilo u programu Go), kodiranje znakova je bilo nešto nepoznato i nisam ga smatrao zanimljivim. Mislim da bi kodiranje znakova moglo biti neophodno jer sam provodio noći na poslu rešavajući probleme koji su se mogli rešiti osnovnim razumevanjem kodiranja znakova.

Istorija kodiranja znakova je veoma bogata. Razvojem telegrafa, bio nam je potreban način kodiranja poruka na način koji bi se mogao prenositi električnom žicom. Jedan od najranijih pokušaja bio je Morzeov kod. Sastoji se od četiri simbola: kratak signal, dugi signal, kratak razmak, dugi razmak (Wikipedia). Svako slovo azbuke moglo se kodirati Morzeovim kodom. Na primer, slovo A je kodirano kao kratak signal praćen dugim signalom. Znak plus "+" je kodiran sa "kratko dugo kratko dugo kratko".

## Skupovi znakova i kodiranje

### Rečnik

Moramo definisati zajednički rečnik da bismo razumeli kodiranje znakova:

- **Znak**: - Znak može biti napisan našom rukom. Prenosi značenje. Na primer, znak "+" je znak sabiranja. To znači dodavanje nečega nečemu drugom. Znak može biti slovo, znak ili ideogram.
- **Skup znakova**: Skup različitih znakova. Često ćete videti ili čuti skraćenicu "charset".
- **Kodna tačka**: svaki znak iz skupa znakova ima ekvivalentnu numeričku vrednost koja jedinstveno identifikuje taj znak. Ova numerička vrednost je kodna tačka.

### Unicode

Postoji jedan skup znakova koji želite da znate: `Unicode`. To je standard koji navodi veliku većinu znakova iz živih jezika koji se danas koriste na računarima.

Sastoji se od 137.374 karaktera za svoju verziju 11.0. Unicode je kao ogromna tabela koja mapira karakter na kodnu tačku. Na primer, karakter "A" je mapiran na kodnu tačku "0041".

Sa Unikodom imamo osnovu, našu tabelu znakova, a sada je sledeći izazov pronaći način da se ti znaci kodiraju, da se te kodne tačke stave u bajtove podataka. To je upravo ono što ASCII i UTF-8 rade.

![0703][0703]  
Unicode code point

![0704][0704]  
Unicode code point za neke znakove

### Kako funkcioniše ASCII?

**Napomena**:  
ASCII označava Američki standardni kod za razmenu informacija. Razvijen je tokom šezdesetih godina. Cilj je bio pronaći način za kodiranje znakova koji se koriste za prenos poruka.

ASCII kodira znakove na sedam binarnih cifara. Još jedna binarna cifra je bit parnosti. Bit parnosti se koristi za otkrivanje grešaka u prenosu. Dodaje se posle prvih sedam bitova i njegova vrednost je 0. Ako je broj jedinica neparan, onda je bit parnosti 1; ako je broj paran, postavlja se na 0.

Bajt podataka može da sačuva svaki karakter (8 bita). Koliko celih brojeva možete kreirati sa samo 7 bitova ? Sa jednim jedinim bitom možemo kodirati dve vrednosti, 0 i 1, sa 2 bita možemo kodirati četiri različite vrednosti. Kada dodate bit, množite sa dva broj vrednosti koje možete kodirati. Sa 7 bitova možete kodirati 128 celih brojeva. Generalno, broj neoznačenih celih brojeva koje možete kodirati sa n binarnih cifara je dva na stepen n.

Broj binarnih cifara i moguće kodirane vrednosti

| Broj bitova | Broj vrednosti |
| :---------: | :------------: |
| 1 | 2 |
| 2 | 4 |
| 3 | 8 |
| 4 | 16 |
| 5 | 32 |
| 6 | 64 |
| 7 | 128 |

ASCII vam omogućava da kodirate 128 različitih znakova. Za svaki znak imamo specifičnu kodnu tačku. Neoznačene celobrojne vrednosti predstavljaju kodne tačke.

![0705][0705]  
USASCII-code-chart

Na prethodnoj slici možete videti USASCII kodnu tabelu. Ova tabela vam omogućava da konvertujete bajt u karakter. Na primer, slovo B je ekvivalentno broju 1000010 (binarno) (kolona 4, red 2)

### Kako funkcioniše UTF-8

**Napomena**:  
UTF-8 znači Univerzalni format transformacije skupa znakova 1 - 8 bita. Izumela su ga dvojica ljudi koji su takođe tvorci jezika Go: Rob Pajk i Ken Tompson! Dizajn ove vrste kodiranja je veoma genijalan. Pokušaću da ga ukratko objasnim:

UTF-8 je sistem kodiranja promenljive širine. To znači da se znakovi kodiraju pomoću jednog do četiri bajta (bajt predstavlja osam binarnih cifara).

![0706][0706]  
UTF-8 sistem kodiranja promenljive širine

Na gornjoj slici možete videti pravila kodiranja UTF-8. Znak može biti kodiran od 1 do 4 bajta.

Kodne tačke koje se mogu kodirati korišćenjem samo jednog bajta su od U+0000 do U+007F (uključujući i njih). Ovaj opseg se sastoji od 128 znakova. (od 0 do 127, postoji 128 brojeva)

Ali potrebno je kodirati više znakova! Zato su tvorci UTF-8 došli na ideju da dodaju bajtove sistemu. Prvi dodatni bajt počinje sa jedinicom i nulom; oni su fiksirani. To signalizira dekoderima da sada koristimo 2 bajta za kodiranje naših znakova, jednostavno dodajemo bitove " 110 ". To govori UTF-8 dekoderima: "Budite oprezni; nas je 2!".

Ako koristimo 2 bajta, imamo 11 slobodnih bitova (8 * 2 - 5 (fiksni bitovi) = 11). Možemo kodirati znakove koji sadrže Unikod kodnu tačku od U+0080 do U+07FF. Koliko znakova to predstavlja?

- 0080 u heksadecimalnom sistemu = 128 u decimalnom sistemu
- 07FF u heksadecimalnom sistemu = 2047 u decimalnom sistemu. od 0080 do 07FF ima 2047-128+1=1920
  znakova

Možda se pitate zašto dodajemo jedinicu broju... To je zato što se znakovi indeksiraju od kodne tačke 0.

Ako koristite 3 bajta, onda će prvi bajt početi sa fiksnim bitovima 1110. Ovo će signalizirati dekoderima da je znak kodiran pomoću 3 bajta. Drugim rečima, sledeći znakovi će početi posle trećeg bajta. Dva dodatna bajta počinju sa 10. Sa tri bajta za kodiranje, imate 16 slobodnih bitova (8 * 3 - 8 (fiksni bitovi) = 16). Možete kodirati znakove od U+0800 do U+FFFF.

Ako ste razumeli kako funkcioniše za 3 bajta, onda ne bi trebalo da imate problema da shvatite kako sistem funkcioniše sa 4 bajta. Unutar našeg prvog bajta, fiksiramo prvih pet bitova ( 11110 ). Zatim imamo tri dodatna bajta. Ako oduzmemo fiksne bitove od ukupnog broja bitova, imamo 21 dostupan bit. To znači da možemo da kodiramo kodne tačke od U+10000 do U+10FFFF.

### String

String je "niz znakova". Na primer, "Test" je string sastavljen od 4 različita znaka: T, e, s i t. Stringovi su rasprostranjeni; koristimo ih za čuvanje sirovog teksta unutar našeg programa. LJudi ih generalno mogu čitati. Na primer, ime i prezime korisnika aplikacije su dva stringa.

Znakovi mogu biti iz različitih skupova znakova. Ako koristite skup znakova ASCII, morate da birate između 128 dostupnih znakova.

Svaki znak ima odgovarajuću kodnu tačku u skupu znakova. Kao što smo ranije videli, kodna tačka je neoznačeni ceo broj koji se proizvoljno izabere. Nizovi se čuvaju pomoću bajtova. Uzmimo primer stringa sastavljenog samo od ASCII znakova:

```go
"Hello"
```

Jedan bajt može da sačuva svaki znak. Ovaj string se može sačuvati sa sledećim bitovima:

![0707][0707]  
Binarni ekvivalent stringa "Hello" (ASCII)

U Go jeziku, stringovi su nepromenljivi, što znači da se ne mogu menjati kada se jednom kreiraju.

### String literala

Postoje dve "vrste" literala stringova:

- `Raw` (sirovi) string literali. Definisani su između \`povratnih\` navodnika.
  - Zabranjeni znakovi su
    - povratni navodnici
  - Odbačeni znakovi su
    - povratak reda unazad (\r)

- Interpretirani string literali. Definisani su između dvostrukih navodnika.
  - Zabranjeni likovi su
    - nove linije
    - neodbačeni dvostruki navodnici

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/string-literals/main.go
package main

import "fmt"

func main() {

    raw := `spring rain:
browsing under an umbrella
at the picture-book store`
    fmt.Println(raw)

    interpreted := "i love spring"
    fmt.Println(interpreted)
}
```

Možete primetiti da unutar ovog isečka koda nismo rekli Go-u koji skup znakova koristimo. To je zato što su string literali implicitno kodirani pomoću UTF-8.

### Runa

Iza kulisa, string je kolekcija bajtova. Možemo iterirati kroz bajtove stringa pomoću for petlje:

![0708a][0708a]  
Volim Golang

Izlaz:

![0708][0708]  
Volim Golang

![0709][0709]  
Volim Golang

Poruka na prethodnoj slici znači "Volim Golang", prva dva karaktera su kineska.

Ovaj program će iterirati kroz svaki karakter stringa. Unutar for petlje, v je tipa rune. rune je ugrađeni tip koji je definisan na sledeći način:

```go
// rune is an alias for int32 and is equivalent to int32 in all ways. It is
// used, by convention, to distinguish character values from integer values.
type rune = int32
```

Rune predstavlja Unicode kodnu tačku.

- Unicode kodne tačke su numeričke vrednosti.

- Po konvenciji, uvek se označavaju u sledećem formatu: "U+X"gde je X heksadecimalni prikaz kodne tačke. X treba da ima četiri karaktera.

- Ako X ima manje od četiri znaka, dodajemo nule.  
  
  Na primer:
  - Znak "o" ima kodnu tačku jednaku 111 (u decimalnom sistemu).  
  - 111 u heksadecimalnom sistemu se piše kao 6F.  
  - Hexadecimalna kodna tačka je U+006F.

Da biste odštampali kodnu tačku u konvencionalnom formatu, možete koristiti glagol formatiranja "%U".

![0710][0710]  
Dekompozicija niza kodnih tačaka Unicode-a

Imajte na umu da možete kreirati runu koristeći jednostruke navodnike:

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/rune/main.go
package main

import "fmt"

func main(){
    var aRune rune = 'Z'
    fmt.Printf("Unicode Code point of &#39;%c&#39;: %U\n", aRune, aRune)
}
```

## Testirajte sebe

### Pitanja i odgovori

1. Tačno ili netačno: "785G" je heksadecimalni broj
   - Netačno. Slovo G ne može biti deo heksadecimalnih brojeva. Međutim,
     slova od A do F mogu biti deo heksadecimalnog broja.
2. Tačno ili netačno: "785f" i "785F" predstavljaju istu količinu
   - Ovo je tačno. Činjenica da je slovo napisano velikim slovom ne
     menja njegovo značenje.
3. Koji je glagol formatiranja za predstavljanje heksadecimalnog broja
   (velikim slovom)?
   - %X
4. Koji je glagol formatiranja za predstavljanje broja u decimalnom
   sistemu?
   - %d
5. Šta je kodna tačka?
   - Kodna tačka je numerička vrednost koja identifikuje znak u skupu
     znakova.
6. Popunite praznine. _______ je skup znakova, ______ je standard
   kodiranja.
   - Unicode je skup znakova, UTF-8 je standard kodiranja.
7. Tačno ili netačno: UTF-8 vam omogućava kodiranje manjeg broja znakova
   nego ASCII.
   - Netačno
8. Koliko bajtova mogu da koristim za kodiranje znaka koristeći UTF-8
   sistem kodiranja?
   - Od 1 do 4 bajta. Zavisi od znaka.

### Ključno

- Heksadecimalni je brojni sistem poput decimalnog i binarnog
- Kod heksadecimalnog sistema, broj se predstavlja pomoću 16 znakova:
  0, 1, 2, 3, 4, 5, 6, 7, 8, 9, A, B, C, D, E, F
- Sa fmt funkcijama ( `fmt.Sprintf` and `fmt.Printf` ) možete koristiti "glagole formatiranja" da biste predstavili broj koristeći određeni brojni sistem.
  - %b za binarni
  - %X i %x za heksadecimalni
  - %d za decimalni
  - %o za oktalni
- **Karakter (znak)**. To je nešto što možemo napisati rukom, a što prenosi značenje.  
  Npr.: "-", "A", "a".
- **Skup znakova**: ovo je skup različitih znakova. Često ćete videti ili čuti skraćenicu "charset".
- **Kodna tačka**: svaki znak iz skupa znakova kao ekvivalentna numerička vrednost koja jedinstveno identifikuje taj znak. Ova numerička vrednost je kodna tačka.
- Unicode je skup znakova koji se sastoji od preko 137.000 znakova.
- Svaki znak ima kodnu tačku. Na primer, "A" znak je ekvivalentan kodnoj tački U+0041.
- ASCII je tehnika kodiranja koja može kodirati samo 128 znakova.
- **UTF-8** je tehnika kodiranja koja može kodirati više od milion znakova
- Sa UTF-8, bilo koji znak se kodira koristeći od 1 do 4 bajta.
- **rune** je ugrađeni tip
- Rune predstavlja Unicode kodnu tačku karaktera (znaka).
- Da biste napravili runu, možete koristiti jednostruke navodnike:

  ```go
  var aRune rune = 'Z'
  ```

- Kada iterirate preko stringa, iterirate preko runa.

```go
// /hexadecimal-octal-ascii-utf8-unicode-runes/iterate-over-string/main.go
package main

import "fmt"

func main() {
    b := "hello"
    for i := 0; i < len(b); i++ {
        fmt.Println(b[i])
    }
    // will output :
    // 104
    // 101
    // 108
    // 108
    // 111
    // and NOT :
    // h
    // e
    // l
    // l
    // o
}
```

- U Go programu, stringovi su nepromenljivi, što znači da se ne mogu menjati kada se jednom kreiraju.

[06 Binarni i decimalni][06] | [00 Sadržaj][00] | [08 Promenljive, konstante i osnovni tipovi][08]

[0701]:./_images/0701.png
[0702]: ./_images/0702.png
[0703]: ./_images/0703.png
[0704]: ./_images/0704.png
[0705]: ./_images/0705.png
[0706]: ./_images/0706.png
[0707]: ./_images/0707.png
[0708a]: ./_images/0708a.png
[0708]: ./_images/0708.png
[0709]: ./_images/0709.png
[0710]: ./_images/0710.png
[06]: 06_Binarni_i_decimalni.md
[00]: 00_Sadržaj.md
[08]: 08_Promenljive_konstante_i_osnovni_tipovi.md
