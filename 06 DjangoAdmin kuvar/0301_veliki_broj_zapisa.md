
# Veliki broj zapisa na listview strani

[Sadržaj](00_sadrzaj.md)

Treba da povaćate broj prikazanih zapisa na Hero `listview` admin strani na 250.

Podrazumevano je 100.:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    list_per_page = 250
```

> [!Note]
>
> Možete takodje postaviti i manju vrednost od 100.

[Sadržaj](00_sadrzaj.md)
