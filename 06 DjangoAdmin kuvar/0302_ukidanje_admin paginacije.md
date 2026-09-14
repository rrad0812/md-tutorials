
# Ukidanje paginacije na list view strani

[Sadržaj](00_sadrzaj.md)

Ako želite da ukinete paginaciju u potpunosti na `listview` strani:

```py
import sys
...
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    list_per_page = sys.maxsize
```

[Sadržaj](00_sadrzaj.md)
