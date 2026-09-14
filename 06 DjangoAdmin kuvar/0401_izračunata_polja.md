
# Izračunata polja na listview strani

[Sadržaj](00_sadrzaj.md)

Imate admin za Origin model:

```py
@admin.register(Origin)
class OriginAdmin(admin.ModelAdmin):
    list_display = ("name",)
```

Pored imena, takođe želimo da prikažemo broj `heroes` i broj `vilians` za svaki origin, što nije DB polje na Origin modelu. To možete učiniti na dva načina.

Dodavanjem metoda na model:

```py
def hero_count(self,):
    return self.hero_set.count()

def villain_count(self):
    return self.villain_set.count()
```

I promeneite  `list_display` na:

```py
list_display = ("name", "hero_count", "villain_count")
```

Dodavanjem metoda na ModelAdmin:

```py
def hero_count(self, obj):
    return obj.hero_set.count()

def villain_count(self, obj):
    return obj.villain_set.count()
```

sada `list_display`, izgleada ovako:

```py
list_display = ("name", "hero_count", "villain_count")
```

Bilo koji metod da primenimo, pokrenuće dva dodatna upita po objektu ( slogu ).

[Sadržaj](00_sadrzaj.md)
