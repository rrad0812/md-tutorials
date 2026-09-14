
# Pisanje akcionih funkcija

[Sadržaj](00_sadrzaj.md)

Prvo ćemo morati da napišemo funkciju koja se poziva kada se akcija pokrene u adminu. Akcione funkcije su regularne funkcije koje uzimaju tri argumenta:

- `ModelAdmin` objekat,
- `HttpRequest` koji predstavlja trenutni zahtev,
- `QuerySet` koji sadrži skup objekata koje je korisnik izabrao.

Našoj funkciji za ažuriranje zapisa neće trebati `ModelAdmin objekt` ili `request` ali ćemo koristiti `queryset`:

```py
def make_published(modeladmin, request, queryset):
    queryset.update(status='p')
```

> [!Note]
>
> Za najbolje performanse koristimo metodu ažuriranja queryseta. Druge vrste
> akcija će možda trebati da se bave svakim objektom pojedinačno, u ovim
> slučajevima bismo iterirali preko queryseta:
>
> ```py
> for obj in queryset:
>     do_something_with(obj)
> ```  

To je zapravo sve što je potrebno za pisanje akcije! Međutim, preduzećemo još jedan neobavezan, ali koristan korak koji će dati akciji "lep" naslov u adminu. Podrazumevano, ova akcija bi se na listi akcija pojavila kao "Make published" sa donjim crtama zamenjenim razmacima. To je u redu, ali možemo pružiti bolje, čoveku prihvatljivije ime pomoću `action()` dekoratora i `make_published` funkcije:

```py
from django.contrib import admin

@admin.action(description='Mark selected stories as published')
def make_published(modeladmin, request, queryset):
    queryset.update(status='p')
```

> [!Note]
>
> Ovo bi moglo izgledati poznato, admin `list_display` opcija koristi sličnu tehniku sa `display()` dekoratorom da pruži čoveku čitljive opise za funkcije povratnog poziva koje su tamo takođe registrovane.  
>
> **Promenjeno Django v3.2**: `description` argument u `action()` dekoratoru ekvivalentan je postavljanju `short_description` atributa akcionoj funkciji u prethodnim verzijama. Direktno postavljanje atributa i dalje je podržano radi povratne kompatibilnosti.

[Sadržaj](00_sadrzaj.md)
