# DjangoQL šema

[Sadržaj](00_sadrzaj.md)

Šema defniniše ograničenja - šta možete uraditi sa `DjangoQL` upitom. Ako ne specificirate šemu, `DjangoQL` će vam pružiti default šemu. On će se kretati rekurzivno kroz polja modela i veze, i uključiti sve što nadje u šemi, tako da će korisnik moći da pretražuje preko svega.

Ponekad to nije ono što želite, bilo zbog db performansi bilo zbog bezbednosti. Ako bi želeli da ograničite search modela ili polja, trebalo bi da definišete šemu. Ovde je jedan primer:

```py
class UserQLSchema(DjangoQLSchema):
    exclude = (Book,)
    suggest_options = {
        Group: ['name'],
    }

    def get_fields(self, model):
        if model == Group:
            return ['name']
        return super(UserQLSchema, self).get_fields(model)

@admin.register(User)
class CustomUserAdmin(DjangoQLSearchMixin, UserAdmin):
    djangoql_schema = UserQLSchema
```

U primeru smo kreirali šemu koja radi tri stvari:

- isključuje model "Book" iz pretrage preko `exclude` opcije, takodje možete koristiti opciju `include`, koja ograničava pretragu samo po izlistanim modelima;

- ograničava raspoloživa polja za pretragu za Group model na polje `name`, u `get_fields()` metodi;

- omogućuje kompletirajuću opciju za Group names preko `suggest_options`.

  Primedba za `suggest_options`:
  - ona gleda za choices model polje parametre prvo,
  - ako nije spec., sinhrono uzima sve vrednosti za data model polja,
  - tako da bi trebali da sprečite velike querset-ove ovde.

    Ako želite da definišete prilagodjenu suggest_options, vidi ispod.

[Sadržaj](00_sadrzaj.md)
