# Dodatak django-admin-lightweight-date-hierarchy

[Sadržaj](00_sadrzaj.md)

U nekoliko projekata smo koristili ovu mogćnost. Odlučili smo da objavimo kao paket.

Instalacija:

```py
pip install django-admin-lightweight-date-hierarchy
```

Dodati u `INSTALLED_APPS`:

```py
INSTALLED_APPS = (
    'django_admin_lightweight_date_hierarchy',
)
```

Postavi `date_hierarchy_drilldown` na `False` na `ModelAdmin`-u da bi se sprečilo podrazumevano propadajuće ponašanje:

```py
@admin.register(MyModel)
class MyModelAdmin(admin.ModelAdmin):
    date_hierarchy = 'created'
    date_hierarchy_drilldown = False
```

[Sadržaj](00_sadrzaj.md)
