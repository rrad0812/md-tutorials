
# Uvod u Alpine JS

## 5 Globali

### 5.1 Alpine.data

`Alpine.data(...)` pruža način za ponovnu upotrebu `x-data` konteksta unutar vaše aplikacije.

Evo jedne izmišljene dropdown komponente na primer:

```html
<div x-data="dropdown">
    <button @click="toggle">...</button>
 
    <div x-show="open">...</div>
</div>
 
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.data('dropdown', () => ({
            open: false,
 
            toggle() {
                this.open = ! this.open
            }
        }))
    })
</script>
```

Kao što vidite, izvukli smo svojstva i metode koje bismo obično definisali direktno unutra x-datau poseban objekat Alpine komponente.

**Registracija iz paketa**:

Ako ste odabrali da koristite korak izgradnje za svoj Alpine kod, trebalo bi da registrujete svoje komponente na sledeći način:

```js
import Alpine from 'alpinejs'
import dropdown from './dropdown.js'
 
Alpine.data('dropdown', dropdown)
 
Alpine.start()
```

Ovo pretpostavlja da imate datoteku dropdown.jssa sledećim sadržajem:

```js
export default () => ({
    open: false,
 
    toggle() {
        this.open = ! this.open
    }
})
```

**Početni parametri**:

Pored `Alpine.data` direktnog pozivanja na provajdere po njihovom imenu (kao što je `x-data="dropdown"`), možete ih pozivati i kao funkcije ( `x-data="dropdown()"` ). Direktnim pozivanjem kao funkcija, možete proslediti dodatne parametre koji će se koristiti prilikom kreiranja početnog objekta podataka, kao što je prikazano:

```html
<div x-data="dropdown(true)">
```

```js
Alpine.data('dropdown', (initialOpenState = false) => ({
    open: initialOpenState
}))
```

Sada možete ponovo koristiti dropdownobjekat, ali mu dodeliti različite parametre po potrebi.

**Inicijalne funkcije**:

Ako vaša komponenta sadrži init()metodu, Alpine će je automatski izvršiti pre nego što prikaže komponentu. Na primer:

```js
Alpine.data('dropdown', () => ({
    init() {
        // This code will be executed before Alpine
        // initializes the rest of the component.
    }
}))
```

**Destroy funkcije**:

Ako vaša komponenta sadrži `destroy()` metodu, Alpine će je automatski izvršiti pre čišćenja komponente.

Glavni primer za ovo je registrovanje obrađivača događaja sa drugom bibliotekom ili API-jem pregledača koji nije dostupan preko Alpine-a. Pogledajte sledeći primer koda o tome kako da koristite metodu `destroy()` za čišćenje takvog obrađivača.

```js
Alpine.data('timer', () => ({
    timer: null,
    counter: 0,
    init() {
      // Register an event handler that references the component instance
      this.timer = setInterval(() => {
        console.log('Increased counter to', ++this.counter);
      }, 1000);
    },
    destroy() {
        // Detach the handler, avoiding memory and side-effect leakage
        clearInterval(this.timer);
    },
}))
```

Primer gde se komponenta uništava je kada se koristi unutar x-if:

```html
<span x-data="{ enabled: false }">
    <button @click.prevent="enabled = !enabled">Toggle</button>
 
    <template x-if="enabled">
        <span x-data="timer" x-text="counter"></span>
    </template>
</span>   
```

**Korišćenje magičnih svojstava**:

Ako želite da pristupite magičnim metodama ili svojstvima iz objekta komponente, to možete učiniti koristeći `this` kontekst:

```js
Alpine.data('dropdown', () => ({
    open: false,
 
    init() {
        this.$watch('open', () => {...})
    }
}))
```  

**Inkapsuliranje direktiva pomoću `x-bind`**:

Ako želite da ponovo koristite više od samog objekta podataka komponente, možete da kapsulirate cele Alpine direktive šablona koristeći `x-bind`.

Sledi primer izdvajanja detalja šablona naše prethodne komponente padajućeg menija pomoću `x-bind`:

```html
<div x-data="dropdown">
    <button x-bind="trigger"></button>
 
    <div x-bind="dialogue"></div>
</div>
```

```js
Alpine.data('dropdown', () => ({
    open: false,
 
    trigger: {
        ['@click']() {
            this.open = ! this.open
        },
    },
 
    dialogue: {
        ['x-show']() {
            return this.open
        },
    },
}))
```

### 5.2 Alpine.store

Alpine nudi globalno upravljanje stanjem putem Alpine.store()API-ja.

**Registracija prodavnice**:

Možete definisati Alpine skladište unutar slušaoca `alpine:init` (u slučaju uključivanja Alpine-a putem `<script>` oznake), ILI ga možete definisati pre ručnog pozivanja `Alpine.start()` (u slučaju uvoza Alpine-a u izgradnju):

Iz oznake skripte:

```html
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

Iz paketa:

import Alpine from 'alpinejs'

```js
Alpine.store('darkMode', {
    on: false,
 
    toggle() {
        this.on = ! this.on
    }
})
 
Alpine.start()
```

**Pristup prodavnicama**:

Možete pristupiti podacima iz bilo kojeg skladišta unutar Alpine izraza koristeći `$store` magično svojstvo:

```html
<div x-data :class="$store.darkMode.on && 'bg-black'">...</div>
```

Takođe možete menjati svojstva unutar prodavnice i sve što zavisi od tih svojstava će automatski reagovati. Na primer:

```html
<button x-data @click="$store.darkMode.toggle()">Toggle Dark Mode</button>
```

Pored toga, možete pristupiti prodavnici spolja koristeći `Alpine.store()` izostavljanje drugog parametra na sledeći način:

```html
<script>
    Alpine.store('darkMode').toggle()
</script>
```

**Inicijalizacija prodavnica**:

Ako obezbedite `init()` metodu u Alpine prodavnici, ona će biti izvršena odmah nakon registracije prodavnice. Ovo je korisno za inicijalizaciju bilo kog stanja unutar prodavnice sa razumnim početnim vrednostima.

```html
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.store('darkMode', {
            init() {
                this.on = window.matchMedia('(prefers-color-scheme: dark)').matches
            },
 
            on: false,
 
            toggle() {
                this.on = ! this.on
            }
        })
    })
</script>
```

Obratite pažnju na novododati `init()` metod u gornjem primeru. Sa ovim dodatkom, onpromenljiva store će biti podešena na željenu šemu boja pregledača pre nego što Alpine prikaže bilo šta na stranici.

**Prodavnice sa jednom vrednošću**:

Ako vam nije potreban ceo objekat za skladište, možete postaviti i koristiti bilo koju vrstu podataka kao skladište.

Evo primera odozgo, ali koristeći ga jednostavnije kao bulovu vrednost:

```html
<button x-data @click="$store.darkMode = ! $store.darkMode">Toggle Dark Mode</button>

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

### 5.3 Alpine.bind

`Alpine.bind(...)` pruža način za ponovnu upotrebu `x-bind` objekata unutar vaše aplikacije.

Evo jednostavnog primera. Umesto ručnog povezivanja atributa pomoću programa Alpine:

```html
<button type="button" @click="doSomething()" :disabled="shouldDisable"></button>
```

Možete da objedinite ove atribute u objekat za višekratnu upotrebu i da ga koristite x-bindza povezivanje sa:

```html
<button x-bind="SomeButton"></button>
 
<script>
    document.addEventListener('alpine:init', () => {
        Alpine.bind('SomeButton', () => ({
            type: 'button',
 
            '@click'() {
                this.doSomething()
            },
 
            ':disabled'() {
                return this.shouldDisable
            },
        }))
    })
</script>
```
