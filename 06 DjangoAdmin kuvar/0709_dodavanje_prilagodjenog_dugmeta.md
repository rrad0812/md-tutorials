
# Dodavanje prilagodjenog dugmeta na changeview stranu

[Sadržaj](0709_dodavanje_prilagodjenog_dugmeta.md)

Model Villain ima polje `is_unique`:

```py
class Villain(Entity):
...
is_unique = models.BooleanField(default=True)
```

Želite da dodate dugme na Villain `changelist` stranu sa nazivom “Napravi Unique”. Svi drugi villain objekti sa istim imenom trebalo bi obrisati.

Počinjemo proširenjem `change_form` šablona dodavanjem novog dugmeta

`(entities/vilian_changeform.html)`

```html
{% extends 'admin/change_form.html' %}
{% block submit_buttons_bottom %}
{{ block.super }}
<div class="submit-row">

<input type="submit" value="Make Unique" name="_make-unique">
</div>
{% endblock %}
{% block submit_buttons_bottom %}
{{ block.super }}

<div class="submit-row">

<input type="submit" value="Make Unique" name="_make-unique">
</div>
{% endblock %}
```

Tada možemo nadjačati response_change i povezati šablon sa VillainAdmin-om.:

```py
@admin.register(Villain)
class VillainAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    change_form_template = "entities/villain_changeform.html"
    def response_change(self, request, obj):
        if "_make-unique" in request.POST:
            matching_names_except_this =
                self.get_queryset(request).filter(name=obj.name)
                    .exclude(pk=obj.id)
            matching_names_except_this.delete()
        
            obj.is_unique = True
            obj.save()
            self.message_user(request, "This villain is now unique")
            return HttpResponseRedirect(".")
        return super().response_change(request, obj)
```

[Sadržaj](0709_dodavanje_prilagodjenog_dugmeta.md)
