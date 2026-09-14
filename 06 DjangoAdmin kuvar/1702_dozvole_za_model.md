# Dozvole za model

[Sadržaj](00_sadrzaj.md)

Django dolazi sa ugrađenim sistemom za potvrdu identiteta. Sistem za potvrdu identiteta uključuje:

- korisnike,
- grupe i
- dozvole.

Kada se kreira model, Django će automatski stvoriti četiri podrazumevane dozvole za sledeće akcije:

- `add`: Korisnici sa ovom dozvolom mogu dodati instancu (objekat) modela.
- `delete`: Korisnici sa ovom dozvolom mogu da izbrišu instancu (objekat) modela.
- `change`:Korisnici sa ovom dozvolom mogu da ažuriraju instancu (objekat) modela.
- `view`: Korisnici sa ovom dozvolom mogu da vide instance (objekte) ovog modela.

Ova dozvola je bila dugo očekivana i konačno je dodata u `Django 2.1`.

Imena dozvola slede vrlo specifičnu konvenciju imenovanja:

`<app>.<action>_<modelname>`

Razdvojimo to:

- `<app>` je naziv aplikacije. Na primer, model `User` se uvozi iz aplikacije `auth` (`django.contrib.auth`).
- `<action>` je jedna od gore navedenih akcija (`add`, `delete`, `change` ili `view`).
- `<modelname>` je ime modela, sva mala slova.

Poznavanje ove konvencije imenovanja može vam pomoći da lakše upravljate dozvolama. Na primer, ime dozvole za promenu korisnika je `auth.change_user`.

[Sadržaj](00_sadrzaj.md)
