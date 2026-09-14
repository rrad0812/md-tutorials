
# Ograničenje pristupa prilagođenim akcijama na User modelu

[Sadržaj](00_sadrzaj.md)

Prilagođene admin akcije zahtevaju posebnu pažnju. Django njih ne poznaje, tako da impodrazumevano ne može ograničiti pristup. Prilagođena akcija biće dostupna bilo kom adminu sa bilo kojom dozvolom na modelu.

Da bismo ilustrovali, dodajte praktičnu admin akciju da biste više korisnika označili kao aktivne:

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    actions = [
        'activate_users',
    ]

    def activate_users(self, request, queryset):
        cnt = queryset.filter(is_active=False).update(is_active=True)
        self.message_user(request, 'Activated {} users.'.format(cnt))
        activate_users.short_description = 'Activate Users' type: ignore
```

Koristeći ovu akciju, staff korisnik može označiti jednog ili više korisnika i aktivirati ih sve odjednom. Ovo je korisno u svim vrstama slučajeva, na primer ako imate grešku u procesu registracije i trebate da aktivirate korisnike na veliko. Ova akcija ažurira korisničke informacije, tako da želite da ih mogu koristiti samo korisnici sa dozvolama za promenu.

Django admin koristi get_action metodu za dobijanje instance akcije. Da biste sakrili metodu activate_users() od korisnika bez dozvole za promenu User modela, nadjačajte metodu get_action():

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.adminimport UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    actions = [
        'activate_users',
    ]

    def activate_users(self, request, queryset):
        assert request.user.has_perm('auth.change_user')
        cnt = queryset.filter(is_active=False).update(is_active=True)
        self.message_user(request, 'Activated {} users.'.format(cnt))
        activate_users.short_description = 'Activate Users' type: ignore

    def get_actions(self, request):
        actions = super().get_actions(request)
        if not request.user.has_perm('auth.change_user'):
            del actions['activate_users']
        return actions
```

get_action() vraća OrderedDict. Ključ je ime akcije, a vrednost funkcija akcije. Da biste prilagodili povratnu vrednost, nadjačajte funkciju activate_users, preuzmete originalnu vrednost i u zavisnosti od
korisničkih dozvola uklonite.

Za staff korisnike bez dozvole change_user(), akcija activate_users se neće pojaviti upadajućem meniju akcija.

[Sadržaj](00_sadrzaj.md)
