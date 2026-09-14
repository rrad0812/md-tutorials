
# AJAX request imlementacija

[Sadržaj](00_sadrzaj.md)

## Inicijalni setup

Posle inicijalnog setup-a za Django, kreiraj jedan projekat i `base.html` fajl.

`base.html`

```html
{% load static %}
<!doctype html>
<html>
<head>
  <meta charset="utf-8">
  <title>{% block title %}Default Title{% endblock %}</title>
  <link rel="stylesheet" type="text/css" href="{% static 'css/app.css' %}">
  {% block stylesheet %}
  {% endblock %}
</head>

<body>
  <header>
  ...
  </header>

  <main>
  {% block content %}
  {% endblock %}
  </main>

  <footer>
  ...
  </footer>

  <script src="https://code.jquery.com/jquery-3.1.0.min.js"></script>
  <script src="{% static 'js/app.js' %}"></script>
  {% block javascript %}
  {% endblock %}

</body>
</html>
```

Primetite da `jQuery` biblioteka i `JavaScript` resursi trebaju biti smešteni na kraj HTML strane da garantuje da DOM će biti učitan kada je skript izvršen i da izbegne inlajn skriptove koji koriste `jQuery`.

`views.py`

```py
from django.contrib.auth.forms import UserCreationForm
from django.views.generic.edit import CreateView

class SignUpView(CreateView):
    template_name = 'core/signup.html'
    form_class = UserCreationForm
```

`url.py`

```py
from django.conf.urls import url
from core import views

urlpatterns = [
    path(' signup/', views.SignUpView.as_view(), name='signup')
]
```

`signup.html`

```html
{% extends 'base.html' %}
​
{% block content %}
  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Sign up</button>
  </form>
{​ % endblock %}
```

[Sadržaj](00_sadrzaj.md)

## AJAX Request implementacija

Ispod implementacije AJAX-a je asinhroni zahtev da potvrdi da li je korisnički ulaz za korisničko ime već iskorišćen ili je dostupan za upotrebu. Prvi korak uključuje gledanje HTML-a generisan kao {{form.AS_P}} da bi pregledao polje za korisničko ime.

Da biste potvrdili korisničko ime, kreiramo listener za događaj promene
`id_username` polja.

`signup.html`

```html
{​ % extends 'base.html' %}

{​ % block javascript %}
<script>
  $("#id_username").change(function () {
    console.log( $(this).val() );
  });
 </script>
​{% endblock %}

{% block content %}

  <form method="post">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Sign up</button>
  </form>
{% endblock %}
```

U implementaciji iznad, događaj promene će se pojaviti svaki put kada se menja vrednost polja za korisničko ime. Zatim kreiramo pogled koji proverava da li je korisničko ime prihvaćeno i vraća JSON odgovor.

`pyviews.py`

```py
from django.contrib.auth.models import User
from django.http import JsonResponse

def validate_username (request):
    username = request.GET.get('username', None)
    data = {
        'is_taken': User.objects.filter(username__iexact=username).exists()
    }
    return JsonResponse(data)
```

Sledeći korak je dodavanje rute na prikaz (views.py).

`urls.py`

```py
from django.conf.urls import url
f​rom core import views

urlpatterns = [
    path(' signup/', views.SignUpView.as_view(), name='signup'),
    path(' ajax/validate_username/', , views.validate_username,
        name='validate_username'),
]
```

Na kraju, implementiramo AJAX:

`signup.html`

```html
{​ % extends 'base.html' %}
{​ % block javascript %}
<script>
  $("#id_username").change(function () {
​    var username = $(this).val();
    $.ajax({
      url: '/ajax/validate_username/',
      data: {
        'username': username
​      },
      dataType: 'json',
      success: function (data) {
        if (data.is_taken) {
          alert("A user with this username already exists.");
        }
      }
​    });
  });
</script>
{% endblock %}

{% block content %}
<form method="post">
  {% csrf_token %}
  {{ form.as_p }}
  <button type="submit">Sign up</button>
</form>
{% endblock %}
```

Kod iznad je dodao događaj za rukovanje korisničkim unosom, koji potom naziva funkciju koja koristi `AJAX` da proveri da li je dato ime iskorišćeno i vraća odgovor.

[Sadržaj](00_sadrzaj.md)
