
# Sprečavanje ne superuser-a od promena vlastitih dozvola

[Sadržaj](00_sadrzaj.md)

Jaki korisnici su slaba tačka. Oni imaju jake dozvole, i potencijalno oštećenje koje mogu izazvati može biti značajno. Za sprečavanje eskalacije prava u slučaju upada, možete sprečiti korisnike da menjaju svoje dozvole:

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
            disabled_fields |= {'username', 'is_superuser', 'user_permissions',}

        # Prevent non-superusers from editing their own permissions
        if (not is_superuser and obj is not None and obj == request.user)
            disabled_fields |= {
                'is_staff', 'is_superuser', 'groups', 'user_permissions',
            }           
        for f in disabled_fields:
            if f in form.base_fields:
                form.base_fields[f].disabled = True
        return form
```

Argument obj je instanca na kojoj trenutno radite:

- Kada je obj is None, forma se koristi za kreiranje novog korisnika.
- Kada je obj is not None, forma se koristiti za ažuriranje postojećeg korisnika.

Da biste proverili da li korisnik koji podnosi zahtev radi na sebi, uporedite sa request.user sa obj. Kada je za korisnika koji pravi zahtev request.user==obj, to znači da se korisnik ažurira. U ovom slučaju onemogućavate sva osetljiva polja koja se mogu koristiti za dobijanje dozvola.

Sposobnost prilagođavanja forme na osnovu objekta je veoma korisna. Može se koristiti za sprovođenje složenih poslovnih uloga.

[Sadržaj](00_sadrzaj.md)
