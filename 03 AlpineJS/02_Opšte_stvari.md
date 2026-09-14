
# 2 Osnovne stvari

[Sadržaj](00_Sadržaj.md)

## 2.1 Instalacija

Postoje 2 načina da uključite Alpine u ​​svoj projekat:

- Uključivanje iz `<script>` oznake
- Uvoz kao modul

Oba su sasvim validna. Sve zavisi od potreba projekta i ukusa programera.

**Iz `<script>` oznake**:

Ovo je ubedljivo najjednostavniji način da počnete sa Alpine-om. Uključite sledeći `<script>` tag u zaglavlje vaše HTML stranice.

```html
<html>
    <head>
        ...
        <script defer src="<https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
    </head>
    ...
</html>
```

Ne zaboravite atribut `defer` u `<script>` oznaci.

Obratite pažnju na @3.x.xu datom CDN linku. Ovo će preuzeti najnoviju verziju Alpine verzije 3. Radi stabilnosti u produkciji, preporučuje se da čvrsto kodirate najnoviju verziju u CDN linku.

```html
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.15.12/dist/cdn.min.js"></script>
```

To je to! AlpineJS je sada dostupan za upotrebu unutar vaše stranice.

Imajte na umu da ćete i dalje morati da definišete komponentu sa `x-data` da bi svi atributi Alpine.JS radili. Više informacija potražite na <https://github.com/alpinejs/alpine/discussions/3805>.

**Kao modul**:

Ako više volite robusniji pristup, možete instalirati Alpine putem NPM-a i uvesti ga u paket.

Pokrenite sledeću komandu da biste je instalirali.

```sh
npm install alpinejs
```

Sada uvezite AlpineJS u ​​vaš paket i inicijalizujte ga ovako:

```js
import Alpine from 'alpinejs'

window.Alpine = Alpine

Alpine.start()
```

Ovaj `window.Alpine = Alpine` deo je opcionalan, ali je lepo imati ga zbog slobode i fleksibilnosti. Na primer, kada se petljate sa AlpineJS iz devtools-a.

Ako ste uvezli Alpine u ​​paket, morate se uveriti da registrujete bilo koji kod proširenja između koda kada uvozite AlpineJS globalni objekat i kada inicijalizujete Alpine pozivanjem `Alpine.start()`. Uverite se da se `Alpine.start()` poziva samo jednom po stranici. Višestruko pozivanje će rezultirati time da se više "instanci" Alpine-a pokreće istovremeno.

[Sadržaj](00_Sadržaj.md)

## 2.2 Stanje

Stanje (JavaScript podaci čije promene AlpineJS prati) je u srži svega što radite u AlpineJS. Možete da obezbedite lokalne podatke delu HTML-a ili da ih učinite globalno dostupnim za upotrebu bilo gde na stranici koristeći `x-data` ili `Alpine.store()` respektivno.

**Lokalno stanje**

AlpineJS vam omogućava da deklarišete stanje HTML bloka u jednom `x-data` atributu bez ostavljanja sopstvenih oznaka.

Evo osnovnog primera:

```html
<div x-data="{ open: false }">
    ...
</div>
```

Sada će bilo koja druga AlpineJS sintaksa na ili unutar ovog elementa moći da pristupi `open`. I kao što biste pretpostavili, kada se `open` promeni iz bilo kog razloga, sve što zavisi od njega će automatski reagovati.

***Ugnežđene promenljive stanja***:

Podaci se mogu ugnježvati u AlpineJS. Na primer, ako imate dva elementa sa priključenim AlpineJS podacima (jedan unutar drugog), možete pristupiti podacima roditelja iznutra podređenog elementa.

```html
<div x-data="{ open: false }">
    <div x-data="{ label: 'Content:' }">
        <span x-text="label"></span>
        <span x-show="open"></span>
    </div>
</div>
```

Ovo je slično određivanju opsega (scoping) u samom JavaSkriptu (kod unutar funkcije može pristupiti promenljivim deklarisanim izvan te funkcije).

Kao što ste možda pretpostavili, ako dete ima svojstvo podataka koje se podudara sa imenom svojstva roditelja, svojstvo deteta će imati prednost.

***Podaci o jednom elementu***

Iako se ovo nekima može činiti očiglednim, vredi napomenuti da se podaci iz programa AlpineJS mogu koristiti unutar istog elementa. Na primer:

```html
<button x-data="{ label: 'Click Here' }" x-text="label"></button>
```

***AlpineJS bez podataka***

Ponekad možda želite da koristite Alpine funkcionalnost, ali vam nisu potrebni nikakvi reaktivni podaci. U tim slučajevima, možete se isključiti iz potpunog prosleđivanja izraza `x-data`. Na primer:

```html
<button x-data @click="alert('I\'ve been clicked!')">Click Me</button>
```

***Podaci koji se mogu ponovo koristiti***

Kada koristite AlpineJS, možda ćete imati potrebu da ponovo koristite deo podataka i/ili odgovarajući šablon.

Ako koristite bekend frejmvork kao što su Rails ili Laravel, AlpineJS prvo preporučuje da izdvojite ceo blok HTML-a u delimični ili uključeni šablon.

Ako iz nekog razloga to nije idealno za vas ili niste u okruženju za kreiranje šablona u pozadini, AlpineJS vam omogućava da globalno registrujete i ponovo koristite deo podataka komponente koristeći `Alpine.data(...)`.

```js
Alpine.data('dropdown', () => ({
    open: false,
    toggle() {
        this.open = ! this.open
    }
}))
```

Sada kada ste registrovali podatke „padajućeg menija“, možete ih koristiti unutar svog označavanja na onoliko mesta koliko želite:

```html
<div x-data="dropdown">
    <button @click="toggle">Expand</button>
    <span x-show="open">Content...</span>
</div>
<div x-data="dropdown">
    <button @click="toggle">Expand</button>
    <span x-show="open">Some Other Content...</span>
</div>
```

**Globalno stanje**:

Ako želite da neke podatke učinite dostupnim svakoj komponenti na stranici, to možete učiniti koristeći Alpine-ovu funkciju "globalnog skladištenja" - `store`.

Možete registrovati promenljivu stanja koristeći `Alpine.store(...)` i referencirati je magičnom `$store()` metodom.

Pogledajmo jednostavan primer. Prvo ćemo registrovati prodavnicu globalno:

```js
Alpine.store('tabs', {
    current: 'first',
    items: ['first', 'second', 'third'],
})
```

Sada možemo pristupiti ili izmeniti njegove podatke sa bilo kog mesta na našoj stranici:

```html
<div x-data>
    <template x-for="tab in $store.tabs.items">
        ...
    </template>
</div>

<div x-data>
    <button @click="$store.tabs.current = 'first'">First Tab</button>
    <button @click="$store.tabs.current = 'second'">Second Tab</button>
    <button @click="$store.tabs.current = 'third'">Third Tab</button>
</div>
```

[Sadržaj](00_Sadržaj.md)

## 2.3 Šabloniranje

Alpine nudi nekoliko korisnih direktiva za manipulisanje DOM-om na veb stranici.

Hajde da ovde pokrijemo nekoliko osnovnih direktiva za šablone, ali obavezno pogledajte dostupne direktive u bočnoj traci za iscrpan spisak.

**Tekstualni sadržaj**:

Alpine olakšava kontrolu tekstualnog sadržaja elementa pomoću x-textdirektive.

```html
<div x-data="{ title: 'Start Here' }">
    <h1 x-text="title"></h1>
</div>
```

Sada će Alpine podesiti tekstualni sadržaj `<h1>` na vrednost title("Start here"). Kada se `title` promeni, promeniće se i sadržaj `<h1>`.

Kao i sve direktive u Alpine-u, možete koristiti bilo koji JavaScript izraz koji želite. Na primer:

```html
<span x-text="1 + 2"></span>

3
```

`x-text` elementa `<span>` sada sadrži zbir brojeva „1“ i „2“.

**Prebacivanje elemenata unutar šablona**:

Prebacivanje elemenata je uobičajena potreba na veb stranicama i aplikacijama. Padajući meniji, modalni prozori, dijalozi, "prikaži još", itd... su svi dobri primeri.

AlpineJS nudi direktive `x-show` i `x-if` za prebacivanje elemenata na stranici.

***x-show***

Evo jednostavne komponente za prebacivanje koja koristi `x-show`.

```html
<div x-data="{ open: false }">
    <button @click="open = ! open">Expand</button>
    <div x-show="open">
        Content...
    </div>
</div>
```

Sada će ceo `<div>` sadržaj biti prikazan i skriven na osnovu vrednosti `open`.

Ispod haube, Alpine dodaje CSS svojstvo `display: none;` elementu kada bi trebalo da bude skriven.

Ovo dobro funkcioniše u većini slučajeva, ali ponekad ćete možda želeti da potpuno dodate i uklonite element iz DOM-a. To je ono čemu `x-if` služi.

***x-if***

Evo istog prekidača od ranije, ali ovaj put se koristi x-ifumesto x-show.

```html
<div x-data="{ open: false }">
    <button @click="open = ! open">Expand</button>
    <template x-if="open">
        <div>
            Content...
        </div>
    </template>
</div>
```

Obratite pažnju da x-ifmora biti deklarisano na `<template>` oznaci. Ovo je da bi Alpine mogao da iskoristi postojeće ponašanje elementa u pregledaču `<template>` i da ga koristi kao izvor cilja `<div>` koji će biti dodat i uklonjen sa stranice.

Kada je `open` vrednost "true", Alpine će dodati `<div>` oznaci `<template>`, a ukloniti je kada je `open` vrednost "false".

**Prebacivanje sa prelazima**:

Alpine olakšava glatki prelazak između „prikazanog“ i „skrivenog“ stanja koristeći x-transitiondirektivu.

x-transition Radi samo sa x-show, ne sa x-if.

Evo, ponovo, jednostavnog primera prebacivanja, ali ovaj put sa primenjenim prelazima:

```html
<div x-data="{ open: false }">
    <button @click="open = ! open">Expands</button>
    <div x-show="open" x-transition>
        Content...
    </div>
</div>
```

Hajde da zumiramo deo šablona koji se bavi prelazima:

```html
<div x-show="open" x-transition>
```

x-transition samo po sebi će primeniti razumne podrazumevane prelaze (zatamnjenje i skaliranje) na prekidač.

Postoje dva načina za prilagođavanje ovih prelaza:

- Pomoćnici u tranziciji
- Prelazne CSS klase.

Hajde da pogledamo svaki od ovih pristupa:

***Pomoćnici u tranziciji***

Recimo da želite da produžite trajanje prelaza, to možete ručno odrediti koristeći .durationmodifikator na sledeći način:

```html
<div x-show="open" x-transition.duration.500ms>
```

Sada će prelaz trajati 500 milisekundi.

Ako želite da odredite različite vrednosti za prelaze unutra i napolje, možete koristiti x-transition:enter`and` x-transition:leave:

```html
<div
    x-show="open"
    x-transition:enter.duration.500ms
    x-transition:leave.duration.1000ms
>
```

Pored toga, možete dodati ili .opacityili .scaleda biste samo prešli na to svojstvo. Na primer:

```html
<div x-show="open" x-transition.opacity>
```

***Prelazne klase***

Ako vam je potrebna preciznija kontrola nad prelazima u vašoj aplikaciji, možete primeniti određene CSS klase u određenim fazama prelaza koristeći sledeću sintaksu (ovaj primer koristi Tailwind CSS ):

```html
<div
    x-show="open"
    x-transition:enter="transition ease-out duration-300"
    x-transition:enter-start="opacity-0 transform scale-90"
    x-transition:enter-end="opacity-100 transform scale-100"
    x-transition:leave="transition ease-in duration-300"
    x-transition:leave-start="opacity-100 transform scale-100"
    x-transition:leave-end="opacity-0 transform scale-90"
>...</div>
```

**Atribut `bind`**

Možete dodati HTML atribute kao što su `class`, `style`, `disabled`, itd... elementima u AlpineJS koristeći `x-bind` direktivu.

Evo primera dinamički vezanog `class` atributa:

```html
<button
    x-data="{ red: false }"
    x-bind:class="red ? 'bg-red' : ''"
    @click="red = ! red"
>
    Toggle Red
</button>
```

Kao prečicu, možete izostaviti i direktno x-bindkoristiti skraćenu sintaksu::

```html
<button ... :class="red ? 'bg-red' : ''">
```

Uključivanje i isključivanje klasa na osnovu podataka unutar Alpine-a je uobičajena potreba. Evo primera uključivanja/isključivanja klase koristeći Alpine-ovu classsintaksu povezivanja objekata:

(Napomena: ova sintaksa je dostupna samo za classatribute)

```html
<div x-data="{ open: true }">
    <span :class="{ 'hidden': ! open }">...</span>
</div>
```

Sada će `hidden` klasa biti dodata elementu ako je `open` `false`, i uklonjena ako je `open` `true`.

**Elementi petlje u šablonima**:

Alpine omogućava ponavljanje delova vašeg šablona na osnovu JavaSkript podataka koristeći x-fordirektivu. Evo jednostavnog primera:

```html
<div x-data="{ statuses: ['open', 'closed', 'archived'] }">
    <template x-for="status in statuses">
        <div x-text="status"></div>
    </template>
</div>
```

```sh
open
closed
archived
```

Slično kao `x-if`, `x-for` mora se primeniti na `<template>` oznaku. Interno, AlpineJS će dodati sadržaj `<template>` oznake za svaku iteraciju u petlji.

Kao što vidite, nova `status` promenljiva je dostupna u opsegu iterativnih šablona.

**Unutrašnji HTML**:

AlpineJS olakšava kontrolu HTML sadržaja elementa pomoću `x-html` direktive.

```html
<div x-data="{ title: '<h1>Start Here</h1>' }">
    <div x-html="title"></div>
</div>
```

Sada će Alpine podesiti tekstualni sadržaj elementa `<div>` pomoću `<h1>Start Here</h1>`. Kada se `title` promeni, promeniće se i sadržaj elementa `<h1>`.

> [!Note]
> Koristite samo na pouzdanom sadržaju, a nikada na sadržaju koji pružaju korisnici. Dinamičko prikazivanje HTML-a od trećih strana može lako dovesti do XSS ranjivosti.

[Sadržaj](00_Sadržaj.md)

## 2.4 Događaji

Alpine olakšava praćenje događaja u pregledaču i reagovanje na njih.

**Osluškivanje jednostavnih događaja**:

Korišćenjem `x-on`, možete slušati događaje pregledača koji se šalju na ili unutar elementa.

Evo osnovnog primera slušanja klika na dugme:

```html
<button x-on:click="console.log('clicked')">...</button>
```

Kao alternativu, možete koristiti skraćenu sintaksu događaja ako želite:. @Evo istog primera kao i pre, ali koristeći skraćenu sintaksu (koju ćemo koristiti od sada):

```html
<button @click="...">...</button>
```

Pored click, možete da slušate bilo koji događaj pregledača po imenu. Na primer: @mouseenter, @keyup, itd... su sve validne sintakse.

**Slušanje određenih tastera**:

Recimo da želite da slušate da li enterse taster pritiska unutar `<input>` elementa. AlpineJS to olakšava dodavanjem `.enter`:

```html
<input @keyup.enter="...">
```

Možete čak kombinovati modifikatore tastera da biste slušali kombinacije tastera kao što je pritiskanje `enter` dok držite `shift`:

```html
<input @keyup.shift.enter="...">
```

**Sprečavanje neizvršenja obaveza**:

Prilikom reagovanja na događaje u pregledaču, često je potrebno "sprečiti podrazumevano ponašanje" (sprečiti podrazumevano ponašanje događaja u pregledaču).

Na primer, ako želite da slušate slanje obrasca, ali da sprečite pregledač da pošalje zahtev za obrazac, možete koristiti `.prevent`:

```html
<form @submit.prevent="...">...</form>
```

Takođe možete da se prijavite `.stop` da biste postigli ekvivalent od `event.stopPropagation()`.

**Pristup objektu događaja**:

Ponekad ćete možda želeti da pristupite objektu događaja izvornog pregledača unutar sopstvenog koda. Da bi ovo olakšao, AlpineJS automatski ubrizgava `$event` magičnu promenljivu:

```html
<button @click="$event.target.remove()">Remove Me</button>
```

**Slanje prilagođenih događaja**:

Pored slušanja događaja pregledača, možete ih i otpremiti. Ovo je izuzetno korisno za komunikaciju sa drugim AlpineJS komponentama ili pokretanje događaja u alatima van samog AlpineJS.

AlpineJS otkriva magičnog pomoćnika `$dispatch` koji je za ovo pozvan:

```html
<div @foo="console.log('foo was dispatched')">
    <button @click="$dispatch('foo')"></button>
</div>
```

Kao što vidite, kada se klikne na dugme, Alpine će poslati događaj pregledača pod nazivom "foo", a naš @fooslušalac na `<div>` će ga preuzeti i reagovati na njega.

**Osluškivanje događaja na prozoru**:

Zbog prirode događaja u pregledaču, ponekad je korisno slušati događaje na objektu prozora najvišeg nivoa.

Ovo vam omogućava potpunu komunikaciju između komponenti kao u sledećem primeru:

```html
<div x-data>
    <button @click="$dispatch('foo')"></button>
</div>
<div x-data @foo.window="console.log('foo was dispatched')">...</div>
```

U gornjem primeru, ako kliknemo na dugme u prvoj komponenti, AlpineJS će poslati događaj "foo". Zbog načina na koji događaji funkcionišu u pregledaču, oni se "proširuju" kroz nadređene elemente sve do "prozora" najvišeg nivoa.

Sada, pošto u našoj drugoj komponenti slušamo "foo" na prozoru (sa `.window`), kada se klikne na dugme, ovaj slušalac će to prikupiti i zabeležiti poruku "foo was dispatched".

[Sadržaj](00_Sadržaj.md)

## 2.5 Životni ciklus

AlpineJS ima nekoliko različitih tehnika za povezivanje sa različitim delovima svog životnog ciklusa. Hajde da prođemo kroz najkorisnije da biste se upoznali sa:

**Inicijalizacija elementa**:

Još jedna izuzetno korisna karakteristika životnog ciklusa u Alpine-u je `x-init` direktiva.

`x-init` može se dodati bilo kom elementu na stranici i izvršiće bilo koji JavaScript koji pozovete unutar njega kada AlpineJS počne sa inicijalizacijom tog elementa.

```html
<button x-init="console.log('Im initing')">
```

Pored direktive, Alpine će automatski pozivati sve init() metode sačuvane na objektu podataka. Na primer:

```js
Alpine.data('dropdown', () => ({

    init() {

        // I get called before the element using this data initializes.

    }

}))
```

**Nakon promene stanja**:

AlpineJS vam omogućava da izvršite kod kada se deo podataka (stanje) promeni. Nudi dva različita API-ja za takav zadatak: `$watch` i `x-effect`.

***$watch***

```html
<div x-data="{ open: false }" x-init="$watch('open', value => console.log(value))">
```

Kao što vidite gore, `$watch` omogućava vam da se povežete sa promenama podataka pomoću tačkaste notacije. Kada se taj deo podataka promeni, AlpineJS će pozvati prosleđeni povratni poziv i proslediti mu novu vrednost, zajedno sa starom vrednošću pre promene.

***x-effect***

`x-effect` koristi isti mehanizam ispod haube kao `$watch` ali ima veoma drugačiju upotrebu.

Umesto da navedete koji ključ podataka želite da pratite, `x-effect` poziva se dati kod i inteligentno traži sve Alpine podatke koji se koriste u njemu. Sada, kada se jedan od tih podataka promeni, `x-effect` izraz će se ponovo pokrenuti.

Evo istog dela koda iz $watchprimera prepisanog pomoću `x-effect`:

```html
<div x-data="{ open: false }" x-effect="console.log(open)">
```

Sada će ovaj izraz biti pozvan odmah i ponovo pozvan svaki put kada se `open` ažurira.

Dve glavne razlike u ponašanju kod ovog pristupa su:

- Navedeni kod će se pokrenuti odmah I kada se podaci promene ( `$watch` je „lenj“ -- neće se pokrenuti dok se podaci ne promene )
- Nema saznanja o prethodnoj vrednosti. (Povratni poziv je dat za `$watch` prijem i nove I stare vrednosti )

**AlpsineJS inicijalizacija**:

***alpine:init***

Obezbeđivanje da se deo koda izvrši nakon što se Alpine učita, ali pre nego što se inicijalizuje na stranici je neophodan zadatak.

Ova udica vam omogućava da registrujete prilagođene podatke, direktive, magije itd. pre nego što AlpineJS uradi svoje na stranici.

Možete se povezati sa ovom tačkom u životnom ciklusu tako što ćete osluškivati događaj koji je AlpineJS pozvao: `alpine:init`

```js
document.addEventListener('alpine:init', () => {
    Alpine.data(...)
})
```

***alpine:initialized***

Alpine takođe nudi udicu koju možete koristiti za izvršavanje koda nakon što se završi inicijalizacija, a koja se zove `alpine:initialized`:

```js
document.addEventListener('alpine:initialized', () => {
    //
})
```

[Sadržaj](00_Sadržaj.md)
