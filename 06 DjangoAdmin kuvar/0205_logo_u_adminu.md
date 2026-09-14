
# Logo u adminu

[Sadržaj](00_sadrzaj.md)

UMSRA marketing želi logo na admin stranama. Potrebno je da nadjačaš podrazumevani šablon. U Django `settings.py`, treba `TEMPLATES` da varijabla izgleda ovako:

```py
TEMPLATES = [
  {
    'BACKEND': 'django.template.backends.django.DjangoTemplates',
    DIRS': [],
    'APP_DIRS': True,
    OPTIONS': {
      'context_processors': [
      'django.template.context_processors.debug',
      'django.template.context_processors.request',
      'django.contrib.auth.context_processors.auth',
      'django.contrib.messages.context_processors.messages',
      ],
    },
  }
]
```

Ovo znači da Django gleda šablone u `app` direktorijumu sa nazivom `templates` unutar svakog `app` direktorijuma, ali možete nadjačati postavkom vrednosti u `TEMPLATES.DIRS`.

Promenimo

```py
'DIRS': [], 
```

na:

```py
'DIRS': [os.path.join(BASE_DIR, 'templates/')],
```

I kreirajmo `templates` direktorijum u projektnom direktorijumu.

Ako je `STATICFILES_DIRS` postavka prazna postavimo je na:

```py
STATICFILES_DIRS = [
  os.path.join(BASE_DIR, "static"),
]
```

Sada kopirajmo `base_site.html` iz admin app u `templates\admin` direktorijum koji ste kreirali. Zamenimo podrazumevani tekst u `brend` bloku sa:

```html
<h1 id="site-name">
<a href="{% url 'admin:index' %}">

<img src="{% static 'umsra_logo.png' %}" height="40px" />
</a>

</h1>
```

Sa promenama `base_site.html` sada izleda kao:

```html
{% extends "admin/base.html" %}
{% load staticfiles %}

{% block title %}
{{ title }} | {{ site_title|default:_('Django site admin')}}
{% endblock %}

{% block branding %}
<h1 id="site-name">
<a href="{% url 'admin:index' %}">
<img src="{% static 'umsra_logo.png' %}" height="40px" />
</a>
</h1>
{% endblock %}

{% block nav-global %}
{% endblock %}
```

[Sadržaj](00_sadrzaj.md)
