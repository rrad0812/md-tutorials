
# Upravljanje velikim FK modelom

[Sadržaj](00_sadrzaj.md)

Možete kreirati veliki broj kategorija ovako:

```py
categories = [Category(**{"name": "cat-{}".format(i)}) for i in range(100000)]
Category.objects.bulk_create(categories)
```

Sada "Category" model ima više od 100000 objekata, kada odete na "Hero" admin, imaćete padajuću listu sa 100000 category izbora. To će napraviti stranicu sporom i padajuću listu teškom za korišćenje.

Možete promeniti "Hero" admin tako da za polje category uvedete `raw_id_fields` listu:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    raw_id_fields = ["category"]
```

[Sadržaj](00_sadrzaj.md)
