
# Dodavanje OneToOne veze kao inline

[Sadržaj](00_sadrzaj.md)

OneToOne veza može biti postavljena na isti način kao i ManyToOne. Ali, samo jedna strana OneToOne veze može biti postavljena kao inline.

Možete imati HeroAcquaintance model sa OneToOne vezom na Hero korišćenjem:

```py
class HeroAcquaintance(models.Model):
    """Non family contacts of a Hero"""
    hero = models.OneToOneField(Hero, on_delete=models.CASCADE)
    ....
```

Možete dodati kao inline na Hero kao:

```py
class HeroAcquaintanceInline(admin.TabularInline):
model = HeroAcquaintance

@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    inlines = [HeroAcquaintanceInline]
    inlines = [HeroAcquaintanceInline]
```

[Sadržaj](00_sadrzaj.md)
