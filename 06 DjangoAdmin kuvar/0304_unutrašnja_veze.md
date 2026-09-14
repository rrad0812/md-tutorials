
# Unutrašnja ManyToMany ili ManyToOne veza

[Sadržaj](00_sadrzaj.md)

Za Hero model možete pratiti njihove roditelje koristeći:

```py
father = models.ForeignKey( "self", related_name="children", null=True, blank=True, on_delete=models.SET_NULL
)
```

Od vas je zatraženo da pokažete decu svakog Hero na `listview` strani. Hero objekti imaju unutrašnju vezu, ali to ne možete dodati u `list_display`. Morate dodati atribut u ModelAdmin i koristiti ga u `list_display`.

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    def children_display(self, obj):
        return ", ".join([child.name for child in obj.children.all()])

    children_display.short_description = "Children"
```

[Sadržaj](00_sadrzaj.md)
