# readonly_fields

[Sadržaj](00_sadrzaj.md)

Na stranici sa detaljima, Django kreira element za uređivanje za svako polje. Tekstualna i numerička polja prikazivaće se kao redovno polje za unos. Polja izbora i polja stranog ključa prikazivaće se kao element `<select>`.

Da bi prikazao polje za potvrdu, Django mora učiniti sledeće:

- Preuzeti ceo povezani model i njegove opise.
- Prikazati celu listu opcija, jedna opcija za svaku povezanu instancu.

Uobičajeni scenario koji se često previdi je strani ključ za model `User`. Kada imate 100 korisnika, možda nećete primetiti opterećenje, ali šta se dešava kada imate 100.000 korisnika? Stranica sa detaljima preuzima celu tabelu korisnika, a lista opcija će rezultirajući HTML učiniti ogromnim. Plaćamo dva puta, prvo za
skeniranje kompletne tabele, a zatim za preuzimanje html datoteke. A da se i ne spominje memorija potrebna za generisanje html datoteke.

Imati element za odabir sa opcijama od 100.000 nije baš upotrebljivo. Najlakši način da sprečite Django da prikazuje polje kao element `<select>` je da ga označite kao `readonly_fields`:

```py
@admin.register(SomeModel)
def SomeModelAdmin(admin.ModelAdmin):
    readonly_fields = ( 'user', )
```

Ovo će prikazati opis povezanog modela, bez mogućnosti da ga promenite u adminu. Druga opcija koja sprečava Django da prikazuje okvir za odabir je označavanje polja kao `raw_id_fields`.

```py
@admin.register(SomeModel)
def SomeModelAdmin(admin.ModelAdmin):
    raw_id_fields = ('user', )
```

Koristeći `raw_id_fields`, Django će prikazati poseban vidžet koji prikazuje ID vrednosti i opciju za otvaranje liste svih vrednosti u iskačućem prozoru. Ova opcija je veoma korisna kada želite da izmenite vrednost stranog ključa.

[Sadržaj](00_sadrzaj.md)
