
# Uvod u AlpineJS

## 4 Magije

[Sadržaj](00_Sadržaj.md)

### 4.1 $el

`$el` je magično svojstvo koje se može koristiti za preuzimanje trenutnog DOM čvora.

```html
<button
    @click="$el.innerHTML = 'Hello World!'">Replace me with "Hello World!"
</button>
```

[Sadržaj](00_Sadržaj.md)

### 4.2 $refs

`$refs` je magično svojstvo koje se može koristiti za preuzimanje DOM elemenata označenih sa `x-ref` unutar komponente. Ovo je korisno kada treba ručno da manipulišete DOM elementima. Često se koristi kao sažetija, ograničenija alternativa za `document.querySelector`.

```html
<button @click="$refs.text.remove()">Remove Text</button>
 
<span x-ref="text">Hello !</span>     
```

Sada, kada `<button>` se pritisne `<span>` biće uklonjeno.

**Ograničenja**:

U V2 je bilo moguće dinamički se povezati $refssa elementima, kao što je prikazano ispod:

```html
<template x-for="item in items" :key="item.id">
    <div :x-ref="item.name">
    some content ...
    </div>
</template>
```

Međutim, u V3, `$refs` može se pristupiti samo za elemente koji su kreirani statički. Dakle, za gornji primer: ako ste očekivali da vrednost `item.name` unutar `$refs` bude nešto poput Batteries, trebalo bi da budete svesni da `$refs` će zapravo sadržati literalni string 'item.name', a ne Batteries.

[Sadržaj](00_Sadržaj.md)

### 4.3 $store

Možete koristiti `$store` da biste lako pristupili globalnim AlpineJS prodavnicama registrovanim pomoću `Alpine.store(...)`. Na primer:

```html
<button x-data @click="$store.darkMode.toggle()">Toggle Dark Mode</button>
 
...
 
<div x-data :class="$store.darkMode.on && 'bg-black'">
    ...
</div>
 
 
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.store('darkMode', {
            on: false,
 
            toggle() {
                this.on = ! this.on
            }
        })
    })
</script>
```

S obzirom da smo registrovali `darkMode` prodavnicu i postavili je na "false", kada se `<button>` pritisne, on biće "true" i boja pozadine stranice će se promeniti u crnu.

**Prodavnice sa jednom vrednošću**:

Ako vam nije potreban ceo objekat za skladište, možete postaviti i koristiti bilo koju vrstu podataka kao skladište.

Evo primera odozgo, ali koristeći ga jednostavnije kao bulovu vrednost:

```html
<button
    x-data @click="$store.darkMode = ! $store.darkMode">Toggle Dark Mode
</button>

...

<div x-data :class="$store.darkMode && 'bg-black'">
    ...
</div>

<script>
    document.addEventListener('alpine:init', () => {
        Alpine.store('darkMode', false)
    })
</script>
```

[Sadržaj](00_Sadržaj.md)

### 4.4 $watch

Možete "pratiti" svojstvo komponente koristeći `$watch` magičnu metodu. Na primer:

```html
<div x-data="{ open: false }" x-init="$watch('open', value => console.log(value))">
    <button @click="open = ! open">Toggle Open</button>
</div>
```

U gornjem primeru, kada se dugme pritisne i `open` promeni, aktiviraće se dati povratni poziv i `console.log` nova vrednost.

Možete pratiti duboko ugnežđena svojstva koristeći notaciju "tačka":

```html
<div x-data="{ foo: { bar: 'baz' }}" x-init="$watch('foo.bar', value => console.log(value))">
    <button @click="foo.bar = 'bob'">Toggle Open</button>
</div>
```

Kada `<button>` se pritisne, foo.bar biće podešeno na "bob" i "bob" će biti zabeleženo u konzoli.

**Dobijanje "stare" vrednosti**:

`$watch` prati prethodnu vrednost svojstva koje se prati. Možete mu pristupiti koristeći opcioni drugi argument povratnog poziva na sledeći način:

```html
<div x-data="{ open: false }" x-init="$watch('open', (value, oldValue) => console.log(value, oldValue))">
    <button @click="open = ! open">Toggle Open</button>
</div>
```

**Duboko posmatranje**:

`$watch`  automatski prati promene na bilo kom nivou, ali treba imati na umu da će, kada se otkrije promena, posmatrač vratiti vrednost posmatranog svojstva, a ne vrednost podsvojstva koje se promenilo.

```html
<div x-data="{ foo: { bar: 'baz' }}" x-init="$watch('foo', (value, oldValue) => console.log(value, oldValue))">
    <button @click="foo.bar = 'bob'">Update</button>
</div>
```

Kada se `<button>` pritisne, foo.bar biće podešeno na "bob", a "{bar: 'bob'} {bar: 'baz'}“ će biti zabeleženo u konzoli (nova i stara vrednost).

> [!Note]
> Promena svojstva "nadgledanog" objekta kao sporedni efekat povratnog `$watch` poziva generisaće beskonačnu petlju i na kraju grešku.

```html
<!-- Infinite loop -->
<div x-data="{ foo: { bar: 'baz', bob: 'lob' }}" x-init="$watch('foo', value => foo.bob = foo.bar)">
    <button @click="foo.bar = 'bob'">Update</button>
</div>
```

[Sadržaj](00_Sadržaj.md)

### 4.5 $dispatch

`$dispatch` je korisna prečica za slanje događaja pregledača.

```html
<div @notify="alert('Hello World!')">
    <button @click="$dispatch('notify')">
        Notify
    </button>
</div>
```

Takođe možete proslediti podatke zajedno sa otpremljenim događajem ako želite. Ovi podaci će biti dostupni kao svojstvo `.detail` događaja:

```html
<div @notify="alert($event.detail.message)">
    <button @click="$dispatch('notify', { message: 'Hello World!' })">
        Notify
    </button>
</div>
```

Ispod haube `$dispatch` nalazi se omotač za detaljniji API: `element.dispatchEvent(new CustomEvent(...))`

**Napomena o širenju događaja**:

Obratite pažnju da, zbog mehurića događaja , kada treba da snimite događaje poslate sa čvorova koji se nalaze u istoj hijerarhiji ugnežđenja, moraćete da koristite .windowmodifikator:

Primer:

```html
<!-- Won't work -->
<div x-data>
    <span @notify="..."></span>
    <button @click="$dispatch('notify')">Notify</button>
</div>

<!-- Will work (because of .window) -->
<div x-data>
    <span @notify.window="..."></span>
    <button @click="$dispatch('notify')">Notify</button>
</div>
```

Prvi primer neće funkcionisati jer kada se `notify` pošalje, proširiće se na svog zajedničkog pretka, `div`, a ne na svog srodnika, `<span>`. Drugi primer će funkcionisati jer srodnik osluškuje `notify` na `window` nivou do kog će se prilagođeni događaj na kraju pojaviti.

**Slanje na druge komponente**:

Takođe možete iskoristiti prethodnu tehniku da bi vaše komponente međusobno komunicirale:

Primer:

```html
<div
    x-data="{ title: 'Hello' }"
    @set-title.window="title = $event.detail"
>
    <h1 x-text="title"></h1>
</div>
 
<div x-data>
    <button @click="$dispatch('set-title', 'Hello World!')">Click me</button>
</div>
<!-- When clicked, the content of the h1 will set to "Hello World!". -->
```

**Otprema u x-model**:

Takođe možete koristiti `$dispatch()` za pokretanje ažuriranja podataka za `x-model` povezivanje podataka. Na primer:

```html
<div x-data="{ title: 'Hello' }">
    <span x-model="title">
        <button @click="$dispatch('input', 'Hello World!')">Click me</button>
        <!-- After the button is pressed, `x-model` will catch the bubbling "input" event, and update title. -->
    </span>
</div>
```

Ovo otvara vrata za pravljenje prilagođenih ulaznih komponenti čija se vrednost može podesiti preko `x-model`.

**Događaji koji se mogu otkazati**:

Vraćenu vrednost možete koristiti `$dispatch` da proverite da li je događaj otkazan ili ne. Ovo je korisno kada želite da sprečite podrazumevano ponašanje akcije.

```html
<div x-data x-on:open="$event.preventDefault()">
    <div x-data="{ open: false }">
        <button @click="if($dispatch('open')){ open = true; }">Click me</button>
        <!-- When the button is pressed an event is dispatched and only if the result is truthy (not prevented by any handler) the content will be shown. -->
 
        <div x-show="open">
            <h1>Hello</h1>
        </div>
    </div>
</div>
```

Ovo bi moglo biti korisno kada želite da sprečite otvaranje/zatvaranje modalnog prozora ili nečeg sličnog korišćenjem obrađivača događaja.

**Opcije prepisivanja**:

Možete koristiti treći parametar `$dispatch` da biste prepisali podrazumevane opcije događaja. Na primer, možete podesiti `bubbles` na false:

```html
<!-- 🚫 Won't work because the event is being listened on the parent element -->
<div x-data="{ title: 'Hello' }" x-on:update-title="title = $event.detail">
    <button @click="$dispatch('update-title', 'Hello World!', {bubbles: false})">Click me</button>
</div>
 
<!-- ✅ Will work because the event is being listened on the same element -->
<div x-data="{ title: 'Hello' }">
    <button x-on:update-title="title = $event.detail" @click="$dispatch('update-title', 'Hello World!', {bubbles: false})">Click me</button>
</div>
```

Ovo je korisno kada želite da sprečite da se događaj širi do roditeljskih elemenata.

[Sadržaj](00_Sadržaj.md)

### 4.6 $nextTick

`$nextTick` je magično svojstvo koje vam omogućava da izvršite dati izraz samo NAKON što je Alpine izvršio svoja reaktivna ažuriranja DOM-a. Ovo je korisno kada želite da interagujete sa DOM stanjem NAKON što se prikažu sva ažuriranja podataka koja ste napravili.

```html
<div x-data="{ title: 'Hello' }">
    <button
        @click="
            title = 'Hello World!';
            $nextTick(() => { console.log($el.innerText) });
        "
        x-text="title"
    ></button>
</div>
```

U gornjem primeru, umesto da se u konzolu evidentira „Hello“, evidentiraće se „Hello World!“ jer `$nextTick` je korišćeno da se sačeka dok Alpine ne završi ažuriranje DOM-a.

**Obećanja**:

`$nextTick` vraća obećanje, omogućavajući upotrebu za `$nextTick` pauziranje asinhrone funkcije dok se ne završe čekajuća ažuriranja DOM-a. Kada se koristi na ovaj način, `$nextTick` takođe ne zahteva prosleđivanje argumenta.

```html
<div x-data="{ title: 'Hello' }">
    <button
        @click="
            title = 'Hello World!';
            await $nextTick();
            console.log($el.innerText);
        "
        x-text="title"
    ></button>
</div>
```

[Sadržaj](00_Sadržaj.md)

### 4.7 $root

`$root` је магично својство које се може користити за преузимање коренског елемента било које Alpine компоненте. Другим речима, најближи елемент на DOM стаблу који садржи `x-data`.

```html
<div x-data data-message="Hello World!">
    <button @click="alert($root.dataset.message)">Say Hi</button>
</div>
```

### 4.8 $data

`$data` je magično svojstvo koje vam daje pristup trenutnom obimu podataka usluge Alpine (obično ga pruža x-data).

Većinu vremena, možete direktno pristupiti Alpine podacima unutar izraza. Na primer,

```js
x-data="{ message: 'Hello Caleb!' }"
```

će vam omogućiti da radite stvari poput `x-text="message"`.

Međutim, ponekad je korisno imati stvarni objekat koji obuhvata ceo opseg koji možete proslediti drugim funkcijama:

```html
<div x-data="{ greeting: 'Hello' }">
    <div x-data="{ name: 'Caleb' }">
        <button @click="sayHello($data)">Say Hello</button>
    </div>
</div>
 
<script>
    function sayHello({ greeting, name }) {
        alert(greeting + ' ' + name + '!')
    }
</script>
```

Sada kada se dugme pritisne, pregledač će upozoriti sa "Hello Caleb!" jer mu je prosleđen objekat podataka koji je sadržao ceo Alpine opseg izraza koji ga je pozvao ( @click="...").

Većini aplikacija neće biti potrebno ovo magično svojstvo, ali može biti veoma korisno za dublje, komplikovanije Alpine uslužne programe.

### 4.9 $id

`$id` je magično svojstvo koje se može koristiti za generisanje ID-a elementa i osiguravanje da neće biti u sukobu sa drugim ID-ovima istog imena na istoj stranici.

Ovaj uslužni program je izuzetno koristan pri kreiranju komponenti koje se mogu ponovo koristiti (verovatno u šablonu za pozadinu) i koje se mogu pojaviti više puta na stranici i koje koriste atribute ID-a.

Stvari poput ulaznih komponenti, modalnih prozora, lista itd. će imati koristi od ovog alata.

**Osnovna upotreba**:

Pretpostavimo da imate dva ulazna elementa na stranici i želite da imaju jedinstveni ID jedan od drugog, možete učiniti sledeće:

```html
<input type="text" :id="$id('text-input')">
<!-- id="text-input-1" -->
 
<input type="text" :id="$id('text-input')">
<!-- id="text-input-2" -->      
```

Kao što vidite, `$id` prima string i izbacuje dodatni sufiks koji je jedinstven na stranici.

**Grupisanje sa x-id-om**:

Recimo sada da želite da imate ista dva ulazna elementa, ali ovaj put želite `<label>` elemente za svaki od njih.

Ovo predstavlja problem, sada morate biti u mogućnosti da dva puta referencirate isti ID. Jedan za atribut `<label>` `for`, a drugi za `id` na ulazu.

Evo načina na koji biste mogli da razmislite da to postignete i koji je potpuno validan:

```html
<div x-data="{ id: $id('text-input') }">
    <label :for="id"> <!-- "text-input-1" -->
    <input type="text" :id="id"> <!-- "text-input-1" -->
</div>
 
<div x-data="{ id: $id('text-input') }">
    <label :for="id"> <!-- "text-input-2" -->
    <input type="text" :id="id"> <!-- "text-input-2" -->
</div> 
```

Ovaj pristup je u redu, međutim, potreba za imenovanjem i čuvanjem ID-a u opsegu vaše komponente deluje glomazno.

Da biste isti zadatak obavili na fleksibilniji način, možete koristiti Alpine-ovu x-iddirektivu za deklarisanje „opsega identifikacije“ za skup identifikatora:

```html
<div x-id="['text-input']">
    <label :for="$id('text-input')"> <!-- "text-input-1" -->
    <input type="text" :id="$id('text-input')"> <!-- "text-input-1" -->
</div>
 
<div x-id="['text-input']">
    <label :for="$id('text-input')"> <!-- "text-input-2" -->
    <input type="text" :id="$id('text-input')"> <!-- "text-input-2" -->
</div>
```

Kao što vidite, x-idprihvata niz imena ID-ova. Sada $id()će svaka upotreba unutar tog opsega koristiti isti ID. Zamislite ih kao „grupe ID-ova“.

**Ugnežđavanje**:

Kao što ste možda i pretpostavili, ove grupe možete slobodno ugnezditi x-id, ovako:

```html
<div x-id="['text-input']">
    <label :for="$id('text-input')"> <!-- "text-input-1" -->
    <input type="text" :id="$id('text-input')"> <!-- "text-input-1" -->
 
    <div x-id="['text-input']">
        <label :for="$id('text-input')"> <!-- "text-input-2" -->
        <input type="text" :id="$id('text-input')"> <!-- "text-input-2" -->
    </div>
</div>
```  

**Ključ ID-ovi (za petlju)**:

Ponekad je korisno navesti dodatni sufiks na kraju ID-a radi njegove identifikacije unutar petlje.

Za ovo, `$id()` prihvata opcioni drugi parametar koji će biti dodat kao sufiks na kraj generisanog ID-a.

Uobičajeni primer ove potrebe je nešto poput komponente liste koja koristi aria-activedescendant atribut da bi rekla pomoćnim tehnologijama koji je element "aktivan" na listi:

```html
<ul
    x-id="['list-item']"
    :aria-activedescendant="$id('list-item', activeItem.id)"
>
    <template x-for="item in items" :key="item.id">
        <li :id="$id('list-item', item.id)">...</li>
    </template>
</ul>
```

Ovo je nepotpun primer liste, ali bi ipak trebalo da bude koristan za demonstraciju scenarija u kojem bi vam moglo biti potrebno da svaki ID u grupi i dalje bude jedinstven za stranicu, ali i da bude ključan unutar petlje tako da možete da referencirate pojedinačne ID-ove unutar te grupe.
