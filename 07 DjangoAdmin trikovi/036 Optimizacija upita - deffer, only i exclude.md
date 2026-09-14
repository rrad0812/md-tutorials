
# Django optimizacija upita - defer, only i exclude

Ovaj članak razmatra razlike između metoda defer() , only() i u Django
REST Framework-a i

QuerySet-u exclude() .
Docker-a

--
U ovom kursu ćete naučiti kako da
podesite razvojno okruženje sa Dokerom

Upiti u bazi podataka su obično usko grlo većine veb aplikacija. Razumevanje kako
kako biste izgradili i implementirali RESTful

pisati efikasne upite može vam pomoći da značajno ubrzate svoje aplikacije. API koji pokreću Pajton, Django i Django
Prilikom rada sa bazama podataka, pravilo je da je preuzimanje manjeg broja REST Framework.

podataka generalno brže.
Kupi odmah 30 dolara

Da biste minimizirali količinu podataka koji se preuzimaju prilikom rada sa Django-
om, možete koristiti metode defer() , only() i exclude() .

Pogledajte kurs

Hajde da saznamo više o njima i kako da ih koristimo u praksi!

Претражи све туторијале

Uvod u projekat
TEME TUTORIJALA

Tokom celog članka, radićemo na veb aplikaciji za nekretnine. Možete klonirati
projekat iz GitHub repozitorijuma i pratiti dalje, ili jednostavno pročitati članak. api arhitektura avs devops

Django django rest framework

Da biste lokalno podesili projekat, pratite uputstva u datoteci README.md . doker brzi api termos

prednji deo heroku kubernetes

Veb aplikacija ima dva modela, Property i to Location : mašinsko učenje pajton reaguj

red zadataka testiranje vju

# estates/models.py veb skrejping

class Location(models.Model):
city = models.CharField(max_length=128)

SADRŽAJ
state = models.CharField(max_length=128)
country = models.CharField(max_length=32)

Uvod u projekat
zip_code = models.CharField(max_length=32)
# ... Početni benčmark

odloži()
class Property(models.Model):

name = models.CharField(max_length=256) samo()
description = models.TextField()

isključi()
property_type = models.CharField(max_length=20, choices=PROPERTY_TYPES)
location = models.ForeignKey(Location, on_delete=models.CASCADE) Zamka funkcija defer() i only()
square_feet = models.PositiveIntegerField()
bedrooms = models.PositiveSmallIntegerField() Zaključak
bathrooms = models.PositiveSmallIntegerField()
has_garage = models.BooleanField(default=False)
has_balcony = models.BooleanField(default=False)
# ... Istaknuti kurs

Razvoj vođen testiranjem
Oba modela imaju nekoliko polja i sadrže to_json() metodu koja serijalizuje sve pomoću Django-a, Django
atribute modela u Pajton rečnik. REST Framework-a i

Docker-a
Povratne informacije



Veb aplikacija pruža jednostavan API sa sledećim krajnjim tačkama:
Kupi odmah 30 dolara

1. / vraća skraćenu serijalizovanu listu svih svojstava
Pogledajte kurs

2. /<int:id>/ vraća sve detalje određenog svojstva

3. /<int:id>/amenities/ vraća pogodnosti određenog objekta

Početni benčmark
Pre nego što počnemo sa optimizacijama, testiraćemo veb aplikaciju koristeći
Django Silk paket. Django Silk je jednostavan alat za profilisanje i inspekciju uživo
koji čuva HTTP zahteve i upite bazi podataka. Oni se kasnije mogu pregledati na
jednostavnoj kontrolnoj tabli.

Da bismo generisali izveštaje o učinku, možemo da pogodimo krajnje tačke veb
aplikacije:

1. http://127.0.0.1:8000/

2. http://127.0.0.1:8000/1/

3. http://127.0.0.1:8000/1/amenities/

Primetićete da odgovor sa prikazom liste i prikazom sadržaja sadrži podskup
atributa nekretnine, dok detaljni prikaz sadrži sve atribute nekretnine.

Zatim, idite na http://127.0.0.1:8000/silk/requests/ i pregledajte upravo kreirane
izveštaje.

U mom slučaju, prikaz liste traje 765 milisekunde ( 301 od kojih se milisekunde
troše na upite), dok druga dva traju 11 milisekunde ( ~1 na upite).

Posmatrajući SQL upite, možemo videti da sva tri proizvode sličan upit:

-- List query
SELECT * FROM "estates_property"
INNER JOIN "estates_location" ON (

"estates_property"."location_id" = "estates_location"."id"
)

-- Detail query
SELECT * FROM "estates_property"
INNER JOIN "estates_location" ON (

"estates_property"."location_id" = "estates_location"."id"
)
WHERE "estates_property"."id" = 1

-- Amenities query
SELECT * FROM "estates_property"
WHERE "estates_property"."id" = 1

Sva tri prikaza preuzimaju sva polja nekretnina iako prikazi sadržaja i liste prikazuju
samo mali podskup njih. Hajde da ovo popravimo!

odloži()
Možete koristiti ovu defer() metodu prilikom upita Django modela da biste
„isključili“ određena polja koja nisu potrebna za vaš konkretni upit. Na primer, u
prikazu liste možete odložiti sva polja koja ne prikazujete (npr. description ,
created_at i updated_at ).

Ova metoda ima najveći uticaj prilikom odlaganja polja koja sadrže mnogo
podataka (npr. tekstualna polja) ili polja koja zahtevaju skupu obradu da bi se
konvertovala u Pajton objekte (npr. JSON).

defer() radi na nivou atributa. Ispod haube, menja SQL ovako:

Povratne informacije



-- No defer(): The query fetches all the columns
SELECT * FROM some_table;

-- Using defer(): The query fetches all the columns except the deferred ones
SELECT column_1, column_2, ... column_n FROM some_table;

U praksi, mogli bismo da odložimo atribut našeg properties_list_view() QuerySet-
a : description

properties = Property.objects.select_related("location").defer(
"description"

)

Koristimo select_related() da bismo izbegli N+1 upit. Više informacija u
ovom članku .

Ako ponovo uporedimo krajnju tačku, možemo videti da SQL SELECT upit više ne
sadrži description atribut. Pored toga, vreme upita je smanjeno sa 765 milisekundi
na 184 milisekunde. To je poboljšanje od oko 4 puta!

samo()
Metod only() je potpuna suprotnost od defer() metode.

Ova metoda vam omogućava da definišete podskup polja koja želite da se preuzmu
iz baze podataka umesto da odložite određena polja. Najkorisnija je kada se
prikazuje samo mali podskup polja.

Ponovo, radi na nivou atributa i slično menja osnovni SQL:

-- No only(): The query fetches all the columns
SELECT * FROM some_table;

-- Using only(): The query fetches only the provided subset of columns
SELECT only_column1, only_column2, ... only_column_n FROM some_table;

Da bismo ga testirali u akciji, možemo izmeniti properties_list_view QuerySet
ovako:

properties = Property.objects.select_related("location").only(
"id", "name", "location", "price"

)

Nakon testiranja, možemo videti još jedno blago poboljšanje. Vreme upita je
smanjeno sa 184 milisekundi na oko 154 milisekundi.

Ako želite da preuzmete samo određena polja povezanog modela, možete
koristiti Django-ov __ separator pretrage. Na primer: location__city će
preuzeti samo polje grada lokacije.

Pored toga, možete izvršiti više poziva ka defer() i only() u jednom upitu.

isključi()
Metoda exclude() je suprotna od filter() metode. Vraća objekte koji se ne
podudaraju sa datim parametrima pretrage. Dok prethodne dve metode rade na
nivou kolone, ova funkcioniše na nivou reda.

Na primer:

Povratne informacije



# Fetches all the apartments
apartments = Property.objects.filter(property_type=PROPERTY_TYPE_APARTMENT)

# Fetches all the properties that aren't apartments
non_apartments =
Property.objects.exclude(property_type=PROPERTY_TYPE_APARTMENT)

Proizvodi sledeći SQL:

-- Using filter(): The query fetches all the apartments
SELECT * FROM "estates_property" WHERE ("estates_property"."property_type" =
AP)

-- Using exclude(): The query fetches all properties that aren't apartments
SELECT * FROM "estates_property" WHERE NOT
("estates_property"."property_type" = AP)

Isto kao i kod filter() vas, možete da slažete uslove. Na primer, da biste
dobili svojstva koja nisu zemljište i veća su od 1000 stope, možete ovo da
uradite:

big_buildings = (
Property.objects.

exclude(property_type=PROPERTY_TYPE_LAND,
square_feet__lt=1000)
)

Zamka funkcija defer() i only()
defer() Nepravilna upotreba only() može dovesti do problema sa

performansama, a ne do poboljšanja.

Iako možete koristiti ove dve metode da biste isključili polja iz početnog upita,
Django vam ne blokira kasniji pristup isključenim poljima. To potencijalno može
dovesti do dodatnih upita bazi podataka koji pogoršavaju performanse nego ako
biste preuzeli sva polja od samog početka.

Trenutno, naš property_amenities_view izgled je ovakav:

def property_amenities_view(request, id):
property = Property.objects.only(

"id", "has_garage", "has_balcony", "has_basement", "has_pool"
).get(id=id)

return JsonResponse({
"id": property.id,
"has_garage": property.has_garage,
"has_balcony": property.has_balcony,
"has_basement": property.has_basement,
"has_pool": property.has_pool,

})

Prikaz koristi only() da bi se izbeglo preuzimanje nepotrebnih polja iz baze
podataka.

Osnovni SQL izgleda otprilike ovako:

SELECT "estates_property"."id",
"estates_property"."has_garage",
"estates_property"."has_balcony",
"estates_property"."has_basement",
"estates_property"."has_pool"

FROM "estates_property" WHERE "estates_property"."id" = 1

Onda, posle nekog vremena, menjamo mišljenje i odlučujemo da dodamo
bedrooms i bathrooms u JSON odgovor, pošto su i oni tehnički pogodni:

Povratne informacije



def property_amenities_view(request, id):
property = Property.objects.only(

"id", "has_garage", "has_balcony", "has_basement", "has_pool"
).get(id=id)

return JsonResponse({
"id": property.id,
"bedrooms": property.bedrooms, # new
"bathrooms": property.bathrooms, # new
"has_garage": property.has_garage,
"has_balcony": property.has_balcony,
"has_basement": property.has_basement,
"has_pool": property.has_pool,

})

Time smo upravo uveli dva dodatna upita. Pošto nismo uključili bedrooms i
bathrooms u only() podskup, Django sada zasebno preuzima dodatna polja.

Osnovni SQL sada izgleda ovako:

-- This query fetches the only() field subset
SELECT "estates_property"."id",

"estates_property"."has_garage",
"estates_property"."has_balcony",
"estates_property"."has_basement",
"estates_property"."has_pool"

FROM "estates_property" WHERE "estates_property"."id" = 1

-- An extra query for fetching the bedrooms
SELECT "estates_property"."id",

"estates_property"."bedrooms"
FROM "estates_property" WHERE "estates_property"."id" = 1

-- An extra query for fetching the bathrooms
SELECT "estates_property"."id",

"estates_property"."bathrooms"
FROM "estates_property" WHERE "estates_property"."id" = 1

Zapamtite, prilikom refaktorisanja koda, trebalo bi da budete oprezni sa only() i
exclude() .

Takođe možete da se zaštitite od pojavljivanja dodatnih SQL upita prilikom
rada na tim prikazima u budućnosti korišćenjem django_assert_num_queries
. Za više informacija o ovome, pogledajte članak Automatizacija testiranja
performansi u Django-u .

Zaključak
Da biste ubrzali svoje Django upite, trebalo bi da pokušate da preuzmete minimalnu
količinu podataka iz baze podataka. Ovo se odnosi i na redove (objekte modela) i
na kolone (atribute modela).

Dve metode koje vam omogućavaju da kontrolišete koji se atributi modela
preuzimaju su defer() i only() . Ove dve imaju najznačajniji uticaj kada se
isključuju atributi koji sadrže mnogo podataka ili zahtevaju skupu obradu.

S druge strane, da biste napravili izuzeća na nivou objekta, možete iskoristiti
exclude() .

Za više saveta za optimizaciju Django-a, pogledajte:

• Saveti za optimizaciju performansi Django-a

• Približno brojanje u Django-u i Postgres-u

• Automatizacija testiranja performansi u Django-u

• Efikasno korišćenje Django REST Framework serijalizatora

 Django

Povratne informacije



Nik Tomazić
Nik je softverski programer iz Slovenije. Zainteresovan je
za objektno orijentisano programiranje i veb razvoj. Voli
da uči nove stvari i prihvata nove izazove. Kada ne
programira, Nik ili pliva ili gleda filmove.

 

PODELI OVAJ TUTORIJAL

 Tviter  Redit  Hakerske vesti  Fejsbuk

PREPORUČENI TUTORIJALI 

Budite u toku sa ažuriranjima kursa
Pridružite se našoj mejling listi da biste bili obavešteni o

ažuriranjima i novim izdanjima.

Унесите своју имејл адресу Pretplatite se

UČI

Kursevi Paketi Blog  TestDriven.io je
ponosni pristalica
otvorenog koda

VODIČI
10% profita od svakog našeg FastAPI

Kompletan Pajton Django i celer Dubinski zaron u Flask
kursa i našeg Flask kursa za veb
razvoj biće donirano FastAPI i Flask

O TESTDRIVEN.IO timovima, respektivno.

Podrška i konsultacije Šta je razvoj vođen testiranjem? Svedočanstva Pratite naše doprinose

Donacije otvorenog koda O nama Upoznajte autore Saveti i trikovi

© Autorska prava 2017–2025 TestDriven Labs.
Razvio Majkl Herman .

Прати @testdrivenio

Povratne informacije