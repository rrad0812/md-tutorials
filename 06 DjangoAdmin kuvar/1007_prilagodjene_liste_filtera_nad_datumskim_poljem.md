
# Prilagodjena lista filtera nad datumskim poljem

[Sadržaj](00_sadrzaj.md)

Možemo pisati prilagodjene filtere tako da možemo postaviti izračunata polja i dodati filtere nad njima.

```py
class CenturyFilter(admin.SimpleListFilter):
    title = 'century'
    parameter_name = 'published_date'

    def lookups(self, request, model_admin):
        return ((21, '21st century'), (20, '20th century'),)

    def queryset(self, request, queryset):
        value = self.value()
    
    if not value:
        return queryset

    start = (int(value) -1)*100
    end = start + 99
    return queryset.filter(published_date year gte=start,
```

```py
published_date_year__lte=end)
```

[Sadržaj](00_sadrzaj.md)
