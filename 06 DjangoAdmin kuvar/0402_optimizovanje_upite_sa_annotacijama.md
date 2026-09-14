
# Optimizovanje upite sa annotacijama

[Sadržaj](00_sadrzaj.md)

Ako imate više izračunatih polja u adminu, admin može postati spor. Uzmimo primer sa modela Origin:

```py
@admin.register(Origin)
class OriginAdmin(admin.ModelAdmin):
    list_display = ("name", "hero_count", "villain_count")
    
    def hero_count(self, obj)
        return obj.hero_set.count()
    
    def villain_count(self, obj):
        return obj.villain_set.count()
```

Da bi popravili performase nadjačajmo metod `get_queryset` da obeležimo ( annotate ) prebrojane zapise. Sa ovakvom promenom ModelAdmin izgleda ovako:

```py
@admin.register(Origin)
class OriginAdmin(admin.ModelAdmin):
    list_display = ("name", "hero_count", "villain_count")
    
    def get_queryset(self, request):
        queryset = super().get_queryset(request)
        queryset = queryset.annotate(
            _hero_count=Count("hero", distinct=True),
            _villain_count=Count("villain", distinct=True),
        )
        return queryset

    def hero_count(self, obj):
        return obj._hero_count

    def villain_count(self, obj):
        return bj._villain_count
```

Sada nema viška upita. Admin sada izgleda kao i pre obeležavanja.

[Sadržaj](00_sadrzaj.md)
