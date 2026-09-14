
# 15 Programi, uniti i blokovi

[prev][f14] [content][f0] [next][f16]

## 15.1 Programi

Paskal program se sastoji od zaglavlja `program`-a, nakon čega eventualno sledi klauzula `uses` i potom `Begin..End` blok.

Zaglavlje programa je dato radi kompatibilnosti sa prethodnim verzijama i kompajler ga ignoriše.

Klauzula `uses` služi za identifikaciju svih unita koje su potrebne programu. Svi identifikatori koji su deklarisani u odeljku `interface` unita u klauzuli `uses` dodaju se poznatim identifikatorima programa. `Sistem`-ski unit ne mora biti na ovoj listi, jer je uvek učitava kompajler.

Redosled kojim se uniti pojavljuju je značajan, on određuje kojim redosledom su inicijalizovane. Uniti se inicijalizuju istim redosledom kao što se pojavljuju u klauzuli `uses`. Identifikatori se pretražuju obrnutim redosledom, tj. kada kompajler traži identifikator, onda prvo traži poslednju jedinicu u klauzuli `uses`, zatim pretposlednju i tako dalje. Ovo je važno u slučaju da dva unita deklarišu različite tipove sa istim identifikatorom.

Kompilator će tražiti kompajlirane verzije ili verzije izvornog koda svih unita u klauzuli `uses` u putanji pretrage unita. Ako je naziv datoteke unita eksplicitno pomenut pomoću ključne reči `in`, izvor se preuzima iz navedenog imena datoteke:

```pascal
program programb;  
 
uses unita in '..\unita.pp';
```

Unit se pretražuje u roditeljskom direktorijumu trenutnog radnog direktorijuma kompajlera. Možete dodati direktivu `{ $ UNITPATH ​​.. }` da biste bili sigurni da će unit biti pronađen bez obzira gde se nalazi trenutni radni direktorijum kompajlera.

Kada kompajler traži datoteke unita, dodaje ekstenziju `.ppu` imenu unita. Na Linux - u i operativnim sistemima gde su imena datoteka osetljiva na velika i mala slova prilikom traženja unita, koristi se sledeći mehanizam:

- Unit se prvo traži u originalnom slučaju.
- Unit se traži isključivo malim slovima.
- Unit traži isključivo velikim slovima.

Pored toga, ako je naziv unita duži od 8 znakova, kompajler će prvo potražiti naziv unita sa ovom dužinom, a zatim će skratiti naziv na 8 znakova i ponovo ga potražiti. Iz razloga kompatibilnosti, ovo važi i za platforme koje podržavaju duga imena datoteka.

Imajte na umu da se gore navedena pretraga vrši u svakom direktorijumu u putanji pretrage.

Programski blok sadrži izjave koje će se izvršiti kada se program pokrene. Treba napomenuti da ove izjave ne moraju nužno biti prve koje se izvršavaju: inicijalizacioni kod unita može takođe sadržati izjave koje se izvršavaju pre programskog koda.

Struktura programskog bloka je razmotrena u nastavku.

## 15,2 Uniti

Unit sadrži skup deklaracija, procedura i funkcija koje može da koristi program ili neki drugi unit.

Unit se uvek sastoji od `interface`-a i `implementation` Opciono, postoji blok za inicijalizaciju i blok za finalizaciju, koji sadrže kod koji će se izvršiti kada se program pokrene, odnosno kada se program zaustavi.

I deo za interfejs i deo za implementaciju mogu biti prazni, ali ključne reči `Interface` i Implementation` moraju biti navedene. Sledeći unit je potpuno validan;

```pascal
unit a;

interface

implementation

end.
```

Interfejs deo deklariše sve identifikatore koji moraju biti izvezeni iz unita. To mogu biti konstantne, tipovi ili identifikatori promenljivih, kao i deklaracije identifikatora procedura ili funkcija. Interfejs deo ne može da sadrži kod koji se izvršava, dozvoljene su samo deklaracije. Sledeći je validan interfejs deo:

```pascal
unit a;  
 
interface  
 
uses b;  
 
Function MyFunction : SomeBType;  
 
Implementation

end.
```

Tip SomeBType je definisan u unitu b.

Sve funkcije i metode koje su deklarisane u delu interfejsa moraju biti implementirane u delu implementacije unita, osim deklaracija eksternih funkcija ili procedura. Ako deklarisana metoda ili funkcija nije implementirana u delu implementacije, kompajler će dati grešku, na primer:

```pascal
unit unita;  

interface  

Function MyFunction : Integer;  

implementation  

end.
```

Rezultat će biti sledeća greška:

```sh
unita.pp(5,10) Error: Forward declaration not solved "MyFunction:SmallInt;"
```

`Implementation` deo je prvenstveno namenjen implementaciji funkcija i procedura deklarisanih u `interface` delu. Međutim, on može da sadrži i sopstvene deklaracije: deklaracije unutar implementacionog dela nisu dostupne van unita.

Delovi inicijalizacije i finalizacije unita su opcioni.

Blok inicijalizacije se koristi za inicijalizaciju određenih promenljivih ili izvršavanje koda koji je neophodan za ispravno funkcionisanje unita. Delovi inicijalizacije unita se izvršavaju redosledom kojim ih je kompajler učitao prilikom kompajliranja programa. Izvršavaju se pre nego što se izvrši prva naredba programa.

Deo finalizacije unita se izvršava obrnutim redosledom od izvršavanja inicijalizacije. Koriste se, na primer, za čišćenje bilo kojih resursa dodeljenih u delu inicijalizacije unita ili tokom životnog veka programa. Deo finalizacije se uvek izvršava u slučaju normalnog završetka programa: bilo da je to zato što je dostignut konačni kraj u programskom kodu ili zato što je negde izvršena instrukcija zaustavljanja ( `Halt` ).

U slučaju da se program zaustavi tokom izvršavanja blokova inicijalizacije jednog od unita, biće finalizovane samo uniti koje su već bili inicijalizovani.Imajte na umu da, za razliku od Delfija, u FreePaskalu blok za finalizaciju može biti prisutan bez bloka za inicijalizaciju. To znači da će se sledeće kompajlirati u Fri Paskalu, ali ne i u Delfiju.

```pascal
Finalization  
  CleanupUnit;  
end. 
```

Sam odeljak za inicijalizaciju (tj. bez finalizacije) može se jednostavno zameniti blokom izjava. To jest, sledećim:

```pascal
Initialization  
  InitializeUnit;  
end.
```

je kompletno ekvivalentan:

```pascal
Begin  
  InitializeUnit;  
end.
```

## 15.3 Imenski prostori: Uniti sa tačkom

Naziv unita može da sadrži tačke. To znači da se uniti mogu organizovati u imenskim prostorima.

Dakle, sledeće je korektna unit deklaracija:

```pascal
unit a.b;  
 
interface  
 
Function C : integer;  
 
implementation  
 
Function C : integer;  
begin  
  Result:=1;  
end;  
 
end.
```

Unit može biti korišćen ovako:

```pascal
program d;  
 
uses a.b;  
 
begin  
  Writeln(c);  
end.
```

Kada rešavate simbole, opseg unita uvek ima prednost nad simbolima unutar unita.

S obzirom na sledeće unite:

```pascal
unit myunit;  
 
interface  
 
var  
  test: record  
    a: longint;  
  end;  
 
implementation  
 
initialization  
  test.a:=2;  
end.
```

i

```pascal
unit myunit.test;  
 
interface  
 
var  
  a: longint;  
 
implementation  
 
initialization  
  a:=1;  
end.
```

Sledeći program će razrešiti myunit.test.a u promenljivu a u unitu myunit.test:

```pascal
uses  
   myunit, myunit.test;  

begin  
  Writeln('myunit.test.a : ',myunit.test.a);  
end.
```

To će štampati:

```pascal
myunit.test.a : 1
```

Obrnuti redosled unita neće to promeniti:

```pascal
uses  
   myunit.test, myunit;  
 
begin  
  Writeln('myunit.test.a : ',myunit.test.a);  
end.
```

će takođe štampati

```pascal
myunit.test.a : 1
```

Slično, sledeći program će razrešiti myunit.test.a u varijablu a u unitu myunit.test:

```pascal
uses  
   myunit.test, myunit;  
 
begin  
  Writeln('a : ',a);  
end.
```

to će štampati:

```pascal
a : 1
```

Slično, sledeće ptogram rešava test.a u promenljivu test.a u unitu myunit:

```pascal
uses  
   myunit.test, myunit;  
 
begin  
  Writeln('test.a : ',test.a);  
end.
```

će štampati

```pascal
test.a : 2 
```

## 15.4 Zavisnosti unita

Kada program koristi unit (recimo unitA ) i taj unit koristi drugi unit, recimo unitB, onda program indirektno zavisi i od unitB. To znači da kompajler mora imati pristup unitB kada pokušava da kompajlira program. Ako unit nije prisutan u vreme kompajliranja, dolazi do greške.

Imajte na umu da identifikatori iz unita od koje program indirektno zavisi nisu dostupni programu. Da biste imali pristup identifikatorima unita, unit mora biti u klauzuli `uses` programa ili unita gde su identifikatori potrebni.

Uniti mogu biti međusobno zavisni, odnosno mogu se pozivati jedan na drugi u svojim uses klauzulama. Ovo je dozvoljeno, pod uslovom da se barem jedna od referenci nalazi u odeljku implementacije unita. Ovo važi i za indirektno međusobno zavisne unite.

Ako je moguće početi od jedne klauzule uses interfejsa unita, a vratiti se tamo preko klauzula uses interfejsa, onda postoji kružna zavisnost unita i kompajler će generisati grešku. Na primer,
sledeće nije dozvoljeno:

```pascal
Unit UnitA;  

interface  

Uses UnitB;  

implementation  
end.  
```

```pascal
Unit UnitB  

interface  

Uses UnitA;  

implementation  
end.
```

Ali sledeće je dozvoljeno:

```pascal
Unit UnitA;  

interface  

Uses UnitB;  

implementation  
end.  
```

```pascal
Unit UnitB  

implementation  

Uses UnitA;  

end. 
```

pošto UnitB koristi UnnitA samo u svom odeljku za implementaciju!

Generalno, loša je ideja imati međusobne zavisnosti jedinica, čak i ako je to samo u odeljcima implementacije.

## 15.5 Blokovi

Uniti i programi su napravljeni od blokova. Blok je napravljen od deklaracija oznaka, konstanti, tipova, promenljivih i funkcija ili procedura. Blokovi se mogu ugnježavati na određene načine, tj. deklaracija procedure ili funkcije može sama po sebi imati blokove.

Oznake koje se mogu koristiti za identifikaciju iskaza u bloku deklarišu se u delu deklaracije oznaka tog bloka. Svaka oznaka može identifikovati samo jedan izjavu.

Konstante koje će se koristiti samo u jednom bloku treba deklarisati u delu deklaracije konstanti tog bloka.

Promenljive koje će se koristiti samo u jednom bloku treba deklarisati u delu deklaracije promenljivih tog bloka.

Tipovi koji će se koristiti samo u jednom bloku treba da budu deklarisani u delu deklaracije tipa tog bloka.

Konačno, funkcije i procedure koje će se koristiti u tom bloku mogu se deklarisati u delu deklaracije procedure/funkcije.

Ova četiri dela deklaracije mogu se međusobno mešati, ne postoji obavezan redosled osim što ne možete koristiti (ili se pozivati na) identifikatore koji još nisu deklarisani.

Nakon različitih delova deklaracije dolazi deo sa izjavom. On sadrži sve radnje koje blok treba da izvrši. Svi identifikatori deklarisani pre dela sa izjavom mogu se koristiti u tom delu sa izjavom.

## 15.6 Opseg

Identifikatori su važeći od tačke njihove deklaracije do kraja bloka u kome se deklaracija dogodila. Opseg u kome je identifikator poznat je opseg identifikatora. Tačan opseg identifikatora zavisi od načina na koji je definisan.

### 15.6.1 Opseg bloka

Opseg važenja promenljive deklarisane u delu deklaracije bloka važi od tačke deklaracije do kraja bloka. Ako blok sadrži drugi blok u kome je identifikator ponovo deklarisan, onda će unutar ovog bloka biti važeća druga deklaracija. Nakon napuštanja unutrašnjeg bloka, prva deklaracija je ponovo važeća. Razmotrimo sledeći primer:

```pascal
Program Demo;  

Var 
  X : Real;         { X is real variable }  

Procedure NewDeclaration  
Var 
  X : Integer;      { Redeclare X as integer}  
begin  
  // X := 1.234;    {would give an error when trying to compile}  
  X := 10; { Correct assignment}  
end;  

begin               { From here on, X is Real again}  
  X := 2.468;  
end.
```

U ovom primeru, unutar procedure, X označava celobrojnu promenljivu. Ona ima svoj sopstveni prostor za skladištenje, nezavisan od promenljive X van procedure.

### 15.6.2 Opseg zapisa

Identifikatori polja unutar definicije zapisa su važeći na sledećim mestima:

- Do kraja definicije zapisa.
- Označavači polja promenljive datog tipa zapisa.
- Identifikatori unutar With naredbe koja operiše sa promenljivom datog tipa zapisa.

### 15.6.3 Obim klase

Identifikator komponente (jedna od stavki na listi komponenti klase) je važeći na sledećim mestima:

- Od tačke deklaracije do kraja definicije klase.
- U svim potomstvenim tipovima ove klase, osim ako se ne nalazi u privatnom delu deklaracije klase.
- U svim blokovima deklaracije metoda ove klase i potomstvenih klasa.
- U With naredbi ti operatori na promenljivoj definiciji date klase.

Imajte na umu da se označitelji metoda takođe smatraju identifikatorima.

### 15.6.4 Opseg unita

Svi identifikatori u interfejs delu unita su važeći od tačke deklaracije, pa sve do kraja unita. Štaviše, identifikatori su poznati u programima ili unitima koje imaju unit u svojoj klauzuli uses.

Identifikatori iz indirektno zavisnih unita nisu dostupni. Identifikatori deklarisani u implementacionom delu unita važe od tačke deklarisanja do kraja unita.

Sistem unit se automatski koristi u svim unitima i programima. NJeni identifikatori su stoga uvek poznati, u svakom Paskal programu, biblioteci ili unitu.

Pravila opsega unita podrazumevaju da se identifikator unita može redefinisati. Da biste imali pristup identifikatoru drugog unita koja je redeklarisana u trenutnom unitu, ispred nje stavite ime te druge jedinice, kao u sledećem primeru:

```pascal
unit unitA;  

interface  

Type  
  MyType = Real;  

implementation  
end.  
```

```pascal
Program prog;  

Uses UnitA;  
 
{ Redeclaration of MyType}  
Type 
  MyType = Integer;  

Var 
  A : Mytype;      { Will be Integer }  
  B : UnitA.MyType { Will be real }  

begin  
end.
```

Ovo je posebno korisno prilikom ponovnog deklarisanja identifikatora sistem unita.

[prev][f14] [content][f0] [next][f16]

[f0]: 00_sadrzaj.md
[f14]: 14_generici.md
[f16]: 16_izuzeci.md
