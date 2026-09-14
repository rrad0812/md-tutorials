# Uslovno sprečavanje ažuriranja polja

[Sadržaj](00_sadrzaj.md)

Ponekad je korisno ažurirati polja User modela direktno u adminu. Ali ne želite da dozvolite bilo kom korisniku da to radi, želite da dozvolite samo superuser-ima da torade.

Recimo da želite da sprečite korisnike koji nisu superuser-i da promene username korisnika. Da biste to uradili, potrebno je da izmenite formu za promenu koji je generisao Django i onemogućite polje za username na osnovu trenutnog korisnika.

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    
    def get_form(self, request, obj=None, **kwargs):
        form = super().get_form(request, obj, **kwargs)
        is_superuser = request.user.is_superuser
        if not is_superuser:
            form.base_fields['username'].disabled = True
        return form
```

Da biste izvršili prilagođavanje obrasca, nadjačajte get_form()metodu. Ovu metodukoristi Django za generisanje podrazumevanog obrasca za promenu modela. Da biste uslovno onemogućili polje, prvo dobavite podrazumevani obrazac koji je generisao Django, a zatim, ako korisnik nije superuser, onemogućite username polje.

Sada, kada korisnik koji nije superuser pokuša da izmeni username korisnika, polje usernameće biti onemogućeno. Svaki pokušaj izmene username polja putem Django admina neće uspeti. Kada superuser pokuša da izmeni username korisnika, polje username će moćida se uređuje i ponašaće se onako kako se očekuje.

[Sadržaj](00_sadrzaj.md)
