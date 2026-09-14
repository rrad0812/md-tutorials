
# on’ ili ’off’ ikone umesto izračunatih boolean polja

[Sadržaj](00_sadrzaj.md)

Prethodno smo dodali izračunato boolean polje:

```py
return obj.benevolence_factor > 75
```

`is_very_benevolent` polje prikazuje string `True` ili `False`, za razliku od ugradjenih `BooleanFields` koji prikazuju `on` ili `off` indikator. Da bi to popravili, dodajmo boolean atribut našem metodu. ModelAdmin sada izgleda ovako:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin):
    list_display = ("name", "is_immortal", "category", "origin", "is_very_benevolent")
    list_filter = ("is_immortal", "category", "origin", IsVeryBenevolentFilter)

    def is_very_benevolent(self, obj):
        return obj.benevolence_factor > 75
    
    is_very_benevolent.boolean = True
```

[Sadržaj](00_sadrzaj.md)
