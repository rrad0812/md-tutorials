
# Instalacija i podešavanje Django, PostgreSQL, Nginx i Gunicorn-om na Ubuntu-u

## Uvod

Django je moćan veb frejmvork koji vam može pomoći da pokrenete svoju Python aplikaciju ili veb lokaciju. Django uključuje pojednostavljeni razvojni server za lokalno testiranje vašeg koda, ali za sve što je čak i malo povezano sa produkcijom, potreban je bezbedniji i moćniji veb server.

U ovom vodiču, instaliraćete i konfigurisati neke komponente na Ubuntu 22.04 (ili bilo kojoj drugoj podržanoj verziji Ubuntu-a) da bi podržale i služile Django aplikacije. Podesićete PostgreSQL bazu podataka umesto korišćenja podrazumevane SQLite baze podataka. Konfigurisaćete Gunicorn aplikacijski server da se poveže sa vašim aplikacijama. Zatim ćete podesiti Nginx da koristi obrnuti proksi za Gunicorn, dajući vam pristup njegovim bezbednosnim i performansnim funkcijama za posluživanje vaših aplikacija.

Instaliraćete Django u virtuelnom okruženju. Instaliranje Django-a u okruženje specifično za vaš projekat omogućiće da se vaši projekti i njihovi zahtevi obrađuju odvojeno.

Kada pokrenete bazu podataka i aplikaciju, instaliraćete i konfigurisati Gunicorn aplikacijski server. Ovo će služiti kao interfejs za našu aplikaciju, prevodeći zahteve klijenata iz HTTP u Python pozive koje naša aplikacija može da obradi. Zatim ćete podesiti Nginx ispred Gunicorn-a kako biste iskoristili njegove visoko efikasne mehanizme za rukovanje vezama i njegove lako implementirane bezbednosne funkcije.

## Preduslovi

Ako koristite Ubuntu verziju 16.04 ili stariju, preporučujemo vam da nadogradite na noviju verziju jer Ubuntu više ne podržava ove verzije.

Da biste završili ovaj vodič, potreban vam je server koji pokreće Ubuntu, zajedno sa korisnikom koji nije root sa `sudo` privilegijama i aktivnim zaštitnim zidom. Za smernice o tome kako da ih podesite, izaberite svoju distribuciju sa ove liste i pratite naš Vodič za početno podešavanje servera.

Koraci za podešavanje Django-a, Nginx-a i Gunicorn-a su:

- [Instalirane paketa iz Ubuntu repozitorijuma](#instaliranje-paketa-iz-ubuntu-repozitorijuma)
- [Kreiranje PostgreSQL baze podataka i postgres korisnika](#kreiranje-postgresql-baze-podataka-i-postgres-korisnika)
- [Kreiranje virtuelnog okruženja za projekat](#kreiranje-virtuelnog-okruženja-za-vaš-projekat)
- [Kreiranje i konfigurisanje novovg projekta](#kreiranje-i-konfigurisanje-novog-projekta)
- [Završetak početnog podešavanja projekta](#završetak-početnog-podešavanja-projekta)
- [Testiranje gunicorn sposobnosti da služi projektu](#testiranje-gunicorn-sposobnosti-da-služi-projektu)
- [Kreiranje sistemskih soketa i servisnih datoteka za Gunicorn](#kreiranje-sistemskih-soketa-i-servisnih-datoteka-za-gunicorn)
- [Provera datoteke gunicorn.socket](#provera-datoteke-gunicornsocket)
- [Testiranje aktivacije soketa](#testiranje-aktivacije-soketa)
- [Konfiguracija Nginx proksi servera za Gunicorn](#konfiguracija-nginx-proksi-servera-za-gunicorn)
- [Rešavanje problema sa Nginx i Gunicorn](#rešavanje-problema-sa-nginx-i-gunicorn)
  - [Nginx prikazuje podrazumevanu stranicu](#nginx-prikazuje-podrazumevanu-stranicu)
  - [Nginx prikazuje grešku 502 Bad Gateway](#nginx-prikazuje-grešku-502-bad-gateway)
  - [Django prikazuje: "nije moguće povezati se sa serverom: Veza odbijena"](#django-prikazuje-nije-moguće-povezati-se-sa-serverom-veza-odbijena)
  - [Dalje rešavanje problema](#dalje-rešavanje-problema)

## Instaliranje paketa iz Ubuntu repozitorijuma

[Preduslovi](#preduslovi)

Da biste započeli proces, preuzećete i instalirati sve potrebne stavke iz Ubuntu repozitorijuma. Kasnije ćete koristiti Python menadžer paketa pipda biste instalirali dodatne komponente.

Prvo treba da ažurirate lokalni aptindeks paketa, a zatim preuzmete i instalirate pakete. Paketi koje instalirate zavise od toga koju verziju Pajtona će vaš projekat koristiti.

Ako koristite Django sa Python3, otkucajte:

```sh
sudo apt update
sudo apt install python3-venv python3-dev libpq-dev postgresql postgresql-contrib nginx curl
```

Ova komanda će instalirati alat za kreiranje virtuelnih okruženja za vaše Pajton projekte, Pajton razvojne datoteke potrebne za kasnije izgradnju Gunikorna, Postgres sistem baze podataka i biblioteke potrebne za interakciju sa njim, i Nginx veb server.

## Kreiranje PostgreSQL baze podataka i postgres korisnika

[Preduslovi](#preduslovi)

Sada možete odmah da kreirate bazu podataka i korisnika baze podataka za našu Django aplikaciju.

Podrazumevano, PostgreSQL koristi šemu autentifikacije pod nazivom "peer autentifikacija" za lokalne veze. U osnovi, to znači da ako se korisničko ime korisnika operativnog sistema podudara sa važećim korisničkim imenom `postgres`, taj korisnik može da se prijavi bez dalje autentifikacije.

Tokom instalacije PostgreSQL-a, kreiran je korisnik operativnog sistema pod nazivom `postgres` koji odgovara `postgres` admin korisniku PostgreSQL-a. Potrebno je da koristite ovog korisnika za obavljanje administrativnih zadataka. Možete koristiti `sudo` i proslediti korisničko ime sa `-u` opcijom.

Prijavite se na interaktivnu PostgreSQL sesiju tako što ćete otkucati:

```sh
sudo -u postgres psql
```

Dobićete PostgreSQL prompt gde možete podesiti naše zahteve.

Prvo, kreirajte bazu podataka za svoj projekat:

```sh
CREATE DATABASE myproject;
```

> [!Note]
>
> Svaka PostgreSQL naredba mora da se završi tačkom-zarezom.

Zatim, kreirajte korisnika baze podataka za naš projekat. Obavezno izaberite bezbednu lozinku:

```sh
CREATE USER myprojectuser WITH PASSWORD 'password';
```

Posle toga, izmenićete nekoliko parametara veze za korisnika koga ste upravo kreirali. Ovo će ubrzati rad sa bazom podataka tako da nećete morati da tražite i podešavate ispravne vrednosti svaki put kada se veza uspostavi.

Postavićete podrazumevano kodiranje znakova na UTF-8, što Django očekuje.

```sh
ALTER ROLE myprojectuser SET client_encoding TO 'utf8';
```

Takođe podešavate podrazumevanu šemu izolacije transakcija na "read committed", što blokira čitanja iz nepotvrđenih transakcija.

```sh
ALTER ROLE myprojectuser SET default_transaction_isolation TO 'read committed';
```

Na kraju, podešavate vremensku zonu. Podrazumevano, Django projekti će biti podešeni da koriste UTC. Ovo su sve preporuke iz samog Django projekta :

```sh
ALTER ROLE myprojectuser SET timezone TO 'UTC';
```

Sada možete dati novom korisniku pristup za administriranje nove baze podataka:

```sh
GRANT ALL PRIVILEGES ON DATABASE myproject TO myprojectuser;
```

Kada završite, izađite iz PostgreSQL prompta tako što ćete otkucati:

```sh
\q
```

PostgreSQL je sada podešen tako da se Django može povezati sa svojom bazom podataka i upravljati njome.

## Kreiranje virtuelnog okruženja za vaš projekat

Sada kada imate spremnu bazu podataka, možete početi sa dobijanjem ostalih zahteva za projekat. Instaliraćete Pajton zahteve u virtuelnom okruženju radi lakšeg upravljanja.

Prvo, kreirajte i promenite direktorijum u kome možete čuvati datoteke projekta:

```sh
mkdir ~/myprojectdir
cd ~/myprojectdir
```

Unutar direktorijuma projekta, kreirajte virtuelno okruženje u Pajtonu tako što ćete otkucati:

```sh
python3 -m venv myprojectenv
```

Ovo će kreirati direktorijum pod nazivom `myprojectenv` unutar vašeg `myprojectdir` direktorijuma. Unutar njega će biti instalirana lokalna verzija Pajtona i lokalna verzija pipza upravljanje paketima. Ovu strukturu virtuelnog okruženja možete koristiti za instaliranje i konfigurisanje izolovanog Pajton okruženja za bilo koji projekat koji želite da kreirate.

Pre instaliranja Pajton zahteva vašeg projekta, potrebno je da aktivirate virtuelno okruženje. To možete učiniti tako što ćete otkucati:

```sh
source myprojectenv/bin/activate
```

Vaša komanda bi trebalo da se promeni izgled komandne linije, kako bi naznačio da sada radite u virtuelnom okruženju Pajtona. Izgledaće otprilike ovako:

```sh
(myprojectenv)user@host:~/myprojectdir$
```

Sa aktivnim virtuelnim okruženjem, instalirajte `Django`, `Gunicorn` i `psycopg2` PostgreSQL adapter sa lokalnom instancom `pip`:

> [!Note]
>
> Kada je virtuelno okruženje aktivirano (kada vaš prompt ima `(myprojectenv)`), koristite `pip` umesto `pip3`, čak i ako koristite `Python3`. Kopija alata u  
  virtuelnom okruženju se uvek zove `pip`, bez obzira na verziju Python-a.

```sh
pip install django gunicorn psycopg2-binary
```

Sada bi trebalo da imate sav softver potreban za pokretanje Django projekta.

## Kreiranje i konfigurisanje novog projekta

[Preduslovi](#preduslovi)

Kada su vaše Python komponente instalirane, sada možete da kreirate stvarne Django projektne datoteke.

Pošto već imate direktorijum projekta, reći ćete Django-u da instalira datoteke ovde. On će kreirati direktorijum drugog nivoa sa stvarnim kodom i postaviti skriptu za upravljanje u ovaj direktorijum.

Ključ za ovo je da eksplicitno definišete direktorijum umesto da dozvolite Django-u da donosi odluke u odnosu na naš trenutni direktorijum:

```sh
django-admin startproject myproject ~/myprojectdir
```

U ovom trenutku, vaš direktorijum projekta ( u ovom primeru `~/myprojectdir` ) treba da ima sledeći sadržaj:

- `manage.py` - Django skripta za upravljanje projektima.
- `myproject/` - Paket projekta. Trebalo bi da sadrži datoteke `__init__.py`, `settings.py`,
  `urlspy`, `asgi.py`, i `wsgi.py`.
- `myprojectenv/` - Direktorijum virtuelnog okruženja koji ste ranije kreirali.

Prvo što treba da uradite je da podesite neke vrednosti od važnosti u novokreiranim datotekama projekta. Otvorite datoteku `settings.py` u uređivaču teksta:

```sh
nano ~/myprojectdir/myproject/settings.py
```

Počnite tako što ćete pronaći `ALLOWED_HOSTS` direktivu. Ona definiše listu adresa servera ili imena domena koji se mogu koristiti za povezivanje sa Django instancom. Svi dolazni zahtevi sa `Host` zaglavljem koje nije na ovoj listi izazvaće izuzetak. Django zahteva da ovo podesite kako biste sprečili određenu klasu bezbednosne ranjivosti.

U uglastim zagradama navedite `IP adrese` ili `imena domena` koji su povezani sa vašim Django serverom. Svaka stavka treba da bude navedena pod navodnicima, a unosi treba da budu odvojeni zarezom. Ako želite zahteve za ceo domen i sve poddomene, dodajte tačku na početak unosa. U donjem isečku koda, postoji nekoliko primera bez komentara koji se koriste za demonstraciju:

> [!Note]
>
> Obavezno uključite `localhost` kao jednu od opcija jer ćete proksirati veze preko lokalne Nginx
  instance.

`~/myprojectdir/myproject/settings.py`

```py
...
# Just add the domain name(s) and IP addresses of your Django server
# ALLOWED_HOSTS = [ 'example.com', '203.0.113.5']
#
# To respond to 'example.com' and any subdomains, start the domain with a dot ( . )
# ALLOWED_HOSTS = ['.example.com', '203.0.113.5']
ALLOWED_HOSTS = ['your_server_domain_or_IP', 'second_domain_or_IP', ..., 'localhost']
...
```

Zatim, pronađite odeljak koji konfiguriše pristup bazi podataka. Počinjaće sa `DATABASES`. Konfiguracija u datoteci je za SQLite bazu podataka. Već ste kreirali PostgreSQL bazu podataka za naš projekat, tako da je potrebno da prilagodite podešavanja.

Promenite podešavanja sa informacijama o vašoj PostgreSQL bazi podataka. Recite Django-u da koristi `psycopg2` adapter koji ste instalirali sa `pip`. Potrebno je da navedete ime baze podataka, korisničko ime baze podataka, lozinku korisnika baze podataka, a zatim navedete da se baza podataka nalazi na lokalnom računaru. Možete ostaviti podešavanje `PORT` kao prazan string:

`~/myprojectdir/myproject/settings.py`

```py
...
DATABASES = {
    'default': {
        'ENGINE': 'django.db.backends.postgresql_psycopg2',
        'NAME': 'myproject',
        'USER': 'myprojectuser',
        'PASSWORD': 'password',
        'HOST': 'localhost',
        'PORT': '',
    }
}
...
```

Zatim, idite na dno datoteke i dodajte podešavanje koje pokazuje gde treba da se smeste statičke datoteke. Ovo je neophodno kako bi Nginx mogao da obrađuje zahteve za ove stavke. Sledeći red govori Django-u da ih smesti u direktorijum koji se zove `static` u osnovnom direktorijumu projekta:

`~/myprojectdir/myproject/settings.py`

```py
...
STATIC_URL = 'static/'

# Default primary key field type
# https://docs.djangoproject.com/en/4.0/ref/settings/#default-auto-field

DEFAULT_AUTO_FIELD = 'django.db.models.BigAutoField'

import os
STATIC_ROOT = os.path.join(BASE_DIR, 'static/')
...
```

Sačuvajte i zatvorite datoteku kada završite.

## Završetak početnog podešavanja projekta

[Preduslovi](#preduslovi)

Sada možete migrirati početnu šemu baze podataka u našu PostgreSQL bazu podataka koristeći skriptu za upravljanje:

```sh
~/myprojectdir/manage.py makemigrations
~/myprojectdir/manage.py migrate
```

Kreirajte admin `superuser` korisnika za projekat tako što ćete otkucati:

```sh
~/myprojectdir/manage.py createsuperuser
```

Moraćete da izaberete korisničko ime, unesete adresu e-pošte i izaberete i potvrdite lozinku.

Možete sakupiti sav statički sadržaj u direktorijum koji ste konfigurisali tako što ćete otkucati:

```sh
~/myprojectdir/manage.py collectstatic
```

Moraćete da potvrdite operaciju. Statičke datoteke će zatim biti smeštene u direktorijum koji se zove `static` unutar direktorijuma vašeg projekta.

Ako ste pratili početni vodič za podešavanje servera, trebalo bi da imate `UFW` zaštitni zid koji štiti vaš server. Da biste testirali razvojni server, potrebno je da omogućite pristup portu koji ćete koristiti.

Napravite izuzetak za port `8000` tako što ćete otkucati:

```sh
sudo ufw allow 8000
```

Konačno, možete testirati svoj projekat pokretanjem Django razvojnog servera pomoću ove komande:

```sh
~/myprojectdir/manage.py runserver 0.0.0.0:8000
```

U vašem veb pregledaču posetite ime domena ili IP adresu vašeg servera, a zatim :8000

http://server_domain_or_IP:8000

Trebalo bi da dobijete podrazumevanu Django `index` stranicu.

Ako dodate `/admin` na kraj URL adrese u adresnoj traci, biće vam zatraženo da unesete admin korisničko ime i lozinku koje ste kreirali pomoću `createsuperuser` komande.

Nakon autentifikacije, možete pristupiti podrazumevanom Django admin interfejsu.

Kada završite sa istraživanjem, pritisnite `CTRL-C` u prozoru terminala da biste isključili razvojni server.

## Testiranje gunicorn sposobnosti da služi projektu

[Preduslovi](#preduslovi)

Poslednja stvar koju treba da uradite pre nego što napustite svoje virtuelno okruženje jeste da testirate Gunicorn kako biste bili sigurni da može da služi aplikaciji. To možete učiniti tako što ćete ući u direktorijum projekta i koristiti komandu gunicorn za učitavanje WSGI modula projekta:

```sh
cd ~/myprojectdir
gunicorn --bind 0.0.0.0:8000 myproject.wsgi
```

Ovo će pokrenuti Gunicorn na istom interfejsu na kojem je radio Django razvojni server. Možete se vratiti i ponovo testirati aplikaciju u svom pregledaču.

> [!Note]
>
> Admin interfejs neće imati primenjen nijedan od stilova jer Gunicorn ne zna kako da pronađe
  statički CSS sadržaj odgovoran za ovo.

Prosledili ste modul programu Gunicorn tako što ste naveli relativnu putanju direktorijuma do Django `wsgi.py` datoteke, koja je ulazna tačka za vašu aplikaciju, koristeći Python-ovu sintaksu modula. Unutar ove datoteke, definisana je funkcija `application`, koja se koristi za komunikaciju sa aplikacijom.

Kada završite sa testiranjem, pritisnite `CTRL-C` u prozoru terminala da biste zaustavili Gunicorn.

Sada ste završili sa konfigurisanjem vaše Django aplikacije. Možete se vratiti iz našeg virtuelnog okruženja tako što ćete otkucati:

```sh
deactivate
```

Indikator virtuelnog okruženja u vašem promptu će biti uklonjen.

## Kreiranje sistemskih soketa i servisnih datoteka za Gunicorn

[Preduslovi](#preduslovi)

Testirali ste da li Gunicorn može da komunicira sa našom Django aplikacijom, ali sada bi trebalo da implementirate robusniji način pokretanja i zaustavljanja aplikacijskog servera. Da biste to postigli, napravićete `systemd` servis i `socket` datoteke za gunicorn.

Gunicorn soket će biti kreiran pri pokretanju sistema i slušaće veze. Kada se veza uspostavi, `systemd` će automatski pokrenuti Gunicorn proces za obradu veze.

Počnite tako što ćete kreirati i otvoriti sistemsku soket datoteku za Gunicorn sa `sudo` privilegijama:

```sh
sudo nano /etc/systemd/system/gunicorn.socket
```

Unutra ćete kreirati `[Unit]` odeljak za opisivanje soketa, `[Socket]` odeljak za definisanje lokacije soketa i `[Install]` odeljak da biste bili sigurni da je soket kreiran u pravo vreme:

`/etc/systemd/system/gunicorn.socket`

```sh
[Unit]
Description=gunicorn socket

[Socket]
ListenStream=/run/gunicorn.sock

[Install]
WantedBy=sockets.target
```

Sačuvajte i zatvorite datoteku kada završite.

Zatim, kreirajte i otvorite sistemsku servisnu datoteku za Gunicorn sa `sudo` privilegijama u vašem uređivaču teksta. Ime servisne datoteke treba da se podudara sa imenom soket datoteke, sa izuzetkom ekstenzije:

`sudo nano /etc/systemd/system/gunicorn.service`

Počnite sa `[Unit]` odeljkom, koji se koristi za određivanje metapodataka i zavisnosti. Ovde stavite opis servisa i recite inicijalnom sistemu da ga pokrene tek nakon što se dostigne cilj mreže. Pošto se vaš servis oslanja na soket iz datoteke soketa, potrebno je da uključite Requiresdirektivu koja označava tu vezu:

`/etc/systemd/system/gunicorn.service`

```sh
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target
```

Zatim ćete otvoriti `[Service]` odeljak. Navedite korisnika i grupu pod kojima želite da se proces pokrene. Daćete svom redovnom korisničkom nalogu vlasništvo nad procesom, jer on poseduje sve relevantne datoteke. Daćete grupno vlasništvo nad `www-data` grupom kako bi Nginx mogao lako da komunicira sa Gunicorn-om.

Zatim ćete mapirati radni direktorijum i navesti komandu koja će se koristiti za pokretanje servisa. U ovom slučaju, morate navesti punu putanju do izvršne datoteke Gunicorn, koja je instalirana u našem virtuelnom okruženju. Zatim ćete povezati proces sa Unix soketom koji ste kreirali u direktorijumu `/run` kako bi proces mogao da komunicira sa Nginx-om. Sve podatke ćete evidentirati na standardnom izlazu kako bi `journald` proces mogao da prikuplja Gunicorn logove. Ovde možete navesti i bilo koja opcionalna podešavanja za Gunicorn. Na primer, u ovom slučaju ste naveli 3 radna procesa:

`/etc/systemd/system/gunicorn.service`

```sh
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application
```

Konačno, dodaćete `[Install]` odeljak. Ovo će reći `systemd`-u sa čim da poveže ovu uslugu ako je omogućite da se pokreće pri pokretanju sistema. Želite da se ova usluga pokrene kada je pokrenut i radi običan višekorisnički sistem:

`/etc/systemd/system/gunicorn.service`

```sh
[Unit]
Description=gunicorn daemon
Requires=gunicorn.socket
After=network.target

[Service]
User=sammy
Group=www-data
WorkingDirectory=/home/sammy/myprojectdir
ExecStart=/home/sammy/myprojectdir/myprojectenv/bin/gunicorn \
          --access-logfile - \
          --workers 3 \
          --bind unix:/run/gunicorn.sock \
          myproject.wsgi:application

[Install]
WantedBy=multi-user.target
```

Time je gunicorn sistemska servisna datoteka završena. Sačuvajte je i zatvorite sada.

Sada možete pokrenuti i omogućiti Gunicorn soket. Ovo će kreirati datoteku soketa u `/run/gunicorn.sock` sadašnjem trenutku i pri pokretanju sistema. Kada se uspostavi veza sa tim soketom, `systemd` će automatski pokrenuti `gunicorn.service` da bi je obradio:

```sh
sudo systemctl start gunicorn.socket
sudo systemctl enable gunicorn.socket
```

Možete potvrditi da je operacija bila uspešna proverom datoteke soketa.

## Provera datoteke gunicorn.socket

[Preduslovi](#preduslovi)

Proverite status procesa da biste saznali da li je mogao da se pokrene:

```sh
sudo systemctl status gunicorn.socket

Output
● gunicorn.socket - gunicorn socket
     Loaded: loaded (/etc/systemd/system/gunicorn.socket; enabled; vendor preset: enabled)
     Active: active (listening) since Mon 2022-04-18 17:53:25 UTC; 5s ago
   Triggers: ● gunicorn.service
     Listen: /run/gunicorn.sock (Stream)
     CGroup: /system.slice/gunicorn.socket

Apr 18 17:53:25 django systemd[1]: Listening on gunicorn socket.
```

Zatim, proverite postojanje datoteke gunicorn.sock u /rundirektorijumu:

```sh
file /run/gunicorn.sock

Output
/run/gunicorn.sock: socket
```

Ako je `systemctl status` komanda ukazala na grešku ili ako ne pronađete `gunicorn.sock` datoteku u direktorijumu, to je indikacija da Gunicorn soket nije mogao biti pravilno kreiran. Proverite logove Gunicorn soketa tako što ćete otkucati:

```sh
sudo journalctl -u gunicorn.socket
```

Ponovo pogledajte `/etc/systemd/system/gunicorn.socket` datoteku da biste rešili eventualne probleme pre nego što nastavite.

## Testiranje aktivacije soketa

[Preduslovi](#preduslovi)

Trenutno, ako ste tek pokrenuli `gunicorn.socket` jedinicu, `gunicorn.service` još uvek neće biti aktivna jer soket još nije primio nikakve veze. Ovo možete proveriti tako što ćete otkucati:

```sh
sudo systemctl status gunicorn

Output
○ gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
     Active: inactive (dead)
TriggeredBy: ● gunicorn.socket
```

Da biste testirali mehanizam aktivacije soketa, možete poslati vezu sa soketom tako što ćete otkucati:

```sh
curl --unix-socket /run/gunicorn.sock localhost
```

Trebalo bi da dobijete HTML izlaz iz vaše aplikacije u terminalu. Ovo ukazuje da je Gunicorn pokrenut i da je mogao da opslužuje vašu Django aplikaciju. Možete proveriti da li Gunicorn servis radi tako što ćete otkucati:

```sh
sudo systemctl status gunicorn

Output
● gunicorn.service - gunicorn daemon
     Loaded: loaded (/etc/systemd/system/gunicorn.service; disabled; vendor preset: enabled)
     Active: active (running) since Mon 2022-04-18 17:54:49 UTC; 5s ago
TriggeredBy: ● gunicorn.socket
   Main PID: 102674 (gunicorn)
      Tasks: 4 (limit: 4665)
     Memory: 94.2M
        CPU: 885ms
     CGroup: /system.slice/gunicorn.service
             ├─102674 /home/sammy/myprojectdir/myprojectenv/bin/python3 /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102675 /home/sammy/myprojectdir/myprojectenv/bin/python3 /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             ├─102676 /home/sammy/myprojectdir/myprojectenv/bin/python3 /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application
             └─102677 /home/sammy/myprojectdir/myprojectenv/bin/python3 /home/sammy/myprojectdir/myprojectenv/bin/gunicorn --access-logfile - --workers 3 --bind unix:/run/gunicorn.sock myproject.wsgi:application

Apr 18 17:54:49 django systemd[1]: Started gunicorn daemon.
Apr 18 17:54:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Starting gunicorn 20.1.0
Apr 18 17:54:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Listening at: unix:/run/gunicorn.sock (102674)
Apr 18 17:54:49 django gunicorn[102674]: [2022-04-18 17:54:49 +0000] [102674] [INFO] Using worker: sync
Apr 18 17:54:49 django gunicorn[102675]: [2022-04-18 17:54:49 +0000] [102675] [INFO] Booting worker with pid: 102675
Apr 18 17:54:49 django gunicorn[102676]: [2022-04-18 17:54:49 +0000] [102676] [INFO] Booting worker with pid: 102676
Apr 18 17:54:50 django gunicorn[102677]: [2022-04-18 17:54:50 +0000] [102677] [INFO] Booting worker with pid: 102677
Apr 18 17:54:50 django gunicorn[102675]:  - - [18/Apr/2022:17:54:50 +0000] "GET / HTTP/1.1" 200 10697 "-" "curl/7.81.0"
```

Ako izlaz iz `curl` ili izlaz `systemctl status` ukazuje na to da je došlo do problema, proverite dnevnike za dodatne detalje:

```sh
sudo journalctl -u gunicorn
```

Proverite da li u `/etc/systemd/system/gunicorn.service` datoteci ima problema. Ako napravite izmene u  `/etc/systemd/system/gunicorn.service` datoteci, ponovo učitajte demon da biste ponovo pročitali definiciju servisa i ponovo pokrenite proces Gunicorn kucanjem:

```sh
sudo systemctl daemon-reload
sudo systemctl restart gunicorn
```

Uverite se da ste rešili gore navedene probleme pre nego što nastavite.

## Konfiguracija Nginx proksi servera za gunicorn

[Preduslovi](#preduslovi)

Sada kada je Gunicorn podešen, potrebno je da konfigurišete Nginx da prosledi saobraćaj gunicorn procesu.

Počnite tako što ćete kreirati i otvoriti novi blok servera u Nginx-ovom `sites-available` direktorijumu:

```sh
sudo nano /etc/nginx/sites-available/myproject
```

Unutra, otvorite novi blok servera. Počećete tako što ćete navesti da ovaj blok treba da sluša na normalnom portu 80 i da treba da odgovara na ime domena ili IP adresu vašeg servera:

`/etc/nginx/sites-available/myproject`

```sh
server {
    listen 80;
    server_name server_domain_or_IP;
}
```

Zatim ćete reći Nginx-u da ignoriše sve probleme sa pronalaženjem `favicon`. Takođe ćete mu reći gde da pronađe statičke elemente koje ste sakupili u svom `static/` direktorijumu. Sve ove datoteke imaju standardni URI prefiks `/static`, tako da možete kreirati blok lokacije koji odgovara tim zahtevima: `~/myprojectdir/static`.

`/etc/nginx/sites-available/myproject`

```sh
server {
  listen 80;
  server_name server_domain_or_IP;

  location = /favicon.ico { 
    access_log off; 
    log_not_found off; 
  }
    
  location /static/ {
    root /home/sammy/myprojectdir;
  }
}
```

Konačno, kreirajte `location / {}` blok koji će odgovarati svim ostalim zahtevima. Unutar ove lokacije, uključićete standardnu `proxy_params` datoteku koja je uključena u Nginx instalaciju, a zatim ćete proslediti saobraćaj direktno do Gunicorn soketa:

`/etc/nginx/sites-available/myproject`

```sh
server {
  listen 80;
  server_name server_domain_or_IP;

  location = /favicon.ico { 
    access_log off; 
    log_not_found off; 
  }
    
  location /static/ {
    root /home/sammy/myprojectdir;
  }

  location / {
    include proxy_params;
    proxy_pass http://unix:/run/gunicorn.sock;
  }
}
```

Sačuvajte i zatvorite datoteku kada završite. Sada možete omogućiti datoteku tako što ćete je povezati sa `sites-enabled` direktorijumom:

```sh
sudo ln -s /etc/nginx/sites-available/myproject /etc/nginx/sites-enabled
```

Testirajte svoju Nginx konfiguraciju za sintaksičke greške tako što ćete otkucati:

```sh
sudo nginx -t
```

Ako se ne prijave greške, ponovo pokrenite Nginx tako što ćete otkucati:

```sh
sudo systemctl restart nginx
```

Konačno, potrebno je da otvorite svoj zaštitni zid (fajervol) za normalan saobraćaj na portu 80. Pošto vam više nije potreban pristup razvojnom serveru, možete ukloniti pravilo za otvaranje i porta 8000:

```sh
sudo ufw delete allow 8000
sudo ufw allow 'Nginx Full'
```

Sada bi trebalo da možete da odete na domen ili IP adresu vašeg servera da biste videli svoju aplikaciju.

> [!Note]
> Nakon konfigurisanja Nginx-a, sledeći korak bi trebalo da bude   obezbeđivanje saobraćaja ka serveru  pomoću SSL/TLS-a. Ovo je važno jer se bez njega sve informacije, uključujući lozinke, šalju preko mreže u običnom tekstu.

Ako imate ime domena, najlakši način da dobijete SSL sertifikat za zaštitu vašeg saobraćaja je korišćenje `Let's Encrypt`-a.

Pratite ovaj vodič za Ubuntu 22.04/Ubuntu 20.04/Ubuntu 18.04 da biste podesili `Let's Encrypt` sa Nginx-om na Ubuntu 22.04. Pratite proceduru koristeći Nginx serverski blok koji ste kreirali u ovom vodiču.

## Rešavanje problema sa Nginx i gunicorn

[Preduslovi](#preduslovi)

Ako ovaj poslednji korak ne prikazuje vašu aplikaciju, moraćete da rešite problem sa instalacijom.

### Nginx prikazuje podrazumevanu stranicu

[Preduslovi](#preduslovi)

Ako Nginx prikazuje podrazumevanu stranicu umesto da se povezuje sa vašom aplikacijom, to obično znači da treba da podesite `server_name` unutar datoteke `/etc/nginx/sites-available/myproject` da bi ukazivalo na IP adresu ili ime domena vašeg servera.

Nginx koristi `server_name` da bi odredio koji blok servera da koristi za odgovor na zahteve. Ako dobijete podrazumevanu Nginx stranicu, to je znak da Nginx nije bio u stanju da eksplicitno upari zahtev sa blokom servera, pa se vraća na podrazumevani blok definisan u `/etc/nginx/sites-available/default`.

Blok `server_name` servera u vašem projektu mora biti specifičniji od onog u podrazumevanom bloku servera da bi bio izabran.

### Nginx prikazuje grešku 502 Bad Gateway

[Preduslovi](#preduslovi)

Greška 502 ukazuje da Nginx ne može uspešno da proksira zahtev. Širok spektar problema sa konfiguracijom se izražava greškom 502, tako da je potrebno više informacija za pravilno rešavanje problema.

Glavno mesto za traženje više informacija je u Nginx-ovim evidencijama grešaka. Generalno, ovo će vam reći koji su uslovi izazvali probleme tokom proksiranja. Pratite Nginx evidencije grešaka tako što ćete otkucati:

```sh
sudo tail -F /var/log/nginx/error.log
```

Sada, napravite još jedan zahtev u pregledaču da biste generisali novu grešku ( pokušajte da osvežite stranicu ). Trebalo bi da dobijete novu poruku o grešci koja će biti zapisana u dnevnik. Ako pogledate poruku, trebalo bi da vam pomogne da suzite problem.

Možda ćete dobiti jednu od sledećih poruka:

```sh
connect() to unix:/run/gunicorn.sock failed (2: No such file or directory)
```

Ovo ukazuje da Nginx nije mogao da pronađe `gunicorn.sock` datoteku na datoj lokaciji. Trebalo bi da uporedite `proxy_pass` lokaciju definisanu u okviru `/etc/nginx/sites-available/myproject` datoteke sa stvarnom lokacijom datoteke `gunicorn.sock` koju je generisala `gunicorn.socket` sistemska jedinica.

Ako ne možete da pronađete `gunicorn.sock` datoteku u `/run` direktorijumu, to generalno znači da `systemd` soket datoteka nije mogla da je kreira. Vratite se na odeljak o proveri Gunicorn soket datoteke da biste prošli kroz korake za rešavanje problema za Gunicorn.

```sh
connect() to unix:/run/gunicorn.sock failed (13: Permission denied)
```

Ovo ukazuje da Nginx nije mogao da se poveže sa Gunicorn soketom zbog problema sa dozvolama. Ovo se može desiti kada se postupak sprovede koristeći root korisnika umesto običnog `sudo` korisnika. Dok `systemd` može da kreira Gunicorn soket datoteku, Nginx ne može da joj pristupi.

Ovo se može desiti ako postoje ograničene dozvole u ​​bilo kojoj tački između korenskog direktorijuma ( / ) i `gunicorn.sock` datoteke. Možete pregledati dozvole i vrednosti vlasništva za socket datoteke i svakog od njenih roditeljskih direktorijuma tako što ćete komandi proslediti apsolutnu putanju do vaše socket datoteke namei:

```sh
namei -l /run/gunicorn.sock

Output
f: /run/gunicorn.sock
drwxr-xr-x root root /
drwxr-xr-x root root run
srw-rw-rw- root root gunicorn.sock
```

Izlaz prikazuje dozvole svake od komponenti direktorijuma. Posmatranjem dozvola (prva kolona),
vlasnika (druga kolona) i vlasnika grupe (treća kolona), možete utvrditi koja vrsta pristupa je dozvoljena datoteci soketa.

U gornjem primeru, soket datoteka i svaki od direktorijuma koji vode do soket datoteke imaju svetske dozvole za čitanje i izvršavanje ( kolona dozvola za direktorijume se završava sa `r-x` umesto `---` ). Nginx proces bi trebalo da bude u mogućnosti da uspešno pristupi soketu.

Ako bilo koji od direktorijuma koji vode do soketa nema dozvolu za čitanje i izvršavanje, Nginx neće moći da pristupi soketu bez dozvola za čitanje i izvršavanje ili bez osiguravanja da je vlasništvo nad grupom dato grupi čiji je Nginx deo.

### Django prikazuje: "nije moguće povezati se sa serverom: Veza odbijena"

[Preduslovi](#preduslovi)

Jedna poruka koju možete dobiti od Django-a kada pokušavate da pristupite delovima aplikacije u veb pregledaču je:

```sh
OperationalError at /admin/login/
could not connect to server: Connection refused
    Is the server running on host "localhost" (127.0.0.1) and accepting
    TCP/IP connections on port 5432?
```

Ovo ukazuje da Django ne može da se poveže sa PostgreSQL bazom podataka. Uverite se da je PostgreSQL instanca pokrenuta tako što ćete otkucati:

```sh
sudo systemctl status postgresql
```

Ako nije, možete ga pokrenuti i omogućiti mu da se automatski pokrene pri pokretanju sistema (ako već nije konfigurisan za to) tako što ćete otkucati:

```sh
sudo systemctl start postgresql
sudo systemctl enable postgresql
```

Ako i dalje imate problema, proverite da li `~/myprojectdir/myproject/settings.py` su podešavanja baze podataka definisana u datoteci ispravna.

### Dalje rešavanje problema

[Preduslovi](#preduslovi)

Za dodatno rešavanje problema, evidencije mogu pomoći u sužavanju uzroka. Proverite svaku od njih redom i potražite poruke koje ukazuju na problematična područja.

Sledeći dnevnici mogu biti korisni:

Proverite logove Nginx procesa tako što ćete otkucati:

```sh
sudo journalctl -u nginx
```

Proverite logove pristupa Nginx-u tako što ćete otkucati:

```sh
sudo less /var/log/nginx/access.log
```

Proverite evidenciju grešaka Nginx-a tako što ćete otkucati:

```sh
sudo less /var/log/nginx/error.log
```

Proverite logove aplikacije Gunicorn tako što ćete otkucati:

```sh
sudo journalctl -u gunicorn
```

Proverite logove Gunicorn soketa tako što ćete otkucati:

```sh
sudo journalctl -u gunicorn.socket
```

Kako ažurirate konfiguraciju ili aplikaciju, verovatno ćete morati ponovo pokrenuti procese da biste se prilagodili promenama.

Ako ažurirate svoju Django aplikaciju, možete ponovo pokrenuti Gunicorn proces da biste primenili promene tako što ćete otkucati:

```sh
sudo systemctl restart gunicorn
```

Ako promenite Gunicorn soket ili servisne datoteke, ponovo učitajte demon i restartujte proces kucanjem:

```sh
sudo systemctl daemon-reload
sudo systemctl restart gunicorn.socket gunicorn.service
```

Ako promenite konfiguraciju bloka Nginx servera, testirajte konfiguraciju, a zatim Nginx tako što ćete otkucati:

```sh
sudo nginx -t && sudo systemctl restart nginx
```

Ove komande su korisne za prihvatanje promena dok podešavate konfiguraciju.

## Zaključak

[Preduslovi](#preduslovi)

U ovom vodiču, podesili ste Django projekat u njegovom sopstvenom virtuelnom okruženju.

Konfigurisali ste Gunicorn da prevodi zahteve klijenata kako bi Django mogao da ih obrađuje. Nakon toga, podesili ste Nginx da deluje kao reverzni proksi za rukovanje klijentskim vezama i opsluživanje odgovarajućeg projekta u zavisnosti od zahteva klijenta.

Django pojednostavljuje kreiranje projekata i aplikacija pružajući mnoge uobičajene delove, omogućavajući vam da se fokusirate na jedinstvene elemente. Korišćenjem opšteg lanca alata opisanog u ovom članku, možete lako da pokrećete aplikacije koje kreirate sa jednog servera.

[Preduslovi](#preduslovi)
