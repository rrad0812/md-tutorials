
# Prefetching

[Sadržaj](00_sadrzaj.md)

Skoro sam radio prijavni sistem za jednu konferenciju. Bilo im je veoma važno da vide tabelu prijava uključujući kolonu sa listom imena programa. Ugrubo modeli izgledaju ovako:

```py
class Program(models.Model):
  name = models.CharField(max_length=20,)

class Price(models.Model):
  program = models.ForeignKey(Program,)
  from_date = models.DateTimeField()
  to_date = models.DateTimeField()

class Order(models.Model):
  state = models.CharField(max_length=20,)
  items = models.ManyToManyField(Price)
```

Ovde je:

- Program – jedna sesija, lektura ili konferencijski dan.
- Price – cena koja se može menjati u vremenu. Model reprezentuje cenu programa u odredjenom vremenskom trenutku.
- Order – prijava na jedan ili više programa. Svaka stavka u prijavi ima cenu u trenutku kada je prijava napravljena.

Kroz log mi ćemo pratiti upite izvršene od strane Djanga. Da bi ih logovali dodajmo sledeće u `settings.py`:

```py
LOGGING = {
  ...
  'loggers': {
    'django.db.backends': {'level': 'DEBUG',},
  },
}
```

[Sadržaj](00_sadrzaj.md)

## Šta je problem?

Pokušajmo da dobijemo ime programa iz jedne prijave:

```shell
>>> o = Order.objects.filter(state='completed').first()
```

```sql
SELECT 
  ...
FROM 
  orders_order
WHERE 
  orders_order.state = 'completed'ORDER BY orders_order.id ASC 
LIMIT 1;
```

```shell
>>> [p.program.name for p in o.items.all()]
```

```sql
SELECT 
  ...
FROM 
  events_price
INNER 
  JOIN orders_order_items ON (events_price.id = orders_order_items.price_id)
WHERE 
 orders_order_items.order_id = '29';

SELECT 
  ...
FROM 
  events_program
WHERE 
  events_program.id = '8';
```

Za dobijanje kompletirane prijave potreban je jedan upit. Za dobijanje imena programa za svaku prijavu potrebana su dva upita. Ako trebamo dva upita za svaku prijavu, ukupan broj upita za 100 prijava je 1 + 100 * 2 = 201! Koristimo Django da smanjimo broj upita:

```shell
>>> o.items.values_list('program name')
```

```sql
SELECT 
  events_program.name
FROM 
  events_price
INNER 
  JOIN orders_order_items ON (events_price.id=orders_order_items.price_id)
INNER 
  JOIN events_program ON (events_price.program_id=events_program.id)
WHERE 
  orders_order_items.order_id = 29 
LIMIT 21;
```

Django je uradio povezivanje izmedju Price i Program i smanjo broj upita na jedan po prijavi. Znači umesto 201 upita sada imamo 101 upit.

## Zašto ne možemo povezati tabele?

Ako imamo strani ključ možemo koristiti `select_related` ili 'snake case' da bi dobili povezana polja u jednom upitu. Na primer mi smo dobijali ime programa za listu cena u jednom upitu korišćenjem:

```py
values_list('program name')
```

Ovo smo mogli da uradimo jer je svaka cena bila povezana sa tačno jednim programom. Ako je veza izmedju dva modela više na više, ne možemo to uraditi. Svaka prijava ima jednu ili više povezanih cena, ako ih povežemo kao dosad dobićemo grešku duplih vrednosti.

```sql
SELECT 
  o.id AS order_id, p.id AS price_idFROM orders_order o
JOIN 
  orders_order_items op ON (o.id = op.order_id)
JOIN 
  events_price p ON (op.price_id = p.id)
ORDER BY 1, 2;
```

order_id | price_id
---------|---------
45       | 38
45       | 56
70       | 38
70       | 50
70       | 77
71       | 38

Prijave 70 i 45 imaju više stavki tako da se pojavljuju više puta u rezultatu. Django ne može upravljati sa ovim problemom!

[Sadržaj](00_sadrzaj.md)

## prefetch_related 2

Django ima lep, ugradjen način rukovanja ovim problemom prefetch_related:

```shell
>>> o = Order.objects.filter(state='completed',)
.prefetch_related('items program',).first()
```

```sql
SELECT 
  ...
FROM 
  orders_order
WHERE 
  orders_order.state = 'completed'
ORDER BY 
  orders_order.id ASC 
LIMIT 1;

SELECT 
  (orders_order_items.order_id) AS _prefetch_related_val_order_id, events_price...
FROM 
  events_price
INNER JOIN 
  orders_order_items ON (events_price.id = orders_order_items.price_id)
WHERE 
  orders_order_items.order_id IN (29);

SELECT
  events_program.id,
  events_program.name
FROM 
  events_program
WHERE 
  events_program.id IN (8);
```

Mi smo rekli Djangu da nam je namera da dobijemo "items" program iz rezultujućeg seta. U drugom i trećem upitu možemo videti da Django dobija rezultat kroz tabelu "orders_order_items" i iz "events_program". Rezultat `prefetching`-a su keširani na objektima.

Šta se dešava kada mi pokušamo da dobijemo imena programa iz rezultata?

```shell
>>> [p.program.name for p in o.items.all()]
```

Nema dodatnih upita – tačno ono što smo želeli! Ovde je važno da kada koristimo prefetch, radimo sa objektima a ne sa queryset-om! Pokušaj da dobijemo imena programa sa upitom će proizvesti isti izlaz, queryset, ali će napraviti i dodatni upit:

```shell
>>> o.items.values_list('program name')
```

```sql
SELECT 
  events_program.name
FROM 
  events_price
INNER JOIN 
  orders_order_items ON (events_price.id = orders_order_items.price_id)
INNER JOIN 
  events_program ON (events_price.program_id = events_program.id)
WHERE 
  orders_order_items.order_id = 29 
LIMIT 21;
```

Na ovaj način imamo, za 100 prijava 3 upita.

[Sadržaj](00_sadrzaj.md)
