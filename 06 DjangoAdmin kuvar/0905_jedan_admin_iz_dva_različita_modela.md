
# Kako kreirati jedan admin iz dva različita modela?

[Sadržaj](00_sadrzaj.md)

Hero ima FK vezu na Category, tako da možete izabrati Category iz Hero admina. Ako želite da kreirate Category objekat iz Heroa dmina, možete promeniti formu Hero admina i prilagoditi ponašanje save_model:

```py
class HeroForm(forms.ModelForm):
    category_name = forms.CharField()

class Meta:
    model = Hero
    exclude = ["category"]

@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    form = HeroForm
    ....

    def save_model(self, request, obj, form, change):
        category_name = form.cleaned_data["category_name"]
        category, _ = Category.objects.get_or_create(name=category_name)
        obj.category = category
        super().save_model(request, obj, form, change)
```

Na ovaj način Hero admin može da kreira i ažurira Category model.

[Sadržaj](00_sadrzaj.md)
