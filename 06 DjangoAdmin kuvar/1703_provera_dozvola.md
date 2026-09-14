
# Proveriti dozvole

[Sadržaj](00_sadrzaj.md)

Dozvole na modelu se odobravaju korisnicima ili grupama. Da proverite da li odredjeni korisnik ima dozvolu, možete uraditi sledeće:

```shell
>>> from django.contrib.auth.models import User
>>> u = User.objects.create_user(username='haki')
>>> u.has_perm('auth.change_user')
False
```

`has_perm()` će uvek vratiti `True` za aktivnog superuser-a, čak iako dozola zaista ne postoji:

```shell
>>> from django.contrib.auth.models import User
>>> superuser = User.objects.create_superuser(
... username='superhaki',
... email='me@hakibenita.com',
... password='secret',
... )
>>> superuser.has_perm('does.not.exist')
True
```

Kada proverate dozovole za `superusera`, one se u stvarnosti ne proveravaju, sistem uvek vraća `True`.

[Sadržaj](00_sadrzaj.md)
