
# 10 Klase

[prev][f9] [content][f0] [next][f11]

U Delfiju mogućnosti OOP-a uglavnom se vrte oko koncepta “Classes”. Klasa može biti razmatrana kao pokazivač na objekat, ili pokazivač na zapis, sa pridruženim metodama.

Razlike izmedju objekata i klasa  su uglavanom da je objekat alociran na steku, kao što bi pravi zapis trebao biti, i da se klase uvek alociraju na heapu. U sledećem primeru:

```pascal
Var  
  A: TSomeObject; // an Object  
  B: TSomeClass;  // a Class
```

Glavana razlika je da će varijabla A zauzeti prostor na seku u zavisnosti veličine objekta(TSomeObject). Varijabla B, će uvek zauzeti na steku veličinu pointera. Aktuelni podaci klase su na heap.

Druga razlika je: klasa uvek mora inicijalizovana kroz svoj konstruktor, dok za objekat to nije obaveza. Pozivanje konstruktora alocira neophodnu memoriju na heap za podatke instance klase.

**Napomena**  
U ranijim verzijama Free Pascala bilo je obavezno, da bi koristili klase, da postavimo `objpas` unit u `uses` klauzuli unita ili programa. To više od verzije 0.99.12 nije potrebno. Od te verzije unit će biti učitana automatski kada `-Mobjfpc` or `-Mdelphi` opcije su specificirani, ili njihove odgovarajuće direktive se koriste:

```pascal
{$mode objfpc}  
{$mode delphi}
```

U stvari, kompajler će dati upozorenje ako pronađe `objpas` unit u `uses` klauzuli.

## 10.1 Definicija klase

Prototip deklaracije klase je kao što sledi:

**Napomena**  
U MacPas modu, Object ključna reč je zamenjen sa class ključnom reči zbog kompatabilnosti sa drugim pascal kompajlerima raspoloživim na Macu.

U deklaraciji klase, više vidljivih blokova kao potreba mogu biti korišćeni: različiti blokovi mogu biti ponovljenim i ne postoji specijalni poredak u kume se mogu pojavljivati.

Metode su noramlne funkcijeske ili proceduralne deklaracije. Kao što se može primetiti, deklaracija klase je najčešće identična deklaraciji objekta. Realna razlika izmedju objekta i klase je u načinu kako se kreiraju.

Vidljivost različitih sekcija je kao što sledi:

- `Private`  
  Svim poljima i metodama koje su u `private` bloku može se pristupiti u modulu (tj. unitu) koji sadrži definiciju klase. Njima se može pristupiti iz klasnih metoda ili izvan njih (tj. iz medoda drugih klasa unita.)
- `Strict Private`
  Svim poljima i metodama koje su u `strict private` bloku, može se pristupiti jedino iz metoda same klase. Druge klase unita, ili nasleđene klase, čak u istom unitu, ne mogu pristupiti `strict private` članovima.
- `Protected`
  Članovima `Protected` sekcije su takođe pristupačni drugim klasama implementiranim u istiom unitu kao i izvedenim klasama implementirane u ovom ili drugim modulima.
- `Strict Protected`
  `Strict protected` članovi su jedino vidljivi izvedenim klasama, ne za druge klase u istom ili drugim unitima.
- `Public`
  sekcije su uvek pristupne.
- `Published`
  Iz perspektive jezika, ovo je isto kao `Public` section, ali kompajler generiše informacije tipa koje su potrebne za automatski streaming te klase, ako je kompajler u {$M+} stanju. Polja definisana u `Published` sekciju moraju biti tipa klase. Niz properties ne može biti u `Published` sekciju.

Klase mogu sadržavati `class methods`: to su rutine koje ne zahtevaju instancu. Self identifikator je valida u tim metodama ali pokazuje na class pointer ( VMT ).

**Napomena**  
Kao i kod funkcija i tipova pokazivača, ponekad je potrebna definicija klase unapred. Definicija klase unapred je jednostavno ime klase, sa ključnom rečju Class, kao u sledećem primeru:

```pascal
Type  
  TClassB = Class;  
  TClassA = Class  
    B : TClassB;  
  end;  
 
  TClassB = Class  
   A : TClassA;  
  end;
```

Kada koristite definiciju klase unapred, klasa mora biti definisana u istom unitu i u istom odeljku (interfejs/implementacija). Ne mora nužno biti definisan u odeljku istog tipa.

Referentni tipovi klasa se koriste za kreiranje instanci određene klase, koja još nije poznata u vreme kompajliranja, ali je specificirana u vreme izvršavanja. U suštini, promenljiva referentnog tipa klase sadrži pokazivač na definiciju navedene klase. Ovo se može koristiti za konstruisanje instance klase koja odgovara definiciji ili za proveru nasleđivanja. Sledeći primer pokazuje kako to funkcioniše:

```pascal
Type  
  TComponentClass = Class of TComponent;  

Function CreateComponent(AClass: TComponentClass; 
                             AOwner: TComponent): TComponent;  

begin  
  // ...  
  Result:=AClass.Create(AOwner);  
  // ...  
end;
```

Ovoj funkciji može biti prosleđena referenca bilo koje klasekoja nasledjuje `TComponent`. Sledeći poziv je validan:

```pascal
Var  
  C : TComponent;  

begin  
  C:=CreateComponent(TEdit, Form1);  
end;
```

Na povratku `CreateComponent` funkcije, C će sadržati instancu klase `TEdit`. Primetite da se sledeći primer neće kompajlirati:

```pascal
Var  
  C : TComponent;  

begin  
  C:=CreateComponent(TStream,Form1);  
end;
```

jer `TStream` ne nasleđuje iz `TComponent`, a `AClass` referencira na `TComponent` class. Kompajler može (i hoće) da proveri ovo u vremenu kompajliranja, i proizvešće grešku.

Referenca klase može takodje biti korišćena za proveru nasledjivanja:

```pascal
  TMinClass = Class of TMyClass;  
  TMaxClass = Class of TMyClassChild;  

Function CheckObjectBetween(Instance : TObject) : boolean;  

begin  
  If not (Instance is TMinClass)  
     or ((Instance is TMaxClass)  
          and (Instance.ClassType<>TMaxClass)) then  
    Raise Exception.Create(SomeError)  
end;
```

Gornji primer će podići izuzetak ako prosledjena instanca nije nasledjena iz `TMinClass` ili `TMaxClass`.

## 10.2 Abstraktne i zapečaćene klase

Klasa može biti deklarisana kao `sealed`. U tom slučaju nije moguće deklarisanje nasledjenih klasa. Kompajler će vratiti grešku:

```pascal
{$mode objfpc}  
{$h+}  
 
Type  
  TMyClass = Class Sealed  
    x : integer;  
  end;  
 
  TMyClass2 = Class(TMyClass)  
    Y : Integer;  
  end;  
 
begin  
end.
```

Ovo će rezultovati greškom:

```sh
Error: Cannot create a descendant of the sealed class "TMyClass"
```

Abstraktna klasa je klasa koja se ne može instancirati direktno. Umesto toga nasledna klasa mora uvek biti instancirana.

## 10.3 Normalna i statička polja

Klase mogu imati polja. U zavisnosti od toga kako su definisana, polja sadrže podatke specifične za instancu klase. Bez obzira kako su definisana, polja mogu imati nametnuta pravila vidljivosti kao i drugi članovi klase.

### 10.3.1 Normalna polja/varijable

Postoji dva načina za deklaraciju normalnih polja. Prvi je klasičan način, sličan definiciji objekta:

```pascal
{$mode objfpc}

type  
  cl=class  
    l : longint;  
  end;  

var  
  cl1,cl2 : cl;  

begin  
  cl1:=cl.create;  
  cl2:=cl.create;  
  cl1.l:=2;  
  writeln(cl1.l);  
  writeln(cl2.l);  
end.
```

Ovaj kod će napraviti izlaz:

```sh
2  
0
```

Ovaj primer demonstira da vrednosti polja su inicijalizovana sa nulom (ili ekvivalentom nule za ne ordinalne tipove, `empty string`, `empty array` and so on).

Drugi način za deklaracije polja (u najnovijim verzijama FreePascala) je korišćenje `var` block:

```pascal
{$mode objfpc}

type  
  cl=class  
  var  
    l : longint;  
  end;
```

Ova definicija je ekvivalentna prehodnoj.

**Napomena**  
Od verzije 3.0 kompajlera, kompajler može preurediti poredak polja u memoriji ako to vodi boljem poravnjavanju i manjoj instanci. To znači, da u instanci polja ne moraju da se pojave u istom poretku kao u deklaraciji. RTTI generisan za klasu će reflektovati ovu promenu.

### 10.3.2 Polja/varijable klase

Slično objektima, klasa može da sadrži statička polja ili promenljive klase: ova polja ili promenljive su globalne za klasu i deluju kao globalne promenljive, ali su poznate samo kao deo klase. Na njih se može referencirati unutar metoda klasa, ali se mogu referencirati i izvan klase davanjem potpunog kvalifikovanog imena.

Opet, postoje dva načina da se definišu promenljive klase. Prvi je ekvivalentan načinu na koji se radi u objektima, koristeći `static` modifikator:

Na primer, izlaz sledećeg programa je isti kao i izlaz za verziju koja koristi objekat:

```pascal
{$mode objfpc}

type  
  cl = class  
    l : longint; static;  
  end;  

var  
  cl1,cl2 : cl;  

begin  
  cl1:=cl.create;  
  cl2:=cl.create;  
  cl1.l:=2;  
  writeln(cl2.l);  
  cl2.l:=3;  
  writeln(cl1.l);  
  Writeln(cl.l);  
end.
```

Izlaz ovo koda će biti:

```sh
2  
3  
3
```

Primetite da poslednja linja koda referencira samu klasu (`cl`), a ne instancu klase (cl1 ili cl2).

In addition to the static field approach, in classes, a Class Var can be used. Similar to the way a field can be defined in a variable block, a class variable can be declared in a class var block:

```pascal
{$mode objfpc}  

type  
  cl=class  
    class var  
      l : longint;  
  end;
```

Ova definicija je ekvivalentna prethodnoj.

Primetimo da je `class variable` povezan za određenu klasu. Nasledne klase će se odnositi na istu instancu, osim ako se promenljiva ponovo deklariše. Sledeći program to pokazuje:

```pascal
{$mode objfpc}  

type  
  TA = class                  // base type  
    class var Code: integer;  
  end;  
  TB = class(TA);  
  TC = class(TA);  
 
begin  
  TA.Code:=0;  
  TB.Code:=1;  
  TC.Code:=2;  
  Writeln(Ta.Code:2,Tb.Code:2,Tc.code:2);  
end.
```

Izlaz je:

```sh
 2 2 2
```

Pošto je povezana sa klasom, može biti preklopljeno u Delphi modu:

```pascal
{$mode delphi}  

type  
  TA = class // base type  
    class var code: integer;  
  end;  
  TB = class(TA)  
    Class var code : integer;  
  end;  
  TC = class(TA)  
    Class var code : integer;  
  end;  
 
begin  
  TA.Code:=0;  
  TB.Code:=1;  
  TC.Code:=2;  
  Writeln(Ta.Code:2,Tb.Code:2,Tc.code:2);  
end.
```

It will print the following:

```sh
 0 1 2
```

Medjutim, u OBJFPC modu to se neće kompajlirati, i daće `duplicate identifier error`.

## 10.4 Instanciranje

Klase moraju biti kreirane pomoću jednog od njihovih konstruktora (može biti više konstruktora). Zapamtite da je instanca klase pokazivač na objekat na heapu. Kada se deklariše promenljiva neke klase, kompajler samo dodeljuje prostor za ovaj pokazivač, a ne za ceo objekat. Konstruktor klase vraća pokazivač na inicijalizovanu instancu objekta na heapu. Dakle, da biste inicijalizovali instancu neke klase, uradili bi sledeće:

```pascal
  ClassVar := ClassType.ConstructorName;
```

Proširena sintaksa `new` i `dispose` se ne može se koristiti za instanciranje i uništavanje instanci klase: ta konstrukcija je rezervisana za upotrebu samo sa objektima. Pozivanje konstruktora će izazvati poziv virtuelne metode klase NewInstance, koja, u svojoj podrazumevanoj implementaciji, poziva GetMem, da dodeli dovoljno prostora za čuvanje podataka o instanci klase, a zatim isprazni memoriju.

Posle toga, kod konstrujtora je izvršen. Konstruktor ima pokazivač na svoje podatke, u `self`.

**Napomena**:

- `{$PackRecords }` direktiva takođe se primenjuje na klase, tj. poravnavanje u memoriji
  različitih polja zavisi od ove direktive.
- Kao za `objects` i `records`, pakovane klase mogu biti deklarisane. Ovo ima isti efekat kao kod
  object ili record, elementi su poravnate na 1-byte granice, što bliže moguće.
- `SizeOf(class)` će vratiti isto kao `SizeOf(Pointer)`, pošto je klasa pokazivač na objekat.
  Da dobijete veličinu instance podataka, korišćenjem `TObject.InstanceSize` metoda.
- Ako se pojavi izuzetak za vreme izvršenja konstruktora, destruktor će biti pozvan automatski.

## 10.5 Destrukcija instance

Instance klase moraju biti uništene korišćenjem destructor. Za razliku od konstruktor, nema izbora za destruktor: destruktor mora imati ime `Destroy`, on mora preklopiti `Destroy` destruktor deklarisan u `TObject`, ne može imati argumente, i nasledjeni destruktor mora uvek biti pozvon.

Destroy će pozvati `FreeInstance`, koja će po podrazumevanoj implementaciji, pozvati `FreeMem` za oslobadjanje memorije zauzete instancom.

Da bi sprečili pozivanje destruktor na Nil instanci, najbolje je pozvati `Free` metod `TObject`-a. Ovaj metod će proveriti da li `Self` nije Nil, i ako nije, tada će pozvati `Destroy`. Ako je `Self` jednak Nil, on će izaći.

Uništavanje instance ne uklanja ili postavlja Nil na referencu instance:

```pascal
Var  
  A : TComponent;  
 
begin  
  A:=TComponent.Create;  
  A.Name:='MyComponent';  
  A.Free;  
  Writeln('A is still assigned: ', Assigned(A));  
end.
```

Posle poziva `Free`, varijabla A neće biti Nil, izlaz programa biće:

```sh
A is still assigned: TRUE
```

Da bi bili sigurni da je varijabla A očišćena pošto je destruktor pozvan, funkcija `FreeAndNil` iz `SysUtils` unit može biti korišćen. To će pozvati `Free` i potom postaviti `Nil` na pokazivač objekta. (A u gornjem primeru):

```pascal
Var  
  A : TComponent;  
 
begin  
  A:=TComponent.Create;  
  A.Name:='MyComponent';  
  FreeAndNil(A);  
  Writeln('A is still assigned: ',Assigned(A));  
end.
```

Posle poziva `FreeAndNil`, varijabla A će sadržavati Nil, izlaz će biti:

```sh
A is still assigned: FALSE
```

**Napomena**  
Ako se pojavi izuzetak za vreme izvršavanje konstruktora, destruktor će biti pozvan automatski.

## 10.6 Metode

### 10.6.1 Deklaracija

Deklaracija metoda klase prati neka pravila kao deklaracija metoda u objektima.

Jedina razlika je `override`, `reintroduce` i `message` direktive.

### 10.6.2 Pozivanje metoda

Pozivanje metoda u klasama nije drugačije od objekata:

```pascal
Var  
  AnObject: TAnObject;  

begin  
  AnObject:= TAnObject.Create;  
  ANobject.AMethod;
```

### 10.6.3 Virtuelne metode

Klase imaju virtuelne metode, baš kao i objekti. Međutim, postoji razlika između to dvoje. Za objekte, dovoljno je ponovo deklarisati isti metod u objektu potomka sa ključnom rečju `virtual` da bi se on zamenio. Za klase je situacija drugačija: virtuelne metode moraju biti zamenjene ključnom reči `override`. Ako to ne uradite, pokrenuće se nova serija virtuelnih metoda, skrivajući prethodnu. Ključna reč `Inherited` neće skočiti na nasleđeni metod, ako je korišćen u virtuelnom.

The following code is wrong:

```pascal
Type  
  ObjParent = Class  
    Procedure MyProc; virtual;  
  end;  
  
  ObjChild  = Class(ObjPArent)  
    Procedure MyProc; virtual;  
  end;
```

Kompajler će prijaviti grešku:

```sh
Warning: An inherited method is hidden by OBJCHILD.MYPROC
```

Kompajler će da kompajlira, ali korišćenje `Inherited` može izazvati čudne efekte.

Korktna deklaracija je:

```pascal
Type  
  ObjParent = Class  
    Procedure MyProc; virtual;  
  end;  

  ObjChild  = Class(ObjPArent)  
    Procedure MyProc; override;  
  end;
```

Ovaj kod će se kompajlirati be upozorenja na greške.

Ako bi virtualni method trebao stvarno biti zamenjen sa metodom istog imena, tada `reintroduce` ključna reč može biti korišćena:

```pascal
Type  
  ObjParent = Class  
    Procedure MyProc; virtual;  
  end;  

  ObjChild  = Class(ObjPArent)  
    Procedure MyProc; reintroduce;  
  end;
```

Ovaj novi metod nije virtuelni.

Da bi se ovo omogućilo, kompajler pravi po klasi ( po tipu ) – tabelu sa virtuelnim metodama: VMT (Virtual Method Table). Ovo je jedna tabela sa pokazivačima na svaku virtuelnu metodu: svaki virtuelni metod ima fiksiranu lokaciju u ovaoj tabeli. Kompajler će koristiti ovu tabelu za look up aktuelni metod koji mora biti korišćen u runtime-u. Kada nasledni objekat preklopi metod, ulaz roditeljskog metoda je prepisan u VMT.

**Napomena**  
Ključna reč `virtual` može biti prepisan sa `dynamic` ključnom reči: `dynamic` metode se ponašaju kao virtualne metode. Za razliku od Delphi, u FPC implementatcija `dynamic` metode su jednake za implementaciju kao virtuelne metode.

### 10.6.4 Class metode

`Class metode` se identifikuju sa ključnom reči `Class` ispred procedure ili function deklaracije:

```pascal
  Class Function ClassName : String;
```  

`Class metode` su metode koje nemaju instancu (tj. Self ne pokazuje na instancu klase) ali koja prati opseg i pravila nasledjivanja klase. Oni mogu biti korišćenje za povratak informacija o trenutnoj klasi, na primer za registraciju ili korišćenje u class factory. Sve dok instanca klase nije raspoloživa, nema informacije o instancama koje mogu biti korišćene.

Class metode mogu biti pozvane iz regularnog metoda ali mogu biti pozvane i korišćenjem identifikatora klase:

```pascal
Var  
  AClass : TClass; // AClass is of type "type of class"  

begin  
  ..  
  if CompareText(AClass.ClassName,'TCOMPONENT')=0 then  
  ...  
```

Ali pozivanje njih iz instance je takođe moguće:

```pascal
Var  
  MyClass : TObject;  

begin  
  ..  
  if MyClass.ClassNameis('TCOMPONENT') then  
  ...
```

Obrnuto nije moguće: Unutar class metoda, Self identifikator pokazuje na VMT tabelu klase. Nema polja, propertija ili regularnih metoda. Pristup regularnom propertiju ili metodi će rezultovati greškom kompajlera.

Primetimo, da class metode mogu biti `virtual`, i mogu biti `override`.

Class metode mogu biti korišćene kao `read` ili `write` specifikatori za regularan property, ali prirodno, taj property će imati istu vrednost za sve instance klase, pošto instance nisu raspoložive u class metodama.

### 10.6.5 Konstruktori i destruktori klase

Takođe se može kreirati konstruktor ili destruktor klase. Oni služe da instanciraju neke promenljive klase ili svojstva klase koja moraju biti inicijalizovana pre nego što se klasa može koristiti. Ovi konstruktori se pozivaju automatski pri pokretanju programa: konstruktor se poziva pre odeljka za inicijalizaciju unita u kome je deklarisan, destruktor se poziva posle sekcije finalizacije unita u kome je deklarisan.

Ovde su neke činjenice kada koristimo class destructors/constructors:

- Može biti samo jedan konstruktor po klasi. Ime je proizvoljno, ali ne moge
  imati parametre.
- Može biti samo jedan konstruktor po klasi. Ime je proizvoljno, ali ne može
  imati parametre.
- Ni konstruktor ni destruktor ne mogu biti virtual.
- Konstruktor/Destruktor je pozvan bez obzira na korišćenje klase: čak iako se
  klasa ne koristi,
  konstruktor i destruktor se pozivaju.
- Ne postoji garancija poretka pozivanja konstruktora klasa. Za ugnježdene
  klase, jedini garantovani poredak je da konstruktori ugneždenih klasa se pozivaju posle konstruktora obuhvatne klase, i da se destruktori pozivaju u kontra redosledu.

```pascal
{$mode objfpc}{$h+}  

Type  
  TA = Class(TObject)  
  Private  
    Function GetA : Integer;  
    Procedure SetA(AValue : integer);  

  public  
    Class Constructor create;  
    Class Destructor destroy;  
    Property A : Integer Read GetA Write SetA;  
  end;  

{Class} Function TA.GetA : Integer;  

begin  
  Result:=-1;  
end;  

{Class} Procedure TA.SetA(AValue : integer);  

begin  
  //  
end;  

Class Constructor TA.Create;  

begin  
  Writeln('Class constructor TA');  
end;  

Class Destructor TA.Destroy;  

begin  
  Writeln('Class destructor TA');  

end;  

Var  
  A : TA;  

begin  
end.
```

Izlaz će biti:

```sh
Class constructor TA  
Class destructor TA
```

### 10.6.6 Statičke (class) metode

FPC poznaje static class metode u klasama: to su metode klase koje imaju Static ključnu reč na kraju deklaracije. Ove metode se ponašaju kao regularne procedure ili funkcije. To znači da:

- Nemaju Sel parametar. Kao rezultat, ne mogu pristupiti property ili poljima
  ili regularnim metodama instance klase.
- Ne mogu biti virtuelne.
- Mogu biti pridružene kao regularne prodeduralne varijable.

Njihovo korišćenje uglavnom uključuje metod u imenski prostor klase, u suprotnosti sa procedurom u imenskom prostoru unita. Primetimo da one imaju pristup svim varijablama klase:

```pascal
{$mode objfpc}  
{$h+}  

Type  
  TA = Class(TObject)  
  Private  
    class var myprivatea : integer;  
  public  
    class Function GetA : Integer;  static;  
    class Procedure SetA(AValue : Integer); static;  
  end;  

Class Function TA.GetA : Integer;  

begin  
  Result:=myprivateA;  
end;  

Class Procedure TA.SetA(AValue : integer);  

begin  
  myprivateA:=AValue;  
end;  

begin  
  TA.SetA(123);  
  Writeln(TA.MyPrivateA);  
end.
```

Ovo će imati izlaz 123.

U implementaciji metode statičke klase, Self identifikator nije dostupan. Metod se ponaša kao da je Self čvrsto kodiran za deklarisanu klasu, a ne za stvarnu klasu sa kojom je pozvan. U redovnim metodama klase, Self sadrži aktuelnu klasu za koju je metoda pozvana:

```pascal
Type  
  TA = Class  
    Class procedure DoIt; virtual;  
    Class Procedure DoitStatic; static;  
  end;  

  TB = CLass(TA)  
    Class procedure DoIt; override;  
  end;  

Class procedure TA.DOit;  

begin  
  Writeln('TA.Doit : ',Self.ClassName);  
end;  

Class procedure TA.DOitStatic;  

begin  
  Doit;  
  Writeln('TA.DoitStatic : ',ClassName);  
end;  

Class procedure TB.DoIt;  

begin  
  Inherited;  
  Writeln('TB.Doit : ',Self.ClassName);  
end;  

begin  
  Writeln('Through static method:');  
  TB.DoItStatic;  
  Writeln('Through class method:');  
  TB.Doit;  
end.
```

When run, this example will print:

```sh
Through static method:  
TA.Doit : TA  
TA.DoitStatic : TA  
Through class method:  
TA.Doit : TB  
TB.Doit : TB
```

Za metod statičke klase, iako je pozvan pomoću TB, klasa (Self, ako je bila dostupna) je postavljena na TA, klasu u kojoj je statička metoda definisana. Za metod klase, klasa je postavljena na stvarnu klasu koja se koristi za pozivanje metode (TB).

### 10.6.7 Message metode

Novo u klasama su `message` metode. Pokazivači na `message` metode su smešteni u posebnoj tabeli, zajedno sa integer ili string konstantantom sa kojima su deklarisani. Primarno su namenjeni za lakše programiranje `callback` funkcija u nekim GUI alatima, kao što su Win32 ili GTK. Za razliku od Delfija, Free Pascal prihvata i stringove kao identifikatore poruka. Message metode su uvek virtualne.

Dodatno, one mogu uzeti samo jedan var argument (tipiziran ili ne):

```pascal
  Procedure TMyObject.MyHandler(Var Msg); Message 1;
```

Implementacija metode message se ne razlikuje od obične metode. Takođe je moguće direktno pozvati metod message, ali to ne bi trebalo da se radi. Umesto toga, trebalo bi koristiti metod `TObject.Dispatch`.

`TObject.Dispatch` metod može biti korišćen za poziv message handlera. On je deklarisan u `system` unitu i prima var parametar koji mora imati prvu poziciju sa message ID tipa `Cardinal`, koji treba biti pozvan:

```pascal
Type  
  TMsg = Record  
    MSGID : Cardinal;  
    Data : Pointer;  

Var  
  Msg : TMSg;  

MyObject.Dispatch (Msg);
```

U ovom primeru, Dispatch metod će gledati na objekat i sve njgove pretke, da vidi da li je message metod sa MSGID deklarisan. Ako je metod nađen, on je pozvan, i prosleđen je parameter.

Ako metod nije nađen, `DefaultHandler` je pozvan. `DefaultHandler` je virtuelna metoda TObject koja ne radi ništa, ali koji može biti preklopljen da omogući bilo koje procesiranje koje je potrebno. `DefaultHandler` je deklarisan ovako:

```pascal
procedure DefaultHandler(var message); virtual;
```

Kao dodatak message metodu sa Integer identifikator, Free Pascal takođe podržava message metod sa string identifikatorom:

```pascal
Procedure TMyObject.MyStrHandler(Var Msg); Message 'OnClick';
```

Princip rada ovog string message hendlera je isti kao ordinarnog integer hendlera:

`TObject.DispatchStr` metod može biti korišćen za poziva message handler. On je deklarisan u `system` unitu i biće prihvaćen var parametar koji mora biti na prvoj poziciji kao `short` string sa message ID koji treba bit pozvan:

```pascal
Type  
  TMsg = Record  
    MsgStr : String[10]; // Arbitrary length up to 255 characters.  
    Data : Pointer;  
Var  
  Msg : TMSg;  

MyObject.DispatchStr (Msg);
```

U ovom primeru, metoda `DispatchStr` će pogledati objekat i sve njegove pretke (počevši od objekta i pretražujući stablo klasa nasleđivanja), da vidi da li je deklarisan metod message sa porukom MsgStr. Ako se pronađe takav metod, on se poziva i prosleđuje parametar Msg.

Ako takav metod nije nađen, `DefaultHandlerStr` je pozvan. `DefaultHandlerStr` je virtual metod TObject klase koji ne radi ništa, ali može biti preklopljen bilo koji procesiranjem koje je potrebno. `DefaultHandlerStr` je deklarisan ovako:

```pascal
  procedure DefaultHandlerStr(var message);virtual;
```

### 10.6.8 Korišćenje inherited klauzule

U preklopljenom metodu, često je neophodno pozvati implementaciju virtuelnog metoda iz roditeljske klase. To može biti urađeno sa `inherited` ključnom reči. Slično, `inherited` ključna reč može biti korišćen za poziv bilo koje metode roditeljske klase.

Prvi slučaj je najjednostavniji:

```pascal
Type  
  TMyClass = Class(TComponent)  
    Constructor Create(AOwner : TComponent); override;  
  end;  

Constructor TMyClass.Create(AOwner : TComponent);  

begin  
  Inherited;  
  // Do more things  
end;
```

U prethodnom primeru, `Inherited` izjava će pozvati Create od TComponent, prosledjujući AOwner kao parametarr: isti parametri koji su prosledjeni tekućoj metodi će biti prosledjeni roditeljskoj metodi. Oni ne moraju biti specificirani ponovo, ako ništa nije specificirano, kompajler će proslediti argumente kao one primljene.

Ako ne postoji inherited metod istog imena, Inherited neće imati efekta. Prisustvo Inherited metoda u toj formi može biti interpretirano kao “pozovi sve overridden metode ako postoje”.

Drugi slučaj je blago komplikovaniji:

```pascal
Type  
  TMyClass = Class(TComponent)  
    Constructor Create(AOwner : TComponent); override;  
    Constructor CreateNew(AOwner : TComponent; DoExtra : Boolean);  
  end;  

Constructor TMyClass.Create(AOwner : TComponent);  
begin  
  Inherited;  
end;  

Constructor TMyClass.CreateNew(AOwner : TComponent; DoExtra : Boolean);  
begin  
  Inherited Create(AOwner);  
  // Do stuff  
end;
```

CreateNew metod će prvo pozvati `TComponent.Create` i proslediti AOwner kao parametar. On neće pozvati `TMyClass.Create`.

Ako ne postoji metod sa takvim imenom u parent klasi, kompajler će izdati grešku.

Iako su primeri dati korišćenjem konstruktora, korišćenje `inherited` klauzule nije ograničeno na konstruktore, to može biti korišćeno za bilo koju proceduru ili funkciju ili destruktor isto tako.

## 10.7 Propertiji

### 10.7.1 Definicija

Klase mogu sadržati propertije kao deo njihove liste polja. Properti je kao normalno polje, tj. njegova vrednost može biti vraćeno ili postavljeno, ali oni omogućavaju redirekciju pristupa polja kroz funkcije ili procedure. Oni obezbeđuju način da se properti poveže sa pisanjem ili čitanjem iz klase „polja“. Ovo omogućava tzv. proveru da li je vrednost važeća prilikom dodeljivanja, ili, prilikom čitanja, omogućava konstruisanje vrednosti u hodu. Štaviše, propertiji mogu biti samo za čitanje ili samo za pisanje.

Read specifikator je ili ime polja koje sadrži property, ili ime metode koja ima isti povratni
tip kao tip propertija. U slučaju prostih tipova, metoda ne mora da prihvata argument. U slučaju propertija niza metoda ne mora da prihvati argument. U slučaju niza propertija, metoda mora prihvatiti prosti argument istog tipa kao index. S slučaju indexed property, metoda mora prihvatiti integer kao argument.

Specifikator čitanja je opcioni, čineći svojstvo samo za pisanje. Imajte na umu da se metode klase ne mogu koristiti kao specifikacije čitanja.

Specifikator pisanja je opcioni: ako ne postoji specifikacija pisanja, syvojstvo je samo za čitanje. Specifikator pisanja je ili ime polja, ili ime procedure metode koja prihvata kao jedini argument promenljivu istog tipa kao i svojstvo. U slučaju svojstva niza, procedura mora prihvatiti dva argumenta: prvi argument mora imati isti tip kao indeks, drugi argument mora biti istog tipa kao svojstvo. Slično, u slučaju indeksiranog svojstva, prvi parametar mora biti ceo broj.

Odeljak (privatan, objavljen) u kome se nalazi navedena funkcija ili procedura je irelevantan. Međutim, obično će ovo biti zaštićena ili privatna metoda.

```pascal
Type  
  MyClass = Class  
    Private  
    Field1 : Longint;  
    Field2 : Longint;  
    Field3 : Longint;  
    
    Procedure  Sety (value : Longint);  
    Function Gety : Longint;  
    Function Getz : Longint;  
    
    Public  
    Property X : Longint Read Field1 write Field2;  
    Property Y : Longint Read GetY Write Sety;  
    Property Z : Longint Read GetZ;  
    end;  

Var  
  MyClass : TMyClass;
```

Sledeće su validne izjave:

```pascal
WriteLn ('X : ', MyClass.X);  
WriteLn ('Y : ',MyClass.Y);  
WriteLn ('Z : ',MyClass.Z);  
MyClass.X := 0;  
MyClass.Y := 0;
```

Ali sledeće će generisati grešku:

```pascal
MyClass.Z := 0;
```

jer je Z read-only property.

Ono što se dešava u gornjim izjavama je da kada vrednost treba da se pročita, kompajler ubacuje poziv različitim getNNN metodama objekta i koristi se rezultat ovog poziva. Kada se izvrši dodela, prevodilac prosleđuje vrednost koja se mora dodeliti kao parametar različitim setNNN metodama.

Zbog ovog mehanizma, svojstva se ne mogu prosleđivati kao var argumenti funkciji ili proceduri, pošto ne postoji poznata adresa svojstva (barem, ne uvek).

### 10.7.2 Indeksirani properti

Ako definicija propertij sadrži indeks, tada Read i Write specifikatori motaju biti metode. Štaviše, te metode zahtevaju dodatni parametar: integer parametar. To mogućava da read ili write ima nekoliko prpertia sa istom metodom. Zbog toga propertiji moraju imati isti tip.

Sledeći primer propertija sa indeksom:

```pascal
{$mode objfpc}  

Type  
  TPoint = Class(TObject)  
  Private  
    FX,FY : Longint;  
    Function GetCoord (Index : Integer): Longint;  
    Procedure SetCoord (Index : Integer; Value : longint);  
  Public  
    Property X : Longint index 1 read GetCoord Write SetCoord;  
    Property Y : Longint index 2 read GetCoord Write SetCoord;  
    Property Coords[Index : Integer]: Longint Read GetCoord;  
  end;  

Procedure TPoint.SetCoord (Index : Integer; Value : Longint);  
begin  
  Case Index of  
   1 : FX := Value;  
   2 : FY := Value;  
  end;  
end;  

Function TPoint.GetCoord (INdex : Integer) : Longint;  
begin  
  Case Index of  
   1 : Result := FX;  
   2 : Result := FY;  
  end;  
end;  

Var  
  P : TPoint;  

begin  
  P := TPoint.create;  
  P.X := 2;  
  P.Y := 3;  
  With P do  
    WriteLn ('X=', X,' Y=', Y);  
end.
```

Kada kompajler naiđe na dodeljivanje X-u, tada se SetCoord poziva sa kao prvim parametrom indeksom (1 u gornjem slučaju) i kao drugim parametrom sa vrednošću koju treba postaviti. Suprotno tome, kada čita vrednost X, kompajler poziva GetCoord i prosleđuje mu indeks 1. Indeksi mogu biti samo celobrojne vrednosti.

### 10.7.3 Niz properti

Niz properti takođe postoje. Ovo su propertiji koja prihvataju indeks, baš kao što to čini niz. Indeks može biti jednodimenzionalan ili višedimenzionalan. Za razliku od normalnih (statičkih ili dinamičkih) nizova, indeks niz propertija ne mora biti redni tip, ali može biti bilo koji tip.

Specifikator čitanja za niz propertije je metoda koja ima isti tip povratka kao tip svojstva. Metoda mora da prihvati kao jedini argument promenljivu istog tipa kao tip indeksa. Za niz properti, ne mogu se navesti polja kao specifikatori čitanja.

Specifikator pisanja za niz properti je ime metode koja prihvata dva argumenta: prvi argument ima isti tip kao indeks, a drugi argument je parametar istog tipa kao i tip svojstva. Kao primer, pogledajte sledeću deklaraciju:

```pascal
Type  
  TIntList = Class  
  Private  
    Function GetInt (I : Longint) : longint;  
    Function GetAsString (A : String) : String;  
    Procedure SetInt (I : Longint; Value : Longint;);  
    Procedure SetAsString (A : String; Value : String);  
  Public  
    Property Items [i : Longint] : Longint Read GetInt Write SetInt;  
    Property StrItems [S : String] : String Read GetAsString Write SetAsstring;  
  end;  

Var  
  AIntList : TIntList;
```

Tada bi važile sledeće izjave:

```pascal
AIntList.Items[26] := 1;  
AIntList.StrItems['twenty-five'] := 'zero';   
WriteLn ('Item 26 : ',AIntList.Items[26]);  
WriteLn ('Item 25 : ',AIntList.StrItems['twenty-five']);
```

Dok bi sledeće izjave generisale greške:

```pascal
AIntList.Items['twenty-five'] := 1;  
AIntList.StrItems[26] := 'zero';
```

Zato što su tipovi indeksa pogrešni.

Niz properti mogu biti višedimenzionalna:

```pascal
Type  
  TGrid = Class  
  Private  
    Function GetCell (I,J : Longint) : String;  
    Procedure SetCell (I,J : Longint; Value : String);  
  Public  
    Property Cellcs [Row,Col : Longint] : String Read GetCell Write SetCell;  
  end;
```

Ako postoji N dimenzija, onda tipovi prvih N argumenata getera i setera moraju odgovarati tipovima N indeksnih specifikacija u definiciji svojstva niza.

### 10.7.4 Default propertiji

Niz propertii mogu biti deklarisani kao podrazumevani propertii. To znači da nije potrebno navesti ime propertija prilikom dodeljivanja ili čitanja. U prethodnom primeru, ako bi definicija propertija stavke bila

```pascal
Property Items[i : Longint]: Longint Read GetInt Write SetInt; Default;
```

Tada bi pridruživanje

```pascal
AIntList.Items[26] := 1;
```

bilo ekvivalentno:

```pascal
AIntList[26] := 1;
```

Samo jedan podrazumevani properti po klasi je dozvoljen, ali izvedene klase mogu redeklarisati podrzumevani properti.

### 10.7.5 Objavljena svojstva

Klase kompajlirane u stanju {$M+} (kao što je TPersistent iz unita class) mogu imati `Published` odeljak. Za metode, polja i svojstva u odeljku `Published`, kompajler generiše RTTI informacije (Informacije o tipu vremena rada), koje se mogu koristiti za ispitivanje definisanih metoda, polja i svojstava u objavljenim odeljcima. Unit `typeinfo` sadrži neophodne rutine za ispitivanje ovih informacija, a ovaj unit se koristi u sistemu strimovanja u FPC u unitu `class`.

RTTI se generiše bez obzira na to šta su specifikacije čitanja i pisanja: polja, funkcije/procedure ili indeksirane funkcije/procedure.

Mogu se objaviti samo polja tipizirana u klasi. Za propertije, svaki jednostavni properti čija je veličina manja ili jednaka pokazivaču, može se proglasiti objavljenim: float, celi brojevi, skupovi (sa manje od 32 različita elementa), nabrojani, klase ili dinamički nizovi (ne niz properti).

Iako su informacije o tipu vremena izvršavanja dostupne za druge tipove, ovi tipovi se ne mogu koristiti za definiciju svojstva ili polja u objavljenom odeljku. Informacije su prisutne da opisuju, na primer, argumente procedura ili funkcija.

### 10.7.6 Informacije o skladištenju

Kompajler nudi dva specifikatora za kontrolu da li se properti strimuje pomoću mehanizma strimovanja, kao što je onaj koji je implementiran u unitu class. Ovi specifikatori pišu dodatne informacije u generisani RTTI, koji se kasnije koristi u sistemu za striming:

- Sačuvana specifikacija.
- Podrazumevana specifikacija i njen pandan - nodefault.

Oba ova specifikatora se procenjuju da bi se odlučilo da li se svojstvo mora strimovati.

Argument uskladištene specifikacije treba da bude ili logička konstanta, logičko polje klase ili funkcija bez parametara koja vraća logički rezultat. Ako je argument uskladištene specifikacije procenjen na False, properti neće biti strimovano.

**Napomena**  
Odsustvo uskladištene specifikacija je ekvivalentno sačuvanom True.

Ako je procena `Stored` dovela do True, podrazumevana vrednost svojstva se smatra:

- Podrazumevana specifikacija može se navesti za redne tipove i skupove. Propertii koja kao
  vrednost imaju ovu podrazumevanu vrednost, sistem za striming neće upisati u tok, čak i ako je Stored vrednost True. Podrazumevana vrednost se čuva u RTTI-u koji se generiše za klasu.

- Properti stringa, pokretne tačke i pokazivača imaju implicitnu podrazumevanu vrednost praznog
  stringa, 0 ili nil, respektivno. Svojstva rednog i skupa nemaju implicitnu podrazumevanu vrednost.

- Specifikator `nodefault` mora da se koristi da označi da svojstvo nema podrazumevanu vrednost.
  Efekat je da se vrednost ovog svojstva uvek upisuje u tok prilikom strimovanja svojstva, ako je sačuvana vrednost True.

**Primedba**:

- Kada se klasa instancira, podrazumevana vrednost se ne primenjuje automatski za properti,
  Odgovornost je programera da to uradi u konstruktoru klase.
- Vrednost -2147483648 se ne može koristiti kao podrazumevana vrednost, jer se koristi interno za
  označavanje nodefault.
- Nije moguće odrediti podrazumevanu vrednost za niz proprtije.
- Nije moguće specificirati Stored direktivu za niz propertije.
- Sve specifikacije skladištenja mogu biti zamenjene u klasama potomaka.
- Specifikator nodefault može da se koristi za zamenu implicitno podrazumevane specifikacije
  propertia niza, pokretnog zareza i pokazivača.
- Mehanizam striminga koji je ovde opisan je onaj koji je implementiran u klasama unita RTL-a.
  Mogu se implementirati i drugi mehanizmi striminga, koji mogu koristiti RTTI informacije na drugačiji način.

### 10.7.7 Overriding and redeclaring properties

Properties can be both overridden and redeclared in descendent classes.

Property redeclaration takes action if the property type is declared, otherwise it is property override. The only difference is that property override replaces or extends the inherited modifiers with the new modifiers, whereas property redeclaration hides all inherited modifiers that are not present in the redeclaration. The type of the redeclared property does not have to be the same as the parent”s class property type.

The example below demonstrates the difference between property override and redeclaration.

### 10.7.7 Preklapanje i ponovno deklarisanje propertija

Propertiji se mogu i preklopiti i ponovo deklarisati u klasama potomaka.

Ponovna deklaracija svojstva preduzima radnju ako je tip svojstva deklarisan, u suprotnom je preklapanje svojstva. Jedina razlika je u tome što preklapanje propertija zamenjuje ili proširuje nasleđene modifikatore novim modifikatorima, dok ponovna deklaracija svojstva sakriva sve nasleđene modifikatore koji nisu prisutni u ponovnoj deklaraciji. Tip ponovo deklarisanog propertija ne mora da bude isti kao tip propertija nadređene klase.

Primer ispod pokazuje razliku između preklapanja svojstva i ponovne deklaracije.

```pascal
type  
  TAncestor = class  
  private  
    FP1 : Integer;  
  public  
    property P: integer Read FP1 write FP1;  
  end;  

  TP1 = class(TAncestor)  
  public  
    // property override  
    property P default 1;  
  end;  

  TPReadOnly = class(TAncestor)  
  public  
    // property redeclaration  
    property P: integer Read FP1;  
  end;
```

TP1 proširuje property P sa podrazumevanom vrednosšću, TPReadOnly redeklariše property P kao read-only.

**Napomena**  
TP1 should set the default value of P to 1 in its constructor.

In case of both property redeclaration and property override, the access to the getter and setter is always static. I.e. property override acts only on the RTTI of the object and is not to be confused with method override.

The keyword “inherited” can be used to refer to the parent definition of the property. For example consider the following code:

TP1 treba da postavi podrazumevanu vrednost P na 1 u svom konstruktoru.

U slučaju i ponovne deklaracije svojstva i zaobilaženja svojstva, pristup geteru i seteru je uvek statičan. tj. nadjačavanje svojstva deluje samo na RTTI objekta i ne treba ga mešati sa zamenom metoda.

Ključna reč `inherited` može se koristiti za upućivanje na roditeljsku definiciju svojstva. Na primer, razmotrite sledeći kod:

```pascal
type  
  TAncestor = class  
  private  
    FP1 : Integer;  
  public  
    property P: integer Read FP1 write FP1;  
  end;  

  TClassA = class(TAncestor)  
  private  
    procedure SetP(const AValue: char);  
    function getP : Char;  
  public  
    constructor Create;  
    property P: char Read GetP write SetP;  
  end;  

procedure TClassA.SetP(const AValue: char);  

begin  
  Inherited P:=Ord(AValue);  
end;  

procedure TClassA.GetP : char;  

begin  
  Result:=Char((Inherited P) and $FF);  
end;
```

TClassA redefiniše P kao properti karaktera umesto celobrojnog propertija, ali koristi properti P roditelja za čuvanje vrednosti.

Mora se voditi računa kada se koriste virtuelne rutine get/set za properti: podešavanje nasleđenog propertija i dalje poštuje normalna pravila nasleđivanja metoda. Razmotrite sledeći primer:

```pascal
type  
  TAncestor = class  
  private  
    procedure SetP1(const AValue: integer); virtual;  
  public  
    property P: integer write SetP1;  
  end;  

  TClassA = class(TAncestor)  
  private  
    procedure SetP1(const AValue: integer); override;  
    procedure SetP2(const AValue: char);  
  public  
    constructor Create;  
    property P: char write SetP2;  
  end;  

constructor TClassA.Create;  
begin  
  inherited P:=3;  
end;
```

U ovom slučaju, prilikom postavljanja nasleđenog svojstva P, biće pozvana implementacija TClassA.SetP1, jer je metod SetP1 poništen.

Ako se implementacija nadređene klase SetP1 mora pozvati, onda se ovo mora eksplicitno pozvati:

```pascal
constructor TClassA.Create;  
begin  
  inherited SetP1(3);  
end;
```

Ponovo deklarisana svojstva predaka su takođe dostupna iznutra i izvan objekta potomka sa direktnim prebacivanjem na pretka:

```pascal
function GetP(const AClassA: TClassA): Integer;  
begin  
  Result := TAncestor(AClassA).P;  
end;
```

## 10.8 Klasni propertiji

Class properties are very much like global property definitions. They are associated with the class, not with an instance of the class.

A consequence of this is that the storage for the property value must be a class var, not a regular field or variable of the class: normal fields or variables are stored in an instance of the class.

Class properties can have a getter and setter method like regular properties, but these must be static methods of the class.

That means that the following contains a valid class property definition:

Propertiji klase su veoma slični definicijama globalnih svojstava. Oni su povezani sa klasom, a ne sa instancom klase.

Posledica ovoga je da skladište za vrednost svojstva mora biti var klase, a ne redovno polje ili promenljiva klase: normalna polja ili promenljive se čuvaju u instanci klase.

Svojstva klase mogu imati metod grtere i setere kao obična svojstva, ali to moraju biti statičke metode klase.

To znači da sledeće sadrži važeću definiciju svojstva klase:

```pascal
TA = Class(TObject)  
Private  
  class var myprivatea : integer;  
  class Function GetB : Integer;  static;  
  class Procedure SetA(AValue : Integer); static;  
  class Procedure SetB(AValue : Integer); static;  
public  
  Class property MyA : Integer Read MyPrivateA Write SetA;  
  Class property MyA : Integer Read GetB Write SetB;  
end;
```

Razlog za zahtev je da je svojstvo klase povezano sa određenom klasom u kojoj je definisano, ali ne i sa klasama potomcima. Pošto metode klase mogu biti virtuelne, to bi omogućilo klasama potomaka da zamene metod, čineći ih neprikladnim za pristup svojstvima klase.

## 10.9 Ugnežđeni tipovi, konstante i promenljive

Definicija klase može da sadrži type sekciju, const sekciju i var sekciju. Sekcije type i const deluju kao obični odeljak type koji se nalazi u implementaciji jedinice ili metode/funkcije/procedure. Var deluju kao redovna polja klase, osim ako se ne nalaze u sekciji var klase, u kom slučaju se ponašaju kao da su definisane na nivou unita, unutar imenskog prostora klase (odeljak 6.3, strana 308).

Međutim, vidljivost ovih sekcija igra ulogu: privatne i zaštićene (stroge ili ne) konstante, tipovi i promenljive mogu se koristiti samo onoliko koliko njihova vidljivost dozvoljava.

Javni tipovi se mogu koristiti van klase, po svom punom imenu:

```pascal
type  
  TA = Class(TObject)  
  Public  
    Type TEnum = (a,b,c);  
    Class Function DoSomething : TEnum;  
  end;  

Class Function TA.DoSomething : TEnum;  

begin  
  Result:=a;  
end;  

var  
  E : TA.TEnum;  

begin  
  E:=TA.DoSomething;  
end.
```

gde je

```pascal
type  
  TA = Class(TObject)  
  Strict Private  
    Type TEnum = (a,b,c);  
  Public  
    Class Function DoSomething : TEnum;  
  end;  

Class Function TA.DoSomething : TEnum;  

begin  
  Result:=a;  
end;  

var  
  E : TA.TEnum;  

begin  
  E:=TA.DoSomething;  
end.
```

Neće se kompajlirati i pokrenuće grešku:

```pascal
tt.pp(20,10) Error: identifier idents no member "TEnum"
```

Imajte na umu da za konstante koje se mogu pisati važe ista pravila kao i za varijable klase u pogledu opsega i preglasavanja u potomcima:

```pascal
{$mode delphi}{$J+}  

type  
  TA = class // base type  
    const CODE: integer = 99;  
  end;  
  TB = class(TA);  
  TC = class(TA);  

begin  
  TA.Code:=0;  
  TB.Code:=1;  
  TC.Code:=2;  
  Writeln(Ta.Code:2,Tb.Code:2,Tc.code:2);  
end.
```

Će ispisati:

```sh
 2 2 2
```

Ali

```pascal
{$mode delphi}{$J+}  

type  
  TA = class // base type  
    const CODE: integer = -99;  
  end;  
  TB = class(TA)  
    const code : integer = -98;  
  end;  
  TC = class(TA)  
    Const code : integer = -97;  
  end;  

begin  
  TA.Code:=0;  
  TB.Code:=1;  
  TC.Code:=2;  
  Writeln(Ta.Code:2,Tb.Code:2,Tc.code:2);  
end.
```

će ispisati:

```sh
 0 1 2
```

## 10.15 Pregled vrsta metoda u Classes

Classes nude različite vrste metoda, sličnih onima u Objects, ali sa važnim razlikama:

 Tip metode | Deklaracija | Karakteristike | Kada koristiti |
 ---------- | ----------- | -------------- | -------------- |
 **Static (Normal)** | `procedure Doit;` | Poziv se određuje u vreme kompajliranja. Tip promenljive određuje metodu. Najbrža metoda poziva. Ne može se override-ovati | Kada polimorfizam nije potreban |
 **Virtual** | `procedure Doit; virtual;` | Poziv se određuje u vreme izvršavanja. Koristi VMT. U potomku se koristi `override`. Može se override-ovati | Standardni polimorfni metod |
 **Dynamic** | `procedure Doit; dynamic;` | Isto kao `virtual` u FPC. U Delphi ima drugu implementaciju. U potomku koristi `override` | Za Delphi kompatibilnost |
 **Abstract** | `procedure Doit; virtual; abstract;` | Mora biti `virtual`. Nema implementaciju.Mora se override-ovati u potomku. Ne može se kreirati instanca | Interfejsi/apstraktne bazne klase |
 **Class Method** | `class procedure Doit;` | Self pokazuje na VMT klase.- Pristup class poljima i metodama. Nema pristup običnim poljima. Može biti virtual i override | Fabrike, registracija tipova |
 **Static Class Method** | `class procedure Doit; static;` | Nema Self parametar. Pristup samo class poljima. Ne može biti virtual. Može se dodeliti proc. promenljivoj | Pomoćne funkcije u namespace-u klase |
 **Class Constructor** | `class constructor Create;` | Poziva se pre initialization sekcije. Samo jedan po klasi.- Bez parametara.- Ne može biti virtual | Inicijalizacija class polja |
 **Class Destructor** | `class destructor Destroy;` | Poziva se nakon finalization sekcije.Samo jedan po klasi. Bez parametara. Ne može biti virtual | Čišćenje class resursa |
 **Message Method** | `procedure Handler(var Msg); message 1;` | Automatski virtual. Poziva se preko Dispatch(). Jedan var parametar. Integer ili string ID | GUI callback funkcije |

### 10.15.1 Ključna razlika: Objects vs Classes

Najvažnija razlika u sintaksi virtuelnih metoda:

**U Objects:**

```pascal
Type
  TParent = Object
    procedure Doit; virtual;  
  end;
  
  TChild = Object(TParent)
    procedure Doit; virtual;   // Ponovo 'virtual'!
  end;
```

**U Classes:**

```pascal
Type
  TParent = Class
    procedure Doit; virtual;   
  end;
  
  TChild = Class(TParent)
    procedure Doit; override;  // Koristi 'override'!
  end;
```

Ako u Classes koristiš `virtual` umesto `override`, dobićeš **warning**:

```sh
Warning: An inherited method is hidden by TCHILD.DOIT
```

### 10.15.2 Primer: Razlika između različitih tipova metoda

```pascal
{$mode objfpc}
{$h+}
program ClassMethodsDemo;

type
  TExample = class
  private
    FValue: Integer;
    class var FClassValue: Integer;
  public
    // Normal method - ima Self, pristup svemu
    procedure NormalMethod;
    
    // Virtual method - može se override-ovati
    procedure VirtualMethod; virtual;
    
    // Abstract method - mora se implementirati u potomku
    procedure AbstractMethod; virtual; abstract;
    
    // Class method - Self pokazuje na VMT
    class procedure ClassMethod;
    
    // Static class method - nema Self
    class procedure StaticClassMethod; static;
    
    // Class constructor - poziva se automatski
    class constructor Create;
    
    // Class destructor - poziva se automatski  
    class destructor Destroy;
    
    // Message method - za callback funkcije
    procedure MessageHandler(var Msg); message 100;
  end;

procedure TExample.NormalMethod;
begin
  FValue := 10;                    // OK - pristup instance polju
  FClassValue := 20;               // OK - pristup class polju
  WriteLn('Normal: ', ClassName);  // OK - pristup class metodi
end;

procedure TExample.VirtualMethod;
begin
  WriteLn('Virtual method called');
end;

class procedure TExample.ClassMethod;
begin
  // FValue := 10;                 // GREŠKA - nema pristup instance poljima
  FClassValue := 30;               // OK - pristup class polju
  WriteLn('Class: ', ClassName);   // OK
end;

class procedure TExample.StaticClassMethod;
begin
  // FValue := 10;                 // GREŠKA
  FClassValue := 40;               // OK
  WriteLn('Static class method');  // OK
  // ClassName se tretira kao TExample.ClassName
end;

class constructor TExample.Create;
begin
  FClassValue := 0;
  WriteLn('Class constructor called');
end;

class destructor TExample.Destroy;
begin
  WriteLn('Class destructor called');
end;

procedure TExample.MessageHandler(var Msg);
begin
  WriteLn('Message handler called');
end;

// Potomak koji override-uje virtual metod
type
  TChild = class(TExample)
    procedure VirtualMethod; override;  // MORA biti 'override'!
    procedure AbstractMethod; override; // Implementacija abstract metoda
  end;

procedure TChild.VirtualMethod;
begin
  inherited;  // Pozovi roditeljski metod
  WriteLn('Child virtual method');
end;

procedure TChild.AbstractMethod;
begin
  WriteLn('Child abstract implementation');
end;

var
  Parent: TExample;
  Child: TChild;
  
begin
  // Class constructor je već pozvan ovde!
  
  Parent := TExample.Create;
  Child := TChild.Create;
  
  Parent.NormalMethod;
  Parent.VirtualMethod;
  
  Child.VirtualMethod;  // Poziva TChild.VirtualMethod
  Child.AbstractMethod;
  
  TExample.ClassMethod;
  TExample.StaticClassMethod;
  
  Parent.Free;
  Child.Free;
  
  // Class destructor će biti pozvan na kraju
end.
```

Output:

```sh
Class constructor called
Normal: TExample
Virtual method called
Virtual method called
Child virtual method
Child abstract implementation
Class: TExample
Static class method
Class destructor called
```

### 10.15.3 Kada koristiti koju vrstu metoda

 Situacija | Preporučena vrsta |
 ---------- | ----------------- |
 Običan metod bez nasleđivanja | Normal (static) |
 Polimorfni metod koji se override-uje | Virtual + override |
 Interfejs koji potomci moraju implementirati | Abstract |
 Metod koji treba pozivati bez instance | Class method |
 Pomoćna funkcija u namespace-u klase | Static class method |
 Inicijalizacija class polja | Class constructor |
 GUI event handler | Message method |
 Factory pattern | Class method (virtual) |

### 10.15.4 Česte greške i kako ih izbegavati

**1. Zaboravljen override:**

```pascal
// LOŠE - kompajler će dati warning
type
  TChild = class(TParent)
    procedure MyProc; virtual;  // Trebalo je: override
  end;
```

**2. Pokušaj pristupa instance poljima iz class metoda:**

```pascal
class procedure TExample.ClassMethod;
begin
  FValue := 10;  // GREŠKA - FValue je instance polje
end;
```

**3. Pozivanje abstract metoda:**

```pascal
var
  Obj: TAbstractClass;
begin
  Obj := TAbstractClass.Create;  // GREŠKA - ne može se kreirati
end;
```

**4. Zaboravljen inherited u destruktoru:**

```pascal
destructor TChild.Destroy;
begin
  // cleanup...
  inherited;  // MORA biti na kraju!
end;
```

### 10.15.5 Zašto Classes, a ne Objects?

 Prednost Classes | Objašnjenje |
 ----------------- | ----------- |
 **Modernija sintaksa** | `override` umesto ponovnog `virtual` |
 **Exception safety** | Automatski poziv destruktora pri greški |
 **Reference counting** | Podrška za automatsko upravljanje memorijom |
 **Interfejsi** | Potpuna podrška za interfejse |
 **RTTI** | Bolja run-time type information |
 **Properties** | Naprednije property funkcionalnosti |
 **Message methods** | Za GUI programiranje |
 **Class methods** | Moćniji class-level metodi |
 **Delphi kompatibilnost** | Lakša portabilnost koda |

**Zaključak:** Za novi kod, **koristi Classes**. Objects su tu za legacy kod ili specifične low-level slučajeve.

Detaljnije o Objects-ima vidi u [05_objects.md](05_objects.md).

## 10.16 Self u Class metodama - praktična upotreba

Kada class metoda (bez `static`) ima Self parametar, Self pokazuje na **VMT (Virtual Method Table)** te klase, što omogućava moćne programske obrasce.

### 10.16.1 Self u različitim kontekstima

 Vrsta metode | Self pokazuje na |
 ------------ | ---------------- |
 **Obična metoda instance** | **Instancu objekta** (konkretni objekat u memoriji - podaci) |
 **Class metoda** | **VMT (Virtual Method Table)** - class pointer, metapodatke klase |
 **Static class metoda** | **Nema Self** - ili se tretira kao hardcoded klasa |

### 10.16.2 Primer razlike u Self-u

```pascal
Type
  TExample = Class
    FData: Integer;                 // Instance polje
    class var FClassData: Integer;  // Class polje
    
    procedure InstanceMethod;       // Self → instanca (objekat sa podacima)
    class procedure ClassMethod;    // Self → VMT (metapodaci klase)
    class procedure StaticMethod; static;   // Nema Self
  end;

procedure TExample.InstanceMethod;
begin
  FData := 10;              // OK - Self.FData (instance podataka)
  WriteLn(Self.ClassName);  // Self je instanca, ali može pristupiti VMT-u
end;

class procedure TExample.ClassMethod;
begin
  // FData := 10;           // GREŠKA - Self je VMT, ne instanca!
  FClassData := 20;         // OK - class podatak
  WriteLn(Self.ClassName);  // Self je VMT (class pointer)
end;

class procedure TExample.StaticMethod;
begin
  // Self;                  // GREŠKA - nema Self!
  FClassData := 30;         // OK - pristup kroz ime klase
end;
```

### 10.16.3 Praktična upotreba

1. **Factory Pattern**

   Self omogućava kreiranje instance prave klase bez hardcoding-a:

   ```pascal
   Type
     TAnimal = Class
       class function CreateAnimal: TAnimal;  // Bez virtual!
     end;
     
     TDog = Class(TAnimal)
     end;
     
     TCat = Class(TAnimal)
     end;
   
   class function TAnimal.CreateAnimal: TAnimal;
   begin
     Result := Self.Create;  // Self je VMT - kreira pravu klasu!
   end;
   
   var
     Dog: TAnimal;
     Cat: TAnimal;
   begin
     Dog := TDog.CreateAnimal;  // Self = TDog VMT → kreira TDog!
     Cat := TCat.CreateAnimal;  // Self = TCat VMT → kreira TCat!
     
     WriteLn(Dog.ClassName);    // "TDog"
     WriteLn(Cat.ClassName);    // "TCat"
   end;
   ```

   **Ključna prednost:** Isti kod radi za sve potomke bez override-ovanja!

2. **Registracija klasa**

   ```pascal
   Type
     TForm = Class
       class procedure RegisterForm;
     end;
   
   class procedure TForm.RegisterForm;
   begin
     // Self.ClassName = ime forme koja poziva
     WriteLn('Registering: ', Self.ClassName);
     FormRegistry.Add(Self.ClassName, Self);
   end;
   
   // Automatski registruje pravu klasu:
   TLoginForm.RegisterForm;   // "Registering: TLoginForm"
   TMainForm.RegisterForm;    // "Registering: TMainForm"
   ```

3. **Type checking i RTTI**

   ```pascal
   class function TAnimal.CanFly: Boolean;
   begin
     Result := Self.InheritsFrom(TBird);  // Da li je ova klasa ptica?
   end;
   
   class function TAnimal.GetSize: Integer;
   begin
     Result := Self.InstanceSize;  // Kolika je instanca ove klase?
   end;
   
   class function TAnimal.GetInfo: string;
   begin
     Result := Format('Class: %s, Size: %d bytes', [Self.ClassName, Self.InstanceSize]);
   end;
   ```

4. **Singleton Pattern**

   ```pascal
   Type
     TDatabase = Class
     private
       class var FInstance: TDatabase;
     public
       class function GetInstance: TDatabase;
     end;
   
   class function TDatabase.GetInstance: TDatabase;
   begin
     if not Assigned(FInstance) then
       FInstance := Self.Create;  // Kreira pravu klasu (može biti potomak)!
     Result := FInstance;
   end;
   
   // Korišćenje:
   var
     DB: TDatabase;
   begin
     DB := TMySQL.GetInstance;       // Kreira TMySQL singleton
     DB := TPostgreSQL.GetInstance;  // Kreira TPostgreSQL singleton
   end;
   ```

### 10.16.4 Poređenje: Sa i bez Self-a

**Bez Self-a (LOŠE - ponavljanje koda):**

```pascal
class function TDog.CreateAnimal: TAnimal;
begin
  Result := TDog.Create;  // Hardcoded - mora override u svakom potomku
end;

class function TCat.CreateAnimal: TAnimal;
begin
  Result := TCat.Create;  // Ponavljanje koda!
end;
```

**Sa Self-om (DOBRO - generički kod):**

```pascal
// Jedan metod u roditeljskoj klasi:
class function TAnimal.CreateAnimal: TAnimal;
begin
  Result := Self.Create;  // Self automatski zna koja je klasa!
end;

// Svi potomci dobijaju funkcionalnost besplatno:
Dog := TDog.CreateAnimal;    // Self = TDog
Cat := TCat.CreateAnimal;    // Self = TCat
Bird := TBird.CreateAnimal;  // Self = TBird
```

### 10.16.5 VMT (Virtual Method Table) - šta sadrži?

**VMT sadrži metapodatke o klasi**:

- Pokazivače na virtuelne metode
- Informacije o tipu (ClassName, ClassType, ClassParent)
- Class polja (class var)
- RTTI (Run-Time Type Information)
- Veličinu instance (InstanceSize)

**Zato class metoda sa Self-om može**:

- Pristupiti class poljima
- Pozvati class metode
- Koristiti ClassName, ClassType, InheritsFrom
- Kreirati instance prave klase (Self.Create)
- **ALI NE MOŽE pristupiti poljima poljima** (jer Self nije instanca)

### 10.16.6 Kada koristiti Self u class metodi?

 Situacija | Da li treba Self? | Koristi |
 ---------- | ----------------- | ------- |
 Factory metod koji kreira instance | **DA** | `Self.Create` |
 Registracija klasa u registry | **DA** | `Self.ClassName` |
 Type checking (InheritsFrom) | **DA** | `Self.InheritsFrom(...)` |
 Pomoćna matematička funkcija | **NE** | `static` metod |
 Konverziona funkcija | **NE** | `static` metod |
 RTTI operacije | **DA** | `Self.InstanceSize` |

**Pravilo:** Ako metod zavisi od **toga koja klasa ga poziva**, treba Self (ne-static class metod). Ako ne zavisi, koristi static.

[prev][f9] [content][f0] [next][f11]

[f0]: 00_sadrzaj.md
[f9]: 09_objekti.md
[f11]: 11_prošireni_zapisi.md
