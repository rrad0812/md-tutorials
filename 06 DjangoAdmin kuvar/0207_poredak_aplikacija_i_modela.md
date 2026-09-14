
# Poredak aplikacija i modela

[Sadržaj](00_sadrzaj.md)

Django podrazumevano redja modele u adminu alfabetski. Šablon korišćen za renderovanje admin index strane je `admin/index.html` i funkcija pogleda je `ModelAdmin.index`.

```py
def index(self, request, extra_context=None):
  """
  Display the main admin index page, which lists all of
  the installed apps that have been registered in this site.
  """
  app_list = self.get_app_list(request)
  context = {
    self.each_context(request),
    'title': self.index_title,
    'app_list': app_list,
    (extra_context or {}),
  }
  request.current_app = self.name

  return TemplateResponse(request, self.index_template or admin/index.html', context)
  ```

Metod `get_app_list`, postavlja poredak modela:

```py
def get_app_list(self, request):
  """
  Return a sorted list of all the installed apps that have
  been registered in this site.
  """
  app_dict = self._build_app_dict(request)
```

[Sadržaj](00_sadrzaj.md)

## Alfabetski poredak aplikacija

```py
app_list = sorted(app_dict.values(), key=lambda x: x['name'].lower())
```

[Sadržaj](00_sadrzaj.md)

## Alfabetski poredak modela unutar svake aplikacije

Za aplikacije na app_list listi:

```py
app['models'].sort(key=lambda x: x['name'])
return app_list
```

Da bi imali prilagodjeni poredak, nadjačajmo `get_app_list`:

```py
class EventAdminSite(AdminSite):
  
  def get_app_list(self, request):
    """
    Return a sorted list of all the installed apps
    that have been registered in this site.
    """
    ordering = {
      "Event heros": 1, "Event villains": 2, "Epics": 3, "Events": 4
    }
    app_dict = self._build_app_dict(request) a.sort(key=lambda x: b.index(x[0]))

    # Sort the apps alphabetically.
    app_list = sorted(app_dict.values(), key=lambda x: x['name'].lower())

    # Sort the models alphabetically within each app.
    for app in app_list:
      app['models'].sort(key=lambda x: ordering[x['name']])
    
    return app_list
```

Kod:

```py
app['models'].sort(key=lambda x: ordering[x['name']])
```

postavlja željeni poredak.

[Sadržaj](00_sadrzaj.md)
