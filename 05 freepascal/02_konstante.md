
# 2 Konstante

[prev][f1] [content][f0] [next][f3]

## 2.1 Obične konstante

Deklaracije običnih konstanti se konstruiše korišćenjem imena identifikatora praćenog tokenom "=", praćenog opcionim izrazom koji se sastoji od legalnih kombinacija brojeva, znakova, logičkih vrednosti ili nabrojanih vrednosti prema potrebi.

Kompajler mora biti u stanju da proceni izraz u konstantnoj deklaraciji u vreme kompajliranja. To znači da većina funkcija u Run-Time biblioteci ne može da se koristi u konstantnoj deklaraciji. Međutim, mogu se koristiti operatori kao što su `+`, `-`, `*`, `/`, `not`, `and`, `or`, `div`, `mod`, `ord`, `chr`, `sizeof`, `pi`, `int`, `trunc`, `round`, `frac`, `odd`.

Kada se prethodno deklarisana obična konstanta koristi u kodu, kompajler će umetnuti stvarnu vrednost konstante umesto imena konstante. To jest, sledeća dva dela koda su potpuno ekvivalentna:

```pascal
Const  
  One = 1;  

begin  
  Writeln(One);  
end.
```

Gornji kod će proizvesti isti izvršni kod kao da je bio napisan ovako:

```pascal
begin  
  Writeln(1);  
end.
```

Samo konstante sledećih tipova mogu biti deklarisane:

- **Ordinal**
- **Set**
- **Pointer** (ali jedina dozvoljena vrednost je Nil).
- **Real**
- **Char**
- **String**.

Slede validne deklaracije konstanti:

```pascal
Const  
  e = 2.7182818;                    { Real type constant. }  
  a = 2;                            { Ordinal (Integer) type constant. }  
  c = '4';                          { Character type constant. }  
  s = 'This is a constant string';  {String type constant.}  
  sc = chr(32)                       
  ls = SizeOf(Longint);  
  P = Nil;  
  Ss = [1, 2];
```

Dodeljivanje vrednosti običnoj konstanti nije dozvoljeno. Dakle, s obzirom na prethodnu deklaraciju, sledeće će dovesti do greške kompajlera:

```pascal
  s := 'some other string';
```

Za konstante niza, tip stringa zavisi od nekih prekidača kompajlera. Ako se želi određeni tip, treba koristiti tipiziranu konstantu, kao što je objašnjeno u sledećem odeljku.

Pre verzije 1.9, Free Pascal nije ispravno podržavao 64-bitne konstante. Od verzije 1.9, mogu se specificirati 64-bitne konstante.

## 2.2 Tipizirane konstante

Ponekad je potrebno specificirati tip konstante, na primer za konstante složenih struktura (definisane kasnije u priručniku). Njihova definicija je prilično jednostavna.

Za razliku od običnih konstanti, vrednost im se može dodeliti u toku rada. Ovo je stari koncept iz Turbo Paskala, koji je zamenjen podrškom za inicijalizovane varijable.

Podršku za dodeljivanje vrednosti tipiziranim konstantama kontroliše `{$J}` direktiva: može se isključiti, ali je podrazumevano uključena (za Turbo Pascal kompatibilnost). Inicijalizovane promenljive su uvek dozvoljene.

**Napomena**:
Treba naglasiti da se tipizirane konstante automatski inicijalizuju pri pokretanju programa. Ovo takođe važi za lokalno tipizirane konstante i inicijalizovane promenljive. Lokalne tipizirane konstante se takođe inicijalizuju pri pokretanju programa. Ako je njihova vrednost promenjena tokom prethodnih poziva funkcije, oni će zadržati promenjenu vrednost, tj. ne inicijalizuju se svaki put kada se funkcija pozove.

## Resourcestring

Posebna vrsta bloka deklaracije konstante je blok `Resourcestring`. Deklaracije stringova resursa su slične deklaracijama konstantnih stringova: stringovi resursa deluju kao konstantni stringovi, ali se mogu lokalizovati pomoću skupa posebnih rutina u jedinici `objpas`. Blok deklaracije niza resursa je dozvoljen samo u Delphi ili Objfpc režimima.

Sledi primer definicije `resourcestring`:

```pascal
Resourcestring  

  FileMenu = '&File...';  
  EditMenu = '&Edit...';
```

Sve string konstante definisane u odeljku `resourcestring` se čuvaju u posebnim tabelama. Stringovima u ovim tabelama može se manipulisati tokom rada pomoću nekih posebnih mehanizama iz unita `objpas`.

Semantički, stringovi se ponašaju kao obične konstante; Nije dozvoljeno da im se dodeljuju vrednosti (osim preko posebnih mehanizama u jedinici `objpas`). Međutim, oni se mogu koristiti u zadacima ili izrazima kao obične string konstante. Glavna upotreba odeljka `resourcestring` je da obezbedi sredstvo za laku internacionalizaciju.

Više o temi `resourcestring` resursa možete pronaći u Vodiču za programere i u referenci unita `objpas`.

**Napomena**  
Imajte na umu da se `resourcestring` koji je dat kao izraz neće promeniti ako se delovi izraza promene:

```pascal
resourcestring  
  Part1 = 'First part of a long string.';  
  Part2 = 'Second part of a long string.';  
  Sentence = Part1+' '+Part2;
```

Ako rutine lokalizacije prevode "Part1" i "Part2", konstanta "Sentence" neće biti automatski prevedena: ona ima poseban unos u tabelama resourcestring i stoga mora biti prevedena odvojeno. Gornja konstrukcija jednostavno kaže da je početna vrednost rečenice jednaka "Part1+’ ’+Part2".

**Napomena**:
Slično, kada koristite resourcestring u nizu konstanti, u nizu će se koristiti samo početne vrednosti resourcestringa: kada se pojedinačne konstante prevedu, elementi u nizu će zadržati svoju originalnu vrednost.

```pascal
resourcestring  
  Yes = 'Yes.';  
  No = 'No.';  

Var  
  YesNo : Array[Boolean] of string = (No,Yes);  
  B : Boolean;  

begin  
  Writeln(YesNo[B]);  
end.
```

Ovo će odštampati „Da“. ili „Ne“. u zavisnosti od vrednosti B, čak i ako su konstante Da i Ne lokalizovane nekim mehanizmom lokalizacije.

[prev][f1] [content][f0] [next][f3]

[f0]: 00_sadrzaj.md
[f1]: 01_tokeni.md
[f3]: 03_tipovi.md
