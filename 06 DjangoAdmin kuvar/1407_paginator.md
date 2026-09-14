# Paginator

[Sadržaj](00_sadrzaj.md)

`Paginator` svakog admin pogleda zahteva ukupan broj zapisa, što dovodi do prebrojavanja zapisa preko cele tabele. Za tabelu sa milionima zapisa, prebrojavanje može potrajati. Za admin pogled za velike tabele, možemo pokušati da sprečimo prebrojavanje. Možemo nadjačati `paginator` atribut admin objekta
sa sledećim blokom koda.

```py
from django.contrib import admin
from django.core.paginator import Paginator
class BigTablePaginator(Paginator):

    @property
    def count(self):
        return 9999999

class BigTableAdmin(admin.ModelAdmin):
    paginator = BigTablePaginator
    show_full_result_count = False
```

Za svaku veliku tabelu, možemo naslediti `BigTableAdmin`. Na taj način možemo izbeći prebrojavanje zapisa u velikim tabelama.

[Sadržaj](00_sadrzaj.md)
