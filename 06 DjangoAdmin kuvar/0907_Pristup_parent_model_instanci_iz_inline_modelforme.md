
# Pristup parent model instanci iz inline modelforme

[Sadržaj](00_sadrzaj.md)

Koristim `TabularInline` u Django adminu, konfigurisan da prikaže jednu extra blank formu.

```py
class MyChildInline(admin.TabularInline):
model = MyChildModel
form = MyChildInlineForm
extra = 1
```

Modeli su MyParentModel->MyChildModel->MyInlineForm. Koristim prilagođenu formu da dinamički radim lookup i popunjavam lookup polja:

```py
class MyChildInlineForm(ModelForm):
    my_choice_field = forms.ChoiceField()
    
    def init (self, *args, **kwargs):
        super(MyChildInlineForm, self). init (*args, **kwargs)
        # Lookup ID of parent model. parent_id = None
        if "parent_id" in kwargs:
            parent_id = kwargs.pop("parent_id")
        elif
            self.instance.parent_id:
            parent_id = self.instance.parent_id
        elif self.is_bound:
            parent_id = self.data['%s-parent'% self.prefix]
        
        if parent_id:
            parent = MyParentModel.objects.get(id=parent_id)
        if rev:
            qs = parent.get_choices()
            self.fields['my_choice_field'].choices = [(r.name,r.value) for r in qs]
```

Ovo radi dobro za inline zapise u edit modu, ali za ekstra blank formu ne prikazuje vrednosti u mome lookup polju, pošto nema još zapisa i ne može lookup pridruženi MyParentModel zapis.

[Sadržaj](00_sadrzaj.md)

**Odgovor_1**:

Od Django 1.9, tu je get_form_kwargs(self, index)metod u BaseFormSet klasi.

```py
class MyFormSet(BaseInlineFormSet):
    
    def get_form_kwargs(self, index):
        kwargs = super().get_form_kwargs(index)
        kwargs['parent_object'] = self.instance
        return kwargs

class MyForm(forms.ModelForm):

    def init (self, *args, parent_object, **kwargs):
        self.parent_object = parent_object
        super(MyForm, self). init (*args, **kwargs)

class MyChildInline(admin.TabularInline):
    formset = MyFormSet
    form = MyForm
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_2**:

Koristim Django 1.10 i ovo radi za mene:

Kreiraj formset i postavi parent objekat u kwargs:

```py
class MyFormSet(BaseInlineFormSet):
    def get_form_kwargs(self, index):
        kwargs = super(MyFormSet, self).get_form_kwargs(index)
        kwargs.update({'parent': self.instance})
        return kwargs
```

Kreiraj objekat Form i pop atribut pre poziva super():

```py
class MyForm(forms.ModelForm):

    def init (self, *args, **kwargs):
        parent = kwargs.pop('parent')
        super(MyForm, self). init (*args, **kwargs)
        # radi šta treba sa parent
```

Postavi u inline admin:

```py
class MyModelInline(admin.StackedInline):
    model = MyModel
    fields = ('my_fields', )
    form = MyFrom
    formset = MyFormSet
```

[Sadržaj](00_sadrzaj.md)

**Odgovor_3**:

AdminModel ima metod `get_formsets`. On prima objekat i vraća formsetove. Mislim da možete dodati neki info o parent objektu u formset klasu i kasnije koristiti formsetovu `__init__` metodu.

[Sadržaj](00_sadrzaj.md)

**Odgovor_4**:

Ako je polje forme od intersa konstruisano iz polja modela, možete koristiti sledeći kod za primenu prilagođenog ponašanja:

```py
class MyChildInline(admin.TabularInline):
    model = MyChildModel
    extra = 1
    
    def get_formset(self, request, parent=None, **kwargs):
    
        def formfield_callback(db_field):
            """
            Constructor of the formfield given the model field.
            """
            formfield = self.formfield_for_dbfield(db_field, request=request)
            if db_field.name == 'my_choice_field' and parent is not None:
                formfield.choices = parent.get_choices()
                return formfield
            return super(MyChildInline, self).get_formset(request, obj=obj,

        formfield_callback=formfield_callback, **kwargs)
        return result
```

[Sadržaj](00_sadrzaj.md)
