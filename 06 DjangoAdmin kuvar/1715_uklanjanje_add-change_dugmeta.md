
# Uklanjanje ‘Add’/’Delete’ dugmeta

[Sadržaj](00_sadrzaj.md)

UMSRA je dodala sve "Category" i "Origin" objekte i želi da onemogući dalje dodavanje ili brisanje objekata. To ćemo postići nadjačavanjem `has_add_permission` i `has_delete_permission` u Django adminu:

```py
def has_add_permission(self, request):
    return False

def has_delete_permission(self, request, obj=None):
    return False
```

Primetite uklanjanje `add` dugmeta sa `listview` strane. `Add` i `delete` dugmad će takodje biti uklonjena sa `changeview` strane.

[Sadržaj](00_sadrzaj.md)
