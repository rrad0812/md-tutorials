
# Sprečavanje ažuriranja polja User modela

[Sadržaj](00_sadrzaj.md)

Admin forme bez nadzora glavni su kandidat za užasne greške. Korisnik koji pripada staff-u može lako ažurirati instancu User modela putem admina na način koji aplikacijane očekuje. U većini slučajeva korisnik neće ni primetiti da nešto nije u redu. Takve greške je obično vrlo teško pronaći i otkloniti.

Da biste sprečili da se takve greške dogode, možete sprečiti admine da menjaju određena polja u User modelu. Ako želite da sprečite bilo kog korisnika, uključujući superuser-a, da ažurira polje,možete ga označiti samo za čitanje. Na primer, polje date_joined se postavlja kada se korisnik registruje. Korisnik ove informacije nikada ne sme da menja, pa ćete ih označiti kao samo za čitanje:

```py
from django.contrib import admin
from django.contrib.auth.models import User
from django.contrib.auth.admin import UserAdmin

@admin.register(User)
class CustomUserAdmin(UserAdmin):
    readonly_fields = [
        'date_joined',
    ]
```

Kada se polje doda u readonly_fields, ono se neće moći uređivati u podrazumevanoj admin formi. Kada je polje označeno samo za čitanje, Django će prikazati ulazni element kao onemogućen.

Ali, šta ako želite da sprečite samo neke korisnike da ažuriraju polje User modela?

[Sadržaj](00_sadrzaj.md)
