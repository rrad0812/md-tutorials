
# Kako postaviti naslov modela u množini?

[Sadržaj](00_sadrzaj.md)

Podrazumevano admin prikazuje imena modela u množini dodajući 's'. Često je to nepravilna množina. Od vas je zatraženo da postavite tačne pravopisne množine: `Categories` i `Heroes`. To možete učiniti postavljanjem `verbose_name_plural` u svojim modelima. Promenite u `models.py`:

```py
class Category(models.Model):
  ...
  class Meta:
    verbose_name_plural = "Categories"

class Hero(Entity):
  ...
  class Meta:
    verbose_name_plural = "Heroes"
```

[Sadržaj](00_sadrzaj.md)
