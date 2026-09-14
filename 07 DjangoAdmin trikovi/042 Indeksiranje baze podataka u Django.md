English

Poslednje ažuriranje 15. januara 2025.

Indeksiranje baze podataka u Django-u
Originalni tekst

Database Indexing in Django
Oluvole Madžijagbe

Ocenite ovaj prevod
Povratne informacije će se koristiti za poboljšanje Google prevodioca

Istaknuti kurs
 Tviter  Redit  Hakerske vesti  Fejsbuk

Razvoj vođen testiranjem
pomoću Django-a, Django

Indeksiranje baze podataka je moćna tehnika koja pomaže u ubrzavanju
REST Framework-a i

pretraživanja podataka i optimizaciji performansi vaših upita u relacionim bazama
Docker-a

podataka. Kreiranjem indeksa omogućavate bazi podataka da mnogo brže pronađe
informacije, što može značajno poboljšati efikasnost vaše aplikacije, posebno kada U ovom kursu ćete naučiti kako da

se radi sa velikim skupovima podataka. podesite razvojno okruženje sa Dokerom
kako biste izgradili i implementirali RESTful

Prilikom razvoja aplikacije pomoću Django-a, razumevanje i implementacija API koji pokreću Pajton, Django i Django

pravilnog indeksiranja na vašoj bazi podataka može značajno poboljšati brzinu REST Framework.

odziva vaših veb aplikacija. Bez obzira da li pravite mali projekat ili složen sistem,
znanje kako efikasno koristiti indeksiranje je ključno za pružanje bržeg i Kupi odmah 30 dolara
besprekornijeg korisničkog iskustva.

Pogledajte kurs
Ovaj članak istražuje osnove indeksiranja baza podataka, njegove prednosti i
mane, i kako ga primeniti u Django aplikaciji.

Претражи све туторијале

Šta je indeksiranje baze podataka?
TEME TUTORIJALA

Indeksiranje baze podataka je tehnika koja se koristi za optimizaciju performansi
api arhitektura avs devops

baze podataka smanjenjem vremena potrebnog za preuzimanje podataka. To
Django django rest framework

podrazumeva korišćenje indeksa baze podataka koji deluje kao putokaz za bazu
podataka da efikasno pronađe redove bez skeniranja cele tabele. doker brzi api termos

prednji deo heroku kubernetes

mašinsko učenje pajton reaguj

Šta je indeks baze podataka? red zadataka testiranje vju

Indeks baze podataka je struktura podataka koja se koristi za poboljšanje brzine veb skrejping

operacija pretraživanja podataka na tabeli baze podataka, po cenu dodatnog
prostora za skladištenje i potencijalnog uticaja na performanse operacija pisanja

SADRŽAJ
(kao što su INSERT , UPDATE i DELETE ).

Šta je indeksiranje baze podataka?

Kako funkcioniše indeksiranje? Indeksiranje u Django-u

Primer projekta
Indeks funkcioniše baš kao indeks u knjizi. Da biste pronašli informacije, umesto da
prelistavate svaku stranicu, možete koristiti indeks da biste brzo pronašli informacije Zaključak

koje su vam potrebne. Slično tome, u bazi podataka, indeks čuva pokazivače na
redove u tabeli. Kada se izvrši upit, mehanizam baze podataka prvo proverava
indeks kako bi brzo pronašao podatke.

Vrste indeksa

1. Primarni indeks: Ovaj indeks se automatski kreira za kolone primarnog ključa
u bazi podataka. Povratne informacije



2. Jedinstveni indeks: Kada je kolona definisana atributom unique , kreira se
jedinstveni indeks koji osigurava da su sve vrednosti u koloni različite.

3. Kompozitni indeks: Ova vrsta indeksa uključuje više kolona.

4. Indeks punog teksta: Optimizovano za pretragu velikih tekstualnih podataka.

5. Grupisani indeks: Određuje fizički redosled podataka u tabeli. Tabela može
imati samo jedan grupisani indeks.

6. Neklasterovani indeks: Sadrži pokazivač na stvarne podatke, što
omogućava više neklasterovanih indeksa po tabeli.

Prednosti i mane
Indeksiranje baza podataka je moćno, ali dolazi sa prednostima i kompromisima.
Razumevanje ovih prednosti može vam pomoći da odlučite kada i kako efikasno
koristiti indeksiranje.

Prednosti

1. Brže preuzimanje podataka : Indeksi značajno smanjuju vreme potrebno za
pronalaženje određenih redova u tabeli, posebno za velike skupove podataka.
Ovo je posebno korisno za upite koji uključuju klauzule WHERE , JOIN i . ORDER
BY

2. Poboljšane performanse upita : Upiti koji često filtriraju ili sortiraju podatke
imaju koristi od indeksa, jer omogućavaju bazi podataka da preskoči
skeniranje cele tabele.

3. Podržava jedinstvena ograničenja : Indeksi takođe pomažu u sprovođenju
jedinstvenosti u kolonama, osiguravajući da se ne ubacuju duplirane
vrednosti, kao što se vidi kod primarnih ključeva i UNIQUE ograničenja.

4. Optimizovane pretrage celog teksta : Specijalizovani indeksi, poput indeksa
celog teksta, čine pretragu velikih tekstualnih polja mnogo bržom, posebno u
bazama podataka kao što su MySQL i Postgres.

5. Bolja skalabilnost : Kako podaci u bazi podataka rastu, pravilno indeksiranje
pomaže u održavanju performansi, osiguravajući da upiti ostanu efikasni čak i
sa milionima redova.

Nedostaci

1. Povećani zahtevi za skladištenje : Indeksi troše dodatni prostor na disku jer
se indeks kreira pored tabela i kolona baze podataka. Za velike tabele sa više
indeksa, ovo može dovesti do značajnog opterećenja skladištenjem.

2. Sporije operacije pisanja : Umetanje, ažuriranje i brisanje postaju sporije jer
baza podataka mora da ažurira i indekse pored samih podataka.

3. Troškovi održavanja : Indekse je potrebno ažurirati kako se podaci u tabeli
menjaju, što dodaje dodatne troškove.

4. Nije uvek efikasno : Ako upit ne koristi indeksiranu kolonu (kolone),
indeksiranje neće pomoći. Na primer, korišćenje funkcija ili proračuna na
indeksiranim poljima može zaobići indeksiranje. Optimizator upita baze
podataka takođe može odlučiti da je skeniranje celog tela efikasnije od
korišćenja indeksiranja, što poništava razlog za indeksiranje baze podataka.

5. Rizik od prekomernog indeksiranja : Dodavanje previše indeksa može
smanjiti ukupne performanse, jer baza podataka troši više vremena na
održavanje indeksa nego na izvršavanje upita.

Kada koristiti indeksiranje
Povratne informacije



1. Indeksira često upitana polja ili kolone koje se koriste u WHERE , JOIN ili ORDER
BY .

2. Izbegavajte indeksiranje polja koja se retko koriste u upitima.

3. Pronađite ravnotežu između performansi čitanja i pisanja na osnovu potreba
vaše aplikacije.

Indeksiranje u Django-u
Django je popularni Python veb frejmvork koji pruža apstrakciju visokog nivoa za
interakcije sa bazama podataka, uključujući indeksiranje. Pomoću Django ORM-a
možete definisati indekse u svojim modelima kako biste optimizovali performanse
upita.

Kreiranje indeksa u Django-u
Django može da kreira nekoliko vrsta indeksa baze podataka, automatski ili putem
eksplicitne konfiguracije u vašim modelima. U Django-u možete definisati indekse
koristeći db_index za jednu kolonu indeksa, kao i indexes opciju u Meta klasi
modela.

Tipovi indeksa u Django-u

Tip indeksa Automatski Primer upotrebe

kreirano?

Indeks Da Podrazumevani primarni ključ
primarnog
ključa

Indeks stranih Da Odnosi sa spoljnim ključem
ključeva

Jedinstveni Da unique=True na poljima
indeks

Indeks sa Ne (eksplicitno) db_index ili Meta.indexes
jednom
kolonom

Kompozitni Ne (eksplicitno) Meta.indexes sa više polja
indeks

Jedinstveni Da ( unique_together ili UniqueConstraint
kompozitni unique_together )
indeks

Indeks punog Ne (eksplicitno, GinIndex

teksta samo Postgres)

Delimični Ne (eksplicitno, Index sa uslovom
indeks samo Postgres)

Korišćenjem ovih automatskih i prilagođenih opcija indeksiranja, Django pruža
robusnu podršku za optimizaciju upita baze podataka.

Povratne informacije



Primer projekta
Da bismo bolje razumeli koncept indeksiranja baze podataka u Django-u, kreirali
smo primer Django aplikacije, sistema za upravljanje proizvodima. Aplikacija se
koristi za upravljanje proizvodima u prodavnici, omogućavajući korisnicima da
efikasno filtriraju, sortiraju i pretražuju.

Sistem upravljanja proizvodima uključuje sledeće funkcije:

1. Dodajte i upravljajte proizvodima sa detaljima kao što su naziv, kategorija,
cena i datum kreiranja.

2. Filtrirajte proizvode po kategoriji da biste pronašli stavke unutar određenih
grupa.

3. Sortirajte proizvode po ceni kako biste pomogli korisnicima da pronađu
najjeftinije ili najskuplje artikle.

4. Koristite indekse da biste poboljšali performanse upita za operacije filtriranja i
sortiranja.

Da biste pratili, klonirajte repozitorijum i pokrenite projekat:

$ git clone https://github.com/testdrivenio/django-db-indexes.git

$ cd django-db-indexes

$ docker compose up -d --build

Da bismo pojednostavili stvari, naš projekat koristi Doker za pokretanje
Django aplikacije, kao i Postgres-a. Zanima vas kako je ovaj projekat
razvijen? Pogledajte članak o Dokerizaciji Django-a pomoću Postgres-a,
Gunicorn-a i Nginx-a .

Django modeli
Obratite pažnju na modele koje smo definisali u product/models.py :

1. ProductWithoutIndex : Ovaj model nema definisane indekse.

2. ProductWithSingleIndex : Ovaj model ima tri indeksa sa jednom kolonom
preko db_index=True , name i category kolona price .

3. ProductWithCompositeIndex : Ovaj model ima kompozitni indeks na
kolonama category i price .

Da biste videli indekse, uđite u psql:

$ docker compose exec api python manage.py dbshell

Model proizvoda bez indeksa

# \d product_without_index

Možete videti podrazumevani indeks primarnog ključa:

Povratne informacije



                              Table "public.product_without_index"

   Column   |           Type           | Collation | Nullable |

Default

------------+--------------------------+-----------+----------

+----------------------------------

 id         | bigint                   | | not null | generated 

by default as identity

 name       | character varying(100) | | not null |

 category   | character varying(50) | | not null |

 price      | integer                  | | not null |

 created_at | timestamp with time zone | | not null |

Indexes:

"product_without_index_pkey" PRIMARY KEY, btree (id)

Model proizvoda sa jednim indeksom

# \d product_with_single_index

Rezultati:

                            Table "public.product_with_single_index"

   Column   |           Type           | Collation | Nullable |

Default

------------+--------------------------+-----------+----------

+----------------------------------

 id         | bigint                   | | not null | generated 

by default as identity

 name       | character varying(100) | | not null |

 category   | character varying(50) | | not null |

 price      | integer                  | | not null |

 created_at | timestamp with time zone | | not null |

Indexes:

"product_with_single_index_pkey" PRIMARY KEY, btree (id)

"product_with_single_index_category_715a00f3" btree (category)

"product_with_single_index_category_715a00f3_like" btree (category 

varchar_pattern_ops)

"product_with_single_index_name_d6fb2180" btree (name)

"product_with_single_index_name_d6fb2180_like" btree (name 

varchar_pattern_ops)

"product_with_single_index_price_5a707788" btree (price)

Pojedinačni indeksi se kreiraju za svaku od kolona gde db_index je podešeno na
True . Indeksi se kreiraju za svaku od njih i za direktne filtere i LIKE za filtere.

Model ProductVithCompositeIndek

# \d product_with_composite_index

Rezultati:

                           Table "public.product_with_composite_index"

   Column   |           Type           | Collation | Nullable |

Default

------------+--------------------------+-----------+----------

+----------------------------------

 id         | bigint                   | | not null | generated 

by default as identity

 name       | character varying(100) | | not null |

 category   | character varying(50) | | not null |

 price      | integer                  | | not null |

 created_at | timestamp with time zone | | not null |

Indexes:

"product_with_composite_index_pkey" PRIMARY KEY, btree (id)

"category_price_idx" btree (category, price)

"product_wit_name_5adbb5_idx" btree (name)

Povratne informacije



Kompozitni indeks se kreira i za i za category . price Ovaj indeks ubrzava filtere
koji kombinuju i kategoriju i cenu nego podešavanje indeksa za svaku kolonu
pojedinačno.

Fiktivni podaci
Moć indeksiranja mnogo bolje dolazi do izražaja kada imate veliki skup podataka.
Da bismo vam pomogli u tome, kreirali smo seed_db komandu za upravljanje. Ova
komanda kreira 500 hiljada zapisa za svaku kategoriju u svakoj tabeli.

Pokrenite komandu za popunjavanje baze podataka:

$ docker compose exec api python manage.py seed_db

Ovo će potrajati, zato slobodno popijte šolju kafe.

Testiranje
Da bismo videli kako tabele i indeksi funkcionišu, imamo još jednu komandu za
upravljanje pod nazivom run_tests . Bacite pogled na skriptu u product/
management/commands/run_tests.py .

Komanda prihvata nekoliko argumenata

1. table_type (obavezno): Ovo je ceo broj sa 1 representing
ProductWithoutIndex , 2 representing ProductWithSingleIndex i
3 representing ProductWithCompositeIndex

2. category Mora biti "electronics" , "clothing" , ili "home appliances"

3. price Ceo broj koji se koristi za filtriranje stavki

Osnovni upit
Počnimo sa jednostavnim upitom za dobijanje svih objekata.

Model proizvoda bez indeksa

$ docker compose exec api python manage.py run_tests --table_type=1 --

category="electronics"

Rezultati:

Povratne informacije



[{'Execution Time': 41.694,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 39.795,

'Actual Total Time': 41.682,

'Async Capable': False,

'Node Type': 'Gather',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Plans': [{'Actual Loops': 3,

'Actual Rows': 0,

'Actual Startup Time': 37.769,

'Actual Total Time': 37.77,

'Alias': 'product_without_index',

'Async Capable': False,

'Filter': '((product_without_index.category)::text = 

'

"'electronics'::text)",

'Node Type': 'Seq Scan',

'Output': ['id',

'name',

'category',

'price',

'created_at'],

'Parallel Aware': True,

'Parent Relationship': 'Outer',

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_without_index',

'Rows Removed by Filter': 500000,

'Schema': 'public',

'Startup Cost': 0.0,

'Total Cost': 20241.5,

'Workers': [{'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 36.902,

'Actual Total Time': 36.903,

'Worker Number': 0},

{'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 36.892,

'Actual Total Time': 36.893,

'Worker Number': 1}]}],

'Single Copy': False,

'Startup Cost': 1000.0,

'Total Cost': 21241.6,

'Workers Launched': 2,

'Workers Planned': 2},

'Planning Time': 0.358,

'Triggers': []}]

Možemo videti da je skeniranje celog tela izvršeno od Node Type: Seq Scan . To je
trajalo oko 42 ms.

Model proizvoda sa jednim indeksom

$ docker compose exec api python manage.py run_tests --table_type=2 --

category="electronics"

Rezultati:

Povratne informacije



[{'Execution Time': 0.033,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 0.019,

'Actual Total Time': 0.02,

'Alias': 'product_with_single_index',

'Async Capable': False,

'Index Cond': '((product_with_single_index.category)::text = '

"'electronics'::text)",

'Index Name': 'product_with_single_index_category_715a00f3',

'Node Type': 'Index Scan',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_with_single_index',

'Rows Removed by Index Recheck': 0,

'Scan Direction': 'Forward',

'Schema': 'public',

'Startup Cost': 0.43,

'Total Cost': 4.45},

'Planning Time': 0.633,

'Triggers': []}]

U ovom upitu, optimizator baze podataka koristi indeks kao tip skeniranja i koristi
ispravan indeks. Za izvršavanje ovog upita potrebno je oko 0,033 ms, što je daleko
bolja opcija nego na tabeli bez indeksa.

Model ProductVithCompositeIndek

$ docker compose exec api python manage.py run_tests --table_type=3 --

category="electronics"

Rezultati:

[{'Execution Time': 0.032,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 0.02,

'Actual Total Time': 0.02,

'Alias': 'product_with_composite_index',

'Async Capable': False,

'Index Cond': '((product_with_composite_index.category)::text = 

'

"'electronics'::text)",

'Index Name': 'category_price_idx',

'Node Type': 'Index Scan',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_with_composite_index',

'Rows Removed by Index Recheck': 0,

'Scan Direction': 'Forward',

'Schema': 'public',

'Startup Cost': 0.43,

'Total Cost': 6.2},

'Planning Time': 0.293,

'Triggers': []}]

Ovo je brže jer baza podataka koristi indeks category_price_idx , koji smo
definisali na tabeli.

Filtrirani upit
Hajde da isprobamo upit koji kombinuje i kategoriju i cenu. Očekuje se da će
kompozitni indeks biti brži od kolone sa jednim indeksom, dok će ova zauzvrat biti
brža u poređenju sa tabelom bez indeksa.

Povratne informacije



Model proizvoda bez indeksa

$ docker compose exec api python manage.py run_tests --table_type=1 --

category="electronics" --price=1000

Rezultati:

[{'Execution Time': 44.069,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 42.227,

'Actual Total Time': 44.056,

'Async Capable': False,

'Node Type': 'Gather',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Plans': [{'Actual Loops': 3,

'Actual Rows': 0,

'Actual Startup Time': 40.342,

'Actual Total Time': 40.342,

'Alias': 'product_without_index',

'Async Capable': False,

'Filter': '((product_without_index.price <= 1000) AND 

'

'((product_without_index.category)::text = 

'

"'electronics'::text))",

'Node Type': 'Seq Scan',

'Output': ['id',

'name',

'category',

'price',

'created_at'],

'Parallel Aware': True,

'Parent Relationship': 'Outer',

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_without_index',

'Rows Removed by Filter': 500000,

'Schema': 'public',

'Startup Cost': 0.0,

'Total Cost': 21804.0,

'Workers': [{'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 39.573,

'Actual Total Time': 39.574,

'Worker Number': 0},

{'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 39.567,

'Actual Total Time': 39.567,

'Worker Number': 1}]}],

'Single Copy': False,

'Startup Cost': 1000.0,

'Total Cost': 22804.1,

'Workers Launched': 2,

'Workers Planned': 2},

'Planning Time': 0.26,

'Triggers': []}]

Ponovo je izvršeno skeniranje celog tela. Trajalo je približno 44 ms.

Model proizvoda sa jednim indeksom

$ docker compose exec api python manage.py run_tests --table_type=2 --

category="electronics" --price=1000

Rezultati: Povratne informacije



[{'Execution Time': 0.041,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 0.011,

'Actual Total Time': 0.011,

'Alias': 'product_with_single_index',

'Async Capable': False,

'Index Cond': '((product_with_single_index.category)::text = '

"'electronics'::text)",

'Index Name': 'product_with_single_index_category_715a00f3',

'Node Type': 'Index Scan',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_with_single_index',

'Rows Removed by Index Recheck': 0,

'Scan Direction': 'Forward',

'Schema': 'public',

'Startup Cost': 0.43,

'Total Cost': 4.45},

'Planning Time': 0.353,

'Triggers': []}]

Model ProductVithCompositeIndek

$ docker compose exec api python manage.py run_tests --table_type=3 --

category="electronics" --price=1000

Rezultati:

[{'Execution Time': 0.036,

'Plan': {'Actual Loops': 1,

'Actual Rows': 0,

'Actual Startup Time': 0.021,

'Actual Total Time': 0.022,

'Alias': 'product_with_composite_index',

'Async Capable': False,

'Index Cond': '(((product_with_composite_index.category)::text = 

'

"'electronics'::text) AND "

'(product_with_composite_index.price <= 1000))',

'Index Name': 'category_price_idx',

'Node Type': 'Index Scan',

'Output': ['id', 'name', 'category', 'price', 'created_at'],

'Parallel Aware': False,

'Plan Rows': 1,

'Plan Width': 35,

'Relation Name': 'product_with_composite_index',

'Rows Removed by Index Recheck': 0,

'Scan Direction': 'Forward',

'Schema': 'public',

'Startup Cost': 0.43,

'Total Cost': 6.2},

'Planning Time': 0.365,

'Triggers': []}]

Ove razlike su obično izraženije na tabelama sa većim skupovima podataka. Da
biste ovo dodatno testirali, možete povećati veličinu grupe komande seed_db da
biste videli koliko indeksiranje ubrzava filtriranje i sortiranje.

Zaključak
Indeksiranje baze podataka je važna tehnika za optimizaciju performansi upita i
osiguravanje da vaša aplikacija ostane efikasna kako vaši podaci rastu. Pažljivom
implementacijom indeksa u vašim Django modelima možete značajno poboljšati

Povratne informacije



brzinu i brzinu odziva vaših upita, posebno za često filtrirana ili sortirana polja.

U ovom članku smo istražili osnove indeksiranja baza podataka, njegove prednosti i
mane, i kako ga primeniti u Django aplikaciji. Kroz primer projekta sistema za
upravljanje proizvodima, videli smo kako efikasno koristiti indeksiranje, posebno sa
db_index atributom i Meta klasom. Takođe smo ga praktično testirali upoređujući

vremenske razlike pri filtriranju na osnovu indeksiranih polja.

Kombinovanjem praktične implementacije sa najboljim praksama, možete iskoristiti
indeksiranje kako biste pružili brže i glađe korisničko iskustvo. Ne zaboravite da
analizirate obrasce upita vaše aplikacije i uravnotežite kompromise indeksiranja za
optimalne rezultate. Sa ovim alatima i tehnikama, na dobrom ste putu ka izgradnji
skalabilnih i visoko efikasnih Django aplikacija!

Srećno indeksiranje.

 Django

Oluvole Madžijagbe
Oluvole je softverski inženjer sa strašću za nove
tehnologije i željom za učenjem. Kada ne programira,
možete ga naći kako igra video igre ili čita uglavnom
romane.

  

PODELI OVAJ TUTORIJAL

 Tviter  Redit  Hakerske vesti  Fejsbuk

PREPORUČENI TUTORIJALI 

Budite u toku sa ažuriranjima kursa
Pridružite se našoj mejling listi da biste bili obavešteni o

ažuriranjima i novim izdanjima.

Унесите своју имејл адресу Pretplatite se

Povratne informacije



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

Povratne informacije