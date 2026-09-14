
# select_related

[Sadržaj](00_sadrzaj.md)

Za "jedan na jedan" veze i polja stranih ključeva, možete koristiti `select_related` za optimizaciju `QuerySet`-a.

Posle upotrebe `select_related()` funkcije na QuerySet-u, Django vraća objekat koji odgovara stranom ključu, zbog toga eliminiše potrebu za kasnijim upitima na db. Ako treba da štampamo sve gradove i njihove provincije, na direktan način:

```shell
>>> cities = City.objects.all()
>>> for c in cities:
... print c.province
```

Ovo vodi do linearnih SQL upita. Ako postoji n objekata sa k stranih ključeva za svaki objekat, to će rezultovati sa n*k+1 SQL upita. U ovom slučaju postoji tri grada, i biće četiri upita:

```sql
SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id = '1' ;

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id = '2' ;

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id = '1' ;
```

> [!Note]
>
> SQL izrazi su uzeti direktno sa Django logger-a.

Ako koristimo `select_related()` funkciju:

```shell
>>> cities = City.objects.select_related().all()
>>> for c in cities:
... print c.province
```

Sada je samo jedan SQL upit:

```sql
SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id,
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_city
INNER JOIN
  QSOptimize_province ON (QSOptimize_city.province_id = QSOptimize_province.id);
```

Django ovde koristi `INNER JOIN` da dobije informacije o provincijama. Rezultati upita su:

Id | city_name      |  province_id | id | province_name
---|----------------|--------------|----|------------------
 1 | Wuhan City     | 1            | 1  | Hubei Province
 2 | Guangzhou City | 2            | 2  | Guangdong Province
 3 | Shiyan City    | 1            | 1  | Hubei Province

Ovaj metod podržava sledeće argumente:

[Sadržaj](00_sadrzaj.md)

## *fields parametri

`select_related()` prihvata varijabilni broj parametara, svaki od njih je ime polja stranog ključa, (tj. sadržaj glavne tabele) koji treba pokupiti, kao i ime stranog ključa itd.

Da bi izabrali polje iz parent tabele treba koristiti dve podcrte "__".

Na primer, da bi dobili Zhang San-ovu provinciju:

```shell
>>> zhangs = Person.objects
      .select_related('living province')
      .get(firstname = u "Zhang", lastname = u "San")
>>> zhangs.living.province
```

Odgovarajući SQL upit je:

```sql
SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id,
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id,
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_person
INNER JOIN
  QSOptimize_city ON (QSOptimize_person.living_id = QSOptimize_city.id)
INNER JOIN
  QSOptimize_province ON(QSOptimize_city.province_id=
    QSOptimize_province.id)
WHERE (
  QSOptimize_person.lastname = San AND QSOptimize_person.first name =
    Zhang
);
```

Django koristi dva INNER JOIN-a za kompletiranje upita, dobija sadržaj tabela city i province i dodaje ih na odgovarajuće vrste rezultujuće tabele, tako da ne treba više upita.

Id | name      |Home |Living City_Id | City_Name | Province_id | Id | Province_name | Town  | Town_id
---|-----------|-----|---------------|-----------|-------------|----|---------------|-------|---------
 1 | Zhang San | 3   | 1             | 1         | Wuhan       | 1  | 1             | Hubei | .

Nespecificirani strani ključevi nisu dodati u rezultat. Treba dobiti Zhang San’s hometown, na sledeći način:

>>> zhangs.hometown.province

```sql
SELECT
  QSOptimize_city.id,
  QSOptimize_city.name,
  QSOptimize_city.province_id
FROM
  QSOptimize_city
WHERE
  QSOptimize_city.id = 3 ;

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id = 1
```

Pošto strani ključ nije specificiran, napravljena su dva upita. Ako je veća dubina, veći je i broj upita. Ako želite i hometown i province gde živi u pre v1.7:

```shell
zhangs = Person.objects.select_related('town province', 'living province').
    get(first name = u "Zhang", LastName = u "San")

>>> zhangs.hometown.province
>>> zhangs.living.province
```

Od ver. 1.7 možete ulančati operacije kao i druge queryset funkcije:

```shell
zangs = Person.objects.select_related('town province')
  .select_related('living province')
  .get(first name = u "Zhang", LastName = u "San")

>>> zhangs.hometown.province
>>> zhangs.living.province
```

Za verzije do "1.7" dobićete rezultate samo posledje operacije, u ovom slučaju nema rezultata za hometown.

Dva SQL upita su generisana kada štampate vašu provinciju.

[Sadržaj](00_sadrzaj.md)

## depth parameter

`select_related()` prihvata depth parametar, koji definiše dubinu `select_related`. Django rekurzivno prolazi sve `OneToOneField` i `ForeignKey`.

Za ilustraciju:

```shell
>>> zhangs = Person.objects.select_related(depth = d)
```

d = 1 je ekvivalentno sa:

```py
select_related("hometown", "living")
```

d = 2 je ekvivalentno sa:

```py
select_related('citizen province', 'living province')
```

[Sadržaj](00_sadrzaj.md)

## *parameters

`select_related()` može biti parametrizovan, što znači da će Django ići sa `select_related` najdublje što može.

Na primer:

```py
zangs = Person.objects.select_related().get(first name=u"Zhang", LastName=u"San")
```

Dve stvari treba primetiti:

- Django može u kompleksnim situacijama probiti granice rekurzije.
- Django ne zna koja polja koristitite, zbog toga uzima sve, što je gubitak performansi.

[Sadržaj](00_sadrzaj.md)

## Zaključak select_related

- `select_related` optimizuje `jedan na jedan` i `više na jedan` veze.
- `select_related` koristi `JOIN` naredbe da optimizuje i poboljša performanse smanjenjem broja upita.
- Možete specifikovati ime polja. Specifikovani rekurzivni upit može biti implementiran sa duplim podcrtama povezanim na ime polja. Ovde nema keširanja.
- Može se specifikovati rekurzivna dubina, i Django automatski kešira sva polja unutar specifikovanog puta. Ako želite da pristupite poljima van specificiranog puta, Django će raditi SQL upit ponovo.
- Podržani su pozvi bez parametara, i Django će uraditi rekurzivne upite što je dublje moguće. Ali postoji granica u rekurzijama i gubitak na performansama.
- "Django >= 1.7", `select_related` ulančava pozive što je ekvivalentno korišćenju varijabilne dužine parametara, "Django < 1.7." ulančavanjem poziva ruši prednje `select_related` pozive i ostavlja samo poslednje.

[Sadržaj](00_sadrzaj.md)
