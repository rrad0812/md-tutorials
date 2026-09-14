# 5 Izrazi

[prev][f4] [content][f0] [next][f6]

Izrazi se javljaju u naredbama dodele ili u testovima.

Izrazi se sastoje od dve komponente: operatora i njihovih operanda. Većina operatora je binarna, tj. zahteva dva operanda. Binarni operatori se uvek javljaju između operanda (kao u X/Y ). Malo operatora je unarno, tj. zahteva samo jedan operand. Unarni operator se uvek javlja ispred operanda, kao u -X .

Izraz se pretvara u vrednost određenog tipa. Rezultujući tip je određen tipovima vrednosti u izrazu i operatorima u izrazu.

Kada se u izrazu koristi više operanda, koriste se pravila prioriteta iz tabele ( 5.1 ).

Tabela 5.1: Prioritet operatora

 Operator | Precedence | Category |
 -------- | ---------- | -------- |
 `not`, `unary +`, `unary -`, `@`, `**` | Highest (first) | Unary operators, power |
`*`, `/`, `div`, `mod`, `and`, `shl`, `shr`, `as`, `<<`, `>>` | Second | Multiplying operators |
`+`, `-`, `or`, `xor`, `><` | Third | Adding operators |
`=`, `<>`, `<`, `>`, `<=`, `>=`, `in`, `is` | Lowest (Last) | relational operators |

Prilikom određivanja prioriteta, kompajler koristi sledeća pravila:

- Kod operacija sa nejednakim prioritetima, operandi pripadaju operatoru sa najvećim prioritetom. Na
  primer, u 5*3+7 , množenje ima veći prioritet od sabiranja, tako da se ono prvo izračunava. Rezultat bi bio 22.
- Ako se zagrade koriste u izrazu, njihov sadržaj se prvo izračunava. Dakle, 5*(3+7) bi rezultiralo
  sa 50.
- U suprotnom, binarni operatori istog prioriteta su levo-asocijativni. 5 * 3 div 7 će dati rezultat
  2, a ne 0.

**Napomena**  
Redosled kojim se izrazi istog prioriteta izračunavaju nije garantovano sleva nadesno. Generalno, u takvom slučaju ne treba praviti pretpostavke o tome koji se podizraz prvi izračunava.

Kompilator će odlučiti koji podizraz će prvo izračunati na osnovu pravila optimizacije. Dakle, u sledećem izrazu:

```pascal
a := g(3) + f(2);
```

Funkcija f(2) može biti izvršena pre funkcije g(3). Ovo ponašanje se značajno razlikuje od Delfija ili Turbo Paskala.

Ako jedan izraz mora biti izvršen pre drugog, potrebno je podeliti iskaz koristeći privremene rezultate:

```pascal
e1 := g(3);  
a := e1 + f(2);
```

Značajan izuzetak od ovog ponašanja je izračunavanje Bulovih izraza: ako je omogućeno kratkotrajno bulovo izračunavanje (podrazumevano), onda će kompajler izvršavati sleva nadesno, ali će i dalje poštovati prioritet, tj. u delovima sa jednakim prioritetom levi operand će uvek biti izračunat pre desnog. Dakle, sledeći primer:

```pascal
True or True and False
```

će se proceniti kao True, jer je ekvivalentno sa

```pascal
True ili (True and False) 
```

## 5.1 Sintaksa izraza

Izraz primenjuje relacione operatore na jednostavne izraze. Jednostavni izrazi su nizovi termina (šta je termin, objašnjeno je u nastavku), spojenih operatorom sabiranja.

Sledeći izrazi su validni:

```pascal
GraphResult<>grError  
(DoItToday=Yes) and (DoItTomorrow=No);  
Day in Weekend
```

A evo i nekoliko jednostavnih izraza:

```pascal
A + B  
-Pi  
ToBe or NotToBe 
```

Članovi se sastoje od činilaca, povezanih operatorima množenja.

Evo nekih važećih termina:

```pascal
2 * Pi  
A Div B  
(DoItToday=Yes) and (DoItTomorrow=No);
```

Faktori su sve ostale konstrukcije.

## 5.2 Pozivi funkcija

Pozivi funkcija su deo izraza (mada, koristeći proširenu sintaksu, mogu biti i iskazi).

Referenca promenljive mora biti referenca promenljive proceduralnog tipa. Oznaka metode može se koristiti samo unutar metode objekta. Kvalifikovana oznaka metode može se koristiti i van metoda objekta. Funkcija koja će biti pozvana je funkcija sa deklarisanom listom parametara koja se podudara sa stvarnom listom parametara. To znači da

- Broj stvarnih parametara mora biti jednak broju deklarisanih parametara (osim ako se ne koriste podrazumevane vrednosti parametara).
- Tipovi parametara moraju biti kompatibilni. Za parametre promenljivih referenci, tipovi parametara moraju biti potpuno isti.

Ako se ne pronađe odgovarajuća funkcija, kompajler će generisati grešku. Koja greška zavisi – između ostalog – od toga da li je funkcija preopterećena ili ne: tj. više funkcija sa istim imenom, ali različitim listama parametara.

Postoje slučajevi kada kompajler neće izvršiti poziv funkcije u izrazu. To je slučaj kada se dodeljuje vrednost promenljivoj proceduralnog tipa, kao u sledećem primeru u Delfi ili Turbo Paskal režimu:

```pascal
 Type  
  FuncType = Function: Integer;  

Var 
  A : Integer;  

Function AddOne : Integer;  
begin  
  A := A+1;  
  AddOne := A;  
end;  

Var F : FuncType;  
    N : Integer;  

begin  
  A := 0;  
  F := AddOne; { Assign AddOne to F, Don't call AddOne }  
  N := AddOne; { N := 1 !!}  
end.
```

U gornjem listingu, dodela vrednosti F neće izazvati poziv funkcije AddOne. Međutim, dodela vrednosti N će pozvati AddOne.

Ponekad je poziv željen, na primer u rekurziji, u tom slučaju, poziv mora biti prinuđen. To se može uraditi dodavanjem zagrade nazivu funkcije:

```pascal
function rd : char;  
var  
  c : char;  
 
begin  
  read(c);  
  if (c='\') then  
    c:=rd();  
  rd:=c;  
end;  
 
var ch : char;  
 
begin  
   ch:=rd;  
   writeln(ch);  
end.
```

Gore navedeno će pročitati karakter i ispisati ga. Ako je unos obrnuta kosa crta, čita se drugi karakter.

Problem sa ovom sintaksom je sledeća konstrukcija:

```pascal
If F = AddOne Then  
  DoSomethingHorrible;
```

Da li kompajler treba da uporedi adrese F i AddOne , ili treba da pozove obe funkcije i uporedi rezultat? U Fpc i ObjFpc režimu ovo se rešava tako što se proceduralna promenljiva smatra ekvivalentna pokazivaču. Stoga će kompajler dati grešku neusklađenosti tipa, jer se AddOne smatra pozivom funkcije sa celobrojnim rezultatom, a F je pokazivač.

Kako onda treba proveriti da li F ukazuje na funkciju AddOne ? Da bi se to uradilo, treba koristiti operator adrese @:

```pascal
If F = @AddOne Then  
  WriteLn ('Functions are equal');
```

Leva strana bulovog izraza je adresa. Desna strana takođe, pa kompajler upoređuje dve adrese. Kako uporediti vrednosti koje obe funkcije vraćaju? Dodavanjem prazne liste parametara:

```pascal  
If F()=Addone then  
  WriteLn ('Functions return same values ');
```

Imajte na umu da ovo poslednje ponašanje nije kompatibilno sa Delfi sintaksom. Uključivanje Delfi režima će vam omogućiti da koristite Delfi sintaksu.

## 5.3 Konstruktori setova

Kada se konstanta tipa seta mora uneti u izraz, mora se navesti konstruktor seta. U suštini, ovo je isto kao kada je tip definisan, samo što nema identifikatora za identifikaciju seta. Konstruktor seta je lista izraza odvojenih zarezima, zatvorena u uglaste zagrade.

Sve grupe setova i elementi skupa moraju biti istog rednog tipa. Prazan set se označava sa [] i može se dodeliti bilo kom tipu seta. Grupa setova sa opsegom [A..Z] čini sve vrednosti u opsegu elementom skupa. Sledeći su validni konstruktori skupova:

```pascal
[Monday..Friday,Sunday]  
[ 2, 3*2, 6*2, 9*2 ]  
['A'..'Z','a'..'z','0'..'9']
```

**Napomena**  
Ako prvi specifikator opsega ima veću rednu vrednost od drugog, rezultujući skup će biti prazan, npr. ['Z'..'A'] označava prazan skup. Treba biti oprezan pri označavanju opsega.

## 5.4 Typecast vrednosti

Ponekad je potrebno promeniti tip izraza, ili dela izraza, da bi bio kompatibilan sa dodelom. To se radi putem pretvaranja (typecast) tipa vrednosti.

Vrednosni typecast ne može se koristiti na levoj strani dodela, kao typecast promenljivih. Evo nekih validnih `typecast`-ova:

```pascal
Byte('A')  
Char(48)  
boolean(1)  
longint(@Buffer) 
```

Generalno, veličina tipa izraza i veličina typecast-a moraju biti iste. Međutim, za ordinalne tipove (bajt, karakter, reč, bulova vrednost, nabrajanja) to nije tako, oni se mogu koristiti naizmenično. To jest, sledeće će funkcionisati, iako se veličine ne podudaraju.

```pascal
Integer('A');  
Char(4875);  
boolean(100);  
Word(@Buffer);
```

Ovo je kompatibilno sa ponašanjem Delfija ili Turbo Paskala.

## 5.5 Typecast promenljivih

Promenljiva se može smatrati jednim faktorom u izrazu. Stoga se može i typecast-ovati. Promenljiva se može typecast-ovati u bilo koji tip, pod uslovom da tip ima istu veličinu kao i originalna promenljiva.

Loša je ideja konvertovati celobrojne tipove u realne tipove i obrnuto. Bolje je osloniti se na kompatibilnost dodeljivanja tipova i koristiti neke od standardnih funkcija za promenu tipa.

Imajte na umu da se pretvaranje tipova promenljivih može pojaviti sa obe strane dodele, tj. da su oba važeća pretvaranja tipova:

Var  
  C : Char;  
  B : Byte;  

begin  
  B:=Byte(C);  
  Char(B):=C;  
end;

Promenljive pokazivači mogu biti pretvorene u proceduralne tipove, ali ne i u pokazivače metoda.

Typecast je izraz datog tipa, što znači da iza typecast može slediti kvalifikator:

Type  
  TWordRec = Packed Record  
  L,H : Byte;  
  end;  

Var  
  P : Pointer;  
  W : Word;  
  S : String;  

begin  
  TWordRec(W).L:=$FF;  
  TWordRec(W).H:=0;  
  S:=TObject(P).ClassName;
...

## 5.6 Neporavnati typecast

Specijalni typecast je `Unaligned typecast` promenljive ili izraza. Ovo nije pravo pretvaranje tipa u tip, već je pre nagoveštaj za kompajler da izraz može biti pogrešno poravnat (tj. nije na poravnatoj memorijskoj adresi). Neki procesori ne dozvoljavaju direktan pristup pogrešno poravnatim strukturama podataka i stoga moraju pristupati podacima bajt po bajt.

Pretvaranje izraza u tip pomoću ključne reči `unaligned` signalizira kompajleru da treba da pristupa podacima bajt po bajt.

Imajte na umu da kompajler pretpostavlja da je pristup svim poljima/elementima upakovanih struktura podataka `unaligned`.

Primer:

program me;  

Var  
  A : packed Array[1..20] of Byte;  
  I : LongInt;  

begin  
  For I:=1 to 20 do  
    A[I]:=I;  
  I:=PInteger(Unaligned(@A[13]))^;  
end.

## 5.7 Operator @

Operator adrese @ vraća adresu promenljive, procedure ili funkcije.

Operator @ vraća tipizirani pokazivač ako je prekidač `$T` uključen. Ako je prekidač `$T` isključen, onda operator adrese vraća netipizirani pokazivač, koji je kompatibilan u dodeli sa svim tipovima pokazivača. Tip pokazivača je ^T , gde je T tip reference promenljive. Na primer, sledeće će se kompajlirati:

Program tcast;  

{$T-} { @ returns untyped pointer }  

Type
  art = Array[1..100] of byte;  

Var Buffer : longint;  
    PLargeBuffer : ^art;  

begin  
 PLargeBuffer := @Buffer;  
end.

Promena `{ $T- }` u `{ $T+ }` će sprečiti kompajler da ovo kompajlira. Dobićete grešku zbog neusklađenosti tipova.

Podrazumevano, operator adrese vraća netipizovani pokazivač: primena operatora adrese na identifikator funkcije, metode ili procedure daće pokazivač na ulaznu tačku te funkcije. Rezultat je netipizovani pokazivač.

To znači da će sledeće funkcionisati:

Procedure MyProc;  

begin  
end;  

Var  
  P : PChar;  

begin  
  P:=@MyProc;  
end;

Podrazumevano, operator adrese mora se koristiti ako vrednost mora biti dodeljena promenljivoj proceduralnog tipa. Ovo ponašanje se može izbeći korišćenjem prekidača -Mtp ili -MDelphi , što rezultira kompatibilnijom sintaksom Delphi ili Turbo Pascal jezika.

## 5.8 Operatori

Operatori se mogu klasifikovati prema tipu izraza na kome operišu. Razmotrićemo ih tip po tip.

### 5.8.1 Aritmetički operatori

Aritmetički operatori se javljaju u aritmetičkim operacijama, tj. u izrazima koji sadrže cele ili realne brojeve. Postoje dve vrste operatora: binarni i unarni aritmetički operatori. Binarni operatori su navedeni u tabeli ( 5.2 ), unarni operatori su navedeni u tabeli ( 5.3 ).

Tabela 5.2: Binarni aritmetički operatori

 Operator | Operacija |
 -------- | --------- |
 `+` | Dodatak |
 `-` | Oduzimanje |
 `*` | Množenje |
 `**` | Stepencijacija |
 `/` | Deljenje |
 `Div` | Deljenje celih brojeva |
 `Mod` | Ostatak |

Sa izuzetkom `Div` i `Mod`, koji prihvataju samo celobrojne izraze kao operand, svi operatori prihvataju realne i celobrojne izraze kao operand.

**Napomena**  
Operator stepenovanja ( `**` ) je dostupan za preopterećenje (poglavlje 15 , strana 846 ), ali nije definisan ni na jednom od standardnih Paskal tipova (brojevi sa pokretnim decimalom i/ili celi brojevi).

Za binarne operatore, tip rezultata će biti ceo broj ako su oba operanda izrazi celobrojnog tipa. Ako je jedan od operanda izraz realnog tipa, onda je rezultat realan broj.

Izuzetno, deljenje ( `/` ) uvek rezultira realnim vrednostima.

Tabela 5.3: Unarni aritmetički operatori

 Operator | Operacija |
 -------- | --------- |
 `+` | Identitet znaka |
 `-` | Inverzija znakova |

Za unarne operatore, tip rezultata je uvek jednak tipu izraza. Operator deljenja ( / ) i operator Mod će izazvati greške tokom izvršavanja ako je drugi argument nula.

Znak rezultata Mod operatora je isti kao znak levog operanda Mod operatora. U stvari, Mod operator je ekvivalentan sledećoj operaciji:

```pascal
I mod J = I - (I div J) * J
```

Ali se izvršava brže od izraza sa desne strane.

### 5.8.2 Logički operatori

Logički bitwise operatori deluju na pojedinačne bitove ordinalnih izraza. Logički operatori zahtevaju operande koji su celobrojnog tipa i proizvode rezultat celobrojnog tipa. Mogući logički operatori su navedeni u tabeli ( 5.4 ).

Tabela 5.4: Logički bitwise operatori

 Operator | Operacija |
 -------- | --------- |
 not | Bitwise negation (unary) |
 and | Bitwise and |
 or | Bitwise or |
 xor | Bitwise xor |
 shl | Bitwise shift to the left |
 shr | Bitwise shift to the right |
 `<<` | Bitwise shift to the left (same as shl) |
 `>>` | Bitwise shift to the right (same as shr) |

Sledeći su validni bitwise logički izrazi:

A shr 1  { same as A div 2, but faster}  
Not 1    { equals -2 }  
Not 0    { equals -1 }  
Not -1   { equals 0  }  
B shl 2  { same as B * 4 for integers }  
1 or 2   { equals 3 }  
3 xor 1  { equals 2 }

### 5.8.3 Bulovi operatori

Bulove operacije se mogu smatrati logičkim operacijama na tipu veličine 1 bita. Stoga, `shl` i `shr` operacije nemaju mnogo smisla. Bulove operacije mogu imati samo operandove bulovog tipa, a rezultujući tip je uvek bulov. Mogući operatori su navedeni u tabeli ( 5.5 ).

Tabela 5.5: Bulovi operatori

 Operator | Operacija |
 -------- | --------- |
 not | logical negation (unary) |
 and | logical and |
 or | logical or |
 xor | logical xor |

**Napomena**  
Podrazumevano, bulovi izrazi se izračunavaju pomoću tzv. kratkog spoja. To znači da se od trenutka kada je rezultat celog izraza poznat, izračunavanje zaustavlja i vraća se rezultat. Na primer, u sledećem izrazu:

```pascal
B := True ili MaybeTrue;
```

Kompilator nikada neće pogledati vrednost MaybeTrue, jer je očigledno da će izraz uvek biti True. Kao rezultat ove strategije, ako je MaybeTrue funkcija, ona neće biti pozvana! (Ovo može imati iznenađujuće efekte kada se koristi zajedno sa svojstvima)!

### 5.8.4 Operatori stringova

Postoji samo jedan operator za stringove: `+` . Njegova akcija je da spaja sadržaj dva stringa (ili znaka) na koje deluje. Ne može se koristiti za spajanje stringova koji se završavaju nulom ( `PChar` ). Sledeće su validne operacije za stringove:

```pascal
'This is ' + 'VERY ' + 'easy !'  
Dirname+'\'
```

Sledeće nije:

```pascal
Var  
  Dirname : PChar;  
...  
  Dirname := Dirname+'\'; 
```

zato što je Dirname string koji se završava nulom.

Imajte na umu da ako su svi stringovi u string izrazima ShortString-ovi, rezultujući string je takođe ShortString. Stoga može doći do skraćivanja: ne postoji automatsko povećanje veličine na AnsiString.

Ako su svi stringovi u string izrazu AnsiStringovi, onda je rezultat AnsiString.

Ako izraz sadrži mešavinu AnsiStringova i ShortStringova, rezultat je AnsiString.

Vrednost prekidača `{ $H }` može se koristiti za kontrolu tipa konstantnih stringova; podrazumevano su to ShortString-ovi (i stoga ograničeni na 255 znakova).

### 5.8.5 Operatori dinamičkih stringova

Postoji samo jedan dinamički operator stringa: `+` . Ovaj operator je dostupan u Delphi režimu, ali mora biti eksplicitno omogućen korišćenjem operatora `{$modeswitch arrayoperators}` u `objfpc` režimu:

```pascal
{$mode objfpc}  
{$modeswitch arrayoperators}
```

Kada je omogućeno, njegova akcija je slična spajanju stringova: spajanje sadržaja dva stringa na koje deluje. Tip elementa niza mora naravno biti identičan za oba niza, tj. sledeće će funkcionisati:

```pascal
{$mode objfpc}  
{$modeswitch arrayoperators}  

var  
  a,b, c : array of byte;  
 
begin  
  a:=[0,1,2];  
  b:=[3,4,5];  
  c:=a+b;  
  writeln('C has ',length(c),' elements'); ```
Type  
  Day = (mon,tue,wed,thu,fri,sat,sun);  
  Days = set of Day;  
 
Procedure PrintDays(W : Days);  
Const  
  DayNames : array [Day] of String[3]  
           = ('mon','tue','wed','thu',  
              'fri','sat','sun');  
Var  
  D : Day;  
  S : String;  
begin  
  S:='';  
  For D:=Mon to Sun do  
    if D in W then  
      begin  
      If (S<>'') then S:=S+',';  
      S:=S+DayNames[D];  
      end;  
  Writeln('[',S,']');  
end;  
 
Const  
  WorkWeek = [mon,tue,wed,thu,fri];  
  WeekEnd = [sat,sun];  
 
Var  
  W : Days;  
 
begin  
  W:=[mon,tue]+[wed,thu,fri]; // equals [mon,tue,wed,thu,fri]  
  PrintDays(W);  
  W:=[mon,tue,wed]-[wed];     // equals [mon,tue]  
  PrintDays(W);  
  W:=[mon,tue,wed]-[wed,thu];     // also equals [mon,tue]  
  PrintDays(W);  
  W:=[mon,tue,wed]*[wed,thu,fri]; // equals [wed]  
  PrintDays(W);  
  W:=[mon,tue,wed]><[wed,thu,fri]; // equals [mon,tue,thu,fri]  
  PrintDays(W);  
  if [mon,tue]<=WorkWeek then  
    Writeln('Must work on monday and tuesday');  
  if Weekend>=[sun] then  
    Writeln('Can rest on sunday');  
end.
```

Ali sledeće neće funkcionisati:

```pascal
var  
  b, c : array of byte;  
  a : array of integer;  
 
begin  
  a:=[0,1,2];  
  b:=[3,4,5];  
  c:=a+b;  
  writeln('C has ',length(c),' elements'); 
```

Kompilator će javiti grešku prilikom kompajliranja ovog koda.

### 5.8.6 Operatori setova

Sledeće operacije nad setovima mogu se izvoditi pomoću operatora: `Union`, `Difference`, `SimmetricDifference`, `Inclusion` i `Intersection`. Elementi se mogu dodavati ili uklanjati iz seta pomoću operatora `Include` ili Exclude`. Operatori potrebni za ovo su navedeni u tabeli ( 5.6 ).

Tabela 5.6: Operatori setova

 Operator | Akcija |
 -------- | ------ |
 `+` | Union |
 `-` | Difference |
 `*` | Intersection |
 `><` | Symmetric difference |
 `<=` | Contains |
 `>=` | Left hand side set is a superset of the one on the right |
 `include` | include an element in the set |
 `exclude` | exclude an element from the set |
 `in` | check whether an element is in a set |

Tip operanda mora biti isti, inače će kompajler generisati grešku.

Sledeći program daje neke validne primere operacija sa setovima:

```pascal
Type  
  Day = (mon,tue,wed,thu,fri,sat,sun);  
  Days = set of Day;  
 
Procedure PrintDays(W : Days);  
Const  
  DayNames : array [Day] of String[3]  
           = ('mon','tue','wed','thu',  
              'fri','sat','sun');  
Var  
  D : Day;  
  S : String;  
begin  
  S:='';  
  For D:=Mon to Sun do  
    if D in W then  
      begin  
      If (S<>'') then S:=S+',';  
      S:=S+DayNames[D];  
      end;  
  Writeln('[',S,']');  
end;  
 
Const  
  WorkWeek = [mon,tue,wed,thu,fri];  
  WeekEnd = [sat,sun];  
 
Var  
  W : Days;  
 
begin  
   W:=[mon,tue]+[wed,thu,fri]; // equals [mon,tue,wed,thu,fri]  
   PrintDays(W);  
   W:=[mon,tue,wed]-[wed];     // equals [mon,tue]  
   PrintDays(W);  
   W:=[mon,tue,wed]-[wed,thu];     // also equals [mon,tue]  
   PrintDays(W);  
   W:=[mon,tue,wed]*[wed,thu,fri]; // equals [wed]  
   PrintDays(W);  
   W:=[mon,tue,wed]><[wed,thu,fri]; // equals [mon,tue,thu,fri]  
   PrintDays(W);  
   if [mon,tue]<=WorkWeek then  
     Writeln('Must work on monday and tuesday');  
   if Weekend>=[sun] then  
     Writeln('Can rest on sunday');  
end.
```

Kao što se može videti, unija je ekvivalentna binarnom OR, dok je presek ekvivalentan binarnom AND, a simetrična razlika je jednaka XOR operaciji.

Operacije `Include` i `Exclude` su ekvivalentne uniji ili razlici sa setom od jednog elementa. Dakle,

```pascal
Include(W, wed);
```

je ekvivalentno

```pascal
  W:=W + [wed];
```

i

```pascal
Exclude(W, wed);
```

je ekvivalentno

```pascal
W:=W - [wed];
```

Operacija `In` rezultira vrednošću True ako je levi operand (element) uključen u desni operand (set), u suprotnom rezultat će biti False.

### 5.8.7 Relacioni operatori

Relacioni operatori su navedeni u tabeli ( 5.7 )

Tabela 5.7: Relacioni operatori

 Operator | Akcija |
 -------- | ------- |
 `=` | Jednako |
 `<>` | Nije jednako |
 `<` | Strogo manje od |
 `>` | Strogo veće od |
 `<=` | Manje od ili jednako |
 `>=` | Veće ili jednako |
 `in` | Element of |

Normalno, levi i desni operandi moraju biti istog tipa. Postoje neki značajni izuzeci, gde kompajler može da obrađuje mešovite izraze:

- Celobrojni i realni tipovi mogu se mešati u relacionim izrazima.
- Ako je operator preopterećen i postoji preopterećena verzija čiji tipovi argumenata odgovaraju tipovima u izrazu.
- Mogu se mešati tipovi kratkih, ANSI i širokih žičanih vlakana.

Poređenje stringova se vrši na osnovu njihove reprezentacije karakternog koda.

Prilikom upoređivanja pokazivača, upoređuju se adrese na koje oni ukazuju. Ovo važi i za pokazivače tipa `PChar`. Da bi se uporedili stringovi na koje `PChar` pokazuje, mora se koristiti funkcija `StrComp` iz unita `String`. Funkcija `in` vraća True ako je levi operand (koji mora imati isti ordinalni tip kao i tip skupa i koji mora biti u opsegu 0..255) element skupa koji je desni operand, u suprotnom vraća False.

### 5.8.8 Operatori klase

Operatori klase se malo razlikuju od gore navedenih operatora u smislu da se mogu koristiti samo u izrazima klase koji vraćaju klasu. Postoje samo dva operatora klase, kao što se može videti u tabeli ( 5.8 ).

Tabela 5.8: Operatori klase

 Operator | Akcija |
 -------- | ------ |
 `is` | Proverava tip klase |
 `as` | Uslovno pretvaranje tipa |

Izraz koji sadrži operator `is` rezultira bulovskim tipom. Operator is može se koristiti samo sa referencom klase ili instancom klase. Upotreba ovog operatora je sledeća:

```pascal
Object is Class
```

Ovaj izraz je potpuno ekvivalentan

```pascal
Object.InheritsFrom(Class)
```

Ako je Objekat Nil, biće vraćeno False.

Sledeći su primeri:

```pascal
Var  
  A : TObject;  
  B : TClass;  
 
begin  
  if A is TComponent then ...;  
  If A is B then ...;  
end;
```

Operator `as` vrši uslovno pretvaranje tipa u tip. Rezultat je izraz koji ima tip klase:

```pascal
Object as Class 
```

Ovo je ekvivalentno sledećim izjavama:

```pascal
If Object=Nil then  
  Result:=Nil  
else if Object is Class then  
  Result:=Class(Object)  
else  
  Raise Exception.Create(SErrInvalidTypeCast);
```

Imajte na umu da ako je objekat Nil, operator ˙as` ne generiše izuzetak.

Slede neki primeri upotrebe operatora `as`:

```pascal
Var  
  C : TComponent;  
  O : TObject;  
 
begin  
  (C as TEdit).Text:='Some text';  
  C:=O as TComponent;  
end; 
```

Operatori `as` i `is` takođe rade na interfejsima (i COM i CORBA). Mogu se koristiti za proveru da li interfejs implementira i drugi interfejs kao u sledećem primeru:

```pascal
{$mode objfpc}  
 
uses  
  SysUtils;  
 
type  
  IMyInterface1 = interface  
    ['{DD70E7BB-51E8-45C3-8CE8-5F5188E19255}']  
    procedure Bar;  
  end;  
 
  IMyInterface2 = interface  
    ['{7E5B86C4-4BC5-40E6-A0DF-D27DBF77BCA0}']  
    procedure Foo;  
  end;  
 
  TMyObject = class(TInterfacedObject, IMyInterface1, IMyInterface2)  
    procedure Bar;  
    procedure Foo;  
  end;  
 
procedure TMyObject.Bar;  
begin  
 
end;  
 
procedure TMyObject.Foo;  
begin  
 
end;  
 
var  
  i: IMyInterface1;  
begin  
  i := TMyObject.Create;  
  i.Bar;  
  Writeln(BoolToStr(i is IMyInterface2, True)); // prints true  
  Writeln(BoolToStr(i is IDispatch, True)); // prints false  
  (i as IMyInterface2).Foo;  
end.
```

Pored toga, operator `is` može se koristiti za proveru da li klasa implementira interfejs, a operator `as` može se koristiti za pretvaranje interfejsa u tip klase:

```pascal
{$mode objfpc}  
var  
  i: IMyInterface;  
begin  
  i := TMyObject.Create;  
  Writeln(BoolToStr(i is TMyObject,True)); // prints true  
  Writeln(BoolToStr(i is TObject,True)); // prints true  
  Writeln(BoolToStr(i is TAggregatedObject,True)); // prints false  
  (i as TMyObject).Foo;  
end. 
```

Iako interfejsi moraju biti COM interfejsi, pretvaranje tipa u klasu će funkcionisati samo ako interfejs dolazi iz Object Pascal klase. Neće raditi na interfejsima dobijenim iz sistema putem COM-a.

[prev][f4] [content][f0] [next][f6]

[f0]: 00_sadrzaj.md
[f4]: 04_promenljive.md
[f6]: 06_izjave.md
