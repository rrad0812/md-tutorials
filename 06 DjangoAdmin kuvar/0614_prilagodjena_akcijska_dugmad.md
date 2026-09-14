# Prilagodjena akcijska dugmad

[Sadržaj](00_sadrzaj.md)

UMSRA je odlučila da su, ukoliko imaju dovoljno kriptonita, svi Heroji su smrtni. Međutim, žele da mogu da se predomisle i kažu da su svi heroji besmrtni. Od vas je zatraženo da dodate dva dugmeta - jedno koje sve heroje čini smrtnim i jedno koje čini sve besmrtnim.

Budući da utiče na sve junake, bez obzira na odabir, ovo mora biti zasebno dugme, a ne padajući meni akcije. Prvo ćemo promeniti šablon na HeroAdmin-u kako bismo mogli da dodamo dva dugmeta:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    change_list_template = "entities/heroes_changelist.html"
```

Tada ćemo nadjačati `get_urls`, i dodati `set_immortal` i `set_mortal` metode na ModelAdmin.

```py
def get_urls(self):
    urls = super().get_urls()
    my_urls = [
        path('immortal/', self.set_immortal),
        path('mortal/', self.set_mortal),
    ]
    return my_urls + urls

def set_immortal(self, request):
    self.model.objects.all().update(is_immortal=True)
    self.message_user(request, "All heroes are now immortal"
    return HttpResponseRedirect("../")

def set_mortal(self, request):
    self.model.objects.all().update(is_immortal=False)
    self.message_user(request, "All heroes are now mortal")
    return HttpResponseRedirect("../")
```

Na kraju, kreirajmo `entities/heroes_changelist.html` šablon proširenjem `admin/change_list.html`:

```html
{% extends 'admin/change_list.html' %}
{% block object-tools %}

<div>
<form action="immortal/" method="POST"> {% csrf_token %}

<button type="submit">Make Immortal</button>
</form>

</div>
<br />
{{ block.super }}

{% endblock %}
```

[Sadržaj](00_sadrzaj.md)
