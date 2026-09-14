
# Prilagodite Django Admin

Django okvir dolazi sa moćnim administrativnim alatom koji se zove admin. Možete ga koristiti  za brzo dodavanje, brisanje ili uređivanje bilo kog modela baze podataka iz web interfejsa. Ali sa malo dodatnog koda, možete da prilagodite Django administratora da podigne svoje administratorske mogućnosti na sledeći nivo.

U ovom vodiču ćete naučiti kako da:

- Dodate atribut u `changelist` objekata modela
- Dodate vezu između objekata modela
- Dodate filtere na listu objekata modela
- Dodate search na listu objekata modela
- Izmenite fome za edit objekata
- Izmenite admin šablone

## Preduslovi

Da biste izvukli maksimum iz ovog vodiča, biće vam potrebno malo poznavanja Djanga, posebno objekata modela. Pošto Django nije deo standardne Python biblioteke, najbolje je da takođe imate neko znanje o pip i pyenv ( ili ekvivalentnom alatu za virtuelno okruženje ).

Isečci koda u ovom vodiču su testirani u odnosu na Django 3.0.7. Svi koncepti su prethodili u Djangu 2.0, tako da bi trebalo da rade u bilo kojoj verziji koju koristite, ali mogu postojati manje razlike.

## Podešavanje Django admina

Django admin obezbeđuje web-bazirani interfejs za kreiranje i upravljanje objektima modela baze podataka. Da biste ga videli u akciji, prvo će vam trebati Django projekat i neki objektni modeli.

Instalirajte Django unutar čistog virtuelnog okruženja:

```sh
$ python -m pip install django
$ django-admin startproject School
$ cd School
$ ./manage.py startapp core
$ ./manage.py migrate
$ ./manage.py createsuperuser
Username: admin
Email address: admin@example.com
Password:
Password (again):
```

- Prvo kreirate novi Django projekat pod nazivom School sa aplikacijom pod   nazivom core.
- Zatim migrirate tabele za autentifikaciju i kreirate administratora.

## Pristup Django admin ekranima

je ograničen na korisnike sa `staff` ili `superuser` pravima, tako da koristite `createsuperuser` komandu da biste kreirali superuser.

Takođe morate da izmenite `school/settings.py` da biste uključili novu aplikaciju pod nazivom core:

`# School/settings.py`

```py
# ...
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "core",    # Add this line
]
```

Direktorijum aplikacije core će početi sa sledećim datotekama unutra:

core/
│
├── migrations/
│   └── __init__.py
├── __init__.py
├── admin.py
├── apps.py
├── models.py
├── tests.py
└── views.py

Zainteresovani ste za dve od ovih datoteka:

1. `models.py` definiše vaše modele baze podataka.
2. `admin.py` registruje vaše modele kod Django admina.

Da biste demonstrirali rezultat, biće vam potrebni neki modeli. Uredite `core/models.py`:

```py
from django.core.validators import MinValueValidator, MaxValueValidator
from django.db import models

class Person(models.Model):
    last_name = models.TextField()
    first_name = models.TextField()
    courses = models.ManyToManyField("Course", blank=True)

    class Meta:
        verbose_name_plural = "People"

class Course(models.Model):
    name = models.TextField()
    year = models.IntegerField()

    class Meta:
        unique_together = ("name", "year", )

class Grade(models.Model):
    person = models.ForeignKey(Person, on_delete=models.CASCADE)
    grade = models.PositiveSmallIntegerField(
        validators=[MinValueValidator(0), MaxValueValidator(100)]
    )
    course = models.ForeignKey(Course, on_delete=models.CASCADE)
```

Ovi modeli predstavljaju učenike i kurseve koje pohađaju u školi. Course ima "name" i "year" u kojoj je ponuđen. "Person" ima "first_name" i "last_name" i može uzeti nulu ili više kurseva. Grade sadrži procentualni rezultat koji je Person dobio na Course.

Imena osnovnih tabela u bazi podataka su malo drugačija od ove, ali su povezana sa modelima prikazanim iznad.

Svaki model koji želite da Django predstavlja u admin interfejsu mora biti registrovan. Ovo radite u `admin.py` datoteci. Modeli iz `core/models.py` su registrovani u odgovarajućoj `core/admin.py` datoteci:

```py
from django.contrib import admin
from core.models import Person, Course, Grade

@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    pass

@admin.register(Course)
class CourseAdmin(admin.ModelAdmin):
    pass

@admin.register(Grade)
class GradeAdmin(admin.ModelAdmin):
    pass
```

Skoro ste spremni da krenete. Kada migrirate svoje modele baze podataka, možete pokrenuti Django razvojni server i videti rezultate:

```sh
$ ./manage.py makemigrations
$ ./manage.py migrate
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, core, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  ...
  Applying core.0001_initial... OK
  Applying core.0002_auto_20200609_2120... OK
  Applying sessions.0001_initial... OK
$ ./manage.py runserver
Watching for file changes with StatReloader
Performing system checks...

System check identified no issues (0 silenced).
Django version 3.0.7, using settings 'School.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

Sada posetite <http://127.0.0.1:8000/admin> da vidite svoj admin interfejs. Od vas će biti zatraženo da se prijavite. Koristite akreditive koje ste kreirali pomoću `createsuperuser` komande za upravljanje.

Početni admin prikaz navodi sve registrovane modele baze podataka.

Sada možete koristiti interfejs za kreiranje objekata u vašoj bazi podataka. Klikom na naziv modela prikazaće se ekran sa spiskom svih objekata u bazi podataka za taj model.

Lista počinje prazna, kao vaša baza podataka. Klikom na Add Person možete da kreirate osobu u bazi podataka. Kada sačuvate, bićete vraćeni na listu Person objekata.

Dobra vest je da imate objekat. Loše vesti, Person object (1) govore id o objektu i ništa više. Podrazumevano, Django administrator prikazuje svaki objekat tako što poziva str(). Ovaj ekran možete učiniti malo korisnijim dodavanjem `__str__()` metode u "Person" klasu u `core/models.py`:

```py
class Person(models.Model):
    last_name = models.TextField()
    first_name = models.TextField()
    courses = models.ManyToManyField("Course", blank=True)

    def __str__(self):
        return f"{self.last_name}, {self.first_name}"
```

Dodavanje Person.__str__() menja prikaz tako da uključuje ime i prezime Person u interfejsu. Možete da osvežite ekran da biste videli promenu.

To je malo bolje! Sada možete videti neke informacije o Person objektu. Dobra je ideja dodati slične metode i objektima Course i Grade objektima:

```py
class Course(models.Model):
    # ...

    def __str__(self):
        return f"{self.name}, {self.year}"

class Grade(models.Model):
    # ...

    def __str__(self):
        return f"{self.grade}, {self.person}, {self.course}"
```

Želite da imate neke podatke u svojoj bazi podataka da biste videli pun efekat vaših prilagođavanja. Možete se malo zabaviti i kreirati sopstvene podatke sada, ili možete preskočiti posao i koristiti uređaj. Proširite okvir ispod da naučite kako da učitate podatke pomoću uređaja.

Sada kada imate neke podatke sa kojima možete da radite, spremni ste da počnete da prilagođavate Django admin interfejs.

## Prilagođavanje Django admina

Pametni ljudi koji su kreirali Django okvir ne samo da su napravili admin, već su to uradili na takav način da ga možete prilagoditi za svoje projekte. Kada ste ranije registrovali PersonAdmin objekat, on je nasledio od admin.ModelAdmin. Većina prilagođavanja koje možete da uradite sa Django adminom se vrši modifikovanjem ModelAdmin, a vi ga sigurno možete modifikovati!

ModelAdmin ima preko trideset atributa i skoro pedeset metoda. Možete koristiti svaki od njih da fino podesite prezentaciju admina i kontrolišete interfejse vaših objekata. Svaka od ovih opcija je detaljno opisana u dokumentaciji.

Povrh svega, admin je napravljen pomoću Django interfejsa šablona. Mehanizam Django šablona vam omogućava da zamenite postojeće šablone, a pošto je admin samo još jedan skup šablona, to znači da možete potpuno promeniti njegov HTML.

Iako je to van okvira ovog uputstva, možete čak i da kreirate više admin sajtova. To može izgledati kao preterano, ali vam omogućava da postanete fensi i definišete različite sajtove za korisnike sa različitim dozvolama.

Django admin je podeljen na tri glavne oblasti:

1. Indeks aplikacije
2. Liste objekata modela
3. Forme za promenu objekata modela

Indeks aplikacije navodi vaše registrovane modele. Lista objekata modela se automatski kreira za svaki registrovani model i navodi objekte za taj model.
Kada dodate ili uredite jedan od tih objekata, to činite pomoću forme za promenu objekta modela.

U prethodnom primeru, indeks aplikacije je pokazao objekte Person, Course i Grade. Klikom na Person prikazuju se liste za Person objekte. Na Person list stranici, klik na Buffy Summers objekat vodi vas do forme za promene da biste uredili Buffy detalje.

### Izmena listchange prikaza pomoću list_display

Implementacija .__str__() je brz način da promenite reprezentaciju objekta Person iz besmislenog stringa u razumljive podatke. Pošto će se ova reprezentacija takođe pojaviti u padajućim menijima i višestrukim odabirima, definitivno želite da je učinite što lakšom za razumevanje.

Stranice sa listama možete da prilagodite na mnogo više načina od obične izmene prikaza stringova objekta. Atribut list_display objekta admin.ModelAdmin određuje koje se kolone prikazuju na listi promena. Ova vrednost je skup atributa objekta koji se modeluje. Na primer, u core/admin.py, izmenite PersonAdmin na sledeći način:

```py
@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    list_display = ("last_name", "first_name")
```

Gornji kod menja vašu Person listu promena da bi prikazao atribute last_name i first_name za svaki Person objekat. Svaki atribut je prikazan u koloni na stranici:

Na dve kolone je moguće kliknuti, što vam omogućava da sortirate stranicu prema podacima kolone. Admin takođe poštuje ordering atribut odeljka Meta:

```py
class Person(models.Model):
    # ...

    class Meta:
        ordering = ("last_name", "first_name")

    # ...
```

Dodavanje ordering atributa će podrazumevano postaviti da svi upiti na Person modelu budu poređani po last_name a potom po first_name. Django će poštovati ovaj podrazumevani redosled u adminu i prilikom preuzimanja objekata.

Tuple list_display može referencirati bilo koji atribut objekta koji se navodi. Takođe može da referencira metod u admin.ModelAdmin. Izmeni PersonAdmin ponovo:

```py
@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    list_display = ("last_name", "first_name", "show_average")

    def show_average(self, obj):
        from django.db.models import Avg
        result = Grade.objects.filter(person=obj).aggregate(Avg("grade"))
        return result["grade__avg"]
```

U gornjem kodu dodajete kolonu adminu koja prikazuje prosek ocena svakog učenika. show_average() se poziva jednom za svaki objekat prikazan na listi.

Parametar obj je objekat za red koji se prikazuje. U ovom slučaju, koristite ga za ispitivanje odgovarajućih Grade objekata za učenika, sa prosečnim odgovorom preko Grade.grade. Rezultate možete videti ovde:

Imajte na umu da prosečnu ocenu zaista treba izračunati u Person objektu modela. Verovatno ćete želeti podatke negde drugde, ne samo u Django administratoru.
Ako ste imali takav metod, mogli biste ga dodati atributu list_display. Primer ovde pokazuje šta možete da uradite u ModelAdmin objektu, ali to verovatno nije najbolji izbor za vaš kod.

Podrazumevano se sortiraju samo one kolone koje su atributi objekta. show_average() nije. To je zato što sortiranje vrši osnovni QuerySet, a ne prikazani rezultati. U nekim slučajevima postoje načini za sortiranje ovih kolona , ali to je van okvira ovog uputstva.

Naslov kolone je zasnovan na nazivu metode. Možete da promenite naslov dodavanjem atributa metodu:

```py
def show_average(self, obj):
    result = Grade.objects.filter(person=obj).aggregate(Avg("grade"))
    return result["grade__avg"]

show_average.short_description = "Average Grade"
```

Django vas podrazumevano štiti od HTML-a u stringovima u slučaju da je string od korisničkog unosa. Da bi Django uključio HTML, morate koristiti `format_html()`:

```py
def show_average(self, obj):
    from django.utils.html import format_html

    result = Grade.objects.filter(person=obj).aggregate(Avg("grade"))
    return format_html("<b><i>{}</i></b>", result["grade__avg"])

show_average.short_description = "Average"
```

`show_average()` sada ima prilagođeni naslov, "Average" i formatiran je da bude u kurzivu.

Nažalost, Django još uvek nije dodao podršku za f-string format_html(), tako da ste zaglavili sa `str.format()` sintaksom.

### Pružanje veza do stranica drugih objekata na changelist prikazu

Sasvim je uobičajeno da objekti upućuju na druge objekte korišćenjem stranih ključeva. Možete pokazati list_display na metodu koja vraća HTML vezu. Unutar `core/admin.py`, izmenite "CourseAdmin" klasu na sledeći način:

```py
from django.urls import reverse
from django.utils.http import urlencode

@admin.register(Course)
class CourseAdmin(admin.ModelAdmin):
    list_display = ("name", "year", "view_students_link")

    def view_students_link(self, obj):
        count = obj.person_set.count()
        url = (
            reverse("admin:core_person_changelist")
            + "?"
            + urlencode({"courses__id": f"{obj.id}"})
        )
        return format_html('<a href="{}">{} Students</a>', url, count)

    view_students_link.short_description = "Students"
```

Ovaj kod uzrokuje da Course lista promena ima tri kolone:

1. Naziv kursa
2. Godina u kojoj je kurs ponuđen
3. Link koji prikazuje broj studenata na kursu

Kada kliknete na 2 učenika, Django vas vodi na Person stranicu sa listom promena sa primenjenim filterom. Filtrirana stranica prikazuje samo one učenike u Psych 101, Buffy i Villow. Xander nije stigao na univerzitet.

Primer koda koristi reverse() za traženje URL adrese u Django adminu. Možete potražiti bilo koju stranicu admina koristeći sledeću konvenciju imenovanja:

```py
"admin:%(app)s_%(model)s_%(page)"
```

Ova struktura imena se razlaže na sledeći način:

- admin imenski prostor.
- app naziv aplikacije.
- mdel objekat modela.
- page tip Django admin stranice.

Za `view_students_link()` gornji primer, koristite `admin:core_person_changelist` da biste dobili referencu na stranicu sa listom promena objekta Person u core aplikaciji.

Evo dostupnih URL adresa:

 Strana     | URL ime                        | Svrha
 -----------|--------------------------------|------------------------------------
 changelist | %(app)s\_%(model)s\_changelist | Stranica Lista objekata modela.
 add        | %(app)s\_%(model)s\_add        | Stranica Forme za kreiranje objekta modela.
 history    | %(app)s\_%(model)s\_history    | Stranica istorije promena objekta.(object_id)
 delete     | %(app)s\_%(model)s\_delete     | Stranica za brisanje objekta modela.(object_id)
 change     | %(app)s\_%(model)s\_change     | Stranica za uređivanje objekta modela.(object_id)

Možete da filtrirate stranicu sa listom dodavanjem stringa upita u URL. Ovaj string upita menja QuerySet korišćeni za popunjavanje stranice. U gornjem primeru, string upita

```py
"?courses__id={obj.id}"
```

filtrira Person listu samo na one objekte koji imaju odgovarajuću vrednost u "Person.course".

Ovi filteri podržavaju pretragu po poljima QuerySet pomoću dvostrukih donjih crta ( `__` ). Možete pristupiti atributima povezanih objekata kao i koristiti modifikatore filtera kao što su `__exact` i `__startswith`.

Možete pronaći sve detalje o tome šta možete da postignete pomoću `list_display` atributa u Django admin dokumentaciji.

### Dodavanje filtera na listchange stranu

Pored filtriranja podataka na listi promena preko URL adrese za pozivanje, možete da filtrirate i pomoću ugrađenog vidžeta. Dodajte `list_filter` atribut objektu "CourseAdminu" `core/admin.py`:

```py
@admin.register(Course)
class CourseAdmin(admin.ModelAdmin):
    list_display = ("name", "year", "view_students_link")
    list_filter = ("year", )
    # ...
```

Prikazaće `list_filter` novi odeljak na stranici sa listom veza. U ovom slučaju, veze filtriraju stranicu po godinama. Lista filtera se automatski popunjava vrednostima year koje koriste "Course" objekti u bazi podataka:

Klikom na godinu sa desne strane promeniće se lista tako da uključuje samo "Course" objekte sa tom "year" vrednošću. Takođe možete da filtrirate na osnovu atributa povezanih objekata koristeći `__` sintaksu za pretragu polja. Na primer, možete da filtrirate "GradeAdmin" objekte prema `course__year`, prikazujući Grade objekte samo za određenu godinu kurseva.

Ako tražite veću kontrolu nad vašim filtriranjem, možete čak i da kreirate objekte filtera koji određuju atribute pretraživanja i odgovarajući QuerySet.

### Dodavanje pretrage na listchange stranu

Filteri nisu jedini način da se smanji količina podataka na ekranu. Django admin takođe podržava pretragu kroz `search_fields opciju, koja dodaje okvir za pretragu. Postavljate ga pomoću tuple koji sadrži imena polja koja će se koristiti za konstruisanje upita za pretragu u bazi podataka.

Sve što korisnik unese u polje za pretragu koristi se u `OR` klauzuli polja koja filtriraju `QuerySet`. Podrazumevano, svaki parametar pretrage je okružen `%` znakovima, što znači da ako tražite, tada će se u rezultatima pojaviti obekti modela sa unešenom reči. Možete biti precizniji tako što ćete navesti `__` modifikator u polju za pretragu.

Uredite "PersonAdmin" u `core/admin.py` na sledeći način:

```py
@admin.register(Person)

class PersonAdmin(admin.ModelAdmin):
    search_fields = ("last_name__startswith", )
```

U gornjem kodu, pretraga se zasniva na prezimenu. Modifikator `__startswith` ograničava pretragu na prezimena koja počinju parametrom pretrage.

Kad god se vrši pretraga na stranici sa listom objekata, Django admin poziva metod vašeg `admin.ModelAdmin` objekta `get_search_results()`. Vraća `QuerySet` sa rezultatima pretrage. Možete fino podesiti pretrage tako što ćete preopteretiti metod i promeniti `QuerySet`. Više detalja možete pronaći u dokumentaciji.

### Promena načina na koji se uređuju modeli

Možete prilagoditi više od samo stranice sa listom objekata. Ekrani koji se koriste za dodavanje ili promenu objekta zasnovani su na `ModelForm`. Django automatski generiše obrazac na osnovu modela koji se uređuje.

Možete da kontrolišete koja polja su uključena, kao i njihov redosled, izmenom opcije fields. Izmenite svoj "PersonAdmin" objekat, dodajući fields
atribut:

```py
@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    fields = ("first_name", "last_name", "courses")
    # ...
```

Stranice `AddView` i `ChangeView` za "Person" sada stavljaju "first_name" atribut ispred "last_name" atributa iako sam model navodi obrnuto.

`ModelAdmin.get_form()` je odgovoran za kreiranje ModelForm za vaš objekat. Možete zameniti ovaj metod da biste promenili obrazac. Dodajte sledeći kod u metod u PersonAdmin:

```py
def get_form(self, request, obj=None, **kwargs):
    form = super().get_form(request, obj,**kwargs)
    form.base_fields["first_name"].label = "First Name (Humans only!):"
    return form
```

Sada, kada se prikaže stranica za dodavanje ili promenu, oznaka polja "first_name" će biti prilagođena.

Promena oznake možda neće biti dovoljna da spreči vampire da se registruju kao studenti. Ako vam se ne sviđa ono `ModelForm` što je Django administrator kreirao za vas, onda možete koristiti form atribut da registrujete prilagođeni obrazac. Napravite sledeće dodatke i izmene u `core/admin.py`:

```py
from django import forms

class PersonAdminForm(forms.ModelForm):
    class Meta:
        model = Person
        fields = "__all__"

    def clean_first_name(self):
        if self.cleaned_data["first_name"] == "Spike":
            raise forms.ValidationError("No Vampires")

        return self.cleaned_data["first_name"]

@admin.register(Person)
class PersonAdmin(admin.ModelAdmin):
    form = PersonAdminForm
# ...
```

Gornji kod nameće dodatnu proveru valjanosti na "Person" stranicama za dodavanje i promenu. `ModelForm` objekti imaju bogat mehanizam validacije. U ovom slučaju, "first_name" polje se proverava u odnosu na ime "Spike". `ValidationError` sprečava studente sa ovim imenom da se registruju.

Promenom ili zamenom ModelForm objekta možete u potpunosti da kontrolišete izgled i validaciju stranica koje koristite za dodavanje ili promenu stranica objekata.

### Zaobilaženje Django admin šablona

Django programeri su implementirali admina koristeći mehanizam Django šablona. Ovo im je malo olakšalo posao, ali vam takođe koristi jer vam omogućava da zaobiđete šablone. Možete u potpunosti da prilagodite admina tako što ćete promeniti šablone koji se koriste za prikazivanje stranica.

Možete videti sve šablone koji se koriste u adminu tako što ćete pogledati unutar Django paketa u vašem virtuelnom okruženju:

```sh
.../site-packages/django/contrib/admin/templates/
│
├── admin/
│   ├── auth/
│   │   └── user/
│   │       ├── add_form.html
│   │       └── change_password.html
│   ├── edit_inline/
│   │   ├── stacked.html
│   │   └── tabular.html
│   ├── includes/
│   │   ├── fieldset.html
│   │   └── object_delete_summary.html
│   ├── widgets/
│   │   ├── clearable_file_input.html
│   │   ├── foreign_key_raw_id.html
│   │   ├── many_to_many_raw_id.html
│   │   ├── radio.html
│   │   ├── related_widget_wrapper.html
│   │   ├── split_datetime.html
│   │   └── url.html
│   ├── 404.html
│   ├── 500.html
│   ├── actions.html
│   ├── app_index.html
│   ├── base.html
│   ├── base_site.html
│   ├── change_form.html
│   ├── change_form_object_tools.html
│   ├── change_list.html
│   ├── change_list_object_tools.html
│   ├── change_list_results.html
│   ├── date_hierarchy.html
│   ├── delete_confirmation.html
│   ├── delete_selected_confirmation.html
│   ├── filter.html
│   ├── index.html
│   ├── invalid_setup.html
│   ├── login.html
│   ├── object_history.html
│   ├── pagination.html
│   ├── popup_response.html
│   ├── prepopulated_fields_js.html
│   ├── search_form.html
│   └── submit_line.html
└── registration/
    ├── logged_out.html
    ├── password_change_done.html
    ├── password_change_form.html
    ├── password_reset_complete.html
    ├── password_reset_confirm.html
    ├── password_reset_done.html
    ├── password_reset_email.html
    └── password_reset_form.html
```

Django šablonski mehanizam ima definisan redosled za učitavanje šablona. Kada učita šablon, koristi prvi šablon koji odgovara imenu. Možete zameniti admin šablone koristeći istu strukturu dira i imena datoteka.

Administratorski šabloni dolaze u dva direktorijuma:

1. admin je za stranice objekata modela.
2. registration je za promenu lozinke i prijavljivanje i odjavljivanje.

Da biste prilagodili stranicu za odjavu, potrebno je da zamenite pravu datoteku. Relativna putanja koja vodi do datoteke mora biti ista kao ona koja se zamenjuje. Datoteka koja vas zanima je `registration/logged_out.html`. Počnite tako što ćete kreirati direktorijum u "School" projektu:

```sh
mkdir -p templates/registration
```

Sada recite Djangu o vašem novom direktorijumu šablona unutar vaše `School/settings.py` datoteke. Potražite `TEMPLATES` direktivu i dodajte direktorijum na `DIR` listu:

`# School/settings.py`

```py
# ...
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",

        # Add the templates directory to the DIR option:
        "DIRS": [os.path.join(BASE_DIR, "templates"), ],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
    },
]
```

Mehanizam šablona pretražuje direktorijume u DIR opciji pre direktorijuma aplikacije, tako da će umesto toga biti učitano sve sa istim imenom kao admin šablon. Da biste videli ovo u akciji, kopirajte `logged_out.html` datoteku u svoj `templates/registration` direktorijum, a zatim je izmenite:

```html
{% extends "admin/base_site.html" %}
{% load i18n %}

{% block breadcrumbs %}<div class="breadcrumbs"><a href="{% url 'admin:index' %}">{% 
trans 'Home' %}</a></div>{% endblock %}

{% block content %}

<p>You are now leaving Sunnydale</p>

<p><a href="{% url 'admin:index' %}">{% trans 'Log in again' %}</a></p>

{% endblock %}
```

Sada ste prilagodili stranicu za odjavu. Ako kliknete na `LOG OFF`, videćete prilagođenu poruku.

Django administratorski šabloni su duboko ugnežđeni i nisu baš intuitivni, ali imate potpunu kontrolu nad njihovom prezentacijom ako vam je potrebna. Neki paketi, uključujući Grappelli i Django Admin Bootstrap, u potpunosti su zamenili Django administratorske šablone da bi promenili njihov izgled.

Django Admin Bootstrap još uvek nije kompatibilan sa Djangom 3, a Grappelli je tek nedavno dodao podršku, tako da možda i dalje ima nekih problema. Uz to, ako želite da vidite moć nadjačavanja administratorskih šablona, pogledajte te projekte!

## Zaključak

Django administrator je moćan ugrađeni alat koji vam daje mogućnost da kreirate, ažurirate i brišete objekte u vašoj bazi podataka koristeći web interfejs. Možete da prilagodite Django administratora da radi skoro sve što želite.

Ovaj vodič dodiruje samo površinu. Količina konfiguracije koju možete da uradite da biste prilagodili Django administratora je zapanjujuća. Možete dublje zaroniti u dokumentaciju da biste istražili teme kao što su ugrađeni obrasci, više admin lokacija, masovno uređivanje, automatsko dovršavanje i još mnogo toga. Srećno kodiranje!
