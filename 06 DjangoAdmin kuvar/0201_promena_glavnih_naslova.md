
# Promena glavnih naslova

[Sadržaj](00_sadrzaj.md)

Podrazumevano Django admin prikazuje tekst "Django administration" na sledećim stranama:

- Login strana
- Listview strana
- HTML title tag

Hajde da zamenimo sa "UMSRA Administration". Možemo napraviti sve tri promene u `urls.py`:

```py
admin.site.site_header = "UMSRA Admin"
admin.site.site_title = "UMSRA Admin Portal"
admin.site.index_title = "Welcome to UMSRA Researcher Portal"
```

> [!Note]
>
> Ispravan način je nasledjivanje admin klase i zadavanje ova tri atributa klase. Kako mi radimo sa admin.site klasom (default klasa admin strana) ove klasne atribute zadajemo u `urls.py`.

[Sadržaj](00_sadrzaj.md)
