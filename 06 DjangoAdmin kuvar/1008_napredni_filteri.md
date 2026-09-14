
# Napredni filteri

[Sadržaj](00_sadrzaj.md)

Sve gornje metode biće korisne samo kao odredjeno proširenje. Preko toga, postoje paketi kao `django-advancedfilters` koji imaju napredne mogućnosti filtriranja.

Za postavku paketa

- Instaliraj paket sa: `pip install django-advanced-filters`
- Dodaj `advanced_filters` u `INSTALLED_APPS`
- Dodaj u projektni `urlconf.py`: `url(r’^advanced_filters/’`,

include(‘advanced_filters.urls’))
-Pokreni python manage.py migrate.

Kada je setup završen možemo pisati:

```py
from advanced_filters.admin import AdminAdvancedFiltersMixin

class BookAdAdminFilter(AdminAdvancedFiltersMixin, admin.ModelAdmin):
    list_display = ('id', 'name', 'author', 'published_date', 'is_available')
    advanced_filter_fields = ('name', 'published_date', 'author', 'is_available')
```

Jednostavno se može kreirati filter za sve knjige publikovane izmedju 1980 i 1990, koje imaju ocenu višu od 3.75 i broj strana veći od 100. Ovakav filter se može imenovati i sačuvati za kasnije korišćenje.

Za više detalja pogledati na: <https://github.com/modlinltd/django-advanced-filters>

[Sadržaj](00_sadrzaj.md)
