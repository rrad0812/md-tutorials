
# Validacija više polja

[Sadržaj](00_sadrzaj.md)

Pogledajmo sada primer forme koji ima polja za first_name i last_name i želimo da oba počinju istim slovom. U ovom slučaju, pošto moramo da izvršimo proveru validacije koja uključuje više od jednog polja, naša validacija se vrši na nivou forme.

Naš model će imati dva CharField polja first_name i last_name.

`models.py`

```py
from django.db import models

class Person(models.Model):
    first_name = models.CharField(max_length=250, blank=False)
    last_name = models.CharField(max_length=250, blank=False)
```

Kao i prethodni primer, započećemo sa osnovnom formom koji nema prilagođenu proveru valjanosti.

`forms.py`

```py
from django.forms import ModelForm
from .models import Person

class PersonForm(ModelForm):

    class Meta:
        model = Person
        fields = ["first_name", "last_name"]
```

Kao i ranije, naša forma će omogućiti da se bilo koji niz znakova pošalje za "first_name" i "last_name" i kreiraće i sačuvati instancu modela sa tim vrednostima bez grešaka.

Da bismo dodali proveru valjanosti, zamenimo metodu `clean` u formi.

`forms.py`

```py
from django.forms import ModelForm, ValidationError # Import ValidationError
from .models import Person

class PersonForm(ModelForm):
    
    def clean(self):
        # Uzmi korisnički unešena imena iz cleaned_data rečnika
        cleaned_data = super().clean()
        first_name = cleaned_data.get("first_name")
        last_name = cleaned_data.get("last_name")

        # Proveri da li je prvo slovo imena i prezimena isto
        if first_name[0].lower() != last_name[0].lower()
            # Ako nije podigni grešku
            raise ValidationError("The first letters of the names do not match")

        return cleaned_data
    
    class Meta:
        model = Person
        fields = ["first_name", "last_name"]
```

Ovo se razlikuje od prethodnog primera gde smo kreirali metodu `clean_<ime polja>` jer sada gledamo više polja umesto jednog polja. Kada zamenjujemo `clean` na `ModelForm`, prvo pozivamo metodu `clean` roditeljske klase da bismo mogli imati pristup `clean_data`.

Ostali tipovi formi možda neće vratiti `cleaned_data` iz metode clean na roditeljskoj klasi. U tom slučaju nazivamo `super().clean()` samostalno bez dodeljivanja i pristupimo očišćenim podacima putem `self.cleaned_data`.

Jednom kada smo dobili `clean_data` možemo dobiti vrednosti "first_name" i "last_name". Tada možemo izvršiti stvarnu proveru valjanosti da li ime i prezime počinju istim slovom. Ako se ne podudaraju, pokrećemo grešku validacije koja se dodaje u formu i prikazuje korisniku.

[Sadržaj](00_sadrzaj.md)
