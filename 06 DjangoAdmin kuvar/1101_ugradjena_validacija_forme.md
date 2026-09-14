
# Ugrađena validacija forme

[Sadržaj](00_sadrzaj.md)

Dobra je praksa ne prihvatati podatke dostavljene od korisnika takvi kakvi jesu.  Podaci forme moraju biti potvrđeni kako bi se osiguralo da su podaci validni i da ispunjavaju sve neophodne uslove. U Django-u se ova provera vrši na dva nivoa - nivou polja i nivou forme, a oba omogućavaju dodavanje prilagođene provere.

Django nam pomaže tako što automatski potvrđuje polja forme. To znači da možemo biti sigurni da su dostavljeni podaci tipa koji očekuje polje forme a ako nisu doći će do greške.

Ako, na primer, očekujemo da korisnik pošalje datum, a umesto toga podnese ’string', pojaviće se greška koja govori korisniku da ’string’ nije važeći datum.

Django takođe može da obezbedi automatsku proveru valjanosti u zavisnosti od polja forme. U Django dokumentima možete pronaći koje su validacije dostupne za svaki tip polja forme.

Na nivou forme, Django može da vrši automatsku proveru jedinstvenosti između više polja. Ove provere zavise od načina na koji smo podesili naše modele i pomažu u osiguravanju da korisnici ne šalju duplikate podataka ako mi to ne želimo.

Ali šta ako želite da budete sigurni da ono što korisnik podnese ispunjava neki drugi uslov koji nije automatski potvrđen? Na primer, ako smo očekivali da će korisnik predati palindrom (reč ili faza istu napred i nazad), kako bismo mogli da proverimo dali je to tačno? Ili, ako bismo samo želeli da korisnici predaju
imena ljudi koji imaju imena i prezimena koja počinju istim slovom, kako bismo to mogli proveriti?

Istražimo oba ova scenarija.

[Sadržaj](00_sadrzaj.md)
