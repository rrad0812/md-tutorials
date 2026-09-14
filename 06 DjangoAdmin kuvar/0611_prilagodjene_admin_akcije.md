
# Prilagodjene admin akcije

[Sadržaj](00_sadrzaj.md)

Kreiranje akcije u Django adminu je vrlo jednostavno. Morate definisati funkciju koja je potom referencirana u ModelAdmin definiciji akcija. Ova funkcija če prihvatiti tri argumenta:

- ModelAdmin
- HTTP request
- queryset izabranih modela

"OrderAdmin" sa korisničkom akcijom koja ažurira status za izabrane stavke:

`admin.py`

```py
class OrderAdmin(admin.ModelAdmin):
    actions = ['update_status']
    
    def update_status(self, request, queryset):
        queryset.update(status='NEW_STATUS')
        update_status.short_description = "Update status"
```

`short_description` property se koristi za prilagodjenje labele u padajućoj listi akcija.

[Sadržaj](00_sadrzaj.md)
