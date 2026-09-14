
# Dodatka dj-pagination

[Sadržaj](00_sadrzaj.md)

Nažalost, nisam uspeo da pronađem biblioteku koja proširuje Djangov osnovni `Paginator` za dodavanje paginacije skupa ključeva. Admin tabela zahteva paginator tog tipa, pa ću iste generisane podatke demonstrirati u novom prikazu aplikacije pomoću dodatka `dj-pagination`.

Dodajte aplikaciju u svoj `INSTALLED_APPS` i middleware - dokumenti dobro objašnjavaju. Zatim kreirajte prikaz u aplikaciji Users:

`{project}/users/views.py`

```py
from django.shortcuts import render from .models import User

def index(request):
  context = {
    'users': User.objects.order_by('id').all()
  }
  return render(request, 'users/index.html', context)
```

Ovaj kod omogućava korisnicima da budu QuerySet dostupni u kontekstu prikaza i kaže mu da prikaže datoteku šablona. Zatim dodajte direktorijum šablona u svoj `TEMPLATES` objekat podešavanja i dodajte novu datoteku šablona:

`{project}/templates/users/index.html`

```html
{% load pagination_tags %}
{% autopaginate users %}
{% for user in users %}
{{ user.name }}
{% endfor %}
{% paginate %}
```

U stvarnoj aplikaciji, dodali biste dodatno označavanje i oblikovanje kako biste prikazali listu korisnika, ali to pokazuje oznake koje vam `dj-pagination` postaju dostupne.

Sada kada imate prikaz i šablon, napravite `urls.py` datoteku za usmeravanje zahteva do prikaza:

`{project}/users/urls.py`

```py
from django.urls import path 
from . import views

app_name = 'users' urlpatterns = [
  # ex: /users/
  path('', views.index, name='index'),
]
```

Na kraju, dodajte ga u svoje URL adrese:

```py
// various imports... 
urlpatterns = [
  // routes...
  path('users/', include('users.urls')),
]
```

Sada, kada usmerite pregledač na `/users stranicu`, videćete listu korisničkih imena sa jednostavnim kontrolama paginacije. Generisani upit vraća rezultate za manje od 100 ms, bez obzira na koju stranicu pređem.

Ako tražite način da imate veću kontrolu nad paginacijom skupova ključeva, `django-infinite-scroll-pagination` bi možda bilo vredno pogledati.

[Sadržaj](00_sadrzaj.md)
