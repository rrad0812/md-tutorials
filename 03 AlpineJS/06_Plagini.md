
# Uvod u AlpineJS

## 6 Plagini

[Sadržaj](00_Sadržaj.md)

### 6.1 Plagin mask

Alpineov dodatak Mask vam omogućava da automatski formatirate polje za unos teksta dok korisnik kuca.

Ovo je korisno za mnoge različite vrste unosa: brojeve telefona, kreditne kartice, iznose u dolarima, brojeve računa, datume itd.

**Instalacija**:

Ovaj dodatak možete koristiti tako što ćete ga uključiti iz `<script>` oznake ili ga instalirati putem `NPM`-a:

***Preko CDN-a**

Možete uključiti CDN verziju ovog dodatka kao `<script>` oznaku, samo se pobrinite da je uključite **PRE** Alpine-ove osnovne JS datoteke.

```html
<!-- Alpine Plugins -->
<script defer src="https://cdn.jsdelivr.net/npm/@alpinejs/mask@3.x.x/dist/cdn.min.js"></script>
 
<!-- Alpine Core -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
```

***Preko NPM-a***

Možete instalirati Masku iz `NPM`-a za upotrebu unutar vašeg paketa na sledeći način:

```js
npm install @alpinejs/mask
```

Zatim ga inicijalizujte iz vašeg paketa:

```js
import Alpine from 'alpinejs'
import mask from '@alpinejs/mask'
 
Alpine.plugin(mask)
 
...
```

**x-mask**:

Primarni API za korišćenje ovog dodatka je `x-mask` direktiva.

Počnimo sa sledećim jednostavnim primerom polja za datum:

```html
<input x-mask="99/99/9999" placeholder="MM/DD/YYYY">
```

Obratite pažnju na to kako tekst koji unosite u polje za unos mora da se pridržava formata koji je dao `x-mask`. Pored prisiljavanja numeričkih znakova, kose crte `/` se takođe automatski dodaju ako ih korisnik prvo ne unese.

Sledeći džoker znakovi su podržani u maskama:

| DŽoker | Opis |
| ------ | ---- |
| * | Bilo koji znak |
| a | Samo alfa znakovi (a...z, A...Z) |
| 9 | Samo numerički znakovi (0-9) |

**Dinamičke maske**:

Ponekad jednostavni literali maske (npr. (999) 999-9999) nisu dovoljni. U tim slučajevima, `x-mask:dynamic` omogućava vam dinamički generisanje maski u hodu na osnovu korisničkog unosa.

Evo primera unosa kreditne kartice kojoj je potrebno da promeni masku na osnovu toga da li broj počinje brojevima „34“ ili „37“ (što znači da je u pitanju Amex kartica i stoga ima drugačiji format).

```html
<input x-mask:dynamic="
    $input.startsWith('34') || $input.startsWith('37')
        ? '9999 999999 99999' : '9999 9999 9999 9999'
">
```

Kao što možete videti u gornjem primeru, svaki put kada korisnik unese vrednost, ta vrednost se prosleđuje izrazu kao `$input`. Na osnovu `$input`, u polju se koristi drugačija maska.

Probajte sami tako što ćete ukucati broj koji počinje sa "34" i jedan koji ne počinje sa "34".

`x-mask:dynamic` takođe prihvata funkciju kao rezultat izraza i automatski će je proslediti `$input` kao prvi parametar. Na primer:

```html
<input x-mask:dynamic="creditCardMask">
 
<script>
function creditCardMask(input) {
    return input.startsWith('34') || input.startsWith('37')
        ? '9999 999999 99999'
        : '9999 9999 9999 9999'
}
</script>
```

**Novčani ulazi**:

Pošto je pisanje sopstvenog dinamičkog izraza maske za novčane unose prilično složeno, Alpine nudi unapred napravljen i čini ga dostupnim kao `$money()`.

Evo potpuno funkcionalne maske za unos novca:

```html
<input x-mask:dynamic="$money($input)">
```

Ako želite da zamenite tačke za zareze i obrnuto (kao što je potrebno u određenim valutama), to možete učiniti koristeći drugi opcioni parametr:

```html
<input x-mask:dynamic="$money($input, ',')">
```

Takođe možete da zamenite separator hiljada tako što ćete navesti treći opcioni argument:

```html
<input x-mask:dynamic="$money($input, '.', ' ')">
```

Takođe možete poništiti podrazumevanu preciznost od 2 cifre korišćenjem bilo kog željenog broja cifara kao četvrtog opcionog argumenta:

```html
<input x-mask:dynamic="$money($input, '.', ',', 4)">   
```

[Sadržaj](00_Sadržaj.md)

### 6.2 Plagin intersect

AlpineJS dodatak Intersect je praktični omotač za Intersection Observer koji vam omogućava da lako reagujete kada element uđe u prikazni prozor.

Ovo je korisno za: lenje učitavanje slika i drugog sadržaja, pokretanje animacija, beskonačno skrolovanje, evidentiranje „pregleda“ sadržaja itd.

**Instalacija**:

Ovaj dodatak možete koristiti tako što ćete ga uključiti iz `<script>` oznake ili ga instalirati putem `NPM`:

***Preko CDN-a***

Možete uključiti CDN verziju ovog dodatka kao `<script>` oznaku, samo se pobrinite da je uključite `PRE` Alpine-ove osnovne JS datoteke.

```html
<!-- Alpine Plugins -->
<script defer src="https://cdn.jsdelivr.net/npm/@alpinejs/intersect@3.x.x/dist/cdn.min.js"></script>
 
<!-- Alpine Core -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>   
```

***Preko NPM***

Možete instalirati Intersect iz NPM-a za upotrebu unutar vašeg paketa na sledeći način:

```sh
npm install @alpinejs/intersect
```

Zatim ga inicijalizujte iz vašeg paketa:

```js
import Alpine from 'alpinejs'
import intersect from '@alpinejs/intersect'
 
Alpine.plugin(intersect)
 
...
```

**x-presek**:

Primarni API za korišćenje ovog dodatka je `x-intersect`. Možete dodavati `x-intersect` bilo kom elementu unutar Alpine komponente, i kada ta komponenta uđe u prikazni prozor (skroluje se u prikaz), dati izraz će se izvršiti.

Na primer, u sledećem isečku, `shown` će ostati "false" dok se element ne pomeri u vidokrug. U tom trenutku, izraz će se izvršiti i "shown" će postati "true":

```html
<div x-data="{ shown: false }" x-intersect="shown = true">
    <div x-show="shown" x-transition>
        I'm in the viewport!
    </div>
</div>
```
 
x-intersect:enter

Sufiks `:enter` je pseudonim od `x-intersect` i funkcioniše na isti način:

```html
<div x-intersect:enter="shown = true">...</div>
```

Možete izabrati da ovo koristite radi jasnoće kada koristite i `:leave` sufiks.

**x-intersect:leave**:

Dodavanje `:leave` sufiksa pokreće vaš izraz kada element napusti prikazni okvir.

```html
<div x-intersect:leave="shown = true">...</div>      
```

> [!Note]
> Podrazumevano, ovo znači da ceo element nije u prikaznom prozoru. Koristite `x-intersect:leave.full` za pokretanje izraza kada samo delovi elementa nisu u prikaznom prozoru.

**Modifikatori**:

***.once***

Ponekad je korisno izračunati izraz samo prvi put kada element uđe u prikazni okvir, a ne i narednih puta. Na primer, prilikom pokretanja "enter" animacija. U tim slučajevima, možete dodati modifikator `.once` na `x-intersect` da biste to postigli.

```html
<div x-intersect.once="shown = true">...</div>
```

***.half***

Izračunava izraz kada prag intersecta pređe 0.5.

Korisno za elemente gde je važno prikazati barem deo elementa.

```html
<div x-intersect.half="shown = true">...</div> // when `0.5` of the element is in the viewport
```

***.full***

Izračunava izraz kada prag intersecta pređe 0.99.

Korisno za elemente gde je važno prikazati ceo element.

```html
<div x-intersect.full="shown = true">...</div> // when `0.99` of the element is in the viewport
```

.treshold

Omogućava vam da kontrolišete `threshold` svojstvo `IntersectionObserver`:

Ova vrednost treba da bude u opsegu od "0-100". Vrednost "0" znači: pokrenuti "intersect" ako BILO KOJI deo elementa uđe u prikazni okvir (podrazumevano ponašanje). Dok vrednost "100" znači: ne pokrenuti "intersect“ osim ako ceo element nije ušao u prikazni okvir.

Bilo koja vrednost između je procenat te dve krajnosti.

Na primer, ako želite da pokrenete intersect nakon što polovina elementa uđe na stranicu, možete koristiti `.threshold.50`:

```html
<div x-intersect.threshold.50="shown = true">...</div> // when 50% of the element is in the viewport
```

Ako želite da se pokrene samo kada 5% elementa uđe u prikazni prozor, možete koristiti: `.threshold.05`, i tako dalje i tako dalje.

***.margin***

Omogućava vam da kontrolišete `rootMargin` `IntersectionObserver`. Ovo efikasno podešava veličinu granice prikaza. Pozitivne vrednosti proširuju granicu izvan prikaza, a negativne vrednosti je skupljaju ka unutra. Vrednosti funkcionišu kao CSS margina:

- jedna vrednost za sve strane;
- dve vrednosti za gore/dole, levo/desno;
- ili četiri vrednosti za gore, desno, dole, levo.

Možete koristiti vrednosti pxi %, ili koristiti goli broj da biste dobili vrednost u pikselu.

```html
<div x-intersect.margin.200px="loaded = true">...</div> // Load when the element is within 200px of the viewport
```

```html
<div x-intersect:leave.margin.10%.25px.25.25px="loaded = false">...</div> // Unload when the element gets within 10% of the top of the viewport, or within 25px of the other three edges
```

```html
<div x-intersect.margin.-100px="visible = true">...</div> // Mark as visible when element is more than 100 pixels into the viewport.
```

[Sadržaj](00_Sadržaj.md)

### 6.3 Plagin resize

AlpineJS dodatak za promenu veličine je praktični omotač za Resize Observer koji vam omogućava da lako reagujete kada element promeni veličinu.

Ovo je korisno za: animacije zasnovane na prilagođenoj veličini, inteligentno lepljivo pozicioniranje, uslovno dodavanje atributa na osnovu veličine elementa itd.

**Instalacija**:

Ovaj dodatak možete koristiti tako što ćete ga uključiti iz `<script>` oznake ili ga instalirati putem `NPM`-a:

***Preko CDN***

Možete uključiti CDN verziju ovog dodatka kao `<script>` oznaku, samo se pobrinite da je uključite `PRE` Alpine-ove osnovne JS datoteke.

```html
<!-- Alpine Plugins -->
<script defer src="https://cdn.jsdelivr.net/npm/@alpinejs/resize@3.x.x/dist/cdn.min.js"></script>

<!-- Alpine Core -->
<script defer src="https://cdn.jsdelivr.net/npm/alpinejs@3.x.x/dist/cdn.min.js"></script>
```

***Preko NPM***

Možete instalirati Resize iz NPM-a za upotrebu unutar vašeg paketa na sledeći način:

```sh
npm install @alpinejs/resize
```

Zatim ga inicijalizujte iz vašeg paketa:

```js
import Alpine from 'alpinejs'
import resize from '@alpinejs/resize'

Alpine.plugin(resize)

...
```

**Promena veličine x-a**:

Primarni API za korišćenje ovog dodatka je `x-resize`. Možete dodavati `x-resize` bilo kom elementu unutar Alpine komponente, a kada se veličina tog elementa promeni iz bilo kog razloga, dati izraz će se izvršiti sa dva magična svojstva: `$width` i `$height`.

Na primer, evo jednostavnog primera korišćenja `x-resize` za prikaz širine i visine elementa dok menja veličinu.

```html
<div
    x-data="{ width: 0, height: 0 }"
    x-resize="width = $width; height = $height"
>
    <p x-text="'Width: ' + width + 'px'"></p>
    <p x-text="'Height: ' + height + 'px'"></p>
</div>
```

Promenite veličinu prozora pregledača da biste videli kako se vrednosti širine i visine menjaju.

```html
Širina: 686 piksela
Visina: 104 piksela
```

**Modifikatori**:

***.document***

Često je korisno posmatrati veličinu celog dokumenta, a ne određenog elementa. Da biste to uradili, možete dodati modifikator `.document` na `x-resize`:

```html
<div x-resize.document="...">
```

Promenite veličinu prozora pregledača da biste videli kako se vrednosti širine i visine dokumenta menjaju.

```html
Širina: 1866 piksela
Visina: 4389 piksela
```

### 6.4
