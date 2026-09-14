
# 02 Jezik Go

[01 Programiranje računara][01] | [00 Sadržaj][00] | [03 Terminal][03]

**Šta ćete naučiti u ovom poglavlju?**

- Poreklo jezika Go: kada je kreiran i od koga.
- Motivacije za stvaranje jezika Go.
- Koje su glavne karakteristike jezika?

**Obrađeni tehnički koncepti!**

- Vreme izgradnje
- Statički tipiziran jezik
- Konkurencija
- Sakupljač smeća
- Zavisnost od softvera

## Mit o stvaranju

Postoji mit o stvaranju jezika Go. Jezik je nastao u kancelariji kompanije Gugl, a to se desilo tokom veoma duge izgradnje koja je trajala 45 minuta.

Ovu priču je ispričao Rob Pajk na [@go-at-google]. Ona nam daje dragocene informacije o motivima koji stoje iza stvaranja Goa. Vremena izrade su bila preduga i mukotrpna... morali su da pronađu način da ih izbegnu; to je bila početna tačka za Go Genezis.

Robert Grisemer, Ken Tompson i Rob Pajk su programeri koji su počeli da rade na jeziku Go još 2007. godine. Rob Pajk tvrdi da je do sredine 2008. jezik bio "uglavnom dizajniran i da je implementacija (kompajler, vreme izvršavanja) počela da radi". Nakon toga, Ijan Lens Tejlor i Ras Koks su se pridružili timu 2008. godine.

Go je programski jezik otvorenog koda koji održava njegova zajednica i osnovni tim programera koji rade u kompaniji Gugl. 16. mart 2011. godine je datum prvog izdanja Go jezika. (Nazvan je "r56"). Go verzija 1 objavljena je 28. marta 2012. godine.

## Motivacija

Go (ili Golang) je napravio Google da bi rešio probleme kompanije. Da biste bolje razumeli razloge, vredi pročitati glavni govor Roba Pajka [@go-at-google].

Koji su izazovi softvera u velikim svetskim kompanijama?

- Baza koda Gugl servisa je ogromna. Gugl ima milione linija koda.
- Ti redovi su napisani u različitim jezicima: C, C++, Java i drugima.
- Vreme izgradnje tih aplikacija "proteglo se na mnogo minuta, čak i sati".
- Ažuriranja nekih delova aplikacije mogu biti skupa.

Cilj prvog Gophera bio je da olakša život programerima tako što će:

- Drastično smanjenje vremena izgradnje programa.
- Dizajniranje jezika koji je lak za učenje, čitanje i debagovanje za mlade programere koji su bili izloženi C, C++ ili Java.
- Dizajniranje efikasnog sistema za upravljanje zavisnostima.
- Izgradnja jezika koji može da proizvede softver koji se dobro skalira na hardveru.

4.1 Definicija nekih pojmova

- Vreme izgradnje : vreme potrebno kompajleru da generiše mašinski čitljivu izvršnu datoteku.
- Statički tipizirani jezik : Davanje precizne definicije ovog koncepta je sada prerano. Ovim terminom ćemo se baviti u narednim poglavljima.
- Zavisnost : deo softvera koji koristi drugi softver.
- Skalabilnost : sposobnost programa da obradi sve veći broj zadataka koje treba obaviti. Na primer, za veb lokaciju se kaže da je skalabilna ako može da prihvati sve veći broj zahteva bez zastoja ili povećanja kašnjenja učitavanja.

## Ključne karakteristika Goa

Kreatori Go-a su usmerili svoje napore na nekoliko ključnih dizajnerskih izbora:

- Kompilirani jezik
- Sa semantikom koja se lako razume i uči
- Statički tipiziran
- Sa ugrađenom konkurentnošću, sistem na kojem je programerima lako raditi
- Sa robusnim upravljanjem zavisnostima
- Sa sakupljačem smeća

Glavni cilj, kako je naveo Rob Pajk, bio je da se programerima pruži jezik koji je lak za učenje, a koji je pogodan za "inženjering velikih softverskih projekata".

### Neki koncepti

- Konkurencija : Program je konkurentan kada se zadaci mogu izvršavati van redosleda ili delimičnim redosledom2.

- Sakupljač smeća (često nazvan GC): Kada pravimo programe, potrebno je da skladištimo podatke i preuzimamo podatke iz memorije. Memorija nije beskonačan resurs. Stoga programer mora da osigura da se neiskorišćeni elementi sačuvani u memoriji povremeno uništavaju. Stavljanje nekih podataka u memoriju naziva se alokacija; obrnuta radnja, koja se sastoji od uklanjanja podataka iz memorije, naziva se dealokacija. Uloga sakupljača smeća je da dealocira memoriju kada se više ne koristi. Kada jezik nema sakupljač smeća, programer mora da sakupi svoje smeće i oslobodi memoriju koja se više ne koristi... Slučajno, Go ima sakupljač smeća.

## Stanje Goa

- Projekat je veoma brzo rastao i sada broji više od hiljadu saradnika.
- U vreme pisanja ovog teksta (8. januar 2020. godine), najnovija verzija programa Go je 1.15.6.
- Organizuje se mnogo sastanaka i konferencija kako bi se ujedinila zajednica
- U 2018. godini organizovano je 19 konferencija: 3 u SAD i 16 u drugim zemljama.
- U 2017. godini organizovano je 13 konferencija.

Programeri žele Go:

U anketi za programere na Stackoverflow-a 2018, 2019. i 2020. godine, Go je među tri najtraženija programska jezika.

## Testirajte sebe

### Pitanja i odgovori

1. Koji je datum rođenja jezika Go?
   - 2007
2. Šta znači istovremenost?
   - Program je konkurentan kada se zadaci mogu izvršavati istovremeno.
3. U proseku, Go pokazuje veoma dugo vreme izrade? Tačno ili netačno?
   - Netačno. Jezik je kreiran da bi se pozabavio upravo ovim problemom.

### Ključno

- Go je rođen 2007. godine.
- Verzija 1 programa Go je objavljena 2012. godine.
- Jezik je lako razumljiv. Njegova semantika ostaje jednostavna.
- Statički je tipiziran
- Kompajliran je
- Možete pisati konkurentne programe pomoću Go-a.
- Gophers - je nadimak Go programera
- Concurency : Ovaj koncept ćemo detaljnije razmotriti u drugom poglavlju. Definicija je preuzeta sa <https://en.wikipedia.org/wiki/Concurrency_(computer_science)>
- Dana 13. avgusta 2019. godine, projekat Go ima 1.349 saradnika
- pogledajte ovu stranicu <https://github.com/golang/go/wiki/Conferences>
- Ovi brojevi su preuzeti sa veb stranice <https://github.com/golang/go/wiki/Conferences>, provereno 13. avgusta 2019.

[01 Programiranje računara][01] | [00 Sadržaj][00] | [03 Terminal][03]

[01]: 01_Programiranje%20računara.md
[00]: 00_Sadržaj.md
[03]: 03_Terminal.md
