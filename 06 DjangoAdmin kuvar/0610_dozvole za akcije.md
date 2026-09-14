
# Dozvola za akcije

[Sadržaj](00_sadrzaj.md)

Akcije mogu biti ograničene na korisnike sa određenim dozvolama, umotavanjem funkcije akcije u `action()` dekorater i prosleđivanjem `permissions` argumenta:

```py
@admin.action(permissions=['change'])
def make_published(modeladmin, request, queryset):
    queryset.update(status='p')
```

`make_published()` akcija će biti dostupna samo korisnicima koji prođu `ModelAdmin.has_change_permission()` proveru. Ako permissions ima više dozvola, akcija će biti dostupna sve dok korisnik prođe bar jednu proveru.

Dostupne vrednosti za `permissions` i odgovarajuće provere metoda su:

- `'add'`: `ModelAdmin.has_add_permission()`
- `'change'`: `ModelAdmin.has_change_permission()`
- `'delete'`: `ModelAdmin.has_delete_permission()`
- `'view'`: `ModelAdmin.has_view_permission()`

Možete odrediti bilo koju drugu vrednost sve dok primenite odgovarajući metod na:

```py
ModelAdmin.has_<value>_permission(self, request)
```

Na primer:

```py
from django.contrib import admin
from django.contrib.auth import get_permission_codename

class ArticleAdmin(admin.ModelAdmin):actions = ['make_published']
    @admin.action(permissions=['publish'])
    def make_published(self, request, queryset):
        queryset.update(status='p')
    
    def has_publish_permission(self, request):
        """Does the user have the publish permission?"""
        opts = self.opts
        codename = get_permission_codename('publish', opts)
        return request.user.has_perm('%s.%s' % (opts.app_label, codename))
```

> [!Note]
>
> **Promenjeno u Django 3.2**: `permissions` argument u `action()` dekoratoru ekvivalentan je postavljanju `allowed_permissions` atributa akcionoj funkciji direktno u prethodnim verzijama.
>
> Direktno postavljanje atributa i dalje je podržano radi povratne kompatibilnosti.

[Sadržaj](00_sadrzaj.md)
