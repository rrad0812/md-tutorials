
# 3 Tipovi

[prev][f2] [content][f0] [next][f4]

Sve varijable imaju tip. Free Pascal podržava iste osnovne tipove kao Turbo Pascal, sa nekim dodatnim tipovima iz Delphija, kao i nekim sopstvenim.

Programer može deklarisati sopstvene tipove, što u suštini definiše identifikator koji može da se koristi za označavanje ovog prilagođenog tipa kada se deklarišu promenljive dalje u izvornom kodu. Deklarisanje tipa se dešava u bloku `Type`,  koji je kolekcija deklaracija tipa, odvojenih tačkom i zarezom.

Svaki od ovih slučajeva će se posebno razmatrati.

## 3.1 Osnovni tipovi

Osnovni ili jednostavni tipovi Free Pascala su Delphi tipovi. Pričaćemo o svakom odvojeno.

### 3.1.1 Redni tipovi

Sa izutetkom tipova floating point vrednosti, svi osnovni tipovi su `ordinal` tipovi. Redni tipovi imaju sledeće krakteristike:

- **Redni tipovi** su prebrojivi i uređeni, tj., u principu je moguće početi brojati jedan po jedan, određenim redosledom. Ovo svojstvo omogućava da se definiše rad funkcija kao `Inc`, `Ord`, `Dec` na rednim tipovima.
- **Redne vrednosti** imaju najmanju moguću vrednost. Pokušaj primene funkcije `Pred` na najmanju moguću vrednost će generisati `range check error` ako je provera opsega omogućena `{R+}`.
- **Redne vrednosti** imaju najveću moguću vrednost. Pokušaj primene funkcije `Succ` na najveću moguću vrednost će generisati `range check error` ako je provera opsega omogućena `{R+}`.

**Napomena**:
`Int64` i `QWord` se smatraju rednim tipovima na 64-bitnim CPU-ima. Na 32-bitnim tipovima oni imaju neke od rednih karakteristika, ali se ne mogu koristiti npr. u `for` petljama.

#### 3.1.1.1 Intidžeri - celi brojevi

Lista predefinisanih integer tipova je prikazan u Tabeli 3.1:

Tabela 3.1: Predefinisani nteger tipovi

 Name |
 ---- |
 Integer |
 Shortint |
 SmallInt |
 LongInt |
 Longword |
 Int64 |
 Byte |
 Word |
 Cardinal |
 QWord |
 ByteBool |
 WordBool |
 LongBool |
 QWordBool |

Celobrojni tipovi, i njihovi opsezi i veličine, koji su unapred definisani u Free Pascal-u, navedeni su u tabeli (3.2). Imajte na umu da tipovi `qword` i `int64` nisu pravi redni brojevi, tako da neke Pascal konstrukcije neće raditi sa ova dva tipa celih brojeva.

Tabela 3.2: Predefinisani intidžer tipovi

 Type | Range | Size in bytes |
 ----- | ----- | ------------ |
 Byte | 0 .. 255 | 1 |
 Shortint | -128 .. 127 | 1 |
 Smallint | -32768 .. 32767 | 2 |
 Word | 0 .. 65535 | 2 |
 Integer | either smallint or LongInt | size 2 or 4 |
 Cardinal | longword | 4 |
 LongInt | -2147483648 .. 2147483647 | 4 |
 Longword | 0 .. 4294967295 | 4 |
 Int64 | -9223372036854775808 .. 9223372036854775807 | 8 |
 QWord | 0 .. 18446744073709551615 | 8 |

**Napomena**  

- Celobrojni tip (Integer) je pseudonim za tip Smallint u podrazumevanom režimu Free Pascala.
- Celobrojni tip (Integer) je pseudonim za tip LongInt u Delphi ili ObjFPC režimu.
- Cardinal tip je trenutno uvek mapiran u tip duge reči (Longword).

**Napomena**  
Kompajler odlučuje o tipu celobrojne konstante na osnovu vrednosti: Celobrojna konstanta dobija najmanji mogući tip sa znakom. Koristi se prvo podudaranje u tabeli (3.3).

Tabela 3.3: Mapiranje integer konstanti

 Range | Type |
------ | ---- |
-128..127 | Shortint |
128..255 | Byte |
-32768..32767 | Smallint |
32768..65535 | Word |
-2147483648..2147483647 | LongInt |
2147483648..4294967295 | Cardinal (longword) |
-9223372036854775808 .. 9223372036854775807 | Int64 |

To znači da su konstante u opsegu -128..127 mapirane u `shortint`, konstante u opsegu 128..255 su mapirane u `byte`, itd. Konstante u opsegu `2147483647..high(cardinal)` se raščlanjuju kao cardinal, a sve decimalne konstante koje se ne uklapaju u bilo koju od gornjih konstanti su parbitne kao konstante iznad.

**Napomena**  
U novijim verzijama Delphija, `LongInt` tip zavisi od platforme i CPU-a. Ovo nije tako u FPC-u, gde je `LongInt` 32-bitni na svim platformama.

Kao pascal kompajler, Free Pascal vrši automatsku konverziju tipova i nadogradnju u izrazima u kojima se koriste različite vrste celih tipova:

- Svaka platforma ima „prirodnu“ veličinu celog broja, u zavisnosti od toga da li je platforma
  8-bitna, 16-bitna, 32-bitni ili 64-bitni. E. g. na AVR-u ovo je 8-bitno.
– Svaki ceo broj manji od „nativne“ veličine se unapređuje u potpisanu verziju „nativne“
  veličina. Celi brojevi jednaki „nativnoj“ veličini zadržavaju svoj predznak.
- Rezultat binarnih aritmetičkih operatora (+, -, *, itd.) određuje se na sledeći način:
  - Ako je bar jedan od operanada veći od prirodne veličine celog broja, rezultat se bira da bude najmanji tip koji obuhvata opsege tipova oba operanda. To znači da će mešanje neoznačenog sa manjim ili jednakim predznakom proizvesti potpisani tip koji je veći od oba.
  - Ako oba operanda imaju isti predznak, rezultat je istog tipa kao i oni. Jedini izuzetak je oduzimanje (-): u slučaju neoznačenog - oduzimanje bez predznaka daje rezultat sa znakom u FPC (kao u Delphiju, ali ne i u TP7).
- Mešanje potpisanih i neoznačenih operanada „nativne“ veličine int daje veći rezultat sa znakom.
  To znači da će mešanje LongInta i longvord na 32-bitnim platformama proizvesti int64. Slično tome, mešanje bajta i shortinta na 8-bitnim platformama (AVR) će proizvesti smallint.

#### 3.1.1.2 Boolean tipovi

Free Pascal podržava `Boolean` tip, sa svoje dve unapred definisane moguće vrednosti `True` i `False`. Ovo su jedine dve vrednosti koje se mogu dodeliti `Boolean` tipu. Naravno, bilo koji izraz koji se rešava u logičku vrednost, takođe može biti dodeljen logičkom tipu.

Tabela 3.4: Boolean tipovi

Name | Size | Ord(True) |
---- | ---- | --------- |
Boolean | 1 | 1 |
Boolean16 | 2 | 1 |
Boolean32 | 4 | 1 |
Boolean64 | 8 | 1 |
ByteBool | 1 | Any nonzero value |
WordBool | 2 | Any nonzero value |
LongBool | 4 | Any nonzero value |
QWordBool | 8 | Any nonzero value |

Pored jednostavnog tipa „Boolean“, postoje dodatni tipovi „Boolean16“, „Boolean32“ i „Boolean64“. To su u stvari celobrojni tipovi, koji su kompatibilni sa dodeljivanjem sa jednostavnim logičkim tipom. Kao ceo broj, vrednosti za `True` i `False` su `1` i `0`. Ovo se može koristiti za povezivanje sa C kodom koji definiše logički broj ove veličine sa vrednostima `0` i `1`.

Da bi povezivanje sa C bio još lakše, Free Pascal takođe podržava tipove `ByteBool`, `WordBool`, `LongBool` i `QWordBool`. Oni su tipa `Byte`, `Word`, `LongInt` ili `Int64`, ali su opet za dodeljivanje kompatibilni sa `Boolean`. Jedina razlika sa tipovima Boolean16/32/64 je u tome koje se vrednosti smatraju `True` ili `False`“: vrednost `False` je ekvivalentna `0` (nula) i svaka vrednost različita od nule se smatra `True` kada se konvertuje u logičku vrednost. Logička vrednost `True` se konvertuje u `Not(0)` u slučaju da je dodeljena promenljivoj tipa `ByteBool`, `WordBool`, `LongBool` ili `QWordBool`.

Pod pretpostavkom da je B tipa `Boolean`, sledeće su važeće dodele:

> B := True;  
> B := False;  
> B := 1<>2;  { Results in B := True }

Logički izrazi se takođe koriste u uslovima.

**Napomena**:  
U Free Pascal-u, logički izrazi se podrazumevano uvek procenjuju na takav način da kada je rezultat poznat, ostatak izraza više neće biti procenjen: ovo se zove prečica logičke evaluacije.

U sledećem primeru, funkcija Func nikada neće biti pozvana, što može imati čudne sporedne efekte.

```pascal  
 B := False;  
 A := B and Func;
```

Ovde je Func funkcija koja vraća vrednost `Boolean` tipa.

Ovo ponašanje je kontrolisano sa `{$B}` kompajlerskom direktivom.

#### 3.1.1.3 Tipovi nabrajanja

`Tipovi nabrajanja` su podržani u Free Pascal-u. Pored Turbo Pascal implementacije, Free Pascal takođe dozvoljava proširenje tipa nabrajanja u stilu C, gde se vrednost dodeljuje određenom elementu liste nabrajanja. Kada koristite dodeljene nabrojane tipove, dodeljeni elementi moraju biti u rastućem numeričkom redosledu na listi, inače će kompajler da se žali. Izrazi koji se koriste u dodeljenim nabrojanim elementima moraju biti poznati u vreme kompajliranja. Dakle, sledeće je ispravna deklaracija nabrojanog tipa:

```pascal
Type  
  Direction = ( North, East, South, West );
```

C-stil tipa nabrajanja izgleda ovako:

```pascal
Type  
  EnumType = (one, two, three, forty := 40, fortyone);
```

ili možete koristiti:

```pascal
Type  
  EnumType = (one, two, three, forty = 40, fortyone);
```

Poslednja notacija je obavezna u režimu Delphi.

Kao rezultat, redni broj "forty" je 40, a ne 3, kao što bi bilo kada dodele ’:= 40’ nije bilo. Redna vrednost "fortyone" je tada 41, a ne 4, kao što bi bilo kada zadatak dodele nije bio prisutan. Nakon dodele u nabrojanoj definiciji, kompajler dodaje 1 dodeljenoj vrednosti da bi dodelio sledećoj nabrojanoj vrednosti.

Prilikom navođenja takvog tipa nabrajanja, važno je imati na umu da nabrojane elemente treba držati u rastućem redosledu. Sledeće će proizvesti grešku kompajlera:

```pascal
Type  
  EnumType = (one, two, three, forty := 40, thirty := 30);
```

Neophodno je držati četrdeset i trideset u ispravnom redosledu. Kada koristite tipove nabrajanja, važno je imati na umu sledeće:

- Funkcije `Pred` i `Succ` se ne mogu koristiti na ovakvim tipovima nabrajanja. Pokušaj da ovo uradite na bilo koji način će dovesti do greške kompajlera.
- Tipovi nabrajanja se čuvaju koristeći podrazumevanu vrednost, nezavisno od stvarnog broja vrednosti: kompajler ne pokušava da optimizuje prostor. Ovo ponašanje se može promeniti pomoću direktive kompajlera `{$PACKENUM n}`, koja kompajleru govori o minimalnom broju bajtova koji će se koristiti za tipove nabrajanja. na primer:

```pascal
Type  
{$PACKENUM 4}  
  LargeEnum = ( BigOne, BigTwo, BigThree );

{$PACKENUM 1}  
  SmallEnum = ( one, two, three );

Var S : SmallEnum;  
    L : LargeEnum;  
begin  
  WriteLn ('Small enum : ',SizeOf(S));  
  WriteLn ('Large enum : ',SizeOf(L));  
end.
```

će, kada se pokrene štampati:

```sh
Small enum : 1  
Large enum : 4
```

#### 3.1.1.4 Subrange tipovi

Subrange tip je opseg vrednosti iz rednog tipa (tip domaćina). Da bi se definisao subrange tip, moraju se navesti njegove granične vrednosti: najviša i najniža vrednost tipa.

Neki od unapred definisanih tipova celih brojeva su definisani kao tipovi podopsega:

```pascal
Type  
  LongInt  = $80000000..$7fffffff;  
  Integer  = -32768..32767;  
  shortint = -128..127;  
  byte     = 0..255;  
  Word     = 0..65535;
```

Subrange tipovi enumeracionih tipova takodje mogu biti definisani:

```pascal
Type  
  Days = (monday,tuesday,wednesday,thursday,friday,saturday,sunday);  
  WorkDays = monday .. friday;  
  WeekEnd = Saturday .. Sunday;
```

### 3.1.2 Karakter tipovi

#### 3.1.2.1 Char

Free Pascal podržava tip `Char`. `Char` je veličine tačno 1 bajt i sadrži jedan ASCII znak.  

- Karakter konstanta se može navesti tako što se karakter stavi u jednostruke navodnike, na
  sledeći način: 'a' ili 'A' su oba karakter konstante.  

- karakter se takođe može navesti njegovom vrednošću (obično ASCII kod), tako što se
  ispred redne vrednosti nalazi simbol broja (#). Na primer, navođenje `#65` bi bilo isto kao `A`.  

- Takođe, znak za umetanje (`^`) može da se koristi u kombinaciji sa slovom da se navede znak sa
  ASCII vrednošću manjom od 27. Tako je `^G` jednako `#7` - G je sedmo slovo u abecedi. Prevodilac je prilično aljkav u vezi sa znakovima koji su dozvoljeni nakon kareta, ali generalno treba pretpostaviti samo slova.  

- Kada znak jednostrukog navodnika mora biti predstavljen, treba ga otkucati dva puta uzastopno,
  tako da `''''` predstavlja znak jednostrukih navodnika.

#### 3.1.2.2 AnsiChar

Da bi razlikovali `Char` od `WideChar`, `system` unit takođe definiše tip `AnsiChar`, koji je isti kao tip `Char`. U budućim verzijama FPC-a, tip `Char` može postati pseudonim za `WideChar` ili `AnsiChar`.

#### 3.1.2.3 WideChar

Free Pascal podržava tip `WideChar`. `WideChar` je veličine tačno 2 bajta i sadrži jedan `UNICODE` znak u `UTF-16` kodiranju.

- Unicode znak se može navesti njegovom vrednošću karaktera (UTF-16 kod), tako što se ispred
  redne vrednosti nalazi simbol broja (`#`).

- Normalan ansi (1-bajtni) literal karaktera se takođe može koristiti za široki karakter,
  kompajler će ga automatski konvertovati u 2-bajtni UTF-16 karakter.

- Sledeće definiše neke grčke znakove (fi, omega):

  ```pascal
  Const  
    C3 : widechar = #$03A8;  
    C4 : widechar = #$03A9;
  ```

- Isto može biti postignuto typecasting-om `word` u `widechar`:

  ```pascal
  Const  
    C3 : widechar = widechar($03A8);  
    C4 : widechar = widechar($03A9);
  ```

### 3.1.3 Real tipovi

Free Pascal koristi matematički koprocesor (ili emulaciju) za sve svoje proračune sa pokretnim zarezom. Realni izvorni tip zavisi od procesora, ali je ili `Single` ili `Double`. Podržani su samo IEEE tipovi sa pokretnim zarezom, a oni zavise od ciljnog procesora i opcija emulacije. Pravi Turbo Pascal kompatibilni tipovi su navedeni u tabeli (3.5).

Tabela 3.5: Podržani Real tipovi (Turbo Pascal)

Type | Range | Significant digits | Size |
---- | ----- | ------------------ | ---- |
Real | platform dependant | ??? | 4 or 8 |
Single | 1.5E-45 .. 3.4E38 | 7–8 | 4 |
Double | 5.0E-324 .. 1.7E308 | 15–16 | 8 |
Extended | 1.9E-4932 .. 1.1E4932 | 19–20 | 10 |
Comp | -2E64+1 .. 2E63-1 | 19–20 | 8 |
Currency | -922337203685477.5808 .. 922337203685477.5807 | 19–20 | 8 |

`Comp` tip je, u stvari, 64-bitni ceo broj i nije dostupan na svim ciljnim platformama. Da biste dobili više informacija o podržanim tipovima za svaku platformu, pogledajte Vodič za programere.

Tip `Currency` je stvarni tip podataka sa fiksnim zarezom koji se interno koristi kao 64-bitni celobrojni tip (automatski skaliran sa faktorom 10000), što minimizira greške zaokruživanja. Ovaj tip treba koristiti pažljivo: kada se koristi u izrazima koji koriste npr. množenja, procena izraza može poći po zlu (izgubiti preciznost) ako srednji rezultati padnu van opsega valute.

Imajte na umu da nisu svi floating point tipovi dostupni na svim platformama. Tip `Single` je jedini garantovano dostupan na svim platformama koje imaju podršku za plutajući zarez (tako da je, na primer, AVR nema). `Double` tip je dostupan na svim platformama sa koprocesorom, a `Extended` tip je dostupan na svim Intel x86 procesorima, osim na Windows 64-bitnoj platformi. Više detalja o dostupnosti možete pronaći u Vodiču za programere.

## 3.2 String tipovi

### 3.2.1 Single-byte string tipovi

- Ako postoji specifikacija veličine (koristeći uglaste zagrade), to ukazuje na maksimalnu
  veličinu stringa, ako ne to je – 255.
- Ako postoji specifikacija kodne stranice, (koristeći okrugle zagrade) on ukazuje na `AnsiString`
  sa pridruženim informacijama kodne stranice.
- Značenje izjave o nizu bez veličine i oznake kodne stranice se različito tumači u zavisnosti od
  prekidača `{$H}`:
  
  ```pascal
  var  
  A : String;
  ```

- Ako veličina i kodna strana nisu prisutne, gornja deklaracija može deklarisati `AnsiString`
  ili `ShortString`.
- Bez obzira na stvarni tip, jednobajtni stringovi (`AnsiString`ili `ShortString`) se mogu
  koristiti naizmenično. Kompajler uvek vodi računa o potrebnim konverzijama tipova. Imajte na umu, međutim, da će rezultat izraza koji sadrži `AnsiString` i `ShortString` uvek biti `AnsiString`.

#### 3.2.1.1 ShortString

Deklaracija stringa deklariše `ShortString` u sledećim slučajevima:

- Ako je prekidač `$H` isključen: `{$H-}`, deklaracija stringa će uvek biti kratka.
- Ako je prekidač na `{$H+}`, a postoji specifikacija maksimalne dužine (veličine), deklaracija je
  deklaracija kratkog stringa.
- Pretpostavlja se da kratki stringovi uvek koriste sistemsku kodnu stranicu.
- Unapred definisani tip `ShortString` je definisan kao string veličine 255:

  ```pascal
  ShortString = String[255];
  ```

- Ako veličina stringa nije navedena, podrazumevano se uzima 255. Stvarna dužina stringa se može
  dobiti pomoću standardne runtime rutine `Length`. Na primer u:

  ```pascal
  {$H-}  
   
  Type  
    NameString = String[10];  
    StreetString = String;
  ```

  "NameString" može da sadrži najviše 10 znakova, dok "StreetString" može da sadrži do 255 znakova.

**Napomena**
Kratki stringovi imaju maksimalnu dužinu od 255 karaktera: kada navedete maksimalnu dužinu, maksimalna dužina ne sme biti veća od 255. Ako se pokuša dužina veća od 255, kompajler će dati poruku o grešci:

```sh
Greška: dužina stringa mora da bude vrednost od 1 do 255
```

**Napomena**  
Za kratke stringove, dužina se čuva u znaku na indeksu 0. Stari Turbo Pascal kod se oslanja na ovo, a implementiran je na sličan način u Free Pascal-u.

Uprkos tome, da biste pisali prenosivi kod, najbolje je da podesite dužinu kratkog stringa `SetLength` i da je preuzmete pozivom `Length`. Ove funkcije će uvek raditi, bez obzira na internu reprezentaciju kratkih stringova ili drugih stringova koji se koriste: ovo omogućava lako prebacivanje između različitih tipova stringova.

#### 3.2.1.2 AnsiString

`AnsiString` su stringovi koji:

- nemaju ograničenje dužine,  
- imaju pridruženu kodnu stranicu,  
- broje im se reference i  
- garantovano su nulto prekinuti.  

Neke činjenice u vezi `AnsiString`-ova:

- Interno, `AnsiString` se tretira kao pokazivač: stvarni sadržaj stringa se čuva na heap-u,
  dodeljuje se onoliko memorije koliko je potrebno za skladištenje sadržaja stringa.

- Ako u deklaraciji nije data kodna stranica, pretpostavlja se sistemska kodna stranica. Koja je
  ovo kodna stranica, određuje konstanta `DefaultSystemCodePage` u `sistem` unitu. Sve se to radi transparentno, tj. njima se može manipulisati kao normalnim `ShortString`-om.

- `AnsiString`-ovi se mogu definisati korišćenjem unapred definisanog tipa `AnsiString` ili pomoću
  ključne reči `String` u režimu `{$H+}`.

- Null-završetak ne znači da se nulti znakovi (char(0) ili #0) ne mogu koristiti: null-završetak
  se ne koristi interno, ali je tu radi pogodnosti kada se radi sa eksternim rutinama koje očekuju string sa nultom završetkom (kao većina C rutina).

- Ako je prekidač `{$H}` uključen, onda će se definicija stringa koja koristi redovnu ključnu reč
  `String` koja ne sadrži specifikaciju dužine takođe smatrati `AnsiString`-om.
  
- Ako je prisutan specificator dužine, koristiće se kratak string, bez obzira na postavku `{$H}`.

- Ako je string prazan (’’), onda je interna reprezentacija pokazivača stringa `Nil`. Ako string
  nije prazan, onda pokazivač pokazuje na strukturu u memoriji heap-a.

- Unutrašnja reprezentacija kao pokazivač i automatski null-termination omogućavaju da se
  `AnsiString` unese u `pchar`. Ako je string prazan (tako da je pokazivač Nil) onda kompajler osigurava da će tip `pchar` pokazivati na nulti bajt.

- Dodeljivanje jednog `AnsiString`-a drugom ne uključuje pomeranje stvarnog stringa. Izjava

  ```pascal
    S2:=S1;
  ```

  rezultira smanjenjem referentnog broja S2 za 1, referentni broj S1 se povećava za 1, i konačno se S1 (kao pokazivač) kopira u S2. Ovo je značajno ubrzanje koda.

- Ako broj referenci stringa dostigne nulu, tada se memorija koju string zauzima automatski
  oslobađa, a pokazivač je postavljen na nula, tako da ne dolazi do curenja memorije.

- Kada se deklariše `AnsiString`, Free Pascal kompajler u početku dodeljuje samo
  memoriju za pokazivač, ne više. Garantovano je da je ovaj pokazivač nula, što znači da je string u početku prazan. Ovo važi za lokalne i globalne `AnsiString`-ove ili `AnsiString`-ove koji su deo strukture (nizovi, zapisi ili objekti).
  
- Imajte na umu da se rezultat funkcije u ovom pogledu smatra ekvivalentnim
  parametru var i stoga neće biti inicijalizovan na nulu. Kao posledica toga, može ukazivati na legitiman ne-Nil `AnsiString` kada funkcija počne.
  
  Ovo uvodi dodatne troškove. Na primer, izjava:
  
  ```pascal
  var  
    A : Array[1..100000] of string;
  ```
  
  kopiraće vrednost Nil 100.000 puta u A. Kada A izađe van opsega, tada će referentni broj 100.000 stringova biti smanjen za 1 za svaki od ovih stringova. Sve ovo se dešava nevidljivo za programera, ali kada se razmatraju problemi sa performansama, ovo je važno.
  
- Memorija za sadržaj stringa biće dodeljena samo kada je stringu dodeljena
  vrednost. Ako string izađe van opsega, onda se njegov broj referenci automatski smanjuje za 1. Ako broj referenci dostigne nulu, memorija rezervisana za string se oslobađa.

- Ako je vrednost dodeljena znaku stringa koji ima broj referenci veći od 1, kao
  što je u sledećim izjavama:

  ```pascal
    S:=T;  { reference count for S and T is now 2 }  
    S[I]:='@';
  ```
  
  onda se kopija stringa kreira pre dodele. Ovo je poznato kao `semantika kopiranja na pisanje`.

- Moguće je prisiliti string da ima broj referenci jednak 1 pomoću poziva `UniqueString`:

  ```pascal
    S:=T;  
    R:=T; // Reference count of T is at least 3  
    UniqueString(T);  
    // Reference count of T is guaranteed 1
  ```
  
  Preporučuje se da to uradite tj., kada kopirate `AnsiString` u `PChar` var i prosleđujete ga C rutini koja modifikuje string.
  
- Funkcija `Length` se mora koristiti za dobijanje dužine `AnsiString`-a: dužina
  se ne čuva u znaku 0 AnsiStringa. Konstrukt:

  ```pascal
    L:=ord(S[0]);
  ```

  što je važilo za Turbo Pascal `ShortString`, više nije tačno za `AnsiString`. Kompajler će upozoriti ako se naiđe na takvu konstrukciju.
  
- Da biste podesili dužinu `AnsiStringa`, mora se koristiti funkcija  
  `SetLength`. Konstantni `AnsiString`-ovi imaju referentni broj -1 i tretiraju se posebno. Mora se dati ista primedba kao i za dužinu: Konstrukcija:

  ```pascal
    L:=12;  
    S[0]:=Char(L);
  ```
  
  što je važilo za Turbo Pascal `ShortString`, više nije tačno za `AnsiString`. Kompajler će upozoriti ako se naiđe na takvu konstrukciju.
  
- Po potrebi kompajler konvertuje `AnsiString`-ove u `ShortString`-ove, što znači da se upotreba `AnsiString`-a i `ShortString`-a može mešati bez problema.

- `AnsiString`-ovi se mogu typecast-ovati na tipove `PChar` ili `Pointer`:

  ```pascal
  Var P : Pointer;  
      PC : PChar;  
      S : AnsiString;  
   
  begin  
    S :='This is an ansistring';  
    PC:=PChar(S);  
    P :=Pointer(S);
  ```

  Postoji razlika između ova dva typecast-a.

  Kada se prazan `AnsiString` konvertuje u pokazivač, pokazivač će biti Nil. Ako se prazan  
  `AnsiString` unese u PChar, onda će rezultat biti pokazivač na nulti bajt (prazan string).
  
  Rezultat takvog typecastinga se mora pažljivo koristiti. Uopšteno govoreći, najbolje je smatrati da je rezultat takvog typecatinga samo za čitanje, tj. pogodan samo za prelazak na proceduru kojoj je potreban argument konstantan `pchar`.
  
  Stoga nije preporučljivo unositi stringove koji imaju broj referenci veći od 1. U ovom slučaju treba da pozovete `UniqueString` da biste bili sigurni da string ima broj referenci 1.
  
#### 3.2.1.3 Konverzija kodne strane

Pošto stringovi imaju informacije o kodnoj stranici povezane sa njima, važno je znati koju kodnu stranicu string koristi:

- `ShirtString` uvek koriste sistemsku kodnu stranicu.
- `AnsiString` koristi sistemsku kodnu stranicu.
- `AnsiString` sa deklarisanom kodnom stranicom koriste tu kodnu stranicu.
- `RowByteString` nema pridružene informacije o kodnoj stranici.
- `Konstantni stringovi` imaju kodnu stranicu izvorne datoteke. Ako nijedna nije
  navedena koristi se sistemska kodna stranica (CP_ACP). Pogledajte Vodič za programere, direktiva
  `{$CODEPAGE }`. Ova kodna stranica se naziva `deklarisana kodna stranica`.

Kompajler će konvertovati kodnu stranicu stringova po potrebi Prilikom dodeljivanja stringa, stvarna kodna stranica izvornog stringa će biti konvertovana u deklarisanu kodnu stranicu ciljnog stringa ako su deklarisane izvorne i ciljne kodne stranice različite.

Ako je string sa deklarisanom stranicom SOURCE_CP dodeljen nizu sa deklarisanom kodnom stranicom DEST_CP, u datoteci sa kodnom stranicom CODE_CP onda sledeće opisuje mehanizam:

- ako (SOURCE_CP = `CP_NONE`) ili (DEST_CP = `CP_NONE`), pogledajte `RowByteString`.
- ako (`CODE_CP` ili `CP_ACP`), onda ako (DEST_CP = `CP_ACP`) i (SOURCE_CP = `CODE_CP`)
  ili obrnuto, neće doći do konverzije, čak i ako u vreme izvršavanja `DefaultSistemCodePage` ima drugačiju vrednost od SOURCE_CP.
– Razlog za ovo stanje (`CODE_CP` ili `CP_ACP`) je kompatibilnost unatrag sa
  prethodnim FPC verzijama:
  - Iako nisu podržavali AnsiStrings sa proizvoljnim kodnim stranicama, uvek su ponovo
    interpretirali AnsiStrings prema trenutnoj vrednosti sistemske kodne stranice.
  - Inače, if (SOURCE_CP ili DEST_CP), podaci stringa će biti konvertovani iz
    kodne stranice SOURCE_CP u kodnu stranicu DEST_CP pre dodeljivanja, pri čemu će CP_ACP biti interpretiran kao trenutna vrednost DefaultSistemCodePage. inače, ako (SOURCE_CP = DEST_CP), konverzija kodne stranice neće biti izvršena.

Ova pravila znače da je sasvim moguće da `AnsiString` promenljiva dobije kodnu stranicu koja se razlikuje od deklarisane kodne stranice. Tj. u trećem slučaju SOURCE_CP može biti `CP_ACP`, dok nakon dodele može imati dinamičku kodnu stranicu jednaku `DefaultSistemCodePage`.

**Napomena**:
Kao što je gore pomenuto, da li će doći do potencijalne konverzije kodne stranice ili ne zavisi samo od deklarisanih kodnih stranica uključenih stringova. To znači da ako dodelite jedan `AnsiString(X)` drugom `AnsiString(X)` i dinamički kod prvog je bio drugačiji od X, podaci stringa neće biti konvertovani u kodnu stranicu X dodelom.

Sve ovo znači da u sledećem kodu:

```pascal
{$h+}  
uses sysutils;  
 
Type  
  TString1 = Type String(1252);  
  TString2 = Type String(1251);  
 
Var  
  A : TString1;  
  B : TString2;  
 
begin  
  A:='123'+'345'+intToStr(123);  
  B:=A;  
  Writeln('B: "',B,'" : ',StringRefCount(B),' -> ',StringCodePage(B));  
  Writeln('A: "',A,'" : ',StringRefCount(A),' -> ',StringCodePage(A));  
end.

This will print:
B: "123345123" : 1 -> 1251  
A: "123345123" : 1 -> 1252
```

Tokom dodeljivanja B := A, sadržaj stringa A se konvertuje u kodnu stranicu stringa B. Imajte na umu da ako dođe do konverzije kodne stranice, mehanizam brojanja referenci se ne koristi: biće dodeljen novi string.

Ova automatska konverzija kodnih stranica može ozbiljno da uspori kod, pa se mora voditi računa da konverzije kodnih stranica budu ograničene na minimum.

Kodna stranica stringa može se eksplicitno postaviti korišćenjem rutine `SetCodePage` `sistem` unita. Pozivanje ove rutine će konvertovati vrednost stringa u traženu kodnu stranicu.

**Napomena**:
Konverzije kodne stranice mogu dovesti do gubitka podataka: ako određeni znak ne može biti predstavljen na ciljnoj kodnoj stranici, izlaz za taj znak je nedefinisan.

**Napomena**
Kada je string čija je statička kodna stranica jednaka kodnoj stranici izvorne datoteke, na bilo šta sa kodnom stranicom CP_ACP (tj., običan ansisting, kratki string ili pchar), takođe neće biti izvršena konverzija. Konverzija kodne stranice se ne vrši kada može dovesti do gubitka podataka: ako određeni znak ne može biti predstavljen u ciljnoj kodnoj stranici, izlaz za taj znak je nedefinisan.

**Napomena**
Podrška kodne stranice zahteva dosta pomoćnih rutina, koje su implementirane u menadžeru `UnicodeString`-a. Na Windows-u se za to koriste sistemske rutine. Na Unices-u, `cvstring` unit se može koristiti za povezivanje sa C bibliotekom i korišćenje podrške za konverziju C biblioteke. Alternativno, unit `fpwidestring` sadrži menadžer `UnicodeString`-a koji je nativno implementiran u Object Pascal.

#### 3.2.1.4 RawByteString

Predefinisani `RawByteString` tip je `AnsiString` tip bez informacija o kodnoj strani (`CP_NONE`):

```pascal
Type  
  RawByteString = type AnsiString(CP_NONE);
```

Tretira se posebno u smislu da ako rutine konverzije naiđu na `CP_NONE` u izvornom ili ciljnom stringu, konverzija kodne stranice se ne vrši, kodna stranica izvornog stringa se čuva.

Iz tog razloga, većina rutina jednobajtnih stringova u `system` i `sysutils` unitima koristi tip `RowByteString`.

#### 3.2.1.5 UTF8String

Stringovi kodne stranice od jednog bajta mogu da čuvaju samo karaktere dostupne na toj kodnoj stranici. Znakovi koji nisu prisutni u kodnoj stranici ne mogu biti predstavljeni tim stringom.

UTF-8 unicode kodiranje je kodiranje koje se može koristiti sa jednobajtnim stringovima: ASCII karakteri (redna vrednost > 128) u ovom kodiranju se mapiraju tačno u `CP_ACP` kodiranje. Ova činjenica se koristi za definisanje tipa stringa od jednog bajta koji može da sadrži sve karaktere:

```pascal
Type  
  UTF8String = type AnsiString(CP_UTF8);
```

`UTF8string` tip stringa se može koristiti za predstavljanje svih Unicode znakova. Međutim, ova moć dolazi po određenoj ceni. Pošto unikod karakter može zahtevati nekoliko bajtova da bude predstavljeno u UTF-8 kodiranju, postoje dve tačke o kojima treba voditi računa kada koristite UTF8String:

- Indeks znakova – koji vraća znak veličine bajta na određenoj poziciji
- Mora se koristiti pažljivo: izraz S[i] neće nužno biti važeći karakter za string S tipa
  `UTF8String`.
- Dužina bajtova stringa nije jednaka broju znakova u string. Standardna `Length` funkcije se ne
  može koristiti za dobijanje dužine stringa, ona će uvek vratiti dužinu stringa u bajtovima.

Za sve ostale kodne stranice, broj znakova u stringu kodne stranice od jednog bajta jednak je dužini stringa u bajtovima.

### 3.2.2 Multi-byte String types

Za tipove višebajtnih stringova, osnovni znak ima veličinu od najmanje 2. To znači da se može koristiti za čuvanje unikod karaktera u UTF16 ili UCS2 kodiranju.

#### 3.2.2.1 UnicodeStrings

`UnicodeString`-ovi (koji se koriste za predstavljanje stringova sa unicode znakovima) se implementiraju na isti način kao i `AnsiString`: stringovi kojima se broje reference, null-terminirani nizovi, samo što su implementirani kao `WideChar` stringovi umesto regularnih `AnsiChar`. `WideChar` je znak od dva bajta (element DBCS: Double Byte Character Set). Za `UnicodeStrings` važe uglavnom ista pravila kao i za `AnsiStrings`. Kompajler transparentno konvertuje `UnicodeString`-ove u `AnsiString`-ove i obrnuto.

Slično tipifikaciji `Ansistring`-a stringa znakova sa nultim završavanjem `PChar`-a, `UnicodeString` se može konvertovati u string znakova sa nultim završavanjem `PUnicodeChar`. Imajte na umu da se string `PUnicodeChar` završava sa 2 nulta bajta umesto sa 1, tako da prevođenje tipa na `pchar` nije automatsko.

Sam kompajler ne pruža podršku za bilo kakvu konverziju iz Unicode-a u AnsiString ili obrnuto. `System` unit ima zapis menadžera `UnicodeString`-a, koji se može inicijalizovati nekim rutinama za rukovanje Unicode specifičnim za OS. Za više informacija pogledajte referencu `system` unita.

Unicode string literal se može konstruisati na sličan način kao široki znak:

```pascal
Const  
  ws2: unicodestring = 'phi omega` : '#$03A8' '#$03A9`;
```

#### 3.2.2.2 WideStrings

Tip `WideString` (koji se koristi za predstavljanje stringova unikodnih znakova u COM aplikacijama) implementiran je na isti način kao UnicodeString na Windows-u, a na drugim platformama su jednostavno istog tipa. Ako interakcija sa COM nije potrebna, treba koristiti tip `UnicodeString`.

U Windows-u, za razliku od `UnicodeString`-a, tip `WideString` se ne računa na referencu i dodeljuje im se posebnom funkcijom prozora koja im omogućava da se koriste za OLE automatizaciju. To znači da se implementiraju kao stringovi `WideChars`-a sa nultom završetkom umesto regularnih znakova. `WideString` poštuje ista pravila kao za `UnicodeStringova`. Slično kao `UnicodeString`-ova, kompajler transparentno konvertuje `WideString` u `AnsiString` i obrnuto.

Za prevođenje i konverziju važe ista pravila kao i za tip `UnicodeString`.

Imajte na umu da se na Windows, pošto se `WideString` string dodeljuje pomoću posebne funkcije, raspored memorije razlikuje od `UnicodeString`-a. Dužina se, na primer, čuva u bajtovima, a ne u znakovima.

### 3.2.3 Const stringovi

Da biste naveli konstantni string, on mora biti zatvoren u jednostrukim navodnicima, baš kao tip `Char`, samo što je sada dozvoljeno više od jednog karaktera. S obzirom da je S tipa `String`, sledeće su važeće dodele:

```pascal
S := 'This is a string.';  
S := 'One' + ', Two' + ', Three';  
S := 'This isn't difficult!';  
S := 'This is a weird character : '#145' !';
```

Kao što se može videti, jednostruki navodnik je predstavljen sa 2 znaka jednostrukog navodnika jedan pored drugog. Čudni znakovi se mogu specificirati njihovom vrednošću karaktera (obično ASCII kod). Primer takođe pokazuje da se mogu dodati dva stringa. Dobijeni string je samo spoj prvog sa drugim stringom, bez razmaka između njih. Međutim, stringovi se ne mogu oduzimati.

Da li je konstantni string sačuvan kao `AnsiString` ili `ShortString` zavisi od podešavanja prekidača `{$H}`.

### 3.2.4 PChar – Null terminated strings

Free Pascal podržava Delphi implementaciju tipa `PChar`. `PChar` je definisan kao pokazivač na tip `Char`, ali dozvoljava dodatne operacije. Tip `PChar` se najbolje može razumeti kao Pascal ekvivalent stringa sa nultim završavanjem u C stilu, tj. promenljiva tipa `PChar` je pokazivač koji ukazuje na string tipa `Char`, koji se završava nul-karakterom (#0). Free Pascal podržava inicijalizaciju `PChar` tipiziranih konstanti ili direktno dodeljivanjem. Na primer, sledeći delovi koda su ekvivalentni:

```pascal
program one;  

var 
  P : PChar;  

begin  
  P := 'This is a null-terminated string.';  
  WriteLn (P);  
end.
```

Rezultat je isti u:

```pascal
program two;  

const 
  P : PChar = 'This is a null-terminated string.';  

begin  
  WriteLn (P);  
end.
```

Ovi primeri takođe pokazuju da je moguće upisati sadržaj stringa u datoteku tipa Text. Unit `String` sadrži procedure i funkcije koje manipulišu tipom `PChar` kao u standardnoj C biblioteci. Pošto je ekvivalentan pokazivaču na promenljivu tipa `Char`, moguće je uraditi i sledeće:

```pascal
Program three;  

Var 
  S : String[30];  
  P : PChar;  

begin  
  S := 'This is a null-terminated string.'#0;  
  P := @S[1];  
  WriteLn (P);  
end.
```

Ovo će imati isti rezultat kao prethodna dva primera. Stringovi koji se završavaju nulom ne mogu se dodati kao normalni Pascal stringovi. Ako dva PChar niza moraju biti spojena; moraju se koristiti funkcije iz `String` unita.

Međutim, moguće je napraviti neku aritmetiku pokazivača. Operatori + i - se mogu koristiti za obavljanje operacija na PChar pokazivačima. U tabeli (3.6), P i K su tipa `PChar`, a I tipa `LongInt`.

Tabela 3.6: PChar pointerska aritmetika

Operacija | Rezultat |
--------- | -------- |
P + I | Dodaj I na adresu pokazanu sa P. |
I + P | Dodaj I na adresu pokazanu sa P. |
P - I | Oduzima I od adrese pokazane sa P. |
P - Q | Vraća, kao intidžer, rastojanje izmedju dve adrese (ili broj karaktera izmedju P i Q) |

### 3.2.5 Veličina stringa

Memorija koju string zauzima zavisi od tipa stringa. Neki tipovi stringova dodeljuju stringu memoriju na heapu, drugi imaju stringove na steku. U tabeli (3.7) rezimiramo upotrebu memorije različitih tipova stringova. U tabeli se koriste sledeće simboličke konstante:

- `L` je stvarna dužina niza.
- `HS` zavisi od verzije Free Pascal-a, ali ima 16 bajtova od Free Pascal-a 2.7.
  1.
- `UHS` veličina je 8 bajtova za sve verzije Free Pascal-a.
- Na Windows-u, veličina `WHS`-a je 4 bajta za sve verzije Free Pascal-a. Na
  svim ostalim platformama, `WHS` je jednak `UHS` jer je tip `WideString` jednak `UnicodeString` tipu.

Tabela 3.7: Veličina strnga u memoriji

String type | Stack size | heap size |
----------- | ---------- | --------- |
ShortString | Declared length + 1 | 0 |
AnsiString | Pointer size | L + 1 + HS |
WideString | Pointer size | 2*(L + 1) + WHS |
UnicodeString | Pointer size | 2*(L + 1) + UHS |
PChar | Pointer size | L+1 |

## 3.3 Struktuirani tipovi

Strukturirani tip je tip koji može da sadrži više vrednosti u jednoj promenljivoj. Strukturirani tipovi mogu biti ugnežđeni na neograničeno nivoa.

Za razliku od Delphija, Free Pascal ne podržava ključnu reč `packed` za sve strukturirane tipove. U narednim odeljcima se govori o svakom od mogućih strukturiranih tipova. Pomenuće se kada tip podržava ključnu reč `packed`.

### 3.3.1 Pakovani struktuirani tipovi

Kada se deklariše strukturirani tip, ne treba praviti pretpostavke o unutrašnjem položaju elemenata u tipu. Kompajler će postaviti elemente strukture u memoriju onako kako misli da će biti najpogodnije. To jest, redosled elemenata će se zadržati, ali lokacija elemenata nije zagarantovana i delimično je regulisana direktivom `$PACKRECORDS` (ova direktiva je objašnjena u Vodiču za programere).

Međutim, Free Pascal omogućava kontrolu rasporeda pomoću ključnih reči `Packed` i `Bitpacked`. Značenje ovih reči zavisi od konteksta:

- **Bitpacked**  
  U ovom slučaju, kompajler će pokušati da poravna redne tipove na granicama bita, kao što je objašnjeno u nastavku.

- **Packed**
  Značenje ključne reči `Packed` zavisi od situacije:

  - U MACPAS režimu, to je ekvivalentno ključnoj reči `Bitpacked`.
  - U drugim režimima, kada je direktiva `$BITPACKING` postavljena na ON, takođe
    je ekvivalentno ključnoj reči `Bitpacked`.
  - U drugim režimima, kada je direktiva `$BITPACKING` podešena na OFF, to znači
    normalno pakovanje na granicama bajtova.

Pakovanje na granicama bajtova znači da svaki novi element strukturiranog tipa počinje na granici bajta.

Mehanizam pakovanja bajtova je jednostavan: kompajler poravnava svaki element strukture na prvoj dostupnoj granici bajta, čak i ako je veličina prethodnog elementa (mali nabrojani tipovi, tipovi podopsega) manja od bajta.

Kada se koristi `bitpacked` mehanizam za pakovanje bitova, kompajler izračunava za svaki redni tip koliko bitova je potrebno za njegovo skladištenje. Sledeći redni tip se zatim čuva na sledećem slobodnom bitu.

Neredni tipovi – koji uključuju, ali nisu ograničeni na – skupove, float, stringove, (upakovane bitovima) zapise, (bitove upakovane) nizove, pokazivače, klase, objekte i proceduralne varijable, čuvaju se na prvoj dostupnoj granici bajta.

Imajte na umu da je unutrašnjost bitova "neprozirna": može se promeniti u bilo kom trenutku u budućnosti. Štaviše: unutrašnje pakovanje zavisi od endiannessa platforme za koju se kompilacija vrši i nije moguća konverzija između platformi. Ovo čini strukture sa bitovima neprikladnim za skladištenje na disku ili transport preko mreža. Međutim, format je isti kao onaj koji koristi GNU Pascal kompajler, a tim što Free Pascal ima za cilj da zadrži ovu kompatibilnost u budućnosti.

Postoje još neka ograničenja za elemente `bitpacked` struktura:

- Adresa se ne može preuzeti, osim ako je veličina bitnaa višestrukost od 8 i
  element se čuva na granici bajta.

- Element strukture sa bitovima ne može se koristiti kao var parametar, osim ako
  veličina bita je višestruka od 8 i element se dešava da se čuva na granici bajta.

Da bi se odredila veličina elementa u strukturi sa bitovima, postoji funkcija `BiteSizeOf`. Vraća veličinu – u bitovima – elementa. Za druge tipove ili elemente struktura koji nisu spakovani u bitovima, ovo će jednostavno vratiti veličinu u bitovima pomnoženu sa 8, tj. povratna vrednost je tada ista kao 8*SizeOf.

Veličina bitnih zapisa i nizova je ograničena:

- Na 32-bitnim sistemima maksimalna veličina je 229 bajtova (512 MB).
- Na 64-bitnim sistemima maksimalna veličina je 261 bajt.

Razlog je taj što pomak elementa mora biti izračunat sa maksimalnom celobrojnom veličinom sistema.

### 3.3.2 Nizovi

Besplatni Pascal podržava nizove kao u Turbo Pascal-u. Podržani su i višedimenzionalni nizovi i (bit) upakovani nizovi, kao i dinamički nizovi Delphija:

#### 3.3.2.1 Statički nizovi

Kada je veličina niza uključena u definiciju niza, on se naziva `statičkim` nizom. Pokušaj pristupa elementu sa indeksom koji je izvan deklarisane veličine će generisati grešku u vremenu izvođenja (ako je provera opsega `{R+}` uključena). Sledi primer važeće deklaracije niza:

```pascal
Type  
  RealArray = Array [1..100] of Real;
```

Važeći indeksi za pristup elementu niza su između 1 i 100, gde su granice 1 i 100 uključene. Kao i u Turbo Pascal-u, ako je tip komponente niza sam po sebi niz, moguće je kombinovati dva niza u jedan višedimenzionalni niz. Sledeća deklaracija:

```pascal
Type  
  APoints = array[1..100] of Array[1..3] of Real;
```

je ekvivalentna deklaraciji:

```pascal
Type  
  APoints = array[1..100, 1..3] of Real;
```

Funkcije `High` i `Low` vraćaju gornju i donju granicu najlevog tipa indeksa niza. U gornjem slučaju, ovo bi bilo 100 i 1. Trebalo bi da ih koristite kad god je to moguće, pošto to poboljšava mogućnost održavanja vašeg koda. Upotreba obe funkcije je jednako efikasna kao i upotreba konstanti, jer se procenjuju u vreme kompajliranja.

Kada se statičke promenljive tipa niza dodeljuju jedna drugoj, sadržaj celog niza se kopira. Ovo važi i za višedimenzionalne nizove:

```pascal
program testarray1;  
 
Type  
  TA = Array[0..9,0..9] of Integer;  
 
var  
  A,B : TA;  
  I,J : Integer;  

begin  
  For I:=0 to 9 do  
    For J:=0 to 9 do  
      A[I,J]:=I*J;  
  For I:=0 to 9 do  
    begin  
    For J:=0 to 9 do  
      Write(A[I,J]:2,' ');  
    Writeln;  
    end;  
  B:=A;  
  Writeln;  
  For I:=0 to 9 do  
    For J:=0 to 9 do  
      A[9-I,9-J]:=I*J;  
  For I:=0 to 9 do  
    begin  
    For J:=0 to 9 do  
      Write(B[I,J]:2,' ');  
    Writeln;  
    end;  
end.
```

Izlaz ovog programa biće dve identične matrice.

#### 3.2.2.2 Dynamički nizovi

Od verzije 1.1, Free Pascal takođe poznaje dinamičke nizove: U tom slučaju veličina niza je izostavljena, kao u sledećem primeru:

```pascal
Type  
  TByteArray = Array of Byte;
```

Kada se deklariše promenljiva tipa dinamičkog niza, početna dužina niza je nula. Stvarna dužina niza mora biti podešena pomoću standardne funkcije `SetLength`, koja će dodeliti neophodnu memoriju da sadrži elemente niza na heap-u.

Sledeći primer će postaviti dužinu na 1000:

```pascal
Var  
  A : TByteArray;  
 
begin  
  SetLength(A, 1000);
```

Nakon poziva `SetLength`, važeći indeksi niza su od 0 do 999: indeks dinamičkog niza je uvek zasnovan na nuli.

`SetLength` se takođe može koristiti za višedimenzionalne nizove. Sledeći primer će kreirati "pravougaoni" niz:

```pascal
Var  
  A : Array of TByteArray;  
 
begin  
  SetLength(A, 10, 100);
```

Nakon poziva `SetLength`, važeći indeksi niza su 0 do 9 za prvu dimenziju i 0 do 99 za drugu dimenziju.

Za razliku od statičkih višedimenzionalnih nizova, dinamički nizovi ne moraju biti „pravougaoni“, tj. e. različiti elementi mogu imati različite dužine:

```pascal
var  
  a: array of array of array of LongInt;  
  i, j, k: LongInt;  
begin  
  SetLength(a, 10, 5);  
  SetLength(a[5], 3);  
 
  for i := Low(a) to High(a) do  
    for j := Low(a[i]) to High(a[i]) do begin  
      SetLength(a[i, j], i * 10 + j);  
      for k := Low(a[i, j]) to High(a[i, j]) do  
        a[i, j, k] := i * 10000 + j * 100 + k;  
    end;  
 
  for i := Low(a) to High(a) do begin  
    for j := Low(a[i]) to High(a[i]) do begin  
      for k := Low(a[i, j]) to High(a[i, j]) do  
        Writeln(a[i, j, k]);  
      Writeln('-------');  
    end;  
    Writeln('=======');  
  end;  
end.
```

Imajte na umu da je dužina niza postavljena u elementima, a ne u bajtovima dodeljene memorije (iako oni mogu biti isti). Količina dodeljene memorije je veličina niza pomnožena sa veličinom 1 elementa u nizu. Memorija će biti odložena na izlazu iz trenutne procedure ili funkcije.

Takođe je moguće promeniti veličinu niza: u tom slučaju će se zadržati onoliko elemenata u nizu koliko stane u novu veličinu. Veličina niza se može promeniti na nulu, što efektivno resetuje promenljivu.

U svakom trenutku, pokušaj da se pristupi elementu niza sa indeksom koji nije u trenutnoj dužini niza će generisati grešku tokom izvršavanja.

Dinamički nizovi broje reference: dodeljivanje jedne promenljive dinamičkog tipa niza drugoj će omogućiti obe varijable da upućuju na isti niz. Suprotno `AnsiString`-ovima, dodeljivanje elementu jednog niza će se odraziti u drugom: nema kopiranja pri pisanju. Razmotrite sledeći primer:

```pascal
Var  
  A,B : TByteArray;  
 
begin  
  SetLength(A,10);  
  A[0]:=33;  
  B:=A;  
  A[0]:=31;
```

Posle druge dodele, prvi element u nizu B će takođe sadržati 31.

To se takođe može videti iz rezultata sledećeg primera:

```pascal
program testarray1;  
 
Type  
  TA = Array of array of Integer;  
 
var  
  A,B : TA;  
  I,J : Integer;  
begin  
  Setlength(A,10,10);  
  For I:=0 to 9 do  
    For J:=0 to 9 do  
      A[I,J]:=I*J;  
  For I:=0 to 9 do  
    begin  
    For J:=0 to 9 do  
      Write(A[I,J]:2,' ');  
    Writeln;  
    end;  
  B:=A;  
  Writeln;  
  For I:=0 to 9 do  
    For J:=0 to 9 do  
      A[9-I,9-J]:=I*J;  
  For I:=0 to 9 do  
    begin  
    For J:=0 to 9 do  
      Write(B[I,J]:2,' ');  
    Writeln;  
    end;  
end.
```

Izlaz ovog programa će biti matrica brojeva, a zatim ista matrica, preslikana.

Kao što je ranije napomenuto, dinamički nizovi broje referenci: ako u jednom od prethodnih primera A izađe van opsega, a B ne, onda niz još nije uklonjen: broj referenci A (i B) se smanjuje za 1. Čim broj referenci dostigne nulu, memorija dodeljena sadržaju niza se uklanja.

Poziv `SetLength` će se uveriti da je broj referenci vraćenog niza 1, to jest, ako su dve dinamičke varijable niza ukazivale na istu memoriju, one to više neće činiti nakon poziva sa dužinom:

```pascal
program testunique;  
 
Type  
  TA = array of Integer;  
 
var  
  A,B : TA;  
  I : Integer;  
 
begin  
  Setlength(A,10);  
  
  For I:=0 to 9 do  
    A[I]:=I;  
  
  B:=A;  
  
  SetLength(B,6);  
  
  A[0]:=123;  
  
  For I:=0 to 5 do  
    Writeln(B[I]);  
end.
```

Takođe je moguće kopirati i/ili promeniti veličinu niza pomoću standardne funkcije Copy, koja deluje kao funkcija kopiranja za nizove:

```pascal
program testarray3;  
 
Type  
  TA = array of Integer;  
 
var  
  A,B : TA;  
  I : Integer;  
 
begin  
  Setlength(A,10);  
  For I:=0 to 9 do  
      A[I]:=I;  
  B:=Copy(A,3,6);  
  For I:=0 to 5 do  
    Writeln(B[I]);  
end.
```

Funkcija `Copy` će kopirati šest elemenata niza u novi niz. Počevši od elementa sa indeksom 3 (tj. četvrti element) niza.

Funkcija `Length` će vratiti broj elemenata u nizu. Funkcija `Low` na dinamičkom nizu će uvek vratiti 0, a `High` funkcija će vratiti vrednost `Length-1`, tj. vrednost najvećeg dozvoljenog indeksa niza.

#### 3.2.2.3 Kompatabilnost dinamičkih tipova nizova

Object Pascal je strogo tipizirani jezik. Dva tehnički različita tipa se ponekad smatraju kompatibilnim sa dodeljivanjem (tj. vrednost jednog tipa može biti dodeljena promenljivoj drugog tipa) pod određenim okolnostima. Dinamički nizovi se smatraju kompatibilnim sa dodeljivanjem kada koriste isti tip elementa. To znači da će se sledeće kompajlirati:

```pascal
{$mode objfpc}  
 
Type  
  TA = Array of Integer;  
  TB = Array of Integer;  
 
Var  
  A : TA;  
  B : TB;  
 
begin  
  SetLength(A,1);  
  A[0]:=1;  
  B:=A;  
end.
```

Ali sledeće neće, iako su integer i word tipovi kompatabilni za dodeljivanje:

```pascal
{$mode objfpc}  
 
Type  
  TA = Array of Word;  
  TB = Array of Integer;  
 
Var  
  A : TA;  
  B : TB;  
 
begin  
  SetLength(A,1);  
  A[0]:=1;  
  B:=A;  
end.
```

#### 3.2.2.4 Konstruktor dinamičkog niza

Od verzije 3.0 Free Pascal-a, tipovi dinamičkog niza imaju konstruktor. Ovo je suštinski, kompajler to obezbeđuje. Do verzije 2.6.4, jedini način da se inicijalizuje dinamički niz bio je sledeći:

```pascal
Type  
  TIntegerArray = Array of Integer;  
 
var  
  A : TIntegerArray;  
 
begin  
  SetLength(A,3);  
  A[0]:=1;  
  A[1]:=2;  
  A[3]:=3;  
  Writeln(Length(A));  
end.
```

Od verzije 3.0 Free Pascal-a, dinamički niz se može inicijalizovati korišćenjem sintakse slične konstruktoru. Konstruktor se zove `Create` i prihvata kao parametre promenljiv broj argumenata tipa elementa niza. To znači da se gornja inicijalizacija sada može uraditi kao:

```pascal
Type  
  TIntegerArray = Array of Integer;  
 
var  
  A: TIntegerArray;  
 
begin  
  A:=TIntegerArray.Create(1,2,3);  
  Writeln(Length(A));  
end.
```

Imajte na umu da ovo neće raditi za dinamičke nizove za koje nije kreiran tip. To jest, sledeće neće raditi:

```pascal
var  
  A : Array of Integer;  
 
begin  
  A:=Array of Integer.Create(1,2,3);  
  Writeln(Length(A));  
end.
```

Ovaj pristup takođe funkcioniše rekurzivno za višedimenzionalne nizove:

```pascal
Type  
  TIntegerArray = Array of Integer;  
  TIntegerArrayArray = Array of TIntegerArray;  
 
var  
  A : TIntegerArrayArray;  
 
begin  
  A:=TIntegerArrayArray.Create(
    TIntegerArray.Create(1,2,3),  
    TIntegerArray.Create(4,5,6),  
    TIntegerArray.Create(7,8,9)
  );  
  Writeln('Length ',length(A));  
end.
```

Međutim, pošto je to konstruktor (kod se pokreće u vreme izvođenja), nije moguće koristiti ovo u sintaksi inicijalizovane promenljive. To jest, sledeće neće raditi:

```pascal
Type  
  TIntegerArray = Array of Integer;  
 
var  
  A : TIntegerArray = TIntegerArray.Create(1,2,3);  
 
begin  
  Writeln('Length ',length(A));  
end.
```

#### 3.2.2.5 Konstantni izrazi sa dinamičkim nizovima

Od verzije 3.2 kompajlera, niz se može konstruisati korišćenjem izraza niza u eksplicitnoj dodeli ili u inicijalizovanoj promenljivoj. Međutim, izraz je drugačiji. U iskazu dodele, on podseća na izraz skupa, u promenljivoj inicijalizacije, mora se koristiti ista sintaksa kao za konstantni niz fiksne dužine:

```pascal
Type  
  TIntegerArray = Array of Integer;  
 
var  
  A : TIntegerArray = (1,2,3);  
  B : TIntegerArray;  
 
begin  
  B:=[3,4,5];  
end.
```

#### 3.2.2.6 Pakovanje i raspakivanje niza

Nizovi se mogu pakovati i bitova. Dva tipa niza koji imaju isti tip indeksa i tip elementa, ali koji su različito upakovani nisu kompatibilni sa dodeljivanjem.

Međutim, moguće je konvertovati normalan niz u niz upakovan u bitove pomoću rutine pakovanja. Moguća je i obrnuta operacija; niz bitova se može konvertovati u normalno upakovani niz korišćenjem rutine raspakivanja, kao u sledećem primeru:

```pascal
Var  
  foo : array [ 'a'..'f' ] of Boolean  
    = ( false, false, true, false, false, false );  
  bar : bitpacked array [ 42..47 ] of Boolean;  
  baz : array [ '0'..'5' ] of Boolean;  
 
begin  
  pack(foo,'a',bar);  
  unpack(bar,baz,'0');  
end.
```

Više informacija o rutinama za pakovanje i raspakivanje možete pronaći u referenci sistemske jedinice.

### 3.3.3 Record tipovi

Free Pascal podržava fiksne zapise i zapise sa varijantnim delovima.

Dakle, sledeće su važeće deklaracije tipa `Record`:

```pascal
Type  
  Point = Record  
    X,Y,Z : Real;  
    end;  
  RPoint = Record  
    Case Boolean of  
      False : (X,Y,Z : Real);  
      True : (R,theta,phi : Real);  
    end;  
  BetterRPoint = Record  
    Case UsePolar : Boolean of  
      False : (X,Y,Z : Real);  
      True : (R,theta,phi : Real);  
    end;
```

Varijantni deo mora biti poslednji u zapisu. Opcioni identifikator u iskazu case služi za pristup vrednosti polja oznake, koja bi inače bila nevidljiva za programera. Može se koristiti da se vidi koja je varijanta aktivna u određenom trenutku. U stvari, to uvodi novo polje u zapisu.

**Napomena**  
Moguće je ugnezditi varijante delova, kao u:

```pascal
Type  
  MyRec = Record  
    X : LongInt;  
      Case byte of  
        2 : (Y : LongInt;  
      case byte of  
        3 : (Z : LongInt);  
      );  
      end;
```

#### 3.3.3.1 Izgled i veličina zapisa

Na izgled i veličinu zapisa utiče pet aspekata:

- Veličina njegovih polja.
  Zahtevi za usklađivanje tipova polja koji zavise od platforme. Imajte na umu da se zahtevi za poravnanje tipa unutar zapisa mogu razlikovati od zahteva za posebnu promenljivu tog tipa. Pored toga, lokacija polja unutar zapisa takođe može uticati na zahteve za poravnanje njegovog tipa.
- Trenutno aktivno podešavanje `{$ALIGN N}` ili `{$PACKRECORDS N}` (ova
  podešavanja zamenjuju svako drugi, tako da je poslednji navedeni aktivni; imajte na umu da ove direktive ne prihvataju potpuno iste argumente, pogledajte uputstvo za programere za više informacija).
- Trenutno aktivno podešavanje `{$CODEALIGN RECORDMIN=X}`.
- Trenutno aktivno podešavanje `{$CODEALIGN RECORDMAKS=X}`.

Izgled i veličina varijantnih delova u zapisima se određuje tako što se zamenjuju poljem čiji je tip zapis sa kao prvim elementom poljem tipa polja oznake ako je za ovo polje oznake deklarisan identifikator, a zatim slede elementi najveće varijante.

Pomak polja F2 u zapisu je jednak zbiru pomaka prethodnog polja F1 i veličine F1, zaokružen na višestruko potrebno poravnanje F2. Ovo potrebno poravnanje se izračunava na sledeći način:

- Potrebno poravnanje je podešeno na podrazumevano poravnanje tipa polja,
  Možda prilagođeno na osnovu činjenice da se ovaj tip javlja u zapisu i na lokaciji polja u zapisu.
- Ako je potrebno poravnanje manje od trenutno aktivnog `{$CODEALIGN
  RECORDMIN=Ks}` podešavanje, menja se na ovu vrednost X.
- Ako je trenutno aktivna postavka `{$ALIGN N}` ili {`$PACKRECORDS N}` numerička
  vrednost: ako potrebno poravnanje je veće od N, menja se u N. tj., ako je N 1, sva polja će biti postavljena jedno iza drugog.
- RESET ili DEFAULT: rezultirajuće potrebno poravnanje zavisi od cilja.
- C: potrebno poravnanje se podešava prema pravilima navedenim u zvanični ABI za
  trenutnu platformu.
- POWERPC, MAC68K: podešavanje vrednosti poravnanja je određeno poštujući
  zvanična ABI pravila za odn. (Klasični) Macintosh PoverPC ili Macintosh 680k0 platforme.

Veličina zapisa je jednaka zbiru pomaka poslednjeg polja zapisa i veličine ovog polja, zaokruženog na višestruko potrebno poravnanje zapisa. Potrebno poravnanje zapisa se izračunava na sledeći način:

- Zahtevano poravnanje je podešeno na poravnanje polja zapisa sa najvećim
  poravnanjem, kako je utvrđeno prilikom postavljanja zapisa.
- Ako se trenutno podešavanje `{$ALIGN N}` ili `{$PACKRECORDS N}` razlikuje od C
  i zahteva poravnanje je veće od trenutno aktivnog {$CODEALIGN RECORDMAKS=Ks}, potrebno poravnanje se menja u X.
- Ako je trenutna postavka `{$ALIGN N}` ili `{$PACKRECORDS N}` jednaka C,
  potrebno poravnanje je određeno praćenjem zvaničnih ABI pravila.

#### 3.3.3.2 Pakovanje i primeri

Free Pascal takođe podržava "upakovani zapis", koji je zapis u kome su svi elementi poravnati po bajtovima. Kao rezultat, dve sledeće deklaracije su ekvivalentne:

```pascal
{$PackRecords 1}  
Trec2 = Record  
  A : Byte;  
  B : Word;  
  end;  
{$PackRecords default}
```

and

```pascal
Trec2 = Packed Record  
  A : Byte;  
  B : Word;  
  end;
```

Obratite pažnju na `{$PackRecords Default}` posle prve deklaracije da biste vratili podrazumevanu postavku!

S obzirom na platformu zavisnu prirodu načina na koji se zapisi postavljaju u memoriju, jedini način da se obezbedi kompatibilan raspored na različitim platformama (pod pretpostavkom da su sva polja deklarisana korišćenjem tipa sa istim značenjem na istim platformama) je korišćenje
`{$PACKRECORDS 1}`.

Konkretno, ako se mora pročitati tipizirana datoteka sa zapisima, proizvedena od strane Turbo Pascal programa, onda su šanse da pokušaj ispravnog čitanja te datoteke neće uspeti. Razlog je taj što podrazumevana postavka `{$PACKRECORDS N}` Free Pascal-a nije nužno kompatibilna sa Turbo Pascal-om. Može se promeniti u `{$PACKRECORDS 1}` ili `{$PACKRECORDS 2}` u zavisnosti od podešavanja korišćenog u Turbo Pascal programu koji kreira datoteku (iako i dalje može da ne uspe sa `{$PACKRECORDS 2}` zbog različitih zahteva za usklađivanje tipa između 16-bitnog MSDOS-a i vaše trenutne platforme).

Ista primedba važi i za Delphi: razmena podataka je garantovano moguća samo ako i proizvođač i potrošač koriste upakovani zapis, ili ako su na istoj platformi i koriste istu postavku
`{$PACKRECORDS X}`.

### 3.3.4 Set tipovi

Free Pascal podržava tip `Set` kao u Turbo Pascal-u.

Svaki od elemenata `Set` tipa mora biti tipa `TargetType`. `TargetType` može biti bilo koji redni tip sa opsegom između 0 i 255. `Set` može da sadrži najviše 256 elemenata. Sledeće su važeće deklaracije seta:

```pascal
Type  
  Days = Set of Char;  

Days = (Mon, Tue, Wed, Thu, Fri, Sat, Sun);  
 
Var  
  WorkDays : Set of days;
```

S obzirom na ove deklaracije, sledeći zadatak je legalan:

```pascal
WorkDays := [Mon, Tue, Wed, Thu, Fri];
```

Na skupovima se može uraditi nekoliko operacija: uzimanje unija ili razlika, dodavanje ili uklanjanje elemenata, poređenja.

Način na koji kompajler čuva skupove zavisi od režima i može se kontrolisati pomoću direktive. Za više informacija pogledajte vodič za programera.

### 3.3.5 File tipovi

Tipovi datoteka su tipovi koji čuvaju niz nekog osnovnog tipa, koji može biti bilo koji tip osim drugog tipa datoteke. Može da sadrži (u principu) beskonačan broj elemenata. Tipovi datoteka se obično koriste za skladištenje podataka na disku. Međutim, ništa ne sprečava programera da napiše upravljački program koji svoje podatke čuva, na primer, u memoriji.

Ako nije dat identifikator tipa, onda je datoteka datoteka bez tipa; može se smatrati ekvivalentnim fajlu bajtova. Netipizirane datoteke zahtevaju posebne komande da bi delovale na njih (pogledajte BlockRead, BlockWrite). Sledeća deklaracija proglašava datoteku zapisa:

```pascal
Type  
  Point = Record  
    X,Y,Z : real;  
    end;  
  PointFile = File of Point;
```

Interno, datoteke su predstavljene zapisom FileRec, koji je deklarisan u unitima `Dos` ili `SysUtils`.

Poseban tip datoteke je tip tekstualne datoteke, predstavljen `TextRec` zapisom. Datoteka tipa `TextRec` koristi posebne ulazno-izlazne rutine. Podrazumevani tipovi ulaznih, izlaznih i StdErr datoteka definisani su u `system` unitu: svi su tipa `Text` i otvaraju se kodom za inicijalizaciju sistemske jedinice.

## 3.4 Pokazivači

Free Pascal podržava upotrebu pokazivača. Promenljiva tipa `pokazivača` sadrži adresu u memoriji, gde se mogu čuvati podaci druge promenljive. Tip pokazivača se može definisati na sledeći način:

Pokazivači su tipizirani, što znači da ukazuju na određenu vrstu podataka. Tip ovih podataka mora biti poznat u vreme kompajliranja.

Dereferenciranje pokazivača (označeno dodavanjem ^ iza imena promenljive) se tada ponaša kao promenljiva. Ova promenljiva ima tip deklarisan u deklaraciji pokazivača, a promenljiva se čuva na adresi na koju ukazuje promenljiva pokazivača. Razmotrite sledeći primer:

```pascal
Program pointers;

type  
  Buffer = String[255];  
  BufPtr = ^Buffer;  

Var B  : Buffer;  
    BP : BufPtr;  
    PP : Pointer;  
```

U ovom primeru, BP je pokazivač na tip Buffer; dok je B promenljiva tipa Buffer. B zauzima 256 bajtova memorije, a BP samo 4 (ili 8) bajtova memorije: dovoljno memorije za skladištenje adrese.

Izraz:

```pascal
BP^
```

je poznato kao dereferenciranje BP. Rezultat je tipa Buffer, dakle

```pascal
BP^[23]
```

označava 23. znak u nizu na koji ukazuje BP.

**Primedba**  
Free Pascal tretira pokazivače na isti način kao i C. To znači da se pokazivač na neki tip može tretirati kao niz ovog tipa.

Sa ove tačke gledišta, pokazivač tada pokazuje na nulti element ovog niza. Dakle, sledeća deklaracija pokazivača

```pascal
Var p : ^LongInt;
```

može se smatrati ekvivalentnim sledećoj deklaraciji niza:

```pascal
Var p : array[0..Infinity] of LongInt;
```

Razlika je u tome što prva deklaracija dodeljuje memoriju samo za pokazivač (ne za niz), a druga deklaracija dodeljuje memoriju za ceo niz. Ako se koristi prva, memorija se mora dodeliti ručno, koristeći funkciju `GetMem`. Referenca P^ je tada ista kao p[0]. Sledeći program ilustruje ovo možda jasnije:

```pascal
program PointerArray;  

var 
  i : LongInt;  
  p : ^LongInt;  
  pp : array[0..100] of LongInt;  

begin  
  for i := 0 to 100 do 
    pp[i] := i; { Fill array }  
  
  p := @pp[0];                     { Let p point to pp }  

  for i := 0 to 100 do  
    if p[i] <> pp[i] then  
      WriteLn ('Ohoh, problem !')  
end.
```

Free Pascal podržava aritmetiku pokazivača kao i C. To znači da, ako je P tipizirani pokazivač, uputstva:

```pascal
Inc(P);  
Dec(P);
```

će se povećati, odnosno smanjiti adresa na koju pokazivač pokazuje sa veličinom tipa P na koji je pokazivač. Na primer:

```pascal
Var P : ^LongInt;  
...  
 Inc (p);
```

povećaće P za 4, jer je 4 veličina LongInta. Ako pokazivač nije tipiziran, pretpostavlja se veličina od 1 bajta (tj. kao da je to pokazivač na bajt: ^Byte.)

Mogu se koristiti i normalni aritmetički operatori na pokazivačima, odnosno sledeće su važeće aritmetičke operacije pokazivača:

```pascal
var  
  p1,p2 : ^LongInt;  
  L : LongInt;  

begin  
  P1 := @P2;  
  P2 := @L;  
  L := P1-P2;  
  P1 := P1-4;  
  P2 := P2+4;  
end.
```

Ovde se vrednost koja se dodaje ili oduzima množi sa veličinom tipa na koji pokazuje pokazivač. U prethodnom primeru P1 će biti umanjen za 16 bajtova, a P2 će biti povećan za 16.

## 3.5 Deklaracija tipa unapred

Programi često moraju da održavaju povezanu listu zapisa. Svaki zapis tada sadrži pokazivač na sledeći zapis (a možda i na prethodni zapis). Za sigurnost tipa, najbolje je da se ovaj pokazivač definiše kao otkucani pokazivač, tako da se sledeći zapis može dodeliti na hrpi pomoću poziva Nev. Da biste to uradili, zapis treba da bude definisan ovako:

```pascal
Type  
  TListItem = Record  
    Data : Integer;  
    Next : ^TListItem;  
  end;
```

Kada pokušava da prevede ovo, kompajler će se žaliti da tip `TListItem` još nije definisan kada naiđe na sledeću deklaraciju: Ovo je tačno, pošto se definicija još uvek raščlanjuje.

Da biste mogli da imate element Next kao tipizirani pokazivač, mora se uvesti „Deklaracija tipa unapred“:

```pascal
Type  
  PListItem = ^TListItem;  
  TListItem = Record  
    Data : Integer;  
    Next : PTListItem;  
  end;
```

Kada kompajler naiđe na deklaraciju tipiziranog pokazivača gde referentni tip još nije poznat, odlaže rešavanje reference za kasnije. Definicija pokazivača je "Deklaracija tipa unapred".

Referentni tip bi trebalo da se uvede kasnije u istom bloku Type. Nijedan drugi blok ne sme da dođe između definicije tipa pokazivača i referentnog tipa. Zaista, čak se i sama reč Type možda neće ponovo pojaviti: u stvari, ona bi pokrenula novi blok tipa, uzrokujući da kompajler razreši sve deklaracije na čekanju u trenutnom bloku.

U većini slučajeva, definicija referentnog tipa će uslediti odmah nakon definicije tipa pokazivača, kao što je prikazano u gornjoj listi. Napred definisani tip se može koristiti u bilo kojoj definiciji tipa nakon njene deklaracije.

Imajte na umu da je deklaracija tipa unapred moguća samo sa tipovima pokazivača zapisa i klasa, ne i sa drugim tipovima.

## 3.6 Proceduralni tipovi

Free Pascal ima podršku za proceduralne tipove, iako se malo razlikuje od Turbo Pascal ili Delphi implementacije.

Dva sledeća primera su važeće deklaracije tipa:

```pascal
Type 
  TOneArg = Procedure (Var X : integer);  
  TNoArg = Function : Real;  

var 
  proc : TOneArg;  
  func : TNoArg;
```

Promenljivoj proceduralnog tipa možete dodeliti sledeće vrednosti:

- Nil, i za pokazivače normalnih procedura i za pokazivače metoda.
- Promenljiva referenca proceduralnog tipa, tj. drugu promenljivu istog tipa.
- Globalna adresa procedure ili funkcije, sa odgovarajućim zaglavljem funkcije ili procedure i 
  pozivom konvencija.
- Adresa metode.

S obzirom na ove deklaracije, sledeće izjave su važeće:

```pascal
Procedure printit (Var X : Integer);  
begin  
  WriteLn (x);  
end;  
...  
Proc := @printit;  
Func := @Pi;
```

Iz ovog primera, razlika sa Turbo Pascal-om je jasna: u Turbo Pascal-u nije neophodno koristiti operator adrese (@) kada se dodeljuje promenljiva proceduralnog tipa, dok je u Free Pascal-u to potrebno. U slučaju da se koriste prekidači -MDelphi ili -MTP, operator adrese može biti izostavljen.

**Primedba**
Modifikatori koji se odnose na konvencije pozivanja moraju biti isti kao i deklaracija; tj. sledeći kod bi dao grešku:

```pascal
Type 
  TOneArgCcall = Procedure (Var X : integer); cdecl;  

var 
  proc : TOneArgCcall;  

Procedure printit (Var X : Integer);  
begin  
  WriteLn (x);  
end;  

begin
  Proc := @printit;  
end.
```

Zato što je tip TOneArgCcall procedura koja koristi konvenciju pozivanja `cdecl`.

U slučaju da se doda modifikovana ugnežđena, onda se proceduralna varijabla može koristiti sa ugnežđenim procedurama. Ovo zahteva da se izvori kompajliraju u macpas ili ISO režimu, ili da se aktivira prekidač režima `NestedProcVars`:

```pascal
{$modeswitch nestedprocvars}  
program tmaclocalprocparam3;  
 
type  
  tnestedprocvar = procedure is nested;  
 
var  
  tempp: tnestedprocvar;  
 
procedure p1( pp: tnestedprocvar);  
begin  
  tempp:=pp;  
  tempp  
end;  
 
procedure p2( pp: tnestedprocvar);  
var  
  localpp: tnestedprocvar;  
begin  
  localpp:=pp;  
  p1( localpp)  
end;  
 
procedure n;  
begin  
  writeln( 'calling through n')  
end;  
 
procedure q;  
 
var qi: LongInt;  
 
  procedure r;  
  begin  
    if qi = 1 then  
      writeln( 'success for r')  
    else  
      begin  
      writeln( 'fail');  
      halt( 1)  
    end  
  end;  
 
begin  
  qi:= 1;  
  p1( @r);  
  p2( @r);  
  p1( @n);  
  p2( @n);  
end;  
 
begin  
  q;  
end.
```

U slučaju da neko želi da dodeli metode klase promenljivoj proceduralnog tipa, proceduralni tip mora biti deklarisan sa modifikatorom `of object`.

Dva sledeća primera su važeće deklaracije tipa za proceduralne varijable metoda (poznate i kao rukovaoci događajima zbog njihove upotrebe u GUI dizajnu):

```pascal
Type 
  TOneArg = Procedure (Var X : integer) of object;  
  TNoArg = Function : Real of object;  

var  
  oproc : TOneArg;  
  ofunc : TNoArg;
```

Ovim funkcijama se može dodeliti metod ispravnog potpisa. Kada se pozove, Self će pokazivati na instancu objekta koji je korišćen za dodeljivanje procedure metode.

Sledeći objektni metodi se mogu dodeliti oproc i ofunc:

```pascal
Type  
  TMyObject = Class(TObject)  
    Procedure DoX (Var X : integer);  
    Function DoY: Real;  
  end;  
 
Var  
  M : TMyObject;  
 
begin  
  oproc:=@M.DoX;  
  ofunc:=@M.DOY;  
end;
```

Kada pozivate oproc i ofunc, Self će biti jednako M.

Ovaj mehanizam se ponekad naziva `Delegiranje`.

**Napomena**  
Kada se porede dve promenljive tipa metoda, upoređuje se samo adresa metode, a ne pokazivač instance. To znači da će sledeći program štampati `True`:

```pascal
Type  
  TSomeMethod = Procedure  of object;  
 
  TMyObject = Class(TObject)  
    Procedure DoSomething;  
  end;  
 
Procedure TMyObject.DoSomething;  
 
begin  
  Writeln('In DoSomething');  
end;  
 
var  
  X,Y : TMyObject;  
  P1,P2 : TSomeMethod;  
 
begin  
  X:=TMyObject.Create;  
  Y:=TMyObject.Create;  
  P1:=@X.DoSomething;  
  P2:=@Y.DoSomething;  
  Writeln('Same method : ',P1=P2);  
end.
```

Ako se oba pokazivača moraju uporediti, mora se izvršiti preinačenje tipa na `TMetod` i oba pokazivača treba da se uporede. `TMethod` je definisan u `system` unitu na sledeći način:

```pascal
TMethod = record  
  Code : CodePointer;  
  Data : Pointer;  
end;
```

Sledeći program će stoga ispisati `False`:

```pascal
Type  
  TSomeMethod = Procedure  of object;  
 
  TMyObject = Class(TObject)  
    Procedure DoSomething;  
  end;  
 
Procedure TMyObject.DoSomething;  
 
begin  
  Writeln('In DoSomething');  
end;  
 
var  
  X,Y : TMyObject;  
  P1,P2 : TMethod;  
 
begin  
  X:=TMyObject.Create;  
  Y:=TMyObject.Create;  
  P1:=TMethod(@X.DoSomething);  
  P2:=TMethod(@Y.DoSomething);  
  Writeln('Same method : ',(P1.Data=P2.Data) and (P1.Code=P1.Code));  
end. 
```

## 3.7 Variantni tipovi

### 3.7.1 Definicija

Od verzije 1.1, FPC ima podršku za varijante. Za maksimalnu podršku za varijante preporučuje se dodavanje unita `Variants` klauzuli `uses` svakog unita koja na neki način koristi varijante: unit `Variants` sadrži podršku za ispitivanje i transformaciju varijanti koja nije podrazumevana podrška koju nude jedinice `System` ili `ObjPas`.

Tip vrednosti uskladištene u varijanti se određuje samo tokom izvršavanja: zavisi šta je dodeljeno varijanti. Gotovo svaki jednostavan tip može se dodeliti varijantama: redni tipovi, tipovi stringova, tipovi int64.

Strukturirani tipovi kao što su setovi, zapisi, nizovi, datoteke, objekti i klase nisu kompatibilni sa varijantom, kao ni pokazivači. Interfejsi i COM ili CORBA objekti se mogu dodeliti varijanti (u suštini zato što su jednostavno pokazivač).

To znači da su sledeći zadaci važeći:

```pascal
Type  
  TMyEnum = (One,Two,Three);  
 
Var  
  V : Variant;  
  I : Integer;  
  B : Byte;  
  W : Word;  
  Q : Int64;  
  E : Extended;  
  D : Double;  
  En : TMyEnum;  
  AS : AnsiString;  
  WS : WideString;  
 
begin  
  V:=I;  
  V:=B;  
  V:=W;  
  V:=Q;  
  V:=E;  
  V:=En;  
  V:=D;  
  V:=AS;  
  V:=WS;  
end;
```

I naravno i obrnuto.

Varijanta može da sadrži niz vrednosti: Svi elementi u nizu imaju isti tip (ali mogu biti tipa "varijanta"). Za varijantu koja sadrži niz, varijanta se može indeksirati:

```pascal
Program testv;  
 
uses variants;  
 
Var  
  A : Variant;  
  I : integer;  
 
begin  
  A:=VarArrayCreate([1,10],varInteger);  
  For I:=1 to 10 do  
    A[I]:=I;  
end.
```

Za objašnjenje `VarArrayCreate` pogledajte Referenca unita.

Imajte na umu da kada niz sadrži string, to se ne smatra „nizom znakova“, pa se varijanta ne može indeksirati za preuzimanje karaktera na određenoj poziciji u stringu.

Kao što se može videti iz gornje definicije, većina jednostavnih tipova može se dodeliti varijanti. Isto tako, varijanta se može dodeliti jednostavnom tipu: ako je moguće, vrednost varijante će biti konvertovana u tip kojem se dodeljuje. Ovo može da ne uspe: Dodeljivanje varijante koja sadrži string celom broju neće uspeti osim ako string ne predstavlja važeći ceo broj. U sledećem primeru, prvi zadatak će raditi, drugi neće uspeti:

```pascal
program testv3;  
 
uses Variants;  
 
Var  
  V : Variant;  
  I : Integer;  
 
begin  
  V:='100';  
  I:=V;  
  Writeln('I : ',I);  
  V:='Something else';  
  I:=V;  
  Writeln('I : ',I);  
end.
```

Prvi zadatak će raditi, ali drugi neće, jer se nešto drugo ne može konvertovati u ispravnu celobrojnu vrednost. Rezultat će biti izuzetak EConvertError.

Rezultat izraza koji uključuje varijantu ponovo će biti tipa varijanta, ali ovo može biti dodeljeno promenljivoj drugog tipa – ako se rezultat može konvertovati u promenljivu ovog tipa.

Imajte na umu da je za procenu izraza koji uključuju varijante potrebno više vremena i da ih stoga treba koristiti oprezno. Ako je potrebno mnogo proračuna, najbolje je izbegavati upotrebu varijanti.

Kada se razmatraju implicitne konverzije tipa (npr. bajt u ceo broj, ceo broj u dupli, char u string) kompajler će ignorisati varijante osim ako se varijanta eksplicitno ne pojavljuje u izrazu.

**Napomena**  
Podrška za varijante interfejsa za isporuku je trenutno prekinuta u kompajleru.

Varijante mogu sadržati referencu na interfejs – normalan interfejs (koji silazi od IIinterface) ili dispečerinterfejs (od IDispatch). Varijante koje sadrže referencu na interfejs za otpremu mogu se koristiti za kontrolu objekta iza njega: kompajler će koristiti kasno vezivanje da izvrši poziv interfejsu za otpremu: neće biti provere imena funkcija i parametara ili argumenata datih funkcijama tokom izvršavanja. Tip rezultata se takođe ne proverava. Kompajler će jednostavno umetnuti kod da uputi poziv za otpremu i povrati rezultat.

To u osnovi znači da na Windows-u možete da uradite sledeće:

```pascal
Var  
  W : Variant;  
  V : String;  
 
begin  
  W:=CreateOleObject('Word.Application');  
  V:=W.Application.Version;  
  Writeln('Installed version of MS Word is : ',V);  
end;
```

Linija

```pascal
  V:=W.Application.Version;
```

se izvršava umetanjem potrebnog koda za ispitivanje interfejsa za otpremu uskladištenog u varijanti V i izvršavanje poziva ako se pronađu potrebne informacije o otpremi.

## 3.8 Tip aliasa

Pseudonimi tipova su način da se tipu da drugo ime, ali se takođe mogu koristiti za kreiranje pravih novih tipova. Šta od ova dva zavisi od načina na koji je pseudonim tipa definisan:

Prvi slučaj je samo način da se tipu da drugo ime:

```pascal
Type  
  MyInteger = Integer;
```

Ovo stvara novo ime koje se odnosi na tip Integer, ali ne stvara stvarni novi tip. To jest, dve varijable:

```pascal
Var  
  A : MyInteger;  
  B : Integer;
```

Imaće isti tip sa tačke gledišta kompajlera (naime: Integer).

Gore navedeno predstavlja način da se tipovi učini nezavisnim od platforme, koristeći samo pseudonime, a zatim definišu ove tipove za svaku platformu pojedinačno. Svaki programer koji zatim koristi ove prilagođene tipove ne mora da brine o veličini osnovnog tipa: za njega je neprozirna. Takođe omogućava korišćenje imena prečica za potpuno kvalifikovana imena tipova. tj. definiše `system.LongInt` kao `OLongInt` i zatim redefiniše `LongInt`.

Pseudonim se često vidi da ponovo izlaže tip:

```pascal
Unit A;  
 
Interface  
 
Uses B;  
 
Type  
  MyType = B.MyType;
```

Ova konstrukcija se često vidi nakon nekog refaktorisanja, kada se neke deklaracije premeštaju iz jedinice A u jedinicu B, da bi se sačuvala kompatibilnost interfejsa jedinice A.

Drugi slučaj je malo suptilniji:

```pascal
Type  
  MyInteger = Type Integer;
```

Ovo ne samo da stvara novo ime koje se odnosi na tip Integer, već zapravo stvara novi tip. To jest, dve varijable:

```pascal
Var  
  A : MyInteger;  
  B : Integer;
```

Neće imati isti tip sa tačke gledišta kompajlera. Međutim, ova dva tipa će biti kompatibilna sa dodeljivanjem. To znači da je zadatak:

```pascal
  A:=B;
```

radiće.

Razlika se može videti kada se ispituju informacije o tipu:

```pascal
If TypeInfo(MyInteger)<>TypeInfo(Integer) then  
  Writeln('MyInteger and Integer are different types');
```

Funkcija kompajlera TypeInfo vraća pokazivač na informacije o tipu u binarnom sistemu. Pošto su dva tipa MyInteger i Integer različiti, oni će generisati različite tipove informacionih blokova, a pokazivači će se razlikovati.

Postoje tri posledice postojanja različitih tipova:

- Da imaju različite informacije o tipu, dakle različite RTTI (Run-Time Type
  Informations).
- Mogu se koristiti kod preopterećenja funkcija, tj

  ```pascaode
  Procedure MyProc(A : MyInteger); overload;  
  Procedure MyProc(A : Integer); overload;
  ```

  radiće. Ovo neće raditi sa pseudonimom jednostavnog tipa.

- Mogu se koristiti kod preopterećenja operatera, tj
  
  ```pascaode
  Operator +(A,B : MyInteger) : MyInteger;
  ```

  takođe će raditi.

## 3.9 Upravljani tipovi

Podrazumevano, pascal tipovima se ne upravlja. To znači da promenljive moraju biti eksplicitno inicijalizovane, finalizovane, dodeljena memorija i tako dalje. Međutim, u Object Pascal-u se upravlja nekoliko tipova, što znači da kompajler inicijalizuje i finalizuje promenljive ovog tipa: ovo je neophodno, na primer za tipove podataka koji se računaju na reference.

Sledećim tipovima se upravlja:

- **AnsiString**
  Oni su inicijalizovani na nula.

- **UnicodeString**
  Oni su inicijalizovani na nula.

- **VideString**
  Oni su inicijalizovani na nula.

- **Interfejs**
  Oni su inicijalizovani na nula.

- **Dinamički nizovi**
  Oni su inicijalizovani na nula.

I bilo koji zapis ili niz čiji elementi sadrže upravljane tipove.

Instance klase koje sadrže upravljane tipove se takođe inicijalizuju, ali sam pokazivač instance klase nije.

Promenljive upravljanih tipova će takođe biti finalizovane: to uopšteno znači da će njihov broj referenci biti smanjen najkasnije na kraju trenutnog opsega.

**Napomena**:
Imajte na umu da ne treba praviti pretpostavke o tačnom vremenu ove finalizacije. Sve to je zagarantovano da su finalizovani kada izađu iz okvira.

[prev][f2] [content][f0] [next][f4]

[f0]: 00_sadrzaj.md
[f2]: 02_konstante.md
[f4]: 04_promenljive.md
