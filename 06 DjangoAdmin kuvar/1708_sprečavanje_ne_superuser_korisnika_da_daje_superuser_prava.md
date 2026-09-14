
# Sprečavanje ne superuser-a da daje superuser prava

[Sadržaj](00_sadrzaj.md)

Superuser je vrlo jaka dozvola koja se ne sme davati olako. Međutim, bilo koji korisnik sa dozvolom za promene na User modelu može od bilo kog korisnika napraviti superuser-a, uključujući i same sebe. To se protivi celoj svrsi sistema dozvola, paželite da zatvorite ovu rupu.

Na osnovu prethodnog primera, da biste sprečili korisnike koji nisu superuser-i da daju sami sebi prava superusrer-a, dodajete sledeće ograničenje:

```py
from typing import Set
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    
    def get_form(self, request, obj=None, **kwargs):
        form = super().get_form(request, obj, **kwargs)
        is_superuser = request.user.is_superuser
        disabled_fields = set() type: Set[str]
        if not is_superuser:
            disabled_fields |= {'username', 'is_superuser',}

        for f in disabled_fields:
            if f in form.base_fields:
                form.base_fields[f].disabled = True
        return form
```

U get_form metodi možete inicijalizovati prazan set disabled_fields koji će sadržati polja za onemogućavanje. Set je struktura podataka koja sadrži jedinstvene vrednosti. U ovom slučaju ima smisla koristiti set, jer polje možete samo jednom onemogućiti. Operator |= se koristi za izvođenje ažuriranja na setu. Dalje, ako korisnik nije superuser, u set dodajte dva polja (username i is_superuser). Oni će sprečiti osobe koje nisu superuser-i da se naprave superuser-ima.

Na kraju, prelistate polja u skupu, označite ih kao onemogućena i vratite formu.

[Sadržaj](00_sadrzaj.md)
