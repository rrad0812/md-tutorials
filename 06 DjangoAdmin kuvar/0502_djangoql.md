
# DjangoQL

[Sadržaj](00_sadrzaj.md)

Napredni jezik Django pretrage sa autokompletiranjem. Podržava logičke operatore zagrade, veze, i radi sa Django modelom.

## Instalacija

```shell
pip install djangoql
```

Dodaj `djangoql` u `INSTALLED_APPS` listu u `settings.py`:

```py
INSTALLED_APPS = [
    ...
    'djangoql',
    ...
]
```

Dodavanje `DjangoQLSearchMixin` u model admin će zameniti standardnu `search` funkcionalnost sa DjangoQL search-om.

Primer:

```py
from django.contrib import admin
from djangoql.admin import DjangoQLSearchMixin
from .models import Book
@admin.register(Book)
class BookAdmin(DjangoQLSearchMixin, admin.ModelAdmin):
    pass
```

[Sadržaj](00_sadrzaj.md)
