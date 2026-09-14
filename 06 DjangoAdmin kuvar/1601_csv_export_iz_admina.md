
# CSV Export iz Django admina

[Sadržaj](00_sadrzaj.md)

Od vas je zatraženo da dodate mogućnost izvoza "Hero" i "Villain" iz admina. Postoji veliki broj nezavisnih aplikacija koje to omogućavaju, ali to je prilično lako i bez dodavanja nove zavisnosti. Dodaćete admin akciju u "HeroAdmin" i "VillanAdmin". Svaka admin akcija uvek ima isti potpis:

```py
def admin_action(ModelAdmin, request, queryset):
```

ili alternativno možete dodati metod u ModelAdmin kao:

```py
class SomeModelAdmin(admin.ModelAdmin):
    def admin_action(self, request, queryset):
```

Da dodate izvoz "HeroAdmin"-u možete napraviti sledeće:

```py
actions = ["export_as_csv"]
def export_as_csv(self, request, queryset):
    pass

export_as_csv.short_description = "Export Selected"
```

To dodaje akciju `export_as_csv`. Sada definišemo `export_as_csv`:

```py
import csv
from django.http import HttpResponse
...
def export_as_csv(self, request, queryset):
    meta = self.model._meta
    field_names = [field.name for field in meta.fields]
    response = HttpResponse(content_type='text/csv')
    response['Content-Disposition'] = 'attachment';
        filename={}.csv'.format(meta)
    writer = csv.writer(response)
    writer.writerow(field_names)
    for obj in queryset:
        row = writer.writerow([getattr(obj, field) for field in field_names])
    return response
```

Ovo izvozi sve izabrane zapise. Primetite, `export_as_csv` nema ništa specifično sa "Hero" modelom, tako da možemo ekstrahovati metod u mixin. Sa tim promenama, kod izgleda ovako:

```py
class ExportCsvMixin:
    def export_as_csv(self, request, queryset):
        meta = self.model._meta
        field_names = [field.name for field in meta.fields]
        response = HttpResponse(content_type='text/csv')
        response['Content-Disposition'] = 'attachment;
        filename={}.csv.format(meta)
        writer = csv.writer(response)
        writer.writerow(field_names)
        for obj in queryset:
            row = writer.writerow(
                [getattr(obj, field) for field in field_names])
        return response

    export_as_csv.short_description = "Export Selected"
        @admin.register(Hero)

class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    list_display = ("name", "is_immortal", "category", "origin",
        "is_very_benevolent")
    list_filter = ("is_immortal", "category", "origin",
        IsVeryBenevolentFilter)
    actions = ["export_as_csv"]
...

@admin.register(Villain)
class VillainAdmin(admin.ModelAdmin, ExportCsvMixin):
    list_display = ("name", "category", "origin")
    actions = ["export_as_csv"]
```

Možete dodati CSV izvoz i drugim modelima, nasledjujući `ExportCsvMixin`.

[Sadržaj](00_sadrzaj.md)
