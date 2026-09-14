
# Prilagodjne Django Error strane

Django dolazi sa podrazumevanim prikazima za stranice 400, 403, 404 i 500, što je odlično. Međutim, podrazumevani šabloni za ove prikaze su minimalni - oni samo ukazuju na to o kojoj grešci se radi. Ovaj blog post će vam pokazati kako da dodate prilagođene šablone i prikaze za stranice sa
greškama.

Šabloni grešaka Django-a korišćeni u ovom postu su upakovani i objavljeni u PyPI . Repozitorijum "django-custom-error-views" sadrži detalje o korišćenju paketa.

## Korišćenje prilagođenih šablona grešaka

Django podrazumevano pokušava da koristi šablone 400.html, 403.html i 500.html iz korenskog direktorijuma. Stoga, možete ih jednostavno dodati i Django će ih koristiti podrazumevano.

Evo primera projekta koji to prikazuje. Kao što vidite, postoje 4 šablona za sve greške 400–500, jedan od ovih šablona na primer koji dodaje logo, dugme za povratak na početnu stranicu, sliku greške i opis. Šabloni su ovde u `<project>/templates/400.html`.

`400.html`

```html
{% extends "django_custom_error_views/base.html" %}
{% load static %}
{% block title %}
    Bad Request
{% endblock title %}

{% block content %}
{% if company_logo %}
    <a href="/"><img class="mt-5" style="max-height: 45px"
    src="{{ STATIC_URL }}/{{ company_logo }}"
    onerror="this.onerror=null; this.src='{{ company_logo }}'"
    alt="Company Logo">
    </a>
{% endif %}

<img class="mt-5" style="max-height: 500px"
src="{% static image|default:'django_custom_error_views/visual-400.jpg' %}"
alt="Error image">

<p class="font-bold text-3xl font-[Poppins] text-center">{{ title| 
    default:"Error, bad request." }}
</p>

<p class="mt-5 max-w-xl text-center">
    {{ description|default:"This ia a malformed or illegal request.  You can find loads to explore on the home page." }}
</p>

<p class="mt-2 max-w-xl text-center">{{ extra_content }}</p>
    {% if exception and render_exception %}
       <p class="mt-2 max-w-xl text-center">The following exception was raised: 
    {{ exception }}
</p>
    {% endif %}
<a type="button"
  class="mt-5 rounded-full font-semibold bg-white px-4 py-2.5 text-sm text-blue-800 shadow-sm ring-1 ring-inset ring-blue-800 hover:bg-gray-50"
  href="/">Return Home</a>

{% endblock %}
```

Gore navedeni šablon je malo složen, a možete ga učiniti mnogo jednostavnijim. Gore navedeni šablon takođe zavisi od konteksta koji se dodaje prilikom renderovanja šablona. To se može uraditi korišćenjem prilagođenih prikaza.

## Korišćenje prilagođenih prikaza

Da bismo dodali kontekst šablonima, biće nam potreban prilagođeni prikaz. Hajde da pogledamo prikaz greške 500.

```py
@requires_csrf_token
def handler500(request, template_name="500.html"):
    """
    500 error handler.
    """
    error_page_title = getattr(settings, "ERROR_PAGE_TITLE", None)
    error_page_description = getattr(settings, "ERROR_PAGE_description", None)
    context = {
        "error_page_title": error_page_title
        "error_page_description": error_page_description
    }

    template = loader.get_template(template_name)
    body = template.render(context, request)
    
    return HttpResponseServerError(body)
```

Sada možete koristiti kontekstualne promenljive u svom šablonu.

```html
...
<p>{{ error_page_title }}</p>
<p>{{ error_page_description }}</p>
...
```

Međutim, da bi Django koristio vaš prilagođeni program za obradu grešaka, potrebno je da dodate sledeći isečak u svoju `urls.py` datoteku.

```py
handler400 = "<my_project>.views.handler400"
handler403 = "<my_project>.views.handler403"
handler404 = "<my_project>.views.handler404"
handler500 = "<my_project>.views.handler500"
```

Sada ćete imati prilagođene prikaze i šablone za stranice sa greškama i možete prilagoditi i promeniti sve što želite.

## Paket otvorenog koda za prilagođene prikaze grešaka

Kao što je već pomenuto, spakovao sam gore navedeno u Django paket koji je objavljen na PyPI-ju i potpuno je prilagodljiv. Instalacija je jednostavna.

```sh
pip install django-custom-error-views
```

Dodajte paket u instalirane aplikacije.

```py
INSTALLED_APPS = [
    "django_custom_error_views",
]
```

Sada će Django koristiti šablone grešaka iz paketa! To je zapravo sve! Međutim, ako želite da ih prilagodite, potrebno je da zamenite obrađivače grešaka dodavanjem obrađivača grešaka iz paketa u urls.py.

```py
handler400 = "django_custom_error_views.views.handler400"
handler403 = "django_custom_error_views.views.handler403"
handler404 = "django_custom_error_views.views.handler404"
handler500 = "django_custom_error_views.views.handler500"
```

Sada možete da prepišete sve stranice i dodate sopstveni brending putem podešavanja Django-a.

```py
DJANGO_CUSTOM_ERROR_VIEWS = {
    "company_logo": "/images/icon.png", # Static image or full URL
    "400": {
        "title": "Custom 400 error.",
        "description": "Custom 400 description.",
        "extra_content": "400 extras.",
    },
    "403": {
        "title": "Custom 403 Error.",
        "description": "Custom 403 description.",
        "extra_content": "403 extras.",
        "render_exception": True,
    },
...
```

Pogledajte paket na GitHabu za sva podešavanja koja možete podesiti.
