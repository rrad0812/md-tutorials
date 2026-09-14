
# Modeli sa kojima ćemo raditi u ovom poglavllju

[Sadržaj](00_sadrzaj.md)

U ovom vodiču naučićemo osnovno, prilagođeno i napredno filtriranje poput lančanih filtera pomoću Django admina. Za ovaj vodič koristiće se modeli Shop i Product.

```py
class Shop(models.Model):
    user = models.ForeignKey(User, null=True, blank=True, on_delete=models.CASCADE)
    name = models.TextField()
    email = models.CharField(max_length=200)
    address = models.CharField(max_length=1024,null=True,
    blank=True)createdAt = models.DateTimeField(auto_now_add=True)
    updatedAt = models.DateTimeField(auto_now=True)
    
    def str (self):
        return u'%s' % (self.name)

class Product(models.Model):
    user = models.ForeignKey(User, null=True, blank=True,on_delete=models.CASCADE)
    refShop = models.ForeignKey(Shop, db_index=True,on_delete=models.CASCADE)
    title = models.TextField(max_length=65)
    price = models.FloatField(default=0)
    withEndDate = models.BooleanField(default=False)
    endDate = models.DateField(blank=True,null=True)
    description = models.CharField(max_length=65, default='', null=True, blank=True)
    createdAt = models.DateTimeField(auto_now_add=True)
    updatedAt = models.DateTimeField(auto_now=True)
    available = models.BooleanField(default=True)
```

Zaista jednostavno, proizvod pripada radnji.

[Sadržaj](00_sadrzaj.md)
