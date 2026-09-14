
# Uvod u AlpineJS

[Sadržaj](00_Sadržaj.md)

## 1 Počnimo

Napravite praznu HTML datoteku negde na računaru sa imenom kao što je: "i-love-alpine.html"

Koristeći uređivač teksta, popunite datoteku ovim sadržajem:

```html
<html>
<head>
    <script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
</head>
<body>
    <h1 x-data="{ message: 'I ❤️ Alpine' }" x-text="message"></h1>
</body>
</html>
```

Otvorite datoteku u veb pregledaču, ako vidite "I ❤️ Alpine", spremni ste za igru!

Hajde da pogledamo tri praktična primera kao osnovu za učenje osnova AlpineJS. Do kraja ove vežbe, trebalo bi da budete više nego opremljeni da počnete sami da gradite stvari. Hajde da krenemo.

- Izgradnja brojača
- Kreiranje padajućeg menija
- Kreiranje unosa za pretragu

[Sadržaj](00_Sadržaj.md)

### 1.1 Izgradnja brojača

Počnimo sa jednostavnom komponentom "brojač" kako bismo demonstrirali osnove praćenja stanja i događaja u AlpineJS, dve osnovne karakteristike.

Umetnite sledeće u `<body>` oznaku:

```html
<div x-data="{ count: 0 }">
    <button x-on:click="count++">Increment</button>
    <span x-text="count"></span>
</div>
```

Sada, možete videti da smo, sa 2 linije AlpineJSkoda umetnuta u ovaj HTML, kreirali interaktivnu komponentu "brojač".

Hajde da ukratko prođemo kroz kod da vidimo šta se dešava:

**Deklarisanje podataka**:

```html
<div x-data="{ count: 0 }">
```

Sve u AlpineJS počinje direktivom `x-data`. Unutar `x-data`, u običnom JavaScript-u, deklarišete objekat podataka - "promenljiva stanja" koji će AlpineJSpratiti.

Svako svojstvo unutar ovog objekta biće dostupno drugim direktivama unutar ovog HTML elementa. Pored toga, kada se jedno od ovih svojstava promeni, promeniće se i sve što se na njega oslanja.

> [!Note]
> `x-data` je potreban na roditeljskom elementu da bi većina AlpineJS direktiva funkcionisale.

Hajde da pogledamo `x-on` kako može da pristupi i izmeni `count` svojstvo odozgo:

**Osluškivanje događaja**:

```html
<button x-on:click="count++">Increment</button>
```

`x-on` je direktiva koju možete koristiti za slušanje bilo kog događaja na elementu. U ovom slučaju slušamo `click` događaj, tako da naš kod izgleda kao `x-on:click`.

Možete osluškivati i druge događaje koje želite. Na primer, osluškivanje događaja `mouseenter` bi izgledalo ovako `x-on:mouseenter`.

Kada se desi `click` događaj, AlpineJS će pozvati pridruženi JavaScript izraz, `count++` u našem slučaju. Kao što vidite, imamo direktan pristup podacima deklarisanim u `x-data` izrazu.

> [!Note]
> Često ćete videti `@` umesto `x-on:`. Ovo je kraća, prijateljskija sintaksa koju mnogi preferiraju. Od sada će ova dokumentacija verovatno koristiti `@` umesto `x-on:`.

**Reagovanje na promene**:

```html
<span x-text="count"></span>
```

`x-text` je AlpineJS direktiva koju možete koristiti da podesite tekstualni sadržaj elementa kao rezultat JavaSkript izraza.

U ovom slučaju, kažemo AlpineJS da uvek osigura da sadržaj ove `span` oznake odražava vrednost objekta `count`.

U slučaju da nije jasno, `x-text`, kao i većina direktiva, prihvata običan JavaScript izraz kao argument. Na primer, umesto toga možete podesiti njegov sadržaj sa:

```html
<span x-text="count * 2">
```

i tekstualni sadržaj `span` će sada uvek biti 2 puta veći od vrednosti `count`.

[Sadržaj](00_Sadržaj.md)

### 1.2 Kreiranje padajućeg menija

Sada kada smo videli neke osnovne funkcionalnosti, hajde da nastavimo i pogledamo važnu direktivu u AlpineJS: `x-show`, izgradnjom izmišljene komponente "padajućeg menija".

Umetnite sledeći kod u `<body>` oznaku:

```html
<div x-data="{ open: false }">
    <button @click="open = !open">Toggle</button>
    <div x-show="open" @click.outside="open = false">Contents...</div>
</div>
```

Ako učitate ovu komponentu, trebalo bi da vidite da je "Contents..." podrazumevano skriven. Možete da ga uključite/isključite na stranici klikom na dugme "Toogle".

Direktive `x-data` and `x-on` bi vam trebalo da budu poznate iz prethodnog primera, pa ćemo preskočiti ta objašnjenja.

**Prebacivanje elemenata**:

```html
<div x-show="open" ...>Contents...</div>
```

`x-show` je izuzetno moćna direktiva u AlpineJS koja se može koristiti za prikazivanje i skrivanje bloka HTML-a na stranici na osnovu rezultata JavaScript izraza, u našem slučaju: `open`.

**Osluškivanje eksternog `click`**:

```html
<div ... @click.outside="open = false">Contents...</div>
```

Primetićete nešto novo u ovom primeru: `.outside`. Mnoge direktive u AlpineJS prihvataju "modifikatore" koji su lančano povezani na kraju direktive i odvojeni su tačkama.

U ovom slučaju, `.outside` govori AlpineJS da umesto da sluša klik unutar `<div>`, da sluša klik samo ako se desi van `<div>`.

Ovo je pomoćna funkcija ugrađena u AlpineJS jer je to uobičajena potreba i njena ručna implementacija je dosadna i složena.

[Sadržaj](00_Sadržaj.md)

### 1.3 Kreiranje unosa za pretragu

Hajde sada da napravimo složeniju komponentu i uvedemo nekoliko drugih direktiva i obrazaca.

Umetnite sledeći kod u `<body>` oznaku:

```html
<div
    x-data="{
        search: '',
        items: ['foo', 'bar', 'baz'],
        get filteredItems() {
            return this.items.filter(
                i => i.startsWith(this.search)
            )
        }
    }"
>
    <input x-model="search" placeholder="Search...">
    <ul>
        <template x-for="item in filteredItems" :key="item">
            <li x-text="item"></li>
        </template>
    </ul>
</div>
```

```html
fu
bar
baz
```

Podrazumevano, sve "stavke" (foo, bar i baz) biće prikazane na stranici, ali ih možete filtrirati kucanjem u polje za unos teksta. Dok kucate, lista stavki će se menjati kako bi odražavala ono što tražite.

Sada se ovde dosta toga dešava, pa hajde da prođemo kroz ovaj isečak koda deo po deo.

**Višeredno formatiranje `x-data` direktive**:

Prvo što bih želeo da istaknem jeste da `x-data` sada ima mnogo više toga u sebi nego ranije. Da bismo olakšali pisanje i čitanje, podelili smo ga u više redova u našem HTML-u. Ovo je potpuno opciono i malo kasnije ćemo više govoriti o tome kako da u potpunosti izbegnemo ovaj problem, ali za sada ćemo sav ovaj JavaScript zadržati direktno u HTML-u.

**Vezivanje sa ulaznim elementima**:

```html
<input x-model="search" placeholder="Search...">
```

Primetićete novu direktivu koju još nismo videli `x-model:`.

`x-model` se koristi za "povezivanje" vrednosti ulaznog elementa sa svojstvom podataka:

```html
<input `x-model="{ search: '', ... }">
```

u našem slučaju "search".

To znači da će se vrednost "search" promeniti svaki put kada se vrednost input elementa promeni.

`x-model` je sposoban za mnogo više od ovog jednostavnog primera.

**Izračunata svojstva korišćenjem gettera**:

Sledeće na šta bih želeo da skrenem vašu pažnju jesu svojstva `items` i `filteredItems` iz `x-data` direktive.

```js
{
    ...
    items: ['foo', 'bar', 'baz'],
    get filteredItems() {
        return this.items.filter(
            i => i.startsWith(this.search)
        )
    }
}
```

Svojstvo `items` bi trebalo da bude samo po sebi jasno. Ovde postavljamo vrednost `items` na JavaScript niz od 3 različite stavke ("foo", "bar" i "baz").

Zanimljiv deo ovog isečka je `filteredItems` svojstvo.

Prefiks `get` za ovo svojstvo `filteredItems` označio je svojstvo tipa "getter" u ovom objektu. To znači da možemo pristupiti `filteredItems` kao da je u pitanju normalno svojstvo u našem objektu podataka, ali kada to učinimo, JavaScript će proceniti datu funkciju "ispod haube" i vratiti rezultat.

Potpuno je prihvatljivo odustati od `get` i jednostavno napraviti `this` kao metodu koju možete pozvati iz šablona, ali neki preferiraju lepšu sintaksu `get` metode.

Sada da pogledamo unutar `filteredItems` gettera i uverimo se da razumemo šta se tamo dešava:

```js
return this.items.filter(
    i => i.startsWith(this.search)
)
```

Ovo je sve običan JavaScript. Prvo dobijamo niz elemenata (foo, bar i baz) i filtriramo ih koristeći dati povratni poziv:

```js
i => i.startsWith(this.search).
```

Prenošenjem ovog povratnog poziva na filter, govorimo JavaScript-u da vrati samo stavke koje počinju stringom: `this.search`, koji će, kao što smo videli sa `x-model` uvek odražavati vrednost unosa.

Možda ćete primetiti da do sada nismo morali da koristimo `this.` za referenciranje svojstava. Međutim, pošto radimo direktno unutar `x-data` objekta, moramo referencirati sva svojstva koristeći `this.[property]` umesto jednostavno `[property]`.

Pošto je AlpineJS "reaktivni" frejmvork, svaki put kada se vrednost `this.search` promeni, delovi šablona koji se koriste `filteredItems` će se automatski ažurirati.

**Elementi petlje**:

Sada kada razumemo deo sa podacima naše komponente, hajde da razumemo šta se dešava u šablonu što nam omogućava da se krećemo kroz `filteredItems` stranicu.

```html
<ul>
    <template x-for="item in filteredItems">
        <li x-text="item"></li>
    </template>
</ul>
```

Prvo što treba primetiti ovde je `x-for` AlpineJS direktiva. `x-for` izrazi imaju sledeći oblik:
`[item] in [items]` gde je [items] bilo koji niz podataka, a [item] je ime promenljive koja će biti dodeljena iteraciji unutar petlje.

Takođe primetite da je `x-for` deklarisano na `<template>` elementu, a ne direktno na `<li>`. Ovo je uslov korišćenja `x-for`. Omogućava AlpineJS da iskoristi postojeće ponašanje `<template>` oznaka u pregledaču u svoju korist.

Sada će se svaki element unutar `<template>` oznake ponoviti za svaku stavku unutra `filteredItems` i svi izrazi izračunati unutar petlje imaće direktan pristup promenljivoj iteracije ( `item` u ovom slučaju ).

[Sadržaj](00_Sadržaj.md)

### 1.4 Rezime

Ako ste stigli dovde, upoznali ste se sa sledećim direktivama u programu Alpine:

- `x-data`
- `x-on`
- `x-text`
- `x-show`
- `x-model`
- `x-for`

To je odličan početak, međutim, postoji još mnogo direktiva u koje treba zaroniti zube. Najbolji način da usvojite AlpineJS jeste da pročitate ovu dokumentaciju. Nema potrebe da češljate svaku reč, ali ako barem prelistate svaku stranicu, bićete mnogo efikasniji kada koristite AlpineJS.

[Sadržaj](00_Sadržaj.md)

