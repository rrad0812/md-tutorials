
# Najbolje prakse za AJAX JavaScript u Django-u

[Sadržaj](00_sadrzaj.md)

- Izbegavajte modifikovanje JavaScript koda sa Python kodom.
- Držite URL reference u HTML-u i upravljajte korisničkim porukama u Pajton kodu.
- Na HTML polju, preferirajte dodavanje ime klase.

Nakon što dodate ime klase, možete da priključite događaj za promenu na klasu `js-validite-username`. Prefiks JS na imenu klase sugerira na JavaScript kod koji komunicira sa ovim elementom.

Forme su u opasnosti od `Cross-Site Request Forgeries` - `CSRF`. Da biste sprečili napade `CSRF`-a, dodajte `{% csrf_token%}` šablon oznaku u formu. To će dodati polje skrivenog unosa koji sadrži token poslat sa svakim `POST` zahtevom.

[Sadržaj](00_sadrzaj.md)
