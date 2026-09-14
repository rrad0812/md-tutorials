
# Dobijanje admin urla za specifične objekte

[Sadržaj](00_sadrzaj.md)

Imate children kolonu koja prikazuje imena dece svakog od Hero objekata. Od vas se traži da linkujete svako od dece na njegovu changeview stranu.

```py
@admin.register(Hero)
class HeroAdmin(admin.ModelAdmin, ExportCsvMixin):
    ...
    def children_display(self, obj):
        display_text = ", ".join([ 
            "<a href={}>{}</a>".format( reverse('admin:{}_{}_change'.format(obj._meta.app_label,obj._meta.model_name), args=(child.pk,)), child.name)
            for child in obj.children.all()
        ])
        if display_text:
            return mark_safe(display_text)
        return "-"
```

Ovde su opcije:

Action  | reverse link
--------|----------------------------------------------------------------------
Delete: | reverse('admin:{}_{}_delete'.format(obj._meta.app_label,obj._meta. model_name), args=(child.pk,))
History:| reverse('admin:{}_{}_history' .format(obj._meta.app_label, obj._meta. model_name), args=(child.pk,))
Change: | reverse('admin:{}_{}_change' .format(obj._meta.app_label, obj._meta. model_name), args=(child.pk,))

[Sadržaj](00_sadrzaj.md)
