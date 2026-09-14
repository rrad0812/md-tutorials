
# Uvod

[Sadržaj](00_sadrzaj.md)

**Django Admin Cookbook** je knjiga o tome kako raditi stvari sa Django adminom. Namenjen je srednjim Django programerima, koji imaju određeno iskustvo sa Django adminom, ali žele proširiti svoje znanje o Django adminu i postići majstorstvo.

Uradjena je u obliku pitanja i odgovora o uobičajenim zadacima koje biste mogli raditi s Django adminom. Sva su poglavlja zasnovana na zajedničkom skupu modela,o kojima možete detaljno pročitati ovde.

Ukratko, imamo dve aplikacije, "Events" i "Entities". Modeli su:

- Events:
  - Epic,
  - Event,
  - EventHero,
  - EventVillain

- Entities:
  - Category,
  - Origin,
  - Entity,
  - Hero,
  - Vilian

Pored toga ovde su i dodatni tekstovi i Django kod prikupljeni po internetu a u vezi raznih kritičnih mesta u Django adminu. Ponešto je i sam autor priložio, ali to je zaista minimalno. Jedino vredno spomena je prevodjenje i stavljanje u kontekst tema već pomenute **Django Admin Cookbok** knjige.

[Sadržaj](00_sadrzaj.md)

Ovaj tutorijal će vas voditi kroz stvaranje prilagođenog Django admina.

## Prvi admin

### Izgradnja razvojnog okruženja

Napravi novo razvojno okruženje:

```shell
python3 -m venv django-venv
```

Aktiviraj okruženje:

```shell
django-venv/bin/activate
```

Koristi pip u komandnoj liniji za instaliranje Djanga:

```shell
(django-venv) python3 -m pip install Django
```

> [!Note]
>
> Nadalje sve što radimo na projektu radimo sa aktiviranom (django-venv) virtuelnom okolinom.

### Izgradnja novog projekta

Sada koristi `django-admin` komandu za kreiranje novog projekta.

> [!Note]  
> Tri su načina:
>
> - Prosledi ime projekta, `project_name`, biće napravljen novi direktorijum sa prosledjenim imenom i u njemu subdirektorijum sa istim imenom i fajlovima projekta:
>
>    ```shell
>    django-admin startproject project_name
>    ```
>
> - Prosledi ime projekta, `project_name` i ime projektnog direktorijuma. Projektni direktorijum već mora postojati. Biće stvoren novi `project_name` direktorijum sa fajlovima projekta unutar projektnog direktorijuma:
>
>    ```shell
>    django-admin startproject project_name project_dir_name
>    ```
>
> - Prosledi ime projekta, `project_name`, biće stvoren novi `project_name` direktorijum sa fajlovima projekta, unutar tekućeg (`project_dir_name`) direktorijuma:
>
>    ```shell
>    django-admin startproject project_name .
>    ```  

Autor predlaže treći način, s'time da je workflow sledeći:

>
```shell
mkdir project_dir_name
cd project_dir_name
(django-venv) django-admin startproject project_name .
```
>

Django-admin će kreirati `project_name` direktorijum u tekućem (`project_dir_name`) direktorijumu sa fajlovima projekta.

Od sada pa nadalje `project_dir_name` je `BASE_DIR` projekta, direktorijum koji sadaži `manage.py` fajl i odakle se pokreću sve komande na projektu. Pored toga tu je i `project_name` direktorijum, koji sadrži fajlove za podešavanje projekta.

### Izgradnja baze podataka

Sledeći korak je stvaranje baze podataka koja će se pojaviti kao nova SQLite datoteka sa imenom `db.sqlite3`. Da bismo to uradili, koristimo `manage.py` fajl iz direktorijuma projekta, koju je kreirala `django-admin startproject` komanda. Komanda koju sada želimo je `migrate`, koja može da kreira osnovne tabele baze podataka.

```shell
python manage.py migrate
```

Na kraju pokreni Django ugradjeni web server:

```shell
python manage.py runserver
```

### Kreiranje superusera

Da bi mogli da se ulogujemo na admin strane potrebe su nam dozvole. Najpovlašćeniji korisnik na admin stranama i komletnom Djnagu je `superuser`. Da bi kreirali superusera potrebno je da pokrenemo komandu:

```shell
python manage.py createsuperuser
```

Posle kompletiranja ove prcedure možemo da posetimo `localhost:8000/` u pregledaču da bi videli Django u akciji.

### Izgradnja aplikacije

Možeš da kreiraš novu aplikaciju pomoću Django `startapp` komande. Vrati se u terminal i pritisni kombinaciju CTRL-C, koja će završiti test server i vratiti nas u komandnu liniju. Zatim pomoću
`manage.py` kreiraj aplikaciju.

```shell
python3 manage.py startapp app_name
```

Sada bi u vašem projektnom direktorijumu trebalo da postoji novi direktorijum `app_name`. Ako pogledate unutra, videćete da je Django stvorio niz fajlova, koji su kostur svake aplikacije.

```shell
app_name/
    __init__.py
    admin.py
    apps.py
    migrations/
    models.py
    tests.py
    views.py
```

Ali pre nego što učinimo bilo šta dalje, moramo da konfigurišemo projekat tako da uključuje novu aplikaciju. Pomoću editora teksta otvorite datoteku `settings.py` u projektnom direktorijumu. Dodajte aplikaciju `app_name` na `INSTALLED_APPS` listu koju tamo pronađete.

```py
INSTALLED_APPS = (
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'app_name',
)
```

To je sve. Možete početi sa stvaranjem modela.

[Nazad](0100_uvod.md)
