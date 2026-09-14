
# Proširenje Django User modela

Django-ov ugrađeni sistem za autentifikaciju je odličan. Uglavnom ga možemo koristiti odmah po pokretanju, što nam štedi mnogo truda u razvoju i testiranju.
Odgovara većini slučajeva upotrebe i veoma je bezbedan. Ali ponekad je potrebno da izvršimo neka fina podešavanja kako bi se uklopio u našu veb
aplikaciju.

Obično želimo da sačuvamo još nekoliko podataka vezanih za našeg korisnika. Ako vaša veb aplikacija ima društvenu privlačnost, možda ćete želeti da
sačuvate kratku biografiju, lokaciju korisnika i druge slične stvari.

U ovom tutorijalu ću predstaviti strategije koje možete koristiti da jednostavno proširite podrazumevani Django korisnički model, tako da ne morate sve implementirati od nule.

## Načini za proširenje postojećeg User modela

Generalno govoreći, postoje četiri različita načina za proširenje postojećeg modela korisnika.

### Opcija 1: Korišćenje proksi modela

#### Šta je proksi model

To je nasleđivanje modela bez kreiranja nove tabele u bazi podataka. Koristi se za promenu ponašanja postojećeg modela (npr. podrazumevani redosled, dodavanje novih metoda itd.) bez uticaja na postojeću šemu baze podataka.

#### Kada treba da koristim proksi model

Trebalo bi da koristite proksi model da biste proširili postojeći korisnički model kada vam nije potrebno da čuvate dodatne informacije u bazi podataka, već jednostavno dodate metode ili promenite menadžer upita modela.

### Opcija 2: Korišćenje veze "jedan-na-jedan" sa korisničkim modelom (profilom)

#### Šta je veza "jedan-na-jedan"

To je običan Django model koji će imati sopstvenu tabelu baze podataka i održavaće odnos "jedan-na-jedan" sa postojećim korisničkim modelom putem
`OneToOneField`.

#### Kada treba da koristim vezu "jedan-na-jedan"

Vezu "jedan-na-jedan" treba da koristite kada treba da sačuvate dodatne informacije o postojećem korisničkom modelu koje nisu povezane sa procesom
autentifikacije. Obično to nazivamo korisničkim profilom.

### Opcija 3: Kreiranje prilagođenog User modela proširivanjem AbstractBaseUser-a

#### Šta je prilagođeni model korisnika koji proširuje AbstractBaseUser

To je potpuno novi model korisnika koji nasleđuje od `AbstractBaseUser`. Zahteva posebnu pažnju i ažuriranje nekih referenci kroz `settings.py`. Idealno bi bilo da se to uradi na početku projekta, jer će dramatično uticati na šemu baze podataka. Potrebno je posebna pažnja prilikom implementacije.

#### Kada treba da koristim prilagođeni model korisnika koji proširuje AbstractBaseUser

Trebalo bi da koristite prilagođeni model korisnika koji proširuje `AbstractBaseUser` kada vaša aplikacija ima specifične zahteve u vezi sa procesom autentifikacije. Na primer, u nekim slučajevima je logičnije koristiti adresu e-pošte kao  identifikacioni token umesto User imena.

### Opcija 4: Kreiranje prilagođenog User modela proširivanjem AbstractUser-a

#### Šta je prilagođeni model korisnika koji proširuje AbstractUser

To je novi model korisnika koji nasleđuje od AbstractUser. Zahteva posebnu
pažnju i ažuriranje nekih referenci kroz `settings.py`. Idealno bi bilo da se to uradi na početku projekta, jer će dramatično uticati na šemu baze podataka. potrebna je posebna pažnja prilikom implementacije.

#### Kada treba da koristim prilagođeni model korisnika koji proširuje klasu AbstractUser

Trebalo bi da ga koristite kada ste potpuno zadovoljni načinom na koji Django obrađuje proces autentifikacije i ne želite ništa da menjate na njemu. Ipak, želite da dodate neke dodatne informacije direktno u model korisnika, bez potrebe za kreiranjem dodatne klase (kao u opciji 2 ).

## Implementacija

### Proširivanje User modela korišćenjem proksi modela

Ovo je manje nametljiv način proširenja postojećeg modela korisnika. Sa tom strategijom nećete imati nikakvih nedostataka. Ali je u mnogim pogledima veoma ograničena.

Evo kako to radite:

```py
from django.contrib.auth.models import User
from .managers import PersonManager

class Person(User):
  objects = PersonManager()

  class Meta:
    proxy = True
    ordering = ('first_name', )

  def do_something(self):
    ...
```

U gornjem primeru definisali smo proksi model pod nazivom "Person". Django- u kažemo da je ovo proksi model dodavanjem sledećeg svojstva unutar `Meta`
klase: `proxy = True`.

U ovom slučaju sam redefinisao podrazumevani redosled, dodelio prilagođenu funkciju "Manager" modelu i takođe definisao novu metodu "do_something".

Vredi napomenuti da će `User.objects.all()` i `Person.objects.all()` vršiti upit na istoj tabeli baze podataka. Jedina razlika je u ponašanju koje definišemo za proksi model.

Ako je to sve što ti treba, samo napred. Neka bude jednostavno.

### Proširivanje User modela korišćenjem veze "jedan-na-jedan"

Postoji velika verovatnoća da je to ono što želite. Lično, to je metod koji ja uglavnom koristim. Kreiraćemo novi Django model za čuvanje dodatnih
informacija koje se odnose na korisnički model.

Imajte na umu da korišćenje ove strategije rezultira dodatnim upitima ili spajanjima za preuzimanje povezanih podataka. U osnovi, svaki put kada pristupate povezanim podacima, Django će pokrenuti dodatni upit. Ali to se može izbeći u većini slučajeva. Vratiću se na to kasnije.

Obično Django model nazivam kao "Profile":

```py
from django.db import models
from django.contrib.auth.models import User

class Profile(models.Model):
  user = models.OneToOneField(User, on_delete=models.CASCADE)
  bio = models.TextField(max_length=500, blank=True)
  location = models.CharField(max_length=30, blank=True)
  birth_date = models.DateField(null=True, blank=True)
```

Sada se tu dešava magija: sada ćemo definisati signale tako da će naš
`Profile` model biti automatski kreiran/ažuriran kada kreiramo/ažuriramo
instance `User` modela.

```py
from django.db import models
from django.contrib.auth.models import User
from django.db.models.signals import post_save
from django.dispatch import receiver

class Profile(models.Model):
  user = models.OneToOneField(User, on_delete=models.CASCADE)
  bio = models.TextField(max_length=500, blank=True)
  location = models.CharField(max_length=30, blank=True)
  birth_date = models.DateField(null=True, blank=True)

  @receiver(post_save, sender=User)
  def create_user_profile(sender, instance, created, **kwargs):
    if created:
      Profile.objects.create(user=instance)

  @receiver(post_save, sender=User)
  def save_user_profile(sender, instance, **kwargs):
    instance.profile.save()
```

U suštini, povezujemo metode `create_user_profile` i `save_user_profile` sa
modelom `User`, kad god se desi događaj `save()`. Ova vrsta signala se
naziva `post_save`.

Odlična stvar. Kako možete da koristite ovaj način?

Manja prašina. Pogledajte ovaj primer u Django šablonu:

```html
<h2>{{ user.get_full_name }}</h2>
<ul>

<li>Username: {{ user.username }}</li>
<li>Location: {{ user.profile.location }}</li>
<li>Birth Date: {{ user.profile.birth_date }}</li>

</ul>
```

Šta kažete na metod unutar prikaza?

```py
def update_profile(request, user_id):
  user = User.objects.get(pk=user_id)
  user.profile.bio = 'Lorem ipsum dolor sit amet, consectetur adipisicing elit...'
  user.save()
```

Generalno govoreći, nikada nećete morati da pozivate metodu `save()` profila. Sve se radi preko modela `User`.

Šta ako koristim Django Forms?

Da li ste znali da možete obraditi više od jednog formulara odjednom?
Pogledajte ovaj isečak:

`forms.py`

```py
class UserForm(forms.ModelForm):

  class Meta:
    model = User
    fields = ('first_name', 'last_name', 'email')

class ProfileForm(forms.ModelForm):
  class Meta:
    model = Profile
    fields = ('url', 'location', 'company')
```

`views.py`

```py
@login_required
@transaction.atomic
def update_profile(request):
  if request.method == 'POST':
    user_form = UserForm(request.POST, instance=request.user)
    profile_form = ProfileForm(request.POST, instance=request.user.profile)
    if user_form.is_valid() and profile_form.is_valid():
      user_form.save()
      profile_form.save()
      messages.success(request, _('Your profile was successfully updated!'
      return redirect('settings:profile')
    else:
      messages.error(request, _('Please correct the error below.'))
  else:
    user_form = UserForm(instance=request.user)
    profile_form = ProfileForm(instance=request.user.profile)
    return render(request, 'profiles/profile.html', {
      'user_form': user_form, 'profile_form': profile_form
    })
```

`profile.html`

```html
<form method="post">
  {% csrf_token %}
  {{ user_form.as_p }}
  {{ profile_form.as_p }}
  <button type="submit">Save changes</button>
</form>
```

A dodatni upiti u bazu podataka o kojima smo govorili?

O, da. O ovom problemu sam se bavio u drugom postu pod nazivom "Optimizacija upita baze podataka".

Ali, ukratko: Django veze su lenje. To znači da će Django slati upite bazi podataka samo ako pristupite jednom od povezanih svojstava. Ponekad to izaziva neke neželjene efekte, poput pokretanja stotina ili hiljada upita. Ovaj problem se može ublažiti korišćenjem `select_related` metode.

Znajući unapred da ćete morati da pristupite povezanim podacima, možete ih unapred učitati u jednom upitu baze podataka:

```py
users = User.objects.all().select_related('profile')
```

### Proširivanje User modela korišćenjem prilagođenog modela proširivanjem AbstractBaseUser-a modela

Onaj dlakavi. Pa, iskreno, trudim se da ga izbegnem po svaku cenu. Ali ponekad ne možete pobeći od toga. I sasvim je u redu. Teško da postoji tako nešto kao najbolje ili najgore rešenje. Uglavnom postoji manje-više odgovarajuće rešenje. Ako je ovo najprikladnije rešenje za vaš slučaj, samo napred.

Morao sam to jednom da uradim. Iskreno, ne znam da li je ovo najčistiji način da se to uradi, ali evo šta se dešava:

Morao sam da koristim `email` adresu kao token za autorizaciju, a u tom scenariju je to `username` bilo potpuno beskorisno za mene. Takođe, nije bilo potrebe za `is_staff` zastavicom, jer nisam koristio Django Admin.

Evo kako sam definisao svoj korisnički model:

```py
from __future__ import unicode_literals
from django.db import models
from django.core.mail import send_mail
from django.contrib.auth.models import PermissionsMixin
from django.contrib.auth.base_user import AbstractBaseUser
from django.utils.translation import ugettext_lazy as _
from .managers import UserManager

class User(AbstractBaseUser, PermissionsMixin):
  email = models.EmailField(_('email address'), unique=True)
  first_name = models.CharField(_('first name'), max_length=30, blank=True)
  last_name = models.CharField(_('last name'), max_length=30, blank=True)
  date_joined = models.DateTimeField(_('date joined'), auto_now_add=True)
  is_active = models.BooleanField(_('active'), default=True)
  avatar = models.ImageField(upload_to='avatars/', null=True, blank=True)

  objects = UserManager()

  USERNAME_FIELD = 'email'
  REQUIRED_FIELDS = []

  class Meta:
    verbose_name = _('user')
    verbose_name_plural = _('users')

  def get_full_name(self):
    # Returns the first_name plus the last_name, 
    # with a space in between.
    full_name = '%s %s' % (self.first_name, self.last_name)
    return full_name.strip()

  def get_short_name(self):
    #  Returns the short name for the user.
    return self.first_name

  def email_user(self, subject, message, from_email=None, **kwargs):
    #  Sends an email to this User.
    send_mail(subject, message, from_email, [self.email], **kwargs)
```

Želeo sam da ga zadržim što je moguće bližim postojećem User modelu. Pošto nasleđujemo od `AbstractBaseUser` moramo da sledimo neka pravila:

- `USERNAME_FIELD`: String koji opisuje ime polja u modelu korisnika koje se koristi kao jedinstveni identifikator. Polje mora biti jedinstveno (tj. mora biti
  `unique=True` podešeno u svojoj definiciji);

- `REQUIRED_FIELDS` : Lista imena polja koja će biti zatražena prilikom kreiranja korisnika putem `createsuperuser` komande za upravljanje;

- `is_active` : Bulovski atribut koji pokazuje da li se korisnik smatra "aktivnim";

- `get_full_name()`: Duži formalni identifikator za korisnika. Uobičajeno tumačenje bi bilo puno ime korisnika, ali može biti bilo koji string koji identifikuje
  korisnika.

- `get_short_name()`: Kratak, neformalni identifikator za korisnika. Uobičajeno tumačenje bi bilo ime korisnika.

U redu, hajde da idemo dalje. Morao sam i da definišem svoj `UserManager`. To je zato što postojeći menadžer definiše metode `create_user` i `create_superuser`.

Dakle, evo kako moj `UserManager` izgleda:

```py
from django.contrib.auth.base_user import BaseUserManager

class UserManager(BaseUserManager):
  use_in_migrations = True

def _create_user(self, email, password, **extra_fields):
   """
   Creates and saves a User with the given email and password.
   """
   if not email:
     raise ValueError('The given email must be set')

  email = self.normalize_email(email)
  user = self.model(email=email, **extra_fields)
  user.set_password(password)
  user.save(using=self._db)
  return user

  def create_user(self, email, password=None, **extra_fields):
    extra_fields.setdefault('is_superuser', False)
    return self._create_user(email, password, **extra_fields)

  def create_superuser(self, email, password, **extra_fields):
    extra_fields.setdefault('is_superuser', True)
    if extra_fields.get('is_superuser') is not True:
      raise ValueError('Superuser must have is_superuser=True.')

    return self._create_user(email, password, **extra_fields)
```

U osnovi sam očistio postojeći UserManager, uklonio `username` i `is_staff` polja.

Sada poslednji potez. Moramo ažurirati naš `settings.py`. Tačnije
`AUTH_USER_MODEL` svojstvo.

```py
AUTH_USER_MODEL = 'core.User'
```

Na ovaj način govorimo Django-u da koristi naš prilagođeni model umesto podrazumevanog. U gornjem primeru, kreirao sam prilagođeni model unutar aplikacije pod nazivom `core`.

Kako treba da referenciram ovaj model?

Pa, postoje dva načina. Razmotrite model pod nazivom "Course":

```py
from django.db import models
from testapp.core.models import User

class Course(models.Model):
  slug = models.SlugField(max_length=100)
  name = models.CharField(max_length=100)
  tutor = models.ForeignKey(User, on_delete=models.CASCADE)
```

Ovo je sasvim u redu. Ali ako kreirate aplikaciju za višekratnu upotrebu koju želite da učinite dostupnom javnosti, toplo se preporučuje da koristite sledeću
strategiju:

```py
from django.db import models
from django.conf import settings

class Course(models.Model):
  slug = models.SlugField(max_length=100)
  name = models.CharField(max_length=100)
  tutor = models.ForeignKey(settings.AUTH_USER_MODEL, on_delete=models.CASCADE
```

### Proširivanje User modela korišćenjem prilagođenog modela Proširivanje apstraktnog korisnika

Ovo je prilično jednostavno jer klasa

```py
django.contrib.auth.models.AbstractUser
```

pruža potpunu implementaciju podrazumevanog korisnika kao apstraktnog modela.

```py
from django.db import models
from django.contrib.auth.models import AbstractUser

class User(AbstractUser):
  bio = models.TextField(max_length=500, blank=True)
  location = models.CharField(max_length=30, blank=True)
  birth_date = models.DateField(null=True, blank=True)
```

Zatim moramo ažurirati naš `settings.py` koji definiše `AUTH_USER_MODEL` svojstvo.

```py
AUTH_USER_MODEL = 'core.User'
```

Slično kao i kod prethodne metode, ovo bi idealno trebalo uraditi na početku projekta i sa posebnom pažnjom. To će promeniti celu šemu baze podataka.

Takođe, poželjno je kreirati strane ključeve za model korisnika uvozom podešavanja

```py
from django.conf import settings 
```

i pozivanjem na `settings.AUTH_USER_MODEL` umesto direktnog pozivanja na prilagođeni model korisnika.

## Zaključci

U redu! Prošli smo kroz četiri različita načina za proširenje postojećeg User modela. Pokušao sam da vam dam što više detalja. Kao što sam već rekao, ne postoji najbolje rešenje. Zaista će zavisiti od toga šta treba da postignete. Neka bude jednostavno i birajte mudro.

- Proksi model: Zadovoljni ste svim što Django korisnik pruža i ne morate da čuvate dodatne informacije.

- User profil: Zadovoljni ste načinom na koji Django obrađuje autentifikaciju i potrebno je da dodate neke atribute koji nisu povezani sa autentifikacijom
  korisniku.

- Prilagođeni model korisnika iz AbstractBaseUser-a: Način na koji Django obrađuje autorizaciju ne odgovara vašem projektu.

- Prilagođeni model korisnika od AbstractUser-a: Način na koji Django obrađuje autentifikaciju je savršen za vaš projekat, ali ipak želite da dodate
  dodatne atribute bez potrebe za kreiranjem posebnog modela.
  