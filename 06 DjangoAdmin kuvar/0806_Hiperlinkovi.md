
# Hiperlinkovi

[Sadržaj](00_sadrzaj.md)

Hajde da pregledamo "BookAdmin# i izaberimo neku od knjiga.

```py
from django.contrib import admin
from .models import Book

class BookAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'author')

admin.site.register(Book, BookAdmin)
```

Ovde je `name` polje povezano sa `changeview` stranom objekta "book". Ali "author" polje je prikazano kao običan tekst. Ako bi smo želeli da predjemo u odgovarajući objekat autora radi promene njegovog imena, morali bi da se vratimo ma kućnu stranicu, izaberemo list stranu modela autora, nadjemo odgovarajućeg i tada preko linka odela autora, nadjemo odgovarajućeg i tada preko linka predjemo na stranicu za promene i promenimo ime autora.

Ovo izgleda kao prilično dosadan niz akcija, pogotovu ako ih treba ponoviti više puta. Ali, ako bi imali `hiperlink` na polju "autor" koji bi vodio direktno na stranu za promene autora, to bi bilo znatno lakše za rad.

Django pruža opciju za pristup admin pogledima njegovim URL reverzirajućim sistemom. Naprimer, možemo dobiti stranu za promene autorovog objekta u admin "Book" modelu korišćenjem sintakse:

```py
reverse(“admin:model_povezani_model_change”, args=id)
```

Primer:

```py
from django.contrib import admin
from django.utils.safestring import mark_safe

class BookAdmin(admin.ModelAdmin):
    list_display = ('name', 'author_link', )

    def author_link(self, book):
        url = reverse("admin:book_author_change", args=[book.author.id])
        link = '<a href="%s">%s</a>' % (url, book.author.name)
        return mark_safe(link)

    author_link.short_description = 'Author'
```

Sada u "BookAdmin" pogledu, "author" polje će biti hiperlinkovano na changeview stranu "AuthorAdmin" modela, možemo je posetiti jednostavnim klikom. Zavisno od zahteva, možemo linkovati bilo koje polje u django sa drugim poljem ili dodati prilagodjeno polje.

[Sadržaj](00_sadrzaj.md)
