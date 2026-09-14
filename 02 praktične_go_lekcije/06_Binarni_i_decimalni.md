
# 06 Binarni i decimalni brojevi

[05 Prva Go aplikacija][05] | [00 Sadržaj][00] | [07 Hexadecimal, octal,ASCII, UTF8, Unicode, rune][07]

**Šta ćete naučiti u ovom poglavlju?**

- Razlika između decimalnog i binarnog sistema brojanja.
- Kako računari čuvaju podatke.
- Koja je razlika između 32-bitnih i 64-bitnih sistema?

**Obrađeni tehnički koncepti!**

- Binarni
- Bitovi, bajtovi
- Decimalni
- Kodiranje, enkoder
- RAM memorija

## Uvod: Numerici, brojevi i količine

Hajde da definišemo neke važne reči:

- Numeric: simbol ili grupa znakova koji predstavljaju broj
- Broj predstavlja količinu, nešto što možemo izbrojati.
- Numeric je reprezentacija cifara.
- Numerički sistem je skup pravila koji nam omogućava da brojimo stvari i da predstavljamo količine.

Svakog dana manipulišemo brojevima predstavljenim decimalnim sistemom: 10,254, 125, 2020, 31.…

Ali decimalni sistem nije jedini i nije se uvek koristio.

Brojeve možemo predstaviti koristeći i druge brojevne sisteme.

Uzmimo za primer količinu "sto dvadeset tri"

- "123" predstavlja ovu količinu u decimalnom sistemu
  - Broj je "123"
- "1111011" predstavlja ovu veličinu u binarnom sistemu
  - Broj je "1111011"
- "7B" predstavlja ovu količinu u heksadecimalnom sistemu
  - Broj je "7B"
- "173" predstavlja ovu količinu u oktalnom sistemu
  - Broj je "173"

Ista količina, ista osnovna vrednost može se izraziti u različitim oblicima.

Zašto je važno to razumeti? Zato što računari neće čuvati podatke koristeći decimalni sistem. Na fizičkom nivou, informacije se čuvaju pomoću nula i jedinica.

Ovo poglavlje će detaljno objasniti kako funkcionišu decimalni i binarni sistem.

## Etimologija i simboli

Reč decimalni broj potiče od latinske reči "Decimus" što znači "deseti". Dok reč binarni potiče od latinske reči "bini" što znači "dva zajedno". Etimologija ove dve reči nam daje naznaku o tome kako su ti sistemi konstruisani:

- Binarni sistem koristi dva simbola, a to su 0 i 1
- Decimalni sistem koristi deset simbola, a to su 0, 1, 2, 3, 4, 5, 6, 7, 8, 9.

Podaci zapisani pomoću binarnog sistema biće zapisani pomoću "0" i "1". Na primer, 101010 se zapisuje pomoću binarnog sistema kodiranja.

Podaci zapisani korišćenjem decimalnog sistema biće zapisani korišćenjem 0, 1, 2, 3, 4, 5, 6, 7, 8, 9. Na primer, 42 se piše korišćenjem decimalnog sistema. Broj 10 može biti kodiran korišćenjem binarnog ili decimalnog sistema (i stoga neće predstavljati istu osnovnu veličinu).

## Decimalni sistem

Hajde da pogledamo broj napisan pomoću decimalnog sistema:  

```sh
123
```

Ovaj broj je sto dvadeset tri. Ovaj broj je sastavljen od "cifara".

- Prva cifra predstavlja broj stotina.
- Druga cifra je broj desetica.
- Poslednja cifra je broj jedinica.

Decimalni sistem je pozicioni. To znači da doprinos cifre broju zavisi od pozicije cifre u broju.

Hajde da napravimo korak dalje. Možemo napisati broj sto sa brojem deset:  

```sh
100=10×10
```

Možemo koristiti stepene desetice:

```sh
10×10=10^2
```

10^2 je ekvivalent1 10×10. Kažemo da je 10 baza i da je 2 stepen. Kada ga čitamo, kažemo deset na stepen broja 2. Prateći istu logiku, možemo napisati:

```sh
10 = 10^1
```

I:

```sh
1=10^0
```

Poslednje može delovati čudno. To je matematičko pravilo: svaki broj različit od nule podignut na 0 jednak je 1.

Imajući to u vidu, možemo razložiti broj 123 sa stepenom od deset:

```sh
123=1×10^2+2×10^1+3×10^0
```

Ako čitate 1×10^2+2×10^1+3×10^0 s leva na desno, možete primetiti da se stepen broja 10 smanjuje jedan po jedan.

Ovaj stepen često odgovara položaju cifre kada se broj zapiše.

- Cifra "1" je na poziciji 2 (1×10^2)
- Cifra "2" je na poziciji 1 (2×10^1)
- Cifra "3" je na poziciji 0 (3×10^0)

Način na koji sam brojao pozicije bi mogao da vas iznenadi. Očekivali ste možda nešto slično:

- cifra "1" je na poziciji 3
- cifra "2" je na poziciji 2
- Cifra "3" je na poziciji 1

Ovo je potpuno tačno ako počinjete brojanje sa 1, ali mi počinjemo brojanje sa nulom. Zapamtite ovu konvenciju jer će vam kasnije pomoći!

![0501][0601]  
Veza izmedju pozicije decimalnih cifara i stepena 10.

U zaključku, veza između cifara i stepena broja je sledeća:

Za dati broj:

```sh
cifra2 ​cifra1 ​cifra0​
```

Odgovarajući broj (količina) je:

```sh
cifra2​×10^2 + cifra1​×10^1 + cifra0​×10^0
```

Naravno, ova relacija važi za brojeve sa više (i manje) od tri cifre!

### Razlomci brojeva

Videli smo kako sistem funkcioniše za okrugle količine; šta je sa razlomljenim brojevima (brojevima sa "decimalnim separatorom", kao što je ​123,14)

I dalje postoji veza sa stepenom desetica.

123,45 = 1×10^2 + 2×10^1 + 3×10^0 + 4×1/10^1 ​+ 5×1/10^2​ ili  
123,45 = 1×10^2 + 2×10^1 + 3×10^0 + 4×10^(-1) ​+ 5×10^(-2)​

Ovo ima smisla jer:1/10^1 = 1/10 = 0.1

- Dakle, 4×1/10^1 = 0,4

1/10^2 = 1/100 = 0.01

- Dakle, 5×1/10^2 = 0.05

## Binarni sistem

Broj zapisan u binarnom obliku sastoji se od nula i jedinica. Binarni sistem, kao i decimalni sistem, je pozicioni brojevni sistem. To znači da svaka cifra ima vrednost koja zavisi od njenog položaja. To je sistem sa osnovom dva (decimalni sistem je osnova 10).

Hajde da pogledamo binarni broj:

```sh
10b
```

Imajte na umu da sam dodao indeks "b" broju jer ovaj broj postoji i u decimalnom sistemu. Decimalni broj 10 NIJE jednak binarnom broju 10. Ne izražava istu količinu. 10b ​je broj sastavljen od dve binarne cifre. Termin binarna cifra ima široko korišćenu skraćenicu: `bit`. Decimalni ekvivalent ovog broja možemo dobiti korišćenjem stepena broja dva:

```sh
10b = (1×2^1 + 0×2^0)d
10b = (1×2)d
10b = 2d
```

Binarni broj 10b ​predstavlja istu količinu kao 2d. ​​​

Uzmimo još jedan primer.

```sh
100010b
```

Naći ćemo njegov decimalni prikaz. Uzimamo svaku cifru; množimo je sa 2^x gde x je pozicija cifre u binarnom broju.

![0602][0602]
Konverzija binarnog u decimalni broj

Inače, postoji poznata šala koju sam jednom čuo od kolege o ovome: "Postoji deset vrsta ljudi na svetu, oni koji razumeju binarni sistem i oni drugi" :)

## Kapacitet skladištenja

### Kapacitet memorije od 2 bita

Koji je maksimalni decimalni broj koji možemo da sačuvamo u dvocifrenom binarnom broju? Evo liste binarnih brojeva koji su sastavljeni od 2 cifre:

```sh
00b = (0×2^1 + 0×2^0)d = 0d
01b = (0×2^1 + 1×2^0)d = 1d
10b = (1×2ˇ1 + 0×2ˇ0)d = 2d
11b = (1×2^1 + 1×2^0)d = 3d
```

Sa dve binarne cifre možemo da sačuvamo brojeve od 0d ​​​do3d. ​​​Maksimalan broj koji se može sačuvati je 3d.

### Kapacitet memorije od 3 bita

Koji je maksimalni decimalni broj koji možemo da sačuvamo u trocifrenom binarnom broju? Evo liste binarnih brojeva koji su sastavljeni od 3 cifre:

```sh
000b = (0×2^2 + 0×2^1 + 0×2^0)d = 0d
001b = (0×2^2 + 0×2^1 + 1×2^0)d = 1d
010b = (0×2^2 + 1×2^1 + 0×2^0)d = 2d
011b = (0×2^2 + 1×2^1 + 1×2^0)d = 3d
100b = (1×2^2 + 0×2^1 + 0×2^0)d = 4d
101b = (1×2^2 + 0×2^1 + 1×2^0)d = 5d
110b = (1×2^2 + 1×2^1 + 0×2^0)d = 6d
111b = (1×2^2 + 1×2^1 + 1×2^0)d = 7d
```

​​​Možemo sačuvati sve brojeve između 0d i 7d. Maksimalni decimalni broj koji se može sačuvati sa 3 bita je 7d​​​.

### Kapacitet memorije od 8 bita (jedan bajt)

Koji je maksimalni decimalni broj koji možemo da sačuvamo u binarnom broju od osam cifara? Mogli bismo da navedemo sve različite binarne brojeve koji se mogu napraviti, ali bi to oduzimalo mnogo vremena.

Da li ste primetili u prethodnim odeljcima da se maksimalni broj pravi samo od jedinica? Čini se logičnim kada je binarni broj sastavljen od samo jedinica; njegova decimalna vrednost biće jednaka zbiru stepena dvojke (od 0 do n) gde je n broj cifara minus 1. Ako u binarnom broju postoji jedna nula, stepen odgovarajuće dvojke se ne računa.

```sh
11111111b = (1×2^7 + 1×2^6 + 1×2^5 + 1×2^4 + 1×2^3 + 1×2^2 + 1×2^1+1×2^0)d = 255d
```

Sa 8 bitova, možemo da sačuvamo sve brojeve između 0d i 255d. ​​​8 `bit`-a se naziva `byte` (bajt). Od 0d ​​​do 255d ​​​postoji 256 brojeva.

### Kako čuvati slike, video zapise,...?

Detaljno smo opisali kako transformisati decimalni broj u binarni broj da bismo ga sačuvali u memoriji. Nadam se da je bilo jasno i zanimljivo. Ali to nije dovoljno. Programi manipulišu sa mnogo više stvari nego samo brojevima.

Možemo da napravimo programe koji koriste:

- Slike
- Tekstove
- Filmove
- 3D modeli, itd

Kako se ti objekti čuvaju u memoriji?

Odgovor je jednostavan: na kraju će čak i fotografije i filmovi biti sačuvani korišćenjem nula i jedinica! Konvertovaćemo ih u binarne datoteke. Ovaj posao će obaviti specijalizovani programi koji se nazivaju enkoderi. Enkoder će kao ulaz uzeti datoteku određenog formata i konvertovaće je u odredišni format. U našem slučaju, odredište je binarno.

Ne moramo da pišemo te programe; sve ih pruža Go. Morate da shvatite da se svaka datoteka ili blok podataka čuva "ispod haube" koristeći binarne cifare (bitove).

Binarna reprezentacija je jednostavno skrivena od nas.

## 32-bitni sistemi u odnosu na 64-bitne sisteme

### Programima je potrebna memorija

Procesor računara je odgovoran za izvršavanje programa. Većina programa treba da čuva i pristupa memoriji. Ako napišete program koji će na ekranu prikazati "42". Iza scene, broj 42 mora biti negde sačuvan. Sistem će takođe morati da ga preuzme iz memorije.

Ovo se radi putem sistema adresiranja. Svaki bit informacije se čuva na preciznoj lokaciji u memorijskoj jedinici i da bi ih dobio, procesor mora imati mogućnost da dobije njihovu punu adresu.

Imajte na umu da postoje dve vrste memorije:

- Centralna memorija: ROM i RAM
- Pomoćna memorija: hard disk, USB ključevi...

Ovde razmatramo samo centralnu memoriju.

### Memorijska adresa je broj

Za procesor, adresa je broj. Memorijska adresa je kao poštanska adresa. Ona precizno identifikuje lokaciju u memorijskom prostoru.

Procesori će čuvati adrese u registrima. Registar je mesto unutar procesora gde se adresa može sačuvati za kasniju upotrebu. Na primer, recimo da imamo program koji vrši sabiranje. Naš program definiše promenljivu koja će čuvati prvu vrednost (recimo 1,234) i drugu koja će čuvati drugu vrednost (recimo 1,290,999). Kada procesor izvrši naš program, moraće da preuzme vrednost 1,234 iz memorije i vrednost 1,290,999. Procesor će čuvati ove dve adrese u svojim registrima.

### Adresabilna memorija - ograničen resurs

Registri imaju ograničen kapacitet; mogu da čuvaju adrese određene veličine (zapisane u bitovima). Za 16-bitni procesor, maksimalni kapacitet tih registara je 16 bita. Za 32-bitne procesore, maksimalni kapacitet je 32 bita. Isto rezonovanje važi i za 64 bita.

Maksimalni kapacitet registra će definisati koliko memorije možemo da adresiramo.

Zašto? Zapamtite da adrese čuvamo na tim registrima, a adresa je broj:

- Na 32 bita možemo da sačuvamo brojeve od 0 do 4.294.967.295 ​​≈ 2^32.
- Sa 32 bita, možemo da sačuvamo 4 milijarde neoznačenih brojeva, 4 milijarde adresa.

- Na 64 bita možemo da sačuvamo brojeve od 0 do 18.446.744.073.709.551.615 ≈ 2^64. To omogućava mnogo adresa.

```go
32 bita = 4 milijarde adresa
64 bita = 18.446.744.073 milijardi adresa (mnogo adresa :))
```

### Odnos između broja mogućih adresa i veličine RAM memorije

RAM memorija je hardverska komponenta sastavljena od memorijskih ćelija. Bitovi se čuvaju u ćelijama. Generalno, za RAM memoriju se kaže da je bajtno adresabilna. To znači da sistem može da preuzima podatke 8 bitova istovremeno.

Videli smo da je veličina memorijske adrese ograničena veličinom registara.

32-bitni sistem može da obrađuje samo adrese koje su sastavljene od 32 bita. Svaki bit je sastavljen od 0 ili 1, što čini 2^32 ≈ 4 milijardu mogućnosti.

- Sa 4 milijarde adresa, koliko bajtova možemo da adresiramo?
  - => 4 milijarde bajtova
  - Ovu količinu možemo pretvoriti u gigabajte.
    - 1 gigabajt = 1 milijarda bajtova
    - Dakle, 4 milijarde bajtova = 4 gigabajta = 4GB!

Sa 32-bitnim sistemima, sistem može da pristupi 4 gigabajta memorije. Instaliranje više od 4 GB RAM-a na 32-bitni sistem je beskorisno.

Na 64-bitnim sistemima, broj adresa je mnogo veći; stoga je i količina adresabilne memorije veća: teoretski 16 exabajta (1 exabajt = ​​​1.073.741.824 GB).

Kao posledica toga, 16 exabajta je teoretski maksimalna količina RAM-a koja se može adresirati na 64-bitnom računaru.

## Testirajte sebe

### Pitanja i odgovori

1. Šta je bit?
   - Ovo je skraćenica od binarna cifra. Ili je 0 ili je 1.
2. Šta je bajt?
   - Bajt ima 8 bitova.
3. Šta je enkoder?
   - Koder prima ulaz u datom formatu i transformiše ga u drugi oblik.
     Proces je reverzibilan.
4. Koja je maksimalna količina RAM memorije koju 32-bitni sistem može
   adresirati?
   - 4 GB
5. Koja je decimalna verzija (001)b?
   - 1

### Ključno

- Numerici predstavljaju brojeve
- Svaki sistem numeracije ima svoj način predstavljanja brojeva.
  100 može biti broj zapisan u binarnom i decimalnom sistemu. Ne predstavlja isti broj/količinu!
- Na fizičkom nivou, podaci se čuvaju u memoriji pomoću binarnih cifara (0 i 1)
- Bit je jedna binarna cifra
- Bajt ima osam bitova
- 64-bitni sistem može da adresira daleko više memorije nego 32-bitni sistem.

[05 Prva Go aplikacija][05] | [00 Sadržaj][00] | [07 Hexadecimal, octal,ASCII, UTF8, Unicode, rune][07]

[0601]: ./_images/0601.png
[0602]: ./_images/0602.png

[05]: 05_Prva_aplikacija.md
[00]: 00_Sadržaj.md
[07]: 07_Hexadecimal_octal_ASCII_UTF8_Unicode_rune.md
