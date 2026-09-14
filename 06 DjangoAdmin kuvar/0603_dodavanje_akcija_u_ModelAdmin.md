
# Dodavanje akcija u ModelAdmin

[Sadržaj](00_sadrzaj.md)

Dalje, moraćemo da obavestimo ModelAdmin o akciji. Ovo funkcioniše kao i svaka druga opcija konfiguracije. Dakle, komplet `admin.py` sa akcijom i njenom registracijom bi izgledao ovako:

`admin.py`

```py
from django.contrib import admin
from myapp.models import Article

@admin.action(description='Mark selected stories as published')
def make_published(modeladmin, request, queryset):
    queryset.update(status='p')

class ArticleAdmin(admin.ModelAdmin):
    list_display = ['title', 'status']
    ordering = ['title']
    actions = [make_published]
```

[Sadržaj](00_sadrzaj.md)
