
# Prilagođeni filter u adminu

[Sadržaj](00_sadrzaj.md)

Sada zamislimo da želimo da dodamo filter zasnovan na poslovnim pravilima,a ne na određenom polju. Da bismo to uradili, treba da:

- Nasledimo klasu `SimpleListFilter`,
- Zadamo ime filtera preko osobine "name".
- Zadamo ime parametra filtera preko osobine `parametar_name`,
- Nadjačamo metodu `lookups`,
- Nadjačamo metodu `queryset`

`lookups` metod treba da vrati listu slogova.

- Prvi element sloga je kodirana vrednost koja će se pojaviti u url-u upita,
- Drugi element sloga je ime filtera koji će biti prikazan na admin interfejsu.

`queryset` metod vraća filtrirani `queryset`. Pogledajmo:

```py
class ProductPriceFilter(admin.SimpleListFilter):
    title = 'price'
    parameter_name = 'price'

    def lookups(self, request, model_admin):
        """
        Vraća listu slogova, prvi element svakog sloga je kodirana vrednost opcije koja će se pojaviti u query URL-u, drugi element je ime opcije koja će se pojaviti na filteru.
        """
        
        return (
            ("low_price", "Filter by low price"), 
            ("high_price","Filter by high price")
        )

    def queryset(self, request, queryset):
        """
        Vraća filtrirani queryset baziran na vrednosti query stringa, vraćenog sa `self.value()`.
        """
        if self.value()=="low_price":
            return queryset.filter(price lte=5)
        elif self.value() == "high_price":
            return queryset.filter(price gt=5)
        else:
            return queryset.all()
```

U ovom primeru ćemo dodati mogućnost filtriranja proizvoda po niskoj ili visokoj ceni. Smatramo da je niska cena cena ispod ili jednaka 5, dok će visoka cena biti iznad 5.

Da biste koristili filter treba samo da ukažete na ime filterske klase u `list_filter` opciji:

```py
list_filter = (ProductPriceFilter, )
```

Kao što vidimo, u interfejsu imamo novi filter "By price". Ako kliknemo na jednu od vrednosti, lista prikazanih rezultata će se promeniti i ako obratimo pažnju na url, možemo videti da izabrana vrednost filtera u url-u sadrži naš parametar dodan u metodu pretraživanja:

<http://127.0.0.1:8000/admin/backoffice/product/?price=low_price>

[Sadržaj](00_sadrzaj.md)
