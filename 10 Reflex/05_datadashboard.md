
# Tutorijal: Data Dashboard

[Sadržaj](00_sadrzaj.md)

Tokom ovog tutorijala izgradićete malu kontrolnu tablu za podatke, gde možete da unesete podatke i to će se prikazati u tabeli i grafikonu.Ovaj vodič ne pretpostavlja bilo kakvo postojeće znanje Refleksa, ali preporučujemo da prvo proverite vodič "Osnove Reflexa".

Tehnike koje ćete naučiti u vodiču su od suštinskog značaja za izgradnju bilo koje Refleks aplikacije i u potpunosti razumeti da će vam pružiti duboko razumevanje Refleksa.

Ovaj vodič je podeljen na nekoliko odeljka:

- **Podešavanje za tutorial**: Polazište za praćenje vodiča
- **Pregled**: Osnove refleks UI (komponente i rekvizite)
- **Pokazivanje dinamičnih podataka**: Kako se koristi stanja da se prikažu podaci u aplikaciji.
- **Dodajte podatke na svoju aplikaciju**: pomoću obrasca i uveđenje rukovaoce događajima.
- **Izveštavanje podataka na grafikonu**: Kako se koristi komponente refleksa.
- **Završno čišćenje i zaključak**: Kako dalje prilagoditi svoju aplikaciju i dodati stil na to.

**Šta gradite?**

U ovom tutorialu gradite interaktivnu kontrolnu tablu sa Refleksom.

```py
import reflex as rx

from collections import Counter

class User(rx.Base):
    """The user model."""
    name: str
    email: str
    gender: str

class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]
    users_for_graph: list[dict] = []

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))
        self.transform_data()

    def transform_data(self):
        """Transform user gender group data into a format suitable for visualization in graphs."""
        # Count users of each gender group
        gender_counts = Counter(
            user.gender for user in self.users
        )

        # Transform into list of dict so it can be used in the graph
        self.users_for_graph = [
            {"name": gender_group, "value": count}
            for gender_group, count in gender_counts.items()
        ]

def show_user(user: User):
    """Show a user in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
        style={"_hover": {"bg": rx.color("gray", 3)}},
        align="center",
    )

def add_customer_button() -> rx.Component:
    return rx.dialog.root(
        rx.dialog.trigger(
            rx.button(
                rx.icon("plus", size=26),
                rx.text("Add User", size="4"),
            ),
        ),
        rx.dialog.content(
            rx.dialog.title(
                "Add New User",
            ),
            rx.dialog.description(
                "Fill the form with the user's info",
            ),
            rx.form(
                rx.flex(
                    rx.input(
                        placeholder="User Name",
                        name="name",
                        required=True,
                    ),
                    rx.input(
                        placeholder="user@reflex.dev",
                        name="email",
                    ),
                    rx.select(
                        ["Male", "Female"],
                        placeholder="male",
                        name="gender",
                    ),
                    rx.flex(
                        rx.dialog.close(
                            rx.button(
                                "Cancel",
                                variant="soft",
                                color_scheme="gray",
                            ),
                        ),
                        rx.dialog.close(
                            rx.button(
                                "Submit", type="submit"
                            ),
                        ),
                        spacing="3",
                        justify="end",
                    ),
                    direction="column",
                    spacing="4",
                ),
                on_submit=State.add_user,
                reset_on_submit=False,
            ),
            max_width="450px",
        ),
    )

def graph():
    return rx.recharts.bar_chart(
        rx.recharts.bar(
            data_key="value",
            stroke=rx.color("accent", 9),
            fill=rx.color("accent", 8),
        ),
        rx.recharts.x_axis(data_key="name"),
        rx.recharts.y_axis(),
        data=State.users_for_graph,
        width="100%",
        height=250,
    )

def index() -> rx.Component:
    return rx.vstack(
        add_customer_button(),
        rx.table.root(
            rx.table.header(
                rx.table.row(
                    rx.table.column_header_cell("Name"),
                    rx.table.column_header_cell("Email"),
                    rx.table.column_header_cell("Gender"),
                ),
            ),
            rx.table.body(
                rx.foreach(State.users, show_user),
            ),
            variant="surface",
            size="3",
            width="100%",
        ),
        graph(),
        align="center",
        width="100%",
    )

app = rx.App(
    theme=rx.theme(radius="full", accent_color="grass"),
)

app.add_page(
    index,
    title="Customer Data App",
    description="A simple app to manage customer data.",
    on_load=State.transform_data,
)
```

Ne brinite ako ne razumete gornji kod, u ovom tutorialu ćemo vas prošetati kroz celu stvar korak po korak.

## Podešavanje za tutorial

Pogledajte dokumente u instalaciju da biste dobili Refleks postavljen na svoju mašinu. Sledite ih da biste kreirali direktorijum koja se zove "Dashboard_Tutorial", u koji ćete sa pip-om instalirati Refleks.

Biraćemo šablon `0` kada pokrećemo `reflex init` da biste dobili prazan templejt. Na kraju pokrenite `reflex run` da biste pokrenuli aplikaciju i potvrdili da je sve ispravno postavljeno.

## Pregled

Sad kad smo postavljeni, hajde da malo pregledamo Refleks!

### Pregled starterskog koda

Unutar našeg `dashboard_tutorial` direktorijuma, unutra smo sa `cd`, nalazi se i `rxconfig.py` datoteka koja sadrži konfiguraciju za našu Refleks aplikaciju.

Postoji i direktorijum `assets` u koji se mogu postaviti statičke datoteke poput slika i stilova, da bi se referencirali u vašoj aplikaciji.

Ono što je najvažnije, postoji i direktorijum koja se takođe zove `dashboard_tutorial`, koji sadrži sav kod za vašu aplikaciju. Unutar ovog direktorijuma nalazi se datoteka po imenu `dashboard_tutorial.py`. Da biste započeli ovaj vodič, izbrisaćemo sav kod u ovoj datoteci tako da možemo početi ispočetka i objasniti svaki korak dok idemo napred.

Prvo što moramo da uradimo je uvoz Refleks-a. Jednom kada to učinimo, možemo da stvaramo komponentu, koja je deo korisničkog interfejsa. Komponente se koriste za prikazivanje, upravljanje i ažuriranje UI elemenata u vašoj aplikaciji.

Pogledajmo primer ispod.

Ovde imamo funkciju pod nazivom `index` koja vraća `text` komponentu (ugrađena Refleks UI komponenta) koja prikazuje tekst "Hello World!".

Zatim definišemo našu aplikaciju koristeći `app = rx.App()` i dodajemo komponentu koju smo upravo definisali (index) na stranicu pomoću `app.add_page(index)`. Ime funkcije (u ovom primeru `index`) koji definiše komponentu, mora biti ono što prosledjujemo u `add_page`.

Definicija aplikacije i dodavanje komponente na stranicu potrebna je za svaku `Refleks` aplikaciju.

```py
import reflex as rx

def index() -> rx.Component:
    return rx.text("Hello World!")

app = rx.App()
app.add_page(index)
```

Ovaj kod će renderisati stranu sa tekstom "Hello World!" kada pokrenemo aplikaciju.

> [!Note]  
> Za ostatak tutorijala kod:
>
> ```py
> app = rx.App()
> app.add_page
> ```
>
> će se podrazumevati i ako nije prikazan u isečcima koda.

### Kreiranje tabele

Kreirajmo novu komponentu koja će prikazati tabelu.

Koristićemo komponentu `rx.table` da to uradimo. Komponenta `rx.table` ima `root`, koji uzima `header` i `body`, koje zauzvrat uzima u komponente `row`. Komponenta `row` uzima `cell` komponente u kojoj su stvarni podaci koji će se prikazati u tabeli.

```py
def index() -> rx.Component:
    return rx.table.root(
        rx.table.header(
            rx.table.row(
                rx.table.column_header_cell("Name"),
                rx.table.column_header_cell("Email"),
                rx.table.column_header_cell("Gender"),
            ),
        ),
        rx.table.body(
            rx.table.row(
                rx.table.cell("Danilo Sousa"),
                rx.table.cell("danilo@example.com"),
                rx.table.cell("Male"),
            ),
            rx.table.row(
                rx.table.cell("Zahra Ambessa"),
                rx.table.cell("zahra@example.com"),
                rx.table.cell("Female"),
            ),
        ),
    )
```

Komponente u Refleksu imaju `props`, koje se mogu koristiti za prilagođavanje komponenti i prenose se kao argumenti ključnih reči na funkciju komponente.

`rx.table.root` komponenta ima `variant` i `size` props, koji prilagođavaju tabelu kao što se vidi  u daljem tekstu.

```py
def index() -> rx.Component:
    return rx.table.root(
        rx.table.header(
            rx.table.row(
                rx.table.column_header_cell("Name"),
                rx.table.column_header_cell("Email"),
                rx.table.column_header_cell("Gender"),
            ),
        ),
        rx.table.body(
            rx.table.row(
                rx.table.cell("Danilo Sousa"),
                rx.table.cell("danilo@example.com"),
                rx.table.cell("Male"),
            ),
            rx.table.row(
                rx.table.cell("Zahra Ambessa"),
                rx.table.cell("zahra@example.com"),
                rx.table.cell("Female"),
            ),
        ),
        variant="surface",
        size="3",
    )
```

### Prikazivanje dinamičkih podataka (State)

Do ove tačke sve podatke koje se prikazujemo u aplikaciji su statički. Ovo nije baš korisno za kontrolnu tablu za podatke. Moramo biti u mogućnosti da pokažemo dinamičke podatke koji se mogu dodati i ažurirati.

Ovde ulazi klasa `state`.

> [!Note]  
> `State` je pajton klasa koja čuva promenljive koje se mogu promeniti kada se aplikacija pokrene,
  kao i funkcije koje mogu promeniti te promenljive.

Da biste definisali `State` klasu, nasledite klasu `rx.state` i definišite polja koja čuvaju stanje vaše aplikacije. State promenljive ( `vars` ) treba da imaju `napomenu tipa` i mogu se inicijalizovati podrazumevanom vrednošću.

U donjem primeru definišemo `State` klasu koje ima promenljivu koja se zove "users" koja je lista lista stringova. Svaka lista u listi "users" predstavlja korisnika i sadrži njegovo "name", "e-mail" i "pol".

```py
class State(rx.State):
    users: list[list[str]] = [
        ["Danilo Sousa", "danilo@example.com", "Male"],
        ["Zahra Ambessa", "zahra@example.com", "Female"],
    ]
```

Da bi se iteriralo preko `state vars`, koje su ovde tipa liste, koristimo funkciju `rx.foreach` da renderujemo listu komponenti. `rx.foreach` uzima iterable (listu, tuple ili dict) i funkciju koja renderuje svaku  iterable stavku.

Ovde je render funkcija "show_user" koja uzima jednog korisnika i vraća `tabelu.row` komponentu koja prikazuje ime korisnika, e-poštu i pol.

```py
class State(rx.State):
    users: list[list[str]] = [
        ["Danilo Sousa", "danilo@example.com", "Male"],
        ["Zahra Ambessa", "zahra@example.com", "Female"],
    ]

def show_user(person: list):
    """Show a person in a table row."""
    return rx.table.row(
        rx.table.cell(person[0]),
        rx.table.cell(person[1]),
        rx.table.cell(person[2]),
    )

def index() -> rx.Component:
    return rx.table.root(
        rx.table.header(
            rx.table.row(
                rx.table.column_header_cell("Name"),
                rx.table.column_header_cell("Email"),
                rx.table.column_header_cell("Gender"),
            ),
        ),
        rx.table.body(
            rx.foreach(State.users, show_user),
        ),
        variant="surface",
        size="3",
    )
```

Kao što vidite, izlaz izgleda isto kao i pre, osim što sada podaci više nisu statički i mogu se menjati iz korisničkog ulaza u aplikaciju.

### Korišćenje odgovarajuće strukture klase za podatke

Do sada su naši podaci definisani kao lista listi stringova, gde se podacima pristupa indeksom tj users[0], users[1]. Ovo nije baš održivo, jer će naša aplikacija postajati sve veća.

Bolji način strukturiranja naših podataka u Refleksu je da koristite klasu da bi predstavljali "users". Na ovaj način možemo pristupiti podacima koristeći atribute klase, tj. "user.name", "user.email".

U Refleksu kada kreiramo ove klase da bismo prikazali naše podatke, klasa mora naslediti od `rx.Base` klase.

`rx.Base` je takođe neophodna ako želimo da imamo `state vars` koje su iterable sa različitim tipovima. Na primer, ako bismo želeli da imamo `age` kao `int`, morali bismo da koristimo `rx.Base` jer to nismo mogli da uradimo sa state varijable definisanim kao `list[list[str]]`.

Render funkcija "show_user" se takođe ažurira da bi pristupila podacima preko atributa, umesto preko indeksa.

```py
class User(rx.Base):
    """The user model."""
    name: str
    email: str
    gender: str

class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]

def show_user(user: User):
    """Show a person in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
    )

def index() -> rx.Component:
    return rx.table.root(
        rx.table.header(
            rx.table.row(
                rx.table.column_header_cell("Name"),
                rx.table.column_header_cell("Email"),
                rx.table.column_header_cell("Gender"),
            ),
        ),
        rx.table.body(
            rx.foreach(State.users, show_user),
        ),
        variant="surface",
        size="3",
    )
```

Zatim dodajemo formu na aplikaciju, tako da možemo dodati nove korisnike na tabelu.

### Korišćenje forme za dodavanje podataka

Kreiramo obrazac koji koristi `rx.form`, koji uzima nekoliko komponenti kao što su `rx.input` i `rx.select`, koji predstavljaju polja u formi, koji vam omogućavaju da dodate informacije.

`rx.input` komponenta uzima u nekoliko props-ova. `placeholder` je tekst koji je prikazan u polju za unos kada je prazno. `name` prop je ime ulaznog polja, koje se prenosi u rečniku kada se obrazac submituje. `required` prop je `boolean` koji određuje da li je polje obavezno za unos.

`rx.select` komponenta uzima listu opcija koje su prikazane u padajućem meniju. Ostali props ovde su identični `rx.input` komponenti.

```py
rx.form(
    rx.input(
        placeholder="User Name", 
        name="name", 
        required=True
    ),
    rx.input(
        placeholder="user@reflex.dev",
        name="email",
    ),
    rx.select(
        ["Male", "Female"],
        placeholder="Male",
        name="gender",
    ),
)
```

Ova forma je vrlo kompaktna što možete videti iz primera, tako da moramo da dodamo malo stila da bi izgledalo bolje. To možemo učiniti dodavanjem komponente `vstac` oko polja forme. Komponenta `vstack` postavlja elemente forme vertikalno.

```py
rx.form(
    rx.vstack(
        rx.input(
            placeholder="User Name",
            name="name",
            required=True,
        ),
        rx.input(
            placeholder="user@reflex.dev",
            name="email",
        ),
        rx.select(
            ["Male", "Female"],
            placeholder="Male",
            name="gender",
        ),
    ),
)
```

Sada ste verovatno shvatili da imamo sva polja forme, ali nemamo načina da podnesemo formu. Postavite dugme za slanje u forme dodavanjem `rx.button` komponente u `vstack` komponentu. `rx.button` komponenta uzima  tekst koji je prikazan na dugmetu i prop `type=submit`. Prop `type` je postavljen da forma podnese kada se kliknete na dugme.

Pored ovoga, potreban nam je način da ažuriramo `state` promenljivu `users` kada se obrazac podnese. Sve promene `state` rukuje se funkcijama u `state` klasi, koje se nazivaju `event henlerima`.

Komponente imaju specijalne props nazvane `event trigeri`, kao što su `on_submit`, koji se mogu koristiti za interaktivne komponente. `Event trigeri` povezuju komponente i `event hendlere`, koje ažuriraju `state`.Različiti `event trigeri` očekuju da od `event hendleri` da će im proslediti različite argumente (a neki ne uzimaju argumente).

Triger u `rx.form`, `on_submit` zakačen je na event hendler `add_user` koji je definisan u `state` klasi. Ovaj event triger očekuje da će proslediti vrednosti polja forme, ka event hendleru "add_user". Podaci obrasca se pruzimaju kao rečnik, i dodaje u promenljivu "users" klase `state`.

```py
class State(rx.State):
    ...
    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))

def form():
    return rx.form(
        rx.vstack(
            rx.input(
                placeholder="User Name",
                name="name",
                required=True,
            ),
            rx.input(
                placeholder="user@reflex.dev",
                name="email",
            ),
            rx.select(
                ["Male", "Female"],
                placeholder="Male",
                name="gender",
            ),
            rx.button("Submit", type="submit"),
        ),
        on_submit=State.add_user,
        reset_on_submit=True,
    )
```

Konačno moramo dodati novu form komponentu koju smo definisali na `index()` funkciju tako da se obrazac donese na stranici.

Ispod je do sada kompletan kod za aplikaciju. Ako isprobate ovu formu, videćete da možete dodati nove korisnike na tabelu popunjavanjem forme i klikom na dugme za slanje. Podaci iz forme će se takođe pojaviti kao tost (mali prozor u uglu stranice) na ekranu kada je dostavljen.

```py
class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))

def show_user(user: User):
    """Show a person in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
    )

def form():
    return rx.form(
        rx.vstack(
            rx.input(
                placeholder="User Name",
                name="name",
                required=True,
            ),
            rx.input(
                placeholder="user@reflex.dev",
                name="email",
            ),
            rx.select(
                ["Male", "Female"],
                placeholder="Male",
                name="gender",
            ),
            rx.button("Submit", type="submit"),
        ),
        on_submit=State.add_user,
        reset_on_submit=True,
    )

def index() -> rx.Component:
    return rx.vstack(
        form(),
        rx.table.root(
            rx.table.header(
                rx.table.row(
                    rx.table.column_header_cell("Name"),
                    rx.table.column_header_cell("Email"),
                    rx.table.column_header_cell("Gender"),
                ),
            ),
            rx.table.body(
                rx.foreach(State.users, show_user),
            ),
            variant="surface",
            size="3",
        ),
    )
```

### Postavljanje obrasca u overlay

U Refleksu, volimo da korisnička interakcija bude što intuitivnija.Postavljanje forme koju smo upravo izgradili u overlay stvara fokusiranu interakciju zatamljavanjem pozadine i osigurava čistiji izgled kada imate više akcionih tačaka kao što su i uređivanje i brisanje.

Forma će postaviti unutar komponente `rx.dialog` (takođe naziva se modal). `rx.dialog.root` sadrži sve delove dijaloga i `rx.dialog.trigger` omotava kontrolu koja će otvoriti dijalog. U našem slučaju triger će biti `rx.button` koji kaže "Dodavanje korisnika" kao što je prikazano u nastavku.

```py
rx.dialog.trigger(
    rx.button(
        rx.icon("plus", size=26),
        rx.text("Add User", size="4"),
    ),
)
```

Nakon trigera imamo `rx.dialog.content` koji sadrži sve u našem dijalogu, uključujući naslov, opis i našu formu. Prvi put za zatvaranje dijaloga bez slanja forme i drugog put je da zatvorite dijalog podnošenjem forme kao što je prikazano u nastavku. Ovo zahteva dva `rx.dialog.close` komponente u okviru dijaloga.

```py
rx.dialog.close(
    rx.button(
        "Cancel",
        variant="soft",
        color_scheme="gray",
    ),
),
rx.dialog.close(
    rx.button("Submit", type="submit"),
)
```

Ukupni kod za dijalog sa formom u njemu je u nastavku.

```py
rx.dialog.root(
    rx.dialog.trigger(
        rx.button(
            rx.icon("plus", size=26),
            rx.text("Add User", size="4"),
        ),
    ),
    rx.dialog.content(
        rx.dialog.title(
            "Add New User",
        ),
        rx.dialog.description(
            "Fill the form with the user's info",
        ),
        rx.form(
            # flex is similar to vstack and used to layout the form fields
            rx.flex(
                rx.input(
                    placeholder="User Name",
                    name="name",
                    required=True,
                ),
                rx.input(
                    placeholder="user@reflex.dev",
                    name="email",
                ),
                rx.select(
                    ["Male", "Female"],
                    placeholder="Male",
                    name="gender",
                ),
                rx.flex(
                    rx.dialog.close(
                        rx.button(
                            "Cancel",
                            variant="soft",
                            color_scheme="gray",
                        ),
                    ),
                    rx.dialog.close(
                        rx.button("Submit", type="submit"),
                    ),
                    spacing="3",
                    justify="end",
                ),
                direction="column",
                spacing="4",
            ),
            on_submit=State3.add_user,
            reset_on_submit=False,
        ),
        # max_width is used to limit the width of the dialog
        max_width="450px",
    ),
)
```

U ovom trenutku imamo aplikaciju koja vam omogućava da dodate korisnike na tabelu popunjavanjem forme. Forma se postavlja u dijalog koji se može otvoriti klikom na dugme "Dodaj korisnika". Mi promenimo ime komponente iz forme da biste dodaj_customer_button i ažurirali to u našoj komponenti index. Do sada je puna aplikacija i kod je ispod.

```py
class User(rx.Base):
    """The user model."""
    name: str
    email: str
    gender: str

class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))

def show_user(user: User):
    """Show a person in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
    )

def add_customer_button() -> rx.Component:
    return rx.dialog.root(
        rx.dialog.trigger(
            rx.button(
                rx.icon("plus", size=26),
                rx.text("Add User", size="4"),
            ),
        ),
        rx.dialog.content(
            rx.dialog.title(
                "Add New User",
            ),
            rx.dialog.description(
                "Fill the form with the user's info",
            ),
            rx.form(
                rx.flex(
                    rx.input(
                        placeholder="User Name",
                        name="name",
                        required=True,
                    ),
                    rx.input(
                        placeholder="user@reflex.dev",
                        name="email",
                    ),
                    rx.select(
                        ["Male", "Female"],
                        placeholder="Male",
                        name="gender",
                    ),
                    rx.flex(
                        rx.dialog.close(
                            rx.button(
                                "Cancel",
                                variant="soft",
                                color_scheme="gray",
                            ),
                        ),
                        rx.dialog.close(
                            rx.button(
                                "Submit", type="submit"
                            ),
                        ),
                        spacing="3",
                        justify="end",
                    ),
                    direction="column",
                    spacing="4",
                ),
                on_submit=State.add_user,
                reset_on_submit=False,
            ),
            max_width="450px",
        ),
    )

def index() -> rx.Component:
    return rx.vstack(
        add_customer_button(),
        rx.table.root(
            rx.table.header(
                rx.table.row(
                    rx.table.column_header_cell("Name"),
                    rx.table.column_header_cell("Email"),
                    rx.table.column_header_cell("Gender"),
                ),
            ),
            rx.table.body(
                rx.foreach(State.users, show_user),
            ),
            variant="surface",
            size="3",
        ),
    )
```

### Crtanje podataka u grafikonu

Poslednji deo ovog vodiča je da se korisnički podaci ugrade u grafikon. Koristićemo ugrađenu grafičku biblioteku Refleksa `Recharts`.

#### Transformacija podataka za graf

Komponente grafikona u Refleksu očekuju da će preuzeti listu rečnika. Svaki rečnik predstavlja podatke o podacima na grafikonu i sadrži vrednosti X i Y. Napravićemo novi event hendler u klasi `state` pod nazivom "transform_data" da transformišete korisničke podatke u format koji očekuju grafičke komponente. Moramo takođe stvoriti novu `state` promenljivu pod nazivom "users_for_graph" za čuvanje transformisanih podataka koji će se koristiti za pružanje grafikona.

```py
from collections import Counter

class State(rx.State):
    users: list[User] = []
    users_for_graph: list[dict] = []

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))
        self.transform_data()

    def transform_data(self):
        """Transform user gender group data into a format suitable for visualization in graphs."""
        # Count users of each gender group
        gender_counts = Counter(
            user.gender for user in self.users
        )

        # Transform into list of dict so it can be used in the graph
        self.users_for_graph = [
            {"name": gender_group, "value": count}
            for gender_group, count in gender_counts.items()
        ]
```

Kao što vidimo iznad "transform_data" event hendler koristi `Counter` klasu iz modula `collections` da broji broj korisnika svakog pola. Zatim kreiramo lisu rečnika iz ovoga koje smo postavili na "users" state "users_for_graph".

Napokon možemo videti da kad god dodamo novog korisnika podnesemo formu i pokrenemo "add_user" event hendler, pozivamo "transform_data" da bismo ažurirali "users_for_graph" `state` promenljiva.

### Izdavanje grafikona

Koristimo komponentu `rx.recharts.bar_chart` da izdamo grafikon. Prolazimo kroz `state` promenljivu za naše grafičke podatke kao `data = State.users_for_graph`.Takođe prolazimo u komponentu `rx.recharts.bar` koja predstavlja trake na grafikonu. Komponenta `rx.recharts.bar` uzima `data_key` prop koji je ključ u rečniku podataka koji predstavlja i vrednost bara. `Stroke` i prop `fill` koriste se za postavljanje boje traka.

Komponenta `rx.recharts.bar_component` takođe uzima `rx.recharts.x_axis` i `rx.recharts.y_axis` komponente koje predstavljaju X i Y ose grafikona. `data_key` komponenta `rx.recharts.x_axis` postavljen je na ključ u rečniku podataka koji predstavlja X vrednost trake. Napokon dodajemo širinu i visine props-a da podesimo veličinu grafikona.

```py
def graph():
    return rx.recharts.bar_chart(
        rx.recharts.bar(
            data_key="value",
            stroke=rx.color("accent", 9),
            fill=rx.color("accent", 8),
        ),
        rx.recharts.x_axis(data_key="name"),
        rx.recharts.y_axis(),
        data=State.users_for_graph,
        width="100%",
        height=250,
    )
```

Konačno dodajemo ovu "graph()" komponentu na našu "index()" komponentu tako da se graf prikazuje na strani. Uključen je kod za celokupnu aplikaciju sa uključenim grafikonom. Ako isprobate to, videćete da se graf ažurira kad god dodate novog korisnika na tabelu.

```py
from collections import Counter

class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]
    users_for_graph: list[dict] = []

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))
        self.transform_data()

    def transform_data(self):
        """Transform user gender group data into a format suitable for visualization in graphs."""
        # Count users of each gender group
        gender_counts = Counter(
            user.gender for user in self.users
        )

        # Transform into list of dict so it can be used in the graph
        self.users_for_graph = [
            {"name": gender_group, "value": count}
            for gender_group, count in gender_counts.items()
        ]

def show_user(user: User):
    """Show a person in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
    )

def add_customer_button() -> rx.Component:
    return rx.dialog.root(
        rx.dialog.trigger(
            rx.button(
                rx.icon("plus", size=26),
                rx.text("Add User", size="4"),
            ),
        ),
        rx.dialog.content(
            rx.dialog.title(
                "Add New User",
            ),
            rx.dialog.description(
                "Fill the form with the user's info",
            ),
            rx.form(
                rx.flex(
                    rx.input(
                        placeholder="User Name",
                        name="name",
                        required=True,
                    ),
                    rx.input(
                        placeholder="user@reflex.dev",
                        name="email",
                    ),
                    rx.select(
                        ["Male", "Female"],
                        placeholder="male",
                        name="gender",
                    ),
                    rx.flex(
                        rx.dialog.close(
                            rx.button(
                                "Cancel",
                                variant="soft",
                                color_scheme="gray",
                            ),
                        ),
                        rx.dialog.close(
                            rx.button(
                                "Submit", type="submit"
                            ),
                        ),
                        spacing="3",
                        justify="end",
                    ),
                    direction="column",
                    spacing="4",
                ),
                on_submit=State.add_user,
                reset_on_submit=False,
            ),
            max_width="450px",
        ),
    )

def graph():
    return rx.recharts.bar_chart(
        rx.recharts.bar(
            data_key="value",
            stroke=rx.color("accent", 9),
            fill=rx.color("accent", 8),
        ),
        rx.recharts.x_axis(data_key="name"),
        rx.recharts.y_axis(),
        data=State.users_for_graph,
        width="100%",
        height=250,
    )

def index() -> rx.Component:
    return rx.vstack(
        add_customer_button(),
        rx.table.root(
            rx.table.header(
                rx.table.row(
                    rx.table.column_header_cell("Name"),
                    rx.table.column_header_cell("Email"),
                    rx.table.column_header_cell("Gender"),
                ),
            ),
            rx.table.body(
                rx.foreach(State.users, show_user),
            ),
            variant="surface",
            size="3",
        ),
        graph(),
    )
```

Jedna stvar koju ste možda primetili o vašoj aplikaciji jeste da se grafikon ne pojavljuje u početku kada pokrenete aplikaciju i da morate da dodate korisnika na tabelu da se prvi put pojavi.To se događa zato što se event hendler "transform_data" poziva samo kada se korisnik doda u tabelu.U sledećem odeljku istražićemo rešenje i za to.

### Finalni kod

#### Revizija app.add_page

Na početku ovog tutorial spomenuo smo da je objekat aplikacije potreban za svaku Refleks aplikaciju. Ova funkcija se koristi za dodavanje komponente na stranicu.

`App.add_page` trenutno izgleda kao ova `app.add_page (index)`. Mogli bismo promeniti rutu da se na stranici redovno postavljajući rutu kao što je "/prilagođena ruta", to bi promenilo rutu na <http://localhost:3000/prilagođena_ruta> za ovu stranicu.

Takođe možemo da postavimo naslov koji će se prikazati na kartici pregledača i opisu kao što je prikazano u rezultatima pretrage.

Da bismo rešili problem koji smo imali gore u našem grafikonu, da se ne učitava kada se stranica učitava, možemo da koristimo `on_load` unutar aplikacije da biste nazvali "transformr_data" event hendler za event kada se stranica učitava. Ovo bi izgledalo kao `onload = state.transform_data`.Ispod pogledajte kako bi izgledala naša `app.add_page`, sa nekim od gore navedenih promena.

```py
app.add_page(
    index,
    title="Customer Data App",
    description="A simple app to manage customer data.",
    on_load=State.transform_data,
)
```

#### Revizija app = rx.app()

Na početku tutoriala spominjali smo i da smo definisali našu aplikaciju koristeći `app = rx.app()`.Takođe možemo da prođemo sa nekim propsima u komponentu rx.app da prilagodimo aplikaciju.

Najvažnija je tema koja vam omogućava da prilagodite izgled i osećaj aplikacije. Prop `theme` je uzima `rx.theme` komponentu koja ima nekoliko propsova koji se mogu postaviti.

`Radius` prop postavlja vrednost globalne promenljive `radius` za aplikaciju koju je nasledjuju sve komponente koje imaju props `radius`. Može se prepisati lokalno za određenu komponentu, ručnim postavljanjem vrednosti props `radius` komponente.

`accent_color` prop postavlja boju aplikacije.

Da biste videli ostale props koji se mogu podesiti na nivou aplikacije pogledajte dokumentaciju.

```py
app = rx.App(
    theme=rx.theme(radius="full", accent_color="grass"),
)
```

Nažalost, u ovom tutorialu, to zapravo ne možemo primeniti na primeru uživo na stranici, ali ako kopirate i zalepite kod ispod u Refleks aplikaciju lokalno, možete je videti u akciji.

## Zaključak

Napokon da napravimo neke konačne stilske ispravke na našu aplikaciju. Dodaćemo neki lebdeći stil na redove tabele i sredinu tabele u "show_user" sa stilom = \ {"_ Hover": {"BG": rk.color ("siva", 3)}}, align = "Centar".

Pored toga, dodaćemo nešto širine = "100%" i Align = "Center" na "index()" komponentu da centrira stavke na stranici i osiguramo da se proteže punu širinu strane.

Pogledajte potpuni kod i interaktivnu aplikaciju ispod:

```py
import reflex as rx
from collections import Counter

class User(rx.Base):
    """The user model."""

    name: str
    email: str
    gender: str

class State(rx.State):
    users: list[User] = [
        User(
            name="Danilo Sousa",
            email="danilo@example.com",
            gender="Male",
        ),
        User(
            name="Zahra Ambessa",
            email="zahra@example.com",
            gender="Female",
        ),
    ]
    users_for_graph: list[dict] = []

    def add_user(self, form_data: dict):
        self.users.append(User(**form_data))
        self.transform_data()

    def transform_data(self):
        """Transform user gender group data into a format suitable for visualization in graphs."""
        # Count users of each gender group
        gender_counts = Counter(
            user.gender for user in self.users
        )

        # Transform into list of dict so it can be used in the graph
        self.users_for_graph = [
            {"name": gender_group, "value": count}
            for gender_group, count in gender_counts.items()
        ]

def show_user(user: User):
    """Show a user in a table row."""
    return rx.table.row(
        rx.table.cell(user.name),
        rx.table.cell(user.email),
        rx.table.cell(user.gender),
        style={"_hover": {"bg": rx.color("gray", 3)}},
        align="center",
    )

def add_customer_button() -> rx.Component:
    return rx.dialog.root(
        rx.dialog.trigger(
            rx.button(
                rx.icon("plus", size=26),
                rx.text("Add User", size="4"),
            ),
        ),
        rx.dialog.content(
            rx.dialog.title(
                "Add New User",
            ),
            rx.dialog.description(
                "Fill the form with the user's info",
            ),
            rx.form(
                rx.flex(
                    rx.input(
                        placeholder="User Name",
                        name="name",
                        required=True,
                    ),
                    rx.input(
                        placeholder="user@reflex.dev",
                        name="email",
                    ),
                    rx.select(
                        ["Male", "Female"],
                        placeholder="male",
                        name="gender",
                    ),
                    rx.flex(
                        rx.dialog.close(
                            rx.button(
                                "Cancel",
                                variant="soft",
                                color_scheme="gray",
                            ),
                        ),
                        rx.dialog.close(
                            rx.button(
                                "Submit", type="submit"
                            ),
                        ),
                        spacing="3",
                        justify="end",
                    ),
                    direction="column",
                    spacing="4",
                ),
                on_submit=State.add_user,
                reset_on_submit=False,
            ),
            max_width="450px",
        ),
    )

def graph():
    return rx.recharts.bar_chart(
        rx.recharts.bar(
            data_key="value",
            stroke=rx.color("accent", 9),
            fill=rx.color("accent", 8),
        ),
        rx.recharts.x_axis(data_key="name"),
        rx.recharts.y_axis(),
        data=State.users_for_graph,
        width="100%",
        height=250,
    )

def index() -> rx.Component:
    return rx.vstack(
        add_customer_button(),
        rx.table.root(
            rx.table.header(
                rx.table.row(
                    rx.table.column_header_cell("Name"),
                    rx.table.column_header_cell("Email"),
                    rx.table.column_header_cell("Gender"),
                ),
            ),
            rx.table.body(
                rx.foreach(State.users, show_user),
            ),
            variant="surface",
            size="3",
            width="100%",
        ),
        graph(),
        align="center",
        width="100%",
    )

app = rx.App(
    theme=rx.theme(radius="full", accent_color="grass"),
)

app.add_page(
    index,
    title="Customer Data App",
    description="A simple app to manage customer data.",
    on_load=State.transform_data,
)
```

I to je to za vaš prvi tutorijal kontrolne table. U ovom tutorialu smo stvorili:

- Tabelu za prikazivanje podataka korisnika
- Formu za dodavanje novih korisnika na tabelu
- Dijalog za prikazivanje forme
- Grafikon za vizualizaciju korisničkih podataka

Pored gore navedenog imamo

- Klasu `state` da vam omogući da pokažete dinamičke podatke koji se vremenom menjaju
- `Event hendlere` koji vam omogućavaju da vaš aplikaciju učinite interaktivnom i odgovorite na
  korisničke akcije
- Dodat je stil u aplikaciju kako bi izgledala bolje.

[Sadržaj](00_sadrzaj.md)
