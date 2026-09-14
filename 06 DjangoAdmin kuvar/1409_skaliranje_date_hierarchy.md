
# Skaliranje admin date_hierarchy

[Sadržaj](00_sadrzaj.md)

Ako niste familijarni sa Django Admin `date_hierarchy` opcijom - trebalo bi, sjajna je.Postavlja `date_hierarchy` atribut `ModelAdmin` na neko `DateField` polje:

```py
from django.contrib import admin
from .models import Sale

@admin.register(Sale)
class SaleAdmin(admin.ModelAdmin):
    date_hierarchy = 'created'
```

Na ovaj način dobijete lep propadajuči meni na vrhu list view strane. Kada izaberete godinu, Django filtrira podatke i prikaže listu meseci za datu godinu.

[Sadržaj](00_sadrzaj.md)
