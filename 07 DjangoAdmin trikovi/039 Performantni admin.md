Best Practises for A Performant Django Admin

Adin Hodovic

Administratorski interfejs koji dolazi sa Django-om je jedna od sjajnih stvari kod Django-a. Dolazi
sa mnoštvom funkcija odmah po instalaciji i ima mnogo paketa otvorenog koda koji dodatno
proširuju osnovnu funkcionalnost. Dobro je dokumentovan i radi veoma dobro, jedina bolna tačka
koju sam pronašao kada sam koristio administratorski interfejs i njegove funkcije je kada sam
imao velike tabele koje sadrže milione objekata. U tom slučaju, pretraživanje, sortiranje,
uređivanje, brojanje i druge funkcije uzrokuju sporo učitavanje administratorskog interfejsa i
stvaraju opterećenje baze podataka. U tom trenutku moramo optimizovati administratorski
interfejs i postoji mnogo malih promena koje možete učiniti da biste ubrzali vreme učitavanja
vašeg administratorskog interfejsa i smanjili svako dodatno opterećenje baze podataka. Ovaj blog
post će opisati pristupe uobičajenim problemima sa performansama koje sam iskusio kada imam
veliku bazu podataka.

Definišite koja polja se mogu sortirati

Podrazumevano, Django administratorski panel dozvoljava sortiranje bilo kojih polja jednim
klikom. Ovo može dovesti do neželjenih rezultata sa velikim skupom podataka, gde sortiranje po
neindeksiranom polju može poslati težak upit bazi podataka, što uzrokuje opterećenje baze
podataka i vremenski istek zahteva. Ako mnogo administratorskih korisnika istovremeno koristi
sortiranje, to može izazvati kaskadne događaje, smanjujući performanse veb servera i baze
podataka. Da biste ih onemogućili, možete definisati podrazumevana polja koja se mogu sortirati
kao što je prikazano u nastavku:

class BaseAdmin(ModelAdmin):
sortable_by: tuple = ("created", "modified")

Ovo je definisano u BaseAdminklasi, a zatim se može konfigurisati za svakog administratora za
model.

Uklonite redosled ili osigurajte redosled na indeksiranim poljima

Redosled može biti veliki za bazu podataka, posebno kod velikih skupova podataka.
Podrazumevano, Django Admin koristi podrazumevani redosled skupova upita, ovo se takođe
može prebrisati u administratorskom delu pomoću orderingpolja . Možete ga definisati i koristiti
polja baze podataka koja su indeksirana kao u sledećem rešenju.

class BaseAdmin(ModelAdmin):
ordering: tuple = ("created", "modified")



Ili možete potpuno ukloniti redosled uklanjanjem polja za redosled, ali i osigurati da
Modelpodrazumevani skup upita nema nikakav redosled.

class User(Model):

class Meta:
ordering = ["created", "modified"] # Remove this

Obratite pažnju na to po koliko polja sortirate i da li su polja indeksirana.

Smanjite listu objekata po stranici

Podrazumevano, Django prikazuje 100 objekata po stranici u changelistprikazu. Možemo
smanjiti ovaj broj da bismo ubrzali vreme učitavanja.

# https://docs.djangoproject.com/en/4.1/ref/contrib/admin/
#django.contrib.admin.ModelAdmin.list_per_page
LIST_PER_PAGE = 20

Paginator velikih tabela (procena broja)

Podrazumevano, Django ima tačnu procenu broja koja na malim tabelama funkcioniše dobro,
međutim, izvršavanje brojanja na 80 miliona objekata oduzima vreme i usporava administraciju.
Da bi se ovo rešilo, Postgres podržava reltuplesprocenu broja, a ne precizan broj objekata čije
izračunavanje zahteva vreme. Očigledan nedostatak je što broj objekata nije tačan, ali da li je to
važno? Ako, recimo, u tabeli postoji 4,4 miliona objekata. Da li vas brine ako piše procena od
4400000, a ne stvarna vrednost od 4400112? Ako je tako, ne bi trebalo da koristite procenjeni
broj. Procena broja takođe ne bi trebalo da se koristi na manjim tabelama. Da biste ovo
implementirali, postoji GitHub gist koji sam podelio. Ispod je primer.

class LargeTablePaginator(Paginator):
"""

    # https://gist.github.com/noviluni/d86adfa24843c7b8ed10c183a9df2afe
    Overrides the count method of QuerySet objects to avoid timeouts.
    - Get an estimate instead of actual count when not filtered (this estimate 
can be stale and hence not fit for
    situations where the count of objects actually matter).
    """

@cached_property
def count(self):

"""
        Returns an estimated number of objects, across all pages.



        """
if not self.object_list.query.where: # type: ignore

try:
with connection.cursor() as cursor:

# Obtain estimated values (only valid with PostgreSQL)
cursor.execute(

"SELECT reltuples FROM pg_class WHERE relname = %s",
[self.object_list.query.model._meta.db_table], # type: 

ignore
)
estimate = int(cursor.fetchone()[0])
return estimate

except Exception: # pylint: disable=broad-except
# If any other exception occurred fall back to default 

behaviour
pass

return super().count

Sada želimo da budemo u mogućnosti da dinamički koristimo LargeTablePaginatorunutar
get_paginatorfunkcije:

class BaseAdmin(ModelAdmin):
large_table_paginator = False

def get_paginator( # pylint: disable=too-many-arguments
self,
request,
queryset,
per_page,
orphans=0,
allow_empty_first_page=True,

):
# Always show count locally
if self.large_table_paginator and not settings.DEBUG:

return LargeTablePaginator(
queryset, per_page, orphans, allow_empty_first_page

)
return self.paginator(queryset, per_page, orphans,

allow_empty_first_page)

Onemogućavamo paginator velikih tabela u režimu za debagovanje, koji je obično podešen u



lokalnom razvoju da bi se lokalno prikazao kompletan broj objekata. Konačno, možete poništiti
large_table_paginatorbulovsku vrednost po administratoru:

class BaseAdmin(ModelAdmin):
large_table_paginator = True

Onemogući brojanje svih rezultata

Na primer, broj rezultata se prikazuje kada izvršite pretragu ili filter. Pored trake za pretragu će
pisati 0 results (1 total), 1 totalpreuzima broj objekata za tu tabelu. Kao što smo ranije
videli, to može biti težak proces na velikim tabelama. Stoga, Django nudi opciju da onemogućite
brojanje podešavanjem show_full_result_countsvojstva na False. Sada neće prikazivati koliko
objekata postoji za tu tabelu, već će ponuditi opciju za uklanjanje svih filtera i prikazivanje svih
objekata 0 results (Show all). Ispod je primer.

class BaseAdmin(ModelAdmin):
show_full_result_count = False

Ukloni detaljne preglede hijerarhije datuma

Django date_hierarchyje odlična funkcija za administratora, ali može biti skupa za bazu
podataka i prouzrokovati dugo vreme učitavanja u korisničkom interfejsu. Django će
podrazumevano izvršiti upit da bi video za koje raspone datuma postoje objekti. Na primer, ako
imate 10 korisnika koji su se pridružili u januaru, martu i avgustu, Django će izvršiti upit i odabrati
različite korisnike po mesecu, ako biste išli na jedan mesec, izvršiće upit da bi odabrao različite
korisnike za svaki dan, prikazujući samo mesec i dane u korisničkom interfejsu za kada su se
korisnici pridružili. Ovo zvuči odlično i funkcioniše dobro za male skupove podataka. Ali ako biste
imali milion korisnika, onda bi detaljna analiza grupisala sve njih po danu kako bi se osiguralo da
samo dani kada su se korisnici pridružili postanu opcije u korisničkom interfejsu.

Filtriranje objekata i prikazivanje samo liste važećih dana ili meseci je ispravno ponašanje, ali nije
optimalno za velike skupove podataka. Ovo možemo zaobići prikazivanjem svih datuma bez obzira
na to da li postoje objekti za te datume, preskačući teške upite bazi podataka. Postoji Django paket
pod nazivom django-admin-lightweight-date-hierarchyi blog post koji je napisao Haki
Benita koji opširno objašnjava problem i rešenje .

Upotreba je jednostavna. Instalirajte paket poetry add django-admin-lightweight-date-
hierarchyi dodajte ga u instalirane pakete.

INSTALLED_APPS = (
...
'django_admin_lightweight_date_hierarchy',
...

)



Zatim postavite date_hierarchy_drilldownzastavicu na Falseda biste je onemogućili.

class BaseAdmin(ModelAdmin):
date_hierarchy = "created"
date_hierarchy_drilldown = False # Disable drilldowns

Takođe možete prilagoditi koji datumi treba da budu dostupni. Podrazumevano sam izabrao da
prikazujem samo prošle datume (objekti se ne kreiraju u budućnosti) i prikazujem samo godine
počev od kada je aplikacija imala svoje prve objekte.

class BaseAdmin(ModelAdmin):

def get_date_hierarchy_drilldown(self, year_lookup, month_lookup):
"""Drill-down only on past dates."""

today = timezone.now().date()

if year_lookup is None and month_lookup is None:
# Applications first year in production
apps_first_year = 2022
return (

datetime.date(y, 1, 1) for y in range(ums_first_year,
today.year + 1)

)

if year_lookup is not None and month_lookup is None:
# Past months of selected year.
this_month = today.replace(day=1)
return (

month
for month in (

datetime.date(int(year_lookup), month, 1) for month in
range(1, 13)

)
if month <= this_month

)

if year_lookup is not None and month_lookup is not None:
# Past days of selected month.
days_in_month = calendar.monthrange(year_lookup, month_lookup)[1]
return (



day
for day in (

datetime.date(year_lookup, month_lookup, i + 1)
for i in range(days_in_month)

)
if day <= today

)

Da biste razumeli nedostatke ponude svih raspona datuma, slika je prikazana ispod.

Kao što vidite na slici, imamo samo 2 objekta, ali prikazujemo sve mesece, paket koji smo
instalirali pomaže u tome. Očigledno je da je filtriranje 2 objekta po datumu, a zatim prikazivanje
samo jednog meseca kao opcije detaljnije analize bolje. Međutim, kao što je ranije pomenuto, ovo
funkcioniše samo sa malim skupovima podataka. Stoga, omogućite i onemogućite detaljniju
analizu po administratorskom nalogu modela i veličini tabele.

Svojstva modela keša

Definisanje prilagođenih svojstava modela, a zatim korišćenje tih svojstava za prikazivanje polja
korisničkog interfejsa u administrativnom delu je uobičajena praksa. Međutim, korišćenje jednog
svojstva u više odeljaka administrativnog dela izazvalo bi prekomerne upite bazi podataka, a
svojstva se mogu keširati pomoću @cached_propertydekoratora. Hajde da pogledamo kako se to
koristi.

class MyModel(Model):

@cached_property
def computational_heavy_query(self):

"""
        Run a heavy query
        """

return (
self.objects.filter(

tag__in=['a', 'b', 'c'],
name__icontains='test'

)



.order_by("-created", "-modified")

.distinct("id")

.all()
)

Sada ćemo u administraciji koristiti ovo svojstvo u više fields.

class BaseAdmin(ModelAdmin):

fields = ("computational_heavy_query_field_1",
"computational_heavy_query_field_2")

readonly_fields = ("computational_heavy_query_field_1",
"computational_heavy_query_field_2")

def computational_heavy_query_field_1(self, obj):
computational_heavy_query = obj.computational_heavy_query
return mark_safe(

f"""
            <p>{computational_heavy_query} 1</p>
            """

)

def computational_heavy_query_field_2(self, obj):
computational_heavy_query = obj.computational_heavy_query
return mark_safe(

f"""
            <p>{computational_heavy_query} 2</p>
            """

)

Pošto svojstvo modela koristi keširanje dodavanjem cached_propertydekoratora, računski
zahtevni upit će se izvršiti samo jednom čak i ako se svojstvo koristi dva puta u administraciji.
Keširana funkcija svojstva će se izvršiti jednom po instanci objekta i trajaće sve dok instanca
postoji. Ako ne koristimo cached_propertydekorator, upit bi se izvršio dva puta. Odličan način za
ubrzavanje teških upita na nivou modela, što će zatim ubrzati administraciju!

Pretraga

Django admin ima ugrađenu podršku za pretragu. Radi koristeći search_fieldspromenljivu i
podrazumevano koristi icontainsza pretražene reči. Radi prilično dobro, ali postaje spor na
velikim skupovima podataka, što je obično slučaj sa pretragama. Pogledaćemo kako optimizovati
pretragu.



Minimizirajte broj polja za pretragu

Što više polja dodate, search_fieldsviše icontainsupita će biti vršeno nad tim poljima. Dodajte
samo ona search_fieldskoja će se intenzivno koristiti, dodavanje više polja će pružiti lepu
funkcionalnost, ali će usporiti pretragu.

Pretrage polja za pretragu

Podrazumevano, Django će koristiti icontainsu upitu baze podataka za polja definisana u
search_fields, to je najbolja generička opcija, ali nije optimalna za razna polja. Na primer, za
UUID-ove, brojeve, statičke stringove možda ćete želeti da koristite , exactšto će biti mnogo
jeftiniji upit od podrazumevanog icontains. Evo primera:

class BaseAdmin(ModelAdmin):

search_fields = ("id__exact", "company_id__exact", "name")

Ispod je kompletna lista pretraga koje možete koristiti kako smatrate da je najbolje, a evo i opisa
pretraga .

list_of_lookups = ['exact', 'iexact', 'gt', 'gte', 'lt', 'lte', 'in',
'contains', 'icontains', 'startswith', 'istartswith', 'endswith', 'iendswith',
'range', 'isnull', 'regex', 'iregex', 'contained_by']

Uklonite polja za automatsko dovršavanje

Оптимизација поља која се аутоматски попуњавају, а која се могу претраживати, може
побољшати перформансе администратора. Стога, проверите која поља се аутоматски
попуњавају и покушајте да их уклоните и видите да ли то решава проблеме са
перформансама приликом уређивања објеката.

@admin.register(Category)
class CategoryAdmin(admin.ModelAdmin):

list_display = ("id", "name", "emoji")
search_fields = ("name",)

@admin.register(Repository)
class RepositoryAdmin(SearchAdmin):

autocomplete_fields = []

DjangoQL

Интеграција претраге „из кутије“ функционише добро, али постаје компликованија за
коришћење са сложеним упитима. На пример, ако желите да претражите објекат са више



поља која су True, онда би то било немогуће или веома компликовано урадити са
подразумеваном административном страном. DjangoQLПакет пружа интеграцију
административне стране која административну страну претвара у напредно поље за
претрагу са аутоматским довршавањем и напредним језиком за претрагу. Пакет се може
пронаћи наГитХаб . Додавање пакета ће вам омогућити да вршите упите како new = True
and date_published ~ "2023-09"бисте пронашли, рецимо, било коју објаву на блогу која
има поље newподешено на Trueи објављена је у 9. месецу 2023. године. Једноставна је за
инсталацију и интеграцију са Дјанго административном страном. Инсталирајте га помоћу
poetry add djangoqlи додајте га у инсталиране апликације.

INSTALLED_APPS = [
...
'djangoql',
...

]

А затим додајте миксин администратору:

from djangoql.admin import DjangoQLSearchMixin

class BaseAdmin(DjangoQLSearchMixin, ModelAdmin):
...

Постоји много других подешавања и прилагођавања које можете да урадите са DjangoQL
пакетом, што је детаљно описано на GitHub страници пакета.

У Django администрацији можемо приказати повезане објекте користећи уграђене линије
или доње црте за приступ повезаним пољима, као на пример company__name. Међутим, то
би могло створити неефикасне упите приликом преузимања повезаних објеката. Django
пружа две опције, коришћење select_relatedи prefetch_relatedза преузимање
повезаних објеката.

Постоји широк спектар ресурса о претходном учитавању и одабиру сродних ресурса. У
наставку ћу дати кратак резиме и пример употребе.

Избором повезаних поља спојићете повезане табеле и преузети сва потребна поља и из
главне табеле и из повезане табеле одједном. Има смисла користити select_relatedу
случајевима када преузимамо листу објеката и желимо истовремено да преузмемо повезане
објекте за сваки објекат на тој листи. Постоје два приступа, први је коришћење ,
list_select_relatedшто ће рећи Django-у да користи , select_relatedали само на
listстраници.

class MyAdmin(BaseAdmin):
list_select_related = ('company',)



Други приступ је да се get_querysetфункција препише да би се изабрали повезани објекти
који ће радити не само на listстраници већ на свим администраторским страницама за тај
модел.

class MyAdmin(BaseAdmin):

def get_queryset(self, request):
return super().get_queryset(request).select_related("company")

Претходно учитавање повезаних објеката функционише другачије и функционише за M2M
релације, док select_relatedне ради. Претходно учитавање повезаних објеката
функционише тако што се врши спајање у , Pythonа не у SQL. Прво се преузимају сви објекти
које сте филтрирали из примарне табеле, а затим се сви те ИД-ови прослеђују M2Mповезаној
табели, преузимајући само оне ИД-ове који постоје у вези са објектима из примарне табеле.
Укупно се извршавају два упита, а затим се може извршити спајање између Pythonдве листе
објеката.

class MyAdmin(BaseAdmin):

def get_queryset(self, request):
return super().get_queryset(request).prefetch_related(

'friends'
)

Аспекти (ново у Django 5.0)

Aspekti su nova funkcija u Django 5.0 za pregled broja filtera u administratorskom delu.
Podrazumevano je dodat prekidač koji, kada je omogućen, prikazuje koliko će rezultata biti
vraćeno za svaki filter. Ovo može biti sporo na velikim skupovima podataka i može se onemogućiti
podešavanjem svojstva show_facetsna Never. Ispod je primer.

@admin.register(Repository)
class RepositoryAdmin(SearchAdmin):

show_facets = admin.ShowFacets.ALLOW # Default, facets will be toggleable
# show_facets = admin.ShowFacets.ALWAYS # Enable facets, always show result 

count for a filter
# show_facets = admin.ShowFacets.NEVER # Disable facets

Rezime

Objave na blogu vode kroz višestruke pristupe optimizaciji kako bi se sortiranje, uređivanje,
pretraživanje, brojanje i druge operacije ubrzale. Možda to u početku nije potrebno, ali kako
projekti rastu, većini vizuelizacija korisničkog interfejsa je potrebna optimizacija u smislu načina



na koji preuzimamo i filtriramo objekte. Ovo ponekad ima svoje nedostatke, ali ne mislim da
previše utiče na funkcionalnost. Nadam se da su ovo lake pobede za performanse Django
administratora u vašem projektu i slobodno podelite bilo koja druga generička rešenja za
optimizaciju!