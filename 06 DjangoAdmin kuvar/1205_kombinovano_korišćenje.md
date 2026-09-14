
# Kombinovano korišćenje

[Sadržaj](00_sadrzaj.md)

Za isti QuerySet, možete koristiti obe funkcije u isto vreme. Dodajmo model Order:

```py
class Order(models.Model):
  customer = models.ForeignKey(Person)
  orderinfo = models.CharField(max_length=50)
  time = models.DateTimeField(auto_now_add = True)

def unicode (self):
  return self.orderinfo
```

Ako imamo "order_id", treba da saznamo provinciju gde je customer ordera bio. Ako pokušamo sa "prefetch_related()":

```shell
>>> plist = Order.objects
  .prefetch_related('customer visitation province')
  .get(id=1)
>>> for city in plist.customer.visitation.all():
... print city.province.name
```

Pošto su četiri tabele uključene: "Order", "Person", "City", "Province", `prefetch_related()` će generisati četiri SQL upita:

```sql
SELECT
  QSOptimize_order.id,
  QSOptimize_order.customer_id,
  QSOptimize_order.orderinfo,
  QSOptimize_order.time
FROM
  QSOptimize_order
WHERE
  QSOptimize_order.id = 1;

SELECT
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id, QSOptimize_person.living_id
FROM
  QSOptimize_person
WHERE
  QSOptimize_person.id IN (1);

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
WHERE QSOptimize_person_visitation.person_id IN (1);

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id IN (1, 2);
```

I rezultujuće tabele će biti:

 id | customer_id | orderinfo      | time
----|-------------|----------------|--------------------
  1 | 1           | Info of order  | 2014-08-10 17:05:48

 id | name      | hometown_id | living_id
----|-----------|-------------|----------
 1  | Zhang San | 3           | 1

prefetch_related_val | id | name           | province_id
---------------------|----|----------------|------------
 1                   |  1 | Wuhan City     | 1
 1                   |  2 | Guangzhou City | 2
 1                   |  3 | Shiyan City    | 1

id | name
---|-------
 1 | Hubei Province
 2 | Gunagding Province

Bolji način je pozvati select_related() jednom, potom prefetch_related():

```shell
>>> plist = Order.objects
  .select_related('customer')
  .prefetch_related('customer visitation province')
  .get(id=1)
>>> for city in plist.customer.visitation.all():
... print city.province.name
```

Rezultat je tri upita. Django će prvo pozvati `select_related`, a potom `prefetch_related` koristeći keširane podatke:

```sql
SELECT
  QSOptimize_order.id,
  QSOptimize_order.customer_id, QSOptimize_order.orderinfo,
  QSOptimize_order.time,
  QSOptimize_person.id,
  QSOptimize_person.firstname,
  QSOptimize_person.lastname,
  QSOptimize_person.hometown_id,
  QSOptimize_person.living_id
FROM
  QSOptimize_order
INNER JOIN
  QSOptimize_person ON (QSOptimize_order.customer_id = QSOptimize_person.id)
WHERE
  QSOptimize_order.id = '1';

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
  QSOptimize_person_visitation.person_id IN ('1', );

SELECT
  QSOptimize_province.id,
  QSOptimize_province.name
FROM
  QSOptimize_province
WHERE
  QSOptimize_province.id IN ('1', '2');
```

Rezultujuće tabele su:

id | customer_id | orderinfo | time       |id | name      | hometown_id |living_id
---|-------------|-----------|------------|---|-----------|-------------|----------
1  | 1           | Info of   | 2014-08-10 |1  | Zhang San | 3           | 1

prefetch_related_val | id | name           | province_id
---------------------|----|----------------|-----------
1                    | 1  | Wuhan City     | 1
1                    | 2  | Guangzhou City | 2
1                    | 3  | Shuiyan City   | 1

id |name
---|--------------------
1  | Hubei Province
2  | Guangdong Province

[Sadržaj](00_sadrzaj.md)
