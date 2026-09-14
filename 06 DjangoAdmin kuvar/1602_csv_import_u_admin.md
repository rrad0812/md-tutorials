
# CSV Import u Django admin

[Sadržaj](00_sadrzaj.md)

Od vas je zatraženo da dozvolite uvoz `CSV`-a na `Hero` adminu. To ćete uraditi dodavanjem linka na listview stranu Hero, koji vodi na stranu sa obrascem za otpremanje.

Napisaćete hendler za `POST` akciju za kreiranje objekata iz CSV-a:

```py
class CsvImportForm(forms.Form):
    csv_file = forms.FileField()

@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    change_list_template = "entities/heroes_changelist.html"

    def get_urls(self)get_urls(self):
        urls = super().get_urls()
        my_urls = [
            ...
            path('import-csv/', self.import_csv),
        ]
        return my_urls + urls

    def import_csv(self, request):
        if request.method == "POST":
            csv_file = request.FILES["csv_file"]
            reader = csv.reader(csv_file)
            #... Create Hero objects from passed in data
            self.message_user(request, "Your csv file has been imported")
            return redirect("..")

form = CsvImportForm()
    payload = {"form":form}
    return render(request, "admin/csv_form.html", payload)
)
```

Zatim kreirate entities/heroes_changelist.html šablon, nadjačavajući `admin/change_list.html`:

```html
{% extends 'admin/change_list.html' %}
{% block object-tools %}
<a href="import-csv/">Import CSV</a>
<br />
{{ block.super }}
{% endblock %}
```

Na kraju kreirate `csv_form.html`:

```html
{% extends 'admin/base.html' %}
{% block content %}
<div>
<form action="." method="POST" enctype="multipart/form-data">
{{ form.as_p }}
{% csrf_token %}
<button type="submit">Upload CSV</button>
</form>
</div>
<br />
{% endblock %}
```

Sa ovim promenama, dobijate link na listview strani.

[Sadržaj](00_sadrzaj.md)
