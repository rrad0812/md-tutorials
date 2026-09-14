
# 11 Prošireni zapisi

[prev][f10] [content][f0] [next][f12]

## 11.1 Definicija

Prošireni zapisi su u mnogo čemu ekvivalentni objektima, a u manjoj meri klasama: to su zapisi koji imaju metode povezane sa njima i svojstva. Kao i objekti, kada su definisani kao promenljiva, oni se dodeljuju na steku. Ne moraju da imaju konstruktor. Prošireni zapisi imaju ograničenja u odnosu na objekte i klase po tome što ne dozvoljavaju nasleđivanje i polimorfizam. Nemoguće je kreirati potomak zapisa.

Zašto onda uvoditi proširene zapise? Uveo ih je Delfi 2005 da bi podržao jednu od funkcija koje je uveo .NET. Delfi je zastareo stari TP stil objekata i ponovo uveo funkcije .NET-a kao proširene zapise. FreePaskal teži da bude kompatibilan sa Delfijem, tako da su prošireni zapisi dozvoljeni i u FreePaskalu, ali samo u Delfi režimu.

Ako su potrebni prošireni zapisi u ObjFPC režimu, onda se mora koristiti prekidač režima:

```pascal
{$mode objfpc}  
{$modeswitch advancedrecords}
```

Kompatibilnost nije jedini razlog za uvođenje proširenih zapisa. Postoje neki praktični razlozi za korišćenje metoda ili svojstava u zapisima:

- Više je u skladu sa objektno orijentisanim pristupom programiranju: tip takođe sadrži sve metode koje rade na njemu.
- Za razliku od proceduralnog pristupa, stavljanje svih operacija koje rade na zapisu u sam zapis, omogućava IDE-u da prikaže dostupne metode na zapisu kada prikazuje opcije za dovršavanje koda.

Definisanje proširenog zapisa je slično definisanju objekta ili klase.

Neka od ograničenja u poređenju sa klasama ili objektima:

- Nema nasleđivanja zapisa.
- Ne postoji objavljeni i zaštićeni odeljak.
- Konstruktori ili destruktori ne mogu biti definisani.
- Metode klase (ako ih uopšte možemo tako nazvati) zahtevaju ključnu reč static .
- Metode ne mogu biti virtuelne ili apstraktne – to je posledica činjenice da nema nasleđivanja.

Osim toga, definicija mnogo podseća na definiciju klase ili objekta, osim što se operatori mogu definisati u proširenom zapisu.

Napomena U slučaju varijantnog zapisa, ključna reč `Case` implicitno započinje odeljak `var`, što znači da promenljive ili metode klase nisu dozvoljene u varijantnom delu zapisa.

Sledi nekoliko primera validnih proširenih definicija zapisa:

```pascal
TTest1 = record  
  a : integer;  
  function Test(aRecurse: Boolean): Integer;  
end;  
 
TTest2 = record  
private  
  A,b : integer;  
public  
  procedure setA(AValue : integer);  
  property SafeA : Integer Read A Write SetA;  
end;  
 
TTest3 = packed record  
private  
  fA,fb : byte;  
  procedure setA(AValue : Integer);  
  function geta : integer;  
public  
  property A : Integer Read GetA Write SetA;  
end;  
 
TTest4 = record  
 private  
   a : Integer;  
 protected  
   function getp : integer;  
 public  
   b : string;  
   procedure setp (aValue : integer);  
   property p : integer read Getp Write SetP;  
 public  
 case x : integer of  
   1 : (Q : string[10]);  
   2 : (S : String[10]);  
 end;
```

Imajte na umu da je moguće odrediti vidljivost za članove zapisa. Ovo je posebno korisno, na primer, prilikom kreiranja interfejsa ka C biblioteci: stvarna polja mogu biti deklarisana kao skrivena, a mogu biti izložena svojstva slična „paskalu“ koja deluju kao stvarna polja. Definicija zapisa TTest3 pokazuje da se upakovana direktiva može koristiti u proširenim zapisima. Prošireni zapisi imaju isti raspored memorije kao i njihovi redovni pandani: metode i svojstva nisu deo strukture zapisa u memoriji.

Definicija zapisa TTest4 u gornjim primerima pokazuje da prošireni zapis i dalje ima mogućnost definisanja varijante. Kao i kod redovnog zapisa, varijanta mora biti poslednja. Ne može da sadrži metode.

## 11.2 Prošireni numeratori zapisa

Prošireni zapisi mogu imati enumerator. U tu svrhu, funkcija koja vraća zapis enumeratora mora biti definisana u proširenom zapisu:

```pascal
{$mode objfpc}  
{$modeswitch advancedrecords}  
type  
  TIntArray = array[0..3] of Integer;  
 
  TEnumerator = record  
  private  
    FIndex: Integer;  
    FArray: TIntArray;  
    function GetCurrent: Integer;  
  public  
    function MoveNext: Boolean;  
    property Current: Integer read GetCurrent;  
  end;  
 
  TMyArray = record  
    F: array[0..3] of Integer;  
    function GetEnumerator: TEnumerator;  
  end;  
 
function TEnumerator.MoveNext: Boolean;  
begin  
  inc(FIndex);  
  Result := FIndex < Length(FArray);  
end;  
 
function TEnumerator.GetCurrent: Integer;  
begin  
  Result := FArray[FIndex];  
end;  
 
function TMyArray.GetEnumerator: TEnumerator;  
begin  
  Result.FArray := F;  
  Result.FIndex := -1;  
end;

Nakon ovih definicija, sledeći kod će kompajlirati i nabrojati sve elemente u F:

var  
  Arr: TMyArray;  
  I: Integer;  
begin  
  for I in Arr do  
    WriteLn(I);  
end;
```

Isti efekat se može postići i pomoću enumerator operatora:

```pascal
{$mode objfpc}  
{$modeswitch advancedrecords}  
type  
  TIntArray = array[0..3] of Integer;  
 
  TEnumerator = record  
  private  
    FIndex: Integer;  
    FArray: TIntArray;  
    function GetCurrent: Integer;  
  public  
    function MoveNext: Boolean;  
    property Current: Integer read GetCurrent;  
  end;  
 
  TMyArray = record  
    F: array[0..3] of Integer;  
  end;  
 
function TEnumerator.MoveNext: Boolean;  
begin  
  inc(FIndex);  
  Result := FIndex < Length(FArray);  
end;  
 
function TEnumerator.GetCurrent: Integer;  
begin  
  Result := FArray[FIndex];  
end;  
 
operator Enumerator(const A: TMyArray): TEnumerator;  
begin  
  Result.FArray := A.F;  
  Result.FIndex := -1;  
end; 
```

Ovo će omogućiti i kodu da se pokrene.

## 11.3 Operatori zapisa

Preopterećenje operatora je detaljno obrađeno u poglavlju o preopterećenju operatora. Međutim, pošto je Delfi implementirao preopterećenje operatora kao deo naprednih zapisa, ovde je potrebno reći nekoliko reči o tome.

Kao što se može videti na sintakskom dijagramu za proširene zapise, FPC podržava Delphi sintaksu za operatore na proširenim zapisima. Ova sintaksa je dostupna i u ObjFPC i u Delphi režimu.

U objfpc režimu, operatori moraju biti definisani svojim simboličkim imenima, baš kao i regularna preopterećenja operatora:

```pascal
{$mode objfpc}  
{$modeswitch advancedrecords}  
 
Type  
  TComplex = record  
    Re,Im : Double;  
    class operator +(a,b : TComplex) : TComplex;  
  end;  
 
class operator TComplex.+ (a,b : TComplex) : TComplex;  
begin  
  Result.re:=A.re+B.re;  
  Result.im:=A.im+B.im;  
end
```

Operatori rade isto kao što bi radili da su definisani korišćenjem uobičajene FPC sintakse za operatore, ali ova sintaksa je naravno ograničena na tipove zapisa. Imajte na umu da imaju ključnu reč class ispred sebe i da im je u implementaciji dodato ime tipa zapisa.

Kao što je gore navedeno, u ObjFPC režimu, operatori moraju biti označeni svojim simboličkim nazivima. Nasuprot tome, u Delphi režimu, mogu se koristiti i nazivi operatora, slično sintaksi u Delphi-ju:

```pascal
{$mode delphi}  
Type  
  TComplex = record  
    Re,Im : Double;  
    class operator add(a,b : TComplex) : TComplex;  
  end;  
 
class operator TComplex.add (a,b : TComplex) : TComplex;  
begin  
  Result.re:=A.re+B.re;  
  Result.im:=A.im+B.im;  
end;
```

To je naravno zato što sintaksa mora biti kompatibilna sa Delfijem.

Ispod je tabela koja povezuje imena simboličkih operatora sa imenom u običnom tekstu. Imajte na umu da neki FPC operatori nemaju ekvivalentno ime koje se koristi u Delphi formatu.

[prev][f10] [content][f0] [next][f12]

[f0]: 00_sadrzaj.md
[f10]: 10_klase.md
[f12]: 12_pomoćnici.md
