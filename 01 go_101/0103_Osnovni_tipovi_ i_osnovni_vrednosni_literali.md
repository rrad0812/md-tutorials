
# Osnovni tipovi i osnovni vrednosni literali

[Ključne reči i identfikatori][0102] | [Sadržaj][00] | [Konstante i promenljive][0104]

Tipovi se mogu posmatrati kao šabloni vrednosti, a vrednosti se mogu posmatrati kao instance tipova. Ovaj članak će predstaviti ugrađene osnovne tipove i njihove literale vrednosti u jeziku Go. Kompozitni tipovi neće biti predstavljeni u ovom članku.

## Ugrađeni osnovni tipovi u Go-u

Go podržava sledeće ugrađene osnovne tipove:

- jedan ugrađeni bulov tip: **bool**.

- 11 ugrađenih celobrojnih numeričkih tipova (osnovni **celobrojni** tipovi):

  - **int8**,
  - **uint8**,
  - **int16**,
  - **uint16**,
  - **int32**,
  - **uint32**,
  - **int64**,
  - **uint64**,
  - **int**,
  - **uint** i
  - **uintptr**.

- dva ugrađena numerička tipa sa pokretnim zarezom:

  - **float32** i
  - **float64**.

- dva ugrađena kompleksna numerička tipa:
  - **complex64** i
  - **complex128**.

- jedan ugrađeni tip stringa:

  - **string**.

Svaki od 17 ugrađenih osnovnih tipova pripada jednoj različitoj vrsti tipa u Go-u. Možemo koristiti gore navedene ugrađene tipove u kodu bez uvoza bilo kakvih paketa, iako su sva imena ovih tipova neeksportovani identifikatori.

15 od 17 ugrađenih osnovnih tipova su numerički tipovi. **Numerički tipovi** uključuju **celobrojne tipove**, **tipove sa pokretnim zarezom** i **kompleksne tipove**.

Go takođe podržava dva ugrađena alijasa tipova,

- **byte** je ugrađeni alias **uint8**. Možemo da posmatramo **byte** i **uint8** kao isti tip.
- **rune** je ugrađeni alias **int32**. Možemo da posmatramo **rune** i **int32** kao isti tip.

Celobrojni tipovi čija imena počinju sa **u** su **neoznačeni** tipovi. Vrednosti neoznačenih tipova su uvek nenegativne. Broj u imenu tipa znači koliko binarnih bitova će vrednost tipa zauzeti u memoriji tokom izvršavanja. Na primer, svaka vrednost **uint8** zauzima 8 bitova u memoriji. Dakle, najveća **uint8** vrednost je 255 (2^8-1), najveća **int8** vrednost je 127 (2^7-1), a najmanja **int8** vrednost je -128 (-2^7).

Ako vrednost zauzima N bitova u memoriji, kažemo da je veličina vrednosti N bitova. Veličine svih vrednosti jednog tipa su uvek iste, pa se veličine vrednosti često nazivaju veličinama tipa.

Često merimo veličinu vrednosti na osnovu broja bajtova koje zauzima u memoriji. Jedan **byte** sadrži 8 bitova. Dakle, veličina tipa **uint32** je četiri bajta.

Veličina vrednosti **uintptr**, **int** i **uint** u memoriji je specifična za implementaciju. Generalno, veličina vrednosti **int** i **uint** je 4 bajta na 32-bitnim arhitekturama i 8 bajta na 64-bitnim arhitekturama. Veličina **uintptr** vrednosti mora biti dovoljno velika da sačuva neinterpretirane bitove bilo koje memorijske adrese.

Realni i imaginarni deo vrednosti **complex64** su oba **float32** vrednosti, a realni i imaginarni deo vrednosti **complex128** su oba **float64** vrednosti.

U memoriji, sve numeričke vrednosti sa pokretnim zarezom u programskom jeziku Go se čuvaju u IEEE-754 formatu.

Bulova vrednost predstavlja istinu. Postoje samo dve moguće bulove vrednosti u memoriji, one su označene sa dve unapred deklarisane imenovane konstante **false** i **true**.

Imenovane konstante biće predstavljene u sledećem članku.

U logici, **string** vrednost označava deo teksta. U memoriji, **string** vrednost čuva niz bajtova, što je UTF-8 kodiranje dela teksta označenog string vrednošću. Više činjenica o stringovima možemo saznati iz članka o stringovima u Gou kasnije.

Iako postoji samo jedan ugrađeni tip za svaki od bulovih i string tipova, možemo definisati prilagođene bulove i string tipove za ugrađene bulove i string tipove. Dakle, može postojati mnogo bulovih i string tipova. Isto važi i za sve vrste numeričkih tipova. Slede neki primeri deklaracija tipova. U ovim deklaracijama, reč **type** je ključna reč.

```go
/* Some type definition declarations */

// status and bool are two different types.
type status bool

// MyString and string are two different types.
type MyString string

// Id and uint64 are two different types.
type Id uint64

// real and float32 are two different types.
type real float32

/* Some type alias declarations */

// boolean and bool denote the same type.
type boolean = bool
// Text and string denote the same type.
type Text = string
// U8, uint8 and byte denote the same type.
type U8 = uint8
// char, rune and int32 denote the same type.
type char = rune
```

Prilagođeni tip *real* definisan gore i ugrađeni tip **float32** možemo nazvati **float32** tipovima. Obratite pažnju da je druga reč **float32** u poslednjoj rečenici opšta referenca, dok je prva određena referenca. Slično tome, *MyString* i **string** su oba string tipovi, **status** i **bool** su oba logički tipovi, itd.

Više o prilagođenim tipovima možemo saznati kasnije u članku Pregled sistema tipova Go.

### Nulte vrednosti

Svaki tip ima **nultu** vrednost. Nulta vrednost tipa može se posmatrati kao podrazumevana vrednost tog tipa.

- Nulta vrednost **bulovog** tipa je **false**.
- Nulta vrednost **numeričkog** tipa je **nula**, iako nule različitih numeričkih tipova mogu imati različite veličine u memoriji.
- Nulta vrednost tipa **string** je **prazan string**.

## Osnovni vrednosni literali

Literal vrednosti je tekstualna reprezentacija vrednosti u kodu. Vrednost može imati mnogo literala. Literali koji označavaju vrednosti osnovnih tipova nazivaju se literali osnovne vrednosti.

### Bulove vrednosti literala

Go specifikacija ne definiše bulove literale. Međutim, u opštem programiranju, možemo posmatrati dva unapred deklarisana identifikatora, **false** i **true**, kao bulove literale. Ali trebalo bi da znamo da ta dva nisu literali u strogom smislu.

Kao što je gore pomenuto, nulte vrednosti bulovih tipova su označene unapred deklarisanom **false** konstantom.

### Literali celobrojnih vrednosti

Postoje četiri oblika celobrojnih literala:

- **decimalni (baza 10)**,
- **oktalni (baza 8)**,
- **heksadecimalni (baza 16)** i
- **binarni (baza 2)**.

Na primer, sledeća četiri cela literala označavaju 15 u decimalnom sistemu.

```go
0xF                     // the hex form (starts with a "0x" or "0X")
0XF

017                     // the octal form (starts with a "0", "0o" or "0O")
0o17
0O17

0b1111                  // the binary form (starts with a "0b" or "0B")
0B1111

15                      // the decimal form (starts without a "0")
```

> [!Note]
> Binarni i oktalni oblik koji počinju sa **0o** ili **0O** su podržani od
> verzije Go 1.13.

Sledeći program će ispisati dva **true** teksta.

```go
package main

func main() {
    println(15 == 017) // true
    println(15 == 0xF) // true
}
```

Imajte na umu da su ova dva **==** operatora poređenja jednakosti, koji će biti predstavljen u okviru uobičajenih operatora.

Generalno, nulte vrednosti celobrojnih tipova se označavaju kao **0** literal, iako postoje mnogi drugi dozvoljeni literali za celobrojne nulte vrednosti, kao što su **00** i **0x0**. U stvari, literali nulte vrednosti predstavljeni u ovom članku za druge vrste numeričkih tipova takođe mogu predstavljati nultu vrednost bilo kog celobrojnog tipa.

### Literali vrednosti sa pokretnim zarezom

Decimalni literal vrednosti sa pokretnim zarezom može da sadrži

- decimalni ceo broj,
- decimalni zarez,
- decimalni razlomljeni deo i
- deo celobrojnog eksponenta ( na bazi 10 ).  
  Takav deo celobrojnog eksponenta počinje slovom e ili E, a sufiksi se završavaju decimalnim ceobrojnim literalom.
  - **xEn** je ekvivalentno x pomnoženo sa 10^n, a
  - **xE-n** je ekvivalentno x podeljeno sa 10^n.
  
Neki primeri:

```go
1.23
01.23   // == 1.23
.23
1.      // == 1.0

// An "e" or "E" starts the exponent part (10-based).
1.23e2  // == 123.0
123E2   // == 12300.0
123.E+2 // == 12300.0
1e-1    // == 0.1
.1e0    // == 0.1
0010e-2 // == 0.1
0e+5    // == 0.0
```

Od verzije Go 1.13, Go takođe podržava i drugi oblik literala sa pokretnim zarezom: heksadecimalni oblik literala sa pokretnim zarezom.

- Heksadecimalni literal sa pokretnim zarezom mora se završavati eksponentskim delom baziranim na 2, koji počinje slovom p ili P, a sufiksi se završavaju decimalnim celobrojnim literalom:
  
  - **yPn** je ekvivalentno je **y** pomnoženo sa **2^n**,
  - **yP-n** je ekvivalentno **y** podeljeno sa **2^n**.

- Isto kao i heksadecimalni celobrojni literali, heksadecimalni literal sa pokretnim zarezom takođe mora da počinje sa **0x** ili **0X**.

- Za razliku od heksadecimalnih celobrojnih literala, heksadecimalni literal sa pokretnim zarezom može da sadrži radix tačku i heksadecimalni razlomljeni deo.

Slede neki dozvoljeni heksadecimalni literali sa pokretnim zarezom:

```go
0x1p-2     // == 1.0/4 = 0.25
0x2.p10    // == 2.0 * 1024 == 2048.0
0x1.Fp+0   // == 1+15.0/16 == 1.9375
0X.8p1     // == 8.0/16 * 2 == 1.0
0X1FFFP-16 // == 0.1249847412109375
```

Međutim, sledeći su nedozvoljeni:

```go
0x.p1    // mantissa has no digits
1p-2     // p exponent requires hexadecimal mantissa
0x1.5e-2 // hexadecimal mantissa requires p exponent
```

> [!Note]
> Sledeći literal je validan, ali nije literal sa pokretnim zarezom. U stvari,
> to je aritmetički izraz za oduzimanje. "ein" znači 14 u decimalnom formatu.
> 0x15e je heksadecimalni celobrojni literal, - je operator oduzimanja, a 2 je
> decimalni celobrojni literal. (Aritmetički operatori biće predstavljeni u
> članku Uobičajeni operatori.)

```go
0x15e-2 // == 0x15e - 2 // a subtraction expression
```

Standardni literali za tipove sa pokretnim zarezom koji čine nultu vrednost su **0.0**, mada postoji mnogo drugih legalnih literala, kao što su **0.,.0**, **0e0**, **0x0p0**, itd. U stvari, literali sa nultom vrednošću predstavljeni u ovom članku za druge vrste numeričkih tipova takođe mogu predstavljati nultu vrednost bilo kog tipa sa pokretnim zarezom.

## Literali imaginarne vrednosti

Imaginarni literal se sastoji od literala sa pokretnim zarezom ili celog broja i malog slova i. Primeri:

```go
1.23i
1.i
.23i
123i
0123i   // == 123i (for backward-compatibility. See below.)
1.23E2i // == 123i
1e-1i
011i   // == 11i (for backward-compatibility. See below.)
00011i // == 11i (for backward-compatibility. See below.)
// The following lines only compile okay since Go 1.13.
0o11i    // == 9i
0x11i    // == 17i
0b11i    // == 3i
0X.8p-0i // == 0.5i
```

Imajte na umu da pre verzije Go 1.13, u imaginarnom literalu, slovo imože imati prefiks samo literala sa pokretnim zarezom. Da bi bio kompatibilan sa starijim verzijama, od Go 1.13, celobrojni literali koji se pojavljuju kao oktalni celobrojni oblici koji ne počinju sa **0o** i **0O** se i dalje posmatraju kao literali sa pokretnim zarezom, kao što su **011i**, **0123i**  i **00011i** u gornjem primeru.

Imaginarni literali se koriste za predstavljanje imaginarnih delova kompleksnih vrednosti. Evo nekih literala za označavanje kompleksnih vrednosti:

```go
1 + 2i       // == 1.0 + 2.0i
1. -.1i     // == 1.0 + -0.1i
1.23i - 7.89 // == -7.89 + 1.23i
1.23i        // == 0.0 + 1.23i
```

Standardni literali za nulte vrednosti kompleksnih tipova su **0.0+0.0i**, mada postoje mnogi drugi legalni literali, kao što su **0i,.0i**, **0+0i**, itd. U stvari, literali nulte vrednosti predstavljeni u ovom članku za druge vrste numeričkih tipova takođe mogu predstavljati nultu vrednost bilo kog kompleksnog tipa.

Koristite **_** u numeričkim literalima za bolju čitljivost

Od verzije Go 1.13, donje crte **_** se mogu pojaviti u celim, pokretnim zarezima i imaginarnim literalima kao separatori cifara radi poboljšanja čitljivosti koda. Ali imajte na umu da u numeričkom literalu,

- Nije dozvoljeno da se **_** koristi kao prvi ili poslednji znak literala,
- Dve strane bilo kog izraza moraju biti ili doslovni prefiksi (kao što je 0X) ili dozvoljeni cifreni znakovi.

Neki dozvoljeni i nedozvoljeni numerički literali koji sadrže donje crte:

```go
// Legal ones:
6_9          // == 69
0_33_77_22   // == 0337722
0x_Bad_Face  // == 0xBadFace
0X_1F_FFP-16 // == 0X1FFFP-16
0b1011_0111 + 0xA_B.Fp2i

// Illegal ones:
_69        // _ can't appear as the first character
69_        // _ can't appear as the last character
6__9       // one side of _ is a illegal character
0_xBadFace // "x" is not a legal octal digit
1_.5       // "." is not a legal octal digit
1._5       // "." is not a legal octal digit
```

### Literali vrednosti runa

Tipovi **runa**, uključujući prilagođeno definisane tipove **runa** i ugrađeni **rune** tip (poznat i kao **int32** tip), su posebni celobrojni tipovi, tako da se sve vrednosti runa mogu označiti celobrojnim literalima predstavljenim gore. S druge strane, mnoge vrednosti svih vrsta celobrojnih tipova takođe mogu biti predstavljene runskim literalima predstavljenim u nastavku u ovom pododeljku.

Vrednost rune je namenjena za čuvanje Unikod kodne tačke. Generalno, kodnu tačku možemo posmatrati kao Unikod znak, ali treba znati da su neki Unikod znakovi sastavljeni od više od jedne kodne tačke.

Runski literal se izražava kao jedan ili više znakova zatvorenih pod navodnicima. Zatvoreni znakovi označavaju jednu vrednost Unicode kodne tačke. Postoje neke manje varijante oblika runskog literala. Najpopularniji oblik runskog literala je jednostavno stavljanje znakova označenih runskim vrednostima između dva jednostruka navodnika. Na primer

```go
'a' // an English character
'π'
'众' // a Chinese character
```

Sledeće varijante runskih literala su ekvivalentne 'a'(vrednost karaktera 'a' u Unikodu je 97).

```go
// 141 is the octal representation of decimal number 97.
'\141'
// 61 is the hex representation of decimal number 97.
'\x61'
'\u0061'
'\U00000061'
```

Imajte na umu da `\` moraju biti praćene tačno tri oktalne cifre da bi se predstavila vrednost bajta, `\x` moraju biti praćene tačno dve heksadecimalne cifre da bi se predstavila vrednost bajta, `\u` moraju biti praćene tačno četiri heksadecimalne cifre da bi se predstavila vrednost rune i `\U` moraju biti praćene tačno osam heksadecimalnih cifara da bi se predstavila vrednost rune. Svaki takav oktalni ili heksadecimalni niz cifara mora predstavljati validnu Unicode kodnu tačku, u suprotnom neće biti kompajliran.

Sledeći program će ispisati 7 *true* tekstova.

```go
package main

func main() {
    println('a' == 97)
    println('a' == '\141')
    println('a' == '\x61')
    println('a' == '\u0061')
    println('a' == '\U00000061')
    println(0x61 == '\x61')
    println('\u4f17' == '众')
}
```

U stvari, četiri varijante oblika runskih literala koje su upravo pomenute retko se koriste za vrednosti runa u praksi. Povremeno se koriste u interpretiranim string literalima (videti sledeći pododeljak za detalje).

Ako je runski literal sastavljen od dva znaka (ne računajući dva navodnika), prvi je znak \, a drugi nije digitalni znak, x, u i U, onda će dva uzastopna znaka biti izbegnuta kao jedan specijalni znak. Mogući parovi znakova koji se izbegavaju su:

```go
\a   (Unicode value 0x07) alert or bell
\b   (Unicode value 0x08) backspace
\f   (Unicode value 0x0C) form feed
\n   (Unicode value 0x0A) line feed or newline
\r   (Unicode value 0x0D) carriage return
\t   (Unicode value 0x09) horizontal tab
\v   (Unicode value 0x0b) vertical tab
\\   (Unicode value 0x5c) backslash
\'   (Unicode value 0x27) single quote

\n je najčešće korišćeni par karaktera za izlaz.
```

Primer:

```go
    println('\n') // 10
    println('\r') // 13
    println('\'') // 39

    println('\n' == 10)     // true
    println('\n' == '\x0A') // true
```

Postoji mnogo literala koji mogu da označe nulte vrednosti runskih tipova, kao što su '\000', '\x00', '\u0000', itd. U stvari, možemo koristiti i bilo koji numerički literal predstavljen gore da predstavimo vrednosti runskih tipova, kao što su 0, 0x0, 0.0, 0e0, 0i, itd.

### Literali string vrednosti

String vrednosti u programu Go su kodirane sa UTF-8. U stvari, sve Go izvorne datoteke moraju biti kompatibilne sa UTF-8 kodiranjem.

Postoje dva oblika string literala, interpretirani string literal (u obliku dvostrukih navodnika) i sirovi string literal (u obliku obrnutih navodnika). Na primer, sledeća dva string literala su ekvivalentna:

```go
// The interpreted form.
"Hello\nworld!\n\"你好世界\""

// The raw form.
`Hello
world!
"你好世界"`
```

U gore navedenom interpretiranom string literalu, svaki \n par znakova će biti izbegnut kao jedan karakter za novi red, a svaki \" par znakova će biti izbegnut kao jedan dvostruki navodnik. Većina takvih parova znakova za izbegavanje su isti kao i parovi znakova za izbegavanje koji se koriste u runskim literalima predstavljenim gore, osim što \" je legalno samo u interpretiranim string literalima i \` je legalno samo u runskim literalima.

Niz karaktera \ i \x praćen nekoliko oktalnih ili heksadecimalnih cifara predstavljenih u poslednjem odeljku takođe se može koristiti u interpretiranim string literalima \u, \U.

```go
// The following interpreted string literals are equivalent.
"\141\142\143"
"\x61\x62\x63"
"\x61b\x63"
"abc"

// The following interpreted string literals are equivalent.
"\u4f17\xe4\xba\xba"

// The Unicode of 众 is 4f17, which is
// UTF-8 encoded as three bytes: e4 bc 97.
"\xe4\xbc\x97\u4eba"

// The Unicode of 人 is 4eba, which is
// UTF-8 encoded as three bytes: e4 ba ba.
"\xe4\xbc\x97\xe4\xba\xba"
"众人"
```

Imajte na umu da je svaki engleski znak (kodna tačka) predstavljen jednim bajtom, dok je svaki kineski znak (kodna tačka) predstavljen sa tri bajta.

U sirovom literalu stringa, nijedan niz karaktera neće biti izbegnut. Znak povratnog navodnika nije dozvoljen u sirovom literalu stringa. Radi bolje kompatibilnosti između platformi, karakteri za povratak reda (Unicode kodna tačka 0x0D) unutar sirovih literala stringa biće odbačeni.

Nulte vrednosti string tipova mogu se označiti kao "" ili `` u bukvalnom obliku.

### Reprezentabilnost osnovnih literala numeričkih vrednosti

Numerički literal može se koristiti za predstavljanje celobrojne vrednosti samo ako ne mora biti zaokružen. Na primer, 1.23e2 može se predstaviti kao vrednosti bilo kog osnovnog celobrojnog tipa, ali 1.23 ne može se predstaviti kao vrednosti bilo kog osnovnog celobrojnog tipa. Zaokruživanje je dozvoljeno kada se numerički literal koristi za predstavljanje osnovnih numeričkih vrednosti koje nisu ceobrojne.

Svaki osnovni numerički tip ima opseg vrednosti koji se može predstaviti. Dakle, ako literal prelazi opseg vrednosti tipa, onda literal nije moguće predstaviti kao vrednosti tog tipa.

Neki primeri:

 Literal | Tipovi vrednosti koje literali mogu da predstavljaju
 --------- | --------------------------------------------------
 256 | Svi osnovni numerički tipovi osim tipova int8 i uint8.
 255 | Svi osnovni numerički tipovi osim int8 tipova.
 -123 | Svi osnovni numerički tipovi osim neoznačenih.
 123 | Svi osnovni numerički tipovi.
 123.000 | -II-
 1.23e2 | -II-
 'a' | -II-
 1.0+0i | -II-
 1.23 | Svi osnovni tipovi brojeva sa pokretnim zarezom i kompleksni numerički tipovi.
 0x10000000000000000 (16 nula) | -II-
 3.5e38 | Svi osnovni tipovi sa pokretnim zarezom i kompleksni numerički tipovi osim tipova float32 i complex64.
 1+2i | Svi osnovni kompleksni numerički tipovi.
 2e+308 | Nema osnovnih tipova.

> [!Note]
>
> - Pošto nijedna vrednost osnovnih celobrojnih tipova datih u Gou ne može da
> sadrži **0x10000000000000000**, literal se ne može predstaviti kao vrednost
> bilo kog osnovnog celobrojnog tipa.
> - Maksimalna IEEE-754 vrednost tipa float32 koja se može tačno predstaviti je > **3.40282346638528859811704183484516925440e+38**, tako da 3.5e38 se ne može predstaviti kao vrednosti bilo kog tipa float32 i complex64.
> - Maksimalna IEEE-754 float64 vrednost koja se može tačno predstaviti je **1.
> 797693134862315708145274237317043567981e+308**, tako da 2e+308se ne može
> predstaviti kao vrednosti bilo kog tipa float64 i complex128.
> - Na kraju, imajte na umu da, iako **0x10000000000000000** može da predstavlja
> vrednosti **float32** tipova, ne može tačno da predstavi nijednu float32
> vrednost u memoriji. Drugim rečima, biće zaokružen na najbližu **float32**
> vrednost koja može biti tačno predstavljena u memoriji kada se koristi kao
> vrednost **float32** tipova.

[Ključne reči i identfikatori][0102] | [Sadržaj][00] | [Konstante i promenljive][0104]

[0102]: 0102_Ključne_reči_i_identifikatori.md
[00]: 00_Sadrzaj.md
[0104]: 0104_Konstante_i_promenljive.md
