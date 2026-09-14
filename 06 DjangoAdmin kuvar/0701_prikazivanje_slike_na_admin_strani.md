
# Prikazivanje slike iz Imagefield na adminu strani

[Sadržaj](00_sadrzaj.md)

U Hero modelu, imate jedno polje sa slikom:

```py
headshot = models.ImageField(null=True, blank=True, upload_to="hero_headshots/")
```

Od vas može biti traženo da promenite podrazumevano ponašanje i da se slika prikazuje na `changeview` strani:

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    readonly_fields = [..., "headshot_image"]

    def headshot_image(self, obj):
        return mark_safe('<img src="{url}" width="{width}" height={height} />'
            .format( url = obj.headshot.url, width=obj.headshot.width,
                height=obj.headshot.height, )
)
```

[Sadržaj](00_sadrzaj.md)
