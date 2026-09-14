
# Događaji u inline formi

[Sadržaj](00_sadrzaj.md)

Možda ćete želeti da pokrenete neki JavaScript kada se umetnuti obrazac (zapis) doda ili ukloni u formi `changeview` strane admina. `formset:added` i `formset:removed` jQuery događaji to dozvoljavaju.

Rukovaocu događaja se prenose tri argumenta:

- `event` je jQuery događaj.
- `$row` je novododani (ili uklonjeni) red (zapis).
- `formsetName` je skup obrazaca kome red pripada.

Događaj se pokreće pomoću prostora imena `django.jQuery`.

U prilagođenom `change_form.html` šablonu proširite `admin_change_form_document_ready` blok i dodajte kod:

U `change_form.html`:

```html
{% extends 'admin/change_form.html' %}
{% load static %}
{% block admin_change_form_document_ready %}
{{ block.super }}
<script src="{% static 'app/formset_handlers.js' %}"></script>
{% endblock %}
```

U `app/static/app/formset_handlers.js`

```js
(function($) {
  $(document).on('formset:added',
    function(event, $row, formsetName){
      if (formsetName == 'author_set') {
        // Do something
      }
    }
  );
  $(document).on('formset:removed',
    function(event, $row, formsetName) {
      // Row removed
    });
})(django.jQuery);
```

Dve stvari koje treba imati na umu:

- JavaScript kod mora da se nalazi u bloku šablona ako nasleđujete `admin/change_form.html` ili se neće prikazati u konačnom HTML -u.
- `{{ block.super }}` se dodaje jer Djangov `admin_change_form_document_ready` blok sadrži JavaScript kod za rukovanje raznim operacijama u obrascu za promenu, pa nam je potrebno i to da se prikaže.

Ponekad ćete morati da radite sa jQuery dodacima koji nisu registrovani u prostoru django.jQuery imena. Da biste to uradili, promenite način na koji kod sluša događaje. Umesto da omotate slušaoca u `django.jQuery` imenski prostor, poslušajte događaj koji se odatle pokrenuo. Na primer:

U `change_form.html`:

```html
{% extends 'admin/change_form.html' %}
{% load static %}
{% block admin_change_form_document_ready %}
{{ block.super }}
<script src="{% static 'app/unregistered_handlers.js' %}"></script>
{% endblock %}
```

U `app/static/app/unregistered_handlers.js`:

```js
django.jQuery(document).on('formset:added', function(event, $row, formsetName) {
  // Row added
});
django.jQuery(document).on('formset:removed', function(event, $row, formsetName) {
  // Row removed
});
```

[Sadržaj](00_sadrzaj.md)
