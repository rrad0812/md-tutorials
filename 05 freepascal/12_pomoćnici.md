
# 12 Pomoćnici za klase, zapise i tipove

[prev][f11] [content][f0] [next][f13]

## 12.1 Definicija

Pomoćnici klasa, zapisa i tipa mogu se koristiti za dodavanje metoda postojećoj klasi, zapisu ili jednostavnom tipu, bez izvođenja klase ili ponovnog deklarisanja zapisa.

Za tip record ili jednostavan tip, pomoćnik tipa se ponaša kao da je record ili jednostavan tip klasa, a metode su deklarisane za nju. Unutar metoda, Self će se pozivati na vrednost tipa record ili jednostavnog tipa.

Za klase, efekat je kao umetanje metode u tabelu metoda klase. Ako je deklaracija pomoćnika u trenutnom opsegu koda, onda se metode i svojstva pomoćnika mogu koristiti kao da su deo deklaracije klase za klasu ili zapis koji pomoćnik proširuje.

Definicija pomoćne metode veoma liči na definiciju regularne klase. Ona jednostavno deklariše neke dodatne konstruktore, metode, svojstva i polja za klasu: klasa, zapis ili jednostavan tip za koji je pomoćna metoda proširenje je naznačen nakon ključne reči `for`.

Pošto se enumerator za klasu dobija putem regularne metode, pomoćnici klase se takođe mogu koristiti za prepisivanje enumeratora za klasu.

Kao što se može videti iz sintaksne dijagrame, moguće je kreirati potomke pomoćnih funkcija: pomoćne funkcije mogu formirati sopstvenu hijerarhiju, što omogućava prepisivanje metoda nadređene pomoćne funkcije. Takođe imaju specifikatore vidljivosti, baš kao zapisi i klase.

Kao i u instanci klase, identifikator Self u metodi pomoćne metode klase odnosi se na instancu klase (ne na instancu pomoćne metode). Za zapis, odnosi se na sam zapis.

Sledi jednostavan pomoćnik klase za klasu TObject , koji pruža alternativnu verziju standardne metode ToString .

```pascal
TObjectHelper = class helper for TObject  
  function AsString(const aFormat: String): String;  
end;  
 
function TObjectHelper.AsString(const aFormat: String): String;  
begin  
  Result := Format(aFormat, [ToString]);  
end;  
 
var  
  o: TObject;  
begin  
  Writeln(o.AsString('The object''s name is %s'));  
end.
```

**Napomena**  
Modifikator pomoćnika je samo modifikator odmah nakon ključnih reči klase ili zapisa . To znači da prvi član klase ili zapisa ne može biti nazvan pomoćnik . Član klase ili zapisa može se nazvati pomoćnik , ali ne može biti prvi, osim ako nije zaštićen znakom & , kao i za sve identifikatore koji se podudaraju sa ključnom reči.

**Napomena**  
Podrška za pomoćnike tipova zahteva korišćenje Delfi režima ili korišćenje prekidača za pomoćnike tipova u drugim režimima:
`{$MODESWITCH TYPEHELPERS}`

**Napomena**  
Podrška za pomoćnike za zapise zahteva upotrebu Delphi režima ili upotrebu prekidača za napredne zapise u drugim režimima:
{$MODESWITCH ADVANCEDRECORDS}

## 12.2 Ograničenja za pomoćnike u klasi

Nije moguće proširiti klasu bilo kojom metodom ili svojstvom. Postoje neka ograničenja u pogledu mogućnosti:

- Destruktori ili destruktori klasa nisu dozvoljeni.
- Konstruktori klasa nisu dozvoljeni.
- Pomoćnici klase ne mogu poticati od pomoćnika zapisa i ne mogu proširivati tipove zapisa.
- Definicije polja nisu dozvoljene. Nisu ni polja klase.
- Svojstva koja se odnose na polje nisu dozvoljena. Ovo je zapravo posledica prethodne stavke.
- Apstraktne metode nisu dozvoljene.
- Virtuelne metode klase ne mogu biti prepisane. Mogu se sakriti tako što im se da isto ime ili se mogu preopteretiti pomoću direktive overload .
- Za razliku od regularnih procedura ili metoda, specifikator preopterećenja mora se eksplicitno koristiti prilikom preopterećenja metoda iz klase u pomoćnoj klasi. Ako se preopterećenje ne koristi, metoda proširenog tipa je skrivena pomoćnom metodom (kao i kod regularnih klasa).

Sledeće menja prethodni primer preopterećenjem metode ToString :

TObjectHelper = class helper for TObject  
  function ToString(const aFormat: String): String; overload;  
end;  

function TObjectHelper.ToString(const aFormat: String): String;  
begin  
  Result := Format(aFormat, [ToString]);  
end;  

var  
  o: TObject;  
begin  
  Writeln(o.ToString('The object''s name is %s'));  
end.

## 12.3 Ograničenja za pomoćnike za zapise

Zapisi ne nude iste mogućnosti kao klase. To se odražava na mogućnosti pri kreiranju pomoćnika za zapise. U nastavku su navedena ograničenja za pomoćnike za zapise:

- Pomoćnik za zapise ne može se koristiti za proširivanje klase. Sledeće neće uspeti:

  ```pascal
  TTestHelper = record helper for TObject  
  end;
  ```

- Unutar deklaracije pomoćnika, metodama/poljima proširenog zapisa se ne može pristupiti, npr. u definiciji svojstva. Naravno, može im se pristupiti u implementaciji. To znači da se sledeće neće kompajlirati:

  ```pascal
  TTest = record  
    Test: Integer;  
  end;  
   
  TTestHelper = record helper for TTest  
    property AccessTest: Integer read Test;  
  end;
  ```

- Pomoćnici za zapise mogu pristupiti samo javnim poljima (u slučaju da se koristi prošireni zapis sa specifikatorima vidljivosti).
- Nasleđivanje pomoćnih funkcija zapisa je dozvoljeno samo u ObjFPC režimu; U Delphi režimu nije dozvoljeno.
- Pomoćnici zapisa mogu poticati samo od drugih pomoćnika zapisa, a ne od pomoćnika klase.
- Za razliku od pomoćnih funkcija klase, pomoćna funkcija potomstvenog zapisa mora proširiti isti tip zapisa.
- U Delfi režimu nije moguće pozvati metodu proširenog zapisa koristeći nasleđeni . To je moguće uraditi u ObjFPC režimu. Sledećem kodu je potreban ObjFPC režim za kompajliranje:

  ```pascal
  type  
    TTest = record  
      function Test(aRecurse: Boolean): Integer;  
    end;  
   
    TTestHelper = record helper for TTest  
      function Test(aRecurse: Boolean): Integer;  
    end;  
   
  function TTest.Test(aRecurse: Boolean): Integer;  
  begin  
    Result := 1;  
  end;  
   
  function TTestHelper.Test(aRecurse: Boolean): Integer;  
  begin  
    if aRecurse then  
      Result := inherited Test(False)  
    else  
      Result := 2;  
  end;
  ```

## 12.4 Razmatranja za (jednostavne) pomoćne funkcije tipova

Za jednostavne tipove, pravila su manje-više ista kao i za zapise, plus postoje neki dodatni zahtevi:

- Podrška za pomoćnike tipova mora biti aktivirana pomoću komande `modeswitch typehelpers`:

  ```pascal
  {$modeswitch typehelpers}
  ```

- Ovaj prekidač režima je podrazumevano omogućen samo u režimu Delphi i DelphiUnicode.
- U Delphi (i DelphiUnicode) režimu, radi strože kompatibilnosti sa Delphi-jem, moraju se koristiti pomoćnici za zapise umesto pomoćnika za tip.
- Režimi ObjFPC i MacPas koriste pomoćnik za tipove, ali se mora koristiti prekidač režima TypeHelpers.
- Sledeći tipovi nisu podržani:
  - Sve vrste datoteka ( tekst , datoteka od ... )
  - Proceduralne promenljive
  - Tipovi poput zapisa, klasa, Objective C klasa, C++ klasa, objekata i interfejsa su takođe zabranjeni, pomoćnik klase mora se koristiti za klase. To znači da će, na primer, sledeće biti neuspešno:

    ```pascal
    TTestHelper = type helper for TObject  
    end; 
    ```

    To naravno znači da su svi ostali jednostavni tipovi podržani.

- Pomoćnici tipova mogu implementirati konstruktore.
- Nasleđivanje pomoćnih funkcija zapisa je dozvoljeno samo u ObjFPC režimu; U Delphi režimu nije dozvoljeno.
- Pomoćnici tipova mogu poticati samo od drugih pomoćnika tipova, ne od pomoćnika klasa ili zapisa.
- Pomoćnik potomstvenog tipa mora proširiti isti tip.

Sledeće daje predstavu o mogućnostima:

```pascal
{$mode objfpc}  
{$modeswitch typehelpers}  
 
type  
 TLongIntHelper = type helper for LongInt  
   constructor create(AValue: LongInt);  
   class procedure Test; static;  
   procedure DoPrint;  
 end;  
 
constructor TLongIntHelper.create(AValue: LongInt);  
 
begin  
  Self:=Avalue;  
  DoPrint;  
end;  
 
class procedure TLongIntHelper.Test;  
 
begin  
   Writeln('Test');  
end;  
 
procedure TLongIntHelper.DoPrint;  
 
begin  
   Writeln('Value :',Self);  
end;  
 
var  
  i: LongInt;  
begin  
  I:=123;  
  i.Test;  
  $12345678.Test;  
  LongInt.Test;  
  I:=123;  
  i.DoPrint;  
  $12345678.DoPrint;  
end.
```

## 12.5 Napomena o opsegu i životnom veku za pomoćne funkcije za zapise i tipove

Za klase, životni vek instance klase eksplicitno upravlja programer. Stoga je jasno šta parametar Self znači i kada je validan.

Zapisi i drugi jednostavni tipovi se alociraju na steku, što znači da izlaze iz oblasti važenja kada se funkcija, procedura ili metoda u kojoj se koriste završi.

U kombinaciji sa činjenicom da su pomoćne metode kompatibilne po tipu sa metodama klase i stoga se mogu koristiti kao obrađivači događaja, ovo može dovesti do iznenađujućih situacija: Pokazivač podataka u pomoćnoj metodi je postavljen na adresu promenljive.

Razmotrite sledeći primer:

```pascal
{$mode objfpc}  
{$modeswitch typehelpers}  
uses  
  Classes;  
 
type  
  TInt32Helper = type helper for Int32  
    procedure Foo(Sender: TObject);  
  end;  
 
procedure TInt32Helper.Foo(Sender: TObject);  
begin  
  Writeln(Self);  
end;  
 
var  
  i: Int32 = 10;  
  m: TNotifyEvent;  
begin  
  m := @i.Foo;  
  WriteLn('Data : ',PtrUInt(TMethod(m).Data));  
  m(nil);  
end.
```

Ovo će ispisati nešto poput (stvarna vrednost za podatke može se razlikovati):

```sh
Data : 6848896  
10
```

Promenljiva i je i dalje u opsegu važenja kada se pozove m.

Ali promena koda u

```pascal
{$mode objfpc}  
{$modeswitch typehelpers}  
uses  
  Classes;  
 
type  
  TInt32Helper = type helper for Int32  
    procedure Foo(Sender: TObject);  
  end;  
 
procedure TInt32Helper.Foo(Sender: TObject);  
begin  
  Writeln(Self);  
end;  
 
Function GetHandler  :TNotifyEvent;  
 
var  
  i: Int32 = 10;  
 
begin  
  Result:=@i.foo;  
end;  
 
Var  
  m: TNotifyEvent;  
begin  
  m := GetHandler;  
  WriteLn(PtrUInt(TMethod(m).Data));  
  m(nil);  
end. 
```

Izlaz će biti:

```sh
140727246638796  
0
```

Stvarni izlaz će zavisiti od arhitekture, ali poenta je u tome što i više nije u opsegu važenja, što izlaz njegove vrednosti čini besmislenim, a moguće je čak i da će dovesti do kršenja pristupa i padova programa.

## 12.6 Nasleđivanje

Kao što je navedeno u prethodnom odeljku, moguće je kreirati potomke pomoćnih klasa. Pošto se može koristiti samo poslednja pomoćna klasa u trenutnom opsegu, neophodno je napraviti potomak pomoćne klase iz druge ako se moraju koristiti metode obe pomoćne klase. Više o ovome u sledećem odeljku.

Potomak pomoćne klase može proširiti klasu koja se razlikuje od njenog roditelja. Sledeći primer je validna pomoćna klasa za TMyObject :

```pascal
TObjectHelper = class helper for TObject  
  procedure SomeMethod;  
end;  
 
TMyObject = class(TObject)  
end;  
 
TMyObjectHelper = class helper(TObjectHelper) for TMyObject  
  procedure SomeOtherMethod;  
end;
```

Klasa `TMyObjectHelper` proširuje klasu `TObjectHelper`, ali ne proširuje klasu `TObject`, već samo klasu TMyObject .

Pošto zapisi ne poznaju nasleđivanje, očigledno je da potomci pomoćnika zapisa mogu samo proširiti isti zapis.

**Napomena**  
Radi maksimalne kompatibilnosti sa Delphi-jem, nemoguće je kreirati potomke pomoćnih zapisa u Delphi režimu.

## 12.7 Upotreba

Kada se pomoćna klasa definiše, njene metode se mogu koristiti kad god je pomoćna klasa u opsegu važenja. To znači da ako je definisana u posebnoj jedinici, onda bi ta jedinica trebalo da bude u klauzuli uses gde god se koriste metode pomoćne klase.

Razmotrite sledeću jedinicu:

```pascal
{$mode objfpc}  
{$h+}  
unit oha;  
 
interface  
 
Type  
  TObjectHelper = class helper for TObject  
    function AsString(const aFormat: String): String;  
  end;  
 
implementation  
 
uses sysutils;  
 
function TObjectHelper.AsString(const aFormat: String): String;  
 
begin  
  Result := Format(aFormat, [ToString]);  
end;  
 
end. 
```

Zatim će se sledeće kompajlirati:

```pascal
Program Example113;  
 
uses oha;  
 
{ Program to demonstrate the class helper scope. }  
 
Var  
  o : TObject;  
 
begin  
  O:=TObject.Create;  
  Writeln(O.AsString('O as a string : %s'));  
end.
```

Ali, ako se kreira druga jedinica (ohb):

```pascal
{$mode objfpc}  
{$h+}  
unit ohb;  
 
interface  
 
Type  
  TAObjectHelper = class helper for TObject  
    function MemoryLocation: String;  
  end;  
 
implementation  
 
uses sysutils;  
 
function TAObjectHelper.MemoryLocation: String;  
 
begin  
  Result := format('%p',[pointer(Self)]);  
end;  
 
end.
```

I dodaje se posle prve jedinice u klauzuli uses:

```pascal
Program Example113;  
 
uses oha,ohb;  
 
{ Program to demonstrate the class helper scope. }  
 
Var  
  o : TObject;  
 
begin  
  O:=TObject.Create;  
  Writeln(O.AsString('O as a string : %s'));  
  Writeln(O.MemoryLocation);  
end. 
```

Tada će se kompajler žaliti da ne poznaje metod „AsString“. To je zato što kompajler prestaje da traži pomoćnike klase čim naiđe na prvi pomoćnik klase. Pošto jedinica ohb dolazi poslednja u klauzuli uses, kompajler će koristiti samo TOObjectHelper kao pomoćnik klase.

Rešenje je ponovna implementacija jedinice ohb:

```pascal
{$mode objfpc}  
{$h+}  
unit ohc;  
 
interface  
 
uses oha;  
 
Type  
  TAObjectHelper = class helper(TObjectHelper) for TObject  
    function MemoryLocation: String;  
  end;  
 
implementation  
 
uses sysutils;  
 
function TAObjectHelper.MemoryLocation: String;  
 
begin  
  Result := format('%p',[pointer(Self)]);  
end;  
 
end.
```

I nakon zamene jedinice ohb sa ohc , primer programa će se kompajlirati i funkcionisati kako se očekuje.

Imajte na umu da nije dovoljno uključiti jedinicu sa pomoćnom klasom jednom u projektu; jedinica mora biti uključena kad god je pomoćna klasa potrebna.

[prev][f11] [content][f0] [next][f13]

[f0]: 00_sadrzaj.md
[f11]: 11_prošireni_zapisi.md
[f13]: 13_interfejsi.md
