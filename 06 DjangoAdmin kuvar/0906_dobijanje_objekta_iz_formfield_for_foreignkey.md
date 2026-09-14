
# Dobijanje objekta iz formfield_for_foreignkey

[Sadržaj](00_sadrzaj.md)

Pokušao sam da filtriram opcije prikazane u foreignkey polju, unutar django admin inlajna, zbog njihovog editovanja.

```py
class ProjectGroupMembershipInline(admin.StackedInline):
    model = ProjectGroupMembership
    extra = 1
    formset = ProjectGroupMembershipInlineFormSet
    form = ProjectGroupMembershipInlineForm
    
    def formfield_for_foreignkey(self, db_field, request=None, **kwargs):
        if db_field.name == 'group':
            kwargs['queryset'] = Group.objects.filter(

        some_filtering_here=object_being_edited)
        return super(ProjectGroupMembershipInline, self)

formfield_for_foreignkey(db_field, request, **kwargs))
```

Verifikovao sam da je kwargs prazan kada se edituje objekat, tako da ne mogu dobiti objekat odatle. Pomoć?

[Sadržaj](00_sadrzaj.md)

**Odgovor_1**:

Da bi filtrirali izbore raspložive u polju stranog ključa admin inlajna, treba nadjačati formu tako da bude moguć apdejt atributa queryset polja forme. Na taj način možete pristupiti self.instance što je objekat koji se edituje u formi.

```py
class ProjectGroupMembershipInlineForm(forms.ModelForm):

def init (self, *args, **kwargs):
super(ProjectGroupMembershipInlineForm, self). init (*args, **kwargs)
self.fields['group'].queryset = Group.objects.filter(

some_filtering_here=self.instance)
```

Nema potrebe da koristite `formfield_for_foreignkey`.

[Sadržaj](00_sadrzaj.md)

**Odgovor_2**:

Drugi način, malo čistiji, sličan gornjem odgovoru:

```py
class ProjectGroupMembershipInline(admin.StackedInline):
    # irrelevant bits....
    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        if db_field.name == "group":
            try:
                parent_id = request.resolver_match.args[0]
                kwargs["queryset"] = Group.objects.filter(some_column=parent_id)
            except IndexError:
                pass
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_3**:

Moguće je to rešiti korišćenjem formfield_for_foreignkey i uklanjanjem object ID-a iz url-a. Nije najleši način dobijnja ID-a ali Django ne obezbeđuje pristup object ID-u.

```py
class ObjectAdmin(admin.ModelAdmin):

    def formfield_for_foreignkey(self, db_field, request, **kwargs):
        obj_id = request.META['PATH_INFO'].rstrip('/').split('/')[-1]
        if db_field.name == 'my_field' and obj_id.isdigit():
            obj = self.get_object(request, obj_id)
            if obj:
                kwargs['queryset'] = models.Object.objects.filter(field=obj)
                return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_4**:

Django smešta parent id u kwargs a ne u args tako da korektan je sledeći način:

```py
parent_id = request.resolver_match.kwargs.get('object_id')
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_5**:

Ovo je rešenje do koga sam došao

```py
def formfield_for_foreignkey(self, db_field, request, **kwargs):
    if db_field.name == "group":
        parent_id = request.resolver_match.kwargs['object_id']
        kwargs["queryset"] = Group.objects.filter(some_column=parent_id)
        return super().formfield_for_foreignkey(db_field, request, **kwargs)
```

[Sadržaj](00_sadrzaj.md)
