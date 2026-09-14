
# Akcije kao ModelAdmin metode

[Sadržaj](00_sadrzaj.md)

Gornji primer prikazuje `make_published` akciju definisanu kao funkcija. To je sasvim u redu, ali sa stanovišta dizajna koda nije savršeno: budući da je akcija čvrsto povezana sa `Article` objektom, ima smisla akciju povezati sa samim `ArticleAdmin` objektom.

```py
class ArticleAdmin(admin.ModelAdmin):
    ...
    actions = ['make_published']

    @admin.action(description='Mark selected stories as published')
    def make_published(self, request, queryset):
        queryset.update(status='p')
```

Preselili smo funkciju `make_published` u klasu `ModelAdmin`. Prvi parametar je sada `self`, i na kraju `make_published` je string u listi actions, umesto direktne reference na callable. Ovo govori da je akcija ModelAdmin metoda.

Definisanje akcija kao metoda daje akciji idiomatičniji pristup ModelAdmin-a samog sebi, omogućavajući akciji da pozove bilo koju od metoda koje pruža admin.

Na primer, možemo koristiti self za "fleš" poruke korisniku obaveštavajući ga da je akcija uspela:

```py
from django.contrib import messages
from django.utils.translation import ngettext

class ArticleAdmin(admin.ModelAdmin):

    def make_published(self, request, queryset):
        updated = queryset.update(status='p')
        self.message_user(request, ngettext(
            '%d story was successfully marked as published.',
            '%d stories were successfully marked as published.', updated,
        ) % updated, messages.SUCCESS)
```

[Sadržaj](00_sadrzaj.md)
