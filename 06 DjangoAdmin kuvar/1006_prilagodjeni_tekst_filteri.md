
# Prilagodjeni tekst filteri

[Sadržaj](00_sadrzaj.md)

Ovde je izbor ograničen. Ali u nekim slučajevima, gde je jako puno filterskih mogućnosti, bolje je prikazati tekst ulaz umesto liste izbora. Hajde da napišemo prilagodjeni filter za filtriranje knjiga po godini publikovanja.

Napišimo ulazni filter.

```py
class PublishedYearFilter(admin.SimpleListFilter):
    title = 'published year'
    parameter_name = 'published_date'
    template = 'admin_input_filter.html'

    def lookups(self, request, model_admin):
        return ((None, None),)

    def choices(self, changelist):
        query_params = changelist.get_filters_params()
        query_params.pop(self.parameter_name, None)
        all_choice = next(super().choices(changelist))
        all_choice['query_params'] = query_params
        yield all_choice

    def queryset(self, request, queryset):
        value = self.value()
        if value:
            return queryset.filter(published_date year=value)
```

`#admin_input_filter.html`

```html
{% load i18n %}
<h3>{% blocktrans with filter_title=title %}By {{filter_title }}
{% endblocktrans %}</h3>
<ul>

<li>
{% with choices.0 as all_choice %}
<form method="GET">

<input type="text" name="{{ spec.parameter_name }}"
value="{{spec.qvalue|default_if_none:"" }}"/>

<input class="btn btn-info" type="submit"value="{% trans 'Apply' %}">
{% if not all_choice.selected %}

<button type="button" class= "btn btn-info">
<a href="{{all_choice.query_string }}">Clear</a></button>

{% endif %}
</form>

{% endwith %}
</li>
</ul>
```

[Sadržaj](00_sadrzaj.md)
