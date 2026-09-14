
# Dodela dozvol preko grupa samo

[Sadržaj](00_sadrzaj.md)

Način upravljanja dozvolama vrlo je specifičan za svaki tim, proizvod i kompaniju. Otkrio sam da je lakše upravljati dozvolama sa grupama. U svojim projektima stvaramgrupe za podršku, urednike sadržaja, analitičare itd. Otkrio sam da upravljanje dozvolama na korisničkom nivou može biti prava gnjavaža. Kada se dodaju novi modeli ili kada se promene poslovni zahtevi, zamorno je ažurirati svakog pojedinačnog korisnika.

Da biste upravljali dozvolama samo pomoću grupa, morate da sprečite korisnike da dodeljuju dozvole drugim korisnicima ili sami sebi. Umesto toga, želite da dozvolite samo pridruživanje korisnika grupama.

Da biste to uradili, onemogućite polje user_permissions za sve korisnike koji nisu superuser-i:

```py
from typing import Set
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    
    def get_form(self, request, obj=None, **kwargs):
        form = uper().get_form(request, obj, **kwargs)
        is_superuser = request.user.is_superuser
        disabled_fields = set() type: Set[str]
        if not is_superuser:
            disabled_fields |= {'username', 'is_superuser', 'user_permissions',}
        for f in disabled_fields:
            if f in form.base_fields:
                form.base_fields[f].disabled = True
        return form
```

Za primenu drugog poslovnog pravila koristite potpuno istu tehniku kao i prethodno. U sledećim odeljcima ćete primeniti složenija poslovna pravila kako biste zaštitili svoj sistem.

[Sadržaj](00_sadrzaj.md)
