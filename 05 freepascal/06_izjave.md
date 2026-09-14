# 6 Izjave

[prev][f5] [content][f0] [next][f7]

Srž svakog algoritma su akcije koje on preduzima. Ove akcije su sadržane u naredbama programa ili unita. Svaka naredba može biti označena i na nju se može skočiti (unutar određenih ograničenja) pomoću naredbi Goto.

Oznaka može biti identifikator ili cela cifra.

## 6.1 Jednostavne izjave

Jednostavna izjava se ne može razložiti na odvojene izjave. Postoje u osnovi četiri vrste jednostavnih izjava:

- izjave dodele
- procedure statement
- goto statement
- raise statement

Od izjava raise biće objašnjena u poglavlju o izuzecima.

### 6.1.1 Izjave dodele

Dodele daju vrednost promenljivoj, zamenjujući bilo koju prethodnu vrednost koju je promenljiva mogla imati.

Pored standardnog operatora dodele u Paskalu ( := ), koji jednostavno zamenjuje vrednost promenljive vrednošću koja je rezultat izraza desno od operatora := , FreePaskal podržava neke konstrukcije u C stilu. Sve dostupne konstrukcije su navedene u tabeli ( 6.1 ).

Tabela 6.1: Dozvoljene C konstrukcije u Free Pascal-u

 Dodela | Rezultat |
------- | -------- |
 a += b | Dodaje b broju a i čuva rezultat u broju a. |
 a -= b | Oduzima b od a i čuva rezultat u a. |
 a *= b | Množi a sa b i čuva rezultat u a. |
 a /= b | Deli broj a sa brojem b i čuva rezultat u broju a. |

Da bi ove konstrukcije funkcionisale, mora biti naveden prekidač komandne linije `-Sc`.

**Napomena**  
Ove konstrukcije su samo radi lakšeg kucanja, ne generišu drugačiji kod. Evo nekoliko primera validnih naredbi dodele:

```pascal
X+=Y;      { Same as X := X+Y, needs -Sc command line switch}  
X/=2;      { Same as X := X/2, needs -Sc command line switch}  
Done := False;  
Weather := Good;  
MyPi := 4* Tan(1);
```

Imajući u vidu da dereferenciranje tipiziranog pokazivača rezultira promenljivom tipa na koju pokazivač pokazuje, sledeće dodele su takođe validne:

```pascal
Var  
  L : ^Longint;  
  P : PPChar;  
 
begin  
  L^:=3;  
  P^^:='A';
```

Obratite pažnju na dvostruko dereferenciranje u drugoj dodeli.

### 6.1.2 Izjave proceduralne

Proceduralne izjave su pozivi potprograma. Postoje različite mogućnosti za pozive procedura:

- Normalan poziv procedure.
- Poziv metode objekta (potpuno kvalifikovan ili ne).
- Ili čak poziv promenljive proceduralnog tipa.

Kompilator FreePascal-a će tražiti proceduru sa istim imenom kao što je dato u naredbi procedure, i sa deklarisanom listom parametara koja odgovara stvarnoj listi parametara. Sledeće su validne izjave procedure:

```pascal
WriteLn('Paskal je jednostavan jezik!');  
Doit();
```

**Napomena**  
Kada se traži funkcija koja odgovara listi parametara poziva, tipovi parametara treba da budu kompatibilni sa dodelom za parametre value i const i treba da se tačno podudaraju za parametre koji se prosleđuju referencom.

### 6.1.3 Izjava goto

FreePaskal podržava goto izjavu.

Prilikom korišćenja goto izjava, potrebno je imati na umu sledeće:

- `Oznaka` skoka mora biti definisana u istom bloku kao i `goto` naredba.
- Skok iz spoljašnje petlje u unutrašnju petlju ili obrnuto može imati čudne efekte.
- Da biste mogli da koristite `goto` naredbu, mora se koristiti prekidač kompajlera `-Sg` ili `{ $ GOTO ON }`.

**Napomena**  
U iso ili macpas režimu, ili sa prekidačem režima `nonlocalgoto`, kompajler će takođe dozvoliti nelokalne `goto` komande.

Goto naredbe se smatraju lošom praksom i treba ih izbegavati koliko god je to moguće. Uvek je moguće zameniti goto naredbu konstrukcijom kojoj nije potreban goto, iako ova konstrukcija možda nije tako jasna kao goto naredba. Na primer, sledeća je dozvoljena goto naredba:

```pascal
label  
  jumpto;  
...  
Jumpto :  
  Izjava;  
...  
Goto jumpto;  
... 
```

## 6.2 Strukturirane izjave

Strukturirane izjave mogu se razbiti na manje jednostavne izjave, koje treba izvršavati više puta, uslovno ili sekvencijalno:

- Uslovne izjave
- Ponavljajuće izjave
- With izjava
- Izjave izuzetaka

Uslovne izjave dolaze u dva oblika:

- Izjava case
- Izjava if

Ponavljajuće izjave dolaze u četiri oblika:

- Izjava for
- Izjava for in
- Izjava repeat
- Izjava while

Sledeći odeljci se bave svakom od ovih izjava.

### 6.2.1 Složene izjave

Složene izjave su grupa izjava, odvojenih tačkama-zarezima, koje su okružene ključnim rečima Begin i End. Poslednja izjava – pre ključne reči End – ne mora biti praćena tačkom-zarezom, iako je dozvoljen. Složena izjava je način grupisanja izjava, izvršavajući izjave sekvencijalno. Oni se tretiraju kao jedna izjava u slučajevima kada sintaksa Paskal jezika očekuje jednu izjavu, kao što je slučaj sa if...then...else izjavama.

#### 6.2.1.1 Case izjava

FreePaskal podržava `case` izjavu.

Konstante koje se pojavjuju u različitim delovima `case` moraju biti poznate u vreme kompajliraja i mogu biti sledećih tipova:

- tipovi nabrajaja,
- ordinalni tipovi (ukjučujući char) ili
- tipovi string.

Izjava `case` takođe mora biti ovog tipa, ili će doći do greške kompajlera. Sve konstante `case` moraju biti istog tipa.

Kompilator će izračunati izjavu `case`. Ako vrednost jedne od konstanti `case` odgovara vrednosti izraza, izvršava se izjava koja sledi posle ove konstante. Nakon toga, program nastavja nakon posledjeg end-a.

Ako nijedna od konstanti `case` ne odgovara vrednosti izraza, lista izjava nakon `else` ili drugačije ključne reči se izvršava. Ovo može biti prazna lista izjava. Ako nije prisutan nijedan deo `else` i nijedna konstanta slučaja ne odgovara vrednosti izraza, tok programa se nastavja nakon konačnog end.

Naredbe tipa `case` mogu biti složene naredbe (npr. blok Begin..End), ali za `else` slučaj je dozvojeno više izjava. Možete ih zatvoriti u dodatni blok Begin..End, ali to nije neophodno.

**Napomena**  
Za razliku od Turbo Paskala, duplirane oznake za slučajeve i veličine nisu dozvojene u FreePaskalu, tako da će sledeći kod generisati grešku prilikom kompajliraja:

```pascal
Var i : integer;  
...  
Case i of  
 3 : DoSomething;  
 1..5 : DoSomethingElse;  
end;
```

Kompilator će generisati grešku "Duplicate case label" prilikom kompajliraja, jer se broj 3 takođe (implicitno) pojavjuje u opsegu 1..5 . Ovo je slično Delfi sintaksi.

Sledeće su validne izjave o padežima:

```pascal
Case C of  
  'a' : WriteLn ('A pressed');  
  'b' : WriteLn ('B pressed');  
  'c' : WriteLn ('C pressed');  
else  
  WriteLn ('unknown letter pressed : ',C);  
end;
```

Ili

```pascal
Case C of  
  'a','e','i','o','u' : WriteLn ('vowel pressed');  
  'y' : WriteLn ('This one depends on the language');  
else  
  WriteLn ('Consonant pressed');  
end; 
```

```pascal
Case Number of  
  1..10   : WriteLn ('Small number');  
  11..100 : WriteLn ('Normal, medium number');  
else  
  WriteLn ('HUGE number');  
end;
```

Klauzula else može da sadrži više naredbi:

```pascal
Case Number of  
  1..10   : WriteLn ('Small number');  
  11..100 : WriteLn ('Normal, medium number');  
else  
  WriteLn ('HUGE number');  
  Writeln('How did we get this much ?');  
end; 
```

FreePaskal dozvojava upotrebu stringova kao oznaka za velika i mala slova, i u tom slučaju promenjiva case takođe mora biti string. Prilikom korišćenja string tipova, promenjiva case i različite oznake se upoređuju na način koji razlikuje velika i mala slova.

```pascal
Case lowercase(OS) of  
  'windows',  
  'dos'   : WriteLn ('Microsoft platform);  
  'macos',  
  'darwin' : Writeln('Apple platform');  
  'linux',  
  'freebsd',  
  'netbsd' : Writeln('Community platform');  
else  
  WriteLn ('Other platform');  
end;
```

Slučaj sa stringovima je ekvivalentan nizu `if then else` izjava, ne vrše se nikakve optimizacije.

Međutim, opsezi su dozvojeni i ekvivalentni su:

```pascal
if (value >= beginrange) and (value <= endrange) then  
  begin  
  end;
```

#### 6.2.1.2 Izjava If..then..else​

Izraz između ključnih reči `if` i `then` mora imati tip rezultata Boolean. Ako se izraz evaluira kao True, onda se izvršava naredba koja sledi ključnu reč `then`.
Ako se izraz evaluira kao False , onda se izvršava naredba koja sledi ključnu reč `else`, ako je prisutna.

Neke tačke koje treba napomenuti:

- Imajte na umu da će se Bulov izraz podrazumevano skraćivati, što znači da će se izvršavanje
  zaustaviti u trenutku kada je ishod poznat sa sigurnošću.
- Takođe, pre ključne reči `else`, nije dozvoljena tačka-zarez ( `;` ), ali sve izjave mogu biti
  složene izjave.
- U ugnežđenim `if.. then .. else` konstrukcijama, može doći do izvesne dvosmislenosti u pogledu
  toga koja `else` izjave se uparuje sa kojom `if` izjavom. Pravilo je da ključna reč `else` odgovara prvoj `if` ključnoj reči (pretraga unazad) koja već nije podudarana sa ključnom reči `else`.

Na primer:

```pascal
If exp1 Then  
  If exp2 then  
    Stat1  
else  
  stat2;
```

Uprkos svom izgledu, iskaz je sintaksički ekvivalentan

```pascal
If exp1 Then  
  begin  
  If exp2 then  
    Stat1  
  else  
    stat2  
  end;
```

a ne:

```pascal
{ NOT EQUIVALENT }  
If exp1 Then  
  begin  
  If exp2 then  
    Stat1  
  end  
else  
  stat2;
```

Ako je potrebna ova poslednja konstrukcija, ključne reči begin i end moraju biti prisutne. U slučaju sumnje, bolje je dodati ih.

Sledeća izjava je validna:

```pascal
If Today in [Monday..Friday] then  
  WriteLn ('Must work harder')  
else  
  WriteLn ('Take a day off.');
```

#### 6.2.1.3 Izjava For..to / downto..do

FreePaskal podržava konstrukciju `For` petlje. `For` petlja se koristi u slučaju da se nešto želi izračunati fiksni broj puta.

Ovde, `Statement` može biti složena naredba. Kada se naiđe na `For` naredbu, kontrolna promenljiva se inicijalizuje početnom vrednošću i upoređuje se sa konačnom vrednošću. Šta se dešava sledeće zavisi od toga da li se koristi to ili downto :

- U slučaju da se koristi `To`, ako je početna vrednost veća od konačne vrednosti, onda se `Statement` nikada neće izvršiti.
- U slučaju da se koristi `DownTo`, ako je početna vrednost manja od konačne vrednosti, onda se `Statement` nikada neće izvršiti.

Nakon ove provere, izvršava se naredba posle `Do`. Nakon izvršenja naredbe, kontrolna promenljiva se povećava ili smanjuje za 1, u zavisnosti od toga da li se koristi `To` ili `Downto`. Kontrolna promenljiva mora biti ordinalnog tipa, nijedan drugi tip se ne može koristiti kao brojač u petlji.

**Primedba**  

- FreePaskal uvek izračunava gornju granicu tačno jednom pre inicijalizacije promenljive brojača početnom vrednošću.
- Nije dozvoljeno menjati (tj. dodeljivati vrednost) vrednost promenljive petlje unutar same petlje.
- Vrednost promenljive petlje je nedefinisana nakon što se petlja završi ili ako se petlja uopšte ne izvrši. Međutim, ako je petlja prerano prekinuta `izuzetkom` ili izjavom `break` ili `goto`, promenljiva petlje zadržava vrednost koju je imala kada je petlja izbačena.
- Za ugnežđene procedure, promenljiva petlje mora biti lokalna promenljiva. Ako deklarišete promenljivu petlje izvan ugnežđene procedure gde se petlja nalazi, kompajler će se žaliti. Međutim, dozvoljeno je koristiti globalnu promenljivu u proceduri.
- Kompilator eksplicitno ne zabranjuje skakanje sa `goto` naredbom u blok for petlje, ali će to rezultirati nepredvidivim ponašanjem.

Sledeće petlje su validne:

```pascal
For Day := Monday to Friday do Work;  
For I := 100 downto 1 do  
  WriteLn ('Counting down : ',i);  
For I := 1 to 7*dwarfs do KissDwarf(i);
```

Sledeće će generisati grešku:

```pascal
For I:=0 to 100 do  
  begin  
  DoSomething;  
  I:=I*2;  
  end;
```

jer se promenljiva petlje I ne može dodeliti unutar petlje.

Sledeće će takođe generisati grešku:

```pascal
program test;  

{$ifdef fpc}  
{$mode delphi}  
{$h+}  
{$endif}  

procedure Proc;  
var  
  i: integer;  
  procedure Nested;  

  begin  
    for i := 1 to 2 do ;  
  end;  

begin  
end;  

begin  
end.
```

jer promenljiva I nije definisana u Nested-u i nije ni globalna promenljiva.

Ali sledeće će se kompajlirati:

```pascal
program test:

{$ifdef fpc}  
{$mode delphi}  
{$h+}  
{$endif}  

var  
  i: integer;  

procedure Nested;  

begin  
  for i := 1 to 2 do ;  
end;  

begin  
end
```

Ako je iskaz složeni iskaz, onda se sistemske rutine Break i Continue mogu koristiti za skok na kraj ili odmah posle kraja iskaza For . Imajte na umu da Break i Continue nisu rezervisane reči i stoga mogu biti preopterećene.

#### 6.2.1.4 Naredba For..in..do​

Od verzije 2.4.2, FreePaskal podržava konstrukciju petlje For..in . Petlja for..in se koristi u slučaju da neko želi da izračuna nešto fiksni broj puta sa nabrojivom promenljivom petlje.

Ovde, Statement može biti složena naredba. Nabrojiva stvar mora biti izraz koji se sastoji od fiksnog broja elemenata: promenljiva petlje će biti jednaka svakom od elemenata redom i naredba koja sledi ključnu reč do će biti izvršena.

Nabrojivi izraz može biti jedan od pet slučajeva:

- Identifikator tipa enumeracije. Petlja će tada biti preko svih elemenata tipa enumeracije.
  Kontrolna promenljiva mora biti tipa enumeracije.
- Set vrednosti. Petlja će tada biti preko svih elemenata u skupu, kontrolna promenljiva mora biti
  osnovnog tipa skupa.
- Vrednost niza. Petlja će se kretati preko svih elemenata u nizu, a kontrolna promenljiva mora biti
  istog tipa kao element u nizu. Kao poseban slučaj, string se smatra nizom znakova.
- Instanca nabrojive klase, objekta ili proširenog zapisa. Ovo je instanca bilo kog strukturiranog
  tipa koji podržava interfejse `IEnumerator` i `IEnumerable`. U ovom slučaju, tip kontrolne promenljive mora biti jednak tipu povratne vrednosti `IEnumerator.GetCurrent`.
- Bilo koji tip za koji je definisan operator enumeratora. Operator enumeratora mora vratiti
  strukturirani tip koji implementira interfejs `IEnumerator`. Tip tipa kontrolne promenljive mora biti jednak tipu tipa povratne vrednosti funkcije `GetCurrent` enumeratora.

Najjednostavniji slučaj petlje for..in je korišćenje nabrojanog tipa:

```personal
Type  
  TWeekDay = (monday, tuesday, wednesday, thursday, friday, saturday, sunday);  
 
Var  
  d : TWeekday;  
 
begin  
  for d in TWeekday do  
    writeln(d);  
end. 
```

Ovo će štampati sve dane u nedelji na ekranu.

Gore navedena konstrukcija for..in je ekvivalentna sledećoj for..to petlji:

```personal
Type  
  TWeekDay = (monday, tuesday, wednesday, thursday, friday, saturday, sunday);  
 
Var  
  d : TWeekday;  
 
begin  
  for d:=Low(TWeekday) to High(TWeekday) do  
    writeln(d);  
end.
```

Drugi slučaj petlje for..in je kada je nabrojivi izraz skup, i tada će se petlja izvršiti jednom za svaki element u skupu:

```pascal
 Type  
  TWeekDay = (monday, tuesday, wednesday, thursday, friday, saturday, sunday);  
 
Var  
  Week : set of TWeekDay  
       = [monday, tuesday, wednesday, thursday, friday];  
  d : TWeekday;  
 
begin  
  for d in Week do  
    writeln(d);  
end.
```

Ovo će ispisati imena dana u nedelji na ekranu. Imajte na umu da je promenljiva d istog tipa kao i osnovni tip skupa.

Gore navedena konstrukcija for..in je ekvivalentna sledećoj konstrukciji for..to :

```pascal
Type  
  TWeekDay = (monday, tuesday, wednesday, thursday, friday, saturday, sunday);  
 
Var  
  Week : set of TWeekDay  
       = [monday, tuesday, wednesday, thursday, friday];  
 
  d : TWeekday;  
 
begin  
  for d:=Low(TWeekday) to High(TWeekday) do  
    if d in Week then  
      writeln(d);  
end.
```

Treća mogućnost za petlju for..in je kada je nabrojivi izraz niz:

```pascal
var  
  a : Array[1..7] of string  
    = ('monday','tuesday','wednesday','thursday', 'friday','saturday','sunday');  
 
Var  
  S : String;  
 
begin  
  For s in a do  
    Writeln(s);  
end. 
```

Ovo će takođe ispisati sve dane u nedelji, što je ekvivalentno

```pascal
var  
  a : Array[1..7] of string  
    = ('monday','tuesday','wednesday','thursday', 'friday','saturday','sunday');  

Var  
  i : integer;  

begin  
  for i:=Low(a) to high(a) do  
    Writeln(a[i]);  
end.
```

Tip string je ekvivalentan nizu tipa char i stoga se string može koristiti u petlji for..in. Sledeći kod će ispisati sva slova abecede, svako slovo u posebnom redu:

```pascal
Var  
  c : char;  
 
begin  
 for c in 'abcdefghijklmnopqrstuvwxyz' do  
   writeln(c);  
end.

Note that multi-dimensional arrays are also supported:
uses  
  SysUtils;  
 
type  
  TTestStringArray = array[0..10] of String;  
 
Var  
  TwoD : array[0..3] of TTestStringArray;  
 
var  
  i,j : integer;  
  S : String;  
begin  
  for i:=0 to 3 do  
    for j:=0 to 10 do  
      TwoD[i,J]:=Format('%.2dx%.2d',[i,j]);  
  for S in twod do  
    Writeln(S);  
end. 
```

Ovo će preći preko svih dimenzija s leva na desno.

Četvrta mogućnost za petlju for..in je korišćenje klasa. Klasa može implementirati interfejs IEnumerable, koji je definisan na sledeći način:

```pascal
IEnumerable = interface(IInterface)  
  function GetEnumerator: IEnumerator;  
end;
```

Stvarni tip povratka GetEnumerator-a ne mora nužno biti IEnumerator interfejs, već može biti klasa koja implementira metode IEnumerator- a:

```pascal
IEnumerator = interface(IInterface)  
  function GetCurrent: TObject;  
  function MoveNext: Boolean;  
  procedure Reset;  
  property Current: TObject read GetCurrent;  
end;
```

Svojstvo Current i metod MoveNext moraju biti prisutni u klasi koju vraća metod GetEnumerator . Stvarni tip svojstva Current ne mora biti TObject. Kada naiđe na petlju for..in sa instancom klase kao operandom „in“, kompajler će proveriti svaki od sledećih uslova:

- Da li klasa u nabrojivom izrazu implementira metodu GetEnumerator
- Da li je rezultat funkcije GetEnumerator klasa sa sledećom metodom:
  - Funkcija MoveNext: Bulova vrednost

- Da li je rezultat funkcije GetEnumerator klasa sa sledećim svojstvom samo za čitanje:
  - Trenutna vrednost nekretnine: AType;

- Tip svojstva mora da se podudara sa tipom kontrolne promenljive petlje for..in .

Ni IEnumerator ni IEnumerable interfejsi ne moraju zapravo biti deklarisani od strane klase enumerable: kompajler će detektovati da li su ovi interfejsi prisutni koristeći gore navedene provere. Interfejsi su definisani samo za kompatibilnost sa Delphi-jem i ne koriste se interno (takođe bi bilo nemoguće nametnuti njihovu ispravnost).

Unit Class sadrži određeni broj klasa koje se mogu nabrojati:

- TFPLista
  Nabraja sve pokazivače u listi.
- TList
  Nabraja sve pokazivače u listi.
- Kolekcija
  Nabraja sve stavke u kolekciji.
- TStringList
  Nabraja sve stringove u listi.
- TKomponent
  Nabraja sve podređene komponente u vlasništvu komponente.

Dakle, sledeći kod će takođe ispisati sve dane u nedelji:

```pascal
{$mode objfpc}  
uses classes;  
 
Var  
  Days : TStrings;  
  D : String;  
 
begin  
  Days:=TStringList.Create;  
  try  
    Days.Add('Monday');  
    Days.Add('Tuesday');  
    Days.Add('Wednesday');  
    Days.Add('Thursday');  
    Days.Add('Friday');  
    Days.Add('Saturday');  
    Days.Add('Sunday');  
    For D in Days do  
      Writeln(D);  
  Finally  
    Days.Free;  
  end;  
end
```

Imajte na umu da kompajler sprovodi bezbednost tipova: deklarisanje D kao celog broja rezultiraće greškom kompajlera:

```sh
testsl.pp(20,9) Greška: Nekompatibilni tipovi: dobijeno je "AnsiString", očekivano je "LongInt"
```

Gore navedeni kod je ekvivalentan sledećem:

```pascal
{$mode objfpc}  
koristi klase;  
 
Var  
  Dani: TStrings;  
  D: String;  
  E: TStringsEnumerator;  
 
begin  
  Dani:=TStringList.Create;  
  try  
    Days.Add('Ponedeljak');  
    Dani.Add('Utorak');  
    Dani.Add('Sreda');  
    Dani.Add('Četvrtak');  
    Dani.Add('Petak');  
    Dani.Add('Subota');  
    Dani.Add('Nedelja');  
    E:=Dani.getEnumerator;  
    try  
      While E.MoveNext do  
        begin  
        D:=E.Current;  
        Writeln(D);  
        end;  
    Finally  
      E.Free;  
    end;  
  Finally  
    Days.Free;  
  end;  
end.
```

Oba programa će dati isti rezultat.

Peta i poslednja mogućnost korišćenja petlje for..in može se koristiti za nabrajanje skoro bilo kog tipa, koristeći operator enumeratora . Operator enumeratora mora vratiti klasu koja ima isti potpis kao i gore navedeni pristup IEnumerator . Sledeći kod će definisati enumerator za tip Integer :

```pascal
Type  
TEvenEnumerator = Class  
  FCurrent : Integer;  
  FMax : Integer;  
  Function MoveNext : Boolean;  
  Property Current : Integer Read FCurrent;  
end;  
 
Function TEvenEnumerator.MoveNext : Boolean;  
 
begin  
  FCurrent:=FCurrent+2;  
  Result:=FCurrent<=FMax;  
end;  
 
operator enumerator(i : integer) : TEvenEnumerator;  
 
begin  
  Result:=TEvenEnumerator.Create;  
  Result.FMax:=i;  
end;  
 
var  
  I : Integer;  
  m : Integer = 4;  
 
begin  
  For I in M do  
    Writeln(i);  
end. 
```

Petlja će ispisati sve parne brojeve različite od nule, manje ili jednake nabrojivoj vrednosti. (2 i 4 u slučaju primera).

Prilikom definisanja operatora enumeratora treba biti oprezan: kompajler će pronaći i koristiti prvi dostupni operator enumeratora za nabrojivi izraz. Za klase to takođe znači da se metoda GetEnumerator čak ni ne razmatra. Sledeći kod će definisati operator enumeratora koji izdvaja objekat iz liste nizova:

```pascal
{$mode objfpc}  
uses classes;  
 
Type  
  TDayObject = Class  
    DayOfWeek : Integer;  
    Constructor Create(ADayOfWeek : Integer);  
  end;  
 
  TObjectEnumerator = Class  
    FList : TStrings;  
    FIndex : Integer;  
    Function GetCurrent : TDayObject;  
    Function MoveNext: boolean;  
    Property Current : TDayObject Read GetCurrent;  
  end;  
 
Constructor TDayObject.Create(ADayOfWeek : Integer);  
 
begin  
  DayOfWeek:=ADayOfWeek;  
end;  
 
Function TObjectEnumerator.GetCurrent : TDayObject;  
begin  
  Result:=FList.Objects[Findex] as TDayObject;  
end;  
 
Function TObjectEnumerator.MoveNext: boolean;  
 
begin  
  Inc(FIndex);  
  Result:=(FIndex<FList.Count);  
end;  
 
operator enumerator (s : TStrings) : TObjectEnumerator;  
 
begin  
  Result:=TObjectEnumerator.Create;  
  Result.Flist:=S;  
  Result.FIndex:=-1;  
end;  
 
Var  
  Days : TStrings;  
  D : String;  
  O : TdayObject;  
 
begin  
  Days:=TStringList.Create;  
  try  
    Days.AddObject('Monday',TDayObject.Create(1));  
    Days.AddObject('Tuesday',TDayObject.Create(2));  
    Days.AddObject('Wednesday',TDayObject.Create(3));  
    Days.AddObject('Thursday',TDayObject.Create(4));  
    Days.AddObject('Friday',TDayObject.Create(5));  
    Days.AddObject('Saturday',TDayObject.Create(6));  
    Days.AddObject('Sunday',TDayObject.Create(7));  
    For O in Days do  
      Writeln(O.DayOfWeek);  
  Finally  
    Days.Free;  
  end;  
end.
```

Gornji kod će ispisati dan u nedelji za svaki dan u nedelji.

Ako klasa nije nabrojiva, kompajler će prijaviti grešku kada se na nju naiđe u petlji for...in.

Napomena Kao i kod petlje for..to , nije dozvoljeno menjati (tj. dodeljivati vrednost) vrednost kontrolne promenljive petlje unutar same petlje.

#### 6.2.1.5 Repeat..until izjava

Izjava repeat se koristi za izvršavanje naredbe dok se ne postigne određeni uslov. Izjava će biti izvršena najmanje jednom. Sintaksa prototipa naredbe Repeat..until je

Ovo će izvršiti naredbe između ponavljanja i do trenutka kada izraz procenjuje vrednost Tačno. Pošto se izraz procenjuje nakon izvršenja naredbi, oni se izvršavaju najmanje jednom.

Imajte na umu činjenicu da će logički izraz biti podrazumevano procenjen prečicom, što znači da će evaluacija biti zaustavljena u tački gde je rezultat poznat sa sigurnošću.

Sledeće su validne ponavljajuće izjave repeat:

```pascal
repeat
  WriteLn ('I =',i);  
  I := I+2;  
until I>100;  
 
repeat  
 X := X/2  
until x<10e-3;
```

Imajte na umu da poslednja izjava pre ključne reči `do` ne zahteva završnu tačku i zarez, ali je dozvoljena.

Sistemske rutine `break` i `continue` se mogu koristiti za prelazak na kraj ili odmah nakon završetka ponavljanja .. `do` naredbe. Imajte na umu da `break` i `continue` nisu rezervisane reči i stoga mogu biti preopterećene.

#### 6.2.1.6 While..do izjava

Naredba while se koristi za izvršavanje naredbe sve dok postoji određeni uslov. Za razliku od petlje ponavljanja, ovo može značiti da se naredba nikada ne izvršava.

Ovo će izvršiti naredbu sve dok Ekpression ima vrednost Tačno. Pošto se izraz procenjuje pre izvršenja naredbe, moguće je da se naredba uopšte ne izvrši. Izjava može biti složena izjava.

Imajte na umu činjenicu da će logički izraz Ekpression biti podrazumevano procenjen prečicom, što znači da će evaluacija biti zaustavljena u tački gde je rezultat poznat sa sigurnošću.

Sledeće su validne izjave while:

```pascal
I := I+2;  
while i<=100 do  
  begin  
  WriteLn ('I =',i);  
  I := I+2;  
  end;  
X := X/2;  
while x>=10e-3 do  
  X := X/2;
```

Oni odgovaraju petljama primera za naredbe ponavljanja.

Ako je izjava složena naredba, onda rezervisane reči Prekini i Nastavi mogu da se koriste za prelazak na kraj ili neposredno posle kraja izjave Vhile. Imajte na umu da Prekini i Nastavi nisu rezervisane reči i stoga mogu biti preopterećene.

#### 6.2.1.7 With izjava

Izjava `with` služi za pristup elementima zapisa ili objekta ili klase, bez potrebe da svaki put navedete ime elementa. Sintaksa za vith naredbu je

Referenca promenljive mora biti promenljiva tipa zapisa, objekta ili klase. U naredbi vith, svaka referenca promenljive ili referenca metode se proverava da bi se videlo da li je polje ili metod zapisa ili objekta ili klase. Ako je tako, onda se pristupa tom polju ili se poziva taj metod. S obzirom na deklaraciju:

```pascal
Type  
  Passenger = Record  
    Name : String[30];  
    Flight : String[10];  
  end;  
 
Var  
  TheCustomer : Passenger;
```

Sledeće izjave su potpuno ekvivalentne:

```pascal
TheCustomer.Name := 'Michael';  
TheCustomer.Flight := 'PS901';
```

i

```pascal
With TheCustomer do  
  begin  
  Name := 'Michael';  
  Flight := 'PS901';  
  end;
```

Izjava

```pascal
With A,B,C,D do Statement;
```

is equivalent to

```pascal
With A do  
 With B do  
  With C do  
   With D do Statement;
```

Ovo je takođe jasan primer činjenice da se promenljive pokušavaju od poslednje do prve, tj. kada kompajler naiđe na referencu promenljive, prvo će proveriti da li je to polje ili metod poslednje promenljive. Ako ne, onda će proveriti pretposlednje, i tako dalje. Sledeći primer to pokazuje;

```pascal
Program testw;  
Type AR = record  
      X,Y : Longint;  
     end;  
     PAR = ^Ar;  
 
Var S,T : Ar;  
begin  
  S.X := 1;S.Y := 1;  
  T.X := 2;T.Y := 2;  
  With S,T do  
    WriteLn (X,' ',Y);  
end.
```

Izlaz ovog programa je

```sh
2 2
```

Pokazujući tako da se Xs,Y u WriteLn naredbi podudaraju sa T promenljivom zapisa.

Napomena Kada koristite Vith naredbu sa pokazivačem ili klasom, nije dozvoljeno menjati pokazivač ili klasu u Vith bloku. Sa definicijama iz prethodnog primera, sledeće ilustruje o čemu se radi:

```pascal
Var p : PAR;  
 
begin  
  With P^ do  
   begin  
   // Do some operations  
   P:=OtherP;  
   X:=0.0;  // Wrong X will be used !!  
   end;
```

Razlog zašto se pokazivač ne može promeniti je taj što kompajler čuva adresu u privremenom registru. Promena pokazivača neće promeniti privremenu adresu. Isto važi i za časove.

#### 6.2.1.8 Izjave o izuzetku

Besplatni Pascal podržava izuzetke. Izuzeci pružaju zgodan način za programiranje grešaka i mehanizama za oporavak od grešaka, i usko su povezani sa klasama. Podrška za izuzetke je objašnjena u poglavlju 17, stranica 935.

[prev][f5] [content][f0] [next][f7]

[f0]: 00_sadrzaj.md
[f5]: 05_izrazi.md
[f7]: 07_procedure_i_funkcije.md
