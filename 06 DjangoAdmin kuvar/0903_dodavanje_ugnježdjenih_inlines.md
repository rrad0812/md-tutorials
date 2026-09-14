
# Dodavanje ugnježdene inlines

[Sadržaj](00_sadrzaj.md)

Imate modele ovako definisane:

```py
class Category(models.Model):
    ...

class Hero(models.Model):
    category = models.ForeignKey(Catgeory)
    ...

class HeroAcquaintance(models.Model):
    hero = models.OneToOneField(Hero, on_delete=models.CASCADE)
```

Želite jednu admin stranu za kreiranje Category, Hero i HeroAcquaintance objekata. Django ne podržava umetnute inline sa ManyToOne ili OneToOne vezama koje povezuju više od jednog nivoa. Imate malo opcija:

- Možete promeniti HeroAcquaintance model, tako da imate direktnu FK vezu na Category:

    ```py
    class HeroAcquaintance(models.Model):
        hero = models.OneToOneField(Hero, on_delete=models.CASCADE)
        category = models.ForeignKey(Category)
    
        def save(self, *args, **kwargs):
            self.category = self.hero.category
            super().save(*args, **kwargs)
    ```

    Potom možete povezati HeroAcquaintanceInline na CategoryAdmin, i dobiti neku
    vrstu umetnutog inline.

- Alternativno, postoje gotovi Django add-on apps, koje pružaju umetnute inlines.

Pogledaj na Github ili DjangoPackages.

[Sadržaj](00_sadrzaj.md)
