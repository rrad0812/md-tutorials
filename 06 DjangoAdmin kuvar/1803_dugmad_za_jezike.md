
# Dugmad za jezike

[Sadržaj](00_sadrzaj.md)

Sa `LocaleMiddleware` i `i18n_patterns`, stranice trebaju biti automatski prevedene na osnovu konteksta ili prefiksa URL-a. Međutim, ipak bi bilo sjajno dozvoliti korisniku da ručno da prebaci jezik iz admin interfejsa.
Klik na dugme je intuitivnije od rada sa prefiksima URL-a.

Postoji mnogo načina za dodavanje jezičnih preklopnika na veb lokaciju Admin. Za mene je najlakši način da dodate ikone zastave na naslovnu traku. Iza scene, svaka ikona zazastave bila bi povezana sa URL prefiksom jezika za stranicu. Na taj način, kad god korisnik klikne na zastavu, tada je ista stranica učitana za željeni jezik.

## Prekidač prefiksa jezičkog koda

[Sadržaj](00_sadrzaj.md)

Pošto URL putevi koriste i18n_patterns, njihovi jezički kodovi mogu biti uniformni. Pomoćna funkcija može lako da doda ili zameni željeni jezik koda kao prefiks URL-u. Na primer, da pretvorimo "/admin/" i "/en/admin/" u "/zh-hans/admin/" za pojednostavljeni kineski. Ova funkcija takođe treba da potvrdi da su put i jezik tačni. Može se staviti bilo gde u projektu.

```py
from django.conf import settings

    def switch_lang_code(path, language):
        # Get the supported language codes
        lang_codes = [c for (c, name) in settings.LANGUAGES]

        #Validate the inputs
        if path == '':
            raise Exception('URL path for language switch is empty')
        elif path[0] != '/':
            raise Exception('URL path for language switch does not start with "/"')
        elif language not in lang_codes:
            raise Exception('%s is not a supported language code' % language)
        
        # Split the parts of the path
        parts = path.split('/')
        
        # Add or substitute the new language prefix
        if parts[1] in lang_codes:
            parts[1] = language
        else:
            parts[0] = "/" + language
        
        #Return the full new path
        return '/'.join(parts)
```

## Prekidač prefiksa šablonskog filtera

[Sadržaj](00_sadrzaj.md)

Konačno, ovu funkciju mora da zove Django šablon kako bi se pružile veze do stranica specifičnih za jezik. Dakle, potreban nam je prilagođeni filter šablona. Modul za implementaciju filtera može se staviti u bilo koju aplikaciju, ali mora biti u podpaketu po imenu `templatetags` - tako Django zna da traži oznake prilagođenih šablonai filtere. Novi filteri će biti lako pisati jer već imamo funkciju switch_lang_code. (Razdvajanje logike za rukovanje prefiksom iz samog filtra.)

Kod je ispod:

U `[app]/templatetags/i18n_switcher.py`

```py
from django import template
from django.template.defaultfilters import stringfilter

register = template.Library()
@register.filter
@stringfilter
def switch_i18n_prefix(path, language):
"""takes in a string path"""

return switch_lang_code(path,language)
@register.filter
def switch_i18n(request, language):

"""takes in a request object and gets the path from it"""
return switch_lang_code(request.get_full_path(), language)
```

## Nadjačavanje Admin šablona

[Sadržaj](00_sadrzaj.md)

Konačno, admin šabloni moraju biti nadjačani tako da možemo dodati nove elemente na Admin stranice.

Bilo koji šablon admina može se nadjačati stvaranjem novih šablona istog imena pod `[project-root]/templates/admin`. Sadržaj roditelja koristiće se ako se izričito ne nadjača unutar datoteke dečijeg šablona. Pošto želimo da promenimo naslovnutraku, kreiramo novu datoteku šablona za base_site.html sa sledećim sadržajem:

```html
{% extends "admin/base_site.html" %}
{% load static %}
{% load i18n %}

<!--custom filter module -->
{% load i18n_switcher %}
{% block extrahead %}
<link rel="shortcut icon" href="{% static 'images/favicon.ico' %}" />
<link rel="stylesheet" type="text/css
href="{% static 'css/custom_admin.css' %}"/>
{% endblock %}

{% block userlinks %}
<a href="{{ request|switch_i18n:'en' }}">
<img class="i18n_flag" src="{% static 'images/flag-usa-16.png' %}"/> </a> /
<a href="{{ request|switch_i18n:'zh-hans' }}">
<img class="i18n_flag" src="{% static 'images/flag-china-16.png' %}"/> </a> /
{% if user.is_active and user.is_staff %}
{% url 'django-admindocs-docroot' as docsroot %} {% if docsroot %}
<a href="{{ docsroot }}">{% trans 'Documentation' %}</a> / {% endif %}
{% endif %}
{% if user.has_usable_password %}
<a href="{% url 'admin:password_change' %}">{% trans 'Change password' %}</a> /
{% endif %}
<a href="{% url 'admin:logout' %}">{% trans 'Log out' %}</a> 
{% endblock %}
```

Statički CSS fajl sa imenom css/custom_admin.css trebalo bi da ima sledećisadržaj:

```css
.i18n_flag img
{ 
width: 16px;
vertical-align: text-top;
}
```

Primetite da je ceo userlinks blok morao da se prepisuje kako bi se na mesto postavila zastava. Statičke datoteke slika zastava su jednostavno besplatne emojis zastave. One su hiperlinkovane na odgovarajući jezik URL za stranicu: `switch_18n` primenjen je na objekt aktivnog zahteva da biste dobili željeni jezik - prefiksovan put. (Napomena: U mom primeru sam uklonio vezu "View site" jer je moj sajt nije trebao.)

## Završno o jezičkim dugmadima

U mom projektu, ja sam izabrao da postavim jezički prefiks prekidač kod u posebnu aplikaciju `i18n_switcher`.

Fajlovi u mom projektu potrebni za admin jezičku dugmad su oraganizaovani na sledeći način:

```shell
[root]
|-i18n_switcher
| |-templatetags
| | |- init .py
| | `-i18n_switcher.py
| |- init .py
| `-apps.py
|-locale
| `-zh_Hans
| `-LC_MESSAGES
| |-django.mo
| ` -django.po
|-static
| |-css
| | `-custom_admin.css
| `-images
| |-flag-china-16.png
| `-flag -usa-16.png
`-templates
`-admin
`-base_site.html
```

Pošto sam kreirao novu aplikaciju za novi kod, takodje sam morao da dodam ime app u INSTALLED_APPS u `settings.py`:

U `settings.py`:

```py
INSTALLED_APPS = [
  ...
  'i18n_switcher.apps.I18NSwitcherConfig',
  ...
]
```

Kao što je već pomenuto, ikone zastava u naslovnoj traci su jedan od načina da se omoguće jednostavne veze do prevedenih stranica. Dobro funkcioniše kada postoji samo nekoliko dostupnih jezičkih izbora. Drugačiji pogled bio bi bolji za više jezika, poputpadajuće liste, druge linije u naslovnoj traci, ili čak i na footer-u stranice.

[Sadržaj](00_sadrzaj.md)
