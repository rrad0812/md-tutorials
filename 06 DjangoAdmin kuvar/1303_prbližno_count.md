
# Približno COUNT

[Sadržaj](00_sadrzaj.md)

Drugi način da se smanji vreme provedeno na COUNT upitu korišćenjem nekih ugrađenih Postgres funkcija za procenu ukupnog broja zapisa kada brojanje traje predugo. Možete videti temeljnu implementaciju pristupa koja preopterećuje count metodu u Djangoovoj `Paginator` klasi.

Prva stvar da se uradi je da se postavi `statement_timeout` na upit i rezervnu opciju procenjenih zapisa. Možete koristiti atomske transakcije da postavite vremensko ograničenje na 150 ms.

```py
try:
  with transaction.atomic(), connection.cursor() as cursor:
    # Limit to 150 ms
    cursor.execute('SET LOCAL statement_timeout TO 150;')
    return super().count
except OperationalError:
  pass
...
```

Ako `count` metoda vraća podatke pre isteka vremenskog ograničenja, onda se koristi stvarna vrednost. Međutim, ako upit potraje duže, Django će se vratiti na približnu vrednost uskladištenu u `pg_class` metapodacima. Metapodaci se ažuriraju kada se izvršavaju komande kao `VACUUM`, `ANALYZE` i `CREATE INDEX`, ili se radi `autovacuum`  na tabeli.

```py
...
with transaction.atomic(), connection.cursor() as cursor:
  # Obtain estimated values (only valid with PostgreSQL)
  cursor.execute("SELECT reltuples FROM pg_class WHERE relname = %s",
    [self.object_list.query.model._meta.db_table])
  estimate = int(cursor.fetchone()[0])
  return estimate
  ...
```

Nakon primene ove metode, maksimalno vreme koje ćete potrošiti na učitavanje COUNT je 150 ms.

[Sadržaj](00_sadrzaj.md)
