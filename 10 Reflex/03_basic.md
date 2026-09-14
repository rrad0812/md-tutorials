
# Refleks osnove

[Sadržaj](00_sadrzaj.md)

Ova stranica daje uvod u najčešće koncepte koje ćete koristiti za izgradnju refleks aplikacija.

Naučićete kako da:

- Instalirate refleks pomoću pip-a
- Kreirate i ugnezdite komponente
- Prilagodite i stilizujete komponente
- Razlikujete između vremena kompilacije i vremena izvođenja
- Prikažete podatke koji se menjaju tokom vremena
- Odgovorite na događaje i ažurirate ekran
- Renderujete uslovni i kreirate liste
- Kreirajte stranice i krećete se između njih.

```sh
pip install reflex
```

Uvezite refleks biblioteku da biste započeli.

```py
import reflex as rx
```

## Kreiranje i gnezdenje komponenti

Komponente su građevinski blokovi za korisnički interfejs vaše aplikacije (UI). One su vizuelni elementi koji čine vašu aplikaciju, poput dugmadi, teksta i slika. Refleks ima širok izbor ugrađenih komponenti da biste brzo započeli.

Komponente se kreiraju koristeći funkcije koje vraćaju objekat komponente.

```py
def my_button():
    return rx.button("Click Me")
```

Komponente se mogu ugnezditi jedna u drugu da bi stvorili složeni UI.

Da bi se komponente gnezdile kao deca, prosledite ih kao `pozicioni` argument roditeljskoj komponenti. U donjem primeru, `rx.text` i `my_button` komponente su deca komponente `rx.box`.

```py
def my_page():
    return rx.box(
        rx.text("This is a page"),
        # Reference components defined in other functions.
        my_button(),
    )
```

Takođe možete da koristite bilo koji osnovni HTML element kroz prostor imena `rx.el.div`. Ovo vam omogućava da koristite standardne HTML elemente direktno u vašoj refleks aplikaciji kada vam je potrebno više kontrole ili kada određena komponenta nije dostupna u biblioteci refleks komponenti.

```py
def my_div():
    return rx.el.div(
        rx.el.p("Use base html!"),
    )
```

Ako vam je potrebna komponenta koju ne pruža refleks, možete da proverite treću stranu ekosistema ili omotajte sopstvenu React komponentu.

## Prilagođavanje i stilizovanje komponente

Komponente se mogu prilagoditi pomoću `props`-ova, koji se prenose kao argumente ključnih reči na funkciju komponente.

Svaka komponenta ima `props` koje su specifične za tu komponentu. Proverite dokumentaciju za komponentu koju koristite da biste videli koji su `props` dostupni.

```py
def half_filled_progress():
    return rx.progress(value=50)
```

Pored komponentnih specifičnih `props`-a, komponente se takođe mogu oblikovati pomoću CSS svojstava koja su prosleđena kao props.

```py
def round_button():
    return rx.button(
        "Click Me", border_radius="15px", font_size="18px"
    )
```

> [!Note]  
> Koristite `snake_case` CSS property imena imovine kao `prop` ime.  
>
> [!Note]  
> Pogledajte vodič za stajling za više informacija o načinima stilizovanja komponenti.

Ukratko, komponente su napravljene od `children` komponenti i `props` propertija.

- **Children**

  - Tekst ili druge refleks komponente ugnežđene unutar komponente.
  - Prosledjuju se kao **positional arguments**.
  
- **Props**
  
  - Atributi koji utiču na ponašanje i izgled komponente.
  - Prosledjuju se kao **keyword arguments**.

## Prikazivanje podataka koji se menjaju tokom vremena

Aplikacije moraju da čuvaju i prikazuju podatke koji se menjaju tokom vremena.

Refleks je to obezbedio kroz `State` klasu:

- što je Pajton klasa koja čuva promenljive koje se mogu promeniti kada se aplikacija pokrene,
- kao i funkcije koje mogu promeniti te promenljive.

Da biste definisali `State` klasu, nasledite je iz `rx.State` klase i definišite polja koja čuvaju stanje vaše aplikacije. `State` promenljive ( `vars` ) treba da imaju `napomenu tipa` i mogu se inicijalizovati podrazumevanom vrednošću.

```py
class MyState(rx.State):
    count: int = 0
```

## Reference "state vars" u komponentama

Da biste referencali `state var` u komponenti, možete je prozvati kao `children` ili `prop`. Komponenta će se automatski ažurirati kada se `state var` promeni.

`Vars` se pozivaju putem atributa state klase. Na primer, da biste referencali `var` "count" u komponenti "MyState", koristite "MyState.count".

```py
class MyState(rx.State):
    count: int = 0
    color: str = "red"

def counter():
    return rx.hstack(
        
        # The heading `color` prop is set to the `color` var in MyState.
        rx.heading("Count: ", color=MyState.color),
        
        # The `count` var in `MyState` is passed as a child to the heading component.
        rx.heading(MyState.count),
    )
```

`Vars` se može referencirati iz više komponenti i automatski će se ažurirati kada se `state` promeni.

## Odgovaranje na događaje i ažuriranje ekrana

Do sada smo definisali `state vars`, ali nismo pokazali kako da ih promenimo. Sve promene `state vars`  rukuje se funkcijama u `state` klasi, koje se nazivaju `event handlers`.

> [!Note]  
> **Event handlers** su jedini način da se promeni `state` u Refleksu.

Komponente imaju posebne `props`, kao što su `on_click`, nazvani `event trigers` koji se mogu koristiti za interaktivne komponente.

## Event trigers

Povežite komponente na `rukovaoce događaja`, koje ažuriraju `state`.

```py
class CounterState(rx.State):
    count: int = 0

    @rx.event
    def increment(self):
        self.count += 1


def counter_increment():
    return rx.hstack(
        rx.heading(CounterState.count),
        rx.button(
            "Increment", on_click=CounterState.increment
        ),
    )
```

Kada se aktivira `event triger` - u ovom slučaju "on_click", poziva se `event hendler` - u ovom slučaju "increment", koji ažurira `var state` - u ovom slučaju "count". UI se automatski preispituje da bi odražavao novo stanje.

## Dekorator @rx.event

### Rukovaoci događaja sa argumentima

Rukovodioci događaja takođe mogu da preuzmu argumente. Na primer, obrada  dogadjaja "increment"  može da preuzme argument da poveća broj u određenom iznosu.

```py
class CounterState2(rx.State):
    count: int = 0

    @rx.event
    def increment(self, amount: int):
        self.count += amount

def counter_variable():
    return rx.hstack(
        rx.heading(CounterState2.count),
        rx.button(
            "Increment by 1",
            on_click=lambda: CounterState2.increment(1),
        ),
        rx.button(
            "Increment by 5",
            on_click=lambda: CounterState2.increment(5),
        ),
    )
```

`Event triger` `on_click` ovde ne donosi nikakve argumente, ali neki `event trigeri` to rade. Na primer, okidač događaja `on_blur` prosledjuje tekst kao argument na rukovalac događaja.

```py
class TextState(rx.State):
    text: str = ""

    @rx.event
    def update_text(self, new_text: str):
        self.text = new_text

def text_input():
    return rx.vstack(
        rx.heading(TextState.text),
        rx.input(
            default_value=TextState.text,
            on_blur=TextState.update_text,
        ),
    )
```

> [!Note]  
> Proverite da li rukovalac događaja ima isti broj argumenata kao i okidač događaja ili će se
  pojaviti greška.

## COMPILE-TIME vs. RUNTIME (VAŽNO)

Pre nego što zaronimo dublje u stanja, važno je razumeti razliku između vremena kompilacije i izvršavanja u Refleksu.

Kada pokrenete svoju aplikaciju, frontend je kompajliran u JavaScript kod koji radi u pregledaču (compile-time). Backend ostaje na Pajtonu i radi na serveru tokom života aplikacije (run-time).

### Kada ne možete da koristite čisti Pithon?

Ne možemo da kompajliramo proizvoljni Pajton kod, samo onaj koji definišemo u komponentama.

> [!Note]  
> Važno je da ne možete da koristite proizvoljne operacije i funkcije u Pajtonu na `state vars` u
  komponentama.

Međutim, budući da su bilo koji rukovaoci događaja u `state` klasi  na backendu, možete koristiti bilo koji Pajton kod ili biblioteku u `state` klasi.

**Primeri koji rade:**

Unutar rukovaoca događaja koristite bilo koji pajton kod ili biblioteku.

```py
def check_even(num: int):
    return num % 2 == 0

class MyState3(rx.State):
    count: int = 0
    text: str = "even"

    @rx.event
    def increment(self):
        # Use any Python code within state.
        # Even reference functions defined outside the state.
        if check_even(self.count):
            self.text = "even"
        else:
            self.text = "odd"
        self.count += 1

def count_and_check():
    return rx.box(
        rx.heading(MyState3.text),
        rx.button("Increment", on_click=MyState3.increment),
    )
```

> [!Note]  
> Koristite bilo koju funkciju Pajtona u komponentama, sve dok je definisana u compile-time (tj. ne
  referencira nijednu state var)

```sh
0true
1false
2true
3false
4true
5false
6true
7false
8true
9false
```

```py
def show_numbers():
    return rx.vstack(
        *[rx.hstack(i, check_even(i)) for i in range(10)]
    )
```

**Primeri koji ne rade:**

Ne možete da uradite `if` izjavu na vars u komponentama, jer vrednost nije poznata u `compile-time`.

```py
class BadState(rx.State):
    count: int = 0

def count_if_even():
    return rx.box(
        rx.heading("Count: "),
        # This will raise a compile error, as BadState.count is a var and not known at compile time.
        rx.text(
            "Even" if BadState.count % 2 == 0 else "Odd"
        ),
        # Using an if statement with a var as a prop will NOT work either.
        rx.text(
            "hello",
            color=(
                "red" if BadState.count % 2 == 0 else "blue"
            ),
        ),
    )
```

Ne možete pokrenuti `for` petlju na listi vars.

```py
class BadState(rx.State):
    items: list[str] = ["Apple", "Banana", "Cherry"]

def loop_over_list():
    return rx.box(
        # This will raise a compile error, as BadState.items is a list and not known at compile time.
        *[rx.text(item) for item in BadState.items]
    )
```

Ne možete da uradite produkcione operacije Pajtonu na state vars u komponentama.

```py
class BadTextState(rx.State):
    text: str = "Hello world"

def format_text():
    return rx.box(
        # Python operations such as `len` will not work on state vars.
        rx.text(len(BadTextState.text)),
    )
```

U narednim odeljcima pokazaćemo kako se baviti ovim slučajevima.

### Uslovno prikazivanje

Kao što je gore pomenuto, ne možete da koristite Pajton `if/else` izjave na state vars u komponentama. Umesto toga, koristite `rx.cond` funkciju da uslovno renderujete komponente.

```py
class LoginState(rx.State):
    logged_in: bool = False

    @rx.event
    def toggle_login(self):
        self.logged_in = not self.logged_in

def show_login():
    return rx.box(
        rx.cond(
            LoginState.logged_in,
            rx.heading("Logged In"),
            rx.heading("Not Logged In"),
        ),
        rx.button(
            "Toggle Login", on_click=LoginState.toggle_login
        ),
    )
```

### Renderovanje liste

Da biste iterirali var koja je lista, koristite funkciju `rx.foreach`. Prenesite var listu i funkciju koja vraća komponentu, kao argumente u `rx.foreach`.

```py
class ListState(rx.State):
    items: list[str] = ["Apple", "Banana", "Cherry"]

def render_item(item: rx.Var[str]):
    """Render a single item."""
    # Note that item here is a Var, not a str!
    return rx.list.item(item)

def show_fruits():
    return rx.box(
        rx.foreach(ListState.items, render_item),
    )
```

The function that renders each item takes in a Var, since this will get compiled up front.

### Var operacije

Ne možete koristiti proizvoljne operacije na state vars u komponentama, ali Refleks ima var operacije koje možete koristiti za manipulisanje state vars.

Na primer, da biste proverili da li je var parna, možete koristiti `%` i `==` var operacije.

```py
class CountEvenState(rx.State):
    count: int = 0

    @rx.event
    def increment(self):
        self.count += 1

def count_if_even():
    return rx.box(
        rx.heading("Count: "),
        rx.cond(
            # Here we use the `%` and `==` var operations to check if the count is even.
            CountEvenState.count % 2 == 0,
            rx.text("Even"),
            rx.text("Odd"),
        ),
        rx.button(
            "Increment", on_click=CountEvenState.increment
        ),
    )
```

## Aplikacija i strane

Refleks aplikacije se kreiraju instantirajući `rx.app` klasu. Stranice su povezane sa specifičnim rutama za URL i stvaraju se definisanjem funkcije koja vraća komponentu.

```py
def index():
    return rx.text("Root Page")

rx.app = rx.App()
app.add_page(index, route="/")
```

## Sledeći koraci

Sada kada imate osnovno razumevanje načina na koji Refleks radi, sledeći korak je početak kodiranja sopstvenih aplikacija. Probajte i sledeće tutorijala:

- Dashboard Tutorijal
- Chatapp Tutorijal

[Sadržaj](00_sadrzaj.md)
