
# Nadjačavanje dozvola na User modelu

[Sadržaj](00_sadrzaj.md)

Ponekad može biti korisno potpuno nadjačati dozvole na User modelu admina. Uobičajeni scenario je kada koristite dozvole na drugim mestima i ne želite da korisnici osoblja unose promene u admin.

Django koristi hook metode za četiri ugrađene dozvole. Interno, kuke koriste trenutne dozvole korisnika za donošenje odluke. Možete da izmenite ove kuke i donesete drugačiju odluku.

Da biste sprečili korisnike osoblja da izbrišu instancu modela, bez obzira na njihove dozvole, možete učiniti sledeće:

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    
    def has_delete_permission(self, request, obj=None):
        return False
```

Baš kao i kod get_form(), obj je instanca kojom se trenutno upravlja:

- Kada je obj is None, korisnik je zatražio prikaz liste.
- Kada obj is not None, korisnik je zahtevao prikaz strane za promenu određene instance.

Imati instancu modela u ovim hook metodama vrlo je korisno za primenu dozvola na nivo instance za različite tipove akcija. Evo i drugih slučajeva upotrebe:

- Sprečavanje promena tokom radnog vremena
- Implementacija dozvola na nivou instance

[Sadržaj](00_sadrzaj.md)
