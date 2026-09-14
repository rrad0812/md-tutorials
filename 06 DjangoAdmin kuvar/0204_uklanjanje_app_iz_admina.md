
# Uklanjanje aplikacija iz admina

[Sadržaj](00_sadrzaj.md)

Django će uključiti `django.contrib.auth` u `INSTALLED_APPS`, to znači da će `User` i `Groups` modeli da budu uključeni u admin sajt automatski.

Ako želite možete ih ukloniti:

```py
from django.contrib.auth.models import User, Group

admin.site.unregister(User)
admin.site.unregister(Group)
```

[Sadržaj](00_sadrzaj.md)
