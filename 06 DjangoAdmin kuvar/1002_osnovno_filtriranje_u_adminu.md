
# Osnovno filtriranje u adminu

[Sadržaj](00_sadrzaj.md)

Recimo da bismo želeli da filtriramo našu prodavnicu u admin po korisniku. Da biste to postigli samo dodajte `list_filter` za "ShopAdmin" klasu (u admin.py ) i odredite oblast na kojima može da filtrira.

```py
list_filter = ('user', )
```

Pun kod:

```py
class ShopAdmin(admin.ModelAdmin):
    fieldsets = [
        (None, {'fields': ['user','name','email','address']}),
    ]
    list_display = ('user','name', 'email', )
    list_filter = ('user',)
    ordering = ('user',)
```

A sada će se u gornjem desnom uglu ekrana pojaviti opcija filtriranja. Još bolje, recimo da bismo želeli da filtriramo samo za korisnike koji imaju prodavnicu. Ovaj put ćemo napisati:

```py
list_filter = (('user', admin.RelatedOnlyFieldListFilter),)
```

Sada opcija filtriranja sadrži samo jednog korisnika, jer drugi korisnici nemaju prodavnice.

[Sadržaj](00_sadrzaj.md)
