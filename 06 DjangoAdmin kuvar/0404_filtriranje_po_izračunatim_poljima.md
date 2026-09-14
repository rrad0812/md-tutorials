
# Kako filtrirati po izračunatim poljima

[Sadržaj](00_sadrzaj.md)

Imate Hero admin koji izgleda ovako:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin):
    list_display = ("name", "is_immortal", "category", "origin", "is_very_benevolent")
    list_filter = ("is_immortal", "category", "origin",)
    
    def is_very_benevolent(self, obj):
        return obj.benevolence_factor > 75
```

Želimo da filtriramo i po izračunatim poljima. Da to uradimo, potrebno je da nasledimo `SimpleListFilter` klasu:

```py
class IsVeryBenevolentFilter(admin.SimpleListFilter):
    title = 'is_very_benevolent'
    parameter_name = 'is_very_benevolent'
    
    def lookups(self, request, model_admin):
        return (('Yes', 'Yes'),('No', 'No'),)

    def queryset(self, request, queryset): value = self.value()
        if value == 'Yes':
            return queryset.filter(benevolence_factor gt=75)
        elif value == 'No':
            return queryset.exclude(benevolence_factor lte=75)
        
        return queryset
```

I promenimo list_filter na:

```py
list_filter = ("is_immortal", "category", "origin", IsVeryBenevolentFilter)
```

[Sadržaj](00_sadrzaj.md)
