
# 16 Izuzeci

[prev][f15] [content][f0] [next][f17]

Izuzeci pružaju zgodan način za programiranje mehanizama za greške i oporavak od grešaka i usko su povezani sa klasama. Podrška za izuzetke zasniva se na tri konstrukcije:

- **Raise** izjava, da bi se pokrenuo izuzetak. Ovo se obično radi da bi se signaliziralo stanje greške.Međutim, takođe je upotrebljivo za prekid izvršavanja i trenutni povratak na dobro poznatu tačku u izvršnoj datoteci.
- **Try...Except** blokovi. Ovi blokovi služe za hvatanje izuzetaka nastalih unutar opsega bloka i za obezbeđivanje koda za oporavak od izuzetaka.
- **Try...Finally** blokovi. Ovi blokovi služe da prisile izvršavanje koda bez obzira na pojavu izuzetka ili ne. Generalno služe za čišćenje memorije ili zatvaranje datoteka u slučaju da se dogodi izuzetak. Kompilator generiše mnogo implicitnih **Try ... Finally** blokova oko procedure, kako bi se nametnula konzistentnost memorije.

**Napomena**  
Pošto izuzeci koriste klase, potreban vam je jedan od objektnih paskal režima da biste mogli da ih koristite:

```pascal
{$MODE OBJFPC}
```

Ili

```pascal
{$MODE DELPHI} 
```

## 16.1 Izjava raise

Ova izjava će izazvati izuzetak. Ako je navedena, instanca izuzetka mora biti inicijalizovana instanca bilo koje klase, koja je tipa raise. Adresa izuzetka i okvir su opcioni. Ako nisu navedeni, kompajler će sam obezbediti adresu. Ako je instanca izuzetka izostavljena, onda se trenutni izuzetak ponovo podiže. Ova konstrukcija se može koristiti samo u bloku za obradu izuzetaka (videti dalje).

**Primedba**  
Kontrola se nikada ne vraća nakon bloka izuzetka. Kontrola se prenosi na prvu izjavu **try...finally** ili **try...except** koja se pojavi prilikom odmotavanja steka. Ako se takva izjava ne pronađe, biblioteka Free Pascal Run-Time Library će generisati grešku **217** tokom izvršavanja. Adresa izuzetka će biti ispisana podrazumevanim rutinama za obradu izuzetaka.

Na primer: Sledeće deljenje proverava da li je imenilac nula i ako jeste, izaziva izuzetak tipa
EDivException.

```pascal
Type
  EDivException = Class(Exception);  

Function DoDiv (X,Y : Longint) : Integer;  
begin  
  If Y=0 then  
    Raise EDivException.Create ('Division by Zero would occur');  
  Result := X Div Y;  
end;
```

Klasa `Exception` je definisana u `SysUtils` unitu RTL-a.

**Napomena**  
Iako se klasa `Exception` koristi kao osnovna klasa za izuzetke kroz ceo kod, ovo je samo nepisani dogovor: klasa može biti bilo kog tipa i ne mora biti potomak klase `Exception`.

Naravno, većina koda zavisi od nepisanog sporazuma da klasa izuzetaka potiče od klase `Exception`.

Sledeći kod pokazuje kako izostaviti rutinu za prijavljivanje grešaka iz steka prikazanog u obrađivaču izuzetaka:

```pascal
{$mode objfpc}  
uses sysutils;  
 
procedure error(Const msg : string);  
begin  
  raise exception.create(Msg) at  
    get_caller_addr(get_frame),  
    get_caller_frame(get_frame);  
end;  
 
procedure test2;  
begin  
  error('Error');  
end;  
 
begin  
  test2;  
end.
```

Program će, kada se pokrene, prikazati sledeći povratni trag:

```sh
Došlo je do neobrađenog izuzetka na adresi $00000000004002D3:  
Izuzetak: Greška  
  $00000000004002D3, red 15 datoteke testme.pp  
  $00000000004002E6, red 19 datoteke testme.pp
```

Red 15 je u proceduri Test2 , a ne u Error, koja je zapravo izazvala izuzetak.

## 16.2 Naredba try...except

Sintaksa:

```pascal
try:
  { list of statement: }
except
  exception On 'Name_Of_Class_Type_Identifier' Do
    statement;
  ...
else
  {list of statement}
end;
```

Ako se tokom izvršavanja liste izjava u `try` bloku ne generiše izuzetak, onda će se sve izjave u listi izvršavati sekvencijalno, a blok `except` će biti preskočen, prenoseći tok programa na naredbu nakon konačnog `end`-a.

Ako se izuzetak dogodi tokom izvršavanja liste `izjava` u `try` bloku, tok programa će biti prebačen na blok `except`. Izjave u listi izjava između mesta gde je izuzetak podignut i bloka izuzetka se ignorišu.

U bloku za obradu izuzetaka proverava se tip izuzetka i ako postoji obrada izuzetaka gde se tip klase podudara sa tipom objekta izuzetka ili je roditeljski tip tipa objekta izuzetka, onda će se izvršiti izjava koja sledi nakon odgovarajućeg `Do`. Koristi se prvi podudarni tip. Nakon što je blok `Do` izvršen, program nastavlja nakon `End` izjave.

Identifikator u izjavi za obradu izuzetaka je opcionalan i deklariše objekat izuzetka. Može se koristiti za manipulaciju objektom izuzetka u kodu za obradu izuzetaka. Opseg ove deklaracije je blok izjave koji sledi ključnu reč `Do`.

Ako se nijedan od `On` obrađivača ne podudara sa tipom objekta izuzetka, onda se izvršava lista izjava nakon `else`. Ako se takva lista ne pronađe, onda se izuzetak automatski ponovo podiže. Ovaj proces omogućava ugnježdavanje blokova `try...except`.

Ako je, s druge strane, izuzetak uhvaćen, onda se objekat izuzetka uništava na kraju bloka za obradu izuzetka, pre nego što se tok programa nastavi. Izuzetak se uništava pozivom destruktora objekta `Destroy` .

Na primer, s obzirom na prethodnu deklaraciju funkcije DoDiv, razmotrimo sledeće

```pascal
Try  
  Z := DoDiv (X,Y);  
Except  
  On EDivException do 
    Z := 0;  
end;
```

Ako se desi da je Y jednako nuli, onda će kod funkcije DoDiv podići izuzetak. Kada se to desi, tok programa se prenosi na izjavu `Except`, gde će obrađivač izuzetaka postaviti vrednost Z na nulu. Ako se ne podigne izuzetak, tok programa se nastavlja nakon poslednje naredbe `End`. Da bi se omogućio oporavak od greške, podržan je blok `Try ... Finally`. Blok `Try...Finally` osigurava da se izjave koje slede ključnu reč `Finally` izvršavaju, čak i ako se dogodi izuzetak.

## 16.3 Naredba try...finally

Naredba `Try..Finally` ima sledeći oblik:

```pascal
Try
  {List of statement}
Finally
  {Finally statements}
End;
```

Ako se ne dogodi izuzetak unutar bloka `try`, program se izvršava kao da ključne reči `Try`, `Finally` i `End` nisu prisutne, osim ako nije zadata komanda `exit`: komanda `exit` prvo izvršava sve izjave u bloku `finally` pre nego što zapravo izađe.

Međutim, ako se dogodi izuzetak, tok programa se odmah prenosi sa tačke gde je izuzetak podignut na prvu izjavu `Finally` liste izjava.

Sve izjave nakon ključne reči `finally` biće izvršene, a zatim će izuzetak biti automatski ponovo pokrenut. Sve izjave između mesta gde je izuzetak pokrenut i prve izjave `Finally` se preskaču.

Kao primer, razmotrite sledeću rutinu:

```pascal
Procedure Doit (Name : string);  

Var 
  F : Text;  

begin  
  Assign (F,Name);  
  Rewrite (name);  

  Try  
    ... File handling ...  
  Finally  
    Close(F);  
  end;  
end;
```

Ako se tokom izvršavanja obrade datoteke dogodi izuzetak, tok programa će se nastaviti od izjave `close(F)`, preskačući sve operacije sa datotekama koje mogu uslediti između mesta gde je izuzetak podignut i naredbe `Close`. Ako se nije dogodio izuzetak, sve operacije sa datotekama će biti izvršene i datoteka će biti zatvorena na kraju.

Imajte na umu da će izjava `Exit` okružena blokom `try .. finally` i dalje izvršiti blok `finally`. Ponovno korišćenje prethodnog primera:

```pascal
Procedure Doit (Name : string);  

Var  
  F : Text;  
  B : Boolean;  

begin  
  B:=False;  
  Assign (F,Name);  
  Rewrite (name);  

  Try  
    // ... File handling ...  
    if B then  
      exit; // Stop processing prematurely  
    // More file handling  
  Finally  
    Close(F);  
  end;  
end;
```

Datoteka će i dalje biti zatvorena, čak i ako se obrada prerano završi korišćenjem naredbe `Exit`.

## 16.4 Ugnežđavanje obrade izuzetaka

Moguće je ugnježvati blokove `Try...Except` sa blokovima `Try...Finally`. Tok programa će se odvijati po principu LIFO (poslednji ušao, prvi izašao): Kod poslednjeg pronađenog bloka `Try...Except` ili `Try...Finally` će se prvo izvršiti. Ako izuzetak nije uhvaćen ili je u pitanju bila `finally` izjava, tok programa će se preneti na pretposlednji blok, do beskonačnosti.

Ako se dogodi izuzetak, a ne postoji program za rukovanje izuzecima koji obrađuje ovaj izuzetak, generisaće se greška tokom izvršavanja 217. Pri korišćenju unita `SysUtils`, instalira se podrazumevani program za rukovanje koji će prikazati poruku objekta izuzetka i adresu na kojoj se izuzetak dogodio, nakon čega će se program završiti sa instrukcijom `Halt`.

## 16.5 Klase izuzetaka

Unit `sysutils` sadrži veliki deo obrade izuzetaka. Ona definiše osnovnu klasu izuzetaka, `Exception`.

```pascal
Exception = class(TObject)  
private  
  fmessage : string;  
  fhelpcontext : longint;  
public  
  constructor create(const msg : string);  
  constructor createres(indent : longint);  
  property helpcontext : longint read fhelpcontext write fhelpcontext;  
  property message : string read fmessage write fmessage;  
end;  

ExceptClass = Class of Exception;
```

I koristi ovu deklaraciju da definiše popriličan broj izuzetaka, na primer:

```pascal
{ mathematical exceptions }  
EIntError = class(Exception);  
EDivByZero = class(EIntError);  
ERangeError = class(EIntError);  
EIntOverflow = class(EIntError);  
EMathError = class(Exception);
```

Jedinica `SysUtils` takođe instalira program za obradu izuzetaka. Ako bilo koji blok za obradu izuzetaka ne obradi izuzetak, ovaj program poziva biblioteka Run-Time. U osnovi, on ispisuje adresu izuzetka i ispisuje poruku objekta `Exception`, a zatim se izlazi sa izlaznim kodom 217. Ako objekat izuzetka nije potomak objekta `Exception`, onda se umesto poruke o izuzetku ispisuje ime klase.

Preporučuje se korišćenje objekta `Exception` ili klase potomka za sve `raise` izjave, jer se tada može koristiti polje `message` objekta `exception`.

[prev][f15] [content][f0] [next][f17]

[f0]: 00_sadrzaj.md
[f15]: 15_programi_uniti_blokovi.md
[f17]: 17_asembler.md
