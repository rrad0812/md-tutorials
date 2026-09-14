
# 8 Django saveta i trikova naučenih tokom godina

Sadržaj

- Automatizujte pomoću komandi za upravljanje
- Organizujte URL-ove po imenskim prostorima aplikacija
- Kalup za kolačiće za čistu skelu
- Održavajte frontend vitkim pomoću vanile HTML-a, oznaka šablona i HTMX-a
- Podesite Sentry za besplatno praćenje grešaka
- Dvaput razmislite o pogledima zasnovanim na klasi
- Koristite Django keširanje za poboljšanje performansi
- Koristite Django Shell Plus za brzo debagovanje i testiranje

Nakon više od 4 godine rada sa Django-om, kako profesionalno tako i lično, naučio sam mnogo saveta i trikova koje bih voleo da sam znao kada sam počeo. U ovom članku ću podeliti neke od najvažnijih koje sam naučio tokom godina.

Pošto je dokumentacija za Django veoma dobra, neću deliti mnogo isečaka koda, već ću objasniti koncepte i pružiti linkove ka zvaničnoj dokumentaciji. Svi ovi saveti i trikovi su poticali iz mog sopstvenog  iskustva, dokumentacije za Django i knjige "Dve kuglice Djanga".

## Automatizujte pomoću komandi za upravljanje

Ako koristite `python manage.py runserver` ili `python manage.py migrate`, već radite sa Django-ovim komandama za upravljanje! One su fantastičan način za automatizaciju rutinskih zadataka, što ih čini idealnim za razne radnje, kao što su slanje imejlova, ažuriranje baza podataka ili pokretanje skripti.

Kreiranje prilagođenih komandi za upravljanje vam omogućava da dodate komande prilagođene jedinstvenim potrebama vašeg projekta.

Neke uobičajene komande za prilagođeno upravljanje koje koristim uključuju:

- Čišćenje starih unosa u bazu podataka koji više nisu potrebni
- Slanje automatskih podsetnika ili dnevnih izveštaja Generisanje i slanje mesečnih izveštaja o učinku marketinškom timu
- Prilagođene komande mogu se zakazati pomoću UNIX cron zadataka ili Windows Task Scheduler-a da se pokreću u određeno vreme.
  Za još složenije zahteve, razmislite o korišćenju Celery-ja za asinhrono upravljanje periodičnim zadacima, omogućavajući ponovne pokušaje i napredne opcije zakazivanja.

### Pisanje prilagođenih django-admin komandi

- Organizujte URL-ove po imenskim prostorima aplikacija

  Za velike projekte sa više aplikacija, organizovanje URL-ova po imenskim prostorima održava stvari čistim i smanjuje rizik od sukoba ruta. U Django-u, imenski prostori grupišu povezane URL-ove, što olakšava navigaciju i referenciranje njima u šablonima i prikazima.
  
  Na primer, ako imate aplikacije blog i shop, dodavanje oznake app_name u svaku datoteku aplikacije `urls.py` vam omogućava da nedvosmisleno pozovete na svaku URL adresu:
  
  ```py
  # blog/urls.py
  app_name = 'blog'
  urlpatterns = [
      path('post/<int:pk>/', views.post_detail, name='post_detail'),
  ]
  ```
  
  ```py
  # shop/urls.py
  app_name = 'shop'
  urlpatterns = [
      path('product/<int:pk>/', views.product_detail, name='product_detail'),
  ]
  ```
  
  Korišćenje ovih imenskih prostora u šablonima postaje jednostavno i jasno, izbegavajući sukobe sa slično imenovanim rutama:
  
  ```html
  <a href="{% url 'blog:post_detail' pk=post.pk %}">Read more</a>
  ```
  
  Ova struktura ne samo da poboljšava čitljivost, već vam omogućava i ponovnu upotrebu imena ruta u različitim aplikacijama, što olakšava upravljanje rastućom bazom koda.

- URL imenski prostori i uključene URL konfiguracije

- Kalup za kolačiće za čistu skelu

- Kada sam prvi put počeo da učim Django, bio sam preplavljen količinom šablonskog koda koji sam morao da napišem za svaki novi projekat i, iskreno, mnogi Django projekti dele istu strukturu. Sa Cookiecutter- om
  možete generisati potpuno strukturirani šablon projekta, zajedno sa najboljim praksama i optimizovanim podešavanjima, za nekoliko sekundi.

  Šablon Django Cookiecutter, koji održavaju autori Two Scoops of Django, je posebno koristan, jer uključuje sve što vam je potrebno za moderan Django projekat: optimizovana podešavanja za bezbednost, testiranje i implementaciju.
  
  Početnici ga mogu koristiti za učenje najboljih praksi, dok napredni korisnici mogu da ga prilagode alatima kao što su Celery za upravljanje zadacima, Docker za kontejnerizaciju i drugim servisima.
  
  Istraživanje generisanog koda je takođe vredno iskustvo učenja, jer uključuje reference na dokumentaciju koje objašnjavaju svaki izbor, pružajući dalji uvid u Django arhitekturu.
  
- Django kolačić

  Održavajte frontend vitkim pomoću vanile HTML-a, oznaka šablona i HTMX-a

  Mnogi programeri se okreću jednostraničnim aplikacijama (SPA) zbog dinamičke interaktivnosti, ali Django-ov sistem za kreiranje šablona je dovoljno moćan da održi frontend vitkim i efikasnim. Django-
  ovi šabloni, u kombinaciji sa oznakama šablona i filterima, omogućavaju opsežno prilagođavanje i besprekornu interakciju sa Django-ovim sistemom za rutiranje.
  
  Kada vam je potrebno samo malo interaktivnosti, HTMX može dodati dinamičke, SPA-slične funkcije vašem HTML-u bez potrebe za teškim JavaScript okvirom. HTMX vam omogućava da lako učitavate sadržaj ili šaljete obrasce asinhrono, tako da dobijate glatku, modernu funkcionalnost bez ugrožavanja jednostavnosti ili vremena učitavanja.
  
  Detaljnije ću govoriti o prednostima HTMX-a u poređenju sa frejmvorcima poput React-a i Vue-a ovde.
  
- Podesite Sentry za besplatno praćenje grešaka
  
  Vođenje evidentiranja ide dalje od samog praćenja grešaka — ono pruža uvid u to kako korisnici interaguju sa vašom aplikacijom i pomaže u identifikaciji uskih grla u performansama. Za produkciona okruženja preporučujem korišćenje Sentry-ja, koji prati greške pomoću detaljnih tragova steka, podataka o zahtevima i uvida u korisnike, pomažući vam da efikasnije rešavate probleme.
  
  Podešavanje Sentry-ja je jednostavno. Nakon konfigurisanja, Sentry će vas obavestiti putem e-pošte kad god dođe do greške i zabeležiti detalje u Sentry kontrolnoj tabli. Evo osnovnog podešavanja:
  
  ```py
  import sentry_sdk
  
  sentry_sdk.init(
      dsn="https://**************.ingest.de.sentry.io/*****",
      traces_sample_rate=1.0,
      profiles_sample_rate=1.0,
  )
  ```
  
  Iako je besplatni plan ograničen, više je nego dovoljno da se razumeju zahtevi koji ne uspevaju u lepom formatu i da se dobije obaveštenje kada nešto krene naopako.
  
  Koristim Sentry već 5 godina i većinu vremena besplatni plan je bio dovoljan za moje potrebe, ako su vam potrebne napredne funkcije poput praćenja performansi ili praćenja izdanja, postoje plaćeni planovi sa više opcija.

- Stražar za Djanga

  Dvaput razmislite o pogledima zasnovanim na klasi. Kada sam se upoznao sa prikazima zasnovanim na klasama, bio sam zapanjen količinom koda koju sam mogao da uštedim koristeći ih, bio je kraći, čistiji i čitljiviji. Međutim, kako sam počeo da radim na složenijim projektima, shvatio sam da prikazi zasnovani na klasama mogu biti mač sa dve oštrice. Ovo je zaista vruća tema u Django zajednici.
  
  Prikazi zasnovani na klasama (CBV) su odlični za smanjenje dupliranja koda i čitljivije CRUD operacije. Međutim, za jedinstveno ili veoma prilagođeno ponašanje, CBV-ovi mogu postati glomazni, posebno kada se radi o duboko ugnežđenom nasleđivanju ili neočekivanom „magičnom“ ponašanju.
  
  Za jednostavnije projekte, CBV-ovi mogu poboljšati razvoj, posebno za liste, detaljne prikaze ili obrasce. Međutim, za složenije slučajeve upotrebe, razmislite o korišćenju prikaza zasnovanih na funkcijama (FBV). FBV-ovi omogućavaju jednostavniju logiku bez dodatnog nasleđivanja i često ih je lakše otklanjati greške.
  
  U praksi:
  
  Koristite CBV-ove kada treba da pratite obrazac za višekratnu upotrebu, standardizovani obrazac (CRUD). Koristite FBV-ove za prikaze koji zahtevaju specifičnu, prilagođenu funkcionalnost.
  
## Django Class-bazirani prikazi

### Koristite Django keširanje za poboljšanje performansi

Djangoovo ugrađeno keširanje može smanjiti vreme učitavanja tako što prikazuje sadržaj direktno iz keša umesto da svaki put šalje upit bazi podataka.

Keširanje treba obaviti nakon što ste optimizovali upite i prikaze baze podataka, međutim, i dalje možete imati koristi od keširanja čak i ako imate mali veb sajt. Jedan paket koji ja zaista volim da koristim je `django-cachalot`.

Django Cachalot kešira Django ORM upite, tako da se ponovljeni upiti za iste podatke prikazuju iz keša umesto iz baze podataka. Podržava automatsko poništavanje — kada se podaci ažuriraju u bazi podataka, ažuriraju se i relevantni unosi u kešu, tako da ne morate da brinete o zastarelim podacima.

Da bi ovo funkcionisalo, moraćete da instalirate keš bekend kao što je Redis. Sa Digital Ocean-om možete imati pristojnu Redis instancu za 15 dolara mesečno, što je više nego dovoljno za većinu veb lokacija.

### Django Cachalot | Django-ov okvir za keširanje

Koristite Django Shell Plus za brzo debagovanje i testiranje

Šta ako vam kažem da postoji bolji način za interakciju sa vašom bazom podataka nego korišćenje Django administratorskog panela ili rasipanja printnaredbi u vašim prikazima (views)?

Django dolazi sa ugrađenim okruženjem koje vam omogućava da interagujete sa svojim projektom direktno iz komandne linije. Možete ga koristiti za testiranje modela, pokretanje upita ili debagovanje koda bez potrebe za pokretanjem servera ili ispisivanjem poruka o debagovanju u prikazima (views).

Međutim, podrazumevana Django shell ima svoja ograničenja, i tu dolazi do izražaja Django Shell Plus.

Paket `django-extensions` uključuje razne alate za razvoj pomoću Django-a, a jedan od najmoćnijih je `shell_plus`. Shell Plus je poboljšana verzija Django-ove podrazumevane shell komande, koja automatski učitava sve modele i podešavanja vašeg projekta. Čak podržava i IPython i Jupyter Notebooks, pružajući vam bogatije interaktivno iskustvo.

Za programere koji redovno rade sa Django ORM-om i žele efikasniji način interakcije sa bazom podataka, `shell_plus` je prekretnica.
