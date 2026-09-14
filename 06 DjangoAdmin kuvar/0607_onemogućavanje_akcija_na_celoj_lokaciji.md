
# Onemogućavanje akcije na celoj lokaciji

[Sadržaj](00_sadrzaj.md)

Ako želite da onemogućite akciju na celoj lokaciji, možete da pozovete `AdminSite.disable_action()`. Na primer, ovom metodom možete ukloniti ugrađenu akciju `"brisanje izabranih objekata"`:

```py
admin.site.disable_action('delete_selected')
```

Kada učinite gore navedeno, ta akcija više neće biti dostupna na celoj veb lokaciji. Ako, međutim, trebate da ponovo omogućite globalno onemogućenu aciju za jedan određeni model, navedite je u `ModelAdmin.actions` listi:

Ovaj ModelAdmin neće imati `delete_selected` raspoloživu klasu akciju:

```py
SomeModelAdmin(admin.ModelAdmin):
    actions = ['some_other_action']
    ...
```

A ovaj će imati:

```py
class AnotherModelAdmin(admin.ModelAdmin):
    actions = ['delete_selected', 'a_third_action']
    ...
```

[Sadržaj](00_sadrzaj.md)
