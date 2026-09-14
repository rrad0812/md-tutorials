
# 4 Promenljive

[prev][f3] [content][f0] [next][f5]

## 4.1 Definicija

Promenljive su eksplicitno imenovane memorijske lokacije određenog tipa. Prilikom dodeljivanja vrednosti promenljivim, kompajler Free Pascal-a generiše mašinski kod za premeštanje vrednosti na memorijsku lokaciju rezervisanu za ovu promenljivu. Gde se ova promenljiva čuva zavisi od toga gde je deklarisana:

- **Globalne promenljive** su promenljive deklarisane u jedinici ili programu, ali ne unutar
  procedure ili funkcije. One se čuvaju na fiksnim memorijskim lokacijama i dostupne su tokom celog vremena izvršavanja programa.
- **Lokalne promenljive** se deklarišu unutar procedure ili funkcije. NJihova vrednost se čuva na
  programskom steku, tj. ne na fiksnim lokacijama.

Kompilator FreePaskala transparentno obrađuje alokaciju ovih memorijskih lokacija, iako se na ovu lokaciju može uticati u deklaraciji.

Kompilator FreePascal-a takođe transparentno obrađuje čitanje vrednosti iz promenljivih ili pisanje vrednosti u njih. Ali čak i ovo programer može eksplicitno da obradi kada koristi svojstva.

Promenljive moraju biti eksplicitno deklarisane kada su potrebne. Memorija se ne dodeljuje osim ako se promenljiva ne deklariše. Korišćenje identifikatora promenljive (na primer, promenljive petlje) koji nije prvo deklarisan predstavlja grešku koju će prijaviti kompajler.

## 4.2 Declaracija

Promenljive moraju biti deklarisane u bloku deklaracije promenljivih unita ili procedure ili funkcije.

To znači da su sledeće validne deklaracije promenljivih:

```pascal
Var  
  curterm1 : integer;  // #1

  curterm2 : integer; cvar; // #2  
  curterm3 : integer; cvar; external;  // #3

  curterm4 : integer; external name 'curterm3';  // #4
  curterm5 : integer; external 'libc' name 'curterm9';  // #5

  curterm6 : integer absolute curterm1;  // #6

  curterm7 : integer; cvar;  export;  // #7
  curterm8 : integer; cvar;  public;  // #8
  curterm9 : integer; export name 'me';  // #9
  curterm10 : integer; public name 'ma';  // #10

  curterm11 : integer = 1; // #11
```

Razlika između ovih deklaracija je sledeća:

- Prvi oblik ( curterm1 ) definiše regularnu promenljivu. Kompilator sve sam upravlja.

- Drugi oblik ( curterm2 ) takođe deklariše regularnu promenljivu, ali navodi da je asemblersko ime
  za ovu promenljivu jednako imenu promenljive kako je napisano u izvornom kodu.

- Treći oblik ( curterm3 ) deklariše promenljivu koja se nalazi eksterno: kompajler će pretpostaviti
  da se memorija nalazi negde drugde i da je asemblersko ime ove lokacije određeno imenom promenljive, kao što je napisano u izvornom kodu. Ime se ne sme navesti.

- Četvrti oblik je potpuno ekvivalentan trećem, deklariše promenljivu koja se čuva eksterno i
  eksplicitno daje asemblerski naziv lokacije. Ako se cvar ne koristi, naziv mora biti naveden.

- Peti oblik je varijanta četvrtog oblika, samo je navedeno i ime biblioteke u kojoj je memorija
  rezervisana.

- Šesti oblik deklariše promenljivu ( curterm6 ) i govori kompajleru da je ona sačuvana na istoj
  lokaciji kao i druga promenljiva ( curterm1 ).

- Sedmi oblik deklariše promenljivu ( curterm7 ) i govori kompajleru da asemblerska oznaka ove
  promenljive treba da bude ime promenljive (razlikuje velika i mala slova) i da mora biti javna, tj. može se pozivati na nju iz drugih objektnih datoteka.

- Osmi oblik ( curterm8 ) je ekvivalentan sedmom: „public“ je pseudonim za „export“.

- Deveti i deseti oblik su ekvivalentni: oni određuju asemblersko ime promenljive.

- Jedanaesti oblik deklariše promenljivu ( curterm11 ) i inicijalizuje je vrednošću (1 u gornjem
  slučaju).

Imajte na umu da asemblerska imena moraju biti jedinstvena. Nije moguće deklarisati ili eksportovati dve promenljive sa istim asemblerskim imenom. Posebno, ne pokušavajte da eksportujete promenljive sa javnim imenom koje počinje sa FPC_ ; kompajler koristi neke interne sistemske rutine sa ovim imenom.

## 4.3 Opseg

Promenljive, baš kao i svaki identifikator, poštuju opšta pravila opsega važenja. Pored toga, inicijalizovane promenljive se inicijalizuju kada uđu u opseg važenja:

- **Globalne inicijalizovane promenljive** se inicijalizuju jednom, pri pokretanju programa.
- **Lokalne inicijalizovane promenljive** se inicijalizuju svaki put kada se uđe u proceduru.

Imajte na umu da se ponašanje lokalno inicijalizovanih promenljivih razlikuje od ponašanja lokalno tipizirane konstante. Lokalna tipizirana konstanta se ponaša kao globalno inicijalizovana promenljiva.

## 4.4 Inicijalizovane promenljive

Podrazumevano, jednostavne promenljive u Paskalu se ne inicijalizuju nakon njihove deklaracije. Bilo kakva pretpostavka da sadrže 0 ili bilo koju drugu podrazumevanu vrednost je pogrešna: One mogu da sadrže gluposti. Da bi se ovo ispravilo, postoji koncept inicijalizovanih promenljivih. Razlika u odnosu na normalne promenljive je u tome što njihova deklaracija uključuje početnu vrednost.

**Napomena**  
Postoje 2 izuzetka od ovog pravila:

- Upravljani tipovi su izuzetak od ovog pravila: Upravljani tipovi se uvek inicijalizuju
  podrazumevanom vrednošću: generalno, to znači postavljanje brojača referenci na nulu ili postavljanje vrednosti pokazivača tipa na Nil.

- Globalne promenljive se inicijalizuju ekvivalentom nule.

Imajte na umu da ponašanje nuliranja određenih promenljivih može dovesti do nevažećeg sadržaja za promenljive:

```pascal
Type  
  TWeekDays =  
    (monday,tuesday,wednesday,thursday,friday,saturday,sunday);  
  TWeekend = saturday..sunday;  
 
var  
  W : TWeekend;  
 
begin  
  Writeln(W);  
end.
```

The above will result, when run, in an error:

```sh
Runtime error 107 at $000000000040024A  
$000000000040024A  
$000000000042BF70  
$00000000004001D2
```

Zbog toga se toplo preporučuje da uvek inicijalizujete promenljive pre nego što ih upotrebite.

Ovo se lako može uraditi u deklaraciji promenljivih. S obzirom na deklaraciju:

```pascal
Var  
  S : String = 'This is an initialized string';
```

Vrednost sledeće promenljive biće inicijalizovana datom vrednošću. Sledeći način je još bolji da se to uradi:

```pascal
Const  
  SDefault = 'This is an initialized string';  
 
Var  
  S : String = SDefault;
```

Inicijalizacija se često koristi za inicijalizaciju nizova i zapisa. Za nizove, inicijalizovani elementi moraju biti navedeni, okruženi okruglim zagradama i odvojeni zarezima. Broj inicijalizovanih elemenata mora biti potpuno isti kao broj elemenata u deklaraciji tipa. Na primer:

```pascal
Var  
  tt : array [1..3] of string[20] = ('ikke', 'gij', 'hij');  
  ti : array [1..3] of Longint = (1,2,3);
```

Za konstantne zapise, svaki element zapisa koji želite da inicijalizujete mora biti naveden u obliku Polje: Vrednost, odvojen tačka-zarezom i okružen okruglim zagradama.Možete izostaviti polja koja ne želite da inicijalizujete, zapravo možete preskočiti sva polja. Ako preskočite polja, kompajler će izdati upozorenje.

Kao primer:

```pascal
Type  
  Point = record  
    X,Y : Real  
    end;  
Var  
  Origin : Point = (X:0.0; Y:0.0);  
  Partial : Point = (X:0.0);  
  Empty : Point = ();
```

Gore navedene deklaracije će rezultirati sledećim upozorenjima:

```sh
iv.pp(7,27) Warning: Some fields coming after "X" were not initialized  
iv.pp(8,20) Warning: Some fields coming after "" were not initialized
```

Redosled polja u konstantnom zapisu mora biti isti kao u deklaraciji tipa, u suprotnom će doći do greške prilikom kompajliranja.

**Napomena**  
Treba naglasiti da se inicijalizovane promenljive inicijalizuju kada dođu u opseg važenja, za razliku od tipiziranih konstanti, koje se inicijalizuju pri pokretanju programa. Ovo važi i za lokalno inicijalizovane promenljive. Lokalno inicijalizovane promenljive se inicijalizuju kad god se rutina pozove. Sve promene koje su se dogodile u prethodnom pozivanju rutine biće poništene, jer se ponovo inicijalizuju.

**Napomena**  
Treba biti oprezan pri korišćenju inicijalizovanih tipova pokazivača kao što je `PChars`. U sledećim primerima, S je pokazivač, koji pokazuje na blok konstantnih (samo za čitanje) programskih podataka. Dodeljivanje karaktera u stringu stoga neće funkcionisati. Dodeljivanje samog S će naravno funkcionisati. Prva rutina će dati grešku, druga ne:

```pascal
procedure foo1;  
var  
  s: PChar = 'PChar';  
begin  
  s[0] := 'a'; // Pokušaj promene konstantnog stringa ne uspeva. Greška!!!  
end;  
 
procedure foo2;  
var  
  s: PChar;  
begin  
  s := 'PChar';  
  s[0] := 'a';  // Sada uspeva jer je s dodeljena string vrednost
end;
```

## 4.5 Inicijalizacija promenljivih korišćenjem podrazumevanih vrednosti

Neke promenljive moraju biti inicijalizovane jer sadrže upravljane tipove. Za promenljive koje su deklarisane u odeljku var funkcije ili u glavnom programu, ovo se dešava automatski. Za promenljive koje su alocirane na hipu, ovo nije nužno slučaj.

Za ovo, kompajler sadrži `Default` intrinzičnu funkciju. Ova funkcija prihvata identifikator tipa kao argument i vratiće ispravno inicijalizovanu promenljivu tog tipa. U suštini, ona će poništiti celu promenljivu.

Sledeći primer daje njegovu upotrebu:

```pascal
type  
  TRecord = record  
    i: LongInt;  
    s: AnsiString;  
  end;  
 
var  
  i: LongInt;  
  o: TObject;  
  r: TRecord;  

begin  
  i := Default(LongInt); // 0  
  o := Default(TObject); // Nil  
  r := Default(TRecord); // ( i: 0; s: '')  
end.

Interesantniji je slučaj kada je promenljiva dodeljena na hipu:

type  
  TRecord = record  
    i: LongInt;  
    s: AnsiString;  
  end;  
 
var  
  i: ^LongInt;  
  o: ^TObject;  
  r: ^TRecord;  

begin  
  i:=GetMem(SizeOf(Longint));  
  i^ := Default(LongInt); // 0  
  o:=GetMem(SizeOf(TObject));  
  o^ := Default(TObject); // Nil  
  r:=GetMem(SizeOf(TRecord));  
  r^ := Default(TRecord); // ( i: 0; s: '')  
end.
```

Radi za sve tipove, osim za različite tipove datoteka (ili složene tipove koji sadrže tip datoteke).

**Napomena**:

- Za generike, upotreba `Default` je posebno korisna, jer tip promenljive možda nije poznat tokom
  deklaracije generika.
- Rezultati funkcija su dostupni kao identifikator `Result` i kao takvi podsećaju na promenljive.
  One nisu promenljive, već se tretiraju kao parametri prosleđeni referencom. Stoga nisu inicijalizovani.

## 4.6 Thread promenljive

Za program koji koristi niti, promenljive mogu biti zaista globalne, tj. iste za sve niti, ili lokalne za nit: to znači da svaka nit dobija kopiju promenljive. Lokalne promenljive (definisane unutar procedure) su uvek lokalne za nit. Globalne promenljive su obično iste za sve niti. Globalna promenljiva može biti deklarisana kao lokalna za nit zamenom ključne reči var na početku bloka deklaracije promenljive sa `ThreadVar` :

```pascal
ThreadVar  
  IOResult : Integer;
```

Ako se ne koriste niti, promenljiva se ponaša kao obična promenljiva. Ako se koriste niti, onda se kopija pravi za svaku nit (uključujući i glavnu nit). Imajte na umu da se kopija pravi sa originalnom vrednošću promenljive, a ne sa vrednošću promenljive u trenutku pokretanja niti.

Promenljive tipa niti treba koristiti štedljivo: Postoji dodatni trošak za preuzimanje ili podešavanje vrednosti promenljive. Ako je ikako moguće, razmislite o korišćenju lokalnih promenljivih; one su uvek brže od promenljivih tipa niti.

Niti nisu podrazumevano omogućene. Za više informacija o programiranju niti, pogledajte poglavlje o nitima u Vodiču za programere .

## 4.7 Svojstva

Globalni blok može deklarisati svojstva, baš kao što bi se ona mogla definisati u klasi. Razlika je u tome što globalnom svojstvu nije potrebna instanca klase: postoji samo jedna instanca ovog svojstva. Osim toga, globalno svojstvo se ponaša kao svojstvo klase. Specifikatori čitanja/pisanja za globalno svojstvo takođe moraju biti regularne procedure, a ne metode.

Koncept globalnog svojstva je specifičan za Free Pascal i ne postoji u Delphi-ju. Za rad sa svojstvima je potreban ObjFPC režim.

Koncept globalnog svojstva može se koristiti za "sakrivanje" lokacije vrednosti, ili za izračunavanje vrednosti u hodu, ili za proveru vrednosti koje su zapisane u svojstvo.

Deklaracija je sledeća:

```pascal
{$mode objfpc}  
unit testprop;  

Interface  

Function GetMyInt : Integer;  
Procedure SetMyInt(Value : Integer);  
 
Property  
  MyProp : Integer Read GetMyInt Write SetMyInt;  
 
Implementation  
 
Uses sysutils;  
 
Var  
  FMyInt : Integer;  
 
Function GetMyInt : Integer;  
 
begin  
  Result:=FMyInt;  
end;  
 
Procedure SetMyInt(Value : Integer);  
 
begin  
  If ((Value mod 2)=1) then  
    Raise Exception.Create('MyProp can only contain even value');  
  FMyInt:=Value;  
end;  
 
end.
```

Specifikatori za čitanje/pisanje mogu se sakriti deklarisanjem u drugoj jedinici koja mora biti u klauzuli uses jedinice. Ovo se može koristiti za skrivanje specifikatora pristupa za čitanje/pisanje za programere, baš kao da su u privatnom odeljku klase (o čemu će biti reči u nastavku). Za prethodni primer, ovo bi moglo izgledati ovako:

```pascal
{$mode objfpc}  

unit testrw;  
 
Interface  
 
Function GetMyInt : Integer;  
Procedure SetMyInt(Value : Integer);  
 
Implementation  
 
Uses sysutils;  
 
Var  
  FMyInt : Integer;  
 
Function GetMyInt : Integer;  
 
begin  
  Result:=FMyInt;  
end;  
 
Procedure SetMyInt(Value : Integer);  
 
begin  
  If ((Value mod 2)=1) then  
    Raise Exception.Create('Only even values are allowed');  
  FMyInt:=Value;  
end;  
 
end.
```

Jedinični testprop bi tada izgledao ovako:

```pascal
{$mode objfpc}  
unit testprop;  
 
Interface  
 
uses testrw;  
 
Property  
  MyProp : Integer Read GetMyInt Write SetMyInt;  
 
Implementation  
 
end.
```

[prev][f3] [content][f0] [next][f5]

[f0]: 00_sadrzaj.md
[f3]: 03_tipovi.md
[f5]: 05_izrazi.md
