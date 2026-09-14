
# Nezavisni sajtovi

[Sadržaj](00_sadrzaj.md)

Uobičajeni način kreiranja admin strana je stavljanje svih modela u jednog admina. Međutim, moguće je imati više admin lokacija u jednoj Django aplikaciji. Trenutno su naši modeli entiteta i događaja na istom mestu. UMSRA ima dve različite grupe koje istražuju događaje i entitete, i zato želi da podeli admine. Zadržaćemo zadanog admina za entitete i kreiraćemo novu subklasu AdminSite za događaje.

U events/admin.py uradimo sledeće:

```py
from django.contrib.admin import AdminSite

class EventAdminSite(AdminSite):
  site_header = "UMSRA Events Admin"
  site_title = "UMSRA Events Admin Portal"
  index_title = "Welcome to UMSRA Researcher Events Portal"

event_admin_site = EventAdminSite(name='event_admin')

event_admin_site.register(Epic)
event_admin_site.register(Event)
event_admin_site.register(EventHero)
```

I promenimo urls.py na:

```py
from events.admin import event_admin_site

urlpatterns = [
  path('entity-admin/', admin.site.urls),
  path('event-admin/', event_admin_site.urls),
]
```

Ovo razdvaja aplikaciju na dva admin sajta. Oba su raspoloživa na:

- /entity-admin/
- /event-admin/.

[Sadržaj](00_sadrzaj.md)
