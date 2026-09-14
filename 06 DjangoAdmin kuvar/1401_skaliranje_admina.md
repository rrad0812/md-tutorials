
# Skaliranje admina

[Sadržaj](00_sadrzaj.md)

Django pruža snažan I fleksibilan admin interfejs. Medjutim kako tabele bivaju punije sa podacima, počinje usporavanje odgovora pri pretrazi. Sa nekom jednostavnim prilagodjenjima moguće je nastaviti sa korišćenjemDjango admina čak i sa velikim dataset-ovima.

## Evidentiranje

Većina Djangovog rada obavlja SQL upite, tako da će naš glavni fokus biti na smanjenju količine upita. Da biste pratili izvršavanje upita, možete da koristite jedno od sledećeg:

- django-debug-toolbar
- možete prijaviti SQL upite na konzolu dodavanjem sledećeg u settings.py.

`settings.py`

```py
LOGGING = {
  ...
  'loggers': {
    'django.db.backends':
    { 'level': 'DEBUG',},
  },
...
}
```

[Sadržaj](00_sadrzaj.md)

## N + 1 problem

N + 1 problem je dobro poznat problem u ORM-ima. Da bismo ilustrovali problem, recimo da imamo ovu šemu:

```py
class Category(models.Model):
    name = models.CharField(max_length=50)

    def str(self):
        return self.name

class Product(models.Model):
    name = models.CharField(max_length=50)
    category = models.ForeignKey(Category)

Implementacijom `__str__` kažemo Django-u da želimo da se naziv kategorije koristi kao opis objekta.

Kad god ispisujemo objekt kategorije, Django će dohvatiti ime kategorije. Kod za
admin stranu:

class Category(models.Model):
    name = models.CharField(max_length=50)

    def str (self):
        return self.name

class Product(models.Model):
    name = models.CharField(max_length=50)
    category = models.ForeignKey(Category)
```

Ovo se čini nedužnim, ali SQL dnevnik otkriva užas:

```sql
SELECT COUNT(*) AS count 
  FROM app_product

SELECT app_product.id, app_product.name, app_product.category_id
  FROM app_product
  ORDER BY app_product.id
  DESC LIMIT 100;

SELECT ... FROM app_category where app_category.id = 1;
SELECT ... FROM app_category where app_category.id = 2;
SELECT ... FROM app_category where app_category.id = 1;
SELECT ... FROM app_category where app_category.id = 4;
SELECT ... FROM app_category where app_category.id = 3;
...
SELECT ... FROM app_category where app_category.id = 2;
SELECT ... FROM app_category where app_category.id = 99;
SELECT ... FROM app_category where app_category.id = 104;
```

Django prvo broji objekte (o tome više kasnije), zatim preuzima stvarne objekte (ograničavajući se na podrazumevanu veličinu stranice od 100), a zatim prosleđuje podatke u obrazac za prikazivanje.

Ime kategorije smo koristili kao opis objekta "category", tako da za svaki proizvod Django mora da preuzme "name" kategorije. Rezultat je dodatnih 100 upita.

Da bismo rekli Djangu da želimo da izvršimo pridruživanje umesto da preuzmemo imena kategorija jednu po jednu, možemo koristiti `list_select_related`:

```py
@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ( 'id', 'name', 'category', )
    list_select_related = ('category',)
```

Sada SQL dnevnik izgleda mnogo lepše. Umesto 101 upita imamo samo 1:

```sql
SELECT app_product.id, app_product.name, app_product.category_id,
    app_category.id, app_category.name
  FROM app_product
  INNER JOIN app_category ON (app_product.category_id = app_category.id)
  ORDER BY app_product.id
  DESC LIMIT 100;
```

Da biste razumeli stvarni uticaj ove postavke, uzmite u obzir sledeće.

Podrazumevana veličina stranice za Django je 100 objekata. Ako imate jedno povezano polje, imate ~ 101 upit. Ako su u prikazu liste prikazana dva povezana objekta, imate ~ 201 upit i tako dalje. Preuzimanje povezanih polja u relaciji može raditi samo za veze `ForeignKey`. Ako želite prikazati relacije `ManyToMany`, malo je složenije (i najčešće pogrešno), ali nastavite čitati.

[Sadržaj](00_sadrzaj.md)
