
# 14 Generici

[prev][f13] [content][f0] [next][f15]

## 14.1 Uvod

Generici su šabloni za generisanje drugih tipova. To mogu biti klase, objekti, interfejsi, pa čak i funkcije, nizovi, zapisi. To je koncept koji dolazi iz C++, gde je duboko integrisan u jezik. Od verzije 2.2, Free Pascal takođe zvanično ima podršku za šablone ili generike. Oni su implementirani kao vrsta makroa koji se čuva u jediničnim datotekama koje kompajler generiše i koji se reprodukuje čim se generička klasa specijalizuje.

Kreiranje i korišćenje generičkih lekova je proces u dve faze.

- Definicija generičkog tipa je definisana kao novi tip: ovo je šablon koda, makro koji kompajler može ponovo da reprodukuje u kasnijoj fazi.
- Generički tip je specijalizovan: ovo definiše drugi tip, koji je specifična implementacija generičkog tipa: kompajler reprodukuje makro koji je sačuvan kada je generički tip definisan.

Postoji nekoliko jedinica distribuiranih sa besplatnim Paskalom koje implementiraju generičke kontejnere i klase. Na primer, jedinica fgl .

## 14.2  Definicija generičkog tipa

Generička definicija tipa je veoma slična definiciji tipa, s tim što sadrži listu rezervisanih mesta za tipove.

Za klase, objekte, proceduralne tipove i proširene zapise, deklaraciju generičkog tipa treba da prati implementacija tipa. To je isto kao i normalna implementacija klase sa jednim izuzetkom, naime, svaki identifikator sa istim imenom kao jedan od identifikatora šablona mora biti identifikator tipa.

Dakle, deklaracija generičkog tipa je veoma slična normalnoj deklaraciji tipa, osim što je prisutan još uvek nepoznati tip. Nepoznati tipovi su navedeni u listi čuvara mesta i nepoznati su dok se klasa ne specijalizuje.

Sledeća je validna generička definicija klase:

```pascal
Type  
  generic TList<_T>=class(TObject)  
  Public  
    type  
       TCompareFunc = function(const Item1, Item2: _T): Integer;  
    var  
      data : _T;  
    procedure Add(item: _T);  
    procedure Sort(compare: TCompareFunc);  
  end;
```

Ovu klasu bi mogla da prati sledeća implementacija:

```pascal
procedure TList.Add(item: _T);  
begin  
  data:=item;  
end;  
 
procedure TList.Sort(compare: TCompareFunc);  
begin  
  if compare(data, 20) <= 0 then  
    halt(1);  
end;
```

Postoje neke značajne stvari u vezi sa ovom deklaracijom i implementacijom:

- Postoji samo jedan privremeni znak `_T`. Biće zamenjen identifikatorom tipa kada je generička klasa specijalizovana. Identifikator `_T` se ne sme koristiti ni za šta drugo osim za privremeni znak tipa. To znači da sledeće ne bi bilo validno:
  
  ```pascal
  procedure TList.Sort(compare: TCompareFunc);  

  Var  
    _t : integer;  
   
  begin  
    // do something.  
  end;
  ```

- Lokalni blok tipa sadrži jedan tip TCompareFunc . Treba napomenuti da stvarni tip još nije poznat unutar definicije generičke klase: definicija sadrži referencu na privremeni element _T . Sve ostale reference identifikatora moraju biti poznate kada je generička klasa definisana, a ne kada je generička klasa specijalizovana.
  
- Blok lokalnih promenljivih je ekvivalentan sledećem:
  
  ```pascal
  generic TList<_T>=class(TObject)  
    Public  
      type  
         TCompareFunc = function(const Item1, Item2: _T): Integer;  
    Public  
      data : _T;  
      procedure Add(item: _T);  
      procedure Sort(compare: TCompareFunc);  
    end;
  ```

- Imajte na umu da imena parametara tipa u deklaraciji i implementaciji moraju biti ista.

Ne mogu se definisati samo generičke klase, već i drugi tipovi:

```pascal
{$mode objfpc}  
{$INTERFACES CORBA}  

type  

  generic PlanarCoordinate<t> = record  
    x,y : t;  
  end;  
 
  TScreenCoordinate = specialize PLanarCoordinate<word>;  
  TDiscreteCoordinate = specialize PlanarCoordinate<integer>;  
  TRealCoordinate = specialize PlanarCoordinate<extended>;  
 
  generic TDistanceFunction<t> = function (x,y : t) : Extended of object;  
 
  TScreenDistance = specialize TDistanceFunction<word>;  
  TDiscreteDistance = specialize TDistanceFunction<integer>;  
  TRealDistance = specialize TDistanceFunction<Extended>;  
 
  generic TArray<t> = array of t;  
 
  TMyIntegerArray = specialize TArray<integer>;  
 
  generic IList<_T> = Interface  
    Function GetItem(AIndex : Integer) : _T;  
    Procedure SetItem(AIndex : Integer; AValue : _T);  
    Function GetCount : Integer;  
    Property Items [AIndex : Integer] : _T Read GetItem Write SetItem;  
    Property Count : Integer Read GetCount;  
  end;  
 
  generic TList<_T>=class(TObject, specialize IList<_T>)  
  public type  
    TCompareFunc = function(const Item1, Item2: _T): Integer;  
    Function GetItem(AIndex : Integer) : _T;  
    Procedure SetItem(AIndex : Integer; AValue : _T);  
    Function GetCount : Integer;  
  Public  
     data : _T;  
     procedure Add(item: _T);  
     procedure Sort(compare: TCompareFunc);  
  end;  
 
  generic TPointSet<t> = array of specialize PlanarCoordinate<t>;  
 
  TScreenPointSet = specialize TPointSet<word>;  
  TDiscretePointSet =  specialize TPointSet<integer>;  
  TRealPointSet = specialize TPointSet<extended>;
```

**Napomena**  
Reč o vidljivosti: tipovi šablona T ili _T su dostupni kao striktno privatni tipovi. To znači da tipovi nisu dostupni u potomačkim klasama, osim ako nisu dostupni putem nekog zaštićenog ili privatnog mehanizma, kao u sledećem primeru:

```pascal
  generic TList<_T>=class(TObject)  
  public type  
    TItemType = _T;  
  end;
```

## 14.3 Specijalizacija generičkih tipova

Kada se generički tip definiše, može se koristiti za generisanje drugih tipova: ovo je kao ponavljanje definicije tipova, sa šablonskim rezervisanim mestima popunjenim stvarnim definicijama tipova.

Ovo se može uraditi u bilo kom bloku definicije tipa.

S obzirom na deklaraciju TList-a u prethodnom odeljku, sledeće bi bila validna definicija tipa:

```pascal
Type  
  TPointerList = specialize TList<Pointer>;  
  TIntegerList = specialize TList<Integer>;
```

Od verzije 3.0 Fri Paskala, ključna reč specialize se takođe može koristiti u deklaraciji promenljive:

```pascal
Var  
  P : specialize TList<Pointer>;
```

Ključna reč specialized je deo specialized tipa, tako da kada se koriste potpuno kvalifikovana imena, ključna reč specialized mora biti posle imena jedinice i imena matičnog tipa.

Tip u izrazu specialized mora biti poznat, osim u drugoj generičkoj definiciji tipa. S obzirom na dve generičke definicije klase:

```pascal
type  
  Generic TMyFirstType<T1> = Class(TMyObject);  
  Generic TMySecondType<T2> = Class(TMyOtherObject);
```

Onda sledeća specijalizacija nije validna:

```pascal
type  
  TMySpecialType = specialize TMySecondType<TMyFirstType>;
```

jer je tip TMyFirstType generički tip i stoga nije u potpunosti definisan. Kompilator će se žaliti:

```sh
Error: Generics cannot be used as parameters when specializing generics
```

Međutim, sledeće je dozvoljeno:

```pascal
type  
  TA = specialize TMyFirstType<Atype>;  
  TB = specialize TMySecondType<TA>;
```

jer je TA već u potpunosti definisana kada je TB specijalizovana.

Međutim, ključna reč specialize može se koristiti u drugoj generičkoj definiciji tipa kao što je prikazano u gornjem primeru:

```pascal
  generic TList<_T>=class(TObject, specialize IList<_T>)
```

i

```pascal
  generic TPointSet<t> = array of specialize PlanarCoordinate<t>;
```

U ovim definicijama, specijalizacija se vrši samo kada je sam generički tip specijalizovan, i tada su tipovi poznati.

**Napomena**  
Nije moguće napraviti unapred definiciju klase koja je specijalizacija generičke klase, tj. sledeće se neće kompajlirati:

```pascal
  TMyClass = Class;  
 
  // Other declarations  
 
  TMyClass = specialize TList<T>;
```

## 14.4 Ograničenje generičkih tipova

Lista šablona tipova može imati dodatne specifikatore za tipove. Ovo je posebno korisno za tipove objekata: ako tip šablona mora poticati iz određene klase, onda se to može navesti u listi šablona:

```pascal
{$mode objfpc}{$h+}  

uses sysutils, classes;  
 
Type  
  generic TList<_T : TComponent> = class(TObject)  
  public  
    Type TCompareFunc = function(const Item1, Item2: _T): Integer;  
  Public  
    data : _T;  
    procedure Add(item: _T);  
    procedure Sort(compare: TCompareFunc);  
end;
```

S obzirom na gornju definiciju, sledeće će se kompajlirati:

```pascal
TPersistentList = specialize TList<TComponent>;
```

Ali ovo se neće kompajlirati

```pascal
TPersistentList = specialize TList<TPersistent>;
```

Kompilator će vratiti grešku:

```sh
Error: Incompatible types: got "TPersistent" expected "TComponent"
```

Više tipova se može grupisati zajedno:

```pascal
Type  
  generic TList<Key1,Key2 : TComponent; Value1 : TObject> = class(TObject)
```

Pored toga, moguće je navesti više od jednog identifikatora tipa za ograničenja tipa klase i interfejsa. Ako je navedena klasa, onda tip koji se koristi za šablon mora biti jednak ili potomak naznačenog tipa:

```pascal
Type  
  generic TList<T: TComponent, IEnumerable> = class(TObject)
```

Klasa koja se koristi za specijalizaciju T mora biti poreklo od TComponent i mora implementirati IEnumerable .

Ako je naveden interfejs, onda tip šablona mora da implementira barem ovaj interfejs, ali može biti i potomak ovog interfejsa:

```pascal
Type  
  generic TGenList<T: IEnumerable> = class(TObject)  
 
  IMyEnum = Interface (IEnumerable)  
    Procedure DoMy;  
  end;  
 
  TList = specialize TGenList<IMyEnum>;  
  TSomeList = Specialize TGenList<TList>;
```

Može se navesti više interfejsa, u tom slučaju tip klase mora implementirati sve navedene interfejse: Moguće je kombinovati jedno ime klase sa nekoliko imena interfejsa.

Ako nisu na snazi ograničenja tipa, kompajler će pretpostaviti da tipovi šablona nisu kompatibilni sa dodelom.

Ovo je posebno važno kada generička klasa sadrži preopterećene metode. Data je sledeća deklaracija generičkog tipa:

```pascal
type  
  generic TTest<T1, T2> = class  
    procedure Test(aArg: LongInt);  
    procedure Test(aArg: T1);  
    procedure Test(aArg: T2);  
  end;
```

Specijalizacija gore navedenog će se kompajlirati ako su T1 i T2 dva različita tipa i nijedan nije takođe LongInt . Sledeće će se kompajlirati:

```pascal
T1 = specialize TTest<String, TObject>;

Ali sledeća dva se neće kompajlirati:

T2 = specialize TTest<String, String>;
```

ili

```pascal
T2 = specialize TTest<String, Longint>;
```

## 14.5  Kompatibilnost sa Delfijem

Podrška za generike u FPC-u je implementirana donekle drugačije nego u Delphi-ju. U ovom odeljku su istaknute glavne razlike.

### 14.5.1  Sintaksni elementi

Sintaksa prikazana na sintaksnim dijagramima je sintaksa potrebna u ObjFPC režimu kompajlera. Međutim, u Delphi režimu, ključne reči specialize i generic ne smeju se koristiti, kao što je prikazano u sledećem primeru:

```pascal
Type  
  TTest<T> = Class(TObject)  
  Private  
    FObj : T;  
  Public  
    Property Obj : T Read FObj Write FObj;  
  end;  
 
  TIntegerTest = TTest<Integer>;
```

Za razliku od režima Objfpc, imena tipova šablona moraju se ponoviti u definicijama metoda.

```pascal
Type  
  TTest<T> = Class(TObject)  
  Private  
    FObj : T;  
  Public  
    Procedure DoIt;  
    Property Obj : T Read FObj Write FObj;  
  end;  
 
Procedure TTest<T>.DoIt;  
 
begin  
end;
```

Ovaj zahtev je direktno povezan sa mogućnošću preopterećenja generičkog tipa pomenutom u sledećem odeljku.

### 14.5.2 Ograničenja tipa zapisa

U Delfi režimu, ograničenja tipa zapisa će takođe dozvoliti upotrebu jednostavnih tipova:

```pascal
Type  
  TList<_T : record> = class(TObject)  
  public  
    Type TCompareFunc = function(const Item1, Item2: _T): Integer;  
  Public  
    data : _T;  
    procedure Add(item: _T);  
    procedure Sort(compare: TCompareFunc);  
 end;  
 
TIntList = TList<Integer>;
```

Ograničenje se primenjuje prilikom specijalizacije tipa. To znači da režim aktivan prilikom specijalizacije tipa određuje da li se jednostavan tip može koristiti ili ne: ako je ograničenje na snimanje kompajlirano korišćenjem ObjFPC režima, kod napisan u Delphi režimu može ga specijalizovati jednostavnim tipom u svakom slučaju.

Na primer:

```pascal
unit tg;  
 
interface  
 
{$mode objfpc}  
 
Type  
  generic TList<_T : record> = class(TObject)  
  public  
    Type TCompareFunc = function(const Item1, Item2: _T): Integer;  
  Public  
    data : _T;  
    procedure Add(item: _T);  
    procedure Sort(compare: TCompareFunc);  
 end;  
 
implementation  
 
generic procedure TList<_T>.Add(item: _T);  
 
begin  
end;  
 
generic  procedure TList<_T>.Sort(compare: TCompareFunc);  
 
begin  
end;  
 
end.
```

može se koristiti u { $ MODE Delphi } za:

```pascal
{$mode delphi}  
uses tg;  
 
Type  
  TIntList = TList<Integer>;  
begin  
end. 
```

### 14.5.3  Preopterećenja tipa

Delfi režim dozvoljava preopterećenja generičkih tipova. To znači da je moguće deklarisati istu generičku klasu sa različitim listama tipova šablona. Stoga su moguće sledeće deklaracije:

```pascal
Type  
  TTest<T> = Class(TObject)  
  Private  
    FObj : T;  
  Public  
    Property Obj : T Read FObj Write FObj;  
  end;  
 
  TTest<T,S> = Class(TObject)  
  Private  
    FObj1 : T;  
    FObj2 : S;  
  Public  
    Property Obj1 : T Read FObj1 Write FObj1;  
    Property Obj2 : S Read FObj2 Write FObj2;  
  end; 
```

### 14.5.4 Razmatranja imenskog prostora

U Delfi režimu, generički tipovi ne ometaju imenski prostor za promenljive, što znači da će se sledeće takođe kompajlirati:

```pascal
Type  
  TTest<T> = Class(TObject)  
  Private  
    FObj : T;  
  Public  
    Property Obj : T Read FObj Write FObj;  
  end;  
 
Var  
  TTest : Integer;
```

Međutim, ovo ne funkcioniše za konstante i funkcije.

## 14.6 Kompatibilnost tipova

Kad god je generička klasa specijalizovana, to rezultira novim, posebnim tipom. Ovi tipovi su kompatibilni sa dodeljivanjem ako se koriste isti tipovi šablona.

Uzmite sledeću generičku definiciju:

```pascal
{$mode objfpc}  
unit ua;  
 
interface  
 
type  
  Generic TMyClass<T> = Class(TObject)  
    Procedure DoSomething(A : T; B : INteger);  
  end;  
 
Implementation  
 
Procedure TMyClass.DoSomething(A : T; B : Integer);  
 
begin  
  // Some code.  
end;  
 
end.
```

I sledeće specijalizacije:

```pascal
{$mode objfpc}  
unit ub;  
 
interface  
 
uses ua;  
 
Type  
  TB = Specialize TMyClass<string>;  
 
implementation  
 
end.
```

Sledeće specijalizacije su identične, ali se pojavljuju u drugoj jedinici:

```pascal
{$mode objfpc}  
unit uc;  
 
interface  
 
uses ua;  
 
Type  
  TB = Specialize TMyClass<string>;  
 
implementation  
 
end.
```

Sledeće će se zatim kompajlirati:

```pascal
{$mode objfpc}  
unit ud;  
 
interface  
 
uses ua,ub,uc;  
 
Var  
  B : ub.TB;  
  C : uc.TB;  
 
implementation  
 
begin  
  B:=C;  
end.
```

Tipovi ub.TB i uc.TB su kompatibilni sa dodelom. Nije bitno što su tipovi definisani u različitim jedinicama. Mogu biti definisani i u istoj jedinici:

```pascal
{$mode objfpc}  
unit ue;  
 
interface  
 
uses ua;  
 
Type  
  TB = Specialize TMyClass<string>;  
  TC = Specialize TMyClass<string>;  
 
 
Var  
  B : TB;  
  C : TC;  
 
implementation  
 
begin  
  B:=C;  
end.
```

Svaka specijalizacija generičke klase sa istim tipovima kao parametri je novi, poseban tip, ali ovi tipovi su kompatibilni sa dodelom ako su tipovi šablona koji se koriste za njihovu specijalizaciju jednaki.

Ako je specijalizacija sa različitim tipom šablona, tipovi su i dalje različiti, ali više nisu kompatibilni sa dodeljivanjem. tj. sledeće se neće kompajlirati:

```pascal
{$mode objfpc}  
unit uf;  
 
interface  
 
uses ua;  
 
Type  
  TB = Specialize TMyClass<string>;  
  TC = Specialize TMyClass<integer>;  
 
 
Var  
  B : TB;  
  C : TC;  
 
implementation  
 
begin  
  B:=C;  
end.

Prilikom kompajliranja, doći će do greške:

Error: Incompatible types: got "TMyClass<System.LongInt>"  
                           expected "TMyClass<System.ShortString>" 
```

## 14.7 Korišćenje podrazumevane intrinzične funkcije

Prilikom pisanja generičkih rutina, ponekad se mora inicijalizovati promenljiva čiji tip nije poznat tokom deklaracije generičke promenljive. Ovde do izražaja dolazi i podrazumevana intrinzična promena (odeljak 4.5 , strana 243 ). Data je sledeća generička deklaracija:

```pascal
type  
  generic TTest<T> = class  
    procedure Test;  
  end;

Sledeći kod će pravilno inicijalizovati promenljivu myt tokom specijalizacije:

procedure TTest.Test;  
var  
  myt: T;  
begin  
  // will have the correct Default if class is specialized  
  myt := Default(T);  
end;
```

## 14.8 Reč o opsegu

Treba naglasiti da svi identifikatori osim čuvara mesta šablona treba da budu poznati kada se generička klasa deklariše. Istovremeno, ništa se ne može pretpostaviti o tipu šablona (osim ako se na njega ne postavi ograničenje).

Ovo funkcioniše na nekoliko načina.

U odsustvu ograničenja tipa, generički kod ne može da pravi pretpostavke o tipu šablona T. Razmotrite sledeću jedinicu:

```pascal
unit ts;  
 
interface  
{$modeswitch advancedrecords}  
 
type  
  PListEl = ^TListEl;  
  TListEl = packed record  
    Prev, Next: PListEl;  
  end;  
 
implementation  
 
type  
  generic LstEnumerator<T> = record  
  private  
    lst, lst_save: T;  
  public  
    constructor Create(const Value: T);  
    function MoveNext: boolean;  
  end;  
 
function LstEnumerator.MoveNext: boolean;  
begin  
  Result:=lst <> nil;  
  if Result then  
    lst:=lst^.next;  
end;  
 
constructor LstEnumerator.Create(const Value: T);  
begin  
  lst:= Value;  
  lst_save := nil;  
end;  
 
Type  
 TMyListEnum = specialize LstEnumerator<TListEl>;  
 
end.
```

Kompilator će izbaciti grešku jer kada kompajlira generičku definiciju, ne može da proveri da

```pascal
   lst:=lst^.next;
```

je tačno. lst je tipa T , ali kompajler (još) ne zna šta je T , i stoga ne može znati da ima polje next .

Ovaj problem se može rešiti ograničenjima tipa:

```pascal
unit ts;  
{$mode delphi}  
interface  
 
type  
  TListEl = class  
    Prev, Next: TListEl;  
  end;  
 
  TMyRecord1 = Class(TListEl)  
    MyField : String;  
  end;  
 
  TMyRecord2 = Class(TListEl)  
    MyInteger : Integer;  
  end;  
 
implementation  
 
type  
  TLstEnumerator<T : TListEl> = class  
  private  
    lst, lst_save: T;  
  public  
    constructor Create(const Value: T);  
    function MoveNext: boolean;  
  end;  
 
function TLstEnumerator<T>.MoveNext: boolean;  
begin  
  Result:=lst <> T(nil);  
  if Result then  
    lst:=T(lst.next);  
end;  
 
constructor TLstEnumerator<t>.Create(const Value: T);  
begin  
  lst:= Value;  
  lst_save := T(nil);  
end;  
 
Type  
 TMyRecord1Enum = TLstEnumerator<TMyRecord1>;  
 TMyRecord2Enum = TLstEnumerator<TMyRecord2>;
```

Ovde, kompajler zna da je lst barem tipa TListEl, i stoga sadrži članove Prev i Next .

Pored tipa šablona, svi ostali tipovi koji se koriste u generičkoj deklaraciji moraju biti poznati. To znači da mora postojati identifikator tipa sa istim imenom. Sledeća jedinica će proizvesti grešku:

```pascal
{$mode objfpc}  
unit myunit;  
 
interface  
 
type  
  Generic TMyClass<T> = Class(TObject)  
    Procedure DoSomething(A : T; B : TSomeType);  
  end;  
 
Type  
  TSomeType = Integer;  
  TSomeTypeClass = specialize TMyClass<TSomeType>;  
 
Implementation  
 
Procedure TMyClass.DoSomething(A : T; B : TSomeType);  
 
begin  
  // Some code.  
end;  
 
end.
```

Gornji kod će rezultirati greškom, jer tip TSomeType nije poznat kada se deklaracija analizira:

```sh
home: >fpc myunit.pp  
myunit.pp(8,47) Error: Identifier not found "TSomeType"  
myunit.pp(11,1) Fatal: There were 1 errors compiling module, stopping
```

Drugi način na koji je ovo vidljivo je sledeći. Pretpostavimo jedinicu

```pascal
{$mode objfpc}  
unit mya;  
 
interface  
 
type  
  Generic TMyClass<T> = Class(TObject)  
    Procedure DoSomething(A : T);  
  end;  
 
 
Implementation  
 
Procedure DoLocalThings;  
 
begin  
  Writeln('mya.DoLocalThings');  
end;  
 
 
Procedure TMyClass.DoSomething(A : T);  
 
begin  
  DoLocalThings;  
end;  
 
end.
```

Kompilator neće dozvoliti kompajliranje ove jedinice, jer funkcija DoLocalThings neće biti vidljiva kada je generički tip specijalizovan:

```sh
Error: Global Generic template references static symtable
```

Sada, ako se jedinica modifikuje i funkcija DoLocalThings se premesti u sekciju interfejsa, jedinica će se kompajlirati. Kada se ovaj generički kod koristi u programu:

```pascal
{$mode objfpc}  
program myb;  
 
uses mya;  
 
procedure DoLocalThings;  
 
begin  
  Writeln('myb.DoLocalThings');  
end;  
 
Type  
  TB = specialize TMyClass<Integer>;  
 
Var  
  B : TB;  
 
begin  
  B:=TB.Create;  
  B.DoSomething(1);  
end.
```

Uprkos činjenici da generički tipovi deluju kao makro koji se reprodukuje tokom specijalizacije, referenca na DoLocalThings se rešava kada se definiše TMyClass , a ne kada se definiše TB. To znači da je izlaz programa:

```sh
home: >fpc -S2 myb.pp  
home: >myb  
mya.DoLocalThings
```

Ovakvo ponašanje je propisano bezbednošću i nužnošću:

- Programer koji se specijalizuje za neku klasu nema načina da zna koje se lokalne procedure koriste, tako da je ne može slučajno „prepisati“.
- Programer koji se specijalizuje za neku klasu nema načina da zna koje se lokalne procedure koriste, pa je ne može ni implementirati, jer ne zna parametre.
- Ako se implementacione procedure koriste kao u gornjem primeru, na njih se ne može pozivati izvan jedinice. Mogle bi biti u potpuno drugoj jedinici, a programer nema načina da zna da li treba da ih uključi pre nego što specijalizuje svoju klasu.

## 14.9  Preopterećenje operatora i generički tipovi

Preopterećenje operatora (poglavlje 15 , strana 846 ) i generičke klase su usko povezani. Zamislite generičku klasu koja ima sledeću definiciju:

```pascal
{$mode objfpc}  
unit mya;  
 
interface  
 
type  
  Generic TMyClass<T> = Class(TObject)  
    Function Add(A,B : T) : T;  
  end;  
 
Implementation  
 
Function TMyClass.Add(A,B : T) : T;  
 
begin  
  Result:=A+B;  
end;  
 
end.
```

Kada kompajler ponovo reprodukuje generički makro, dodavanje mora biti moguće. Za specijalizaciju poput ove:

```pascal
TMyIntegerClass = specialize TMyClass<integer>;
```

Ovo nije problem, jer bi metoda Dodaj postala:

```pascal
Procedure TMyIntegerClass.Add(A,B : Integer) : Integer;  
 
begin  
  Result:=A+B;  
end;
```

Kompilator zna kako da sabere dva cela broja, tako da će se ovaj kod kompajlirati bez problema. Ali sledeći kod:

```pascal
Type  
  TComplex = record  
   Re,Im : Double;  
  end;  
 
Type  
  TMyIntegerClass = specialize TMyClass<TComplex>;
```

Neće se kompajlirati, osim ako nije definisano dodavanje dva TComplex tipa. To se može uraditi korišćenjem operatora zapisa:

```pascal
{$modeswitch advancedrecords}  
uses mya;  
 
Type  
  TComplex = record  
     Re,Im : Double;  
     class operator +(a,b : TComplex) : TComplex;  
  end;  
 
class operator TComplex.+ (a,b : TComplex) : TComplex;  
 
begin  
  Result.re:=A.re+B.re;  
  Result.im:=A.im+B.im;  
end;  
 
Type  
  TMyComplexClass = specialize TMyClass<TComplex>;  
 
begin  
  // Code here  
end.
```

Trenutno, zbog ograničenja implementacije, neće raditi korišćenje globalnog operatora, tj. sledeće još uvek ne radi:

```pascal
uses mya;  
 
Type  
  TComplex = record  
     Re,Im : Double;  
  end;  
 
operator + (a,b : TComplex) : TComplex;  
 
begin  
  Result.re:=A.re+B.re;  
  Result.im:=A.im+B.im;  
end;  
 
Type  
  TMyComplexClass = specialize TMyClass<TComplex>;  
 
begin  
  // Code here  
end.
```

Očekuje se podrška za ovu konstrukciju u budućoj verziji FreePaskala.

[prev][f13] [content][f0] [next][f15]

[f0]: 00_sadrzaj.md
[f13]: 13_interfejsi.md
[f15]: 15_programi_uniti_blokovi.md
