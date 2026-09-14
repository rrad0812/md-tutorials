# Prilagodjena polja za pretragu

[Sadržaj](00_sadrzaj.md)

Dublje prilagodjenje pretrage može biti postignuto sa prilagodjenim `search` poljima. Prilagodjena `search` polja mogu biti korišćenja:

- za pretragu po annotacijama,
- za definisanje `suggest_options`, ili
- za pretragu po lookup poljima

Osim ovoga ožemo raditi definisanje pune prilagodjene search logike.

U `djangoql.schema`, DjangoQL definiše sledeća polja bazne klase koje mogu biti nasledjene za definisanje sopstvenog ponašanja:

- IntField
- FloatField
- StrFieldBoolField
- DateField
- DateTimeField
- RelationField

Ovde su primeri za česte slučajeve upotrebe:

[Sadržaj](00_sadrzaj.md)

## Prilagodjena pretraga queryseta po annotacijama

```py
from djangoql.schema import DjangoQLSchema, IntField

class UserQLSchema(DjangoQLSchema):

    def get_fields(self, model):
        fields = super(UserQLSchema, self).get_fields(model)
        if model == User:
            fields += [IntField(name='groups_count')]
            return fields

@admin.register(User)
class CustomUserAdmin(DjangoQLSearchMixin, UserAdmin):
    djangoql_schema = UserQLSchema

    def get_queryset(self, request):
        qs = super(CustomUserAdmin, self).get_queryset(request)
        return qs.annotate(groups_count=Count('groups'))
```

Prvo smo dodali "groups_count" annotaciju na queryset koji koristi Django admin u
"CustomUserAdmin.get_queryset()" metodi. On bi trebao da sadrži broj grupa kojima jedan korisnik pripada.

Kada naš queryset sada podigne tu kolonu, možemo filtrirati po njoj. Samo treba da bude uključena u šemu.

U "UserQLSchema.get_fields()" definišemo prilagodjeno integer search polje za "User" model. Njegovo ime treba da je isto sa imenom kolone u queryset-u.

[Sadržaj](00_sadrzaj.md)

## Prilagođene suggest_options

```py
from djangoql.schema import DjangoQLSchema, StrField

class GroupNameField(StrField):
    model = Groupname = 'name'
    suggest_options = True

    def get_options(self, search):
        return super(GroupNameField, self) \
    
    .get_options(search) \
        .annotate(users_count=Count('user'))\
            .order_by('-users_count')

class UserQLSchema(DjangoQLSchema):def get_fields(self, model):
    if model == Group:
        return ['id', GroupNameField()]
    return super(UserQLSchema, self).get_fields(model)

@admin.register(User)
class CustomUserAdmin(DjangoQLSearchMixin, UserAdmin):

djangoql_schema = UserQLSchema
```

U ovom primeru definisali smo prilagođeno GroupNameField koje sortira sugestije za grupu imena po popularnosti, (broj korisnika u grupi) umesto default alphabetical sortinga.

[Sadržaj](00_sadrzaj.md)

## Prilagodjeni search lookup

DjangoQL bazno pruža dva osnovna metoda koje možete prepisati:

- get_lookup_name() i
- get_lookup_value(value):

```py
class UserDateJoinedYear(IntField):name = 'date_joined_year'

    def get_lookup_name(self): 
        return 'date_joined year'

class UserQLSchema(DjangoQLSchema):
    def get_fields(self, model):
        fields = super(UserQLSchema, self).get_fields(model)
        if model == User:
            fields += [UserDateJoinedYear()]return fields

@admin.register(User)
class CustomUserAdmin(DjangoQLSearchMixin, UserAdmin):
    djangoql_schema =     UserQLSchema
```

U ovom primeru mi smo definisali prilagodjeno "date_joined_year" search polje za korisnike. I koristili smo ugradjeni Django year filter opciju u `get_lookup_name()` za filtriranje samo po "year".

Slično možete koristiti `get_lookup_value` hook za promenu search vrednosti pre nego se koristi u filteru.

[Sadržaj](00_sadrzaj.md)
