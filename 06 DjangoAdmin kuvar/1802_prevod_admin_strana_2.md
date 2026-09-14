
# Prevod admin strana II

[Sadržaj](00_sadrzaj.md)

Koraci ispod prikazuju kako da dodate prevode posebno za admina. Ako ne postoji posebna potreba, koristite lenji prevod za sve slučajeve.

## Gotove fraze

[Sadržaj](00_sadrzaj.md)

Admin stranice sajta automatski se generišu pomoću šablona sa puno konzerviranih fraza za stvari poput "login" i "save" i "delete". Kako da njih prevedete? Srećom, Django već ima prevode za mnoge glavne jezike.

Pogledajte listu na:

```html
django/contrib/admin/locale
```

za dostupne jezike. Django će automatski koristiti prevode za ove jezike na veb lokaciji Admin - nema šta drugo što treba da uradite! Ako vam je potreban jezik koji nije dostupan, snažno vas ohrabrujem da doprinesete novim prevodima na projektu Django tako da ih svi mogu deliti.

## Prilagođeni admin naslovi

[Sadržaj](00_sadrzaj.md)

Postoji nekoliko načina za postavljanje prilagođenih naslova Admin stranice. Moja poželjna metoda je da ih postavim u root `urls.py` datoteku. Gde god da su postavljeni, označite ih za lenji prevod. Lako ih je prevideti!

```py
from django.contrib import admin
from django.utils.translation import gettext_lazy as _

admin.site.index_title = _('My Index Title')
admin.site.site_header = _('My Site Administration')
admin.site.site_title = _('My Site Management')
```

## Imena aplikacija

[Sadržaj](00_sadrzaj.md)

Imena aplikacija su još jedan skup fraza koji se mogu lako propustiti. Dodajte polje `verbose_name` sa prevodnim stringom na svaku klasu `AppConfig` u projektu. Nemojte jednostavno pokušati prevesti string za polje `name`: Django će dati runtime izuzetak!

```py
from django.apps import AppConfig
from django.utils.translation import gettext_lazy as _

class CustomersConfig(AppConfig):
  name = 'customers'
  verbose_name = _('Customers')
  ```

## Imena modela

[Sadržaj](00_sadrzaj.md)

Modeli su puni stringova kojima su potrebni prevodi. Evo stvari koje treba potražiti:

- Dajte svakom polju `verbose_name` vrednost, jer se identifikatori ne mogu prevesti.
- Označite tekstove pomoći, opise izbora i dodatne poruke kao prevodne.
- Dodajte `Meta class` sa `verbose_name`, `verbose_name_plural` vrednostima.
- Pazite na bilo koje druge stringove kojima bi mogao trebati prevod.

Evo primera modela:

```py
from django.db import models
from django.core.validators import RegexValidator
from django.utils.translation import gettext_lazy as

class Customer(models.Model):
  name = models.CharField(
    max_length=100,
    help_text=_('First and last name.'),
    verbose_name=_('name')
  )
  address =
    models.CharField( max_le
    ngth=100,
    verbose_name=_('address')
  )
  phone =
    models.CharField(max
    _length=10,
    validators=[
      RegexValidator('^\d{10}$', _('Phone must be exactly 10 digits.'))
    ],
    verbose_name=_('phone number')
  )

  class Meta:
    verbose_name = _('customer')
    verbose_name_plural = _('customers')
```

## Pokreni komande

[Sadržaj](00_sadrzaj.md)

Kada su svi stringovi markirani za prevodjenje, generiši datoteke poruka:

```shell
python manage.py makemessages -lzh_Hans # pojednostavljeni kineski
```

Zatim kompajliraj poruke

```shell
python manage.py compilemessages
```

> [!Warning]
>
> Jezički kod i naziv lokala mogu biti različiti! Na primer, uzmite pojednostavljeni kineski: Kod jezika je "ZH-Hans", ali ime lokala je "zh_hans". Primetite podvlaku i velika slova.
>
> Imena lokala često uključuju šifru zemlje i različite jezičke nijanse, poput američkog engleskog vs. britanskog engleskog. Pogledajte django/contrib/admin/local za listu primera.

[Sadržaj](00_sadrzaj.md)
