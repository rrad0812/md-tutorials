
# 1 Tokeni

[content][f0] [next][f2]

Tokeni su osnovni leksički gradivni blokovi izvornog koda; oni su "reči” jezika; karakteri se kombinuju u tokene prema pravilima programskog jezika. Postoji pet klasa tokena:

- **Rezervisane reči**  
  To su reči koje imaju fiksno značenje u jeziku. Ne mogu se menjati ili redefinisati.
- **Identifikatori**  
  Ovo su nazivi simbola koje programer definiše. Mogu se menjati i ponovo koristiti. Oni podležu pravilima o opsegu jezika.
- **Operatori**  
  To su obično simboli za matematičke ili druge operacije: +, -, * i tako dalje.
- **Separatori**  
  Ovo je obično beli prostori.
- **Konstante**  
  Numeričke ili karakterne konstante se koriste za označavanje stvarnih vrednosti u izvornom kodu, kao što su 1 (celobrojna konstanta) ili 2.3 (konstanta sa plutanjem) ili "String konstanta" (string: deo teksta).

U ovom poglavlju opisujemo sve Pascal rezervisane reči, kao i različite načine označavanja nizova, brojeva, identifikatora itd.

## 1.1 Symboli

Free Pascal dozvoljava sve znakove, cifre i neke posebne simbole znakova u Pascal izvornoj datoteci.

**Recognised symbols**:

- slova `A .. Z` i `a .. z`
- cifre  `0 .. 9`
- hexa cifre `$A .. $F`, `$a .. $f`
- Sledeći znakovi imaju posebno značenje:
  `' + - * / = < > [ ] . , ( ) : ^ @ { } $ # & %`
- a takođe i sledeći parovi znakova:
  `<< >> ** <> >< <= >= := += -= *= /= (* *) (. .) //`
- Kada se koristi u specifikaciji opsega, par znakova `(.`. je ekvivalentan leva uglata zagrada `[`.
- Isto tako, par znakova `.)` je ekvivalentan desnoj uglastoj zagradi `]`.
- Kada se koristi za graničnike komentara, par znakova `(*` je ekvivalentan leva zagrada `{`.
- i par znakova `*)` je ekvivalentan desnoj zagradi `}`.

Ovi parovi znakova zadržavaju svoje normalno značenje u string izrazima.

## 1.2 Komentari

Komentari su delovi izvornog koda koje kompajler potpuno odbacuje. Oni postoje samo u korist programera, tako da on može da objasni određene delove koda. Za kompajlera, kao da nema komentara.

Sledeći deo koda pokazuje komentar:

```pascal
(* My beautiful function returns an interesting result *)  
Function Beautiful : Integer;  
```

Upotreba `(*` i `*)` kao graničnika komentara datira od prvih dana Pascal jezika. Zamenjeni su uglavnom upotrebom `{` i `}` kao graničnika komentara, kao u sledećem primeru:

```pascal
{ My beautiful function returns an interesting result }  
Function Beautiful : Integer;  
```

Komentar takođe može da obuhvata više redova:

```pascal
{  
  My beautiful function returns an interesting result,  
  but only if the argument A is less than B.  
}  
Function Beautiful (A,B : Integer): Integer;
```

Komentari u jednom redu se takođe mogu napraviti pomoću `//` graničnika:

```pascal
// My beautiful function returns an interesting result  
Function Beautiful : Integer;  
```

Komentar se proteže od znaka `//` do kraja reda. Ovakvu vrstu komentara uveo je Borland u kompajleru Delphi Pascal.

Free Pascal podržava upotrebu ugneždenih komentara. Sledeće konstrukcije su validni komentari:

```pascal
(* This is an old style comment *)  
{  This is a Turbo Pascal comment }  
// This is a Delphi comment. All is ignored till the end of the line.
```

Slede validni načini umetnutih komentara:

```pascal
{ Comment 1 (* comment 2 *) }  
(* Comment 1 { comment 2 } *)  
{ comment 1 // Comment 2 }  
(* comment 1 // Comment 2 *)  
// comment 1 (* comment 2 *)  
// comment 1 { comment 2 }
```

Sledeće daje grešku jer nisu na istoj liniji:

```pascal
 // Valid comment { No longer valid comment !!  
    }
```

i ovako daje grešku

```pascal
 // Valid comment (* No longer valid comment !!  
    *)
```

Kompajler će reagovati sa "invalid character” greškom kada nađe ovakvu konstrukciju, bez obzira na -Mtp prekidač.

**Napomena**  
U TP i Delphi modu, ugnježdeni komentari nisu dozvoljeni. za maksimalnu kompatabilnost sa postojećim kodom tih kompajlera.

## 1.3 Rezervisane reči

Rezervisane reči su deo Pascal jezika i kao takve ih programer ne može redefinisati. Pascal ne razlikuje velika i mala slova, tako da će kompajler prihvatiti bilo koju kombinaciju velikih ili malih slova za rezervisane reči.

Pravimo razliku između Turbo Pascal i Delphi rezervisanih reči. U TP režimu, prepoznaju se samo rezervisane Turbo Pascal reči, ali Delphi se mogu redefinisati. Podrazumevano, Free Pascal prepoznaje rezervisane reči Delphi.

### 1.3.1 Turbo Pascal reserved words

Sledeće rezervisane reči postoje u Turbo Pascal modu:

 ... | ... | ... | ... | ... | ... | ... | ... |
 --- | --- | --- | --- | --- | --- | --- | --- |
absolute | and | array | asm | begin | case | const | constructor |
destructor | div | do | downto | else | end | file | for |
function | goto | if | implementation | inherited | inline | interface | label |
mod | nil | object | of | operator | or | packed | procedure |
program | record | reintroduce | repeat | self | set | shl | shr |
string | then | to | type | unit | until | uses | var |
while | with | xor | | | | | |

### 1.3.2 Object Pascal rezervisane reči

Rezervisane reči Object Pascala korišćene u Delphi ili Objfpc modu) su iste kao u Turbo Pascal, sa sledećim dodatnim rezervisanim rečima:

 ... | ... | ... | ... | ... | ... | ... |
 --- | --- | --- | --- | --- | --- | --- |
 as | class | dispinterface | except | exports | finalization | finally |
 initialization | inline | is | library | on | out | packed |
property | raise | resourcestring | threadvar | try | | |

### 1.3.3 Modifikatori

Sledi lista svih modifikatora. Nisu baš rezervisane reči u smislu da se mogu koristiti kao identifikatori, ali na određenim mestima imaju posebno značenje za kompajlera, tj., kompajler ih smatra delom Pascal jezika.

 ... | ... | ... | ... | ... | ... | ... |
 --- | --- | --- | --- | --- | --- | --- |
 absolute | abstract | alias | assembler | bitpacked | break | cdecl |
 continue | cppdecl | cvar | default | deprecated | dynamic | enumerator |
 experimental | export | external | far | far16 | forward | generic |
 helper | implements | index | interrupt | iocheck | local | message |
 name | near | nodefault | noreturn | nostackframe | oldfpccall | otherwise |
 overload | override | pascal | platform | private | protected | public |
 published | read | register | reintroduce | result | safecall | saveregisters |
 softfloat | specialize | static | stdcall | stored | strict | unaligned |
 unimplemented | varargs | virtual | winapi | write | | |

**Napomena**  
Predefinisani tipovi kao `Byte`, `Boolean` i `constants` kao i `maxint` nisu rezervisane reči. Oni su identifikatori, deklarisani u `system` unitu. To znači da ti tipovi mogu biti redefinisani u drugim unitima. Kako bilo, programer se ne ohrabruje da to radi, pošto to može izazvati konfuziju.

**Napomena**  
Od verzije 2.5.1 moguće je koristiti rezervisane reči kao identifikatore uz prefix `&` znak. To znači da je sledeće moguće:

```pascal
var  
  &var : integer;  
 
begin  
  &var:=1;  
  Writeln(&var);  
end.
```

Kako bilo, to nije preporučljivo u novom kodu, jer je kod manje čitljiv. Ovo je uglavnom namenjeno za popravku starog koda kada je lista rezervisanih reči bila manja.

## 1.4 Identifikatori

Identifikatori označavaju programski definisana imena za određene:

- `constants`,
- `types`,
- `variables`,
- `procedures`,
- `functions`,
- `units` i
- `programs`.

Sva imena definisana u izvornom kodu – isključujući rezervisane reči – su označena kao identifikatori.

Identifikatori se sastoje od 1 do 127 značajnih znakova (slova, cifara i donje crte), od kojih prvo mora biti slovo (a–z ili A–Z) ili donja crta (_).

Kao i Pascal rezervisane reči, identifikatori su neosetljivi na velika i mala slova, odnosno i jedno i drugo.
  
```pascal
  myprocedure;
```

and

```pascal
  MyProcedure;
```

referenciraju na istu proceduru.

**Napomena**  
Od verzije 2.5.1 moguće je navesti rezervisanu reč kao identifikator tako što ćete je dodati znakom ampersanda (&). To znači da je moguće sledeće:

```pascal
program testdo;  
 
procedure &do;  
 
begin  
end;  
 
begin  
  &do;  
end.
```

Rezervisana reč `do` se koristi kao identifikator za deklaraciju kao i za pozivanje procedure.

## 1.5 Hint direktive

Većina identifikatora (konstante, promenljive, funkcije ili metode, svojstva) mogu imati `hint` direktivu pridodatu njihovoj definiciji.

Kad god prevodilac kasnije naiđe na identifikator označen direktivom nagoveštaja, tada će se prikazati upozorenje koje odgovara navedenom nagoveštaju.

- **deprecated**
Upotreba ovog identifikatora je zastarela, umesto toga koristite alternativu. Deprecated ključna reč može biti praćena string konstantom sa porukom. Kompajler će prikazati ovu poruku kad god se naiđe na identifikator.

- **eksperimental**
Upotreba ovog identifikatora je eksperimentalna: može se koristiti za označavanje novih funkcija koje bi trebalo koristiti sa oprezom.

- **platform**
Ovo je identifikator zavisan od platforme: možda nije definisan na svim platformama.

- **notimplemented**
Ovo bi trebalo da se koristi samo za funkcije i procedure. Trebalo bi da se koristi da signalizira da određena funkcija još nije implementirana.

Slede primeri:

```pascal
Const  
  AConst = 12 deprecated;  

var  
  p : integer platform;  

Function Something : Integer; experimental;  

begin  
  Something:=P+AConst;  
end;  

begin  
  Something;  
end.
```

Ovo će rezultovati u sledećem izlazu:

```sh
testhd.pp(11,15) Warning: Symbol "p" is not portable  
testhd.pp(11,22) Warning: Symbol "AConst" is deprecated  
testhd.pp(15,3) Warning: Symbol "Something" is experimental
```

`Hint directives` can follow all kinds of identifiers:

- `units`,
- `constants`,
- `types`,
- `variables`,
- `functions`,
- `procedures` ind
- `methods`.

## 1.6 Brojevi

Brojevi su podrazumevano označeni decimalnim zapisom. Realni (ili decimalni) brojevi se pišu korišćenjem inženjerske ili naučne notacije (npr. 0,314E1).

Za konstante celobrojnog tipa, Free Pascal podržava četiri formata:

- **Decimalni format**, normalni format, (osnova 10). Ovo je standardni format.
- **Heksadecimalni format** (baza 16), na isti način kao što to radi Turbo
  Pascal. Da biste odredili konstantu vrednost u heksadecimalnom formatu, dodajte je ispred znaka dolara (`$`). Dakle, heksadecimalno $FF jednako 255 decimalno. Imajte na umu da su mala i mala slova beznačajna kada se koriste heksadecimalne konstante.
- **Oktalni format**, od verzije 1.0.7, (baza 8) je takođe podržan. Da biste
  naveli konstantu u oktalnom formatu, dodajte znak ampersanda (`&`). Na primer, 15 je navedeno u oktalnoj notaciji kao &17.
- **Binarni zapis** (baza 2). Binarni broj se može odrediti tako što se ispred
  njega stavi procenat znak (`%)`. Dakle, 255 se može navesti u binarnoj notaciji kao %11111111.

**Napomena**:
`Oktalni` i `Binarni` zapis nisu podržani u TP ili Delphi režimu kompatibilnosti.

## 1.7 Label

`Label` je naziv za lokaciju u izvornom kodu na koju se može preskočiti sa druge lokacije pomoću `goto` izjave. Oznaka je standardni identifikator ili niz cifara.

**Napomena**:
Prekidači `-Sg` ili `-Mtp` moraju biti specificirani pre nego što se `labele` mogu koristiti. Podrazumevano, Free Pascal ne podržava naredbe `label` i `goto`. Direktiva `{$GOTO ON}` takođe se može koristiti da se dozvoli korišćenje `label` i naredbe `goto`.

Slede primeri važećih labela:

```pascal
Label  
123,  
abc;
```

## 1.8 Nizovi znakova ili stringovi

Niz znakova (ili skraćeno string) je niz od nula ili više znakova (veličine u bajtovima), zatvoren u jednostruke navodnike i u jednom redu izvornog koda programa; u stringu se ne mogu pojaviti literalni znakovi `#CR` ili `#LF`.

Skup znakova bez ičega između navodnika (’’) je prazan string.

String se sastoji od standardnih, 8-bitnih ASCII znakova ili Unicode (obično kodiranih UTF-8) znakova. Kontrolni string se može koristiti za određivanje znakova koji se ne mogu ukucati na tastaturi, kao što je #27 - ESC znak.

Jednostruki navodnik se može ugraditi u string tako što ćete ga ukucati dvaput. C konstrukcija eskejp znakova u stringu (koristeći obrnutu kosu crtu) ​​nije podržana u Pascal-u.

Sledeće su važeće string konstante:
  
```pascal  
  'This is a pascal string'  
  ''  
  'a'  
  'A tabulator character: '#9' is easy to embed'
```

Sledeći string je pogrešan:

```pascal
  'the string starts here  
   and continues here'
```

Gornji string mora biti unet kao:

```pas
  'the string starts here'#13#10'   and continues here'
```

na Windows OS, ili
  
```pascal  
  'the string starts here'#10'   and continues here'
```

na Uniksima uključujući i Mac OS X, i:

```pascal
  'the string starts here'#13'   and continues here'
```

na klasičnim Mac-like OS.

Moguće je koristiti druge skupove znakova u stringovima; u tom slučaju kodna stranica izvorne datoteke mora biti specificirana sa `{$CODEPAGE XXX}` direktivom ili sa opcijom komandne linije `-Fc` za kompajler. U tom slučaju će znakovi u stringu biti interpretirani kao znakovi sa navedene kodne stranice.

[content][f0] [next][f2]

[f0]: 00_sadrzaj.md
[f2]: 02_konstante.md
