
# Modeli za optimizaciju queryset upita

[Sadržaj](00_sadrzaj.md)

Kada db ima strane ključeve, korišćenjem `select_related()` i `prefetch_related()` možemo smanjiti broj db zahteva i poboljšati performanse.

**Opis primera**:

`models.py:`

```py
from django.db import models
class Province(models.Model):
    name = models.CharField(max_length=10)

    def unicode (self):
        return self.name

class City(models.Model):
    name = models.CharField(max_length=5)
    province = models.ForeignKey(Province)

    def unicode (self):
        return self.name

class Person(models.Model):
    firstname = models.CharField(max_length=10)
    lastname = models.CharField(max_length=10)
    visitation = models.ManyToManyField(City, related_name = "visitor")
    hometown = models.ForeignKey(City, related_name = "birth")
    living = models.ForeignKey(City, related_name = "citizen")
```

> [!Note]
> Kreirana aplikacija se zove "QSOptimize"
>
> [!Note]
> Zbog jednostavnosti, imamo dva podatka u tabeli "qsoptimize_province":
>
> - Hubei Province i Guangdong Province,
>
> i tri podatka u tabeli "qsoptimize_city":
>
> - Wuhan City, Shiyan City i Guangzhou City.

[Sadržaj](00_sadrzaj.md)
