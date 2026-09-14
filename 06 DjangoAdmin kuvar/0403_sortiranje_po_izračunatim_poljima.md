
# Sortiranje po izračunatim poljima

[Sadržaj](00_sadrzaj.md)

Django dodaje mogućnosti sortiranja na poljima koja su atributi na modelima. Kada dodate izračunato polje, Django ne zna kako da izvrši `order_by`, pa ne dodaje mogućnost sortiranja na tom polju. Ako želite dodati sortiranje na izračunato polje, morate reći Django-u šta da prosledi u `order_by`. To možete učiniti postavljanjem atributa `admin_order_field` na metodi izračunatog polja.

Polaziš od admina kojeg si napisao u prethodnom poglavlju i dodaš:

```py
hero_count.admin_order_field = '_hero_count'
villain_count.admin_order_field = '_villain_count'
```

Sa ovim promenama `AdminModel` postaje:

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
        return

    hero_count.admin_order_field = '_hero_count'
    villain_count.admin_order_field = '_villain_count'
```

Sada možemo sortirati i po izračunatim poljima (`hero_count`).

[Sadržaj](00_sadrzaj.md)
