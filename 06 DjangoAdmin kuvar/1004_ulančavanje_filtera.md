
# Ulančavanje filtere u adminu

[Sadržaj](00_sadrzaj.md)

Ok, idemo sada malo dalje. Recimo da bismo želeli da primenimo povezane filtere (tj. ulančane filtere). Na Product možemo dodati filtriranje korisnika

```py
list_filter=('user', ProductPriceFilter, 'refShop', )
```

Sada, ako filtriramo prema korisniku, i dalje ćemo videti sve vrednosti Shop (u Shop filteru). Moglo bi biti bolje da prikažemo samo prodavnice koje pripadaju korisniku kojeg smo upravo filtrirali. To je ono što predstavlja ulančani filter. Spisak vrednosti prikazanih u filtru zavisiće od vrednosti izabrane u drugom filtru.

Da bismo to uradili, ispitaćemo URL upita i utvrditi da li je korisnik korišćen za filtriranje rezultata, ako je tako, URL će sadržati

```html
? user_id__exact = "value"
```

Na osnovu toga možemo stvoriti prilagođeni filter koji će odabrati vrednosti samo zaizabranu vrednost korisničkog ID-a.

```py
class UserShopFilter(admin.SimpleListFilter):
    title = 'shop'
    parameter_name = 'refShop'
    
    def lookups(self, request, model_admin):
        if 'user_id__exact' in request.GET:
            id = request.GET['user_id__exact']
            shops = set([c.refShop
                for c in model_admin.model.objects.all().filter(user=id)])
        else:
            shops = set([c.refShop
                for c in model_admin.model.objects.all()])
        return [(s.id, s.name) for s in shops]

    def queryset(self, request, queryset):
        if self.value():
            return queryset.filter(refShop_id__exact = self.value())
```

Dakle, u metodu lookups, proveravamo da li imamo izabranog korisnika i da li ćemo odabrati samo prodavnice koje pripadaju tom korisniku:

```py
shops = set([c.refShop for c in model_admin.model.objects.all().filter(user=id)])
```

U suprotnom možemo odabrati sve prodavnice:

```py
shops = set([c.refShop for c in model_admin.model.objects.all()])
```

Tada možemo vratiti rezultate koji će biti prikazani u adminu:

```py
return [(s.id, s.name) for s in shops]
```

Sada ako na adminu izaberemo korisnika B, moći ćemo da vidimo samo prodavnice koje pripadaju našem korisniku B. To je zato što smo primenili filter koji zavisi od drugog filtera.

[Sadržaj](00_sadrzaj.md)
