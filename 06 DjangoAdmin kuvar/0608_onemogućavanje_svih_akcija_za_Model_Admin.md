
# Onemogućavanje svih akcija za ModelAdmin

[Sadržaj](00_sadrzaj.md)

Ako želite da nema dozvoljenih akcija na raspolaganju za dati ModelAdmin, postavite ModelAdmin.actions na None:

```py
class MyModelAdmin(admin.ModelAdmin):
    actions = None
```

Ovo govori da ModelAdmin ne prikazuje ili dozvoljavaja bilo kakve akcije korisnicima, uključujući bilo koje akcije na celoj lokaciji.

[Sadržaj](00_sadrzaj.md)
