
# Korišćenje sa standardnim admin searchom

[Sadržaj](00_sadrzaj.md)

`DjangoQL` će prepoznati ako imate definisano `search_fields` u ModelAdmin klasi.Vi ćete moći da izaberete izmedju naprednog DjangoQL search-a i standardnog Django search-a (spec. sa search_fields).

Primer:

```py
@admin.register(Book)
class BookAdmin(DjangoQLSearchMixin, admin.ModelAdmin):
    ...
    search_fields = ('title', 'author__name')
```

Za gornji primer jednan čekboks koji kontroliše search mode će se pojaviti blizu search input boksa. Ako je chekboks uključen, DjangoQL search se koristi. Takodje, tu je i opcija `djangoql_completion_enabled_by_default` (postavljena na `True` podrazumevano):

```py
@admin.register(Book)
class BookAdmin(DjangoQLSearchMixin, admin.ModelAdmin):
    search_fields = ('title', 'author__name')
    djangoql_completion_enabled_by_default = False
```

Ako ne želite dva serch moda, jednostavno uklonite `search_fields` iz ModelAdmin klase.

[Sadržaj](00_sadrzaj.md)
