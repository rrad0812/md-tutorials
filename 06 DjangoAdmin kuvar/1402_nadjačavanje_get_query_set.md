
# Nadjačavanje get_queryset

[Sadržaj](00_sadrzaj.md)

Za strane ključeve i izračunata polja nema potrebe za pokretanjem dodatnih upita po vrsti. Izvršavanje višestrukih upita za svaki objekat može dovesti do značajnog usporavanja. Možemo prepisati `get_queryset` metod u `ModelAdmin` klasi, i koristiti `select_releated` metod na poljima stranih ključeva:

```py
class MyModelAdmin(ModelAdmin):

    def get_queryset(self, request):
        queryset = super().get_queryset(request)
        return queryset.select_related('foreign_key_1', 'foreign_key_2')
```

ili koristiti annotacije da smanjimo broj upita:

```py
class MyModelAdmin(ModelAdmin):
    
    def get_queryset(self, request):
        queryset = super().get_queryset(request)
        return queryset.annotate(field_count=Count('field'))
```

[Sadržaj](00_sadrzaj.md)
