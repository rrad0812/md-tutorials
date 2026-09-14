
# Validacija pojedinačnog polja

[Sadržaj](00_sadrzaj.md)

Možemo da proverimo da li je reč koju je poslao korisnik palindrom tako što ćemo izvršiti dodatnu proveru na nivou polja. Prvo, pogledajmo naš model palindroma koji ima samo jedno polje za našu reč.

`#models.py`

```py
from django.db import models

class Palindrome(models.Model):
    word = models.CharField(max_length=250, blank=False)
    
    def _ str_ (self):
        return self.word
```

Dalje, napravićemo formu da bi korisnik mogao da preda palindrom.

Nasleđujemo od `ModelForm` jer želimo da sačuvamo korisničke podatke na modelu "Palindrome". U ovom trenutku, forma ima samo Django-ovu automatsku proveru forme. Ako bismo formu povezali sa prikazom, šta bi se dogodilo kada korisnik pošalje reč u formu koja nije palindrom?

Čuva se u bazi podataka. To nije ono što želimo, ali još uvek nismo učinili ništa da to sprečimo! Da bismo proverili da li je poslata reč palindrom, moramo da modifikujemo našu `PalindromeForm` tako da uključuje test koji proverava da li je reč ista napred i nazad.

`forms.py`

```py
from django.forms import ModelForm, ValidationError # Import ValidationError
from .models import Palindrome

class PalindromeForm(ModelForm):

    def clean_word(self):
        # Uzmi korisnički unešenu reč iz rečnika cleaned_data
        data = self.cleaned_data["word"]
        
        # Proveri da li je reč palindrom
        if data != "".join(reversed(data)):
            # Ako nije podigni grešku
            raise ValidationError("The word is not a palindrome")

        # Vrati podatak, uvek, čak iako nije menjan
        return data

    class Meta:
        model = Palindrome
        fields = ["word"]
```

Samo treba da izvršimo dodatnu proveru na nivou polja da bismo proverili da li je reč palindrom. Da bismo dodali ovu dodatnu validaciju polja, definišemo metodu `clean_word` koja deluje samo na polju word. Ova metoda će se pozvati nakon Djangove automatske provere valjanosti koja za nas stvara rečnik `clean_data`.
"Word" dobijemo iz `clean_data` i čuvamo je kao "data".

Sada konačno moramo da proverimo da li je reč koju je korisnik poslao palindrom upoređujući je sa obrnutom verzijom samog sebe. Ako je reč palindrom, provera prolazi i metoda vraća izvorni podatak. Ako reč nije palindrom, podižemo `ValidationError` sa prilagođenom porukom o grešci. Ova poruka će se dodati
u formu i prikazati korisniku kada se ponovo prikaže sa greškama zbog neuspešnog potvrđivanja forme.

Ako je naša forma imala više polja koja smo želeli da potvrdimo, samo treba da dodamo metodu `clean_"fieldname"` za svako polje za koje želimo da dodamo prilagođenu proveru.

Mogli smo da uključimo i više provera validacije u našu metodu "clean_word" ako smo, na primer, takođe želeli da se uverimo da je svaki podneti palindrom duži od četiri slova.

[Sadržaj](00_sadrzaj.md)
