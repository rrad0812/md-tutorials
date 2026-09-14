
# Povezivanje modela sa trenutnim korisnikom

[Sadržaj](00_sadrzaj.md)

"Hero" model ima "added_by" polje:

```py
added_by = models.ForeignKey(settings.AUTH_USER_MODEL, null=True, blank=True,
    on_delete=models.SET_NULL)
```

Vi želite da "added_by" polje bude automatski postavljeno na trenutnog korisnika kada se objekat kreira u adminu:

```py
def save_model(self, request, obj, form, change):
    if not obj.pkobj.pk:
        # Only set added_by during the first save.
        obj.added_by = request.user
        super().save_model(request, obj, form, change)
```

Umesto toga, ako želite da sačuvate tekućeg korisnika prilikom promena zapisa:

```py
def save_model(self, request, obj, form, change):
obj.updated_by = request.user
super().save_model(request, obj, form, change)
```

Ako želite sa sakrijete "added_by", "updated_by" polja od prikazivanja na `changeview` strani:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    exclude = ['added_by', 'updated_by',]
```

[Sadržaj](00_sadrzaj.md)
