
# Django šabloni - Ugrađene oznake i filteri

Django je moćan okvir za kreiranje web aplikacija u Python-u. NJegove karakteristike uključuju modele baza podataka, URL adrese za rutiranje, autentifikaciju, upravljanje korisnicima, administrativne alate i jezik šablona. Možete da sastavite HTML za višekratnu upotrebu koji se menja na osnovu podataka koje prosledite jeziku šablona. Django šabloni koriste oznake i filtere da definišu mini-jezik koji je sličan Python-u—ali nije Python.

Django šablone ćete upoznati kroz oznake i filtere koje koristite za sastavljanje HTML-a za višekratnu upotrebu.

U ovom vodiču ćete naučiti kako da:

- Napišete, kompajlirajte i renderujte Django šablon
- Koristite `render()` prečicu u pogledima da biste brzo koristili šablone
- Koristite oznake šablona za uslove i petlje u svojim šablonima
- Kreirate šablone za višekratnu upotrebu sa nasleđivanjem i uključivanjem
- Izmenite prezentaciju svojih podataka pomoću filtera šablona
- Kreirate Django projekat

Da biste eksperimentisali sa Django šablonima, biće vam potreban projekat da biste se mogli igrati sa kodom. Gradićete filmsku palatu:  najmanji, najjednostavniji filmski web-sajt na svetu.

Za detaljniji primer pokretanja novog projekta, možete pročitati `Početak rada sa Djangom, 1. deo: Napravite aplikaciju za portfolio`.

Django nije deo standardne Python biblioteke, tako da ćete prvo morati da ga instalirate. Kada radite sa bibliotekama trećih strana, trebalo bi da koristite virtuelno okruženje.

Kada imate virtuelno okruženje, pokrenite sledeće komande da biste krenuli:

```sh
python -m pip install django==3.2.5
django-admin startproject moviepalace
cd moviepalace
python manage.py startapp moviefacts
```

- Linija 1. Instalira Django u vaše virtuelno okruženje koristeći pip.
- Linija 2., Django-admin komanda kreira novi Django projekat pod nazivom `moviepalace`.
  Django projekat se sastoji od aplikacija u kojima živi vaš kod.
- Linija 4. kreira aplikaciju pod nazivom `moviefacts`.

Skoro ste spremni da krenete. Poslednji korak je da kažete Djangu o vašoj novokreiranoj `moviefacts` aplikaciji. To radite tako što ćete urediti `moviepalace/settings.py` datoteku i dodati `moviefacts` na listu `INSTALLED_APPS`:

```py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "moviefacts",
]
```

Kada ste registrovali `moviefacts` kao aplikaciju, sada možete napisati pogled koji sadrži šablon.

## Priprema za korišćenje Django šablona

Django je kreiran u novinama da bi pomogao u brzoj izgradnji web aplikacija. Jedan od ciljeva okvira je bio da odvoji poslovnu logiku od logike prezentacije.

Web dizajneri, a ne Python programeri, često su radili na razvoju HTML-a u novinama. Zbog toga su programeri odlučili da ne dozvole izvršavanje Python-a u jeziku šablona. Ova odluka je pojednostavila ono što je dizajnerima trebalo da znaju i zaštite svoj kod iz bezbednosnih razloga. Krajnji rezultat je bio poseban mini-jezik. Ovaj pristup je u suprotnosti sa PHP pristupom, gde je kod direktno ugrađen u HTML.

### Sastavljanje i prikazivanje Django šablona

Django šabloni vam omogućavaju da dinamički menjate izlazni sadržaj unutar konteksta prikazivanja. Možete zamisliti šablone kao obrazac pisma, gde sadržaj pisma uključuje mesta gde se informacije mogu umetnuti. Možete pokrenuti proces renderovanja više puta sa različitim podacima i svaki put dobiti različite rezultate.

Django obezbeđuje `Template` i `Context` klase koje predstavljaju šablon stringova koji se renderuje i podatke koji se koriste tokom generisanja. Klasa `Context` je omotač za `dict` i obezbeđuje parove ključ-vrednost za popunjavanje generisanog sadržaja. Rezultat renderovanog šablona može biti bilo koji tekst, ali je često HTML. Django je ipak web okvir.

Vreme je da napravite svoj prvi šablon. Da biste ga videli u akciji, prvo će vam trebati pogled. Dodajte sledeći kod u `moviefacts/views.py`:

```py
# moviefacts/views.py
from django.http import HttpResponse
from django.template import Context, Template

def citizen_kane(request):
    content = """{{movie}} was released in {{year}}"""
    template = Template(content)
    context = Context({"movie": "Citizen Kane", "year": 1941})

    result = template.render(context)
    return HttpResponse(result)
```

U ovom pogledu, vidite neke od glavnih koncepata koji čine Django šablonski jezik:

- Linija 6. sadrži reference na movie i year. Ovo je slično Python f-stringu. Duple zagrade, ili
  brkovi, označavaju stavke koje Django zamenjuje kada renderuje šablon.
- Linija 7. instancira Template objekat prosleđivanjem stringa koji specificira šablon.
- Linija 8. stvara Context objekat popunjavanjem rečnikom. Objekat Context sadrži sve podatke
  dostupne šablonu kada ga Django renderuje. Šablon sadrži dve stavke koje treba zameniti:
  {{movie}} sa "Citizen Kane" i {{year}} sa 1941.
- Linija 10. sadrži poziv metodi .render() koja generiše rezultat.
- Linija 11. vraća sadržaj za prikazivanje umotan u HttpResponse objekat.

Da biste ovo testirali, moraćete da ovaj pogled učinite dostupnim u pregledaču, tako da ćete morati da dodate rutu. Izmenite `moviepalace/urls.py` na sledeći način:

```py
# moviepalace/urls.py
from django.urls import path
from moviefacts import views

urlpatterns = [
    path("citizen_kane/", views.citizen_kane),
]
```

Uverite se da koristite virtuelno okruženje u kojem ste instalirali Django, a zatim pokrenite Django razvojni server da vidite rezultat:

```sh
python manage.py runserver
```

Pokrenite svoj pogled tako što ćete posetiti <http://127.0.0.1:8000/citizen_kane/>.

Objekat `HttpResponse` vraća sadržaj kao HTML, ali pošto string ne sadrži nikakve oznake, vaš pretraživač će ovo tretirati kao da je to tekst unutar oznake `<body>`. Vraća loše oblikovan HTML, ali je za sada dovoljno dobar za igru. Ako je sve prošlo dobro, trebalo bi da vidite da je vaš šablon prikazan sa podacima iz vašeg konteksta:

```html
Citizen Kane was released in 1941
```

Vaš šablon je renderovan, a Django je zamenio promenljive `movie` i `year` sa `Citizen Kane` i `1941`.

### Konfigurisanje Djanga za učitavanje datoteka šablona

U web aplikaciji, vaša najverovatnija upotreba šablona će biti za ispis HTML-a - puno i puno HTML-a. Jezik šablona Django je napravljen da pojednostavi ovaj proces. Za razliku od prethodnog primera, obično ne koristite stringove šablona u svojim pogledima. Umesto toga, učitavate šablone iz drugih datoteka.

Da biste učitali šablon sa diska, prvo morate da kažete Djangu gde da ga pronađe. Unutar `moviepalace/settings.py`,  izmenite `"DIRS"` vrednost u `TEMPLATES`:

```py
TEMPLATES = [{
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [
            BASE_DIR / "templates",
        ],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
...
```

U podrazumevanoj `settings.py` datoteci koju je `django-admin` komanda kreirala, `DIRS` lista je prazna. Django će sada tražiti šablone u direktorijumu pod nazivom `moviepalace/templates`. Imajte na umu da Django koristi strukturu dvostrukog dira za sopstvenu konfiguraciju. Na primer, `settings.py` se nalazi u `moviepalace/moviepalace`. Dir za šablone treba da bude u korenu projekta, a ne u diru konfiguracije.

Kada `APP_DIRS` je na liniji 7. `True`, Django će takođe tražiti šablone u poddiru aplikacije. Django očekuje da šabloni aplikacija budu unutar direktorijuma nazvanog `templates` ispod dira aplikacije.

Kako odlučujete gde ćete čuvati svoje šablone? Ako će vaša aplikacija biti višekratna za upotrebu u drugim projektima i ima šablone koji su specifični za nju, zadržite šablone uz aplikaciju. U suprotnom, držite svoje šablone zajedno u diru šablona projekta. Pogledajte lekciju o Django strukturi dvostrukih dirova u kursu `Početak rada sa Djangom` za više informacija o tome kako da strukturirate svoje Django projekte.

Nakon što izvršite promenu u `moviepalace/settings.py` datoteci, ne zaboravite da kreirate direktorijum `templates`:

```sh
$ pwd
/home/realpython/moviepalace
$ mkdir templates
```

Sa završenom konfiguracijom i kreiranim dirom, sada ste spremni da učitate šablon iz datoteke.

### Učitavanje Django šablona iz datoteka

Hajde da ponovo izgradimo iskustvo `Citizen Kane`-a koristeći datoteku. Napravite `templates/simple.txt` i dodajte string šablona koji se koristi u vašem `citizen_kane()` pogledu:

```html
{{movie}} was released in {{year}}
```

Možete napisati kod za učitavanje datoteke kao string, napraviti Template objekat i uraditi isto što ste uradili u `citizen_kane()` pogledu, ili možete koristiti `render()` prečicu, koja sve to radi za vas. Dodajte sledeće u svoju `moviefacts/views.py` datoteku:

```py
# moviefacts/views.py
from django.shortcuts import render

def casablanca(request):
    return render(
        request, "simple.txt", {"movie": "Casablanca", "year": 1942}
    )
```

Sa novim pogledom na mestu, ne zaboravite da dodate rutu u `moviepalace/urls.py`:

```py
# moviepalace/urls.py
from django.urls import path
from moviefacts import views

urlpatterns = [
    path("citizen_kane/", views.citizen_kane),
    path("casablanca/", views.casablanca),
]
```

Poseta <http://127.0.0.1:8000/casablanca/> bi trebalo da proizvede slične rezultate kao u primeru građanina Kejna:

```txt
Casablanca was released in 1942
```

Prečica render() je uobičajen način prikazivanja šablona. Direktno korišćenje `Template` objekta se radi samo u retkim slučajevima, na primer kada želite da date moć šablona svojim korisnicima.

Primer `Template` direktnog korišćenja može biti kada se dozvoli korisniku da unese pismo obrasca.

Pismo obrasca može da sadrži promenljive kao što je primalac pisma. Dozvoljavajući korisniku da koristi Django šablon, možete iskoristiti prednosti ugrađenog mehanizma zamene promenljivih za zamenu imena primaoca.

### Izbor jezika šablona

Django podržava više šablonskih mehanizama. Isporučuje se sa dva:

- **Jezik Django šablona**: originalni Django šablonski jezik i onaj o kome učite u ovom vodiču
- **Jinja2**: nekadašnja biblioteka samo treće strane koja je sada uključena u Django, ali je izvan opsega ovog uputstva

Možete da promenite koji se šablon koristi izmenom TEMPLATES vrednosti u `moviepalace/settings.py`:

```py
TEMPLATES = [
    {
        "BACKEND": "django.template.backends.django.DjangoTemplates",
        "DIRS": [
            BASE_DIR / "templates",
        ],
        "APP_DIRS": True,
        "OPTIONS": {
            "context_processors": [
                "django.template.context_processors.debug",
                "django.template.context_processors.request",
                "django.contrib.auth.context_processors.auth",
                "django.contrib.messages.context_processors.messages",
            ],
        },
```

Podešavanje `BACKEND` na liniji 3. je mesto gde određujete svoj mehanizam za renderovanje. Možete izabrati ili Django šablonski mehanizam ili Jinja2 motor tako što ćete promeniti `BACKEND` naziv modula sa dot putanjom:

- `django.template.backends.django.DjangoTemplates`
- `django.template.backends.jinja2.Jinja2`

Dostupni su i šabloni nezavisnih proizvođača. Njihovo korišćenje zahteva instaliranje biblioteke pip i promenu `BACKEND` vrednosti u ime putanje.

Ostatak ovog vodiča će se fokusirati isključivo na originalnu pozadinu Django šablona da bi vam pružio solidnu osnovu za kreiranje HTML-a u vašim Django projektima.

## Učenje o Django šablonima, oznakama i filterima

Do sada ste videli šablone koji sadrže jednostavnu zamenu promenljivih. Jezik šablona Django ide mnogo dublje od toga. Dobijate pristup mnogim strukturama i kontrolama na koje ste navikli u Python-u, osim u okviru njegovog sopstvenog mini-jezika.

### Oznake i filteri Django šablona

Jezik šablona Django ima tri načina kontrole onoga što se renderuje: vrednosti, oznake i filteri. Sve što stavite u šablon što nije jedan od ova tri biće prikazano onako kako ste ga napisali. U ovom vodiču ćete proći kroz tri glavna dela jezika šablona:

1. Interpretirani podaci, koje beležite dvostrukim zagradama, `{{ value }}`
2. Oznake koje beležite zagradama i procentima, `{% tag_name %}`
3. Filteri, koji menjaju interpretirane podatke i primenjujete pomoću pipe operatora (`|`), kao u `{{ value | filter }}`

Kao što ste videli u prethodnom odeljku, kada Django renderuje šablon u tekst, on koristi poseban rečnik koji se zove `Context`. Kontekst je stanje sa kojim se šablon priakzuje. Pored toga što sadrži interpretirane podatke koji će se prikazati sa dvostrukim zagradama, sadržaj objekta `Context` se može koristiti za donošenje logičkih odluka.

Oznake su poput ključnih reči i funkcija jezika šablona. U Python-u, ključne reči i funkcije obezbeđuju tok kontrole i alate na kojima je izgrađen vaš kod. Slično, Django-ove ugrađene oznake obezbeđuju nasleđivanje, uslovne operacije, petlje, komentare i upravljanje tekstom. Na primer:
`{% lorem %}` oznaka se zamenjuje uzorkom teksta lorem ipsum kada se prikaže.

Django filteri rade unutar dvostrukih zagrada i omogućavaju vam da menjate prezentaciju podataka koje pogledujete. Na primer, date filter formatira objekat `datumtime` slično onome kako `strftime()` funkcioniše u Python-u. Ako je Context rečnik sadržao datetime objekat pod nazivom today, `{{ today | date:"Y"}}` primenjuje date filter na today, vraćajući godinu.

Ostatak ovog vodiča će vas voditi kroz uobičajene oznake i filtere, dajući primere kako ih koristite.

### Nasleđivanje i uključivanje šablona

Postoji mnogo šablona unutar HTML-a. Većina web lokacija ima zajednički izgled koji deli svaka stranica. Svaka prikazana stranica često ponavlja isto zaglavlje i podnožje, što uključuje iste stilove i često uključuje iste JavaScript datoteke za metriku i merenje. Ručno ponavljanje ovoga na svakoj stranici značilo bi da bi unošenje promene zahtevalo mnogo posla. Nasleđivanje šablona i uključivanje su spas!

Postoje dva načina sastavljanja delova Django šablona zajedno. Nasleđivanje funkcioniše kao nasleđivanje klasa u Python-u, sa šablonima koji zamenjuju vrednosti njihovih roditelja. Uključivanje ubacuje sadržaj u šablon iz drugog šablona. Ovo je slično `#include` direktivi u programskom jeziku C.

Da biste ih videli u akciji, kreirajte `templates/base.html`:

```html
<!-- templates/base.html -->
<html>
<body>
  {% block heading %}
    <h1>Movie Palace</h1>
  {% endblock heading %}

  {% block content %}
  {% endblock content %}
</body>
</html>
```

Ova deklaracija je kao osnovna klasa u objektno orijentisanom programiranju. Možete je koristiti samostalno ili možete podesiti druge šablone za nasleđivanje od njega. Ako se ovaj šablon prikaže kakav jeste, sadržaj oznaka `{% block %}` će se pogledati onakav kakav jeste. Ovo pruža korisnu podrazumevanu vrednost u slučaju kada dete ne nadjača blok.

U `base.html`, postoji naslov koji kaže `<h1>Movie Palace</h1>`, a nema sadržaja. Magija se dešava kada drugi šablon nasledi ovaj. U podređenom šablonu, opciono možete zameniti bilo koji od definisanih blokova. Uobičajena praksa je popunjavanje bloka `content` sadržajem za stranicu dok ostavljanje šablonskog HTML-a unutar ove osnovne datoteke. Napravite šablon koji nasleđuje od `base.html` pozvanog `templates/falcon.html`:

```html
<!-- templates/falcon.html -->
{% extends "base.html" %}

{% block heading %}
  {{block.super}}
  <h2>The stuff that dreams are made of</h2>
{% endblock heading %}

{% block content %}
  <p>
    {% include "simple.txt" %}
  </p>
{% endblock content %}
```

Datoteka `falcon.html` pokazuje dve stvari: nasleđivanje i uključivanje. Nasleđujete roditeljski fajl pomoću `{% extends %}` oznake. Ovde `falcon.html` nasleđuje od `base.html`. Django prikazuje `base.html` datoteku u potpunosti, osim kada dete `falcon.html` zameni blok. Na primer, content odeljak u `falcon.html` zamenjuje blok sa istim imenom u `base.html`.

Unutar sekcije block definisana je posebna promenljiva: `{{block.super}}`. Ova promenljiva sadrži sve što je bilo u roditeljskom bloku.

- U liniji 5., roditeljski blok unosi `<h1>Movie Palace</h1>` naslov,
- dok falcon.html dodaje `<h2>` blok u liniju 6..

Pored nasleđivanja, možete uraditi i inkluziju. Na liniji 11., `{% include %}` oznaka ubacuje sadržaj `simple.txt`. Ovo vam omogućava da ponovo koristite isečke HTML-a. Ovde se ponovo koristi šablon koji je prvobitno definisan za `casablanca()` pogled.

Da biste sve ovo videli na delu, biće vam potreban novi pogled da biste ih pogledali. Napravite sledeće u `moviefacts/views.py`:

```py
# moviefacts/views.py
def maltese_falcon(request):
    return render(
        request,
        "falcon.html",
        {"movie": "Maltese Falcon", "year": 1941},
    )
```

Ovaj pogled je skoro identičan `casablanca()`, osim što prikazuje `falcon.html` i prosleđuje odgovarajuće podatke filma. Ažurirajte svoju `moviepalace/urls.py` datoteku putanjom za `maltese_falcon` pogled, a zatim posetite stranicu:

Zamenili ste zaglavlje, uključili ste `simple.txt`, a Django je sve to prikazao sa podacima filma "Malteški soko".

Unutar `falcon.html`, `{% extends %}` oznaka čvrsto kodira ime roditeljskog šablona. Ovo je najčešći slučaj, ali se roditeljski šablon umesto toga može imenovati u promenljivoj konteksta. Ovaj trik vam može omogućiti da koristite istu podređenu stranicu i u prijavljenom i u javnom stanju, menjajući roditeljski šablon na osnovu situacije.

Šablon `falcon.html` koristi samo jedan nivo nasleđivanja, ali niste ograničeni samo na to. Unuci mogu naslediti od dece, koja zauzvrat nasleđuju od roditelja. Možete imati onoliko slojeva koliko je potrebno da organizujete svoj izlaz.

## Istraživanje zajedničkih oznaka šablona

Možete da iskoristite prednosti preko dvadeset pet ugrađenih oznaka koje Django 3.2.5 uključuje. Ove oznake su poput ključnih reči i funkcija jezika šablona. Omogućavaju vam da radite uslovnu logiku, petlje, nasleđivanje, manipulaciju tekstom i još mnogo toga.

### Uslovni kod

U Python-u koristite ključne reči `if`, `elif` i `else` ključne reči za uslovno izvršavanje koda. U Django šablonima koristite oznake da biste postigli istu stvar.

Da biste videli ove oznake u akciji, biće vam potreban pogled sa malo više podataka. Uredite `moviefacts/views.py` datoteku i dodajte sledeću funkciju:

```py
# moviefacts/views.py
def psycho(request):
    data = {
        "movie": "Psycho",
        "year": 1960,
        "is_scary": True,
        "color": False,
        "tomato_meter": 96,
        "tomato_audience": 95,
    }
    return render(request, "psycho.html", data)
```

Dodajte odgovarajuću rutu za psycho u `moviepalace/urls.py`, a zatim kreirajte `templates/psycho.html`:

```html
<!-- templates/psycho.html -->
{% extends "base.html" %}

{% block content %}
  <p>
    {{movie}} was released in {{year}}.  It was {% if not is_scary %}not
    {% endif %}scary.
  </p>

  <p>
    {% if color %}
      Color
    {% else %}
      Black and white 
    {% endif %}
  </p>

  <p>
    {% if THX %}
      Sound was awesome
    {% endif %}
  </p>

  <p>
    {% if tomato_meter > tomato_audience %}
      Critics liked it better than audience
    {% elif tomato_meter == tomato_audience %}
      Critics liked it the same as audience
    {% else %}
      The audience liked it better than the critics
    {% endif %}
  </p>

  <p>Copyright <b>MoviePalace</b></p>
{% endblock content %}
```

Oznaka `{% if %}` funkcioniše kao `if` ključna reč Python-a. Ako je testirani uslov `True`, onda se blok prikazuje.

- Linija 6. do 7. pokazuju upotrebu `inline if`, sa `is_scary` vrednošću negiranom not ključnom reči.
- Linije 11. do 15. pokazuju upotrebu višelinijskog `if`-uslova. Ovaj blok uslova takođe sadrži `{% else %}` klauzulu. Međutim, za razliku od pisanja `if` izjava u Python-u, ne zaboravite na završnu `{% endif %}` oznaku.
- Linija 19. proverava `THX` promenljivu, koju Django neće pronaći. Razmatra se promenljiva koja je u kontekstu `False`, tako da se ovaj blok o kvalitetu zvuka ne prikazuje.
- Linije 25. do 31. prakazuje upotrebu uslova sa Bulovim operatorima poređenja. Možete koristiti iste vrste operatora koje biste koristili u Python-u unutar oznaka.

Ako pokrenete razvojni server i posetite pogled, primetićete da su uslovne oznake funkcionisale.

Uslovne oznake vam omogućavaju da kontrolišete prikaz HTML blokova sa istom snagom kao i odeljci uslova u vašem Python kodu. Poigrajte se sa podacima u `psycho()` pogledu i vidite kako to menja izlaz uslovnih blokova.

### Petlje

Postoji jedna oznaka Django šablona za petlju: `{% for %}`. Koristi sličnu sintaksu Python-ovoj `for` izjavi i obezbeđuje neke ugrađene varijable koje daju informacije o tome gde se nalazite u iteraciji.

Dosadašnji pogledi filmova su imali vrlo malo podataka u sebi i ništa za iteriranje. Da biste se igrali sa petljama, želećete da dodate još jedan pogled u `moviefacts/views.py`:

```py
# moviefacts/views.py
def listing(request):
    data = {
        "movies": [
            (
                "Citizen Kane",   # Movie
                1941,             # Year
            ),
            (
                "Casablanca",
                1942,
            ),
            (



                "Psycho",
                1960,
            ),
        ]
    }
    return render(request, "listing.html", data)
```

Ovaj pogled ima promenljivu konteksta koja se zove `movies`, koja je lista. Šablon koji ćete praviti koristiće oznake petlje za ponavljanje ove liste. Kreirajte `templates/listing.html` na sledeći način:

```html
<!-- templates/listing.html -->
{% extends "base.html" %}

{% block content %}
  <h2> Movies</h2>
  <ul>
    {% for movie in movies %}
      <li>
        #{{forloop.counter}} {{movie.0}} was released in {{movie.1}}
      </li>
    {% endfor %}
  </ul>

  <h2> Goodies </h2>
  <ul>
    {% for goodie in confectionaries %}
      <li> {{goodie.0}} &mdash; ${{goodie.1}}</li>
    {% empty %}
      <li> <i>There are no goodies available</i> </li>
    {% endfor %}
  </ul>
{% endblock content %}
```

- Linije 7. do 11. sadrže {% for %} blok, koji je sličan for ključnoj reči u Python-u. Blok se ponavlja movies i dodeljuje lokalnu promenljivu imenovanu movie tokom svake petlje. Unutar bloka pišete HTML koji koristi prednosti parova dvostrukih zagrada za pogled sadržaja unutar `<li>` oznaka.

- Linije 8. do 10. se ponavljaju jednom za svaku iteraciju. Petlja koristi poseban objekat koji se zove forloop. Ovo je lokalna promenljiva koju Django ubacuje u kontekst šablona. Objekat ima nekoliko članova, od kojih svaki sadrži informacije o trenutnoj iteraciji petlje:

Promenljiva | Opis
----------- | ------------------------------------------
forloop.counter | Indeksni broj iteracije zasnovan na 1
forloop.counter0 | Indeksni broj iteracije zasnovan na 0
forloop.revcounter | Broj iteracija preostalih u petlji (1-indeksirano)
forloop.revcounter0 | Broj preostalih iteracija u petlji (indeksirano 0)
forloop.first | True ako je ovo prva iteracija
forloop.last | True ako je ovo poslednja iteracija
forloop.parentloop | Sadrži kontekst roditeljske petlje unutar ugnežđenih petlji

Hajde da još malo ispitamo liniju unutar bloka petlje:

```html
<!-- From inside of templates/listing.html -->
<li>
  #{{forloop.counter}} {{movie.0}} was released in {{movie.1}}
</li>
```

Sadržaj `movies` je lista torki. Svaki tuple sadrži naziv i godinu filma. Django šablonske oznake ne dozvoljavaju upotrebu indeksa. Umesto da koristite velike zagrade za pristup vrednosti u točki ili listi kao što to radite u Python-u, koristite broj kao svojstvo objekta. Vrednosti `{{movie.0}}` i `{{movie.1}}` su prva i druga vrednost tuple `movie`.

Django petlja takođe podržava posebnu oznaku koja se zove `{% empty %}`. Otvorite `listing.html` ponovo za primer ove oznake:

```html
<!-- From inside of templates/listing.html -->
{% for goodie in confectionaries %}
  <li> {{goodie.0}} &mdash; ${{goodie.1}}</li>
{% empty %}
  <li> <i>There are no goodies available</i> </li>
{% endfor %}
```

Ova oznaka je kao else klauzula. Ako u oznaci {% for %} nema šta da se ponavlja, Django pokreće `{%empty %}` blok umesto toga.

Oznaka `{% empty %}` je zgodnija od umotavanja `{% for %}` bloka u `{% if %}` blok da biste postigli isti rezultat. U ovom slučaju, nema confectionaries definisanog u kontekstu. Jezik Django šablona tretira nedostajuće vrednosti kao prazne, tako da prikazuje `{% empty %}` deo bloka.

Dodajte rutu u moviepalace/urls.py, pokrenite svoj Django razvojni server i posetite <http://127.0.0.1:8000/listing/> da vidite rezultat:

Postoji još jedna podržana oznaka nalik petlji: `{% cycle %}`. Ova oznaka uzima niz argumenata i vraća svaki redom svaki put kada se pozove:

```html
{% for movie in movies %}
    <tr class="{% cycle 'row1' 'row2' %}">
   ...
    </tr>
{% endfor %}
```

Kada `{% cycle %}` ponestane argumenata, vraća se na početak. Najčešća upotreba ovoga je dodavanje traka u tabele, menjajući vrednost stila svakog reda:

Isti rezultat možete postići pomoću `{% for %}` oznake, ali taj pristup bi zahtevao više koda.

### Komentari

Iako možete pisati komentare u HTML-u koristeći `<!-- -->`, odgovor preuzet u pretraživač korisnika uključuje te komentare. Django uključuje `{% comment %}` oznaku čiji je sadržaj u potpunosti uklonjen:

```html
{% comment %}
    Nothing to see here!
{% endcomment %}
```

Takođe možete da uključite parametar stringa u `{% comment %}` oznaku koja deluje kao beleška. Možda izgleda malo čudno, ali je korisno ako komentarišete kod. Koristite blok da komentarišete kod, a zatim belešku da se podsetite zašto ste komentarisali kod.

### Specijalni znakovi

Podrazumevano, Django šablon za prikazivanje automatski izbegava sve promenljive koje renderuje. Ovo pokušava da spreči oštećenje vašeg HTML-a ako string sadrži znak poseban za HTML, kao što su ugaone zagrade ( `<`, `>`)  koje se koriste u HTML oznakama. Oznaka `{% autoescape %}` vam omogućava da kontrolišete ovo ponašanje:

```html
{% autoescape off %}
    {{ content }}
{% endautoescape %}
```

Oznaka `{% autoescape %}` uzima samo jedan parametar on ili off, uključujući ili isključujući mehanizam za bekstvo, respektivno. Razmotrite sledeći šablon:

```html
<pre>
{% autoescape on %}
    Escaped: {{ my_text }}
{% endautoescape %}
{% autoescape off %}
    Not Escaped: {{ my_text }}
{% endautoescape %}
</pre>
```

Ako `my_text` sadrži `Hello <b>world</b>`, prikazani sadržaj bi imao dve verzije stringa:

- `autoescape on` menja HTML posebne znakove tako da se pogleduju onakvima kakvi su u stringu.
- Nasuprot tome, sa `autoescape off`, podebljana oznaka se prosleđuje u pretraživač.

Svim jezicima koji koriste specijalne znakove potreban je način direktnog korišćenja istih specijalnih znakova. Ako želite da koristite znakove koji označavaju oznaku, možete ih umotati u `{% verbatim %}` oznaku:

```html
{% verbatim %}
    Django uses mustaches braces to render variables: {{ is_scary }}.
{% endverbatim %}
```

Mnogi JavaScript okviri imaju sopstvene jezike šablona. Format dvostruke zagrade za promenljive ili oznake je prilično uobičajen. Ako treba da uključite JavaScript šablone unutar vašeg Django šablona, umotavanje u `{% verbatim %}` oznaku će dovesti do toga da ih Django renderer šablona ignoriše, ostavljajući ih na mestu za JavaScript.

### Datumi i vremena

Često morate da pogledete informacije o datumu i vremenu na web lokaciji. Oznaka `{% now %}` u jeziku Django šablona vam daje trenutni datum i vreme. Potreban je string parametar koji specificira format datuma i vremena za pogled. Iz istorijskih razloga, string formata je zasnovan na PHP `date()` funkciji, a ne na Pythonovoj `strftime`. Spisak mogućih znakova formata dostupan je u dokumentaciji.

Uobičajena upotreba oznake `{% now %}` je prikazivanje tekuće godine kao deo izjave o autorskim pravima:

```html
<p>Copyright 2012-{% now "Y" %}</p>
```

Gornji primer će pogledati obaveštenje o autorskim pravima sa godinama u rasponu od 2012. do sadašnje.

### URL adrese

Django mehanizam za pregled i rutiranje ima ugrađen način imenovanja vaših URL adresa. Čvrsto kodiranje URL-ova je loša praksa. Ako se nešto promeni, onda morate da ulovite sve vreme kada ste to koristili u svom kodu. Umesto da razrešite URL u pogledu i da ga prosledite kao kontekst, možete da koristite oznaku {% url %}:

```html
<a href="{% url 'home_page' filter %}">Home</a>
```

Argument za {% url %} je bilo koji naziv URL obrasca, na isti način na koji funkcioniše Django reverse(). Opciono uzima pozicione ili imenovane argumente za osnovni pogled na koji se poziva.

## Istraživanje ključnih filtera šablona

Kao što ste ranije naučili, oznake Django šablona su ključne reči i funkcije jezika šablona. Filteri su poput malih funkcija koje menjaju podatke na mestu pre nego što ih Django renderuje.

Možete staviti filtere unutar dvostrukih zagrada tako što ćete dodati cev (|) i naziv filtera iza promenljive.
Ovo je inspirisano načinom na koji možete da prenosite komande jedne na druge u Unix školjkama.

Sada ćete pogledati neke uobičajene filtere koji se koriste sa stringovima, listama, datumima i vremenima, zajedno sa primerima koji pokazuju kako da ih koristite.

### String filteri

Filteri mogu da rade na svim vrstama podataka. Mnogi od njih direktno odgovaraju Python funkcijama sa istom svrhom. Filter upper je isti kao i str.upper() metod u Python-u. Korišćenjem menja sadržaj promenljive u velika slova. Isprobajte tako što ćete izmeniti templates/listing.html, dodati upper filter promenljivoj movie.0:

```html
<!-- templates/listing.html -->
{% extends "base.html" %}

{% block content %}
  <h2> Movies</h2>
  <ul>
    <li>
       #{{forloop.counter}} {{movie.0|upper}} was released in {{movie.1}}
    </li>
```

Sa gornjom promenom, svaki od naslova filmova na listi će biti u velikim i velikim slovima. Nije iznenađujuće što postoji i odgovarajući lower filter.

Filter center dodaje dopunu na obe strane niza. Pading je znak za razmak, tako da u zavisnosti od vašeg fonta i oznake koja ga obavija, ovaj filter može imati ograničen efekat:

```html
<pre>{{value}} becomes *{{ value|center:"16" }}*</pre>
```

Ako value sadrži "Elephant", ovaj primer rezultira sa četiri razmaka na svakoj strani:

```html
<pre>Elephant becomes *    Elephant    *</pre>
```

Ako treba da uklonite znakove iz stringa, možete koristiti filter cut. Argument za cut je podstring za uklanjanje:

```html
<pre>*{{value}}* becomes *{{ value|cut:" " }}*</pre>
```

Pogledajte šta se dešava kada koristite filter cut sa value od "Bates Motel":

```html
<pre>*Bates Motel* becomes *BatesMotel*</pre>
```

Imajte na umu da argument za cut nije lista znakova koje želite da isečete, već string koji želite da isečete. Ako biste umesto toga pokušali da smanjite ovu vrednost pomoću "ae", ništa se ne bi dogodilo. Nema "ae" podstringa u "Bates Motel".

### Filteri liste

Filteri mogu da rade na vrednostima sličnim listi. Filteri first i last vraćaju prvu i poslednju stavku u iterable, respektivno:

```html
{{ goodies|first }}
```

Ako goodies sadrži ["popcorn", "peanuts", "cola"], onda ovaj primer rezultira "popcorn". Korišćenje last filtera bi rezultiralo "cola".

Pored korišćenja filtera za pristup delovima liste, možete ih koristiti za dobijanje podataka sa liste. Filter join je sličan Python str.join() metodi. Vraća string sastavljen od delova liste:

```html
{{ goodies|join:", " }}
```

Koristite dvotačku (:) da biste prosledili parametre filteru. Spajanje goodies liste sa zarezom kao parametrom vraća jedan string koji sadrži listu razdvojenu zarezima: "popcorn, peanuts, cola".

Da biste pronašli dužinu liste, koristite length filter. Ovo može biti posebno korisno kada se kombinuje sa drugim filterima. Filter pluralize dodaje znak "s" na kraj reči kada se radi sa više stavki. Pokušajte da dodate sledeći red templates/listing.html neposredno ispod `<h2>Movies</h2>` naslova:

```html
<!-- templates/listing.html -->
{% extends "base.html" %}

{% block content %}
  <h2> Movies</h2>
  <p>{{movies|length}} movie{{movies|length|pluralize}} found.</p>
```

Django zamenjuje prvi set duplih zagrada dužinom liste movies. Drugi set radi nešto pametno: spajanje filtera. Filter pluralize uzima broj. Ako je broj 2 ili veći, vraća "s". Korišćenjem length i zatim pluralize prenosite dužinu movies do pluralize.

Šta radite ako dodavanjem reči "s"ne množite reč? Filter podržava argument koji određuje završni sufiks:

```html
You have {{num_boxes}} box{{num_boxes|pluralize:"es"}}.
```

U ovom primeru, umesto dodavanja "s", pluralize dodaje "es", vraća "boxes". Za reči koje imaju različite završetke, možete da navedete i jedan završetak i završetak u množini:

```html
You have {{num_cherries}} cherr{{num_cherries|pluralize:"y,ies"}}.
```

U gornjem primeru, pluralize dodaje string naveden pre zareza ako je rezultat jednine i dodaje string naveden posle zareza ako je rezultat množine. Na ovaj način tačno dobijate "cherry" ili "cherries".

### Filteri za datum i vreme

Prezentacija datuma ili vremena treba da bude specifična za lokaciju korisnika. U okviru vašeg koda, preporučuje se da koristite samo Python datetime objekat. Trebalo bi da prepustite formatiranje ovog objekta vremenu renderovanja. Postoje filteri koji će vam pomoći u tome.

Filter dateuzima parametar sličan oznaci {% now %} koju ste ranije videli u vodiču. Poput te oznake, parametri za date filter određuju format prezentacije datuma:

```html
<p> {{release_date|date:"Y-m-d"}}</p>
```

Ovaj primer bi predstavio sadržaj u ISO 8601release_date formatu godina-mesec-dan. Za Građanina Kejna, to bi bilo 1941-09-05 (5. septembar 1941) za široko izdanje. Iako se zove filter, on takođe podržava vremena, pod pretpostavkom da radi sa date objektom, a ne samo sa objektom datetimedate.

Ako vas zanima samo vreme, postoji filter i za to. Filter time koristi podskup konfiguracionih parametara za date, samo one koji imaju veze sa vremenom. Upotreba je slična:

```html
<p> {{showing_at|time:"H:i"}}</p>
```

Ovaj primer prikazuje vreme u 24-časovnom formatu sata. Za film u 21:30, Django ovo prikazuje kao "21:30".

Popularni format koji čini vreme čitljivijim je prikazivanje vremena koje je prošlo od događaja. Ovo vidite u mnogim klijentima web pošte koji pogleduju informacije kao što je Primili ste e-poruku „pre 3 dana“. Django timesince filter vam daje ovu moć:

```html
<p> {{release_date|timesince}}</p>
```

Ovaj primer pokazuje koliko je vremena prošlo od datuma objavljivanja filma. Ovaj filter podrazumevano izračunava vreme koje je prošlo od sada. Alternativno, možete dodati parametar timesince koji određuje datum poređenja.

Postoji prateći filter koji treba timesince pozvati timeuntil. Umesto da gleda unazad na datum, gleda unapred. Ovo možete koristiti za pogled odbrojavanja do konferencije ili sličnog događaja.

I jedno i drugo timesince izračunajte timeuntil razliku u vremenu prema serveru kada Django renderuje šablon. Izračunavanje se vrši korišćenjem informacija o vremenskoj zoni u settings.py. Ovo može da stvori čudne rezultate ako vaš server nije u UTC-u, a vaša Django konfiguracija koristi podrazumevanu UTC. Uverite se da se ove dve konfiguracije poklapaju ili će vaš proračun vremenske razlike biti netačan.

### Biblioteke oznaka i filtera treće strane

Pored ugrađenih oznaka i filtera Django šablona, možete napisati i svoje. Možete pronaći korisne biblioteke nezavisnih proizvođača koje uključuju prilagođene oznake i filtere. Jedna takva biblioteka je django-simple-tags. Nemojte ovo brkati sa funkcijom sličnog imena simple_tag() u Djangu.

Da biste koristili biblioteku treće strane u svojim šablonima, potrebno je da pratite tri koraka:

1. Instalirajte biblioteku aplikacija
2. Registrujte aplikaciju sa INSTALLED_APPS u settings.py
3. Učitajte oznake u svoj šablon

Instaliranje Django biblioteke aplikacija se ne razlikuje od bilo kog drugog paketa. Možete ga instalirati koristeći pip:

```html
python -m pip install django-simple-tags
```

Sa instaliranim paketom, ažurirajte INSTALLED_APPS deo datoteke settings.py da bi Django bio svestan aplikacije:

```py
INSTALLED_APPS = [
    "django.contrib.admin",
    "django.contrib.auth",
    "django.contrib.contenttypes",
    "django.contrib.sessions",
    "django.contrib.messages",
    "django.contrib.staticfiles",
    "moviefacts",
    "django_simple_tags",
]
```

Sada je Django svestan aplikacije, ali još ne možete da koristite oznake. Pre nego što ih možete koristiti, prvo ih morate učitati unutar šablona. Ovo radite sa {% load %}oznakom:

```html
{% load django_simple_tags %}

Current value of DEBUG is {% get_setting "DEBUG" %}
```

U prvom redu, {% load %} oznaka dodaje sve oznake iz django-simple-tags u kontekst vašeg šablona. Imajte na umu da nema citata oko django_simple_tags. Ovo treba da bude u skladu sa import izjavama. Biblioteka django-simple-tags ima preko dvadeset oznaka i filtera. Ovaj primer prikazuje {% get_setting %} vraćanje vrednosti DEBUG iz Django podešavanja.

Postoji mnogo biblioteka nezavisnih proizvođača i mnogo oznaka koje mogu da pojednostave vaš HTML. Njihovo korišćenje je udaljeno samo jednu pip install oznaku {% load %}.

## Zaključak

Oznake i filteri Django šablona daju vam moćan način da napravite HTML izlaz na način koji se može ponovo koristiti. Šabloni su definisani na svom jeziku kako bi se odvojila poslovna logika od logike pogleda. Oznake su poput ključnih reči i funkcija jezika, dok vam filteri omogućavaju da modifikujete postojeće podatke pre nego što ih pogledete.

Ako želite da pratite neke Django tutorijale o Real Python-u, možete pogledati Prvi deo sa Django-om,
deo 1: Pravljenje aplikacije za portfolio, putanju učenja za Django za web razvoj i listu dostupnih Django tutorijala.

I poslednja odgovarajuća reč iz uputstva za filmsku palatu morala bi da bude: Rosebud.
