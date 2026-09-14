
# Django sa Postgres, Gunicorn, Nginx i Docker

Ovo je korak po korak vodič koji se detaljno opisuju kako da konfigurišete Django da se pokrene na Dockeru i Dockeru sa Postgresom. Za proizvodno okruženje dodaćemo Nginx i Gunicorn. Pogledajte takođe kako da poslužimo statičke i medijske datoteke Django putem Nginx-a.

U ovom tutorijalu ćete naučiti kako da postavite razvojno okruženje sa Docker-om.

## Novi Django projekat

```sh
mkdir django-on-docker && cd django-on-docker
mkdir app && cd app
python3.11 -m venv env
source env/bin/activate
(env)
(env) pip install django==4.2.3 Project Setup
(env) django-admin startproject hello_django .
(env) python manage.py migrate
(env) python manage.py runserver Postgres
```

Slobodno zamenite `virtualenv` i `pip` za `Poetry` ili `pipenv`. Za više proizvodnih `dockerfile` pogledajte moderna pajton okruženja.

Idite na <http://localhost:8000/> da biste videli Django ekran dobrodošlice. Ubijte server. Zatim izađite i uklonite virtuelno okruženje.

Imamo jednostavan projekat Django-a sa kojim ćemo raditi.

Kreirajte `requirements.txt` datoteku u direktoriju `app` i dodajte Django kao zavisnost:

```sh
Django == 4.2.3
```

Pošto ćemo se preseliti na Postgres, idemo napred i uklonimo datoteku db.sqlite3 iz direktorija "app".

Vaš direktorijum projekta sada bi trebalo da izgleda ovako:

```sh
└── app
    ├── hello_django
    │   ├── __init__.py
    │   ├── asgi.py
    │   ├── settings.py
    │   ├── urls.py
    │   └── wsgi.py
    ├── manage.py
    └── requirements.txt
```

## Docker

Instalirajte Docker, ako ga već nemate, dodajte `dockerfile` u direktorijum "app":

```sh
#pull official python base image
FROM python:3.11.4-slim-buster


# set work directory
WORKDIR /usr/src/app


# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1


# install dependencies
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# copy project
COPY . .
```

Dakle, počeli smo sa `slim-buster` slikom na bazi Docker-a za Python 3.11.4. Zatim smo postavili radni direktorij zajedno sa dve promenljive okoline:

1. PYTHONDONTWRITEBYTECODE
   Sprečava Pajton da piše `pyc` datoteke na disk (ekvivalentno `python -B` opciji)

2. PYTHONUNBUFFERED
   Sprečava pajton od buferisanja `stdout`-a i `stderr`-a. (equivalent to `python -u` option)

Konačno, ažurirali smo `pip`, kopirali smo datoteku `requirements.txt`, instalirali smo zavisnosti i kopirali sve.

Dalje, dodajmo `docker-compose.yml` datoteku u korenski direktorijum projekta:

```sh
version: '3.8'
services:
web:
build: ./app
command: python manage.py runserver 0.0.0.0:8000

volumes:
  - ./app/: /usr/src/app/

ports:
  - 8000:8000

env_file:
  - ./.env.dev
```

Ažurirajmo `SECRET_KEY`, `DEBUG` i `ALLOWED_HOSTS`  varijabe u `settings.py`:

```py
SECRET_KEY = os.environ.get("SECRET_KEY")
DEBUG = bool(os.environ.get("DEBUG", default=0))

# 'DJANGO_ALLOWED_HOSTS' should be a single string of hosts with a space between each.
# For example: 'DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]'
ALLOWED_HOSTS = os.environ.get("DJANGO_ALLOWED_HOSTS").split(" ")
```

Obavezno dodajte uvoz na vrh:

```py
import os
```

Onda, napravite `.env.dev` datoteku u korenu projekta za čuvanje varijabli okoline za razvoj:

```py
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
```

Izgradimo sliku:

```sh
docker-compose build
```

Jednom kada je slika izgrađena, pokrenite kontejner:

```sh
docker-compose up -d
```

Idite na <http://localhost:8000/> da ponovo vidite Django ekran dobrodošlice.

Proverite greške u zapisima ako ovo ne radi putem:

```sh
docker-compose logs -f .
```

## Postgres

Da biste konfigurisali postgres, moraćemo dodati novu uslugu u datoteku `docker-compose.yml`, ažurirati postavke Django i instalirati `psycopg2`.

Prvo dodajte novu uslugu zvanu `db` na `docker-compose.yml`:

```sh
version: '3.8'

services:
web:
build: 
  ./app
command: 
  python manage.py runserver 0.0.0.0:8000

volumes:
  - ./app/:/usr/src/app/

ports:
  - 8000:8000

env_file:
  - ./.env.dev

depends_on:
  - db

db:
  image: postgres:15

volumes:
  - postgres_data: /var/lib/postgresql/data/

environment:
  - POSTGRES_USER=hello_django
  - POSTGRES_PASSWORD=hello_django
  - POSTGRES_DB=hello_django_dev

volumes:
  postgres_data:
```

Ova konfiguracija će vezati `postgres_data` na `/var/lib/postgresql/data/` direktorijum u kontejneru.

Takođe smo dodali ključeve enviromenta da definišemo ime za podrazumevanu bazu podataka i korisničko ime i lozinku.

Pregledajte odeljak "Enviroment varijable" Postgres Docker Hub stranice za više informacija.

Trebaće nam i nove promenljive okoline za veb uslugu, tako da ažurirate .env.dev kao:

```sh
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_dev
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
```

Ažurirajmo `DATABASES`  rečnik u `settings.py`:

```py
DATABASES = {
  "default": {
    "ENGINE": os.environ.get("SQL_ENGINE",
    "django.db.backends.sqlite3"),
    "NAME": os.environ.get("SQL_DATABASE", BASE_DIR / "db.sqlite3"),
    "USER": os.environ.get("SQL_USER", "user"),
    "PASSWORD": os.environ.get("SQL_PASSWORD", "password"),
    "HOST": os.environ.get("SQL_HOST", "localhost"),
    "PORT": os.environ.get("SQL_PORT", "5432"),
  }
}
```

Ovde je baza podataka konfigurisana na osnovu promenljivih okoline koje smo upravo definisali.Zapazite podrazumevane vrednosti.

Dodajmo psycopg2 u `requirements.txt`:

```sh
Django==4.2.3
psycopg2-binary==2.9.6
```

Izgradimo novu sliku i pokrenimo kontejner:

```sh
docker-compose up -d --build
```

Pokrenimo migracije:

```sh
docker-compose exec web python manage.py migrate --noinput
```

Ako dobijemo sledeću grešku

```sh
django.db.utils.OperationalError: FATAL:  database "hello_django_dev" does not exist
```

pokrenimo

```sh
docker-compose down -v  
```

da uklonimo volumene zajedno sa kontejnerima. Zatim ponovo izgradimo slike, pokrenimo kontejner i primenimoe migracije.

Osigurajte da su podrazumevane Đango table stvorene:

```sh
docker-compose exec db psql --username=hello_django -- dbname=hello_django_dev

psql (15.3)
Type "help" for help.
hello_django_dev=# \l
                             List of databases
-------------------------------------------------------------------------------------
       Name       |    Owner     | Encoding |  Collate   |   Ctype    | Access privileges
------------------|--------------|----------|------------|------------|--------------
 hello_django_dev | hello_django | UTF8     | en_US.utf8 | en_US.utf8 |
 postgres         | hello_django | UTF8     | en_US.utf8 | en_US.utf8 |
 template0        | hello_django | UTF8     | en_US.utf8 | en_US.utf8 | =c/ hello_django              
 template1        | hello_django | UTF8     | en_US.utf8 | en_US.utf8 | =c/ hello_django
(4 rows)

hello_django_dev=# \c hello_django_dev
```

Sada ste konektovani na bazu podaataka "hello_django_dev" kao korisnik "hello_django".

```sh
hello_django_dev=# \dt
                     List of relations
------------------------------------------------------------
 Schema | Name                       | Type  | Owner
--------+----------------------------+-------+--------------
 public | auth_group                 | table | hello_django
 public | auth_group_permissions     | table | hello_django
 public | auth_permission            | table | hello_django
 public | auth_user                  | table | hello_django
 public | auth_user_groups           | table | hello_django
 public | auth_user_user_permissions | table | hello_django
 public | django_admin_log           | table | hello_django
 public | django_content_type        | table | hello_django
 public | django_migrations          | table | hello_django
 public | django_session             | table | hello_django
 (10 rows)

hello_django_dev=# \q
```

Možete da proverite da je volume kreiran pokretanjem:

```sh
docker volume inspect django-on-docker_postgres_data
```

Trebali biste videti nešto slično:

```sh
[
  {
    "CreatedAt": "2023-07-20T14:15:27Z",
    "Driver": "local",
    "Labels": {
      "com.docker.compose.project": "django-on-docker",
      "com.docker.compose.version": "2.19.1",
      "com.docker.compose.volume": "postgres_data"
  },
    "Mountpoint": "/var/lib/docker/volumes/django-on-
    docker_postgres_data/_data",
    "Name": "django-on-docker_postgres_data",
    "Options": null,
    "Scope": "local"
  }
]
```

Zatim dodajte datoteku `entrypoint.sh` u direktorijum `app` da biste proverili da je Postgres zdrav pre unošenja migracija i pokretanje Django Development Server-a:

```sh
#!/bin/sh

if [ "$DATABASE" = "postgres" ] then
  echo "Waiting for postgres..."
  while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
  done
  echo "PostgreSQL started"
fi

python manage.py flush --no-input
python manage.py migrate

exec "$@"
```

Ažurirajte dozvole za datoteku `app/entrypoint.sh` lokalno:

```sh
chmod +x app/entrypoint.sh
```

Zatim ažurirajte `Dockerfile` da kopirate preko `entrypoint.sh` datoteka i pokrenite ga kao Docker entrypoint komanda:

```sh
# pull official python base image
FROM python:3.11.4-slim-buster

# set work directory - 2
WORKDIR /usr/src/app

# set environment variables - 2
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update && apt-get install -y netcat

# install dependencies - 2
RUN pip install --upgrade pip
COPY ./requirements.txt .
RUN pip install -r requirements.txt

# copy entrypoint.sh
COPY ./entrypoint.sh .
RUN sed -i 's/\r$//g' /usr/src/app/entrypoint.sh
RUN chmod +x /usr/src/app/entrypoint.sh

# copy project - 2
COPY . .

# run entrypoint.sh
ENTRYPOINT ["/usr/src/app/entrypoint.sh"]
```

Dodajte `DATABASE`  environment varijable u `.env.dev`:
Add the DATABASE  environment variable to `.env.dev`:

```py
DEBUG=1
SECRET_KEY=foo
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]

SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_dev
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
```

Test it out again:

1. Ponovo izgradite slike
2. Pokrenite kontejnere
3. Probajte na <http://localhost:8000/>

>[!Note]
>
> Prvo, uprkos dodavanju postgresa, još uvek možemo da kreiramo nezavisnu sliku za Django sve dok varijabla baze podataka nije postavljena na postgres.Da
> testirate, izgradite novu sliku, a zatim pokrenite novi kontejner:

```sh
docker build -f ./app/Dockerfile -t hello_django:latest ./app

docker run -d \
    -p 8006:8000 \
    -e "SECRET_KEY=please_change_me" -e "DEBUG=1" -e

"DJANGO_ALLOWED_HOSTS=*" \
    hello_django python /usr/src/app/manage.py runserver 0.0.0.0:8000
```

Trebali biste moći da vidite stranicu dobrodošlice na <http://localhost:8000>
Trebali biste moći da vidite stranu dobrodošlice na <http://localhost:8006>

Drugo, možda želite da komentarišete bazu podataka i migrirate naredbe u sharepoint.sh skriptu tako da ne rade na svakom početku kontejnera ili ponovo pokrenuti:
Drugo, možda ćete želeti da komentarišete Flush i migrirate komande u ulaznoj enteypoint.sh skripte tako da ne rade na svakom početku i počnite:

```sh
#!/bin/sh

if [ "$DATABASE" = "postgres" ] then
  echo "Waiting for postgres..."
  while ! nc -z $SQL_HOST $SQL_PORT; do
    sleep 0.1
  done
  echo "PostgreSQL started"
fi

# python manage.py flush --no-input
# python manage.py migrate

exec "$@"
```

Umesto toga, možete ih ručno pokrenuti, nakon što se kontejneri pokrenu, kao:

```sh
docker-compose exec web python manage.py flush --no-input
docker-compose exec web python manage.py migrate
```

## Gunicorn

Krećemo se dalje, za proizvodna okruženja, dodajmo Gunicorn, proizvodni WSGI server:

```sh
Django==4.2.3
gunicorn==21.2.0
psycopg2-binary==2.9.6
```

Pošto još uvek želimo da koristimo ugrađeni server Django u razvoju, kreiramo novu kompoziciju pod nazivom `docker-compose.prod.yml` za produkciju:

```sh
version: '3.8'
services:
web:

build: ./app

command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000

ports:
  - 8000:8000

env_file:
  - ./.env.prod

depends_on:
  - db

db:
  image: postgres:15

volumes:
  - postgres_data:/var/lib/postgresql/data/

env_file:
  - ./.env.prod.db

volumes:

postgres_data:

```

Ako imate više okruženja, možda ćete želeti da pogledate upotrebu `docker-compose.override.yml` konfiguracione datoteke. Ovom pristupom, dodali biste baznu konfiguraciju na `docker-compose.yml` datoteka i zatim koristite `docker-compose.override.yml` datoteku za nadjačavanje tih konfiguracionih postavki na osnovu enviromenta.

Zabeležite zadanu komandu. Vodimo Gunicorn, a ne na Django Development Server. Takođe smo uklonili volume sa veb servisa, jer nam to ne treba u proizvodnji.Konačno, koristimo odvojene promenljive datoteke za zaštitu enviromenta da biste definisali promenljive enviromenta za obe usluge koje će se preneti na kontejner na vreme izvođenja.

`.env.prod:`

```sh
DEBUG=0
SECRET_KEY=change_me
DJANGO_ALLOWED_HOSTS=localhost 127.0.0.1 [::1]
SQL_ENGINE=django.db.backends.postgresql
SQL_DATABASE=hello_django_prod
SQL_USER=hello_django
SQL_PASSWORD=hello_django
SQL_HOST=db
SQL_PORT=5432
DATABASE=postgres
```

`.env.prod.db:`

```sh
POSTGRES_USER=hello_django
POSTGRES_PASSWORD=hello_django
POSTGRES_DB=hello_django_prod
```

Dodajte dve datoteke u root projekta.Verovatno ćete želeti da ih držite van kontrole verzije, pa ih dodajte na `.gitignore` datoteku.

Srušite kontejnere za razvoj (i pridružene količine sa -v zastavom):

```sh
docker-compose down -v
```

Zatim, izgradite proizvodne slike i okrenete posude:

```sh
docker-compose -f docker-compose.prod.yml up -d --build
```

Proverite da li je `hello_django_prod` baza podataka stvorena zajedno sa podrazumevanim django tabelama. Isprobajte stranu admin na <http://localhost:8000/admin>. Statičke datoteke se više ne učitavaju. To se očekuje od kada je režim uklanjanja pogrešaka isključen. To ćemo popraviti uskoro.

Opet, ako kontejner ne pokrene, proverite greške u dnevnicima sa

```sh
docker-compose -f docker-compose.prod.yml logs -f .
```

### Produkcioni dockerfile

Da li ste primetili da i dalje pokrećemo `flush` baze podataka (što očisti bazu podataka) i migriraju komande svaki put kada se kontejner pokrene?Ovo je u redu u razvoju, ali kreirajmo novu ulaznu datoteku za proizvodnju.

`entrypoint.prod.sh:`

```sh
#!/bin/sh

if [ "$DATABASE" = "postgres" ] then
  echo "Waiting for postgres..."
  while ! nc -z $SQL_HOST $SQL_PORT; do
      sleep 0.1
  done
  echo "PostgreSQL started"
fi

exec "$@"
```

Ažurirajte dozvole za datoteku lokalno:

```sh
chmod +x app/entrypoint.prod.sh
```

Da biste koristili ovu datoteku, napravite novi `Dockerfile` pod nazivom `Dockerfile.prod` za upotrebu sa proizvodnjom:

### BUILDER

```sh
# pull official python base image
FROM python:3.11.4-slim-buster as builder


# set work directory
WORKDIR /usr/src/app

# set environment variables
ENV PYTHONDONTWRITEBYTECODE 1
ENV PYTHONUNBUFFERED 1

# install system dependencies
RUN apt-get update && \
    apt-get install -y --no-install-recommends gcc

# lint
RUN pip install --upgrade pip
RUN pip install flake8==6.0.0

COPY . /usr/src/app/

RUN flake8 --ignore=E501,F401 .

# install python dependencies
COPY ./requirements.txt .

RUN pip wheel --no-cache-dir --no-deps --wheel-dir /usr/src/app/wheels -r 
```

`requirements.txt`

```sh
### FINAL

# pull official base image
FROM python:3.11.4-slim-buster

# create directory for the app user
RUN mkdir -p /home/app

# create the app user
RUN addgroup --system app && adduser --system --group app

# create the appropriate directories
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
WORKDIR $APP_HOME

# install dependencies
RUN apt-get update && apt-get install -y --no-install-recommends netcat

COPY --from=builder /usr/src/app/wheels /wheels
COPY --from=builder /usr/src/app/requirements.txt .

RUN pip install --upgrade pip
RUN pip install --no-cache /wheels/*

# copy entrypoint.prod.sh
COPY ./entrypoint.prod.sh .
RUN sed -i 's/\r$//g' $APP_HOME/entrypoint.prod.sh
RUN chmod +x  $APP_HOME/entrypoint.prod.sh

# copy project
COPY . $APP_HOME

# chown all the files to the app user
RUN chown -R app:app $APP_HOME

# change to the app user
USER app

# run entrypoint.prod.sh
ENTRYPOINT ["/home/app/web/entrypoint.prod.sh"]
```

Evo, koristili smo doker multi-fazni da bismo smanjili konačnu veličinu slike. U osnovi, graditelj je privremena slika koja se koristi za izgradnju pajtonskih točkova.Točkovi se zatim kopiraju na konačnu proizvodnu sliku i slika građevinara se odbacuje.

Mogli biste uzeti više faza izgradnje da dodate korak dalje i koristite jedan `DockerFile` umesto da kreirate dva `DockerFiles`. Razmislite o prednostima i nedostacima koristeći ovaj pristup preko dve različite datoteke.

Da li ste primetili da smo stvorili korisnika koji nije root? Podrazumevano, Docker pokreće kontejnerske procese kao root unutar kontejnera. Ovo je loša praksa jer napadači mogu dobiti pristupu root-u docker host-a ako uspeju da se izbore iz kontejnera. Ako ste root u kontejneru, bićete root i na hostu.

Ažurirajte veb uslugu u okviru `docker-compose.prod.yml` datoteka za izgradnju sa `dockerfile.prod`:

```sh
web:
build:
context: ./app

dockerfile: Dockerfile.prod

command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000

ports:
  - 8000:8000

env_file:
  - ./.env.prod

depends_on:
  - db

Try it out:

docker-compose -f docker-compose.prod.yml down -v
docker-compose -f docker-compose.prod.yml up -d --build
docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
```

### Nginx

Sledeće, dodajmo Nginx u miks da deluje kao obrnuti proksi za Gunicorn za obradu zahteva klijenta, kao i da služe statičke datoteke.

Dodajte uslugu na `docker-compose.prod.yml`:

```sh
nginx:
build: ./nginx

ports:
  - 1337:80

depends_on:
  - web
```

Zatim, u lokalnom root projektu, kreirajte sledeće datoteke i mape:

```sh
└── nginx
    ├── Dockerfile
    └── nginx.conf
```

`Dockerfile`:

```sh
FROM nginx:1.25
RUN rm /etc/nginx/conf.d/default.conf
COPY nginx.conf /etc/nginx/conf.d
```

`nginx.conf:`

```sh
upstream hello_django {
    server web:8000;
}

server {
    listen 80;
    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
}
```

Zatim, ažurirajte veb servis u `docker-compose.prod.yml`, zamena portova sa izlaganjem:

```sh
web:
build:
context: ./app

dockerfile: Dockerfile.prod

command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000

expose:
  - 8000

env_file:
  - ./.env.prod

depends_on:
  - db
```

Sada je priključak 8000 izložen interno samo na druge usluge docker-a. Port se više neće objavljivati u host mašini.

Za više na portovima vs izložite, pregledajte ovo pitanje preliva.

Ispitajte ga ponovo.

```sh
docker-compose -f docker-compose.prod.yml down -v
docker-compose -f docker-compose.prod.yml up -d --build
docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
```

Osigurajte da se aplikacija postavlja i radi na <http://localhost:1337>.

Vaša struktura projekta treba da izgleda kao:

```sh
├── .env.dev
├── .env.prod
├── .env.prod.db
├── .gitignore
├── app
│   ├── Dockerfile
│   ├── Dockerfile.prod
│   ├── entrypoint.prod.sh
│   ├── entrypoint.sh
│   ├── hello_django
│   │   ├── __init__.py
│   │   ├── asgi.py
│   │   ├── settings.py
│   │   ├── urls.py
│   │   └── wsgi.py
│   ├── manage.py
│   └── requirements.txt
├── docker-compose.prod.yml
├── docker-compose.yml
└── nginx
    ├── Dockerfile
    └── nginx.conf
```

Bring the containers down once done:

```sh
docker-compose -f docker-compose.prod.yml down -v
```

Pošto je Gunicorn, web aplikacija neće poslužiti statičke datoteke.Dakle, kako treba da se obrađuju statičke i medijske datoteke u ovoj konfiguraciji?

### Static Files

Ažurirajte `settings.py`:

```sh
STATIC_URL = "/static/"
STATIC_ROOT = BASE_DIR / "staticfiles"
```

### Razvoj

Svaki zahtev za <http://localhost:8000/static/*> biće serviran iz `staticfiles` direktorijuma.

Da biste testirali, prvo ponovo izgradite slike i pokrećete nove kontejnere po uobičajenoj proceduri. Osigurajte da se statičke datoteke i dalje pravilno poslužuju na <http://localhost:8000/admin>.

### Produkcija

Za proizvodnju dodajte volume na Veb i Nginx services u `docker-compose.prod.yml` tako da će svaki kontejner deliti onaj imenovan `staticfiles`:

```sh
version: '3.8'
services:
web:

build:
context: ./app

dockerfile: Dockerfile.prod

command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000

volumes:
  - static_volume:/home/app/web/staticfiles

expose:
  - 8000

env_file:
  - ./.env.prod

depends_on:
  - db

db:
  image: postgres:15

volumes:
  - postgres_data:/var/lib/postgresql/data/

env_file:
  - ./.env.prod.db

nginx:
build: ./nginx

volumes:
- static_volume:/home/app/web/staticfiles

ports:
- 1337:80

depends_on:
- web

volumes:
postgres_data:
static_volume:
```

We need to also create the `/home/app/web/staticfiles` folder in `Dockerfile.prod`:

```sh
# create the appropriate directories
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
WORKDIR $APP_HOME
```

Zašto je to potrebno?

Docker sastavite normalno montira pod nazivom zapremine kao root. A pošto koristimo korisnika koji nije root, dobićemo grešku koja je uskraćivala dozvolu kada se `colectstatic` naredba pokrene ako direktorijum već ne postoji da biste to mogli da se oko toga možete da preuzmete:

1. Kreirajte mapu u dockerfile (izvor)
2. Promenite dozvole direktorijuma nakon što je montiran (izvor)

Koristili smo bivše.

Zatim ažurirajte Nginx konfiguraciju za usmerevanje statičkih datoteka u direktorijum `staticfiles`:

```sh
upstream hello_django {
    server web:8000;
}

server {
    listen 80;
    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    location /static/ {
        alias /home/app/web/staticfiles/;
    }
}
```

Spin niz razvojne kontejnere:

```sh
docker-compose down -v
```

Test:

```sh
docker-compose -f docker-compose.prod.yml up -d --build
docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --no-input --clear
```

Again, requests to <http://localhost:1337/static/*>  will be served from the "staticfiles" directory.

Navigate to <http://localhost:1337/admin> and ensure the static assets load correctly.

You can also verify in the logs -- via `docker-compose -f docker-compose.prod.yml logs -f  --` that requests to the static files are served up successfully via Nginx:

nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /admin/HTTP/1.1" 302 0 "-" "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6)
    AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36""-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /admin/login/?next=/admin/ HTTP/1.1" 200 2214 "-" "Mozilla/5.0 (Macintosh; Intel
    Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko)Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/css/base.css HTTP/1.1" 304 0 <http://localhost:1337/admin/login/?next=/admin/>    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/css/nav_sidebar.css HTTP/1.1" 304 0 <http://localhost:1337/admin/login/?next=/admin/> "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36""-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/
admin/css/responsive.css HTTP/1.1" 304 0 <http://localhost:1337/admin/login/?next=/admin/> "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6)
    AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/css/login.css HTTP/1.1" 304 0 <http://localhost:1337/admin/login/?next=/admin/>
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/js/nav_sidebar.js HTTP/1.1" 304 0 <http://localhost:1337/admin/login/?next=/admin/> "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/css/fonts.css HTTP/1.1" 304 0 <http://localhost:1337/static/admin/css/base.css>
    "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/fonts/Roboto-Regular-webfont.woff HTTP/1.1" 304 0 <http://localhost:1337/static/admin/css/fonts.css> "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"
nginx_1  | 192.168.144.1 - - [23/Aug/2021:20:11:00 +0000] "GET /static/admin/fonts/Roboto-Light-webfont.woff HTTP/1.1" 304 0 <http://localhost:1337/static/admin/css/fonts.css> "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/92.0.4515.159 Safari/537.36" "-"

Jednom donete kontejnere:

```sh
docker-compose -f docker-compose.prod.yml down -v
```

### Media Files

Da biste testirali rukovanje medijskim datotekama, počnite tako što ćete kreirati novu reklamu Django:

```sh
docker-compose up -d --build
docker-compose exec web python manage.py startapp upload
```

Dodajte novu aplikaciju na INSTALLED_APPS  navedenu u `settings.py`:

```py
INSTALLED_APPS = [
  "django.contrib.admin",
  "django.contrib.auth",
  "django.contrib.contenttypes",
  "django.contrib.sessions",
  "django.contrib.messages",
  "django.contrib.staticfiles",
  "upload",
]
```

`app/upload/views.py`

```py
from django.shortcuts import render
from django.core.files.storage import FileSystemStorage

def image_upload(request):
  if request.method == "POST" and request.FILES["image_file"]:
    image_file = request.FILES["image_file"]
    fs = FileSystemStorage()
    filename = fs.save(image_file.name, image_file)
    image_url = fs.url(filename)
    print(image_url)
    return render(request, "upload.html", {
      "image_url": image_url
    })

  return render(request, "upload.html")
```

Dodajte `templates` direktorijum u `app/upload` direktorijumu, i potom dodaj novi šablon pod imenom `upload.html`:

```html
{% block content %}

<form action="{% url "upload" %}" method="post" enctype="multipart/form-
data">
    {% csrf_token %}
    <input type="file" name="image_file">
    <input type="submit" value="submit" />
</form>
    {% if image_url %}
    <p>File uploaded at: <a href="{{ image_url }}">{{ image_url }}</a></p>
    {% endif %}
  {% endblock %}
```

`app/hello_django/urls.py:`

```py
from django.contrib import admin
from django.urls import path
from django.conf import settings
from django.conf.urls.static import static
from upload.views import image_upload

urlpatterns = [
    path("", image_upload, name="upload"),
    path("admin/", admin.site.urls),
]

if bool(settings.DEBUG):
  urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)
```

`app/hello_django/settings.py:`

MEDIA_URL = "/media/"
MEDIA_ROOT = BASE_DIR / "mediafiles"

### Razvoj - 2

Testiranje:

```sh
docker-compose up -d --build
```

Trebali biste moći da učitate sliku na <http://localhost:8000/>, a zatim pogledajte
sliku na <http://localhost:8000/media/IMAGE_FILE_NAME>.

### Produkcija - 2

Za proizvodnju dodajte još jednu količinu na Veb i Ngink Services:

```sh
version: '3.8'
services:
web:

build:
context: ./app

dockerfile: Dockerfile.prod

command: gunicorn hello_django.wsgi:application --bind 0.0.0.0:8000

volumes:
- static_volume:/home/app/web/staticfiles
- media_volume:/home/app/web/mediafiles

expose:
- 8000

env_file:
- ./.env.prod

depends_on:
- db

db:
image: postgres:15

volumes:
- postgres_data:/var/lib/postgresql/data/

env_file:
- ./.env.prod.db

nginx:
build: ./nginx

volumes:
- static_volume:/home/app/web/staticfiles

- media_volume:/home/app/web/mediafiles

ports:
- 1337:80

depends_on:
- web

volumes:
postgres_data:
static_volume:
media_volume:
```

Create the "/home/app/web/mediafiles" folder in Dockerfile.prod:

### Stvorite odgovarajuće direktorijume

```sh
ENV HOME=/home/app
ENV APP_HOME=/home/app/web
RUN mkdir $APP_HOME
RUN mkdir $APP_HOME/staticfiles
RUN mkdir $APP_HOME/mediafiles
WORKDIR $APP_HOME
```

Ažurirajte Nginx config ponovo:

```sh
upstream hello_django {
    server web:8000;
}

server {
    listen 80;
    location / {
        proxy_pass http://hello_django;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $host;
        proxy_redirect off;
    }
    location /static/ {
        alias /home/app/web/staticfiles/;
    }
    location /media/ {
        alias /home/app/web/mediafiles/;
    }
}
```

Dodaj sledće u `settings.py`:

```sh
CSRF_TRUSTED_ORIGINS = ["http://localhost:1337"]
```

Ponovo izgradi:

```sh
docker-compose down -v
docker-compose -f docker-compose.prod.yml up -d --build
docker-compose -f docker-compose.prod.yml exec web python manage.py migrate --noinput
docker-compose -f docker-compose.prod.yml exec web python manage.py collectstatic --no-input --clear
```

Ispitajte ga jedan krajnji vreme:

1. Pošaljite sliku na <http://localhost:1337/>.
2. Zatim pogledajte sliku na <http://localhost:1337/media/IMAGE_FILE_NAME>.

Ako vidite `413 Request Entity Too Large` greška, moraćete povećati maksimalnu dozvoljenu veličinu tela za zahtev klijenta na kontekstu servera ili lokacije unutar Nginx config-a.

Primer:

```sh
location / {
    proxy_pass http://hello_django;
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $host;
    proxy_redirect off;
    client_max_body_size 100M;
}
```

Zaključak

U ovom tutorialu smo prošetali kako da kontekturizujemo veb aplikaciju Django sa postgresom za razvoj. Takođe smo stvorili i docer compose spremnu proizvodnju koji dodaje Gunicorn i Nginx u da se bave statičkim i medijskim datotekama. Sada možete da testirate proizvod za proizvodnju lokalno.

U pogledu stvarnog raspoređivanja u proizvodno okruženje, verovatno ćete želeti da koristite:

Potpuno upravljani servis baze podataka - poput RDS-a ili oblaka SQL - umesto da upravljate sopstvenim primerom postgresa unutar kontejnera koji nije root
korisnik za usluge DB i Nginx.

Django na Docker seriji:

1. Docer Django sa postgresom, Guniorn i Nginx (Ovaj vodič!)
2. Osiguravanje kontejnerske aplikacije Django sa šifriranjem
3. Primena Django na AWS sa Docker-om i šifriranjem

Preporučeni tutorijali:

Securing a Deploying Django to Asynchronous Tasks
Containerized Django AWS with Docker and with Django and Celery
Application with Let's Let's Encrypt
Encrypt Michael Herman
