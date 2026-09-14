
# Kako filtrirati FK vrednosti

[Sadržaj](00_sadrzaj.md)

"Hero" model ima `ForeignKey` vezu na "Category". Tako se svi category objekti pokazuju u padajućoj listi za polje "category". Ako umesto toga, želimo da se pojavi poskup, Django pruža mogućnost prilagodjavanja nadjačavanjem `formfield_for_foreignkey` metoda:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == "category":
            kwargs["queryset"] = Category.objects.filter(name in=['God', 'Demi  
                God'])
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

[Sadržaj](00_sadrzaj.md)
