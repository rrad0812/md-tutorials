
# Prevodi admin strana

[Sadržaj](00_sadrzaj.md)

Django je fantastičan veb okvir Pajtona, a jedna od njegovih sjajnih funkcija je internacionalizacija (ili "i18n"). Prilično je lako dodati prevode na skoro bilo koji string u aplikaciji Django, ali šta je sa
prevođenjem Admin stranice sajta? Naslovi, imena i akcije - svima su potrebni prevodi. Te admin stranice se automatski generišu, pa kako se njihove reči mogu prevesti?

Ovaj vodič vam pokazuje kako je to lako uraditi.

## I18N Pregled

[Sadržaj](00_sadrzaj.md)

Ako ste novi za prevode u Django, prvo pročitajte zvaničnu Translation stranicu. Ukratko, svi stringovi kojima je potreban prevod treba da se prenesu u funkciju prevođenja za Pajton kod ili blok prevod za Django šablonski kod. Django management komande generišu datoteke poruka za specifične za jezik, u kojima prevodioci dodaju prevode za obeležene stringove i konačno ih sastavljaju za upotrebu u aplikaciji. Imajte na umu da prevodi zahtevaju da se `gettext` alati postave na vašu mašinu. Django takođe pruža neku naprednu logiku za rukovanje posebnim slučajevima kao i formatima datuma i pluralizacijom.

## Početno podešavanje

Projektu Django je potrebno malo osnovne konfiguracije pre nego što se urade prevodi, koji su potrebni i za glavnu lokaciju i za admin.

### Omogućavanje internacionalizacije

Budite sigurni da su sledeće postavke date u `settings.py`:

U `settings.py`:

```py
LANGUAGE_CODE = 'en-us' # or other appropriate
codeUSE_I18N = True
USE_L10N = True
```

One su dodate podrazumevano. Booleani trebaju postavljeni na `False` za aplikacije koje ne žele internacionalizaciju sa malim dobicima u brzini, mi ih trebamo tako da će biti `True` da bi se prevodi dešavali.

### Promena lokalnih puteva

[Sadržaj](00_sadrzaj.md)

Podrazumevano se datoteke sa porukama generišu u `locale` direktorijume za svaku aplikaciju sa stringovima označenim za prevod. Možete opciono da postavite `LOCALE_PATHS` da biste promenili puteve. Na primer, možda će biti najlakše staviti sve datoteke sa porukama u jedan takav direktorijum, a ne da ih podelite aplikacijama:

U `settings.py`:

```py
LOCALE_PATHS = [os.path.join(BASE_DIR, 'locale')]
```

Ovo će izbeći umnožavanje prevođenja između aplikacija. To je dobra strategija za male projekte, ali budite upozoreni da to neće dobro skalirati za veće projekte.

### Middleware softver za automatski prevod

[Sadržaj](00_sadrzaj.md)

Django nudi `LocaleMiddleware` da automatski prevodi stranice koristeći "kontekstne tragove" poput prefiksa jezika u URL-u, vrednosti sesija i kolačića. Dakle, ako korisnik pristupa veb lokaciji iz Kine, tada bi trebalo da automatski dobije kineske prevode! Da biste koristili `middleware`, dodajte:

```py
django.middleware.locale.localemiddleware
```

na postavku `MIDDLEWARE` u `settings.py` Proverite da li dolazi posle

```py
SessionMiddleware
CacheMiddleware
```

a pre

```py
CommonMiddleware,
```

ako se koriste i ti `middleware` programi.

U `settings.py`

```py
MIDDLEWARE = [
  #...
  'django.middleware.locale.LocaleMiddleware',
  #...
```

### Prefiks jezika šablona URL-a

[Sadržaj](00_sadrzaj.md)

Dobijanje automatskih prevoda iz kontekstnih tragova je sjajno, ali je ipak korisno imati direktne URL adrese na različite prevedene stranice. Funkcija: `i18n_patterns` može lako dodati kod jezika kao prefiks u obrasce URL-a. Može se primeniti na sve URL adrese za veb lokaciju ili samo podskup URL-ova (kao što je Admin veb lokacija).

Opciono, mogu se postaviti šabloni tako da URL-ovi bez prefiksa jezika će koristiti podrazumevani jezik. Glavna upozorenja za korišćenje `i18n_patterns`-a je da se mora koristiti iz root URLconf-a, a ne iz
uključenih. Projektna `urls.py` datoteka treba da izgleda ovako:

U `urls.py`:

```py
from django.conf.urls.i18n import i18n_patterns
from django.contrib import admin
from django.urls import path

urlpatterns = i18n_patterns(
  ...
  path('admin/', admin.site.urls),
  #...
  #If no prefix is given, use the default language
  prefix_default_language=False
)
```

### Ograničavanje izbor jezika

[Sadržaj](00_sadrzaj.md)

Kada dodate jezičke prefikse u URL adrese, toplo preporučujem ograničavanje dostupnih jezika. Django obuhvata gotove datoteke sa gotovim porukama za nekoliko jezika. Sajt bi izgledalo loše ako je, na primer,
prefiks "/fr/" bio dostupsn bez ikakvih francuskih prevoda. Podesite dostupne jezike koristeći jezike u `settings.py`:

`settings.py`

```py
from django.utils.translation import gettext_lazy as _

LANGUAGES = [
  ('en', _('English')),
  ('zh-hans', _('Simplified Chinese')),
]
```

Primetite da jezički kodovi prate `ISO 639-1` standard.

[Sadržaj](00_sadrzaj.md)
