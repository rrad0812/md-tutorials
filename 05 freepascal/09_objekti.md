# 9 Objekti

[prev][f8] [content][f0] [next][f10]

## 9.1 Deklaracija

FreePaskal podržava objektno orijentisano programiranje. U stvari, veći deo kompajlera je napisan pomoću objekata. Ovde predstavljamo neka tehnička pitanja u vezi sa objektno orijentisanim programiranjem u FreePaskalu.

Objekte treba tretirati kao posebnu vrstu zapisa. Zapis sadrži sva polja koja su deklarisana u definiciji objekta i pokazivače na metode koje su povezane sa tipom objekta.

Objekat se deklariše baš kao što bi se deklarisao zapis; osim što se sada procedure i funkcije mogu deklarisati kao da su deo zapisa. Objekti mogu "nasleđivati" polja i metode od "roditeljskih" objekata. To znači da se ova polja i metode mogu koristiti kao da su uključeni u objekte deklarisane kao "podređeni" objekat.

Štaviše, koncept vidljivostije je uveden: polja, procedure i funkcije mogu biti deklarisane kao javne, zaštićene ili privatne. Podrazumevano, polja i metode su javne i izvoze se van trenutnog unita.

Polja ili metode koje su deklarisane kao privatne dostupne su samo u trenutnom unitu: njihov opseg je ograničen na implementaciju trenutnog unita.

Sledeća je validna definicija objekta:

```pascal
Type  
  TObj = object  
  
  Private  
    Caption : ShortString;  
  
  Public  
    Constructor init;  
    Destructor done;  
    Procedure SetCaption (AValue : String);  
    Function GetCaption : String;  
  end;
```

Sadrži par konstruktor/destruktor i metodu za dobijanje i postavljanje natpisa. Polje Natpis je privatno za objekat: ne može mu se pristupiti van unita u kojoj je deklarisan TObj.

**Napomena**  
U MacPas režimu, ključna reč `Object` je zamenjena ključnom reči `class` radi kompatibilnosti sa drugim Pascal kompajlerima dostupnim na Mac-u. To znači da se objekti ne mogu koristiti u MacPas režimu.

**Primedba**  
FreePaskal takođe podržava pakovani objekat. Ovo je isto što i objekat, samo što su elementi (polja) objekta poravnati po bajtovima, baš kao kod pakovanog zapisa. Deklaracija pakovanog objekta je slična deklaraciji pakovanog zapisa:

```pascal
Type  
  TObj = packed object  
   Constructor init;  
   ...  
   end;  
  Pobj = ^TObj;  
Var PP : Pobj;
```

Slično, direktiva `{$PackRecords}` deluje i na objekte.

## 9.2 Apstraktni i zapečaćeni objekti

Objekat se može proglasiti zapečaćenim. U tom slučaju nije moguće deklarisati descendentni objekat. Prevodilac će vratiti grešku ako naiđe na deklaraciju potomka:

```pascal
Type  
  TMyClass = object Sealed  
    x : integer;  
  end;  

  TMyClass2 = object(TMyClass)  
    Y : Integer;  
  end;  

begin  
end.
```

Ovo će dovesti do sledeće greške:

```sh
Error: Cannot create a descendant of the sealed class "TMyClass"
```

Apstraktna klasa je klasa koja se ne može direktno koristiti. Umesto toga, uvek se mora koristiti klasa potomaka. Međutim, radi Delphi kompatibilnosti, kompajler ignoriše ovu direktivu.

## 9.3 Polja

Polja objekata su kao polja zapisa. Njima se pristupa na isti način kao što bi se pristupilo polju zapisa: korišćenjem kvalifikovanog identifikatora. S obzirom na sledeću izjavu:

```pascal
Type TAnObject = Object  
  AField : Longint;  

  Procedure AMethod;  
  end;  

Var AnObject : TAnObject;
```

onda bi sledeći zadatak bio validan:

```pascal
  AnObject.AField := 0;
```

Unutar metoda, poljima se može pristupiti pomoću kratkog identifikatora:

```pascal
Procedure TAnObject.AMethod;  
begin  
  ...  
  AField := 0;  
  ...  
end;
```

Ili se može koristiti self identifikator. Self identifikator se odnosi na trenutnu instancu objekta:

```pascal
Procedure TAnObject.AMethod;  
begin  
  ...  
  Self.AField := 0;  
  ...  
end;
```

Nije moguće pristupiti poljima koja se nalaze u privatnim ili zaštićenim odeljcima objekta izvan metoda objekta. Ako se ovo ipak pokuša, kompajler će se žaliti na nepoznati identifikator.

Takođe je moguće koristiti naredbu `with` sa instancom objekta, baš kao i sa zapisom:

```pascal
With AnObject do  
  begin  
    Afield := 12;  
    AMethod;  
  end;
```

U ovom primeru, između `begin` i `end`, to je kao da je `AnObject` dodat pred identifikatorima `AField` i `AMethod`.

## 9.4 Class ili static polja

Objekat može da sadrži polja `class` ili `static`: ova polja su globalna za tip objekta i ponašaju se kao globalne promenljive, ali su poznata samo u opsegu važenja objekta. Razlika između `static` i `class` promenljivih je isključivo u načinu na koji rade: ključna reč `static` će uvek raditi, ključna reč `class` će zahtevati `{ $ MODE DELPHI }` ili `{ $ MODE OBJFPC }`.

Može se pozivati na njih unutar metoda objekta, ali se mogu pozivati i izvan objekta navođenjem potpuno kvalifikovanog imena.

Na primer, izlaz sledećeg programa:

```pascal
{$mode objfpc}  
type  
  cl=object  
    l : longint; static;  
    class var v : integer;  
  end;  

var  
  cl1,cl2 : cl;  

begin  
  Writeln('Static');  
  cl1.l:=2;  
  writeln(cl2.l);  
  cl2.l:=3;  
  writeln(cl1.l);  
  Writeln(cl.l);  
  Writeln('Class');  
  cl1.v:=4;  
  writeln(cl2.v);  
  cl2.v:=5;  
  writeln(cl1.v);  
  Writeln(cl.v);  
end.
```

biće sledeći:

```pascal
Static  
2  
3  
3  
Class  
4  
5  
5
```

Imajte na umu da poslednji red koda upućuje na sam tip objekta (cl), a ne na instancu objekta (cl1 ili cl2).

## 9.5 Konstruktori i destruktori

Free Pascal podržava konstruktore i destruktore. Programer je odgovoran za eksplicitno pozivanje konstruktora i destruktora kada se koriste objekti.

Deklaracija konstruktora ili destruktora je sledeća:

```pascal
Type  
  TObj = object;  
    Constructor init;  
    ...  
    end;  
  Pobj = ^TObj;  

Var PP : Pobj;
```

Tada su sledeća tri poziva ekvivalentna:

```pascal
 pp := new (Pobj,Init);
```

i

```pascal
  new(pp,init);
```

i takodje

```pascal
  new (pp);  
  pp^.init;
```

U poslednjem slučaju, kompajler će izdati upozorenje da se proširena sintaksa `new` i `dispose` mora koristiti za generisanje instanci objekta. Moguće je zanemariti ovo upozorenje, ali je bolja praksa programiranja koristiti proširenu sintaksu za kreiranje instanci objekta. Slično, `Dispose` procedura prihvata ime destruktora. Zatim će biti pozvan destruktor pre uklanjanja objekta iz gomile.

S obzirom na upozorenje kompajlera, sledeće poglavlje predstavlja Delphi pristup objektno orijentisanom programiranju i može se smatrati prirodnijim načinom objektno orijentisanog programiranja.

## 9.6 Metode

Objektne metode su kao obične procedure ili funkcije, samo što imaju implicitni dodatni parametar: `self`. `Self` ukazuje na objekat sa kojim je metoda pozvana. Prilikom implementacije metoda, potpuno kvalifikovani identifikator mora biti dat u zaglavlju funkcije. Kada se deklarišu metode, mora se dati normalan identifikator.

### 9.6.1 Deklaracija

Deklaracija metode je kao normalna deklaracija funkcije ili procedure, sa nekim dodatnim specifikacijama, koji su deo deklaracije objekta: sa stanovišta deklaracija, definicije metoda su deklaracije normalne funkcije ili procedure. Za razliku od TP i Delphija, polja se mogu deklarisati nakon metoda u istom bloku, tj. sledeće će generisati grešku prilikom kompajliranja sa Delphi ili Turbo Pascal, ali ne i sa FPC:

```pascal
Type  
  MyObj = Object  
    Field : Longint;  
    Procedure Doit;  
  end;
```

### 9.6.2 Ppozivanje metoda

Metode se pozivaju isto kao što se pozivaju i normalne procedure, samo što se ispred njih dodaje identifikator instance objekta. Da bi se utvrdilo koja se metoda poziva, potrebno je znati tip metode. U nastavku ćemo obraditi različite tipove.

#### 9.6.2.1 Obične ili statičke metode

Normalne (statičke - u smislu da se poziv odredjuje za vreme compile-time) metode su metode koje nisu deklarisane bez `apstract` ili `virtual` ključne reči. Prilikom pozivanja obične metode, koristi se deklarisana (tj. metoda vremena kompajliranja) objekta. Na primer, razmotrite sledeće deklaracije:

```pascal
Type  
  TParent = Object  
    ...  
    procedure Doit;  
    ...  
    end;  
  PParent = ^TParent;  
  
  TChild = Object(TParent)  
    ...  
    procedure Doit;  
    ...  
    end;  
  PChild = ^TChild;
```

Kao što je vidljivo, i roditeljski i podređeni objekti imaju metod pod nazivom `Doit`. Razmotrite sada sledeće deklaracije i pozive:

```pascal
Var  
  ParentA,ParentB : PParent;  
  Child           : PChild;  
 
begin  
  ParentA := New(PParent,Init);  
  ParentB := New(PChild,Init);  
  Child := New(PChild,Init);
  ParentA^.Doit;  
  ParentB^.Doit;  
  Child^.Doit;
```

Od tri poziva metode Doit, samo poslednji će pozvati TChild.Doit, dok će druga dva poziva pozvati TParent.Doit. To je zato što kod običnih metoda, kompajler u vreme kompajliranja određuje koja metoda treba da se pozove. Pošto je ParentB tipa TParent, kompajler odlučuje da se mora pozvati sa TParent.Doit, iako će biti kreiran kao TChild. Može postojati situacija kada metoda koja se zapravo poziva treba da zavisi od stvarnog tipa objekta u vreme izvršavanja. Ako je tako, metoda ne može biti obična metoda, već mora biti virtuelna metoda.

#### 9.6.2.2 Virtuelne metode

Da bi se rešila situacija iz prethodnog odeljka, kreiraju se virtuelne metode. To se jednostavno radi dodavanjem modifikatora `virtual` deklaraciji metode. Nasledni objekat zatim može da prepiše metodu novom implementacijom ponovnim deklarisanjem metode (sa istom listom parametara) koristeći ključnu reč `virtual`.

Vraćajući se na prethodni primer, razmotrite sledeću alternativnu deklaraciju:

```pascal
Type  
  TParent = Object  
    ...  
    procedure Doit;virtual;  
    ...  
    end;  
  PParent = ^TParent;  
  
  TChild = Object(TParent)  
    ...  
    procedure Doit;virtual;  
    ...  
    end;  
  PChild = ^TChild;
```

Kao što je vidljivo, i roditeljski i podređeni objekti imaju metod koji se zove Doit. Razmotrite sada sledeće deklaracije i pozive:

```pascal
Var  
  ParentA,ParentB : PParent;  
  Child           : PChild;  
 
begin  
  ParentA := New(PParent,Init);  
  ParentB := New(PChild,Init);  
  Child := New(PChild,Init);  

  ParentA^.Doit;  
  ParentB^.Doit;  
  Child^.Doit;
```

Sada će se pozivati različite metode, u zavisnosti od stvarnog tipa objekta tokom izvršavanja. Za ParentA se ništa ne menja, pošto je kreiran kao instanca TParent. Za Child se situacija takođe ne menja: ponovo se kreira kao instanca TChild.

Međutim, za ParentB se situacija menja: Iako je deklarisan kao TParent, kreira se kao instanca TChild. Sada, kada se program pokrene, pre pozivanja Doit, program proverava koji je stvarni tip ParentB, i tek tada odlučuje koja metoda mora biti pozvana. Pošto je ParentB tipa TChild, biće pozvana TChild.Doit. Kod za ovu proveru stvarnog tipa objekta tokom izvršavanja ubacuje kompajler tokom kompajliranja.

Za TChild.Doit se kaže da "preklapa" TParent.Doit. Moguće je pristupiti TParent.Doit iz TChild.Doit, pomoću `inherited` ključne reči:

```pascal
Procedure TChild.Doit;  
begin  
  inherited Doit;  
  ...  
end;
```

U gornjem primeru, kada se pozove TChild.Doit, prvo što se radi jeste poziv TParent.Doit. `Inherited` ključna reč ne može se koristiti u običnim metodama, već samo u virtuelnim metodama.

Da bi ovo mogao da uradi, kompajler čuva – za svaki tip objekta – tabelu sa virtuelnim metodama: VMT (Virtuelna tabela metoda). Ovo je jednostavno tabela sa pokazivačima na svaku od virtuelnih metoda: svaka virtuelna metoda ima svoju fiksnu lokaciju u ovoj tabeli (indeks). Kompajler koristi ovu tabelu da bi pronašao stvarnu metodu koja se mora koristiti. Kada potomački objekat prepiše metodu, unos roditeljske metode se prepisuje u VMT-u. Više informacija o VMT-u možete pronaći u Vodiču za programere.

Kao što je ranije napomenuto, objekti koji imaju VMT moraju biti inicijalizovani konstruktorom: objektna promenljiva mora biti inicijalizovana pokazivačem na VMT stvarnog tipa sa kojim je kreirana.

#### 9.6.2.3 Apstraktne metode

Apstraktna metoda je posebna vrsta virtuelne metode. Metoda koja je deklarisana kao apstraktna nema implementaciju za ovu metodu. Na nasleđenim objektima je da je prepišu i implementiraju.

Iz ovoga sledi da metoda ne može biti apstraktna ako nije virtuelna. Druga posledica je da se instanca objekta koji ima apstraktnu metodu ne može direktno kreirati.

Razlog je očigledan: ne postoji metoda na koju bi kompajler mogao da pređe! Metoda koja je deklarisana kao apstraktna nema implementaciju za ovu metodu. Na nasleđenim objektima je da je prepišu i implementiraju. Nastavljajući naš primer, pogledajte ovo:

```pascal
Type  
  TParent = Object  
    ...  
    procedure Doit; virtual; abstract;  
    ...  
    end;  
  PParent=^TParent;

  TChild = Object(TParent)  
    ...  
    procedure Doit;virtual;  
    ...  
    end;  
  PChild = ^TChild;
```

Kao što je vidljivo, i roditeljski i podređeni objekti imaju metod koji se zove Doit. Razmotrite sada sledeće deklaracije i pozive:

```pascal
Var  
  ParentA,ParentB : PParent;  
  Child           : PChild;  

begin  
   ParentA := New(PParent,Init);  
   ParentB := New(PChild,Init);  
   Child := New(PChild,Init);  
   
   ParentA^.Doit;  // Greška. Ne može se izvesti instanca klase koja ima apstraktnu metodu
   ParentB^.Doit;  
   Child^.Doit;
```

Pre svega, red 3 će generisati grešku kompannjlera, navodeći da se ne mogu generisati instance objekata sa apstraktnim metodama: Kompajler je otkrio da PParent ukazuje na objekat koji ima apstraktnu metodu. Komentarisaje reda 3 bi omogućilo kompajliraje programa.

Nasleđena primedba Ako je apstraktna metoda prepisana, roditeljska metoda ne može biti pozvana sa , jer ne postoji roditeljska metoda; kompajler će ovo detektovati i žaliti se na to, ovako:
testo.pp(32,3) Greška: Apstraktne metode se ne mogu direktno pozivati

Ako se, putem nekog mehanizma, apstraktna metoda pozove tokom izvršavanja, doći će do greške tokom izvršavanja. (preciznije, greška tokom izvršavanja 211)

#### 9.6.2.4 Class ili static metode

`Class` metode ili `static` metode su metode koje su globalne za tip objekta. Kada se pozovu, implicitni pokazivač „self“ nije dostupan. To znači da se normalne metode ne mogu pozvati iz njih i da se ne može pristupiti nijednom polju objekta. Međutim, mogu se koristiti promenljive klase.

Class ili static metode su regularne metode, mogu se dodeliti proceduralnoj promenljivoj.

Sledeći program demonstrira sve ovo. Komentarisane izjave se neće kompajlirati.

```pascal
{$APPTYPE CONSOLE}  
{$IFDEF FPC}{$MODE DELPHI}{$H+}{$ENDIF}  

type  
  TTest = object  
    const Epsylon = 100;  
    var f : integer;  
    class var cv1,cv2:integer;  
    procedure myproc;  
    class procedure testproc;  
    class procedure testproc2; static;  
    procedure testproc3; static;  
  end;  

  procedure TTest.myproc;  
  begin  
    cv1:=0;  
    f:=1;  
  end;  

  class procedure TTest.Testproc;  
  begin  
    cv1:=1;  
    // f:=1;  
  end;  

  class procedure TTest.Testproc2;  
  begin  `
    cv1:=2;  
    // f:=1;  
  end;  

  procedure TTest.Testproc3;  
  begin  
    cv1:=3;  
    // f:=1;  
  end;  

Var  
  P : Procedure;  

begin  
  P:=@TTest.Myproc;  
  P:=@TTest.Testproc;  
  P:=@TTest.Testproc2;  
  P:=@TTest.Testproc3;  
end.
```

Dekomentarisanje jedne od komentarisanih izjava i pokušaj kompajliranja rezultujućeg koda će dovesti do greške kompajlera:

```sh
ocv.pp(32,6) Error: Only class methods, class properties and  
  class variables can be accessed in class methods
```

## 9.7 Pregled vrsta metoda

Tabela ispod sumira sve vrste metoda koje su dostupne u **Object** tipu podataka:

 Tip metode | Deklaracija | Karakteristike | Kada koristiti |
 --------- | ----------- | -------------- | -------------- |
 **Obične (statičke)** | `procedure Doit;` | Poziv se određuje u vreme kompajliranja. Tip promenljive određuje koja metoda se poziva. Najbrža metoda poziva. Ne može se override-ovati. | Standardni metodi bez potrebe za polimorfizmom. |
 **Virtual** | `procedure Doit; virtual;` | Poziv se određuje u vreme izvršavanja. Koristi VMT (Virtual Method Table). Može se nadjačati u potomcima. U potomku takođe koristite `virtual` (ne `override`!) | Kada je potreban polimorfizam |
 **Abstract** | `procedure Doit; virtual; abstract;` | Mora biti `virtual`. Nema implementaciju.  Mora se implementirati u potomku. Ne može se kreirati instanca objekta sa abstract metodama | Interfejsi, šablonski obrazac |
 **Class/Static** | `class procedure Doit`; ili `procedure Doit; static;` | Nema pristup `Self`. Nema pristup običnim poljima. Ima pristup class poljima. Može se dodeli proceduralnoj promenljivoj | Pomoćne funkcije bez potrebe za instancom |

### 9.7.1 Važne razlike u odnosu na C++ i Classes

#### Virtuelne metode u Objects vs Classes

U **Objects** (trenutni tip):

```pascal
Type
  TParent = Object
    procedure Doit; virtual;  // U roditeljskoj klasi
  end;
  
  TChild = Object(TParent)
    procedure Doit; virtual;  // U potomku TAKOĐE virtual!
  end;
```

U **Classes** (pogledaj [Classes](08_classes.md):

```pascal
Type
  TParent = Class
    procedure Doit; virtual;   // U roditeljskoj klasi
  end;
  
  TChild = Class(TParent)
    procedure Doit; override;  // U potomku override!
  end;
```

#### Ključne razlike

  Aspekt | Objects | Classes |
 ------- | ------- | ------- |
 **Alokacija** | Može se na steku ili heap-u | Uvek na heap-u (sa `Create`) |
 **Inicijalizacija** | `New(P, Constructor)` | `P := TClass.Create` |
 **Destrukcija** | `Dispose(P, Destructor)` | `P.Free` ili `P.Destroy` |
 **Override sintaksa** | `virtual` u oba | `virtual` → `override` |
 **Nasleđivanje** | Prototipsko | Klasično OOP |
 **Automatsko čišćenje** | Ne | Da (sa `try-finally`) |
 **RTTI** | Ograničeno | Potpuno |
 **Interfejsi** | Ne | Da |
 **Property** | Ograničeno | Potpuno |

### 9.7.2 Pojašnjenje: Obične (Statičke) vs Class/Static

**VAŽNO: Terminološka zabuna!**

**"Obični (Statičke)"** metod i **"Class/Static"** metod su **potpuno različite stvari**:

  Karakteristika | Obični (Statički) metod | Class/Static metod |
  -------------- | --------------------- | ------------------ |
  **Druga imena** | Obični metod, Regularni metod | Metod klase |
  **Self pokazivač** | ✅ DA (pokazuje na **instancu**) | ❌ NE (ili pokazuje na **VMT**) |
  **Pristup poljima instance** | ✅ DA | ❌ NE |
  **Pristup class poljima** | ✅ DA | ✅ DA |
  **"Static" znači** | Poziv određen u vreme **kompajliranja** | Pripada **klasi**, ne instanci |
  **Deklaracija** | `procedure Doit;` | `class procedure Doit;` ili `procedure Doit; static;` |

**Primer razlike:**

```pascal
type
  TExample = object
    FValue: Integer;                 // Obično polje (instance)
    class var FClassValue: Integer;  // Class polje
    
    procedure NormalMethod;          // Static(Normal) - ima Self
    class procedure ClassMethod; static;  // Class/Static - nema Self
  end;

procedure TExample.NormalMethod;
begin
  FValue := 10;          // ✅ OK - pristupa polju instance
  FClassValue := 20;     // ✅ OK - pristupa class polju
  WriteLn(Self.FValue);  // ✅ OK - Self postoji
end;

class procedure TExample.ClassMethod;
begin
  // FValue := 10;       // ❌ GREŠKA - nema pristup instance poljima!
  FClassValue := 30;     // ✅ OK - pristup class polju
  // WriteLn(Self.FValue); // ❌ GREŠKA - Self ne postoji!
end;
```

### 9.7.3 Kako rade obične (statičke) metode u nasleđivanju

- **Ključno razumevanje**: TIP PROMENLJIVE određuje koja se metoda poziva, NE stvarni tip objekta!  
- **Radi kao u Pythonu** - poziv se određuje u vreme kompajliranja:

```pascal
program StaticInheritanceDemo;

type
  TAnimal = object
    procedure Speak; virtual;        // Virtual - biće override-ovan
    procedure Identify;              // Static - NEĆE biti override-ovan
  end;
  AnimalPointer: ^TAnimal;

  TDog = object(TAnimal)
    procedure Speak; virtual;        // Override virtual metoda
    procedure Identify;              // Ovo je NOVA metoda, ne override!
  end;
  DogPointer: ^TDog;

procedure TAnimal.Speak;
begin
  WriteLn('Some animal sound');
end;

procedure TAnimal.Identify;
begin
  WriteLn('I am an animal');
end;

procedure TDog.Speak;
begin
  WriteLn('Woof!');
end;

procedure TDog.Identify;
begin
  WriteLn('I am a dog');
end;

begin
  // Kreiramo TDog instancu
  New(DogPointer, Create);
  
  // Dečiji pokazivač dodelimo roditeljskom pokazivaču
  AnimalPointer := DogPointer;  // ✅ Dozvoljeno - TDog je TAnimal
  
  WriteLn('=== Poziv preko AnimalPointer (tip ^TAnimal) ===');
  AnimalPointer^.Speak;      // Output: "Woof!"
                             // ✅ Virtual - gleda STVARNI tip (TDog)
  
  AnimalPointer^.Identify;   // Output: "I am an animal"
                             // ❌ Static - gleda TIP PROMENLJIVE (TAnimal)
  
  WriteLn('=== Poziv preko DogPointer (tip ^TDog) ===');
  DogPointer^.Speak;         // Output: "Woof!"
  DogPointer^.Identify;      // Output: "I am a dog"
  
  Dispose(DogPointer, Destroy);
end.
```

**Output:**

```sh
=== Poziv preko AnimalPointer (tip ^TAnimal) ===
Woof!
I am an animal
=== Poziv preko DogPointer (tip ^TDog) ===
Woof!
I am a dog
```

**Objašnjenje:**

- `AnimalPointer` je **tipa** `^TAnimal`, ali pokazuje na **instancu** `TDog`
- Za **virtual** metode: koristi VMT → poziva `TDog.Speak` ✅
- Za **static** metode: koristi tip promenljive → poziva `TAnimal.Identify` ❌

Ovo je **kao u Pythonu**, ali u Pascalu se to zove "static binding" vs "dynamic binding".

### 9.7.4 Virtual metode mogu (ali ne moraju) biti override-ovane

Virtual metod u roditelju **IMA implementaciju**. Potomak može:

1. **Override-ovati potpuno** (zameniti kod):

   ```pascal
   type
     TParent = object
       procedure DoWork; virtual;
     end;
     
     TChild = object(TParent)
       procedure DoWork; virtual;  // Potpuno nova implementacija
     end;
   
   procedure TParent.DoWork;
   begin
     WriteLn('Parent doing work');
   end;
   
   procedure TChild.DoWork;
   begin
     WriteLn('Child doing work differently');  // Bez inherited
   end;
   ```

2. **Override-ovati i ulančati** (dodati kod):

   ```pascal
   procedure TChild.DoWork;
   begin
     inherited DoWork;  // Pozovi roditeljski kod prvo
     WriteLn('Child adds extra work');
   end;
   ```

3. **Ne override-ovati** (nasleđuje roditeljski kod):

   ```pascal
   type
     TChild = object(TParent)
       // procedure DoWork;  // Ne deklarišemo - koristi roditeljski
     end;
   
   var
     C: ^TChild;
   begin
     New(C, Create);
     C^.DoWork;  // Poziva TParent.DoWork
   end;
   ```

### 9.7.5 RTTI = Run-Time Type Information

RTTI omogućava proveru i manipulaciju tipova u vreme izvršavanja:

```pascal
type
  TAnimal = object
    constructor Create;
    procedure Speak; virtual;
  end;
  
  TDog = object(TAnimal)
    procedure Speak; virtual;
  end;

var
  Animal: ^TAnimal;
begin
  Animal := New(PDog, Create);
  
  // RTTI provere:
  if Animal is TDog then           // Provera tipa
    WriteLn('This is a dog!');
    
  // TypeInfo,ClassName i slično...
end;
```

**RTTI u Objects vs Classes:**

- **Objects**: Ograničen RTTI (samo osnovne provere)
- **Classes**: Pun RTTI (streaming, reflection, property inspection)

### 9.7.6 Pregled: Kako se pozivi metoda određuju

 Tip metoda | Određivanje poziva | Vreme određivanja | Primer |
 ---------- | ------------------ | ----------------- | ------- |
 **Static (Normal)** | Po **tipu promenljive** | **Compile-time** | `AnimalPtr^.Method` → `TAnimal.Method` |
 **Virtual** | Po **stvarnom tipu objekta** | **Run-time** (VMT) | `AnimalPtr^.Method` → `TDog.Method` |
 **Class/Static** | Po **klasi** (ne instanci) | **Compile-time** | `TAnimal.ClassMethod` |

### 9.7.7 Praktični savet: Kada koristiti šta

  Scenario | Preporuka | Razlog |
  -------- | --------- | ------- |
 Metod se nikad ne menja | Obični (Statički) | Brži, jednostavniji |
 Metod se override-uje u potomcima | Virtual | Polimorfizam |
 Metod bez potrebe za instancom | Class/Static | Ne zahteva objekat |
 Treba pristup poljima | Static (Normal) | Ima Self |
 Interfejs koji potomci implementiraju | Abstract | Forsira implementaciju |

### 9.7.8 Kompletan primer: Static vs Virtual

```pascal
program CompleteDemo;

type
  TAnimal = object
    procedure Speak; virtual;        // Virtual - dynamic binding
    procedure Identify;              // Static - static binding
  end;

  TDog = object(TAnimal)
    procedure Speak; virtual;        
    procedure Identify;              
  end;

procedure TAnimal.Speak;
begin
  WriteLn('Some animal sound');
end;

procedure TAnimal.Identify;
begin
  WriteLn('I am an animal');
end;

procedure TDog.Speak;
begin
  WriteLn('Woof!');
end;

procedure TDog.Identify;
begin
  WriteLn('I am a dog');
end;

var
  AnimalPointer: ^TAnimal;
  DogPointer: ^TDog;
  
begin
  New(DogPointer, Create);
  AnimalPointer := DogPointer;  // Dečiji → roditeljski pokazivač
  
  WriteLn('Poziv preko AnimalPointer (tip ^TAnimal):');
  AnimalPointer^.Speak;      // "Woof!" - virtual gleda stvarni tip
  AnimalPointer^.Identify;   // "I am an animal" - static gleda tip promenljive
  
  WriteLn('Poziv preko DogPointer (tip ^TDog):');
  DogPointer^.Speak;         // "Woof!"
  DogPointer^.Identify;      // "I am a dog"
  
  Dispose(DogPointer, Destroy);
end.
```

### 9.7.9 Preporuka

Za **novi kod**, razmotri korišćenje **Classes** umesto **Objects**:

- Sintaksa bliža modernom OOP-u (Delphi, C++, Java)
- `override` direktiva je eksplicitnija i bezbednija
- Bolja podrška za RTTI i interfejse
- Kompatibilno sa Delphi kodom

Vidi detaljnije u [Classes](08_classes.md).

**Objects** se i dalje koriste kada je:

- Potrebna alociranje na steku (bolje performanse)
- Potrebna kompatibilnost sa starijim kodom
- Rad sa low-level sistemskim kodom

[prev][f8] [content][f0] [next][f10]

[f0]: 00_sadrzaj.md
[f8]: 08_propterećenje_operatora.md
[f10]: 10_klase.md
