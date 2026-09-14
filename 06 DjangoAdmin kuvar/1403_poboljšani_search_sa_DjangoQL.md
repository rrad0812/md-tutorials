# Poboljšani search sa DjangoQL

[Sadržaj](00_sadrzaj.md)

Standardni search radi dobro ali posle ulaska u sferu miliona zapisa, search postaje problem. Kada je više od jednog polja u `search_fields`, SQL upit ima višestruke `JOIN` komande ulančane jedna na drugu. Rešenje može biti uvodjenje `DjangoQL` paketa.

`DjangoQLSearchMixin` menja standardni Django `search` sa `DjangoQL search`-om. DjangoQL pomaže nam da radimo `search` kroz specifikovana polja, uključujući povezana. Sve što je potrebno je dodavanje `DjangoQLSearchMixin` na `ModelAdmin`:

```py
from djangoql.admin import DjangoQLSearchMixin

class MyModelAdmin(DjangoQLSearchMixin, ModelAdmin):
    pass
```

DjangoQL autokompletira polja modela, podržava logičke operatore, zagrade i povezivanje tabela. Možete dobiti višestruke JOIN komande sa samo nekoliko linija koda i fokus na ono što pretražujete bez ograničenja kao u tvrdo kodovanom `search_fields`.

[Sadržaj](00_sadrzaj.md)
