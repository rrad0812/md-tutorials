# Primena dozvole

[Sadržaj](00_sadrzaj.md)

Django modeli sami po sebi ne primenjuju dozvole. Jedino mesto gde se dozvole primenjuju podrazumevano je Django admin. Razlog zbog kojeg modeli ne primenjuju dozvole je taj što model obično nije svestan korisnika koji izvršava akciju. U Django app, korisnik se obično dobija iz `request`-a. Zbog toga se, najčešće, dozvole primenjuju na sloju prikaza.

Na primer, da biste sprečili korisnika bez dozvole da pristupi prikazu koji prikazuje korisničke informacije, uradite sledeće:

```py
from django.core.exceptions import PermissionDenied

def users_list_view(request):
    if not request.user.has_perm('auth.view_user'):
        raise PermissionDenied()
```

Ako se korisnik, koji je podneo zahtev, prijavio se i bio potvrđen identitet, tada će `request.user` biti instanca `User`, ako se korisnik nije prijavio, tada će `request.user` biti instanca `AnonimousUser`. Ovo je poseban objekat koji Django koristi za označavanje neovlašćenog korisnika.

Korišćenje `has_perm` na `AnonimousUser` uvek vraća `False`.

Ako korisnik koji pravi zahtev nema dozvolu `view_user`, tada pokrećete izuzetak `PermissionDenied` i klijentu se vraća odgovor sa statusom `403`.

Da bi olakšao primenu dozvola u prikazima, Django nudi dekorator nazvan `permission_required` koji radi istu stvar:

```py
from django.contrib.auth.decorators import permission_required

@permission_required('auth.view_user')
def users_list_view(request):
    pass
```

Da biste primenili dozvole u šablonima, trenutnim korisničkim dozvolama možete pristupiti putem posebne promenljive šablona, koja se naziva `perms`. Na primer, ako želite da prikažete dugme za brisanje samo korisnicima sa dozvolom za brisanje, učinite sledeće:

```html
{% if perms.auth.delete_user %}
  <button>Delete user!</button>
{% endif %}
```

Neke popularne nezavisne aplikacije, poput Django Rest framework-a, takođe pružaju korisnu integraciju sa dozvolama za Django model.

[Sadržaj](00_sadrzaj.md)
