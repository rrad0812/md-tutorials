
# Autokompletiranje za povezana polja

[Sadržaj](00_sadrzaj.md)

Hajdemo u "BookAdmin" i pokušajmo da dodamo novu knjigu. Da bi popravili podrazumevano ponašanje, možemo pružiti autokompletirajuću opciju za polje "author" tako da korisnicimogu pretraživati i izabrati potrebnog autora.

```py
from book.models import Book

class AuthorAdmin(admin.ModelAdmin):
    search_fields = ('name',)

class BookAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'author')
    autocomplete_fields = ('author',)

admin.site.register(Book, BookAdmin)
```

`ModelAdmin` pruža `autocomplete_fields` opciju za `select2` autokompletirajući ulaz. Takodje treba da definišemo `search_fields` na povezanom polju modela tako da admin može da pretražuje po njemu.

[Sadržaj](00_sadrzaj.md)
