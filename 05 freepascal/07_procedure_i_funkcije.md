
# 7 Korišćenje funkcija i procedura

[prev][f6] [content][f0] [next][f8]

FreePaskal podržava upotrebu funkcija i procedura. Podržava

- Preopterećenje funkcija, tj. funkcije sa istim imenom, ali različitim listama parametara.
- Konstantne parametre.
- Otvorene nizove (tj. nizovi bez granica).
- Promenljivi broj argumenata kao u C-u.
- Konstrukcija slična Return-u kao u C-u, kroz ključnu reč `Exit`.

**Napomena**  
U mnogim narednim pasusima reči procedura i funkcija će se koristiti naizmenično. Navedene izjave važe za obe, osim ako nije drugačije naznačeno.

## 7.1 Deklaracija

Deklaracija procedure definiše identifikator i povezuje ga sa blokom koda. Procedura se zatim može pozvati pomoću izjave procedure.

Deklaracija procedure nakon koje sledi blok implementira radnju procedure u tom bloku. Sledeća procedura je validna:

```pascal
Procedure DoSomething (Para : String);  
begin  
  Writeln ('Got parameter : ',Para);  
  Writeln ('Parameter in upper case : ',Upper(Para));  
end;
```

Imajte na umu da je moguće da procedura poziva samu sebe.

## 7.2 Deklaracija funkcije

Deklaracija funkcije definiše identifikator i povezuje ga sa blokom koda. Blok koda će vratiti rezultat. Funkcija se zatim može pozvati unutar izraza ili pomoću izjave, ako je uključena proširena sintaksa.

Rezultatski tip funkcije može biti bilo koji prethodno deklarisani tip, suprotno Turbo Paskalu, gde su se mogli vratiti samo jednostavni tipovi.

## 7.3 Rezultati funkcija

Rezultat funkcije može se podesiti postavljanjem promenljive `result` ili to može biti identifikator funkcije ili (samo u ObjFPC ili Delphi režimu) poseban identifikator rezultata:

```pascal
Function MyFunction : Integer;  
begin  
  MyFunction:=12; // Return 12  
end;
```

U Delphi ili ObjFPC režimu, gore navedeno se može kodirati i kao:

```pascal
Function MyFunction : Integer;  
begin  
  Result:=12;  
end;
```

Kao proširenje Delfi sintakse, ObjFPC režim takođe podržava posebno proširenje procedure ključnom reči `Exit` :

```pascal
Function MyFunction : Integer;  
begin  
  Exit(12);  
end;
```

Poziv `Exit` postavlja rezultat funkcije i prelazi na krajnji blok deklaracije funkcije. Može se posmatrati kao ekvivalent C instrukcije return.

**Napomena**  
Rezultati funkcija se tretiraju kao parametri koji se prenose po referenci. To je posebno važno za upravljane tipove: Rezultat funkcije može biti različit od nil pri ulasku i postavljen na validnu instancu tipa.

## 7.4 Liste parametara

Kada se argumenti moraju proslediti funkciji ili proceduri, ovi parametri moraju biti deklarisani u formalnoj listi parametara te funkcije ili procedure. Lista parametara je deklaracija identifikatora na koje se može pozivati samo u toj proceduri ili bloku funkcije.

Konstantni parametri, izlazni parametri i promenljivi parametri takođe mogu biti netipizovani parametri ako nemaju identifikator tipa.

Od verzije 1.1, FreePascal podržava podrazumevane vrednosti i za konstantne parametre i za vrednosne parametre, ali samo za jednostavne tipove. Kompilator mora biti u OBJFPC ili DELPHI režimu da bi prihvatio podrazumevane vrednosti.

### 7.4.1 Vrednosni parametri

Kada se parametri deklarišu kao vrednosni parametri, procedura dobija kopiju parametara koje prosleđuje pozivajući izraz. Bilo kakve izmene ovih parametara su isključivo lokalne za blok procedure i ne šire se nazad na pozivajući blok.

Blok koji želi da pozove proceduru sa vrednosnim parametrima mora da prosledi proceduri parametre kompatibilne sa dodelom. To znači da tipovi ne bi trebalo da se potpuno podudaraju, već se mogu konvertovati u stvarne tipove parametara. Ovaj kod za konverziju ubacuje sam kompajler.

**Napomena**  
Mora se voditi računa pri korišćenju vrednosnih parametara: vrednosni parametri intenzivno koriste stek, posebno kada se koriste veliki parametri. Ukupna veličina svih parametara u formalnoj listi parametara treba da bude manja od 32K radi prenosivosti (Intelova verzija ovo ograničava na 64K).

Otvoreni nizovi mogu se prosleđivati kao vrednosni parametri.

Za parametar jednostavnog tipa (tj. ne strukturiranog tipa), može se navesti podrazumevana vrednost. To može biti netipizovana konstanta. Ako poziv funkcije izostavi parametar, podrazumevana vrednost će biti prosleđena funkciji. Za dinamičke nizove ili druge tipove koji se mogu smatrati ekvivalentnim pokazivaču, jedina moguća podrazumevana vrednost je Nil.

Sledeći primer će ispisati 20 na ekranu:

```pascal
program testp;  

Const  
  MyConst = 20;  

Procedure MyRealFunc(I : Integer = MyConst);  
begin  
  Writeln('Function received : ',I);  
end;  

begin  
  MyRealFunc;  
end.
```

### 7.4.2 Promenljivi parametri

Kada se parametri deklarišu kao promenljivi parametri, procedura ili funkcija odmah pristupa promenljivoj koju je pozivni blok prosledio u svojoj listi parametara. Procedura dobija pokazivač na promenljivu koja je prosleđena i koristi taj pokazivač za pristup vrednosti promenljive. Iz ovoga sledi da će se sve promene napravljene na parametru vratiti nazad na pozivni blok. Ovaj mehanizam se može koristiti za vraćanje vrednosti u procedurama. Zbog toga, pozivni blok mora proslediti parametar potpuno istog tipa kao i tip deklarisanog parametra. Ako to ne učini, kompajler će generisati grešku.

Promenljivi i konstantni parametri mogu biti netipizovani. U tom slučaju promenljiva nema tip i stoga je nekompatibilna sa svim ostalim tipovima. Međutim, operator adresiranja se može koristiti na njoj ili se može proslediti funkciji koja takođe ima netipizovani parametar. Ako se netipizovani parametar koristi u dodeli ili mu se mora dodeliti vrednost, mora se koristiti pretvaranje tipa u tip.

Promenljive tipa datoteke moraju uvek biti prosleđene kao parametri promenljivih.

Otvoreni nizovi se mogu prosleđivati kao promenljivi parametri.

**Primedba**  

Imajte na umu da podrazumevane vrednosti nisu podržane za promenljive parametre. Ovo ne bi imalo
mnogo smisla jer poništava svrhu mogućnosti vraćanja vrednosti pozivaocu.

Rezultat funkcije se interno tretira kao promenljivi parametar i može imati početnu vrednost
različitu od nule (ili ne-nil). Ovo je posebno važno za upravljane tipove.

### 7.4.3 Izlazni parametri

Svrha izlaznog parametra je da vrati vrednosti pozivajućoj rutini: promenljiva se prosleđuje referencom. Početna vrednost parametra pri ulasku u funkciju se odbacuje i ne bi trebalo da se koristi.

Ako se promenljiva mora koristiti za prosleđivanje vrednosti funkciji i preuzimanje podataka iz funkcije, onda se mora koristiti promenljivi parametar. Ako se mora preuzeti samo vrednost, može se koristiti parametar izlaza (out) .

Nepotrebno je reći da podrazumevane vrednosti nisu podržane za izlazne parametre.

Razlika između izlaznih parametara i parametara po referenci je veoma mala (međutim, videti dole za upravljane tipove): prvi daju kompajleru više informacija o tome šta se dešava sa argumentima kada se prosleđuju proceduri: on zna da promenljiva ne mora biti inicijalizovana pre poziva. Sledeći primer ilustruje ovo:

```pascal
Procedure DoA(Var A : Integer);  
begin  
  A:=2;  
  Writeln('A is ',A);  
end;  

Procedure DoB(Out B : Integer);  
begin  
  B:=2;  
  Writeln('B is ',B);  
end;  

Var  
  C,D : Integer;  

begin  
  DoA(C);  
  DoB(D);  
end.
```

Obe procedure, DoA i DoB, rade praktično isto. Ali deklaracija DoB daje više informacija kompajleru, omogućavajući mu da detektuje da D ne mora biti inicijalizovan pre nego što se pozove DoB. Pošto parametar A u DoA može primiti vrednost kao i vratiti je, kompajler primećuje da C nije bio inicijalizovan pre poziva DoA :

```sh
početna: >fpc -S2 -vwhn testo.pp  
testo.pp(19,8) Savet: Promenljiva „C“ izgleda nije inicijalizovana
```

Ovo pokazuje da je bolje koristiti parametre `out` kada se parametar koristi samo za vraćanje vrednosti.

**Napomena**  
Parametri `Out` su podržani samo u Delphi i ObjFPC režimu. Za ostale režime, `out` je validan identifikator.

**Napomena**  
Za upravljane tipove (tipove sa brojanjem referenci), korišćenje parametara `Out` stvara određeno opterećenje: kompajler mora biti siguran da je vrednost ispravno inicijalizovana (tj. da ima broj referenci nula (0)). Ovu inicijalizaciju obično vrši pozivalac.

### 7.4.4 Konstantni parametri

Pored promenljivih parametara i vrednosnih parametara, Free Pascal takođe podržava konstantne parametre.

Određivanje parametra kao konstante daje kompajleru naznaku da sadržaj parametra neće biti promenjen pozvanom rutinom. Ovo omogućava kompajleru da izvrši optimizacije koje inače ne bi mogao, a takođe i da izvrši određene provere koda unutar rutine: naime, može zabraniti dodeljivanje parametru. Štaviše, konstantni parametar ne može biti prosleđen drugoj funkciji koja zahteva promenljivi parametar: kompajler može i ovo da proveri. Glavna upotreba ovoga je smanjenje veličine steka, time poboljšanje performansi, a i dalje zadržavanje semantike prosleđivanja vrednosti...

**Napomena**  
Suprotno Delfiju, ne treba praviti nikakve pretpostavke o tome kako se konstantni parametri prenose osnovnoj rutini. Posebno, pretpostavka da se parametri velike veličine prenose referencom nije tačna. Za ovo treba koristiti tip parametra `constref`, koji je dostupan od verzije 2.5.1 kompajlera.

Izuzetak je konvencija pozivanja `stdcall`-a : radi kompatibilnosti sa COM standardima, veliki konstantni parametri se prosleđuju referencom.

**Napomena**  
Treba napomenuti da je navođenje parametra `const` ugovor između programera i kompajlera. Programer je taj koji govori kompajleru da se sadržaj parametra `const` neće menjati kada se rutina izvrši, a ne kompajler programeru da se parametar neće menjati.

Ovo je posebno važno i vidljivo kada se koriste tipovi sa referencama. Za takve tipove, (nevidljivo) povećanje i smanjenje bilo kog brojača referenci se izostavlja kada se koristi `const`. To često omogućava kompajleru da izostavi nevidljive `try/finally` okvire za ove rutine.

Kao sporedni efekat, sledeći kod neće proizvesti očekivani rezultat:

```pascal
Var  
  S : String = 'Something';  
 
Procedure DoIt(Const T : String);  
begin  
  S:='Something else';  
  Writeln(T);  
end;  
 
begin  
  DoIt(S);  
end.
```

Izlaz:

```pascal
Something else
```

Ovakvo ponašanje je by design.

Konstantni parametri takođe mogu biti netipizovani.

Što se tiče vrednosnih parametara, konstantni parametri mogu dobiti podrazumevane vrednosti.

Otvoreni nizovi mogu se prosleđivati kao konstantni parametri.

### 7.4.5 Parametri otvorenog niza

FreePaskal podržava prenos otvorenih nizova, tj. procedura se može deklarisati sa nizom neodređene dužine kao parametrom, kao u Delfiju. Parametrima otvorenog niza može se pristupiti u proceduri ili funkciji kao nizu koji je deklarisan sa početnim indeksom 0 i indeksom poslednjeg elementa High(parametar) . Na primer, parametar

```pascal
Row : Array of Integer;
```

bilo bi ekvivalentno

```pascal
Row : Array[0..N-1] of Integer; 
```

Gde bi N bila stvarna veličina niza koja se prosleđuje funkciji. N-1 se može izračunati kao `High(Row)`.

Konkretno, ako se prosleđuje prazan niz, onda High(Parameter) vraća -1, dok low(Parameter) vraća 0.

Otvoreni parametri mogu se prosleđivati po vrednosti, po referenci ili kao konstantni parametar. U ovom drugom slučaju, procedura dobija pokazivač na stvarni niz. U prvom slučaju, dobija kopiju niza. U funkciji ili proceduri, otvoreni nizovi mogu se prosleđivati samo funkcijama koje su takođe deklarisane sa otvorenim nizovima kao parametrima, a ne funkcijama ili procedurama koje prihvataju nizove fiksne dužine. Sledi primer funkcije koja koristi otvoreni niz:

```pascal
Function Average (Row : Array of integer) : Real;  
Var I : longint;  
    Temp : Real;  
begin  
  Temp := Row[0];  
  For I := 1 to High(Row) do  
    Temp := Temp + Row[i];  
  Average := Temp / (High(Row)+1);  
end;
```

Od FPC 2.2, takođe je moguće proslediti delimične nizove funkciji koja prihvata otvoreni niz. To se može uraditi navođenjem opsega niza koji treba proslediti otvorenom nizu.

S obzirom na deklaraciju

```pascal
Var  
  A : Array[1..100];
```

Sledeći poziv će izračunati i ispisati prosek 100 brojeva:

```pascal
Writeln('Average of 100 numbers: ',Average(A));
```

Ali sledeće će izračunati i ispisati prosek prve i druge polovine:

```pascal
Writeln('Average of first 50 numbers: ',Average(A[1..50]));  
Writeln('Average of last  50 numbers: ',Average(A[51..100]));
```

### 7.4.6 Niz konstanti

U ObjPas ili Delphi režimu, FreePaskal podržava konstrukciju niza konstanti (Array of Const) za prosleđivanje parametara potprogramu unapred.

Ovo je poseban slučaj konstrukcije otvorenog niza, gde je dozvoljeno proslediti bilo koji izraz u nizu funkciji ili proceduri. Izraz mora imati jednostavan tip rezultata: strukture se ne mogu proslediti kao argument. To znači da se mogu proslediti svi ordinalni, float ili string tipovi, kao i pokazivači, klase i interfejsi (pošto su poslednja dva zapravo pokazivači).

Elementi niza const se konvertuju u poseban zapis varijante:

```pascal
Type  
  PVarRec = ^TVarRec;  
  TVarRec = record  
     case VType : Ptrint of  
       vtInteger    : (VInteger: Longint);  
       vtBoolean    : (VBoolean: Boolean);  
       vtChar       : (VChar: Char);  
       vtWideChar   : (VWideChar: WideChar);  
       vtExtended   : (VExtended: PExtended);  
       vtString     : (VString: PShortString);  
       vtPointer    : (VPointer: Pointer);  
       vtPChar      : (VPChar: PChar);  
       vtObject     : (VObject: TObject);  
       vtClass      : (VClass: TClass);  
       vtPWideChar  : (VPWideChar: PWideChar);  
       vtAnsiString : (VAnsiString: Pointer);  
       vtCurrency   : (VCurrency: PCurrency);  
       vtVariant    : (VVariant: PVariant);  
       vtInterface  : (VInterface: Pointer);  
       vtWideString : (VWideString: Pointer);  
       vtInt64      : (VInt64: PInt64);  
       vtQWord      : (VQWord: PQWord);  
   end;
```

Stoga, unutar tela procedure, niz konstantnih argumenata je ekvivalentan otvorenom nizu TVarRec :
Procedura Testit (Args: Niz konstanti);  

```pascal
Procedure Testit (Args: Array of const);  

Var I : longint;  

begin  
  If High(Args)<0 then  
    begin  
    Writeln ('No aguments');  
    exit;  
    end;  
  Writeln ('Got ',High(Args)+1,' arguments :');  
  For i:=0 to High(Args) do  
    begin  
    write ('Argument ',i,' has type ');  
    case Args[i].vtype of  
      vtinteger    :  
        Writeln ('Integer, Value :',args[i].vinteger);  
      vtboolean    :  
        Writeln ('Boolean, Value :',args[i].vboolean);  
      vtchar       :  
        Writeln ('Char, value : ',args[i].vchar);  
      vtextended   :  
        Writeln ('Extended, value : ',args[i].VExtended^);  
      vtString     :  
        Writeln ('ShortString, value :',args[i].VString^);  
      vtPointer    :  
        Writeln ('Pointer, value : ',Longint(Args[i].VPointer));  
      vtPChar      :  
        Writeln ('PChar, value : ',Args[i].VPChar);  
      vtObject     :  
        Writeln ('Object, name : ',Args[i].VObject.Classname);  
      vtClass      :  
        Writeln ('Class reference, name :',Args[i].VClass.Classname);  
      vtAnsiString :  
        Writeln ('AnsiString, value :',AnsiString(Args[I].VAnsiString);  
    else  
        Writeln ('(Unknown) : ',args[i].vtype);  
    end;  
    end;  
end;
```

U kodu je moguće proslediti proizvoljan niz elemenata ovoj proceduri:

```pascal
S:='Ansistring 1';  
T:='AnsiString 2';  
Testit ([]);  
Testit ([1,2]);  
Testit (['A','B']);  
Testit ([TRUE,FALSE,TRUE]);  
Testit (['String','Another string']);  
Testit ([S,T])  ;  
Testit ([P1,P2]);  
Testit ([@testit,Nil]);  
Testit ([ObjA,ObjB]);  
Testit ([1.234,1.234]);  
TestIt ([AClass]);
```

Ako je procedura deklarisana sa modifikatorom cdecl, onda će kompajler proslediti niz kao što bi ga prosledio C kompajler. Ovo, u stvari, emulira C konstrukciju promenljivog broja argumenata, kao što će pokazati sledeći primer:

```pascal
program testaocc;  
{$mode objfpc}  

Const  
  P : PChar = 'example';  
  Fmt : PChar = 'This %s uses printf to print numbers (%d) and strings.'#10;  

// Declaration of standard C function printf:  
procedure printf (fm : pchar; args : array of const);cdecl; external 'c';  

begin  
  printf(Fmt, [P,123]);  
end.
```

**Napomena**  
Treba napomenuti da ovo ne važi za Delfi, tako da kod koji se oslanja na ovu funkciju neće biti prenosiv.

**Napomena**  
Imajte na umu da ne postoji podrška za DWord (ili Cardinal) argumente u nizu const. Oni se konvertuju u vtInteger / vtLongint . Ovo je radi kompatibilnosti sa Delphi-jem, a kompajler će ignorisati sve rezultujuće provere opsega kada je u Delphi režimu.

### 7.4.7 Netipizovani parametri

Parametri `Var`, `Out` i `Const` mogu biti netipizovani. U tom slučaju, promenljiva nema tip u proceduri funkcije i stoga je nekompatibilna sa svim ostalim tipovima: Kompilator jednostavno prosleđuje adresu prosleđene promenljive rutini, tako da je sve što je dostupno u pozvanoj rutini adresa, bez ikakvih informacija o tipu. Ovo važi i za konstantne parametre.

Korišćenje analogije

```pascal
procedure Something(const Data; Len: cint);
```

Ekvivalentno je sledećoj C/C++ deklaraciji:

```pascal
void Something(void* Data; int Len);
```

To znači da je praktično sve što se može uraditi u rutini koristiti operator adresiranja ili proslediti argument drugoj funkciji koja takođe ima netipizovani parametar.

Unutar rutine sa netipizovanim parametrom, ako se netipizovani parametar koristi u izrazu ili mu se mora dodeliti vrednost, uvek se mora koristiti pretvaranje tipa.

Sledeći primer to pokazuje:

```pascal
{$mode objfpc}  
uses types;  

procedure doit(const d);  

begin  
  Writeln('As integer: ',PInteger(@D)^);  
  Writeln('As Byte   : ',PByte(@D)^);  
end;  

Var  
  D : Integer;  

begin  
  D:=$0FFFFFF0;  
  DoIt(D);  
end.
```

Ovo će pisati:

```pascal
As integer: 268435440  
As Byte   : 240
```

Imajte na umu da, pošto je potrebna adresa, funkciji ili proceduri ne mogu biti prosleđeni konstantni izrazi, tj. s obzirom na gornju definiciju, sledeće neće raditi:

```pascal
DoIt($0FFFFFF0);
```

Ovo će rezultirati sledećom greškom:

```sh
Error: Variable identifier expected
```

### 7.4.8 Upravljani tipovi i broj referenci

Neki tipovi (Unicodestring, Ansistring, interfejsi, dinamički nizovi) se tretiraju donekle posebno od strane kompajlera: podaci imaju broj referenci koji se povećava ili smanjuje u zavisnosti od toga koliko referenci na podatke postoji.

Kvalifikatori za parametre u pozivima funkcija ili procedura utiču na to šta se dešava sa brojem referenci upravljanih tipova:

- ništa (prenos po vrednosti): broj referenci parametra se povećava pri ulasku i smanjuje pri izlasku.
- izlaz : broj referenci vrednosti koja se prosleđuje se smanjuje za 1, a promenljiva koja se prosleđuje u proceduru se inicijalizuje na „prazno“ (obično Nil , ali to je detalj implementacije na koji se ne treba oslanjati).
- var ništa se ne dešava sa brojačem referenci. Referenca na originalnu promenljivu se prosleđuje, a njena promena ili čitanje ima potpuno isti efekat kao promena/čitanje originalne promenljive.
- Ovaj slučaj sa const je malo komplikovan. Ništa se ne dešava sa brojačem referenci jer ovde možete proslediti nevrednosti. Konkretno, možete proslediti klasu koja implementira interfejs umesto samog interfejsa, što može dovesti do neočekivanog oslobađanja klase.

**Napomena**  
Rezultat funkcije se interno tretira kao var parametar funkcije i primenjuju se ista pravila kao i za var parametre.

Sledeći primer pokazuje opasnosti:

```pascal
{$mode objfpc}  

Type  
  ITest = Interface  
    Procedure DoTest(ACount : Integer);  
  end;  

  TTest = Class(TInterfacedObject,ITest)  
    Procedure DoTest(ACount : Integer);  
    Destructor destroy; override;  
  end;  

Destructor TTest.Destroy;  
begin  
  Writeln('Destroy called');  
end;  

Procedure TTest.DoTest(ACount : Integer);  
begin  
  Writeln('Test ',ACount,' : ref count: ',RefCount);  
end;  

procedure DoIt1(x: ITest; ACount : Integer);  
begin  
  // Reference count is increased  
  x.DoTest(ACount);  
  // And decreased  
end;  

procedure DoIt2(const x: ITest; ACount : Integer);  
begin  
  // No change to reference count.  
  x.DoTest(ACount);  
end;  

Procedure Test1;  
var  
  y: ITest;  
begin  
  y := TTest.Create;  
  // Ref. count is 1 at this point.  
  y.DoTest(1);  
  // Calling DoIT will increase reference count and decrease on exit.  
  DoIt1(y,2);  
  // Reference count is still one.  
  y.DoTest(3);  
end;  

Procedure Test2;  
var  
  Y : TTest;  
begin  
  Y := TTest.Create; // no count on the object yet  
  // Ref. count is 0 at this point.  
  y.DoTest(3);  
  // Ref count will remain zero.  
  DoIt2(y,4);  
  Y.DoTest(5);  
  Y.Free;  
end;  

Procedure Test3;  
var  
  Y : TTest;  
begin  
  Y := TTest.Create; // no count on the object yet  
  // Ref. count is 0 at this point.  
  y.DoTest(6);  
  // Ref count will remain zero.  
  DoIt1(y,7);  
  y.DoTest(8);  
end;  

begin  
  Test1;  
  Test2;  
  Test3;  
end.
```

Izlaz ovog primera je:

```sh
Test 1 : ref count: 1  
Test 2 : ref count: 2  
Test 3 : ref count: 1  
Destroy called  
Test 3 : ref count: 0  
Test 4 : ref count: 0  
Test 5 : ref count: 0  
Destroy called  
Test 6 : ref count: 0  
Test 7 : ref count: 1  
Destroy called  
Test 8 : ref count: 0
```

Kao što se može videti, u test3, broj referenci se smanjuje sa 1 na 0 na kraju DoIt poziva, što uzrokuje oslobađanje instance pre nego što se poziv vrati.

Sledeći mali program demonstrira brojanje referenci koje se koristi u stringovima:

```pascal
{$mode objfpc}  
{$H+}  

// Auxiliary function to extract reference count.  
function SRefCount(P : Pointer) : integer;  
Type  
  PAnsiRec = ^TAnsiRec;  
  TAnsiRec = Record  
    CodePage    : TSystemCodePage;  
    ElementSize : Word;  
  {$ifdef CPU64}  
  { align fields  }  
    Dummy       : DWord;  
  {$endif CPU64}  
    Ref         : SizeInt;  
    Len         : SizeInt;  
  end;  
begin  
  if P=Nil then  
    Result:=0  
  else  
    Result:=PAnsiRec(P-SizeOf(TAnsiRec))^.Ref;  
end;  

Procedure ByVar(Var S : string);  
begin  
  Writeln('By var, ref count : ',SRefCount(Pointer(S)));  
end;  

Procedure ByConst(Const S : string);  
begin  
  Writeln('Const, ref count : ',SRefCount(Pointer(S)));  
end;  

Procedure ByVal(S : string);  
begin  
  Writeln('Value, ref count : ',SRefCount(Pointer(S)));  
end;  

Function FunctionResult(Var S : String) : String;  
begin  
  Writeln('Function argument, ref count : ',SRefCount(Pointer(S)));  
  Writeln('Function result, ref count : ',SRefCount(Pointer(Result)));  
end;  

Var  
  S,T : String;  

begin  
  S:='Some string';  
  Writeln('Constant       : ',SrefCount(Pointer(S)));  
  UniqueString(S);  
  Writeln('Unique         : ',SRefCount(Pointer(S)));  
  T:=S;  
  Writeln('After Assign   : ',SRefCount(Pointer(S)));  
  ByVar(S);  
  ByConst(S);  
  ByVal(S);  
  UniqueString(S);  
  T:=FunctionResult(S);  
  Writeln('After function : ',SRefCount(Pointer(S)));  
end.
```

[prev][f6] [content][f0] [next][f8]

[f0]: 00_sadrzaj.md
[f6]: 06_izjave.md
[f8]: 08_propterećenje_operatora.md
