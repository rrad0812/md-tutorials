
# Elminisanje COUNT upita

[Sadržaj](00_sadrzaj.md)

Posle dostizanja odredjene veličine, cena `COUNT(*)` upita postaje značajna na `change_list` i izaziva timeout greške. MySQL je naš go-to izbor i mi koristimo `Django-MySQL` paket da proširimo Djangovu ugradjenu podršku za MySQL. Paket dolazi sa korisnom `count_tries_approx` metodom koja vraća aproksimativni broj umesto tačnog broja zapisa. Na taj način eliminišemo puni sken na InnoDB i postižemo značjno brži rezultat.

```py
class MyModelAdmin(ModelAdmin):

    def get_queryset(self, request):
        qs = super().get_queryset(request)
        return qs.count_tries_approx()
```

Pre:

```sql
SELECT COUNT(*) AS ` count` FROM `mytable`
```

Posle: `count_tries_approx`:

```sql
SELECT TABLE_ROWS
  FROM INFORMATION_SCHEMA.TABLES
  WHERE TABLE_SCHEMA = DATABASE() AND TABLE_NAME = 'mytable'
```

[Sadržaj](00_sadrzaj.md)
