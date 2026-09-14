
# Nadjačavanje save ponašanja add-change strane

[Sadržaj](00_sadrzaj.md)

ModelAdmin ima save_model metod, koji se koristi za kreiranje i ažuriranje objekata modela. Njegovim nadjačavanjem, možemo prilagoditi ponašanje save za admin. Hero model ima sledeće polje:

```py
added_by = models.ForeignKey(settings.AUTH_USER_MODEL, null=True, blank=True,
    on_delete=models.SET_NULL)
```

Ako želite da uvek sačuvate tekućeg korisnika kada je Hero ažurirao, uradite:

```py
def save_model(self, request, obj, form, change):
    obj.added_by = request.user
    super().save_model(request, obj, form, change)
```

[Sadržaj](00_sadrzaj.md)
