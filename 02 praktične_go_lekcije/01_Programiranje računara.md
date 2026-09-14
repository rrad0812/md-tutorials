
# 01 Programiranje računara

[00 Sadržaj][00] | [02 Jezik Go][02]

**Šta ćete naučiti u ovom poglavlju?**

- Navešćemo različite hardverske delove računara.
- Videćemo šta je program i kako izgleda.
- Razumećemo kako se program učitava i izvršava.

**Obrađeni tehnički koncepti!**

- Memorijska jedinica, aritmetička i logička jedinica, ulaz/izlaz, kontrolna jedinica
- Centralna memorija, pomoćna memorija
- Isparljiva i neisparljiva memorija
- RAM/ROM
- Procesor
- Jezici visokog i niskog nivoa
- Asemblerski jezik, asembler
- Kompiliran i interpretiran jezik

## Uvod

Ova knjiga je o Gou. Pre nego što pređemo na našu glavnu temu, potrebno je da znate neka osnovna znanja o računarima.

Iskusni programeri mogu preskočiti ovo poglavlje. Početnici bi trebalo da odvoje malo vremena da ga prouče.

Vaši programi će se pokretati na hardveru. Poznavanje načina rada vašeg hardvera može poboljšati dizajn vaših programa.

Prvo ćemo opisati glavne komponente računara. Zatim ćemo videti šta je program i kako ga mašina koristi.

## Četiri hardverske komponente

Računar se sastoji od četiri glavna dela:

- Memorijska jedinica (MU) gde se čuvaju podaci i programi. Na primer, možemo da sačuvamo u memorijskoj jedinici ocene sa fakulteta. Takođe možemo da držimo program koji će izračunati prosečnu ocenu tog razreda.

- Aritmetička i logička jedinica (ALU). Njena uloga je da izvršava aritmetičke i logičke operacije nad podacima sačuvanim u memorijskoj jedinici. Ova jedinica može da izvršava, na primer, sabiranja, inkrementacije, dekrementacije, koje se nazivaju operacije. Generalno, svaka operacija zahteva dva operanda i daje rezultat.
  
  Recimo da želimo da saberemo 5 i 4. Ta dva broja su operandi. Rezultat ove operacije je 9. Operandi se učitavaju iz memorijske jedinice. ALU je električno kolo koje je dizajnirano za izvršavanje operacija.

- Ulazna i izlazna jedinica (I/OU) biće zadužena za učitavanje podataka u memorijsku jedinicu sa ulaznog uređaja. Ova jedinica takođe šalje podatke iz memorijske jedinice na izlazni uređaj.

  Ulazni uređaj je, na primer, tačped vašeg računara, vaša tastatura, vaš miš.

  Izlazni uređaj je, na primer, vaš monitor.

- Kontrolna jedinica (CU) će primati instrukcije iz programa i kontrolisaće aktivnost ostalih jedinica.

Četiri hardverske komponente predstavljaju šematski prikaz komponenti računara.

### Memorija

Računar se sastoji od dve vrste memorije:

- Centralna memorija
- Pomoćna memorija

Postoje dve kategorije memorije:

- Volatile (promenljiva)
- Non-volatile (ne promenljiva)

#### Centralna memorija

Ova vrsta memorije je neophodna za pokretanje operativnih sistema i svih ostalih programa koje će vaš računar pokretati. Centralna memorija sadrži dve vrste skladišta:

- RAM (memorija sa slučajnim pristupom). Ova vrsta skladištenja zahteva električnu energiju za čuvanje podataka. Kada isključite računar, memorija koja se nalazi u ovoj vrsti skladištenja biće obrisana.Operativni sistem i programi koje koristite biće učitani u ovu memoriju. Ova vrsta memorije je isparljiva.

- ROM (memorija samo za čitanje). Ovo je memorija koja sadrži podatke neophodne za ispravan rad računara. Ova vrsta memorije nije isparljiva (kada isključite računar, neće biti obrisana). Dizajnirana je da bude samo čitljiva i da je sistem ne ažurira.

#### Pomoćna memorija

Ova vrsta memorije nije promenljiva. Kada nestane struje, sačuvani podaci se neće izbrisati. Evo nekih primera pomoćne memorije: čvrsti diskovi, USB ključevi, CD-ROM, DVD... itd.

Čitanje i pisanje u ovu vrstu memorije je sporo u poređenju sa RAM memorijom.

Neki čvrsti diskovi sekvencijalno pristupaju memoriji. Sistem bi trebalo da poštuje određeni redosled. Poštovanje ovog redosleda pristupa traje duže nego režim slučajnog pristupa. Imajte na umu da neki čvrsti diskovi omogućavaju slučajni pristup memoriji.

#### SSD hard disk

Hard diskovi, takođe poznati kao Hard Disk Drajv (HDD), sastoje se od magnetnih diskova koji se rotiraju. Podaci se čitaju i pišu zahvaljujući pokretnoj magnetnoj glavi. Čitanje i pisanje generišu rotaciju i kretanje magnetne glave, što troši vreme.

SSD (Solid-State Drives) diskovi nisu tako konstruisani. Nemaju magnetnu glavu niti magnetne diskove. Umesto toga, podaci se čuvaju u ćelijama fleš memorije. Pristup podacima je brži na toj vrsti diska. Imajte na umu da SSD takođe košta više od tradicionalnih elektromagnetnih hard diskova.

### Procesor

CPU je skraćenica za Central Processing Unit (centralna procesorska jedinica). CPU se takođe označava kao procesor. CPU sadrži:

- Aritmetička i logička jedinica (ALU)
- Kontrolna jedinica (CU)

Procesor (CPU) će biti odgovoran za izvršavanje instrukcija koje daje program. Na primer, program može dati instrukciju za sabiranje dva broja. Ti brojevi će biti preuzeti iz memorijske jedinice i prosleđeni ALU-u. Program može takođe zahtevati izvršavanje I/O operacija kao što je čitanje podataka sa čvrstog diska i njihovo učitavanje u RAM memoriju za dalju obradu. CPU će izvršiti te instrukcije.

Centralni procesor (CPU) je centralna komponenta računara.

### Program

Da bi računari nešto uradili, moramo im dati precizna uputstva. Ovaj skup instrukcija se naziva "program".

Prateći zvaničniju definiciju,

> [!Note]
>
>[@institute1990ieee]  
> Program je "kombinacija računarskih instrukcija i definicija podataka koje
> omogućavaju računarskom hardveru da vrši izračunavanje".

Uzmimo primer. Zamislite program koji traži od korisnika da unese dva broja. Program sabira te brojeve, a rezultat se zatim prikazuje na monitoru. Instrukcije koje treba napisati su:

- Na monitoru se prikazuje „Molimo vas da unesete svoj prvi broj i pritisnete enter“.

- Kada se unese broj i pritisne taster „Enter“ na tastaturi, sačuvajte broj u memoriji. Označimo sa AA ovaj broj.

- Na monitoru se prikazuje „Molimo vas da unesete drugi broj i pritisnete enter“.

- Kada se unese broj i pritisne taster „Enter“ na tastaturi, sačuvajte broj u memoriji. Označimo BB ovaj broj.

- Pošaljite ALU-u dva broja (AA i BB) i opkod za sabiranje i sačuvajte rezultat u memoriju.

- Prikažite rezultat na monitoru

Izvršavaju se dve vrste instrukcija:

- U/I operacije : preuzimanje brojeva sačuvanih u memoriji, čuvanje brojeva u memoriji sa ulaznog uređaja (tastature), učitavanje podataka iz memorije i njihovo prikazivanje korisniku.

- Aritmetička operacija : sabiranje dva broja.

Ovde imamo skup instrukcija koje su napisane jednostavnim engleskim jezikom. Mašina ne razume engleske rečenice. Te rečenice treba prevesti na jezik koji mašina razume. Koji je taj jezik?

### Kako razgovarati sa mašinom

#### Programski jezici su formalni jezici

Instrukcije koje se daju mašini napisane su pomoću programskih jezika. Programski jezici su formalni jezici. Sastavljeni su od reči koje su konstruisane od abecede (skup različitih znakova). Te reči su organizovane prema određenim pravilima. Go je programski jezik, kao što su x86 Assembly, Java, C, C++, Javascript...

To su dve vrste programskih jezika:

- Nizak nivo
- Visok nivo

Programski jezici niskog nivoa su bliži instrukcijama procesorske jedinice. Jezici višeg nivoa pružaju konstrukcije koje ih čine lakšim za učenje i korišćenje u svakodnevnom životu.

Neki jezici visokog nivoa se kompajliraju, drugi se interpretiraju, a neki su između. U narednim odeljcima ćemo videti šta ta dva termina znače.

![0101][0101]  
Klasifikacija programskih jezika

Da bismo razgovarali sa procesorskom jedinicom računara, možemo koristiti mašinski jezik. Mašinski jezici se sastoje isključivo od nula i jedinica. Instrukcija napisana mašinskim jezikom je skup od 0 i 1. Svaki procesor (ili familija procesora) definiše listu instrukcija koje se nazivaju skup instrukcija. Postoji instrukcija za sabiranje brojeva, povećanje za jedan, smanjenje za jedan, kopiranje podataka sa jedne lokacije u memoriji na drugu... itd.

Moguće je pisati računarske programe direktno u mašinskom jeziku. Međutim, to nije lako.

#### Asemblerski jezik

Asemblerski jezik je programski jezik niskog nivoa. Instrukcije programa napisanog na asemblerskom jeziku odgovaraju mašinskim instrukcijama. Asemblerski jezici koriste mnemonike, što su kratke reči koje odgovaraju mašinskoj instrukciji. Na primer `MOV` naložiće mašini da premesti podatke sa jedne lokacije na drugu. Programeri takođe mogu komentarisati kod (što nije moguće sa mašinskim jezikom).

Da bi kreirao program u asemblerskom jeziku, programer će napisati instrukcije u jednoj ili više datoteka. Te datoteke se nazivaju izvorne datoteke.

Evo primera instrukcije napisane u x86 asemblerskom jeziku za Linux:

```asm
// assembly code
mov    eax,1
int    0x80
```

Te dve linije će izvršiti sistemski poziv koji će zatvoriti program ("1" predstavlja broj sistemskog poziva koji znači "zlaz iz programa"). Imajte na umu da se asemblerski jezik razlikuje od mašine do mašine. Kažemo da je specifičan za mašinu.

Asembler se koristi za konvertovanje izvornih datoteka napisanih u asemblerskom jeziku u datoteke objektnog koda. Kažemo da asemblujemo program. Linker će zatim transformisati te datoteke objektnog koda u izvršnu datoteku. Izvršna datoteka sadrži sva potrebna uputstva računara za pokretanje programa.

![0102][0102]  
Od asemblerskog koda do izvršne datoteke

#### Jezici visokog nivoa

Na tržištu postoji mnoštvo programskih jezika visokog nivoa, poput Go-a. Ovi jezici nisu usko vezani za mašinsku arhitekturu. Oni nude zgodan način pisanja instrukcija. Na primer, ako želimo da napravimo sistemski poziv za izlazak iz našeg programa, možemo napisati u go:

```go
os.Exit(1)
```

Sa C jezikom, možemo napisati:

```c
// C Code
exit(1)
```

Sa Javom, možemo napisati:

```java
// Java Code
System.exit(1);
```

U ovom primeru, ne moramo da premeštamo broj u registar; koristimo jezičke konstrukcije (funkcije, pakete, metode, promenljive, tipove...).

Cilj ove knjige je da vam pruži precizne i sažete definicije tih alata za izgradnju Go aplikacija.

Programi visokog nivoa se takođe pišu u datoteke. Datoteke se nazivaju "izvorne datoteke". Generalno, programski jezici zahtevaju dodavanje određene ekstenzije nazivu datoteke. Za Go programe dodaćemo ".go" na kraj svake datoteke koju ćemo pisati. U PHP-u ekstenzija je ".php".

Kada se napišu izvorne datoteke, program koji one definišu ne može se odmah izvršiti. Izvorna datoteka mora biti kompajlirana pomoću kompajlera. Kompajler će transformisati izvorne datoteke u izvršnu datoteku. Kompajler je takođe program. Go je deo porodice kompajliranih jezika.

![0103][0103]  
Go je kompajlirani jezik

#### Kompajlirano naspram interpretiranog

Imajte na umu da se neki programski jezici interpretiraju. Kada su izvorne datoteke napisane, programer ne mora da ih kompajlira. Kada su izvorne datoteke spremne, sistem može da izvrši program zahvaljujući interpreteru. Svaka instrukcija zapisana u izvornu datoteku se prevodi i izvršava od strane interpretera. U nekim slučajevima, interpreteri čuvaju u kešu kompajliranu verziju programa kako bi poboljšali performanse (izvorne datoteke se ne prevode svaki put). PHP, Python, Ruby, Perl su interpretirani jezici.

## Testirajte sebe

### Pitanja i odgovori

1. Gde se čuvaju programi?
   - U memorijsku jedinicu (MU)
2. Čitanje podataka sa čvrstog diska je sporije od čitanja podataka iz
   RAM-a. Tačno ili netačno?
   - Tačno. Preuzimanje i pisanje podataka u RAM memoriju je izuzetno
     brzo, dok pristup podacima sačuvanim na čvrstim diskovima generalno traje duže.
3. Možete li pisati u ROM? Tačno ili netačno?
   - Netačno. Ova vrsta memorije se može samo čitati. Koristi se za
     čuvanje OS-a (operativnog sistema)
4. Koje su dve vrste memorije?
   - Centralna memorija
   - Pomoćna memorija
5. Koja je definicija "volatile memorije“?
   - Volatile memorija će biti obrisana kada se računar isključi.
6. Koji program transformiše kod napisan na asemblerskom jeziku u
   objektni kod?
   - Asembler će kao ulaz uzeti kod asemblerskog jezika i generisati
     mašinski kod.
7. Koji program transformiše objektni kod u izvršnu datoteku?
   - Linker
8. Navedite dve prednosti jezika visokog nivoa u poređenju sa jezicima
   niskog nivoa?
   - Oni nude visokokvalitetne konstrukcije koje su lakše za upotrebu.
   - Kod neće biti specifičan za tehničku arhitekturu mašine. Kažemo da
     je kod prenosiv.
9. Da li je Go interpretirani jezik? Tačno ili netačno?
   - Netačno: Go je kompajlirani jezik

### Ključno

- Na makro nivou, računar se sastoji od:
  - Memorijska jedinica (MJ): za čuvanje podataka i programa
  - Aritmetička i logička jedinica (ALU): za izvršavanje izračunavanja
  - Ulazna i izlazna jedinica (IOU): za upravljanje ulaznim i izlaznim uređajima.
  - Kontrolna jedinica (CU) će upravljati MU, ALU i IOU prateći instrukcije koje je dao izvršeni program.
- CPU znači da je centralna procesorska jedinica (takođe nazvana procesor ili mikroprocesor) sastavljena od ALU i CU.
- Program je skup instrukcija.
- Programeri pišu programe pomoću programskog jezika.
- Programski jezici se sastoje od reči i znakova koji moraju biti poređani prema određenim pravilima.
- To su programski jezici visokog i niskog nivoa.
- Mašinski jezik i asemblerski jezik su niskog nivoa. Instrukcije napisane u tim jezicima su usko povezane sa organizacijom i mogućnostima hardvera. One pružaju malo apstrakcija.
- Go je programski jezik visokog nivoa koji se kompajlira.

[00 Sadržaj][00] | [02 Jezik Go][02]

[0101]: ./_images/0101.png
[0102]: ./_images/0102.png
[0103]: ./_images/0103.png

[00]: 00_Sadržaj.md
[02]: 02_Jezik_Go.md
