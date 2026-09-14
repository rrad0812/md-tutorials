
# Prilagodjene admin akcije sa prolaznom stranom i formom

[Sadržaj](00_sadrzaj.md)

Da biste dodali prolaznu stranu, nastavićete sa gornjim kodom, ali umesto da odmah izvršite ažuriranje, tretiraćete ga kao prikaz i vratiti HTTP response koji uključuje novi obrazac i kontekst. Da bi počeli vratimo šablon sa praznim kontekstom:

`admin.py`

```py
from django.shortcuts import render
class OrderAdmin(admin.ModelAdmin):
    actions = ['update_status']
    
    def update_status(self, request, queryset):
        return render(request, 'admin/order_intermediate.html', context={})

        update_status.short_description = "Update status"
```

Da zadržimo admin look & feel proširimo djangov admin osnovni šablon:

`admin/order_intermediate.html`

```html
{% extends "admin/base_site.html" %}
{% block content %}
Are you sure you want to execute this action?
{% endblock %}
```

Pokušajte sa ovim kodom i bičete odvedeni na prolaznu stranu koja vas pita da li želite da izvršite izabranu akciju. Medjutim ne želite opciju koja ne radi ništa. Kako to da poboljšamo?

[Sadržaj](00_sadrzaj.md)

## Dodavanje forme da izvrši akciju

Prvo dodajmo jednostavnu formu na naš šablon. Koristimo django form iz našeg konteksta, za sada pišemo kod ručno.Tri su važne form osobine koje morate da uključite:

1. `action key`, koji bi trebao da mapira na prilagodjenu akciju,
2. `apply key`, koji može biti bilo šta. Ovo je za hvatanje našeg submission kasnije u pogledu.
3. Izabrane stavke, u formi `selected_action` koji bi trebalo da su lista `pk` izabranih stavki.

Sada naš šablon izgleda ovako:

`'admin/order_intermediate.html'`

```html
{% extends "admin/base_site.html" %}
{% block content %}

<form action="" method="post">{% csrf_token %}
<p>
Are you sure you want to execute this action on the selected items?
</p>
{% for order in orders %}
<p>
{{ order }}
</p>
<input type="hidden" name="_selected_action" value="{{ order.pk }}" />
{% endfor %}
<input type="hidden" name="action" value="update_status" />
<input type="submit" name="apply" value="Update status"/>

{% endblock %}
```

Primetite da smo dodali `hidden` polja za `_selected_action` vrednosti kao i `action` property. Sada za hvatanje form submit-a, u našoj admin akciji:

`admin.py`

```py
from django.shortcuts import render
from django.http import HttpResponseRedirect
class OrderAdmin(admin.ModelAdmin):

actions = ['update_status']
def update_status(self, request, queryset):

    """All requests here will actually be of type POST so we will need to check 
       for our special key 'apply' rather than the actual request type"""

    if 'apply' in request.POST:
        # The user clicked submit on the intermediate form.
        # Perform our update action:
        queryset.update(status='NEW_STATUS')
        # Redirect to our admin view after our update has
        # completed with a nice little info message
        # saying our models have been updated:
        self.message_user(request,
            "Changed status on {} orders".format(queryset.count()))

        return HttpResponseRedirect(request.get_full_path())

    return render(request, 'admin/order_intermediate.html',
        context={'orders':queryset})
```

[Sadržaj](00_sadrzaj.md)
