
# Db view na adminu

[Sadržaj](00_sadrzaj.md)

Kreirajmo db pogled na bazi podataka:

```sql
create view entities_entity as
  select id, name 
    from entities_hero 

  union

  select 10000+id as id, name 
    from entities_villain
```

Kao rezultat dobićemo imena "Hero" i "Vilian" objekata. Id za "Vilian" objekte je povećan na 10000+id:

```sql
sqlite> select * from entities_entity;

1|Krishna
2|Vishnu
3|Achilles
4|Thor
5|Zeus
6|Athena
7|Apollo
10001|Ravana
10002|Fenrir
```

Tada gradimo ali sa:

```py
managed = False
```

"AllEntity" model:

```py
class AllEntity(models.Model):
  name = models.CharField(max_length=100)
  
  class Meta:
    managed = False
    db_table = "entities_entity"
```

I dodajemo u admin:

```py
@admin.register(AllEntity)
class AllEntiryAdmin(admin.ModelAdmin):
  list_display = ("id", "name")
```

[Sadržaj](00_sadrzaj.md)
