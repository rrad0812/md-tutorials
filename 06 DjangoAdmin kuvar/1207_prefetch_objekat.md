
# Prefetch objekat

[Sadržaj](00_sadrzaj.md)

Od ver. 1.7 uveden je novi `Prefetch` objekat koji proširuje mogućnosti `prefetch_related`. Novi objekat pruža programeru mogućnost da nadjača upit koji bi koristio Django za prefetch povezanih objekata.

U našem prethodnom primeru Django koristi dva upita za prefetch – jedan kroz tabelu i jedan kroz program.

Šta ako kažemo Djangu da ih spoji?

```shell
>>> prices_and_programs = Price.objects.select_related('program')
>>> o = Order.objects.filter(state='completed')
        .prefetch_related(Prefetch('items', queryset=prices_and_programs))
        .first()
(0.01) 
```

```sql
SELECT 
  ...
FROM 
  orders_order
WHERE 
  orders_order.state = 'completed' 
ORDER BY 
  orders_order.id
ASC LIMIT 1;

SELECT (
  orders_order_items.order_id) AS _prefetch_related_val_order_id,
  events_price..., events_program...
INNER 
  JOIN events_program ON (events_price.program_id = events_program.id)
WHERE 
  orders_order_items.order_id IN (29);
```

Kreirali smo upit koji povezuje cene sa programima. Potom smo rekli Djangu da koristitaj upit za `prefetch` vrednosti. To je kao da smo rekli Djangu da nam je namera da dobijemo i items i programs za svaku prijavu.

Sada možemo da vratimo imena programa:

```shell
>>> [p.program.name for p in o.items.all()]
```

Nema dodatnih upita ! Idemo na sledeći nivo!

Kada smo govorili o modelima, rekli smo da su prices modelovane kao SCD tabela.
To znači da možemo da pravimo upit samo na aktivne cene.

Jedna cena je aktivna na odredjenom datumu ako je izmedju `from_date` i `end_date`:

```shell
>>> from django.utils import timezone
>>> now = timezone.now()
>>> active_prices = Price.objects.filter(from_date lte=now, to_date gt=now,)
```

Korišćenjem Prefetch objekta mi kažemo Djangu da smesti `prefetch` objekte u novi atribut rezultujućeg seta:

```shell
>>> from django.utils import timezone
>>> now = timezone.now()
>>> active_prices_and_programs = (Price.objects
      .filter(from_date lte=now,to_date gt=now,)
      .select_related('program'))
>>> o = Order.objects.filter(state='completed').prefetch_related(Prefetch('items',
      queryset=active_prices_and_programs, to_attr='active_prices',),).first()
```

```sql
SELECT 
  ...
FROM 
  orders_order
WHERE 
  orders_order.state = 'completed'
ORDER BY 
  orders_order.id
ASC LIMIT 1;

SELECT 
  ...
FROM 
  events_price
INNER 
  JOIN orders_order_items ON (events_price.id = orders_order_items.price_id)
INNER 
  JOIN events_program ON (events_price.program_id = events_program.id)
WHERE (orders_order_items.order_id IN (29) AND
      events_price.from_date <= '2017–04–29T07:53:00.210537+00:00' :: timestamptz AND
      events_price.to_date > '2017–04–29T07:53:00.210537+00:00' :: timestamptz);
```

Vidimo da Django izvršava dva upita, i prefetch upit sada sadrži prilagodjeni filter koji smo definisali. Da bi dobili aktivne cene možemo koristiti novi atribut `to_attr`:

```shell
>>> [p.program.name for p in o.active_prices]
```

Nema dodatnih upita !

[Sadržaj](00_sadrzaj.md)
