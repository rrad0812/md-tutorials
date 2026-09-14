
# Sigurnost admina

## Sadržaj

[Sadržaj](00_sadrzaj.md)

- **Koristite SSL**

  Postavite svoju lokaciju iza HTTPS-a. Ako ne koristite HTTPS, moguće je da neko uhodi vašu (ili lozinku korisnika) dok ste u kafiću, na aerodromu ili na drugom javnom mestu. Pročitajte više o omogućavanju SSL-a i dodatnim koracima koje ćete možda morati preduzeti u Django dokumentima).

- **Promenite URL**

  Promenite podrazumevani admin URL iz /admin/ u nešto drugo. Uputstva su u Django dokumentaciji, ali ukratko, zamenite admin/ u svojoj URL konfiguraciji nečim drugim:

  ```py
  urlpatterns = [
    path('my-special-admin-login/', admin.site.urls),
  ]
  ```

  Za još veću sigurnost, smestite admin u potpunosti na drugi domen. Ako vam je potrebna još veća sigurnost, poslužite admin iza VPN-a ili nekog drugog mesta koje nije javno.

  Obavezno promenite ime tabele, ime indeksa, kolonu hijerarhije datuma i vremensku zonu.

- **Koristite 'django-admin-honeypot'**

  Nakon što premestite svoju admin lokaciju na novi URL(ili čak odlučite da je hostujete na svom domenu), instalirajte biblioteku `django-admin-honeypot` na vaš stari `/admin/` URL da biste uhvatili pokušaje da hakuju vašu stranu.

  `django-admin-honeypot` generiše lažni ekran za prijavljivanje admina i e-poštom će poslati adminu vaših strana kad god neko pokuša da se prijavi na vaš stari `/admin/` URL.

  E-mail koji je generisao `django-admin-honeypot` sadržaće IP adresu napadača, tako da za dodatnu sigurnost ako primetite ponovljene pokušaje prijave sa iste IP adrese možete toj adresi blokirati upotrebu vaše veb lokacije.

- **Zahtevajte jače lozinke**

  Većina vaših korisnika će odabrati loše lozinke. Omogućavanje validacije lozinke može osigurati da vaši korisnici odaberu jače lozinke, što će zauzvrat povećati sigurnost njihovih podataka i podataka kojima imaju
  pristup u adminu. Zahtevajte jake lozinke omogućavanjem provere lozinke.

  Django dokumentacija ima sjajan uvod o tome kako omogućiti validatore lozinki koji se isporučuju sa Django-om.

  Pogledajte validacije lozinki nezavisnih proizvođača poput `django-zkcvbn-password` da biste lozinke vaših korisnika učinili još sigurnijima. Scot Hacker ima sjajan post o tome šta čini jaku lozinku i primeni biblioteke `python-zkcvbn` u projektu Python.

- **Koristite dvofaktorsku autentifikaciju**

  Dvofaktorska autentifikacija (2FA) je kada vam je potrebna lozinka i nešto drugo za potvrdu identiteta korisnika za vašu veb lokaciju. Verovatno ste upoznati sa aplikacijama koje zahtevaju lozinku, a zatim vam pošalju kod za prijavu pre nego što vam omoguće prijavu, te aplikacije koriste 2FA.
  
  Postoje tri načina na koje možete da omogućite 2FA na svojoj veb lokaciji
  
  - 2FA sa SMS-om, gde sms-om šaljete kod za prijavu. Ovo je bolje od zahteva samo lozinke, ali SMS poruke je iznenađujuće lako presresti.

  - 2FA sa aplikacijom kao što je Google Authenticator, koja generiše jedinstvene kodove za prijavljivanje za bilo koju uslugu na koju se registrujete. Da bi postavili ove aplikacije, korisnici će morati da skeniraju QR kod na vašoj veb lokaciji da bi se registrovali sa svojom aplikacijom. Tada će aplikacija generisati prijavni kod koji mogu koristiti za prijavljivanje na vašu veb lokaciju.

  - 2FA sa Yubi.ey je najsigurniji način da omogućite 2FA na svojoj veb lokaciji. Ova metoda zahteva da vaši korisnici imaju fizički uređaj, Yubi.ey, da se priključe na USBport kada pokušaju da se prijave.
  
  Biblioteka `django-two-factor-auth` može pomoći da omogućite neku od gore napomenutih 2FA metoda.
  
- **Koristite najnoviju verziju Django-a**

  Uvek koristite najnoviju minor verziju Django-a da biste bili u toku sa bezbednosnim ispravkama i ispravkama grešaka. Od ovog pisanja, to je Django 2.0.1. Nadogradite na najnovije dugoročno izdanje (LTS) što je pre moguće, ali svakako uverite se da je vašprojekat nadograđen pre nego što padne iz podrške.

- **Nikada nemojte pokretati `DEBUG` u produkciji**

  Kada je DEBUG postavljen na `True` u vašoj datoteci podešavanja, prikazaće se greške sa potpunim povratnim podacima koji će verovatno sadržati informacije za koje ne želite daih vide krajnji korisnici.

  Možda ćete imati i druga podešavanja ili metode koji su omogućeni samo u režimu otklanjanja grešaka koji mogu predstavljati rizik za vaše korisnike i njihove podatke. Da biste to izbegli, koristite različite datoteke postavkiza lokalni razvoj i za produkcijsku primenu. Pogledajte sjajni uvod Vitora Freitasa o korišćenju više datoteka za podešavanja.

- **Zapamtite svoje okruženje**

  Admin treba izričito da navede u kom ste okruženju kako biste sprečili korisnike daslučajno izbrišu proizvodne podatke. To možete lako postići pomoću biblioteke `django-admin-env-notice` koja će postaviti baner označen bojama na vrh vaše administratorske stranice.

- **Proverite da li postoje greške**

  Ovo nije specifično za Django admina, ali ipak je odlična praksa da zaštitite svojuaplikaciju. Pronađite sigurnosne greške pomoću

  ```shell
  python manage.py check --deploy.
  ```
  
  Ako ovu komandu pokrenete kada lokalno izvodite projekat, verovatno ćete videti neka upozorenja koja neće biti relevantna u proizvodnji. Na primer, podešavanje DEBUG je verovatno `True`, ali već koristite zasebnu datoteku za podešavanja da biste se pobrinuli za to u proizvodnji, zar ne?
  
  Izlaz za ovu naredbu će izgledati otprilike ovako:
  
  ```shell
  ?: (security.V002)
  U svom MIDDLEWARE-u nemate django.middleware.clickjacking.XFrameOptionsMiddleware, tako da se na vašim stranama neće prikazivati zaglavlje x-frame-options. Osim ako postoji dobar razlog da se vaša stranica prikazuje u okviru, trebali biste razmotriti omogućavanje ovog zaglavlja kako biste sprečili napade kliktanjem.
  
  ?: (security.V012)
  SESSION_COOCKIE_SECURE nije postavljeno na True. Korišćenje sigurnog kolačića sesije otežava snifers-ima mrežnog prometa da otmu korisničke sesije.
  ?: (security.V016)
  U svom MIDDLEVARE imate django.middleware.csrf.CsrfViewMiddleware, ali
  CSRF_COOCKIE_SECURE niste postavili na True. Korišćenje CSRF kolačića otežava snifersima mrežnog saobraćaja krađu CSRF tokena.
  ```

  Provera sistema je identifikovala 3 problema.

- **Proverite veb sajt**

  Ovo je još jedan savet koji je specifičan za admina, ali je i dalje dobra praksa. Jednom raspoređen na lokaciji, pokrenite svoju veb stranicu kroz "Sasha's Pony Checkup".Ova stranica će vam dati sigurnosnu ocenu i urednu listu stvari koje treba da uradite da biste je poboljšali.

  Testiraće vašu veb lokaciju na neke stvari koje smo gore naveli, a takođe će preporučiti i druge načine za zaštitu vaše veb lokacije od određenih ranjivosti i vrsta napada.

[Sadržaj](00_sadrzaj.md)
