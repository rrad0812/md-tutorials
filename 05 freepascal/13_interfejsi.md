
# 13 Interfejs

[prev][f12] [content][f0] [next][f14]

## 13.1 Definicija

Od verzije 1.1, FPC podržava interfejse. Interfejsi su alternativa višestrukom nasleđivanju (gde klasa može imati više roditeljskih klasa) kao što je implementirano, na primer, u C++. Interfejs je u osnovi imenovani skup metoda i svojstava: klasa koja implementira interfejs pruža sve metode onako kako su navedene u definiciji interfejsa. Nije moguće da klasa implementira samo deo interfejsa: to je sve ili ništa.

Interfejsi se takođe mogu uporediti u hijerarhiji, tačno kao klase: definicija interfejsa koja nasleđuje drugu definiciju interfejsa sadrži sve metode iz roditeljskog interfejsa, kao i metode eksplicitno navedene u definiciji interfejsa. Klasa koja implementira interfejs zatim mora implementirati sve članove interfejsa, kao i metode roditeljskog interfejsa (interfejsa).

Interfejs može biti jedinstveno identifikovan pomoću GUID-a. GUID je skraćenica za Globalno jedinstveni identifikator, 128-bitni ceo broj koji je garantovano uvek jedinstven .Posebno na Windows sistemima, GUID interfejsa može i mora se koristiti kada se koristi COM.

Uz ovu definiciju treba napomenuti i sledeće:

- Interfejsi se mogu koristiti samo u DELPHI režimu ili u OBJFPC režimu.
- Ne postoje specifikatori vidljivosti. Svi članovi su javni (zaista, ne bi imalo mnogo smisla učiniti ih privatnim ili zaštićenim).
- Svojstva deklarisana u interfejsu mogu imati metode samo kao specifikatore za čitanje i pisanje.
- Ne postoje konstruktori niti destruktori. Instance interfejsa ne mogu biti direktno kreirane: umesto toga, mora se kreirati instanca klase koja implementira interfejs.
- Samo modifikatori konvencije pozivanja mogu biti prisutni u definiciji metode. Modifikatori kao što su `virtuel`, `abstract` ili `dynamic`, pa samim tim ni `override`, ne mogu biti prisutni u definiciji interfejsa.

Sledeći su primeri interfejsa:

```pascal
IUnknown = interface ['{00000000-0000-0000-C000-000000000046}']  
  function QueryInterface(const iid : tguid;out obj) : longint;  
  function _AddRef : longint;  
  function _Release : longint;  
end;  
IInterface = IUnknown;  

IMyInterface = Interface  
  Function MyFunc : Integer;  
  Function MySecondFunc : Integer;  
end;
```

Kao što se može videti, GUID koji identifikuje interfejs je opcionalan.

## 13.2 Identifikacija interfejsa: GUID

Interfejs se može identifikovati pomoću GUID-a. To je 128-bitni broj, koji je predstavljen u tekstualnom obliku (stročni literal): `['{HHHHHHHH-HHHH-HHHH-HHHH-HHHHHHHHHHHH}']`

Svaki znak H predstavlja heksadecimalni broj (0–9, A–F). Format sadrži 8-4-4-4-12 brojeva. GUID takođe može biti predstavljen sledećim zapisom, definisanim u jedinici objpas (automatski se uključuje u DELPHI ili OBJFPC režimu):

```pascal
PGuid = ^TGuid;  
TGuid = packed record  
  case integer of  
    1 : (  
      Data1 : DWord;  
      Data2 : word;  
      Data3 : word;  
      Data4 : array[0..7] of byte;  
    );  
    2 : (  
      D1 : DWord;  
      D2 : word;  
      D3 : word;  
      D4 : array[0..7] of byte;  
    );  
    3 : (               { uuid fields according to RFC4122 }  
      time_low : dword;  
      time_mid : word;  
      time_hi_and_version : word;  
      clock_seq_hi_and_reserved : byte;  
      clock_seq_low : byte;  
      node : array[0..5] of byte;  
    );  
end;
```

Konstanta tipa TGUID može se odrediti korišćenjem string literala:

```pascal
{$mode objfpc}  
program testuid;  
 
Const  
  MyGUID : TGUID = '{10101010-1010-0101-1001-110110110110}';  
 
begin  
end.
```

Obično se GUID-ovi koriste samo u Windows-u, kada se koriste COM interfejsi.

## 13.3 Implementacije interfejsa

Kada klasa implementira interfejs, trebalo bi da implementira sve metode tog interfejsa. Ako metod interfejsa nije implementiran, kompajler će javiti grešku. Na primer:

```pascal
Type  
  IMyInterface = Interface  
    Function MyFunc : Integer;  
    Function MySecondFunc : Integer;  
  end;  
 
  TMyClass = Class(TInterfacedObject,IMyInterface)  
    Function MyFunc : Integer;  
    Function MyOtherFunc : Integer;  
  end;  
 
Function TMyClass.MyFunc : Integer;  
begin  
  Result:=23;  
end;  
 
Function TMyClass.MyOtherFunc : Integer;  
begin  
  Result:=24;  
end; 
```

rezultovaće u grešci:

```sh
Error: No matching implementation for interface method  
"IMyInterface.MySecondFunc:LongInt" found
```

Normalno, imena metoda koje implementiraju interfejs moraju biti jednaka imenima metoda u definiciji interfejsa. Kompilator će tražiti odgovarajuće metode u svim vidljivim metodama: metodama klase i metodama roditeljskih klasa sa zaštićenom ili višom vidljivošću.

Međutim, moguće je obezbediti alijase za metode koje čine interfejs: to jest, kompajleru se može reći da je metod interfejsa implementiran postojećom metodom sa drugim imenom. To se radi na sledeći način:

```pascal
Type  
  IMyInterface = Interface  
    Function MyFunc : Integer;  
  end;  
 
  TMyClass = Class(TInterfacedObject,IMyInterface)  
    Function MyOtherFunction : Integer;  
    Function IMyInterface.MyFunc = MyOtherFunction;  
  end; 
```

Ova deklaracija govori kompajleru da je metoda MyFunc interfejsa IMyInterface implementirana u metodi MyOtherFunction klase TMyClass.

## 13.4 Nasledjivanje interfejsa

Moguće je dozvoliti da jedan interfejs bude potomak drugog interfejsa:

```pascal
IParentInterface = interface  
  ['{0F78D56E-85A6-4024-98D7-720D7C7B9573}']  
  procedure Foo;  
end;  
 
IChildInterface = interface(IParentInterface)  
  ['{1AB2EB85-6843-462E-8CE4-32ECC065011E}']  
  procedure Bar;  
end; 
```

`IChildInterface` će imati dve metode: foo i bar. Svaka klasa koja implementira ovaj interfejs će stoga morati da implementira oba interfejsa:

```pascal
TImplementor = class(TInterfacedObject, IChildInterface)  
public  
  procedure Foo;  
  procedure Bar;  
end;  
 
procedure TImplementor.Foo;  
begin  
 
end;  
 
procedure TImplementor.Bar;  
begin  
 
end; 
```

Imajte na umu da kada klasa deklariše podređeni interfejs, on se može dodeliti promenljivoj sa tim podređenim interfejsom. S obzirom na gore navedene deklaracije, sledeće će se kompajlirati:

```pascal
var  
  Child: IChildInterface;  
 
begin  
  Child := TImplementor.Create;
```

Ali to ne znači da je automatski kompatibilno i sa dodelom promenljive tipa roditeljskog interfejsa. Sledeće se neće kompajlirati:

```pascal
var  
  Parent: IParentInterface;  
 
begin  
  Parent := TImplementor.Create;
```

Da bi se ovo kompajliralo, potrebno je deklarisati klasu kao:

```pascal
TImplementor = class(TInterfacedObject,  
                     IParentInterface,  
                     IChildInterface)  
public  
  procedure Foo;  
  procedure Bar;  
end;
```

Razlog za to je što iako klasa zapravo implementira metode IParentInterface-a , kompajler proverava samo stvarno deklarisane interfejse prilikom provere kompatibilnosti dodele: svi deklarisani interfejsi se stavljaju u tabelu i proverava se samo sadržaj te tabele.

Ista provera se vrši tokom izvršavanja programa: kompajler generiše tabelu svih interfejsa koje klasa deklariše, a ova tabela se proverava tokom izvršavanja programa. To znači da će se sledeće kompajlirati ako je deklarisan samo IChildInterface :

```pascal
ParentImplementorInstance := (TImplementor.Create as IParentInterface); 
```

i dalje će dovesti do greške tokom izvršavanja:

```sh
home:~> ./ti  
An unhandled exception occurred at $0000000000411A27:  
EInvalidCast: Invalid type cast  
$0000000000411A27
```

## 13.5 Delegiranje interfejsa

Ponekad se metode interfejsa implementiraju pomoću pomoćnog (ili delegatskog) objekta, ili je instanca klase dobila pokazivač interfejsa za ovaj interfejs i to treba koristiti. To može biti, na primer, kada se interfejs mora dodati nizu potpuno nepovezanih klasa: potrebna funkcionalnost interfejsa se dodaje posebnoj klasi, a svaka od ovih klasa koristi instancu pomoćne klase za implementaciju funkcionalnosti.

U takvom slučaju, moguće je naložiti kompajleru da interfejs nije implementiran samim objektom, već da se zapravo nalazi u pomoćnoj klasi ili interfejsu. To se može uraditi pomoću modifikatora svojstva implements .

Ako klasa ima pokazivač na željeni interfejs, sledeće će naložiti kompajleru da kada se zahteva IMyInterface interfejs, treba da koristi referencu u polju:

```pascal
type  
  IMyInterface = interface  
    procedure P1;  
  end;  
 
  TMyClass = class(TInterfacedObject, IMyInterface)  
  private  
    FMyInterface: IMyInterface; // interface type  
  public  
    property MyInterface: IMyInterface  
       read FMyInterface implements IMyInterface;  
  end;
```

Interfejs ne mora nužno biti u polju, može se koristiti bilo koji identifikator za čitanje.

Ako je interfejs implementiran pomoću objekta delegata (pomoćnog objekta koji zapravo implementira interfejs), onda se može koristiti i sa ključnom reči implements :

```pascal
{$interfaces corba}  

type  
  IMyInterface = interface  
    procedure P1;  
  end;  
 
  // NOTE: Interface must be specified here  
  TDelegateClass = class(TObject, IMyInterface)  
  private  
    procedure P1;  
  end;  
 
  TMyClass = class(TInterfacedObject, IMyInterface)  
  private  
    FMyInterface: TDelegateClass; // class type  
    property MyInterface: TDelegateClass  
      read FMyInterface implements IMyInterface;  
  end;
```

Treba napomenuti da, za razliku od Delfija, klasa delegata mora eksplicitno da navede interfejs: kompajler neće tražiti metode u klasi delegata, već će jednostavno proveriti da li klasa delegata implementira navedeni interfejs.

Moguće je implementirati više interfejsa koristeći jedan delegirani objekat:

```pascal
{$interfaces corba}  

type  
  IMyInterface = interface  
    procedure P1;  
  end;  
  IMyInterface1 = interface  
    procedure P2;  
  end;  
 
  // NOTE: Interface must be specified here  
  TDelegateClass = class(TObject, IMyInterface,IMyInterface1)  
  private  
    procedure P1;  
    procedure P2;  
  end;  
 
  TMyClass = class(TInterfacedObject, IMyInterface, IMyInterface1)  
  private  
    FMyInterface: TDelegateClass; // class type  
    property MyInterface: TDelegateClass  
      read FMyInterface implements IMyInterface,IMyInterface1;  
  end;
```

Nije moguće mešati rešavanje metoda i delegiranje interfejsa. To znači da nije moguće implementirati deo interfejsa putem rešavanja metoda, a deo interfejsa putem delegiranja. Sledeći pokušaji implementacije IMyInterface- a delimično putem rešavanja metoda (P1), a delimično putem delegiranja. Kompilator neće prihvatiti sledeći kod:

```pascal
{$interfaces corba}  

type  
  IMyInterface = interface  
    procedure P1;  
    procedure P2;  
  end;  
 
  TMyClass = class(TInterfacedObject, IMyInterface)  
    FI : IMyInterface;  
  protected  
    procedure IMyInterface.P1 = MyP1;  
    procedure MyP1;  
    property MyInterface: IMyInterface  read FI implements IMyInterface;  
  end;
```

Kompilator će izbaciti grešku:

```sh
Error: Interface "IMyInterface" can't be delegated by "TMyClass",  
it already has method resolutions
```

Međutim, moguće je implementirati jedan interfejs putem rešavanja metoda, a drugi putem delegiranja:

```pascal
{$interfaces corba}  

type  
  IMyInterface = interface  
    procedure P1;  
  end;  
 
  IMyInterface2 = interface  
    procedure P2;  
  end;  
 
  TMyClass = class(TInterfacedObject,  
                   IMyInterface, IMyInterface2)  
    FI2 : IMyInterface2;  
  protected  
    procedure IMyInterface.P1 = MyP1;  
    procedure MyP1;  
  public  
    property MyInterface: IMyInterface2  
       read FI2 implements IMyInterface2;  
  end;
```

Imajte na umu da se delegiranje interfejsa može koristiti da bi se navelo da klasa implementira nadređene interfejse:

```pascal
IGMGetFileName = interface(IUnknown)  
  ['{D3ECCB42-A563-4cc4-B375-79931031ECBA}']  
  function GetFileName: String; stdcall;  
  property FileName: String read GetFileName;  
end;  
 
IGMGetSetFileName = Interface(IGMGetFileName)  
  ['{ECFB879F-86F6-41a3-A685-0C899A2B5BCA}']  
  procedure SetFileName(const Value: String); stdcall;  
  property FileName: String read GetFileName write SetFileName;  
end;  
 
TIntfDelegator = class(TInterfacedObject, IGMGetFileName, IGMGetSetFileName)  
 protected  
  FGetSetFileName: IGMGetSetFileName;  
 public  
  constructor Create;  
  destructor Destroy; override;  
  property Implementor: IGMGetSetFileName read FGetSetFileName  
    implements IGMGetFileName, IGMGetSetFileName;  
end; 
```

## 13.6 Interfejsi i COM

Kada se koriste interfejsi na Windows-u koji bi trebalo da budu dostupni COM podsistemu, konvencija pozivanja treba da bude stdcall  – ovo nije podrazumevana konvencija pozivanja Free Pascal-a, tako da bi trebalo da bude eksplicitno navedena.

COM ne poznaje svojstva. Poznaje samo metode. Zato, kada navodite definicije svojstava kao deo definicije interfejsa, imajte na umu da će svojstva biti poznata samo u programu kompajliranom u Free Pascal-u: drugi Windows programi neće biti svesni definicija svojstava.

## 13.7  CORBA i drugi interfejsi

COM nije jedina arhitektura gde se koriste interfejsi. CORBA poznaje interfejse, UNO (OpenOffice API) koristi interfejse, kao i Java. Ovi jezici ne poznaju IUnknown interfejs koji se koristi kao osnova svih interfejsa u COM-u. Stoga bi bila loša ideja da interfejs automatski proizilazi iz IUnknown-a ako nije naveden roditeljski interfejs. Stoga je u Free Pascal-u uvedena direktiva { $ INTERFACES } : ona određuje šta je roditeljski interfejs interfejsa, deklarisanog bez roditelja. Više informacija o ovoj direktivi možete pronaći u Programerskom vodiču .

Imajte na umu da se COM interfejsi podrazumevano broje kao reference, jer potiču od IUnknown .

Corba interfejsi se identifikuju jednostavnim stringom, tako da su kompatibilni sa dodelom sa stringovima, a ne sa TGUID-om . Kompilator ne vrši automatsko brojanje referenci za CORBA interfejse, tako da je programer odgovoran za vođenje knjigovodstva referenci.

## 13.8 Brojanje referenci

Svi COM interfejsi koriste brojanje referenci. To znači da kad god se interfejs dodeli promenljivoj, njegov broj referenci se ažurira. Kad god promenljiva izađe iz opsega važenja, broj referenci se automatski smanjuje. Kada broj referenci dostigne nulu, obično se oslobađa instanca klase koja implementira interfejs.

Mora se voditi računa o ovom mehanizmu. Kompilator može, ali i ne mora da kreira privremene promenljive prilikom izvršavanja izraza i dodeli interfejs privremenoj promenljivoj, a tek onda dodeli privremenu promenljivu stvarnoj rezultujućoj promenljivoj. Ne treba praviti nikakve pretpostavke o broju privremenih promenljivih ili vremenu kada su finalizovane – ovo se može (i zaista se razlikuje) od načina na koji drugi kompilatori (npr. Delphi) obrađuju izraze sa interfejsima. Npr. pretvaranje tipa je takođe izraz:

```pascal
Var  
  B : AClass;  
 
begin  
  // ...  
  AInterface(B.Intf).testproc;  
  // ...  
end;
```

Pretpostavimo da se interfejs intf broji po referencama. Kada kompajler izvršava B.Intf , on kreira privremenu promenljivu. Ova promenljiva može biti oslobođena samo kada se procedura završi: stoga je nevažeće, na primer, osloboditi instancu B pre izlaska iz procedure, jer kada se privremena promenljiva finalizuje, pokušaće se ponovo osloboditi B.

Pored toga, rezultati funkcije mogu pri unosu ukazivati na validan COM interfejs koji nije nil: to je zato što se rezultat funkcije tretira kao var parametar.

[prev][f12] [content][f0] [next][f14]

[f0]: 00_sadrzaj.md
[f12]: 12_pomoćnici.md
[f14]: 14_generici.md
