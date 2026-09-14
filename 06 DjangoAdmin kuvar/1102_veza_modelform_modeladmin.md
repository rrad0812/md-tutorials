
# Veza ModelForm - ModelAdmin

[Sadržaj](00_sadrzaj.md)

U Django adminu, `ModelAdmin` i `ModelForm` su usko povezani i ključni za generisanje interfejsa za administraciju. Evo kako funkcioniše njihova veza:

## ModelForm kao temelj

`ModelForm` je standardna Django klasa za kreiranje forme koja je direktno povezana sa modelom. Ona automatski generiše polja forme koja odgovaraju poljima u vašem modelu. Na primer, ako imate polje "CharField" u modelu, `ModelForm` će automatski kreirati odgovarajuće tekstualno polje u formi.

Osnovne funkcije `ModelForm`:

- `Generisanje polja forme`: Automatski stvara polja za unos podataka na osnovu definicije vašeg modela.

- `Validacija`: Pruža ugrađenu validaciju podataka. Kada korisnik unese podatke i pošalje formu, `ModelForm` proverava da li su podaci ispravni pre nego što ih sačuva.

- `Save podataka`: Ima metodu `save()` koja direktno kreira ili ažurira instancu modela u bazi podataka.

## ModelAdmin kao kontroler

`ModelAdmin` je klasa koja služi kao interfejs između vašeg modela i Django admin panela. On kontroliše kako će se vaš model prikazivati i uređivati u adminu. `ModelAdmin` koristi `ModelForm` "ispod haube" kako bi izvršio svoj posao.

`ModelAdmin` koristi `ModelForm` na sledeći način:

- `Automatsko korišćenje`: Kada registrujete model sa `admin.site.register(MyModel)`, Django admin automatski kreira `ModelForm` za vaš model. Ovaj podrazumevani `ModelForm` sadrži sva polja iz vašeg modela.

- `Prilagođavanje forme`: Ako želite da prilagodite formu za dodavanje i izmenu podataka (npr. da promenite redosled polja, dodate ili uklonite neka polja, ili da dodate prilagođenu validaciju), to možete uraditi u klasi `ModelAdmin` pomoću atributa form.

Primer: Možete definisati svoju klasu `ModelForm` i dodeliti je atributu `form` u `ModelAdmin` klasi. Time efektivno menjate podrazumevanu formu koju admin koristi.

U `models.py`:

```py
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
```

U `admin.py`

```py
from django import forms
from django.contrib import admin
from .models import Product

# Definišemo prilagođeni ModelForm
class ProductAdminForm(forms.ModelForm):
    class Meta:
        model = Product
        fields = '__all__' # Koristimo sva polja

    def clean_name(self):
        # Dodajemo prilagođenu validaciju za polje 'name'
        name = self.cleaned_data.get('name')
        if "test" in name.lower():
            raise forms.ValidationError("Ime proizvoda ne sme sadržati reč 'test'.")
        return name

# Povezujemo prilagođeni ModelForm sa ModelAdmin-om
class ProductAdmin(admin.ModelAdmin):
    form = ProductAdminForm # Ovde povezujemo ModelForm sa ModelAdmin

admin.site.register(Product, ProductAdmin)
```

U ovom primeru, `ProductAdminForm` je prilagođeni `ModelForm` koji dodaje jedinstvenu validaciju. Klasa `ProductAdmin` onda govori Django adminu da umesto podrazumevane forme, koristi `ProductAdminForm` za rad sa modelom Product.

Zaključak veze ModelForm - ModelAdmin

- `ModelForm` je srž logike za kreiranje forme, validaciju i save podataka.

- `ModelAdmin` je kontrolni panel koji koristi `ModelForm` za upravljanje podacima unutar Django admin interfejsa.

- Njihova veza je simbiotička: `ModelAdmin` pruža prikaz i kontrolu, dok `ModelForm` obezbeđuje funkcionalnost forme za unos i obradu podataka. Ako vam ne treba posebna logika, Django admin će se sam pobrinuti za sve, ali ako želite da prilagodite ponašanje forme, `ModelAdmin` je mesto gde "povezujete" svoju prilagođenu `ModelForm` klasu.

[Sadržaj](00_sadrzaj.md)
