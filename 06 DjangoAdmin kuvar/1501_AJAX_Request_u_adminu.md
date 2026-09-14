
# AJAX Request

[Sadržaj](00_sadrzaj.md)

`AJAX` je akronim za `Asynchronous JavaScript And XML`. AJAX tehnologije omogućavaju web stranama da rade ažuriranje asinhronom razmenom podataka sa serverom. U suštini, web strana može biti ažurirana bez učitavanja kompletne web strane. To se ostvaruje preko `XMLHttpRequest` objekta za slanje i primanje podataka. Kada se događaj kao: slanje podataka forme, učitavanje strane ili kada je neko dugme kliknuto, na web strani dogodi, jedan `XMLHttpRequest` objekat je kreiran i poslan je zahtev na server. Server će odgovoriti na zahtev. Zahtevani odgovor je uhvaćen i web server odgovara sa zahtevanim podacima.

`AJAX` je koristan u scenarijima kada želite da napravite `GET` ili `POST` zahtev za učitavanjem podataka sa servera asinhrono. To omogućava aplikacijama da budu dinamičnije i redukuje vreme učitavanja.

[Sadržaj](00_sadrzaj.md)

## Kako Ajax radi

- `Javascript kod u Browser/client-strani`

  Brauzer podnosi zahtev kada se događaj dogodi na web stranici. JavaScript kod generiše `XHR` objekat koji sadrži JavaScript objekt ili podatke i šalje se kao objekt zahteva na webserver. `XHR` objekt će u osnovi  imenovati funkciju povratnog poziva na serveru ili URL-u.

- `Zahtevom upravlja webserver`

  Kada je zahtev poslan, odgovarajuća funkcija povratnog poziva rukuje zahtevom i ona će poslati uspeh ili neuspeh odgovor. Zbog asinhrone prirode zahteva, kod će se izvršiti bez prekida, a server obrađuje zahtev.

- `Uspešan ili neuspešan odgovor`

  Uspešan odgovor će sadržati podatke u više formata poput `teksta`, `JSON`, `XML`.

- `Stvarni primer`

  `AJAX` ima veoma mnogo aplikacija na veb stranama i aplikacijama. Primer je taster sličan na Facebooku ili Linkedln. Kada se post lajkuje, nekoliko akcija se izvodi i na serveru i na klijentu. Na primer, boja tastera može se promeniti ili će se broj lajkova na postu biti uvećan u bazi podataka.

- `Pojednostavljeni proces`

  Kada se klikne na lajk dugme, `XHR` objekat se šalje na server, server prima zahtev i vrši radnju kreiranu za zahtev. U ovom slučaju se registruje lajk za post. Server zatim šalje odgovor uspeha, a taster lajk menja boju ili je povećan broj lajkova ili oboje.

[Sadržaj](00_sadrzaj.md)

## Pogled na Django

Django je opisan kao web okvir za perfekcioniste sa rokovima i prevladava kao Pajton web okvir. Django se može okarakterisati kao `MVC` (`Model View Controller`) i `MTV` (`Model Template View`). Klase modela su u orm-u i instance odgovaraju redovima u tabeli. Šablon sistem je dizajniran za lakoću upotrebe i ograničava meru u kojoj se HTML koristi kroz Python izvorni kod. View je funkcija koja pravi odgovor uz pomoć šablona.

[Sadržaj](00_sadrzaj.md)

## Korišćenje jQuery za AJAX u Django

`JQuery` je jedna od najpopularnijih JavaScript biblioteka koje su korišćene za razvijanje drugih okvira JavaScript-a poput React-a ili Angular-a. Omogućava funkcije sa DOM na web stranama.

[Sadržaj](00_sadrzaj.md)

## Zašto koristiti jQuery

`Ajax` ima drugačiju implementaciju na više brouzera. Različiti pretraživači koriste razne `XHR` API-e koji analiziraju zahtev i odgovoraju neznatno različito.

`JQuery` vam pruža ugrađeno rešenje da ažurirate svoj kod iako je to što je učinio nezavisno. JQuery kod se aktivno razvija i stoga se ažurira.

**jQuery pruža različite metode korišćenja DOM-a**:

`jQuery` sadrži mnoge DOM objekte i srodne metode. Ovi se objekti mogu koristiti u zavisnosti od potreba programera, a `JQuery` ga čini pogodnijim za upotrebu.

**JQuery je osnova za mnoge popularne okvire**:

Okviri poput React JS-a, Angular JS zasnivaju se na `jQuery`-u. `jQuery` je praktično svuda u popularnim okvirima JavaScript. `jQuery` je veoma popularan.

**jQuery je viralna biblioteka među web programerima**:

JavaScript okviri pružaju gotove widgete, a okviri mogu vam dozvoliti da napišete JavaScript višeg nivoa i malo više pythonic. Postoje i drugi načini za upotrebu `Ajax` u Django-u, ali ovi razlozi čine `JQuery` popularnim izborom. Stoga ćemo ga koristiti da bismo demonstrirali kako raditi sa zahtevima u Django-u.

**Scenario registracija korisnika**:

Kao web programer, želite da validirate polje "User" "name" u registraciju ili u prikazu registracije, čim korisnik završi upisivanje korisničkog imena. Provera koju želite da napravite je jednostavna.

[Sadržaj](00_sadrzaj.md)
