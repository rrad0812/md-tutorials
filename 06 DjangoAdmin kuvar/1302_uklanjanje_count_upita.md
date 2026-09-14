
# Uklanjanje COUNT upita

[Sadržaj](00_sadrzaj.md)

`COUNT` upit dominira u vremenu učitavanja za prvu stranicu rezultata. Prilikom preskakanja na kasnije stranice, `OFFSET` upit je najsporiji, ali fokusiraću se na poboljšanje `COUNT` u ovoj prvoj opciji.

Ovo može iznenaditi, ali jedno rešenje je potpuno uklanjanje upita za brojanje. Zar to neće slomiti korisnički interfejs !?

Nekako ... U ovom slučaju, moglo bi biti razumno ne znati koliko stranica ima u tabeli "Users". Ne dešava se često da se korisnici nađu na pet milionitoj stranici tabele zapisa. Navigacija do sledeće i prethodne stranice obično je dovoljna kontrola. Korišćenje okvira za pretragu ili filtera je verovatno bolji metod za pronalaženje zapisa koji želite.

Pogledajte Django-ova polja za pretraživanje za informacije o omogućavanju pretraživanja i filtriranja na admin tabli i slobodno pročitajte pganalize članak o potpunom pretraživanju teksta u Djangu.

Najpoznatiji primer ove vrste paginacije je stranica sa rezultatima Google pretrage. Pri dnu stranice, skraćena kontrola paginacije prikazuje direktne veze samo na prvih 10 stranica.

To ne znači da vas Google sprečava da vidite ostale milijarde rezultata. Jednostavno vam govori da bi precizniji termin za pretragu bio bolji način da dođete do tih rezultata od paginacije.

Da se oslobodite `COUNT` smisla u svojoj aplikaciji, Django olakšava skrivanje. Prvo prepišite svojstvo `count` podrazumevanog Paginator -a :

```py
# {project}/users/paginator.py
from django.core.paginator import Paginator
from django.utils.functional import cached_property

class UserPaginator(Paginator):
    @cached_property
    def count(self):
        return 9999999999
```

Primetite da je vrednost čuvara mesta broj koji je mnogo veći nego što očekujete da ćete imati rezultate.

Django reaguje na ovo prilagođavanje tako što prikazuje prvih nekoliko stranica u komponenti paginacije, kao što biste očekivali. Međutim, poslednjih nekoliko stranica će biti lažni. Kada kliknete na stranicu koja ne postoji, Django će vas odvesti na poslednju stranicu bez obzira na to koliko zapisa imate.

Nakon što zamenite podrazumevani paginator, uvezite ga u svoj administratorski model:

`{project}/users/admin.py`

```py
from django.contrib import admin from .models import User
from .paginator import UserPaginator

@admin.register(User)
class UserTableAdmin(admin.ModelAdmin):
    show_full_result_count = False
    paginator = UserPaginator
```

Imajte na umu da sam takođe postavio `show_full_result_count` na `False`. Ovo će isključiti drugi upit za brojanje koji sam ranije primetio.

Nakon ažuriranja aplikacije ovim promenama, smanjio sam vreme za prvu stranicu sa ~5 sekundi na 8ms. Imajte na umu da ova tabela i dalje pati od sporih `OFFSET` upita. Prelazak na stranicu 50,0000 trajao je 18 sekundi. Pre nego što vam pokažem kako da rešite `OFFSET` problem, pokazaću vam još jedan način za poboljšanje `COUNT` upita.

[Sadržaj](00_sadrzaj.md)
