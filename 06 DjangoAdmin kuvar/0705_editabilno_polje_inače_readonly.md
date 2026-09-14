
# Editabilno polje kada se objekat kreira, inače readonly

[Sadržaj](00_sadrzaj.md)

Potrebno je, na primer, da polja "name" i "category" budu readonly pošto je objekat kreiran. Međutim, prilikom prvog upisivanja polja moraju se uređivati. To možete napraviti nadjačavanjem metode `get_readonly_fields`:

```py
def get_readonly_fields(self, request, obj=None):
    if obj:
        return ["name", "category"] else:
    return []
```

`obj` je `None` za vreme kreiranja, dok je za vreme editovanja postavljen.

[Sadržaj](00_sadrzaj.md)
