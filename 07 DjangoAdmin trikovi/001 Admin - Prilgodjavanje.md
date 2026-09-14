
# Prilagodjenje Django admina

Django-ov automatski generisan admin sajt je jedna od najvećih prednosti ovog frejmvorka. Centralizovani admin interfejs vam omogućava da lako pregledate i manipulišete podacima vaših modela. Ovo vam može uštedeti mnogo vremena prilikom razvoja i upravljanja sadržajem.

Iako je admin sajt veoma prilagodljiv, mnogi programeri nisu svesni njegovih punih mogućnosti. To dovodi do toga da programeri kreiraju prikaze i funkcije za stvari koje bi se mogle lako implementirati uz malo podešavanja admin sajta.

U ovom članku ćemo pogledati kako da prilagodimo Django-ov administratorski sajt kroz praktične primere. Obradićemo ugrađene opcije prilagođavanja, kao i prilagođavanje putem paketa trećih strana kao što su "DjangoQL", "django-import-export" i "django-admin-interface".

## Sadržaj

- [Ciljevi](#ciljevi)
- [Podešavanje projekta](#podešavanje-projekta)
- [Osnovno prilagođavanje admin sajta](#osnovno-prilagođavanje-admin-sajta)
- [Django model i admin](#django-model-i-admin)
- [Prilagodjavanje changelist strane pomoću klase ModelAdmin](#prilagodjavanje-changelist-strane-pomoću-klase-modeladmin)
  - [Prikaz kontrola pomoću list_display na changelist strani](#prikaz-kontrola-pomoću-list_display-na-changelist-strani)
  - [Prikaz prilagođenih polja na changelist strani](#prikaz-prilagođenih-polja-na-changelist-strani)
  - [Filtriranje objekata ModelAdmina na changelist strani](#filtriranje-objekata-modeladmina-na-changelist-strani)
  - [Pretraživanje objekata modela na changelist strani](#pretraživanje-objekata-modela-na-changelist-strani)
- [Rukovanje redovima ugradjenih](#rukovanje-redovima-ugradjenih-modela)
- [Prilagođene admin akcije](#prilagođene-admin-akcije)
- [Zamena Django admin forme](#zamena-django-admin-forme)
- [Zamena Django admin šablona](#zamena-django-admin-šablona)
- [Napredna pretraga pomoću DjangoQL](#napredna-pretraga-pomoću-djangoql)
- [Uvoz i izvoz podataka pomoću django-import-export](#uvoz-i-izvoz-podataka-pomoću-django-import-export)
- [Stilizovanje admin sajta pomoću django-admin-interface](#stilizovanje-admin-sajta-pomoću-django-admin-interface)
- [Zaključak](#zaključak)

### Ciljevi

[Sadržaj](#sadržaj)

Do kraja ovog članka, moći ćete da:

- Izvršite osnovnu konfiguraciju Django admin sajta
- Objasnite kako atributi Django modela utiču na admin sajt
- Koristite `list_display` za kontrolu koja polja modela se prikazuju
- Dodate prilagođena polja u `list_display` i formatirate postojeća
- Dodate veze do povezanih objekata modela u `list_display`
- Omogućite pretragu i filtere putem `search_fields` i `list_filter`
- Rukujte ugrađenim redovima (`inlines`) modela za `N:1` i `M:M` odnose
- Koristite Django admin akcije i kreirate prilagođene akcije
- Zamenite Django admin forme i šablone
- Koristite DjangoQL za naprednu funkcionalnost pretraživanja
- Uvozite podatke i izvozite podatke u različite formate koristeći `django-import-export`
- Izmenite izgled vašeg admin sajta putem `django-admin-interface` dodatka.

## Podešavanje projekta

[Sadržaj](#sadržaj)

Da bih demonstrirao različite opcije prilagođavanja admin sajta, pripremio sam jednostavnu veb aplikaciju. Veb aplikacija služi kao sistem za prodaju karata za događaje. Omogućava vam upravljanje mestima održavanja, koncertima, kategorijama koncerata i kartama.

Prvo, preuzmite izvorni kod iz repozitorijuma na GitHub-u:

```sh
git clone https://github.com/duplxey/django-admin-customization.git --branch base
cd django-admin-customization
```

Napravite virtuelno okruženje i aktivirajte ga:

```sh
python3.11 -m venv env && source env/bin/activate
```

Instalirajte zahteve i migrirajte bazu podataka:

```sh
(venv)$ pip install -r requirements.txt
(venv)$ python manage.py migrate
```

Kreirajte superusera i popunite bazu podataka:

```sh
(venv)$ python manage.py createsuperuser
(venv)$ python manage.py populate_db
```

Pokrenite razvojni server:

```sh
(venv)$ python manage.py runserver
```

Otvorite svoj omiljeni veb pregledač i idite na <http://localhost:8000/admin>. Pokušajte da koristite svoje superkorisničke akreditive za pristup admin stranici Django-a. Nakon toga, uverite se da je baza podataka popunjena sa nekoliko mesta održavanja, kategorija koncerata, koncerata i karata.

Pre nego što nastavite, predlažem da proverite modele u `tickets/models.py`. Obratite pažnju na to koja polja model ima i kako su modeli povezani.

## Osnovno prilagođavanje admin sajta

[Sadržaj](#sadržaj)

Admin sajt Django-a pruža neke osnovne opcije konfiguracije. Ove opcije vam omogućavaju da promenite

- naslov,
- zaglavlje,
- URL adresu sajta

i još mnogo toga. Ovo `site_header` podešavanje može biti posebno korisno ako imate više okruženja i želite da ih lako razlikujete.

Podešavanja se obično menjaju u glavnoj datoteci `urls.py` admin.site vašeg projekta.

Preimenujte Django administratora u "TicketPlus" i označite trenutno okruženje kao dev:

`# core/urls.py`

```py
admin.site.site_title = "TicketsPlus site admin (DEV)"
admin.site.site_header = "TicketsPlus admin"
admin.site.index_title = "Site admin"
```

Sva podešavanja se mogu videti proverom Django-ove datoteke `contrib/admin/sites.py`. Još jedna stvar koju bi trebalo da uradite jeste da promenite podrazumevanu `/admin` URL adresu. Ovo će otežati zlonamernim akterima da pronađu vaš administratorski panel.

Promenite datoteku `core/urls.py` ovako:

`# core/urls.py`

```py
urlpatterns = [
  path("secretadmin/", admin.site.urls),
]
```

Vaša administratorska stranica bi sada trebalo da bude dostupna na <http://localhost:8000/secretadmin>.

## Django model i admin

[Sadržaj](#sadržaj)

Neki atributi Django modela direktno utiču na Django admin sajt. Najvažnije:

- `__str__()` koristi se za prikazivanje definisanog imena objekta
- `class Meta` se koristi za podešavanje različitih opcija metapodataka (npr. `ordering` i verbose_name`)

Evo primera kako se ovi atributi koriste u praksi:

`# tickets/models.py`

```py
class ConcertCategory(models.Model):
    
    name = models.CharField(max_length=64)
    description = models.TextField(max_length=256, blank=True, null=True)

    class Meta:
        verbose_name = "concert category"
        verbose_name_plural = "concert categories"
        ordering = ["-name"]

    def __str__(self):
        return f"{self.name}"
```

Naveli smo oblik množine jer množina od "concert category" nije "concert categorys".

Postavljanjem `ordering` atributa, kategorije su sada podrazumevano sortirane po imenu, u opadajućem redosledu.

## Prilagodjavanje `changelist` strane pomoću klase ModelAdmin

[Sadržaj](#sadržaj)

U ovom odeljku ćemo pogledati kako se koristi klasa `ModelAdmin` za prilagođavanje admin sajta.

### Prikaz kontrola pomoću `list_display` na `changelist` strani

[Sadržaj](#sadržaj)

Atribut `list_display` vam omogućava da kontrolišete koja modela se prikazuju na stranici `changelist` modela. Još jedna odlična stvar je to što može da prikaže povezana polja modela pomoću `__` operatora.

Podesimo za ConcertAdmin `list_display` atribut:

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
    list_display = ["name", "venue", "starts_at", "price", "tickets_left"]
    readonly_fields = ["tickets_left"]
```

Sačekajte da se server osveži i pogledajte stranicu sa spiskom koncerata u admin delu.

Nova lista izgleda odlično, ali postoji problem. Dodavanjem mesta održavanja u `list_display`, uveli smo problem `N + 1 upita`. Pošto Django treba da preuzme ime mesta održavanja za svaki koncert posebno, izvršava se `N + 1 upita`.

Da bismo izbegli problem N + 1, možemo koristiti atribut `list_select_related`, koji funkcioniše slično metodi `select_related`:

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
  list_display = ["name", "venue", "starts_at", "price", "tickets_left"]
  list_select_related = ["venue"]
  readonly_fields = ["tickets_left"]
```

Zatim, postavite elemente `list_display` za ostale klase.

`# tickets/admin.py`

```py
class VenueAdmin(admin.ModelAdmin):
    list_display = ["name", "address", "capacity"]

class TicketAdmin(admin.ModelAdmin):
    list_display = ["customer_full_name", "concert", "payment_method",
        "paid_at", "is_active",
    ]
    list_select_related = ["concert", "concert__venue"]  # to avoid N + 1
```

### Prikaz prilagođenih polja na changelist strani

[Sadržaj](#sadržaj)

Podešavanje `list_display` se takođe može koristiti za dodavanje prilagođenih polja. Da biste dodali prilagođeno polje, morate definisati novu metodu unutar `ModelAdmin` klase.

Dodajte polje "Rasprodato", koje je `True` ako nema dostupnih karata.

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
    list_display = ["name", "venue", "starts_at", "tickets_left", "display_sold_out"]
    list_select_related = ["venue"]
  
    def display_sold_out(self, obj):
        return obj.tickets_left == 0
  
    # Podešavanje dodatnih osobina prilagodjenog atributa display_sold_out
    display_sold_out.short_description = "Sold out"
    display_sold_out.boolean = True
```

Sa `short_description` podesili smo ime kolone i sa `boolean` rekli smo Djangu da ta kolona ima bulovsku vrednost. Na ovaj način, Django prikazuje ikonu krstića/kvačice umesto `False` i `True`. Takođe smo morali da dodamo našu "display_sold_out" metodu u `list_display`.

Zatim, dodajmo prilagođeno polje pod nazivom "display_price":

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
    list_display = [
      "name", "venue", "starts_at", "tickets_left", "display_sold_out", "display_price"
    ]
  
    # ...
    def display_price(self, obj):
      return f"${obj.price}"
  
    display_price.short_description = "Price"
    display_price.admin_order_field = "price"
```

Sa `admin_order_field` rekli smo Djangu da ako korisnik klikne na zaglavlje kolone "display_price", sortiranje će biti izvedeno po koloni "price".

Ponekad može biti korisno dodati veze do povezanih objekata modela umesto samo prikazivanja njihovog imena. Da bismo demonstrirali kako se to radi, povezaćemo mesta održavanja na stranici sa listom koncerata.

Pre nego što to uradimo, pogledajmo strukturu URL-ova admin sajta Django:

 Strana          | URL                              | Opis
-----------------|----------------------------------|-------------------------------------
Changelist admin | `admin:<app>_<model>_changelist` | Prikazuje listu objekata
Add admin        | `admin:<app>_<model>_add`        | Forma za dodavanje objekta
Change admin     | `admin:<app>_<model>_change`     | Forma za promenu objekta (objectId)
Delete admin     | `admin:<app>_<model>_delete`     | Forma za brisanje objekta (objectId)
Hystory admin    | `admin:<app>_<model>_history`    | Istoriju objekta (objectId)

Da bismo dodali vezu do strane za promenu mesta održavanja, moraćemo da koristimo sledeći URL:

- Format:   `admin:<app>_<model>_change`
- Aktuelno: `admin:tickets_venue_change`

Dodajte "display_venue" metodu u "ConcertAdmin"

`# tickets/admin.py`

```py
class ConcertAdmin(DjangoQLSearchMixin, admin.ModelAdmin):
    list_display = [
        "name", "venue", "starts_at", "tickets_left", "display_sold_out", "display_price",   
        "display_venue",
    ]
    list_select_related = ["venue"]
  
    # ...
    def display_venue(self, obj):
      link = reverse("admin:tickets_venue_change", args=[obj.venue.id])
      return format_html('<a href="{}">{}</a>', link, obj.venue)
   
    display_venue.short_description = "Venue"
```

Koristili smo `reverse` metod da bismo obrnuli URL i prosledili `obj.venue.id` kao `objectId`.

Ne zaboravite na uvoz:

```py
from django.urls import reverse
from django.utils.html import format_html
```

Sačekajte da se razvojni server osveži i idite na stranicu sa spiskom koncerata. Mesta održavanja bi sada trebalo da budu klikabilna i da vode do odgovarajućeg objekta u `VenueAdmin`.

### Filtriranje objekata `ModelAdmina` na `changelist` strani

[Sadržaj](#sadržaj)

Da biste omogućili filtriranje, morate navesti koja polja ili povezana polja modela treba da budu filtrirana. Najbolje od svega je što Django može da ulančava filtere, tj da filtrira po dva ili više polja istovremeno.

Samo napred i dodajte atribut `list_filter` u "ConcertAdmin" ovako:

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
  # ...
  list_filter = ["venue"]
```

Da biste filtrirali po poljima povezanog objekta, koristite sve podcrte `__` operator.

Prilikom izbora filtera, vodite računa da ne uključite polja sa previše vrednosti. Na primer, `tickets_left` je loš izbor filtera jer za svaki koncert ima različit broj preostalih karata.

Za napredniju funkcionalnost filtriranja, možete definisati i prilagođene filtere. Da biste definisali prilagođeni filter, morate navesti opcije ili tzv. `lookups` i `queryset` za svaki lookup.

Na primer, da biste filtrirali po tome da li je koncert rasprodat ili ne, kreirajte "SoldOutFilter" i uključite ga u "ConcertAdmin" u `list_filters`:

`# tickets/admin.py`

```py
class SoldOutFilter(SimpleListFilter):
  title = "Sold out"
  parameter_name = "sold_out"
  
  def lookups(self, request, model_admin):
    return [
      ("yes", "Yes"),
      ("no", "No"),
    ]

  def queryset(self, request, queryset):
    if self.value() == "yes":
      return queryset.filter(tickets_left=0)
    else:
      return queryset.exclude(tickets_left=0)

class ConcertAdmin(admin.ModelAdmin):
  # ...
  list_filter = ["venue", SoldOutFilter]
```

Ne zaboravite na uvoz:

```py
from django.contrib.admin import SimpleListFilter
```

Posetite svoju admin stranu i uverite se da filteri rade kako je očekivano.

### Pretraživanje objekata modela na `changelist` strani

[Sadržaj](#sadržaj)

Django admin pruža osnovnu funkcionalnost pretrage. Može se omogućiti navođenjem koja polja modela treba da budu pretraživa putem `search_fields` atributa. Imajte na umu da Django ne podržava `fuzzy upite` podrazumevano.

Hajde da omogućimo pretragu naših koncerata po njihovim imenima, mestima održavanja i adresama mesta održavanja.

Dodajte atribut `search_fields` ConcertAdmin ovako:

`# tickets/admin.py`

```py
class ConcertAdmin(admin.ModelAdmin):
# ...
search_fields = ["name", "venue__name", "venue__address"]
```

Sačekajte da se server osveži i testirajte polje za pretragu.

> [!Note]
>
> Da bi pretraga bila efikasna polja pretrge treba da su indeksirana. To važi i za polja pretrage povezanih modela.

## Rukovanje redovima ugradjenih modela

[Sadržaj](#sadržaj)

Admin interfejs vam omogućava da uređujete modele na istoj stranici kao i roditeljski model putem `inlines` ugradjenih redova. Django pruža dve vrste `inlines`:

- `StackedInline` i
- `TabularInline`.

Glavna razlika između njih je u njihovom izgledu.

Koristimo `inlines` element za prikazivanje detalja koncerata na stranici sa detaljima o mestu održavanja.

Napravite `ConcertInline` kao `inlines` i dodajte ga u `VenueAdmin` ovako:

`# tickets/admin.py`

```py
class ConcertInline(admin.TabularInline):
  model = Concert
  fields = ["name", "starts_at", "price", "tickets_left"]

  # optional: make the inline read-only
  readonly_fields = ["name", "starts_at", "price", "tickets_left"]
  can_delete = False
  max_num = 0
  extra = 0
  show_change_link = True

class VenueAdmin(admin.ModelAdmin):
  list_display = ["name", "address", "capacity"]
  
  inlines = [ConcertInline]
```

Posetite admin sajt i idite na stranicu sa detaljima nekog mesta održavanja. Pomerite se nadole i trebalo bi da se nalazi odeljak "Concerts".

## Prilagođene admin akcije

[Sadržaj](#sadržaj)

Admin akcije Django-a vam omogućavaju da izvršite "akciju" na objektu ili grupi objekata. Akcija se može koristiti za izmenu atributa objekta, brisanje objekta, njegovo kopiranje i tako dalje. Akcije se prvenstveno koriste za često izvršavane "akcije" ili grupne izmene.

Savršen primer je aktiviranje ili deaktiviranje tiketa. Pretpostavimo da imamo mnogo tiketa koje želimo da aktiviramo. Bilo bi prilično zamorno kliknuti na svaki od njih, promeniti im "is_active" svojstvo i sačuvati model. Umesto toga, možemo definisati akciju koja će upravo to uraditi.

Definišite "activate_tickets" i "deactivate_tickets" akcije i dodajte ih na "TicketAdmin" na sledeći način:

`# tickets/admin.py`

```py
@admin.action(description="Activate selected tickets")
def activate_tickets(modeladmin, request, queryset):
    queryset.update(is_active=True)

@admin.action(description="Deactivate selected tickets")
def deactivate_tickets(modeladmin, request, queryset):
    queryset.update(is_active=False)

class TicketAdmin(admin.ModelAdmin):
    # ...
    actions = [activate_tickets, deactivate_tickets]
```

Ponovo otvorite admin stranu, idite na `changelist` prikaz zahteva za tikete i trebalo bi da vidite prilagođene akcije. Testirajte ih aktiviranjem i deaktiviranjem više zahteva odjednom.

## Zamena Django admin forme

[Sadržaj](#sadržaj)

Podrazumevano, Django automatski generiše `ModelForm` za vaš model. Taj obrazac se zatim koristi na stranici za dodavanje i izmene. Ako želite da prilagodite obrazac ili implementirate jedinstvenu validaciju podataka, moraćete da ga prepišete.

Da bismo ovo demonstrirali, podelićemo polje `customer_full_name` na dva unosna polja i prikazati radio dugmad umesto padajućeg menija za načine plaćanja.

Napravite datoteku forms.py u aplikaciji za karte :

`# tickets/forms.py`

```py
from django import forms
from django.forms import ModelForm, RadioSelect
from tickets.models import Ticket

class TicketAdminForm(ModelForm):
    first_name = forms.CharField(label="First name", max_length=32)
    last_name = forms.CharField(label="Last name", max_length=32)

    class Meta:
        model = Ticket
        fields = [
            "concert",
            "first_name",
            "last_name",
            "payment_method",
            "is_active"
        ]
    
        widgets = {
            "payment_method": RadioSelect(),
        }

    def __init__(self, *args, **kwargs):
        instance = kwargs.get('instance')
        initial = {}
        if instance:
            customer_full_name_split = instance.customer_full_name.split(" ", maxsplit=1)
            initial = {
                "first_name": customer_full_name_split[0],
                "last_name": customer_full_name_split[1],
            }
            super().__init__(*args, **kwargs, initial=initial)
  
    def save(self, commit=True):
        self.instance.customer_full_name = self.cleaned_data["first_name"] + " 
        " \ + self.cleaned_data["last_name"]
        return super().save(commit)
```

Ovde smo:

- dodali polja forme "first_name" i "last_name".
- koristili `Meta` klasu da odredimo na koji model se ova forma odnosi i koja polja treba uključiti.
- u forminoj `__init__()`, popunili smo ga koristeći podatke instance modela.
- definisali smo `save()`, spojili smo `first_name` i `last_name` i sačuvali ga kao `customer_full_name`.

Zatim, postavite `TicketAdmin` form:

`# tickets/admin.py`

```py
class TicketAdmin(admin.ModelAdmin):
  # ...
  form = TicketAdminForm
```

Ne zaboravite na uvoz:

```py
from tickets.forms import TicketAdminForm
```

Ponovo pokrenite razvojni server i idite na stranicu sa detaljima tiketa. Ako je sve prošlo kako treba, trebalo bi da vidite da su ime i prezime sada u odvojenim
poljima i da načini plaćanja koriste radio dugmad umesto padajućeg menija.

## Zamena Django admin šablona

[Sadržaj](#sadržaj)

Admin sajt Django-a vam omogućava da prilagodite bilo koji njegov vizuelni aspekt zamenom šablona. Sve što treba da uradite je:

- Pogledate izvorni kod Django-a i kopirajte originalni šablon.

- Nalepite šablon u "templates/admin" ili "templates/registration", respektivno.

Većinu vremena ćete moći da se izvučete samo promenom dela originalnog šablona.

Na primer, ako želimo da dodamo poruku iznad forme za prijavu, možemo naslediti `login.html`, a zatim promeniti `content_title` blok:

`<!-- templates/admin/login.html -->`

```html
{% extends "admin/login.html" %}

{% block content_title %}

<p style="background: #ffffcc; padding: 10px 8px">
    This is a really important message.
</p>

{% endblock %}
```

Idite na stranicu za prijavu i trebalo bi da vidite žutu poruku.

## Napredna pretraga pomoću DjangoQL

[Sadržaj](#sadržaj)

DjangoQL je moćan paket treće strane koji vam omogućava da izvršavate napredne upite bez oslanjanja na sirovi SQL. Ima sopstvenu sintaksu i automatsko dovršavanje, podržava logičke operatore i radi za bilo koji Django model.

Počnite instaliranjem paketa:

```sh
(env)$ pip install djangoql==0.17.1
```

Dodaj INSTALLED_APPS u `core/settings.py`:

`# core/settings.py`

```py
INSTALLED_APPS = [
  # ...
  "djangoql",
]
```

Zatim, dodajte `DjangoQLSearchMixin` kao roditeljsku klasu svim `ModelAdmin` klasama gde želite da omogućite napredne mogućnosti pretraživanja.

Dodajmo to, na `TicketAdmin` primer:

`# tickets/admin.py`

```py
class TicketAdmin(DjangoQLSearchMixin, admin.ModelAdmin):
  # ...
```

Ne zaboravite na uvoz:

```py
from djangoql.admin import DjangoQLSearchMixin
```

Sada možete koristiti isto polje za pretragu kao i ranije da biste izvršili napredne upite. 

Primeri:

1. `is_active = True` vraća aktivne karte,
2. `payment_method = "ET" or payment_method = "BC"` vraća karte kupljene kriptovalutama
3. `concert.venue.name ~ "Amphitheatre"` vraća karte za koncerte u amfiteatrima.
4. `concert.tickets_left > 500` vraćanje koncerte sa više od 500 preostalih karata.

Za više informacija o jeziku DjangoQL pogledajte "Referencu za jezik DjangoQL".

## Uvoz i izvoz podataka pomoću django-import-export

[Sadržaj](#sadržaj)

U ovom odeljku ćemo pogledati kako da uvezemo i izvezemo podatke o objektima putem `django-import-export`, što je odličan paket za lak uvoz i izvoz podataka u različitim formatima, uključujući JSON, CSV i YAML. Paket takođe dolazi sa ugrađenom admin integracijom.

Prvo, instalirajte ga:

```sh
(env)$ pip install django-import-export==3.2.0
```

Zatim, dodajte ga na `INSTALLED_APPS` u `core/settings.py`:

`# core/settings.py`

```py
INSTALLED_APPS = [
  # ...
  "import_export",
]
```

Prikupite statičke datoteke:

```sh
(env) python manage.py collectstatic
```

Nakon toga, dodajte `ImportExportActionModelAdmin` kao roditeljsku klasu svim `ModelAdmin` klasama koje želite da budu uvezene/izvezene.

Evo jednog primera za TicketAdmin:

`# tickets/admin.py`

```py
class TicketAdmin(DjangoQLSearchMixin, ImportExportActionModelAdmin):
  # ...
```

Morali smo da uklonimo `admin.ModelAdmin` osnovnu klasu, jer `ImportExportActionModelAdmin` već nasleđuje od nje. Uključivanje obe klase bi rezultiralo greškom `TypeError`.

Ne zaboravite na uvoz:

```py
from import_export.admin import ImportExportActionModelAdmin
```

Ako želite da model bude samo za izvoz, koristite `ExportActionModelAdmin`.

Ako sada odete na stranicu sa vašim tiketom, trebalo bi da vidite da je dodata akcia izvoza. Testirajte je tako što ćete izabrati nekoliko tiketa i željeni format. Zatim kliknite na "Go".

Zatim možete testirati funkcionalnost uvoza, uvozom upravo izvezene datoteke.

## Stilizovanje admin sajta pomoću `django-admin-interface`

[Sadržaj](#sadržaj)

Prilagođavanje izgleda admin sajta putem zamene šablona može biti nezgodno. Možete slučajno pokvariti stvari, Django administratorski šabloni se mogu promeniti u budućnosti, a održavanje će biti muka.

Bolji pristup stilizaciji vašeg admin sajta je putem paketa `django-admin-interface`. Ovaj paket dolazi sa prelepim, unapred napravljenim temama admin interfejsa i omogućava vam da lako prilagodite različite aspekte vašeg admin sajta, uključujući promenu boja, naslova, favicon, logotipa i još mnogo toga.

Počnite tako što ćete ga instalirati putem pip-a:

```sh
(env)$ pip install django-admin-interface==0.26.0
```

Zatim, dodajte `admin_interface` i `colorfield` ispred `django.contrib.admin` u
`INSTALLED_APPS`:

`# core/settings.py`

```py
INSTALLED_APPS = [
  #...
  "admin_interface",
  "colorfield",
  #...
  "django.contrib.admin",
  #...
]
```

Po želji dodajte:

X_FRAME_OPTIONS = "SAMEORIGIN" # omogućuje korišćenje modala umesto popups
SILENCED_SYSTEM_CHECKS = ["security.W019"] # ignoriše redudantne poruke upozorenja

Migrirajte bazu podataka:

```sh
(env) python manage.py migrate
```

Prikupite statičke datoteke:

```sh
(env) python manage.py collectstatic --clear
```

Pokrenite razvojni server i idite na <http://localhost:8000/secretadmin>. Primetićete da vaš Django admin sajt izgleda modernije i da će postojati odeljak "Administratorski interfejs".

Kliknite na "Administratorski interfejs > Teme" da biste videli sve trenutno instalirane teme. Podrazumevano, trebalo bi da postoji samo jedna tema pod nazivom „Django“. Ako želite, možete instalirati još tri teme putem fiksnih dodataka:

```sh
(env)$ python manage.py loaddata admin_interface_theme_bootstrap.json
(env)$ python manage.py loaddata admin_interface_theme_foundation.json
(env)$ python manage.py loaddata admin_interface_theme_uswds.json
```

Klik na postojeću temu vam omogućava da prilagodite sve prethodno pomenute aspekte.

## Zaključak

[Sadržaj](#sadržaj)

U ovom članku smo obradili mnoge koncepte prilagođavanja Django admin panela. Sada bi trebalo da budete u mogućnosti da primenite ove koncepte u praksi i prilagodite Django admin sajt potrebama vašeg projekta.

Predlažem vam da pogledate i naše ostale članke vezane za Django administratora:

- Dodavanje grafikona u Django pomoću Chart.js
- Podrška za više jezika u Django-u
- Kreiranje prilagođenog korisničkog modela u Django-u

Preuzmite konačni izvorni kod iz `django-admin-customization` GitHub repozitorijuma.
