
# Jedan model dva puta na adminu

[Sadržaj](00_sadrzaj.md)

Možete to uraditi:

- jednom kao regularan admin, i
- drugi put kao readonly admin.

> [!Note]
>
> Neki korisnici će videti samo `read only` zbog dozvola.

Ako pokušate da registrujete isti model dva puta:

```py
admin.site.register(Hero)
admin.site.register(Hero)
```

dobićete grešku:

```py
raise AlreadyRegistered('The model %s is already registered' % model. name )
```

Rešenje je naslediti `Hero` model kao `ProxyModel`:

`models.py`

```py
class HeroProxy(Hero):
  class Meta:
    proxy = True

  ...
```

`admin.py`

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
  list_display = ("name", "is_immortal", "category", "origin", "is_very_benevolent")
  ....

@admin.register(HeroProxy)
class HeroProxyAdmin(admin.ModelAdmin):
  readonly_fields = ("name", "is_immortal", "category", "origin", ...)
```

[Sadržaj](00_sadrzaj.md)
