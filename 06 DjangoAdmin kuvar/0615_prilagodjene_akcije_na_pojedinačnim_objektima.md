
# Prilagodjene akcije na pojedinačnim objektima

[Sadržaj](00_sadrzaj.md)

Grupne admin akcije su neefikasne na pojedinačnim objektima. Na primer, za brisanje jednog korisnika
treba da sledimo sledeće korake.

1. Izabrati objekat.
2. Klikuti na akcijsku padajuću listu.
3. Izabrati “Delete selected” akciju.
4. Kliknuti na Go dugme.
5. Potvrditi da objekat treba da bude obrisan.

Za brisanje jednog zapisa imamo pet klikova. To je previše za jednu akciju. Da pojednostavimo proces, napravimo dugme na nivou vrste (objekta). To može biti postignuto pisanjem funkcije koja će insertovati delete dugme za svaki zapis.

ModelAdmin instance pružaju set imenovanih URL-ova za CRUD operacije. Dobijanje objekt url-a za jednu stranu biće:

```py
{{ app_label }}_{{model_name }}_{{ page }}
```

Na primer, za dobijanje delete URL-a za book objekat možemo pozvati

```py
reverse(‘admin:book_book_delete’, args=[book_id])
```

Možemo dodati "delete" dugme i dodati ga u `list_display` tako da je delete dugme raspoloživo za svaki pojedinačni objekat.

```py
from django.contrib import admin
from django.utils.html import format_html
from book.models import Book

class BookAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'author', 'is_available', 'delete')

    def delete(self, obj):
        view_name = "admin:{}_{}_delete".format(obj._meta.app_label,
        obj._meta.model_name)
        link = reverse(view_name, args=[book.pk])
        html = '<input type="button" onclick="location.href=\'{}\'"value = "Delete"/>'.format(link)
        return format_html(html)
```

Sada u adminu imamo delete dugme za svaki pojedinačni objekat. Za brisanje jednog objekta je potreban klik na delete dugme, i potom drugi klik za potvrdu brisanja.

U prethodnom primeru mi smo koristili ugradjeni admin "delete" pogled. Možemo takodje napraviti prilagodjeni pogled i povezati ga sa prilagodjenom akcijom na individualni objekat. Na primer, možemo dodati dugme koje će markirati status knjige na raspoloživ.

[Sadržaj](00_sadrzaj.md)
