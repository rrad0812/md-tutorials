
# prefetch_related vs selected_related

[Sadržaj](00_sadrzaj.md)

Ako želimo da dobijemo ko je sve rodjen u Hubei province, najlakše je da dobijemo:

- Hubei province prvo,
- potom sve gradove u Hubei,
- i na kraju njihove žitelje:

```shell
>>> hb = Province.objects.get(name iexact = u"Hubei Province")
>>> people = []
>>> for city in hb.city_set.all():
... people.extend(city.birth.all())
```

Ovo će dovesti do 1 + (broj gradova u Hubei) SQL upita.

Pokušajmo sa prefetch_related():

```shell
>>> hb = Province.objects.prefetch_related("city_set birth")
  .objects.get(name iexact = u"Hubei Province")
>>> people = []
>>> for city in hb.city_set.all():
... people.extend(city.birth.all())
```

Ovo je prefetch dubine 2, biće tri SQL upita:

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

SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname, QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id
FROM
  QSOptimize_person
WHERE
  QSOptimize_person.hometown_id IN (1, 3);
```

Možda je reverzni upit jednostavniji?

```py
People = list(Person.objects.select_related("town__province")
.filter(town province name iexact = u"hubei__province")
```

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
  QSOptimize_city ON (QSOptimize_person.hometown_id =
  QSOptimize_city.id)
INNER JOIN
  QSOptimize_province ON (QSOptimize_city.province_id = QSOptimize_province.id)
WHERE
  QSOptimize_province.name LIKE 'Hubei Province';
```

id | name      |hometown_id | living_id |id | name   | province_id | id | name
---|-----------|------------|-----------|---|--------|-------------|----|------
1  | Zhang San | 3          | 3         | 1 | Shiyan | 1           | 1  | Hubei
2  | Li        | 4          | 1         | 3 | Wuhan  | 1           | 1  | Hubei
3  | Wang Mazi | 3          | 2         | 3 | Shiyan | 1           | 1  | Hubei

`select_related()` je efikasniji od `prefetch_related()`. Zbog toga najbolje je koristi ga gde je moguće, kao što je `više na jedan` veza.

[Sadržaj](00_sadrzaj.md)

## Zaključak prefetch_related vs selected_related

- Pošto `select_related()` uvek rešava problem u jednom SQL upitu i `prefetch_related()` pravi upite na svakoj povezanoj tabeli, efikasnost `select_related()` je obično veća.
- Razmišljajte o `prefetch_related()` samo ako `select_related()` ne može da reši problem.
- Možete koristiti i `select_related()` i `prefetch_related()` u jednom QuerySet-u da smanjite broj SQL upita.
- `select_related()` pre `prefetch_related()` je validno. Ako ide posle biće ignorisano.

[Sadržaj](00_sadrzaj.md)
