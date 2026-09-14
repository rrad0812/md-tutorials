
# Kako promeniti tekst na FK padajućoj listi?

[Sadržaj](00_sadrzaj.md)

"Hero" model ima FK vezu na Category model. U padajućoj listi, umesto samo imena, možda želite da prikažete tekst: "Category: name".

Možete promeniti `str` metod na modelu "Category", ali vi želite samo promenu na adminu. Možete to uraditi nasledjivanjem `forms.ModelChoiceField` sa prilagodjenom `label_from_instance`:

```py
class CategoryChoiceField(forms.ModelChoiceField):

    def label_from_instance(self, obj):
        return "Category: {}".format(obj.name)
```

Sada možete nadjačati `formfield_for_foreignkey` za korišćenje novog tipa polja za polje "category":

```py
def formfield_for_foreignkey(self, db_field, request, **kwargs):
    if db_field.name == 'category':
        return CategoryChoiceField(queryset=Category.objects.all())
    return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

[Sadržaj](00_sadrzaj.md)
