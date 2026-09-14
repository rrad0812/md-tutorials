# prefetch_related

[Sadržaj](00_sadrzaj.md)

Za `više na više` i `jedan na više` vezu, `prefetch_related()` može biti korišćen za optimizaciju. Ali ovde nema `jedan na više` veza, reći ćete vi. U stvari, `ForeignKey` je `više na jedan` veza, a sa druge strane gledano to je `jedan na više` veza.

`prefetch_related()` i `select_related()` su dizajnirani da smanje broj SQL upita, ali su implementirane na različit način. `select_related()` rešava problem sa JOIN naredbama.

Ali, za `više na više` veze, ako bi koristili `JOIN` vezu, dobijene tabele bile bi ogromne, `n x m` veličine. Rešenje je da `prefetch_related()` odvojeno pravi upite i da zatim Python procesira njihove veze. Da bi dobili gradove koje je Zhang San posetio, `prefetch_related()` bi uradio sledeće:

```py
zhangs = Person.objects.prefetch_related('visitation')
.get(first name=u"Zhang", LastName=u"three")

>>> for city in zhangs.visitation.all():
... print city
...
```

SQL upiti su kao što sledi:

```sql
SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id
FROM
  QSOptimize_person
WHERE
  QSOptimize_person.lastname='San' AND
  QSOptimize_person.firstname='Zhang');

SELECT
  QSOptimize_person_visitation.person_id AS _prefetch_related_val,
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON ( 
    QSOptimize_city.id = QSOptimize_person_visitation.city_id
)
WHERE
  QSOptimize_person_visitation.person_id IN (1);
```

Prvi SQL upit daje samo Zhang San Person objekat. Drugi upit je kritičan.

Id | name       | Hometown_id | Living id
---|------------|-------------|----------
 1 | Zhang San  | 3           | 1

Prefetch_related | Id | Name           | Province_id
-----------------|----|----------------|------------
 1               | 1  | Wuhan City     | 1
 1               | 2  | Guangzhou City | 2
 1               | 3  | Shiyan City    | 1

Ili ako želimo da dobijemo sve gradove provincije Hubei:

```shell
>>> hb = Province.objects.prefetch_related('city_set')
.get(name iexact = u "Hubei Province")

>>> for city in hb.city_set.all():
... city.name
```

Ovo će dovesti do sledećih SQL upita:

```sql
SELECT
QSOptimize_province.id,
QSOptimize_province.name
FROM
QSOptimize_province
WHERE
QSOptimize_province.name LIKE 'Hubei Province';

SELECT
QSOptimize_city.id,
QSOptimize_city.name,
QSOptimize_city.province_id
FROM
QSOptimize_city
WHERE
QSOptimize_city.province_id IN (1);
```

A rezultujuće tabele :

Id | Name
---|-----
1  | Hubei

Id | Name       | Province_id
---|------------|-----------
1  |Wuhan City  | 1
3  |Shiyan City | 1

Prefech je implementiran korišćenjem `IN` naredbe. Kada postoji više objekata u `QuerySet`-u, problem performansi, u zavisnosti od vrste db se može pojaviti.

[Sadržaj] (#sadržaj)

## *lookups parameter

`Prefetch_related()` je korišćen u "Django < 1.7". Kao `select_related()`, `prefetch_related()` takodje podržava duboke upite, kao napr. provincije koje su posetili svi Zhang ljudi:

```py
zhangs = Person.objects.prefetch_related('visitation province')
.filter(first_name__iexact=u'Zhang')

>>> for i in zhangs:
for city in i.visitation.all():

print city.province
```

Generisani SQL upiti su:

```sql
SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id 
FROM
  QSOptimize_person
WHERE
  QSOptimize_person.first name LIKE 'Zhang';

SELECT
  QSOptimize_person_visitation.person_id AS _prefetch_related_val,
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id 
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id = QSOptimize_person_visitation.city_id)
WHERE
  QSOptimize_person_visitation.person_id IN (1, 4);

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id IN (1, 2);
```

A rezultujuće tabele su:

id | name      | hometown_id | living_id
---|-----------|-------------|----------
1  | Zhang San | 3           | 1
4  | Zhang San | 2           | 2

_prefetch_related_val | Id | Name           | province_id
----------------------|----|----------------|------------------
 1                    | 1  | Wuhan City     | 1
 1                    | 2  | Guangzhou City | 2
 4                    | 2  | Guangzhou City | 2
 1                    | 3  | Skiyan City    | 1

Id | Name
---|------------------
 1 | Hubei Province
 2 | Guangdong Province

Ulančavanje `prefetch_related` dodaje ove upite.

Primetite da kada koristimo `QuerySet`, pošto je db zahtev promenjen u ulančanoj operaciji, keširanje će biti ignorisano. To dovodi do db re-request za dobijanje podataka, što može izazvati probleme sa performansama.

Promena zahteva prema db može značiti potrebu za različitim `filters()`, `exclude()`, itd, tako da to veoma menja SQL kod. `all()` ne menja krajnji db zahtev, tako da on ne vodi ka ponovnom db zahtevu.

Na primer, da bi dobili gradove sa rečju "city" koje je svako posetio, to će voditi do brojnih SQL upita:

```py
plist = Person.objects.prefetch_related('visitation')
[p.visitation.filter(name__icontains=u "city") for p in plist]
```

Pošto su četiri osobe u db, rezultat je 2 + 4 SQL upita:

```sql
SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id
 FROM
  QSOptimize_person;

SELECT
  QSOptimize_person_visitation.person_id AS _prefetch_related_val,
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id =
  QSOptimize_person_visitation.city_id)
WHERE
  QSOptimize_person_visitation.person_id IN (1, 2, 3, 4);

SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id =
  QSOptimize_person_visitation.city_id)
WHERE
  QSOptimize_person_visitation.person_id= '1' AND
  QSOptimize_city.name LIKE '% city%';

SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id =
  QSOptimize_person_visitation.city_id)
WHERE
  QSOptimize_person_visitation.person_id= '2' AND
  QSOptimize_city. name LIKE '% market%';

SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id = QSOptimize_person_visitation.  
    city_id)
WHERE
  QSOptimize_person_visitation. person_id = '3' AND 
    QSOptimize_city. name LIKE '% city%';

SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_person_visitation ON (QSOptimize_city.id =
    QSOptimize_person_visitation.city_id)
WHERE
  QSOptimize_person_visitation. person_id = '4' AND
    QSOptimize_city. name LIKE '% city%';
```

`QuerySet` izračunavanje je lenjo, i rezultati nisu dostupni dok se ne koriste. Kada se pokrene druga linija Python koda, `for` petlja tretira "plist" kao iterator, koji okida db upite. Prva dva `SQL` upita izazvana su sa `prefetch_related`.

Iako su sve zahtevane `city` informacije bile uključene u rezultate upita, jer je `Person.visitation` fitriran u petlji, ovo obično menja db zahtev. Zbog toga, ove operacije će zanemariti keširane podatke i ponovo uraditi `SQL` upit.

Ali šta ako je takav zahtev? U "Django >= 1.7", možete to uraditi kroz `Prefetch` objekat. Ako je "Django < 1.7", možete uraditi ovako:

```py
plist = Person.objects.prefetch_related('visitation')
[[city for city in p.visitation.all() if u"city" in city.name] for p in plist]
```

[Sadržaj](00_sadrzaj.md)

## Prefetch objekat

U "Django >= 1.7", možete koristiti Prefetch objekat da kontrolišete ponašanje `prefetch_related` funkcije.

Karakteristike Prefetch objekta:

- `Prefetch` objekat može specifikovati samo jednu prefetch operaciju.
- `Prefetch` objekti specifikuju polja na isti način kao parametri u `prefetch_related`, lookup polja se prave duplom podcrtom.
- Možete manuelno specificirati QuerySet korišćen od `prefetch` kroz `queryset` parameter.
- Ime `prefetch` atributa može biti specficirano sa `to_attr` parametrom.
- `Prefetch` objekti i `lookup` parametri specificirani u formi stringa mogu se miksovati.

Nastavimo sa primerima da dobijemo gradove sa "Wu" i "Zhou" u svim gradovima koje je svako posetio:

```py
Wus = City.objects.filter(name icontains = u"wu")
Zhous = City.objects.filter(name icontains = u"state")

plist = Person.objects.prefetch_related(
  Prefetch('visitation', queryset = wus, to_attr = "wu_city"),
  Prefetch('visitation', queryset = zhous, to_attr = "zhou_city"),
)

[p.wu_city for p in plist]
[p.zhou_city for p in plist]
```

[Sadržaj](00_sadrzaj.md)

## None

Možete isprazniti `prefetch_related` prosledjenjem `None`:

```py
>>> prefetch_cleared_qset = qset.prefetch_related(None)
```

[Sadržaj](00_sadrzaj.md)

## Zaključak prefetch_related

- `Prefetch_related optimizuje` `jedan na više` i `više na više` veze.
- `Prefetch_related` optimizuje veze izmedju tabela vraćanjem njihovog sadržaja odvojeno i korišćenjem Python-a za procesiranje njihovih veza.
- Možete specificirati ime polja koje zahteva `prefetch_related` korišćenjem varijabilne dužine parametara, na isti način kao kod `select_related`.
- U "Django >= 1.7", kompleksni upiti mogu biti implementirani kroz `Prefetch` objekte, ali izgleda da u nižim verzijama Django-a to isto može biti implementirano.
- `Prefetch` objekti i stringovi mogu se mešati kao parametri `prefetch_related`.
- Ulanačni pozivi `prefetch_related` dodaju `prefetch` umesto da ga menjaju.
- `Prefetch_related` može biti očišćen prosledjenjem `None`.

[Sadržaj](00_sadrzaj.md)
