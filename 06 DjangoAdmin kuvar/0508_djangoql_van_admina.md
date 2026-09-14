
# DjangoQL van admina

[Sadržaj](00_sadrzaj.md)

Da. Možete dodati DjangoQL search funkcionalnost na bilo koji Django model korišćenjem DjangoQLQuerySet:

```py
from django.db import models
from djangoql.queryset import DjangoQLQuerySet

class Book(models.Model):
    name = models.CharField(max_length=255)
    author = models.ForeignKey('auth.User')
    objects = DjangoQLQuerySet.as_manager()
```

Sa gornjim primerom možemo raditi pretraživanja kao:

```py
qs = Book.objects.djangoql(
    'name ~ "war" and author.last_name = "Tolstoy"'
)
```

Vraća normalni queryset, možete ga proširiti i ponovo koristiti ako je neophodno.

```py
print(qs.count())
```

Alternativno možete dodati DjangoQL search na svaki queryset, čak iako nije instanca `DjangoQLQuerySet`:

```py
from django.contrib.auth.models import User from djangoql.queryset import apply_search
qs = User.objects.all()
qs = apply_search(qs, 'groups = None') print(qs.exists())
```

Šema može bti spec. bilo kao `queryset` opcija, ili prosledjena `djangoql()` queryset metod direktno:

```py
class BookQuerySet(DjangoQLQuerySet):
    djangoql_schema = BookSchema

class Book(models.Model):
    ...
    objects = BookQuerySet.as_manager()
```

Sada, "Book.objects.djangoql()" će koristiti BookSchema po default-u:

```py
Book.objects.djangoql('name ~ "Peace") uses BookSchema
```

Prepisivanje default queryset šeme sa `AnotherSchema`:

```py
Book.objects.djangoql('name ~ "Peace", schema=AnotherSchema)
```

Možete obezbediti šemu kao opciju za `apply_search()`

```py
qs = User.objects.all()
Book.objects.djangoql('name ~ "Peace", schema=AnotherSchema)
```

[Sadržaj](00_sadrzaj.md)
