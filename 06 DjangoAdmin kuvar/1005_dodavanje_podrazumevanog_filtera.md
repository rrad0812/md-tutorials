
# Dodavanje podrazumevanog filtera u admin

[Sadržaj](00_sadrzaj.md)

Recimo da za admin model naše pasmine želimo da korisnik može istovremeno da vidi sve pasmine iste vrste, dakle pse u jednom trenutku, mačke ili bilo koju drugu, ali ne sve njih zajedno.

Da bismo to uradili, moramo da zamenimo dodatni metod klase SimpleListFilter, value metodu. Ovaj metod je odgovoran da filtru kaže da li postoji izabrana stavka za filtriranje ili ne.

Da bismo to postigli, dodali smo value() metodu u naš filter:

```py
from django.contrib import admin
from adminfilters.models import Species, Breed

class SpeciesListFilter(admin.SimpleListFilter):
    """ 
    Ovaj filter uvek vraća subset instanci modela, bilo filtriranjem 
    po korisničkom izboru ili po default vrednosti.
    """
    title = 'species' parameter_name = 'species'default_value = None

    def lookups(self, request, model_admin):
        """
        Vraća listu slogova, prvi element svakog sloga je kodirana vrednostopcije koja
        će se pojaviti u query URL-u, drugi element je je ime opcije koja će se
        pojaviti na filteru.
        """
        list_of_species = []
        queryset = Species.objects.all(objects.all()for species in queryset:
        list_of_species.append( (str(species.id), species.name) )
        return sorted(list_of_species, key=lambda tp: tp[1])

    def queryset(self, request, queryset):
        """
        Vraća filtrirani queryset baziran na vrednost query stringa,vraćenog sa
        `self.value()`.
        """
        # Uporedjuje zahtevanu vrednost da odluči kako da filtrira queryset.

        if self.value():
            return queryset.filter(species_id = self.value())
        return queryset

    def value(self):
        """
        Prepisivanje ovog metoda imaćemo default vrednost uvek.
        """
        value = super(SpeciesListFilter, self).value()
        
        if value is None:
            if self.default_value is None:
                # Ako nema Species, vrati prvi po name. Inače None.
                first_species = Species.objects.order_by('name').first()
                value = None if first_species is None else first_species.id
                self.default_value = value
            else:
                value = self.default_value
        return str(value)

@admin.register(Breed)
class BreedAdmin(admin.ModelAdmin): list_display = ('name', 'species', )
    list_filter = (SpeciesListFilter, )
```

Samo zamenjujući metodu `value`, postižemo da ako nije izabran nijedan filter, dajemo filteru zadanu vrednost. Ostatak filtera već će vratiti filtrirani upit postavljen na `ModelAdmin`.

> [!Note]
>
> Čak i pritiskom na filter "All" vratiće se lista filtrirana prema podrazumevanoj vrednosti.

[Sadržaj](00_sadrzaj.md)

## Podrazumevani filteri - II način

Postoji mnogo pristupa za primenu podrazumevanih filtera. Neki pristupi uključuju prilagođene filtere ili

ubrizgavanje posebnih parametara upita u zahtev. Želeli smo dato izbegnemo. Otkrili smo da je sledeći pristup jednostavan i jasan:

from urllib.parse import urlencode from django.shortcuts import redirect

```py
class DefaultFilterMixin:

    def get_default_filters(self, request):
        """
        Postavlja default filtere na stranu.
        Vraća (dict): Default filter.
        """
        raise NotImplementedError()

    def changelist_view(self, request, extra_context=None):
        ref = request.META.get('HTTP_REFERER', '')
        path = request.META.get('PATH_INFO', '')
        # Ako već ima query parametre ili strana je ref. samu sebe
        # (propadanjem ili redirektom) ne primenjuj default filter.
        if request.GET or ref.endswith(path):
            return super().changelist_view(request, extra_context=extra_context)
        query = urlencode(self.get_default_filters(request))
        return redirect('{}?{}'.format(path, query))
```

Ako je prikazu liste pristupljeno iz drugog prikaza, a nisu navedeni parametri upita, generišemo podrazumevani upit i preusmeravamo.

Primenimo podrazumevani filter na našoj stranici proizvoda da bismo prikazali samo proizvode stvorene u poslednjih mesec dana:

```py
from django.utils import timezone

@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin, DefaultFilterMixin):
    date_hierarchy = 'created'

    def get_default_filters(self, request):
        now = timezone.now()
        return {'created year': now.year, 'created month': now.month,}
```

Ako idemo na stranicu detalja ili ako dođemo do stranice sa parametrima upita,podrazumevani filter se neće primeniti.

[Sadržaj](00_sadrzaj.md)
