
# Dozvola za kreiranje samo jednog objekta

[Sadržaj](00_sadrzaj.md)

UMSRA traži da ograničite broj Category objekata na 1. Oni žele da svaki entitet bude iste kategorije:

```py
MAX_OBJECTS = 1
def has_add_permission(self, request):
    if self.model.objects.count() >= MAX_OBJECTS:
        return False
    return super().has_add_permission(request)
```

Ovo će sakriti add dugme sve dok je ispunjen uslov. Možete postaviti MAX_OBJECTS na bilo koju vrednost.

[Sadržaj](00_sadrzaj.md)
