
# Naivna paginacija

[Sadržaj](00_sadrzaj.md)

Moglo bi se reći da većina veb okvira koristi "naivan" pristup paginaciji. Korišćenjem  PostgreSQL `COUNT`, `LIMIT` i `OFFSET` radi dobro za većinu veb aplikacija, ali ako imate tabele sa milionima zapisa ili više, degradira performanse brzo.

Django je odličan okvir za izradu veb aplikacija, ali njegova podrazumevana metoda paginacije u velikoj meri pada u ovu zamku. U ovom članku ću vam pomoći da razumete ograničenja Django-ove stranice i ponuditi tri alternativne metode koje će poboljšati performanse vaše aplikacije. Usput ćete videti kompromise i slučajeve upotrebe za svaku metodu tako da možete odlučiti koja najbolje odgovara vašoj aplikaciji.

[Sadržaj](00_sadrzaj.md)

## Razumevanje naivne paginacije

Pogledajmo primer vrste kontrole paginacije koju veb aplikacija može da koristi:

- U ovoj paginaciji korisnik može da ode na prethodnu i sledeću stranicu ili da pređe direktno na određenu stranicu. Upit da biste dobili desetu stranicu koristeći Postgres-ov `LIMIT` i `OFFSET` pristup mogao bi izgledati ovako:

```sql
SELECT *
  FROM users
  ORDER BY created_at DESC
  LIMIT 10 OFFSET 100;
```

Uočite da za dobijanje ispravnog `OFFSET` broja stranice morate pomnožiti broj stranice x `LIMIT`.

Potreban je još jedan upit za prikaz naše kontrole paginacije. Morate znati broj zapisa u tabeli. Bez tih informacija nećete znati koliko stranica morate da pretražite.

```sql
SELECT count(*) 
  FROM users
```

Možda ćete videti kako ovaj pristup vrlo brzo može postati pravi problem performansi.

[Sadržaj](00_sadrzaj.md)

## Predstava naivne paginacije

Da biste bolje razumeli uska grla performansi `LIMIT` i `OFFSET`, možete uvesti neke testne podatke i isprobati ih.

Prvo, napravite tabelu dovoljno veliku da naiđe na usporavanja:

```sql
CREATE TABLE USERS ( id serial, name varchar(50) );

INSERT INTO users SELECT
  --- Ten million records 
  generate_series(1,10000000) AS id,
  --- Example: "e6f2c6842d146c518185e1e47add9532"
  substr(md5(random()::text), 0, 50) AS name;
```

Kada pokrenete upit da biste dobili desetu stranicu rezultata, odgovor je gotovo trenutan. Na svom Macbook Pro računaru iz 2018. sa najnovijom verzijom Postgresa vidim podatke u 89 ms.

Međutim, za upite dalje, vreme čekanja se povećava. Uz `LIMIT` od 10 i `OFFSET` od 5.000.000, odgovor traje 2,62 sekunde.

Na kraju, možete pogledati `COUNT` upit koji se izvodi u svih 10 miliona redova. Taj upit sada traje letargičnih 4,45 sekundi.

Spora performansa u ovim primerima je izazvana na način na koji `OFFSET` i `COUNT` rade. Da biste sa `OFFSET` došli do navedene stranice, potrebno je da baza podataka pređe svaki indeks do stranice koju želite. Zbog toga se performanse degradiraju što dalje zavirite u tabelu.

Naivni pristup paginaciji koja koristi `COUNT`, `LIMIT` i `OFFSET` je održivo rešenje za tabele ispod milion redova. U velikim tabelama možete očekivati spore upite i loše korisničko iskustvo.

[Sadržaj](00_sadrzaj.md)

## Rukovanje paginacijom u adminu

[Sadržaj](00_sadrzaj.md)

Sada kada imate osnovno znanje o performansama upita za paginaciju u Postgresu, možete početi da shvatate zašto paginacija usporava u Djangu. Da bih pokazao kako Django upravlja paginacijom, napravio sam novu aplikaciju sa modelom korisnika i ubacio 10 miliona zapisa prilagođavajući naš raniji upit.

`{project}/users/models.py`

```py
from django.db import models

class User(models.Model):
    name = models.CharField(max_length=50)
```

Koristio sam administratorsku stranicu da testiram brzinu paginacije.

U `{project}/users/admin.py`

```py
from django.contrib import admin 
from .models import User

admin.site.register(User)
```

Sada kada je User model predstavljen na admin panelu, mogu videti tabelu sa 10 miliona zapisa.

Koristeći `django-debug-toolbar` mogu zaviriti u SQL upite koje Django generiše u realnom vremenu. Za generisanje ovog korisničkog interfejsa
koriste se dva upita:

```sql
-- Count the total number of records - 2.43 seconds
SELECT COUNT(*) AS " count" 
  FROM "users_user"

-- Get first page of items - 2ms
SELECT "users_user"."id", "users_user"."name" 
  FROM "users_user"
  ORDER BY "users_user"."id" DESC 
  LIMIT 100
```

Ovi upiti bi trebali izgledati poznato jer su gotovo identični gornjim upitima naivne paginacije. Čudno ali upit za brojanje se pokreće dva puta, što znači da kada učitate Django admin panel, morate čekati da baza podataka dva puta prebroji svaki red tabele.

Kada kliknete na stranicu 99,999, Django će ponovo pokrenuti dva upita za brojanje i još jedan upit za paginaciju koristeći LIMIT i OFFSET:

```sql
-- Get the 99,999 page (100 results per page) - 13.34 seconds
SELECT "users_user"."id", "users_user"."name" FROM "users_user"
  ORDER BY "users_user"."id" DESC
  LIMIT 100 OFFSET 9999900
```

Ovom upitu je potrebno 13 sekundi da se završi!

Jasno je da je naivan pristup paginaciji u Djangu spor za velike tabele. Vremenom će vaše tablice baze podataka verovatno rasti, a kako dostignu desetine miliona zapisa, vi i vaši klijenti ćete početi da primećujete ovo užasno vreme učitavanja. Šta možete učiniti u vezi s tim?

U sledećim odeljcima pokazaću vam tri opcije za poboljšanje performansi paginacije u aplikaciji Django.

[Sadržaj](00_sadrzaj.md)
