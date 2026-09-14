
# Editovanje višestrukih modela

[Sadržaj](00_sadrzaj.md)

Ukratko, treba da koristite `inlines`. Imate "Category" model, i potrebno je da dodajete i editujete "Villain" modele unutar "Category" admina:

```py
class VillainInline(admin.StackedInline):
    model = Villain

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):
    ...
    inlines = [VillainInline]
```

Vidite da je forma za dodavanje i editovanje "Villain" unutar "Category" admina. Ako `Inline` model ima više polja, koristite `StackedInline` inače koristite `TabularInline`.

[Sadržaj](00_sadrzaj.md)
