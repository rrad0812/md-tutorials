
Baze podataka u Django-u

Objektno-relaciono mapiranje (ORM) je sloj između aplikacije i baze podataka. Ono konvertuje upite napisane u kodu u SQL i konvertuje tabelarne rezultate u objekte.

Ovo pojednostavljuje kreiranje aplikacije na dva načina:

1. Omogućava programerima da se fokusiraju na objekte , a ne na osnovnu bazu podataka.
2. ORM se može ažurirati da koristi drugu bazu podataka bez potrebe za prepisivanjem gomile koda.

Podrazumevana razvojna baza podataka za Django projekte je SQLite. Međutim, možete koristiti i MySQL i PostgreSQL. Tip baze podataka je definisan u settings.py datoteci podešavanjem DATABASES.

DATABASES = {
  'default': {
    'ENGINE': 'django.db.backends.sqlite3',
    'NAME': os.path.join(BASE_DIR, 'db.sqlite3'),
  }
}

Modeli koriste atribute klase za definisanje polja za dati model. Drugim rečima, svaka klasa predstavlja tabelu u bazi podataka . Svaki objekat ima metode za interakciju sa bazom podataka ( Save za umetanje i ažuriranje, Delete za uklanjanje zapisa i Queries za učitavanje zapisa), koje su već uključene u Django. 

U donjem primeru, prva tabela se zove Stavka , a polja u tabeli sadrže informacije o Naslovu , Opisu i Iznosu svake stavke u tabeli, zajedno sa svojstvima svakog polja. Sintaksa za određivanje svojstava polja je: property_name = models.Type(parameters)

from django.db import models

class Item(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField()
    amount = models.IntegerField()

Modeli su obično organizovani na sledeći način:

izbori
polja baze podataka
atributi prilagođenog menadžera

Meta razred
def __str__()
def save()
def get_absolute_url()

Tipovi i svojstva polja

Polja u modelu predstavljaju polja baze podataka tabele koju model definiše. Polja se u Django-u određuju pomoću atributa klase.
Django ima mnogo ugrađenih tipova polja.
Primeri uobičajenih tipova polja prikazani su u tabeli ispod.

Polje 			Primer vrednosti 		Dodatna svojstva

DecimalField 	0,5, 3,14
ImageField 		najbolji_avatar.jpg 	Potvrđuje da je otpremljena datoteka datoteka slike.
DateTimeField 	datum_vreme(1960, 1, 1, 8, 0, d 0)

Različiti tipovi polja mogu imati različite atribute. Ispod su neki primeri, a kompletna lista se nalazi u Django dokumentaciji u odeljku 

ModelField Reference :

Atribut max_length se koristi za definisanje maksimalne dužine CharField-a ili jednog od njegovih podređenih polja kao što je URLField.
Ako null je podešeno na Tačno, polje se može sačuvati kao null, što znači da nema podataka za to polje u datom zapisu.
Atribut blank se može podesiti na vrednost „true“ u tekstualnom polju kako bi se omogućio prazan string.
Atribut default se može koristiti za postavljanje podrazumevane vrednosti za polje.
Atribut choices se koristi za ograničavanje vrednosti koje se mogu sačuvati u tom polju na skup datih izbora.
Atribut help_text se koristi da pomogne korisniku da pronađe unosne stavke.

Imena polja
 
Konvencija u Django-u za nazive polja je da se koriste mala slova i donje crte ( _ ) umesto velikih i malih slova (CamelCasing).
Django primenjuje neka podrazumevana pravila prikaza za imenovanje polja koja treba imati na umu:

Donje crte se zamenjuju razmacima kada se prikazuju first_name prikazuje first name

DŽango automatski piše veliko prvo slovo imena polja blog prikazuje se kao Blog

Django dodaje „s“ imenu polja da bi označio stavke u množini. Ovo možete prilagoditi po potrebi (videti dole).
blog prikazuje se kao Blogs
entry prikazuje se kao Entrys

Django vam omogućava da prilagodite etiketu sledećim parametrima:
verbose_name - omogućava vam da unesete prilagođena imena koja se razlikuju od imena polja. Primer: name = models.CharField('album', max_length=50)

verbose_name_plural - omogućava vam da zamenite Django-ovo podrazumevano podešavanje za imena u množini kada
dodavanje `an` s nije gramatički ispravno. Primer: verbose_name_plural = 'entries'

Primarni ključevi
DŽango automatski kreira id ključ na svakoj tabeli kao primarni ključ. Međutim, možete ovo prilagoditi ako više volite da odredite
primarni ključ umesto da koristite podrazumevani. Ispod su pravila:

1. Ako je polje string , dodajte ga primary_key=True u odeljak parametara deklaracije CharField .
2. Ako je polje ceo broj , dodajte ga primary_key=True u odeljak parameters deklaracije IntField .

Još jedna napomena... ako vam je potrebno da dozvolite null vrednost u BooleanField, koristite NullBooleanField umesto toga.

Odnosi
Polje ForeignKey (više prema jedan)
Da biste označili odnos dete-roditelj, možete koristiti Django-ovu models.ForeignKey komandu za kreiranje odnosa.

class Artist(models.Model):
    name = models.CharField(max_length=50)
    year_formed = models.PositiveIntegerField()

class Album(models.Model):
    name = models.CharField(max_length=50)
    artist = models.ForeignKey(Artist, on_delete=models.CASCADE)

Počevši od Django 2.0, on_delete argument je obavezan za polja ForeignKey . On govori Django-u kako da rukuje objektima na koje se poziva ForeignKey kada se objekat obriše. Opcije su:

model.CASCADE - Briše objekat koji sadrži spoljni ključ.
model.PROTECT - Sprečava brisanje referenciranog objekta izazivanjem greške.
model.SET_NULL - Postavlja spoljni ključ na null; null=True Argument mora biti prisutan da bi ovo funkcionisalo.
model.SET_DEFAULT - Postavlja spoljni ključ na podrazumevanu vrednost; default= Argument mora biti podešen da bi ovo ispravno funkcionisalo.
model.DO_NOTHING - Ne preduzima nikakvu akciju. (Ovo može prouzrokovati grešku integriteta ako vaš bekend baze podataka primenjuje referencijalni integritet.)

Ostala polja odnosa
Polje ManyToManyField ima relaciju „mnogi-prema-mnogi“ i funkcioniše kao polje ForeignKey , ali ima nekoliko dodatnih argumenata o kojima treba voditi računa. Više detalja možete pronaći u Django dokumentaciji .

PoljeOneToOneField ima relaciju jedan-na-jedan i konceptualno je slično stranom ključu ( ForeignKey) sa unique=True , ali će „obrnuta“ strana relacije direktno vratiti jedan objekat.

Skup upita
QuerySet je kolekcija objekata iz vaše baze podataka koja može imati nekoliko filtera za ograničavanje rezultata. Obično se podešava u Django menadžeru modela .
QuerySets se ne izvršava dok se ne prisilno ne izvršava ili ne poziva, što ga čini veoma efikasnim načinom za preuzimanje podataka.

Uobičajene metode upita
all() - skup upita sadrži sve instance modela
create() - prihvata argumente polja korišćenih u modelu i kreira novu instancu
get() - koristi se za dobijanje specifične instance modela određene datim argumentom (npr., id=1); izdaće grešku ako se ne pronađu instance koje odgovaraju argumentu
filter() - koristi se za filtriranje objekata koji odgovaraju datim kriterijumima; vratiće prazan skup upita bez greške ako se ne pronađu odgovarajući objekti
get_or_create() - traži određenu instancu i, ako ona ne postoji, kreiraće je
delete() - briše sve instance unutar preuzetog skupa upita

Menadžeri modela
Menadžer modela je interfejs preko kojeg se operacije upita baze podataka pružaju Django modelima. 

Podrazumevani menadžer
modela za sve Django modele naziva se objekti . Više informacija potražite u Menadžerima .

Međutim, možete kreirati prilagođene menadžere modela proširivanjem osnovne klase Manager. Razlozi zbog kojih biste to možda želeli da uradite uključuju:

Potrebno je dodati dodatne metode menadžera
Potrebno je izmeniti početni QuerySet menadžera

Korišćenje više menadžera na istom modelu:

class AuthorManager(models.Manager):
  def get_queryset(self):
    return super(AuthorManager, self).get_queryset().filter(role='A')

class EditorManager(models.Manager):
  def get_queryset(self):
    return super(EditorManager, self).get_queryset().filter(role='E')

class Person(models.Model):
  first_name = models.CharField(max_length=50)
  last_name = models.CharField(max_length=50)
  role = models.CharField(max_length=1, choices=(('A', _('Author')), ('E', _('Editor'))))
  people = models.Manager()
  authors = AuthorManager()
  editors = EditorManager()

Pozivanje prilagođenih QuerySet metoda iz menadžera:

def editors(self):
  return self.get_queryset().editors()

class Person(models.Model):
  first_name = models.CharField(max_length=50)
  last_name = models.CharField(max_length=50)
  role = models.CharField(max_length=1, choices=(('A', _('Author')), ('E', _('Editor'))))
  people = PersonManager()

Ugrađeni modeli

Django dolazi sa nekim ugrađenim modelima spremnim za upotrebu. Često korišćeni ugrađeni model je model korisnika sa ugrađenom autentifikacijom.

Django korisnički model

Ugrađena polja Primarna polja za Django User model su:

username - Obavezno polje koje ima 150 znakova ili manje i može da sadrži alfanumeričke znakove, kao i _ , @ , + , . , i ~
password - Obavezno polje koje sadrži lozinku korisnika.
email - opciono
first_name - opciono
last_name - opciono
is_staff - Određuje da li ovaj korisnik može pristupiti administratorskoj stranici.
is_superuser - Implicitno određuje da li korisnik ima sve dozvole bez njihovog dodeljivanja.

Atributi
is_authenticated - atribut samo za čitanje koji je uvek Tačno za prijavljenog korisnika

Prilagodite korisnički model

Iako ugrađeni Django User model obavlja osnovne funkcije umesto nas, on možda ne sadrži sve informacije koje želimo o korisniku.
Najčešće preporučeni način za rešavanje ovog problema je proširenje Django User modela.
U nastavku su dva veoma dobra članka koja vas vode kroz proširivanje DJango User modela.

Korišćenje model obrazaca

Django dolazi sa dve osnovne klase za izradu obrazaca. Form vam omogućava da izgradite standardne obrasce, dok ModelForm vam omogućava da izgradite obrasce za kreiranje ili ažuriranje instanci modela.

1. Napravite HTML datoteku koja će sadržati obrazac.

Dodajte novi HTML obrazac za čuvanje obrasca. Primer ispod je datoteka sa oznakom create.html koja opisuje metod i raspored obrasca.

</body>
</html>

2. Izmenite datoteku models.py da biste povezali prikaz sa formularom.
Dodajte sledeću liniju za uvoz da biste koristili klasu ModelForm : from Django.forms import ModelForm
Dodajte klasu koja detaljno opisuje sadržaj formulara. Ispod je primer datoteke model.py koja kreira formular za obradu
podataka za dodavanje muzičkih izvođača:

# models.py
from django.db import models
from django.forms import ModelForm

# Create your models here.
class Artist(models.Model):

""" Define Artist field """
    name = models.CharField('artist', max_length=50)
    year_formed = models.PositiveIntegerField()

class ArtistForm(ModelForm):
class Meta:

        model = Artist
        fields = ['name', 'year_formed']

Podrazumevano, obrazac će biti kreiran kao tabela. Međutim, možete ga lako izmeniti korišćenjem komandi u nastavku:
{{ form.as_p }} koristiće <p> elemente
{{ form.as_ul }} koristiće <li> elemente

3. Izmenite datoteku views.py da biste dodali funkciju za kreiranje forme.
Dodajte sledeću liniju za uvoz: from django.http import HttpResponseRedirect
Dodajte funkciju za obradu podataka koji se obrađuju. Ispod je primer funkcije koja će obraditi zahtev i ili popuniti obrazac
podacima o muzičkom izvođaču ILI će sačuvati informacije ako ih je korisnik dodao:

def artistcreate(request):
if request.method == "GET":

        form = ArtistForm()
return render(request, 'app/create.html', ['form':form });

elif request.method == "POST":
        form = ArtistForm(request.POST)
        form.save()

return HttpResponseRedirect('/artists')

4. Izmenite odeljak urlpatterns u datoteci urls.py
Dodajte obrazac URL-a u urlpatterns odeljak da biste prešli na ispravan prikaz:
url('artists/create', app.views.artistcreate, name='artistcreate')



Uklanjanje polja
Promena polja (promena atributa polja)

Koraci uključeni u migraciju:
1. Izvršite migraciju ( python manage.py makemigrations )
2. Primeni migraciju ( python manage.py migrate )

Proces migracije može izgledati ovako:
1. Izvršite migraciju pokretanjem sledeće komande:

$ python manage.py makemigrations --name <migration_name> <app_name>

Napomena: Svaka migracija će imati drugačije ime. Ako ne dodelite ime, Django će joj automatski dati generičko ime.

2. Pregledajte migracije koje su izvršene.
Da biste videli broj i naziv svake migracije koja je izvršena, koristite sledeću komandu:

$ python manage.py showmigrations <app_name>

Ili da biste videli sve završene migracije, koristite ovu komandu:

$ python manage.py showmigrations --list

Rezultati prikazuju sve migracije koje još nisu izvršene. Obratite pažnju na [ ] simbole. Da je migracija završena, videli biste
[X] .

3. Opciono : SQL izraze koji će se koristiti možete zapravo videti tako što ćete otkucati sledeću komandu gde je „app“ naziv aplikacije,
a „0001_initial“ naziv migracije koju želite da vidite:

$ python manage.py sqlmigrate app 0001_initial

Izlaz će izgledati ovako:



4. Primenite migraciju.
Primenite sve migracije kucanjem komande:

$ python manage.py migrate

Ne morate da primenite sve migracije odjednom. Možete ih primeniti i po aplikaciji ili po aplikaciji i nazivu migracije:

$ python manage.py migrate <app_name> <migration_name>

Za više uslužnih programa komandne linije koji rade sa migracijama, pogledajte odeljak Migracije na stranici Administracija Django-a u
ovoj knjizi.

Najbolje prakse
Pišite modele masnih ćelija
„Pišite debele modele, mršave poglede.“ - Aleksandar Šurigin
To znači da treba da učinite svoje modele robusnim tako što ćete logiku razbiti na male metode unutar vaših modela. Ovo vam
omogućava da ih koristite više puta iz više izvora (administrativni interfejs, korisnički interfejs na frontendu, krajnje tačke API-ja,
višestruki prikazi) u nekoliko redova koda umesto kopiranja i lepljenja tona koda. Dakle, sledeći put kada šaljete korisniku imejl, proširite
model funkcijom imejla umesto da pišete ovu logiku u svom kontroleru.
Ovo takođe olakšava jedinično testiranje vašeg koda jer možete testirati logiku imejla na jednom mestu, umesto više puta u svakom
kontroleru gde se to dešava.
(Šurigin, 2017)

Stil modela
Redosled unutrašnjih klasa modela i standardnih metoda treba da bude sledeći (uz napomenu da nisu svi obavezni):

1. Sva polja baze podataka
2. Atributi prilagođenog menadžera
3. class Meta



        verbose_name_plural = 'people'

Imena polja
Imena polja treba da budu napisana malim slovima, koristeći donje crte ( model_name ) umesto camelCase ( modelName ).
Ne uključujte naziv modela u naziv polja. Ako model Blog ima polje za status, nemojte ga imenovati blog_status . Umesto toga
koristite naziv status . (Stepanov, 2018)

Izbori
Ako choices je definisano za dato polje modela, definišite svaki izbor kao listu torki, sa imenom napisanim velikim slovima kao
atributom klase na modelu. Primer:

class MyModel(models.Model):
    DIRECTION_UP = 'U'
    DIRECTION_DOWN = 'D'
    DIRECTION_CHOICES = [
        (DIRECTION_UP, 'Up'),
        (DIRECTION_DOWN, 'Down'),
    ]

(Projekat DŽango, 2019)

Koristite eksplicitno imenovanje
Koristite eksplicitno imenovanje u svojim modelima od samog početka kako biste izbegli probleme kako vaši modeli postaju sve
složeniji.
Zbog implicitnih pravila imenovanja u DŽangu, sledeće tri linije koda su identične onima u DŽangu:

# All 3 are equivalent!
full_name = models.CharField(max_length=100)
full_name = models.CharField('full name', max_length=100)
full_name = models.CharField(verbose_name='full name', max_length=100)

Kada dodajete relaciona polja, bolje je da eksplicitno definišete imena ovih polja nego da dozvolite Django-u da primenjuje svoja pravila
imenovanja.
(Vinsent, 2018)

Čuvanje korisničkih datoteka
Kada koristite tipove polja FileField() ili ImageField() , definišete putanju gde će se datoteke otpremiti.
Čuvanje više datoteka u jednoj fascikli znači da će sistem datoteka sporije tražiti potrebnu datoteku. Da biste izbegli takve probleme,
možete učiniti sledeće:



Uvek koristite unique i unique_together za logički jedinstvene podatke.
unique=True se koristi u parametrima polja modela da bi se naznačilo da vrednost u ovom polju mora biti jedinstvena u celoj

tabeli.
unique_together=['first_field', 'second_field'] se koristi u meta opcijama da bi se naznačila lista polja koja moraju biti

jedinstvena kada se posmatraju zajedno.
Nikada ne koristite required=False u svakoj oblasti.
