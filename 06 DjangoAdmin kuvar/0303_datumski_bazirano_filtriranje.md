
# Datumski bazirano filtriranje na listview strani

[Sadržaj](00_sadrzaj.md)

Možete dodati datumski bazirano filtriranje sa postavkom `date_hierarchy`:

```py

@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    date_hierarchy = 'added_on'
```

Ovo može biti skupo na velikom broju objekata. Kao alternativa, možete naslediti
`SimpleListFilter`, i dozvoliti filtriranje samo na godinama ili mesecima.

Možete dodati datumski bazirano filtriranje sa postavkom `date_hierarchy`:

```py

@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    date_hierarchy = 'added_on'
```

Ovo može biti skupo na velikom broju objekata. Kao alternativa, možete naslediti
`SimpleListFilter`, i dozvoliti filtriranje samo na godinama ili mesecima.

[Sadržaj](00_sadrzaj.md)
