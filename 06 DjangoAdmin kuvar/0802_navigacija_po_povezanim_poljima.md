
# Navigacija po povezanim poljima

[Sadržaj](00_sadrzaj.md)

Ponekad može biti korisno brzo kretanje između objekata. Nakon pokušaja da naučimo osoblje za podršku da filtrira pomoću parametara URL-a, konačno smo odustali i stvorili dva jednostavna dekoratora.

## to_parent_link

Napravite vezu do stranice povezanog modela:

```py
def admin_change_url(obj):
    app_label = obj._meta.app_label
    model_name = obj._meta.model. name .lower()
    return reverse('admin:{}_{}_change'.format( app_label, model_name ), args=(obj.pk,))

def to_parent_link(attr, short_description, empty_description="-"):
    """
    Dekorator za render linka na povezani admin detalj model
    attr (str) : Ime povezanog polja.
    short_description (str): Opis polja.
    empty_description (str): Vrednost za prikazivanje ako je povezanoi polje None.
    Treba da vrati link tekst.
    Korišćenje:
    @to_parent_link('credit_card', ('Credit Card'))
    def credit_card_link(self, credit_card):

    return credit_card.name
    """
    def wrap(func):

        def field_func(self, obj):
            related_obj = getattr(obj, attr)
            if related_obj is None:
                return empty_description
            url = admin_change_url(related_obj)
            return format_html('<a href="{}">{}</a>', url, func(self, related_obj)

        field_func.short_description = short_description
        field_func.allow_tags = True
        return field_func

    return wrap

Dekorator će prikazati vezu ( <a href="..."> ... </a> ) do povezanog modela. Ako na primer želimo da dodamo vezu sa svakog proizvoda na stranicu sa detaljima o kategoriji, koristimo dekorater ovako:

@admin.register(models.Product)
class ProductAdmin(admin.ModelAdmin):
    list_display = ( 'id', 'name', 'category_link', )
    admin_select_related = ('category',)
    
    @to_parent_link('category', _('Category'))
    def category_link(self, category):
        return category
```

[Sadržaj](00_sadrzaj.md)

## to_child_link

Komplikovanije veze poput „svi proizvodi kategorije“ zahtevaju drugačiji pristup. Napravili smo dekorator koji prihvata niz upita i povezuje se na prikaz liste povezanog modela:

```py
def admin_changelist_url(model):
    app_label = model._meta.app_label
    model_name = model. name .lower()
    return reverse('admin:{}_{}_changelist'.format(app_label, model_name))

def to_child_link(attr, short_description, empty_description='-',   
    query_string=None):
    """
    Decorator used for rendering a link to the list display of a related model.
    attr (str): Name of the related field.
    short_description (str): Field display name.
    empty_description (str): Value to display if the related field is None.
    query_string (function): Optional callback for adding a query string to the link.
    Receives the object and should return a query string.
    The wrapped method receives the related object and should return the
    linktext.
    Usage:
    @admin_changelist_link('credit_card', _('Credit Card'))
    def credit_card_link(self, credit_card):

    return credit_card.name
    """
    def wrap(func):
        def field_func(self, obj):
            related_obj = getattr(obj, attr)
            if related_obj is None:
                return empty_description
            url = admin_changelist_url(related_obj.model)
            if query_string:
                url += '?' + query_string(obj)
                return format_html('<a href="{}">{}</a>', url, func(self, related_obj))

        field_func.short_description = short_description
        field_func.allow_tags = True
        return field_func
    return wrap
```

Da bismo dodali vezu iz kategorije u svoje proizvode, u CategoryAdmin uradimo sledeće:

```py
@admin.register(models.Category)
class CategoryAdmin(admin.ModelAdmin):
    list_display = ('id', 'name', 'products_link', )
    @to_child_link('products', _('Products'), query_string=lambda c:
    'category_id={}'.format(c.pk))
    def products_link(self, products):
        return _('Products')
```

Budite oprezni sa argumentom proizvoda. Veoma je primamljivo učiniti nešto poput ovog:

```py
@to_child_link('products',('Products'),
    query_string=lambda c:'category_id={}'.format(c.pk))

def products_link(self, products):
    “ Dont do that! “
    return 'see {} products'.format(products.count())
```

Gornji primer rezultiraće dodatnim upitima.

[Sadržaj](00_sadrzaj.md)
