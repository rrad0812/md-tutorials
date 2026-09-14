
# Uslovno omogućavanje ili onemogućavanje akcija

[Sadržaj](00_sadrzaj.md)

Konačno, možete uslovno da omogućite ili onemogućite akcije po zahtevu (a time i po korisniku) nadjačavanjem `ModelAdmin.get_actions()`. Ovo vraća rečnik dozvoljenih akcija. Ključevi su imena akcija, a vrednosti su slogovi.

Na primer, ako želite da korisnici čija imena počinju sa „J“ mogu da obrišu objekte u velikom broju:

```py
class MyModelAdmin(admin.ModelAdmin):
    ...
    def get_actions(self, request):
        actions = super().get_actions(request)
        if request.user.username[0].upper() != 'J':
            if 'delete_selected' in actions:
                del actions['delete_selected']
        return actions
```

[Sadržaj](00_sadrzaj.md)
