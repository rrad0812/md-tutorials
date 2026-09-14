
# Pristupiti parent instanci iz inlines

[Sadržaj](00_sadrzaj.md)

Moj cilj je da nadjačam has_add_permission funkciju bazirano na vrednosti polja status parent instance. Ja ne želim da dozvolim dodavanje dečijih instanci ako je status parent instance različit od 1.

```py
class ChildInline(admin.TabularInline):
    model = Child
    form = ChildForm
        fields = (
            ...
        )
    extra = 0

    def has_add_permission(self, request):
        # Return True only if the parent has status == 1
        # How to get to the parent instance?
        #return True

class ParentAdmin(admin.ModelAdmin):
    inlines = [ChildInline,]
    ...
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_1**:

Za dobijanje parent objekta iskoristiti podatke iz reqeust. resolve funkcija iz django.urls modula vraća podatke koji su nama interesantni. Funkcioniše kada su argumenti prisutni u zahtevu.

```py
from django.contrib import admin
from django.urls import resolve
from app.models import YourParentModel, YourInlineModel

class YourInlineModelInline(admin.StackedInline):
    model = YourInlineModel
    
    def get_parent_object_from_request(self, request):
        resolved = resolve(request.path_info)
        if resolved.args:
            return self.parent_model.objects.get(pk=resolved.args[0])
        return None

    def has_add_permission(self, request):
        parent = self.get_parent_object_from_request(request)
        # Validate that the parent status is active (1)
        if parent:
            return parent.status == 1
        # No parent - return original has_add_permission() check
        return super(YourInlineModelInline, self).has_add_permission(request)

@admin.register(YourParentModel)
class YourParentModelAdmin(admin.ModelAdmin):
    inlines = [YourInlineModelInline]
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_2**:

Da bi došli do parent_obj u child klasi, nadjačamo get_formset, postavljanjući parent_obj kao prosledjeni obj datog metoda.

```py
class ChildInline(admin.TabularInline):
    model = Child
    form = ChildForm
    fields = (...)
    extra = 0

    def get_formset(self, request, obj=None, **kwargs):
        self.parent_obj = obj
        return super(ChildInline, self).get_formset(request, obj, **kwargs)

    def has_add_permission(self, request):
        return self.parent_obj.status == 1

class ParentAdmin(admin.ModelAdmin):
    inlines = [ChildInline, ]
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_3**:

Samo treba da dodate obj parametar i proverite parent model status:

```py
class ChildInline(admin.TabularInline):
    model = Child
    form = ChildForm
    fields = (
        ...
    )
    extra = 0
    
    def has_add_permission(self, request, obj=None):
        if obj.status == 1:
            return True
        else:
            return False

class ParentAdmin(admin.ModelAdmin):
    inlines = [ChildInline,]
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_4**:

Pokušao sam predloženo ali ne radi za mene, morao sam da koristim varijantu iz Odgovor_1:

```py
def get_parent_object_from_request(self, request):
    resolved = resolve(request.path_info)
    if resolved.kwargs:
        return self.parent_model.objects.get(pk=resolved.kwargs["object_id"])
    return None
```
