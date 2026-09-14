
# Veze do drugih strana sa listama objekata modela

[Sadržaj](00_sadrzaj.md)

Uobičajeno je da se objekti pozivaju na druge objekte pomoću stranih ključeva. list_display možete usmeriti na metodu koja vraća HTML vezu. Unutar core/admin.py izmenite klasu CourseAdmin na sledeći način:

```py
from django.urls import reverse
from django.utils.http import urlencode
@admin.register(Course)
class CourseAdmin(admin.ModelAdmin):

list_display = ("name", "year", "view_students_link")
def view_students_link(self, obj):

count = obj.person_set.count()
url = (reverse( "admin:core_person_changelist") + "?"

+urlencode({"courses id":
f"{obj.id}"})

)
return format_html('<a href="{}">{} Students</a>', url, count)
```

Ovaj kod dovodi do toga da lista objekata modela Course ima tri kolone:

- Naziv kursa
- Godina u kojoj je kurs ponuđen
- Veza koja prikazuje broj studenata na kursu

Kada kliknete na 2 učenika, preusmerava vas na stranicu sa listom objekata modela Person sa primenjenim filterom. Filtrirana stranica prikazuje samo studente na odgovarajućem kursu: Psich 101, Buffy i Villow. Xander se nije prijavio na ovaj kurs.

Primer koda koristi reverse()metodu za dobijanje je URLadrese u Django adminu. Možete potražiti bilo koju stranicu admina koristeći sledeću konvenciju:

```py
"admin:%(app)s_%(model)s_%(page)"
```

Ova struktura deli se na sledeći način:

- `admin`: je prostor imena.
- `app`: je naziv aplikacije.
- `model`: je objekt modela.
- `page`: je tip stranice u Django adminu.

Za gornji primer view_students_link() koristite admin: core_person_changelist da biste dobili referencu na stranicu liste objekta modela Personu core aplikacije.Evo dostupnih naziva URL-ova:

 Strana     | URL                            | Ime
------------|--------------------------------|---------
Listview    | %(app)s\_%(model)s\_changelist | listview
Addview     | %(app)s\_%(model)s\_add        | add
Historyview | %(app)s\_%(model)s\_history    | history  
Deleteview  | %(app)s\_%(model)s\_delete     | delete
Changeview  | %(app)s\_%(model)s\_change     | change

Stranicu liste objekata možete da filtrirate dodavanjem stringa na URL-u. Ovaj string upita modifikuje QuerySet koji se koristi za popunjavanje stranice. U gornjem primeru, string upita "?courses id = {obj.id}" filtrira listu osoba samo prema onim objektima koji imaju odgovarajuću vrednost u polju "Person.course".

Ovi filteri podržavaju pretragu polja `QuerySet` pomoću dvostrukih donjih crta (`__`). Možete pristupiti atributima povezanih objekata, kao i koristiti modifikatore filtera kao što su `exact` i `startswith`.

[Sadržaj](00_sadrzaj.md)
