
# Upravljanje History/Model logovima

[Sadržaj](00_sadrzaj.md)

Django history je jednostavna proširiva Django admin lokaciju koju možete koristiti. Naprimer, smeštanjem više odgovarajućih podataka o promenama napravljenim u Django adminu. Kreirajmo model:

```py
from django.db import models

class School(models.Model):
    school_name = models.CharField(max_length=30)
    address = models.CharField(max_length=30)
    principle_name = models.CharField(max_length=30)

    def str (self):
        return str(self.school_name)
```

Napravimo `makemigrations/migrate`

```shell
python manage.py makemigrations
python manage.py migrate
```

Registrujmo novi model na admin.

```py
from django.contrib import admin
from schoolapp.models import School

@admin.register(School)
class SchoolAdmin(admin.ModelAdmin):
    pass
```

Ako pogledate history stranu modela videćete koje su promene napravljene na objektu. Podrazumevao, smeštaju se samo promene napravljene na samom objektu. Recimo da želimo više podataka da sačuvamo o promenama:

```py
from django.contrib import admin
from django.contrib.admin.options import get_content_type_for_model
from django.contrib.admin.models import LogEntry, ADDITION
from schoolapp.models import School

@admin.register(School)
class SchoolAdmin(admin.ModelAdmin):

    def save_model(self, request, obj, form, change):
        super().save_model(request, obj, form, change)
        if change:
            change_message = '{} -{} -{}'.format(obj.school_name,
            obj.address, obj.principle_name)
            LogEntry.objects.create(

    user=request.user,content_type=get_content_type_for_model(obj),
    object_id=obj.id,action_flag=2,
    change_message=change_message,object_repr=obj. str ()[:200]
)
```

Mi smo nadjačali prodrazumevanu implementaciju save_model metoda za dati model.Ovo će sačuvati sve u change_message, što je vidljivo u action koloni.

[Sadržaj](00_sadrzaj.md)
