
# Označavanje polja kao readonly

[Sadržaj](00_sadrzaj.md)

UMSRA je odlučila da privremeno zaustavi praćenje drveta familija mitoloških entiteta. Od vas je traženo da napravite father, mother i spouse polja readonly.

> [!Note]
>
> **Readonly** polja se mogu postaviti samo ona koja su na `listview` strani, tj., ne možete proglasiti polje `readonly` ako se ono ne prikazuje na `listview` strani.

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    readonly_fields = ["father", "mother", "spouse"]
```

[Sadržaj](00_sadrzaj.md)
