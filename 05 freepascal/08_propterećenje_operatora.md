# 8 Preopterećenje operatora

[prev][f7] [content][f0] [next][f9]

## 8.1 Uvod

Fri Paskal podržava preopterećenje operatora. To znači da je moguće definisati dejstvo nekih operatora na samodefinisanim tipovima i time omogućiti upotrebu ovih tipova u matematičkim izrazima.

Pored toga, preopterećenje operatora može se koristiti za neke posebne radnje koje, strogo govoreći, nisu operatori u matematičkom smislu te reči. Ovo važi za procedure Inc i Dec (koje se mogu koristiti samo kao procedura) i za implicitno i eksplicitno dodeljivanje ( := i pretvaranje tipova u tip). Operator nabrajanja (za definisanje nabrojivog niza za tip) je takođe deo ove klase operatora.

Definisanje akcije operatora je veoma slično definiciji funkcije ili procedure, samo što postoje neka ograničenja u pogledu mogućih definicija, kao što će biti prikazano u nastavku.

Preopterećenje operatora je, u suštini, moćan alat za notaciju; ali nije ni više od toga, jer se isti rezultati mogu dobiti regularnim pozivima funkcija. Prilikom korišćenja preopterećenja operatora, važno je imati na umu da neka implicitna pravila mogu proizvesti neočekivane rezultate. Ovo će biti naznačeno.

Postoje 2 načina za deklarisanje preopterećenja operatora. Prvi način je originalni način kako je implementiran u Free Pascal-u i najmoćniji je. O njemu se govori u ovom poglavlju. Postoji i drugi način, koji se može implementirati samo za napredne (ili proširene) zapise, o njemu se govori u poglavlju o proširenim zapisima.

## 8.2 Deklaracije operatora

Definisanje akcije operatora je veoma slično definisanju funkcije.

Lista parametara za operator poređenja ili aritmetički operator mora uvek da sadrži dva parametra, sa izuzetkom unarnog minusa ili plusa, gde je potreban samo jedan parametar. Tip rezultata operatora poređenja mora biti Bulova vrednost .

Korisnički definisani jednostavni tipovi mogu se koristiti za operatore, kao i za zapise i nizove. Postoje neka ograničenja u vezi sa preopterećenjem operatora:

- Ovaj način deklarisanja operatora je podržan samo u ObjFPC i FPC režimima.
- Nije moguće definisati operatore na klasama.
- Operatori + i - ne mogu biti definisani na nabrojanim tipovima.
- Kada je prekidač režima ArrayOperators aktivan:
  `{$modeSwitchArrayOperators}`
  onda operator `+` ne može biti preopterećen na dinamičkim nizovima, jer ga interno obrađuje kompajler.

**Napomena**  
Prilikom kompajliranja u Objfpc režimu, identifikator rezultata može biti izostavljen. Rezultatu se tada može pristupiti preko standardnog simbola rezultata .

Ako se identifikator rezultata izostavi, a kompajler nije u jednom od ovih režima, doći će do sintaksičke greške.

Blok izjava sadrži neophodne izjave za određivanje rezultata operacije. Može da sadrži proizvoljno velike delove koda; izvršava se kad god se operacija naiđe u nekom izrazu. Rezultat bloka izjava mora uvek biti definisan; uslove greške ne proverava kompajler, a kod mora da vodi računa o svim mogućim slučajevima, izbacujući grešku tokom izvršavanja ako se naiđe na neki uslov greške.

U nastavku će biti ispitane tri vrste definicija operatora. Na primer, u ovom poglavlju će se koristiti sledeći tip za definisanje preopterećenih operatora na:

```pascal
type  
  complex = record  
    re : real;  
    im : real;  
  end;
```

Ovaj tip će biti korišćen u svim primerima.

Izvorni kodovi biblioteke za vreme izvršavanja sadrže dve jedinice koje intenzivno koriste preopterećenje operatora:

- ucomplex
  Ova jedinica sadrži kompletan račun za kompleksne brojeve.
- matrix
  Ova jedinica sadrži kompletan račun za matrice.

## 8.3 Operatori dodele

Operator dodele definiše akciju dodeljivanja jednog tipa promenljive drugom. Tip rezultata mora da odgovara tipu promenljive levo od izraza dodele, pojedinačni parametar operatora dodele mora imati isti tip kao izraz desno od operatora dodele.

Ovaj sistem se može koristiti za deklarisanje novog tipa i definisanje dodele za taj tip. Na primer, da biste mogli da dodelite novodefinisani tip „Složeni“

```pascal
Var  
  C,Z : Complex; // New type complex  

begin  
  Z:=C;  // assignments between complex types.  
end;
```

Sledeći operator dodeljivanja bi morao da bude definisan:

```pascal
Operator := (C : Complex) z : complex;
```

Da biste mogli da dodelite pravi tip složenom tipu na sledeći način:

```pascal
var  
  R : real;  
  C : complex;  
 
begin  
  C:=R;  
end;
```

mora biti definisan sledeći operator dodele:

```pascal
Operator := (r : real) z : complex;
```

Kao što se vidi iz ovog iskaza, on definiše akciju operatora := pri čemu je desno realan izraz, a levo složen izraz.

Primer implementacije ovoga može biti sledeći:

```pascal
operator := (r : real) z : complex;  
 
begin  
  z.re:=r;  
  z.im:=0.0;  
end;
```

Kao što se može videti u primeru, identifikator rezultata (z u ovom slučaju) se koristi za čuvanje rezultata dodele. Prilikom kompajliranja u Delphi režimu ili ObjFPC režimu, upotreba specijalnog identifikatora Result je takođe dozvoljena i može se zameniti za z, tako da bi gore navedeno bilo ekvivalentno

```pascal
operator := (r : real) z : complex;  
 
begin  
  Result.re:=r;  
  Result.im:=0.0;  
end;
```

Operator dodele se takođe koristi za pretvaranje tipova iz jednog tipa u drugi. Kompajler će razmotriti sve preopterećene operatore dodeljivanja dok ne pronađe onaj koji odgovara tipovima izraza za levu i desnu ruku. Ako takav operator nije pronađen, daje se greška „nepodudaranja tipa“.

Napomena Operator dodele nije komutativan; kompajler nikada neće promeniti ulogu dva argumenta. Drugim rečima, s obzirom na gornju definiciju operatora dodele, sledeće nije moguće:

```pascal
var  
  R : real;  
  C : complex;  
 
begin  
  R:=C;  
end;
```

Ako je moguće obrnuto dodeljivanje, onda i za to mora biti definisan operator dodele. (Ovo nije tako za realne i kompleksne brojeve.)

**Napomena**  
Operator dodele se takođe koristi u implicitnim konverzijama tipa. Ovo može imati neželjene efekte. Razmotrite sledeće definicije:

```pascal
operator := (r : real) z : complex;  
function exp(c : complex) : complex;
```

Zatim će sledeći zadatak dati neslaganje tipa:

```pascal
Var  
  r1,r2 : real;  
 
begin  
  r1:=exp(r2);  
end;
```

Do neusklađenosti dolazi zato što će kompajler naići na definiciju ekp funkcije sa složenim argumentom. On implicitno pretvara r2 u kompleks, tako da može da koristi gornju funkciju eksp. Rezultat ove funkcije je kompleks koji se ne može dodeliti r1, pa će kompajler dati grešku „nepodudaranja tipa“. Kompajler neće dalje tražiti drugi ekp koji ima ispravne argumente.

Moguće je izbeći ovaj konkretan problem navođenjem

```pascal
r1:=system.exp(r2);
```

Kada radi eksplicitno prebacivanje tipa, kompajler će pokušati implicitnu konverziju ako je prisutan operator dodele. To znači da

```pascal
Var  
  R1 : T1;  
  R2 : T2;  
 
begin  
  R2:=T2(R1);
```

Rukovaće operater

```pascal
Operator := (aRight: T1) Res: T2;
```

Međutim, eksplicitni operator se može definisati, a onda će se koristiti umesto njega kada kompajler naiđe na primenu tipa.

Obrnuto nije tačno: u redovnom dodeljivanju, kompajler neće uzeti u obzir eksplicitne operatore dodele.

S obzirom na sledeće definicije:

```pascal
uses  
  sysutils;  
 
type  
  TTest1 = record  
    f: LongInt;  
  end;  
  TTest2 = record  
    f: String;  
  end;  
  TTest3 = record  
    f: Boolean;  
  end;
```

Moguće je kreirati operatore dodeljivanja:

```pascal
operator := (aRight: TTest1) Res: TTest2;  
begin  
  Writeln('Implicit TTest1 => TTest2');  
  Res.f := IntToStr(aRight.f);  
end;  
 
operator := (aRight: TTest1) Res: TTest3;  
begin  
  Writeln('Implicit TTest1 => TTest3');  
  Res.f := aRight.f <> 0;  
end;
```

Ali takođe se mogu definisati operatori za prebacivanje tipa:

```pascal
operator Explicit(aRight: TTest2) Res: TTest1;  
begin  
  Writeln('Explicit TTest2 => TTest1');  
  Res.f := StrToIntDef(aRight.f, 0);  
end;  
 
operator Explicit(aRight: TTest1) Res: TTest3;  
begin  
  Writeln('Explicit TTest1 => TTest3');  
  Res.f := aRight.f <> 0;  
end;
```

Dakle, sledeći kod

```pascal
var  
  t1: TTest1;  
  t2: TTest2;  
  t3: TTest3;  
begin  
  t1.f := 42;  
  // Implicit  
  t2 := t1;  
  // theoretically explicit, but implicit op will be used,  
  // because no explicit operator is defined  
  t2 := TTest2(t1);  
  // the following would not compile,  
  // no assignment operator defined (explicit one won't be used here)  
  //t1 := t2;  
  // Explicit  
  t1 := TTest1(t2);  
  // first explicit (TTest2 => TTest1) then implicit (TTest1 => TTest3)  
  t3 := TTest1(t2);  
  // Implicit  
  t3 := t1;  
  // explicit  
  t3 := TTest3(t1);  
end.
```

će proizvesti sledeći izlaz:

```pascal
Implicit TTest1 => TTest2  
Implicit TTest1 => TTest2  
Explicit TTest2 => TTest1  
Explicit TTest2 => TTest1  
Implicit TTest1 => TTest3  
Implicit TTest1 => TTest3  
Explicit TTest1 => TTest3
```

## 8.4 Aritmetički operatori

Aritmetički operatori definišu radnju binarnog operatora. Moguće operacije su:

- **množenje**  
  Da bi se pomnožila dva tipa, operator množenja * mora biti preopterećen.
- **deljenje**  
  Da bi se dva tipa podelila, operator deljenja / mora biti preopterećen.
- **sabiranje**  
  Da biste sabrali dva tipa, operator sabiranja + mora biti preopterećen. Imajte na umu da se `+` ne može koristiti za dinamičke nizove, jer je ovo interna operacija kompajlera za spajanje nizova (pogledajte takođe prekidač režima `ARRAYOPERATORS`).
- **oduzimanje**  
  Da bi se oduzela dva tipa, operator oduzimanja - mora biti preopterećen.
- **stepenovanje**  
  Da bi se stepenovala dva tipa, operator stepenovanja ** mora biti preopterećen.
- **unarni minus**  
  koristi se za uzimanje negativne strane argumenta koji sledi.
- simetrična razlika
  Da bi se izračunala simetrična razlika dve strukture, operator >< mora biti preopterećen.

Definicija aritmetičkog operatora uzima dva parametra, osim unarnog minusa, kome je potreban samo jedan parametar. Prvi parametar mora biti tipa koji se nalazi levo od operatora, drugi parametar mora biti tipa koji se nalazi desno od aritmetičkog operatora. Tip rezultata mora se podudarati sa tipom koji se dobija nakon aritmetičke operacije.

Da biste kompajlirali izraz kao

```pascal
var  
  R : real;  
  C,Z : complex;  

begin  
  C:=R*Z;  
end;
```

potrebna je definicija operatora množenja kao:

```pascal
Operator * (r : real; z1 : complex) z : complex;  
 
begin  
  z.re := z1.re * r;  
  z.im := z1.im * r;  
end;
```

Kao što se može videti, prvi operator je realan broj, a drugi je kompleksan. Tip rezultata je kompleksan.

Množenje i sabiranje realnih i kompleksnih brojeva su komutativne operacije. Međutim, kompajler nema pojma o ovoj činjenici, tako da čak i ako je definisano množenje između realnog i kompleksnog broja, kompajler neće koristiti tu definiciju kada naiđe na kompleksni i realni broj (tim redosledom). Potrebno je definisati obe operacije.

Dakle, s obzirom na gornju definiciju množenja, kompajler neće prihvatiti sledeću izjavu:

```pascal
var  
  R : real;  
  C,Z : complex;  
 
begin  
  C:=Z*R;  
end;
```

Pošto tipovi Z i R ne odgovaraju tipovima u definiciji operatora.

Razlog za ovo ponašanje je taj što je moguće da množenje nije uvek komutativno. Npr. množenje matrice (n,m) sa matricom (m,n) rezultiraće matricom (n,n) , dok je množenje matrice (m,n) sa matricom (n,m) matrica (m,m) , koja ne mora biti ista u svim slučajevima.

## 8.5 Operator poređenja

Operator poređenja može biti preopterećen da bi se uporedila dva različita tipa ili da bi se uporedila dva jednaka tipa koja nisu osnovni tipovi. Ako operandi nisu jednostavni tipovi, tip rezultata operatora poređenja ne mora uvek biti bulova vrednost, ali se onda ne mogu koristiti u naredbama if , repeat ili while .

Operatori poređenja koji mogu biti preopterećeni su:

- **jednako**  
  (=) Da se ​​utvrdi da li su dve promenljive jednake.
- **nejednako**  
  ( <> ) Da bi se utvrdilo da li su dve promenljive različite.
- **manje od**  
  ( < ) Da bi se utvrdilo da li je jedna promenljiva manja od druge.
- **veće od**  
  ( > ) Da bi se utvrdilo da li je jedna promenljiva veća od druge.
- **veće ili jednako**  
  ( > =) Da bi se utvrdilo da li je jedna promenljiva veća ili jednaka drugoj.
- **manje ili jednako**  
  ( < =) Da bi se utvrdilo da li je jedna promenljiva veća ili jednaka drugoj.

Ako ne postoji poseban operator za nejednako sa ( <> ), onda, da bi izračunao izraz koji sadrži operator nejednako sa , kompajler koristi operator jednako sa ( = ) i negira rezultat. Suprotno nije tačno: ako ne postoji operator „jednako sa“ već operator „nejednako sa“, kompajler ga neće koristiti za izračunavanje izraza koji sadrži operator jednako sa ( = ).

Na primer, sledeći operator omogućava upoređivanje dva kompleksna broja:

```pascal
operator = (z1, z2 : complex) b : boolean;
```

Gore navedena definicija omogućava poređenja sledećeg oblika:

```pascal
Var  
  C1,C2 : Complex;  
 
begin  
  If C1=C2 then  
    Writeln('C1 and C2 are equal');  
end;
```

Definicija operatora poređenja zahteva dva parametra, sa tipovima koje operator treba da uporedi. Ovde takođe, kompajler ne primenjuje komutativnost: ako su dva tipa različita, onda je potrebno definisati dva operatora poređenja.

U slučaju kompleksnih brojeva, na primer, potrebno je definisati 2 poređenja: jedno sa kompleksnim tipom prvo, a jedno sa realnim tipom prvo.

S obzirom na definicije

```pascal
operator = (z1 : complex;r : real) b : boolean;  
operator = (r : real; z1 : complex) b : boolean; 
```

moguća su sledeća dva poređenja:

```pascal
Var  
  R,S : Real;  
  C : Complex;  
 
begin  
  If (C=R) or (S=C) then  
   Writeln ('Ok');  
end;
```

Imajte na umu da je redosled realnog i kompleksnog tipa u dva poređenja obrnut.

Sledeći primer pokazuje da tip rezultata ne mora biti logička vrednost:

```pascal
Type  
  TMyRec = record a,b : integer; end;  
 
operator = (x,y : TMyRec) r : string;  
 
begin  
  if (x.a=y.a) and (x.b=y.b) then  
    R:='equal'  
  else  
    R:='differ';  
end;  
 
var  
  x,y : TMyRec;  
 
begin  
  x.a:=1;  
  y.a:=1;  
  Writeln(x=y);  
  x.a:=2;  
  y.a:=3;  
  Writeln(x=y);  
end. 
```

Kada se izvrši, ovaj primer će ispisati

```sh
equal  
differ
```

očigledno, izjava kao

```pascal
if (x=y) then  
  writeln('Equal');
```

neće se kompajlirati, jer if naredba zahteva proveru bulove vrednosti:

```sh
Error: Incompatible types: got "ShortString" expected "Boolean" 
```

## 8.6 In operator

Od verzije 2.6 Free Pascal-a, operator In takođe može biti preopterećen. Prvi argument operatora in mora biti operand levo od ključne reči in . Sledeća koda preopterećuje operator in za zapise:

```pascal
{$mode objfpc}{$H+}  
 
type  
  TMyRec = record A: Integer end;  
 
operator in (const A: Integer; const B: TMyRec): boolean;  
begin  
  Result := A = B.A;  
end;  
 
var  
  R: TMyRec;  
begin  
  R.A := 10;  
  Writeln(1 in R); // false  
  Writeln(10 in R); // true  
end. 
```

Operator in takođe može biti preopterećen za druge tipove osim ordinalnih tipova, kao u sledećem primeru:

```pascal
{$mode objfpc}{$H+}  
 
type  
  TMyRec = record A: Integer end;  
 
operator in (const A: TMyRec; const B: TMyRec): boolean;  
begin  
  Result := A.A = B.A;  
end;  
 
var  
  S,R: TMyRec;  
begin  
  R.A := 10;  
  S.A:=1;  
  Writeln(S in R); // false  
  Writeln(R in R); // true  
end. 
```

## 8.7 Logički operatori

Logički operatori `and`, `or`, `xor` i `not` mogu biti preopterećeni. Ovi operatori se obično koriste na jednostavnim tipovima na dva različita načina:

- Kao bulovski operatori, u kom slučaju je rezultat bulovska vrednost (osim za `not` )
- Kao bitski operatori, u kom slučaju je rezultat ordinalni tip.

Prilikom preopterećenja ovih operatora, tip rezultata nije ograničen. To znači da možete definisati operatore kao bulove logičke operatore:

```pascal
Type  
  Rec = record  
    a,b : Boolean;  
  end;  
 
Operator and (r1,r2 : Rec) z : boolean;  
 
begin  
  z:=(R1.a and R2.a) or (R1.b and r2.b);  
end;  
 
Operator or (r1,r2 : Rec) z : Boolean;  
 
begin  
  z:=(R1.a or R2.a) and (R1.b or r2.b)  
end;  
 
Operator xor (r1,r2 : Rec) z : Boolean;  
 
begin  
  z:=(R1.a xor R2.a) and (R1.b xor r2.b)  
end;  
 
Operator not (r1 : Rec) z : rec;  
 
begin  
  z.A:=not R1.a;  
  z.B:=not R1.b;  
end;  
 
 
var  
r1,r2 : Rec;  
 
begin  
  Writeln(r1 and r2);  
  Writeln(r1 or r2);  
  Writeln(r1 xor r2);  
  Writeln((not r1).a);  
end. 
```

Ali je takođe moguće imati različite tipove povratka:

```pascal
Operator and (r1,r2 : Rec) z : string;  
 
begin  
  Str(Ord((R1.a and R2.a) or (R1.b and r2.b)),Z);  
end;
```

Kompilator će uvek proveriti tip povratka da bi odredio konačni tip izraza, a dodele će biti proverene radi bezbednosti tipa.

## 8.8 Operatori povećanja/smanjenja

Procedure `Inc` i `Dec` su – radi optimizacije – intrinzični elementi kompajlera. Mogu se zameniti na uobičajeni način deklarisanjem procedure:

```pascal
procedure Inc(var s : String);  
begin  
  Inc(S,'.');  
end;  
 
procedure Inc(var s : String; T : String);  
begin  
 S:=S+T;  
end; 
```

Ali pošto su intrinzični elementi kompajlera, mogu se implementirati i kao operatori za prilagođene tipove:

```pascal
operator Inc (s,T : String) : z : string;  
 
begin  
  Z:=S+T;  
end; 
```

## 8.9 Operator enumerator

Operator enumeratora može se koristiti za definisanje enumeratora za bilo koji tip. Mora vratiti klasu, objekat ili prošireni zapis koji ima isti potpis kao i interfejs IEnumerator. Imajte na umu da kompajler ne prepoznaje ovaj operator u režimu sintakse Delphi.

Sledeći kod će definisati enumerator za tip Integer:

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

[prev][f7] [content][f0] [next][f9]

[f0]: 00_sadrzaj.md
[f7]: 07_procedure_i_funkcije.md
[f9]: 09_objekti.md
