
# Django admin i dozvole

[Sadržaj](00_sadrzaj.md)

Django admin ima vrlo tesnu integraciju sa ugrađenim sistemom za potvrdu identiteta, aposebno dozvole za model. Django admin primenjuje dozvole za model:

- Ako korisnik nema dozvole za model, neće moći da ga vidi ili da mu pristupi u adminu.
- Ako korisnik ima dozvolu za pregled i promenu na modelu, tada će moći da pregleda i ažurira instance, ali neće moći da dodaje nove instance ili briše postojeće.

Sa odgovarajućim dozvolama, admin korisnici će manje grešiti, a uljezi će teže naneti štetu.

## Primenite prilagođene poslovne uloge u Django Admin

[Sadržaj](00_sadrzaj.md)

Jedno od najranjivijih mesta u svakoj aplikaciji je sistem za potvrdu identiteta. U Django app ovo je `User` model. Dakle, da biste bolje zaštitili svoju aplikaciju, krenućete sa `User` modelom.

Prvo treba da preuzmete kontrolu nad admin stranicom `User` modela. Django već ima vrlo lepu admin stranicu za upravljanje korisnicima. Da biste iskoristili taj sjajni posao, proširićete ugrađeni `User` model za admin korisnika.

## Postavka prilagođenog User admina

[Sadržaj](00_sadrzaj.md)

Da biste obezbedili prilagođenog admina za `User` model, morate da opozovete registraciju postojećeg `User` modela koji nudi Django i da registrujete svoj:

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

# Deregistrujte osnovni user model
admin.site.unregister(User)

class CustomUserAdmin(UserAdmin):
    pass

# Registrujte svoj prilagodjeni user model admin:
@admin.register(User)
```

Vaš `CustomUserAdmin` je prošireni Djangov `UserAdmin`. Ako se sada prijavite na Django admin <http://127.0.0.1:8000/admin/auth/user>, videćete nepromenjen `User` admin. Proširenjem `UserAdmin`-a, možete da koristite sve ugradjene mogućnosti koje pruža Django admin.

[Sadržaj](00_sadrzaj.md)
