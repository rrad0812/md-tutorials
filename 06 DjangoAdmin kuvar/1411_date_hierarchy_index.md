# date_hierarchy

[Sadržaj](00_sadrzaj.md)

Otkrili smo da se ovaj indeks može koristiti za poboljšanje generisanja upita sa predikatom hijerarhije datuma u PostgresSQL 9.4:

```sql
CREATE INDEX yourmodel_date_hierarchy_ix ON yourmodel_table (
  extract('day' from created at time zone 'America/New_York'),
  extract('month' from created at time zone 'America/New_York'),
  extract('year' from created at time zone 'America/New_York')
)
```

[Sadržaj](00_sadrzaj.md)
