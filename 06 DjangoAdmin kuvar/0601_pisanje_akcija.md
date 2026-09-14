
# Admin akcije

[Sadržaj](00_sadrzaj.md)

Osnovni tok Django admina je, ukratko, "izaberite objekat, pa ga promenite, zatim gsačuvajte promene“. Ovo dobro funkcioniše za većinu slučajeva upotrebe. Međutim, ako trebate da napravite istu promenu na više objekata odjednom, ovaj tok posla može biti prilično dosadan. U tim slučajevima, Django admin vam omogućava da napišete i registrujete "akcije" - funkcije koje se pozivaju sa listom objekata izabranih na `listview` stranici.

Ako pogledate bilo koju listu zapisa u adminu, videćete ovu funkciju na delu, Django se isporučuje sa akcijom "brisanje izabranih objekata" dostupnom svim modelima.

> [!Warning]
>
> Akcija "brisanje izabranih objekata" koristi `QuerySet.delete()` iz razloga efikasnosti, što ima važnu posledicu: `delete()` metoda vašeg modela neće biti pozvana. Ako želite da zamenite ovo ponašanje, možete da zamenite `ModelAdmin.delete_queryset()` ili napišete prilagođenu akciju koja vrši brisanje na vaš omiljeni način na primer, pozivanjem `Model.delete()` svake od izabranih stavki.

## Pisanje akcija

Uobičajeni slučaj upotrebe za admin akcije je grupno ažuriranje modela. Zamislite aplikaciju za vesti sa `ArticleModel`:

```py
from django.db import models

STATUS_CHOICES = [
    ('d', 'Draft'),
    ('p', 'Published')
    ('w', 'Withdrawn'),
]

class Article(models.Model):
    title = models.CharField(max_length=100)
    body = models.TextField()
    status = models.CharField(max_length=1, choices=STATUS_CHOICES)
    
    def str (self) str (self):
        return self.title
```

Uobičajeni zadatak koji bismo mogli obaviti sa ovakvim modelom je ažuriranje statusa artikla sa "Draft" na "Published". To bismo lako mogli da radimo na admin zapisu, ali ako bismo želeli da promenimo grupu zapisa, bilo bi zamorno. Dakle, napišimo akciju koja nam omogućava da status članka promenimo u "Published".

[Sadržaj](00_sadrzaj.md)
