# Polja za pretragu na listview strani

[Sadržaj](00_sadrzaj.md)

Django admin pruža `search_fields` opciju na ModelAdmin. Postavkom ove opcije će biti omogućen boks za pretragu na `listview strani`, za filtriranje objekata na modelu. Može uraditi pretragu po svim poljima modela i povezanim poljima modela takodje.

```py
class BookAdmin(admin.ModelAdmin):
    search_fields = ('name', 'author__name')
```

Sa više polja pretrage `search_fields`, upiti postaju spori jer rade case-insensitive pretragu po svim terminima pretrage na svim poljima iz `search_fields`. Jedan primer pretraga "python for data analysis" se pretvara u sledeću SQL klauzulu:

```sql
WHERE
    (name ILIKE '%python%' OR author.name ILIKE '%python %') AND
    (name ILIKE '%for%' OR author.name ILIKE '%for% ') AND
    (name ILIKE '%data%' OR author.name ILIKE '%data%') AND
    (name ILIKE '%analysis%' OR author.name ILIKE '%analysis%')
```

[Sadržaj](00_sadrzaj.md)
