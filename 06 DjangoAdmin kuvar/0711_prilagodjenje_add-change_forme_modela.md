
# Prilagodjenje add/change forme modela

[Sadržaj](00_sadrzaj.md)

Forme koje se koriste za dodavanje ili promenu objekta modela zasnivaju se na ModelForm. Django automatski generiše formu na osnovu modela koji se uređuje.

Uređivanjem opcija možete da kontrolišete koja su polja uključena, kao i njihov redosled. Izmenite svoj PersonAdmin objekat, dodajući atribut fields:

```py
@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):

fields = ("first_name", "last_name", "courses")
...
```

Stranice Add i Change za Person model sada stavljaju atribut first_name ispred last_name, iako sam model određuje obrnuto.

ModelAdmin.get_form() metod odgovoran je za kreiranje ModelForm-e za vaš objekat. Ovu metodu možete nadjačati da biste promenili formu. Dodajte sledeći kod u PersonAdmin:

```py
def get_form(self, request, obj=None, **kwargs):
    form = super().get_form(request, obj, **kwargs)
    form.base_fields["first_name"].label = "First Name (Humans only!):"
    return form
```

Sada, kada se prikaže stranica Add ili Change, oznaka polja first_name biće prilagođena.

Promena labele polja možda neće biti dovoljna da spreči vampire da se registruju kao studenti. Ako vam se ne sviđa ModelForm koji je Django admin kreirao za vas, tada možete da koristite form atribut da biste registrovali svoj prilagođeni obrazac.

Napravite sledeće dodatke i promene na core/admin.py:

```py
from django import forms

class PersonAdminForm(forms.ModelForm):

class Meta:
    model = Person
    fields = " all "
    
    def clean_first_name(self):
        if self.cleaned_data["first_name"] == "Spike"
            raise forms.ValidationError("No Vampires")
        return self.cleaned_data["first_name"]

@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    form = PersonAdminForm
    ...
```

Gornji kod primenjuje dodatnu proveru valjanosti na stranicama Person Add i Change. Objekti ModelForm imaju bogat mehanizam za proveru valjanosti. U ovom slučaju, polje first_name se proverava prema imenu „Spike“. Greška ValidationError sprečava studente sa ovim imenom da se registruju. Nadjačavanjem ModelForm klase možete u potpunosti kontrolisati izgled i proveru valjanosti stranica koje koristite za stranice formi za dodavanje ili promenu objekata modela.

[Sadržaj](00_sadrzaj.md)
