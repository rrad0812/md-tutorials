
# Povezana polja na listview strani

[Sadržaj](00_sadrzaj.md)

Django admin ima `ModelAdmin` klasu koja pruža opcije i funkcionalnosti za modele u adminu. To su `list_display`, `list_filter`, `search_fields` za specificiranje polja za odgovarajuću akciju.

`search_fields`, `list_filter` i druge opcije takodje pružaju uključivanje `ForeignKey` ili `ManyToMany` polja sa lookup API notacijom. Na primer, da bi pretražili po polju "name" "BestSellerAdmin" modela, možemo specificirati "book.name" u `search_fields`.

```py
from django.contrib import admin 
from book.models import BestSeller

class BestSellerAdmin(RelatedFieldAdmin):
    search_fields = ('book name', )
    list_display = ('id', 'year', 'rank', 'book')
    
admin.site.register(Bestseller, BestsellerAdmin)
```

Primetite da je "BestSellerAdmin" klasa nasledjena iz `RelatedFieldAdmin` klase.

Django ne dozvoljava istu notaciju u `list_display` opciji. Da bi uključili `ForeignKey` polje ili `ManyToMany` polje u `list_display`, moramo pisati prilagodjenu metodu i dodati je u `list_display`.

```py
from django.contrib import admin
from book.models import BestSeller

class BestSellerAdmin(RelatedFieldAdmin):
    list_display = ('id', 'rank', 'year', 'book', 'author')
    search_fields = ('book name', )

    def author(self, obj):
        return obj.book.author
    
    author.description = 'Author'
```

Dodavanje ForeignKey na taj način u `list_display` postaje zamorno, pogotovu ako je više polja sa `ForeignKey` na povezane modele. Možemo napisati prilagodjenu klasu za dinamičko postavljanje atributa kao metode tako da možemo postaviti `ForeignKey` polja u `list_display`.

```py
def get_related_field(name, admin_order_field=None,short_description=None):
    related_names = name.split(' ')

def dynamic_attribute(obj):
    for related_name in related_names:
    obj = getattr(obj, related_name)
    return obj

dynamic_attribute.admin_order_field = admin_order_field or name
dynamic_attribute.short_description = short_description or related_names[-1].   
    title ().replace('_', ' ')

return dynamic_attribute

class RelatedFieldAdmin(admin.ModelAdmin):

    def getattr (self, attr):
        if ' ' in attr:
            return get_related_field(attr)
        return self.getattribute (attr)

class BestSellerAdmin(RelatedFieldAdmin):
    list_display = ('id', 'rank', 'year', 'book','book author')
```

Nasledjivanjem klase `RelatedFieldAdmin`, možemo direktno koristiti `ForeignKey` polja u `list_display`. Medjutim to će dovesti do N+1 problema.

<https://github.com/theatlantic/django-nested-admin>.

[Sadržaj](00_sadrzaj.md)
