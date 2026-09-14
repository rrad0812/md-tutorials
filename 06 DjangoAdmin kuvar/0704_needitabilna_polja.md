
# Needitabilna polja u adminu

[Sadržaj](00_sadrzaj.md)

Ako imate polje sa `editable=False` u modelu, to je polje automatski sakriveno na vašem modelu. To se takodje dogadja sa poljima koja su markirana sa `auto_now` ili `auto_now_add`, jer su kod njih atribut `editable=False` već postavljeni.

Ako želite da ta polja budu prikazana, možete ih dodati u `readonly_fields`:

```py
@admin.register(Villain)
class VillainAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    readonly_fields = ["added_on"]
```

[Sadržaj](00_sadrzaj.md)
