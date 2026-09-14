# Zaključak

[Sadržaj](00_sadrzaj.md)

Validacija prilagođene forme vrši se na različite načine, u zavisnosti od toga da li je validacija potrebna na nivou polja za jedno polje forme ili na nivou forme za više polja.

Da bi se dodala validacija za jedno polje, u formu se dodaje metoda `clean_<ime_polja>` za željeno ime polja koje treba potvrditi. Ovom metodom treba dobiti podatke o polju izrečnika `self.cleaned_data` i vratiti podatke o polju bez obzira da li su izmenjeni ili ne. Uslovni izrazi se dodaju metodi za izvođenje željene provere valjanosti i podizanje ValidationErrors ako provera valjanosti ne uspe.

Za validaciju odnosa između više polja poziva se metoda `clean` na formi. Podaci polja postaju dostupni pozivom metode roditeljske klase clean putem `super().clean()` koji vraća očišćene podatke polja za `ModelForm`. Sada se može pristupiti svim podacima polja iz forme i izvršiti željena provera valjanosti da bi se po potrebi pojavila `ValidationError`.

[Sadržaj](00_sadrzaj.md)
