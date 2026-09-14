
# Ajax request u adminu na edit strani

[Sadržaj](00_sadrzaj.md)

U ovom tutorijalu učimo kako kreirati Ajax request za validaciju nekih podataka dok editujemo podatke u Django admin edit formi.

## Izgradnja Ajax request-a od nule u admin-u

Koristimo sledeći model:

```py
class Product(models.Model):
    user = models.ForeignKey(User, null=True, blank=True, on_delete=models.CASCADE)
    refShop = models.ForeignKey(Shop, db_index=True,on_delete=models.CASCADE)
    title = models.TextField(max_length=65)
    price = models.FloatField(default=0)
    withEndDate = models.BooleanField(default=False)
    endDate = models.DateField(blank=True,null=True)
    description = models.CharField(max_length=65, default='', null=True, blank=True)
    createdAt = models.DateTimeField(auto_now_add=True)
    updatedAt = models.DateTimeField(auto_now=True)
    available = models.BooleanField(default=True)

    def clean(self):
        from datetime import datetime, timedelta
        from django.core.exceptions import ValidationError
        
        today = datetime.now().date()
        if self.endDate:

        if self.endDate < today:
            raise ValidationError("You cannot set an endDate in the past")

        if self.withEndDate and self.endDate is None:
            raise ValidationError("Please select an end date !")

    def unicode (self):
        return u'%s - %s' % (self.refShop,self.title)
```

Recimo da bi smo, kada dodajemo ili editujemo model Product, voleli da verifikujemo cenu unešenu od strane korisnika u odredjenom rangu vrednosti. Da bi to uradili treba da osluškujemo (korišćenjem javascript-a) ’price’ ulaznu liniju, i ako je vrednost unešena, uradimo jedan Ajax zahtev da proverimo da li je cena u odgovarajućem rangu.

Prvo moramo da prepišemo podrazumevani change_form.html fajl admina, prvo kreiramo novi direktorijum `<application>/templates/admin/<application>/product/`
I kopiramo unutra fajl podrazumevani change_form.html fajl iz našeg virtualenv cp .../site-packages/django/contrib/admin/templates/admin/change_form.html
Pre modifikacije tog fajla, pogledajmo izvorni kod strane, da bi znali kako je ’price’ polje imenovano:

```html
<div class="form-row field-price">
<div>

<label class="required" for="id_price">Price:</label>
<input type="number" name="price" value="2.0" step="any" required id="id_price">

</div>
</div>
```

Kao što smo očekivali ulazno polje je idetifikovano sa “id_price”. Sada modifikujemo dati fajl dodavanjem javascript koda unutar pratećeg bloka:

```html
{% block admin_change_form_document_ready %}
<script src="https://code.jquery.com/jquery-3.3.1.min.js"></script> <script>
  django.jQuery(document).ready(function(){
    django.jQuery("#id_price").change(function(){
      console.log("Price change");
    });
  })
</script>
```

Pre implementacije Ajax request-a, mi samo logujemo u javascript konzolu promenu cene, da proverimo je sve OK. Ako je sve OK. Sada možemo implementirati Ajax poziv:

```js
<script>

django.jQuery(document).ready(function(){
  django.jQuery("#id_price").change(function(){

  console.log("Price change");
  var price = $(this).val();
  var url="http://127.0.0.1:8000/backoffice/checkPrice/"
  $.ajax({ // initialize an AJAX request
    url: url, // set the url of the request
    data: {
      'price': price
    },
    success: function(data) { // `data` is the return of the `checkPrice` view
      // function
    console.log(data);
    }
  });
  });
})
</script>
```

Dobili smo cenu kao $(this).val(); i potom smo konstruisali naš url na kome ćemo napraviti poziv. Sada, ako reučitamo našu stranu i pokušamo da promenimo cenu, videćemo na javascript konzolnom logu:

Imamo 404 error page što je logično jer još nismo implementirali checkPrice prikaz. Uradimo to.

U views.py fajlu aplikacije, kreirajmo jednostavnu funkciju prikaza da proverimo vrednost cene:

```PY
from django.http import JsonResponse

def checkPrice(request):
  price = request.GET['price']
  if int(price) > 10:
    data = {"status": "KO"}
   else:
     data = {"status":"OK"}
   return JsonResponse(data)
```

Do vas je da u prikazu implementirate vaša biznis pravila. Za ovaj primer, proverićemo da li je cena veća od 10 i i vraćamo nazad JSON sa KO ako nije ili OK statusom ako jeste.

Potom u našem backoffice direktorijumu, kreirajmo urls.py fajl:

```py
#coding=utf-8
from django.conf.urls import url
from backoffice.views import *

urlpatterns = [
    url(r'^backoffice/checkPrice/$', checkPrice, name='checkPrice'),
]
```

Na kraju deklarišemo novu rutu unutar projektnog urls.py fajla:

```PY
from django.conf.urls import url,include
from django.contrib import admin
from backoffice.admin import

site admin.site = site admin.autodiscover()
urlpatterns = [
    url('', include('backoffice.urls')),url(r'^admin/', admin.site.urls),
]
```

Sada, ako reučitate stranu i pokušate da promenite cenu, videćete u konzolnom logu Ajax odgovor sa statusom. Na vama je šta ćete da uradite sa Ajax odgovorom. Na primer, proverićemo da li je status OK i ako jeste postavićemo cenu na 10 u input boksu.

```JS
<script>

django.jQuery(document).ready(function(){
  django.jQuery("#id_price").change(function(){
  
  console.log("Price change");
  var price = $(this).val();
  var url="http://127.0.0.1:8000/backoffice/checkPrice/"
  $.ajax({ // initialize an AJAX request
    url: url, // set the url of the
    request data: {
      'price': price
    },
    success: function (data) {// `data` is the return of the `checkPrice` view
      // function
      console.log(data);
      if (data["status"]=="KO"){
        $("input#id_price").val(10);
      }
    }
  });
  });
})

</script>
```

Sada, ako reučitate admin stranu i pokušate da unesete vrednost veću od 10 za cenu, vrednost će automatski biti postavljena na 10, zahvaljujući Ajax pozivu.

[Sadržaj](00_sadrzaj.md)
