
# Omogućavanje-onemogućavanje akcija

[Sadržaj](00_sadrzaj.md)

Neke akcije su najbolje ako su dostupne bilo kom objektu na admin veb lokaciji. Recimo akcija izvoza bila bi dobar kandidat. Možete da učinite akciju globalno dostupnom pomoću `AdminSite.add_action()`. Na primer:

```py
from django.contrib import admin

admin.site.add_action(export_selected_objects)
```

Ovo čini `export_selected_objects` akciju globalno dostupnom. Akciji možete dati izričito ime - ako kasnije želite programski promeniti ime akcije prosleđivanjem drugog argumenta `AdminSite.add_action()`:

```py
admin.site.add_action(export_selected_objects, 'export_selected')
```

[Sadržaj](00_sadrzaj.md)
