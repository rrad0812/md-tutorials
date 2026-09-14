
# Renderovanje iterabilnih objekata

[Sadržaj](00_sadrzaj.md)

Rekli smo da ne možemo koristiti Python `for loops` kada referenciramo referenciramo `state vars` u Reflexu. Umesto toga, koristimo `rx.foreach` komponentu za renderovanje komponenti sa kolekcijama podataka.

Za dinamički sadržaj koji bi trebao da skroluje automatski da prikaže najnovije stavke, koristimo auto scroll komponentu zajdno sa `rx.foreach`.

```py
class IterState(rx.State):
    color: list[str] = [
        "red",
        "green",
        "blue",
    ]

def colored_box(color: str):
    return rx.button(color, background_color=color)

def dynamic_buttons():
    return rx.vstack(
        rx.foreach(IterState.color, colored_box),
    )
```

Ovde je isti primer sa `lambda` funkcijom.

```py
def dynamic_buttons():
    return rx.vstack(
        rx.foreach(
            IterState.color,
            lambda color: colored_box(color),
        ),
    )
```

Možete koristiti `lambda` funnkciju direktno sa komponentama be definisanja posebnih funkcija.

```py
def dynamic_buttons():
    return rx.vstack(
        rx.foreach(
            IterState.color,
            lambda color: rx.button(
                color, background_color=color
            ),
        ),
    )
```

U ovom prvom jednostavnom primeru prolazimo kroz listu boja i prikazujemo dinamički broj dugmadi.

Prvi argument funkcije `rx.foreach` je var stanja kroz koji želite da iterirate. Drugi argument je funkcija koja uzima stavku iz podataka i vraća komponentu. U ovom slučaju, `colored_box` funkcija uzima boju i vraća dugme sa tom bojom.

## For vs Foreach

- Uobičajeno **for** petlju koristite kada iterirate preko `konstanti`.
- Koristite petlju **foreach** kada iterirate preko `state vars`.

Gornji primer je mogao biti napisan korišćenjem obične Python for petlje, pošto su podaci konstantni.

```py
colors = ["red", "green", "blue"]

def dynamic_buttons_for():
    return rx.vstack(
        [colored_box(color) for color in colors],
    )
```

Međutim, čim vam je potrebno da podaci budu dinamični, morate ih koristiti `rx.foreach`.

```py
class DynamicIterState(rx.State):
    color: list[str] = [
        "red",
        "green",
        "blue",
    ]

    def add_color(self, form_data):
        self.color.append(form_data["color"])

def dynamic_buttons_foreach():
    return rx.vstack(
        rx.foreach(DynamicIterState.color, colored_box),
        rx.form(
            rx.input(name="color", placeholder="Add a color"),
            rx.button("Add"),
            on_submit=DynamicIterState.add_color,
        ),
    )
```

## Render funkcija

Funkcija za prikazivanje svake stavke može se definisati ili kao posebna funkcija ili kao lambda funkcija. U primeru ispod, mi definišemo funkciju "colored_box" odvojeno i prosledimo je u`rx.foreach` funkciju.

```py
class IterState2(rx.State):
    color: list[str] = [
        "red",
        "green",
        "blue",
    ]

def colored_box(color: rx.Var[str]):
    return rx.button(color, background_color=color)

def dynamic_buttons2():
    return rx.vstack(
        rx.foreach(IterState2.color, colored_box),
    )
```

Primetite da je naznaka tipa za parametar "color" u funkciji "colored_box". `rx.Var[str]` (a ne samo `str`). To je zato što `rx.foreach` funkcija passes stavku kao `Var object`, što je omotač oko aktuelne vrednosti. To je ono što nam omogućava da kompajliramo frontend bez poznavanja stvarne vrednosti var stanja (koja je poznata samo u vreme izvođenja).

## Enumeracija iterabilnih objekata

Funkcija takođe može uzeti indeks kao drugi argument, što znači da možemo da nabrajamo niz podataka kao što je prikazano u primeru ispod.

```py
class IterIndexState(rx.State):
    color: list[str] = [
        "red",
        "green",
        "blue",
    ]

def create_button(color: rx.Var[str], index: int):
    return rx.box(
        rx.button(f"{index + 1}. {color}"),
        padding_y="0.5em",
    )

def enumerate_foreach():
    return rx.vstack(
        rx.foreach(IterIndexState.color, create_button),
    )
```

Ovde je isti priimer korišćenjem `lambda` funkcije.

```py
def enumerate_foreach():
    return rx.vstack(
        rx.foreach(
            IterIndexState.color,
            lambda color, index: create_button(
                color, index
            ),
        ),
    )
```

## Iteracija rečnika

Možemo iterirati kroz `dict` koristeći `foreach`. Kada je `dict` prosleđen funkciji koja renderuje svaka item, to je prezentovano kao lista `key-value` parova kao:

```py
[("sky", "blue"), ("balloon", "red"), ("grass", "green")].
```

```py
class SimpleDictIterState(rx.State):
    color_chart: dict[str, str] = {
        "sky": "blue",
        "balloon": "red",
        "grass": "green",
    }

def display_color(color: list):
    # color is presented as a list key-value pairs [("sky", "blue"), ("balloon", "red"), ("grass", "green")]
    return rx.box(rx.text(color[0]), bg=color[1], padding_x="1.5em")

def dict_foreach():
    return rx.grid(
        rx.foreach(
            SimpleDictIterState.color_chart,
            display_color,
        ),
        columns="3",
    )
```

## Ugneždjeni primeri

`rx.foreach` može biti korišćen sa umetnutim `state vars`. Ovde mi koristimo umetnute `foreach` komponente da renderujemo umetnute `state vars`. "rx.foreach(project["technologies"], get_badge)" unutar "project_item" funkcije, renderuje `dict` vrednosti koje su tipa liste `list`. "rx.box(rx.foreach(NestedStateFE.projects, project_item))" unutar "projects_example" funkcije renderuje svaki `dict` unutar svih `state var` projekata.

```py
class NestedStateFE(rx.State):
    projects: list[dict[str, list]] = [
        {
            "technologies": [
                "Next.js",
                "Prisma",
                "Tailwind",
                "Google Cloud",
                "Docker",
                "MySQL",
            ]
        },
        {"technologies": ["Python", "Flask", "Google Cloud", "Docker"]},
    ]

def get_badge(technology: rx.Var[str]) -> rx.Component:
    return rx.badge(technology, variant="soft", color_scheme="green")

def project_item(project: rx.Var[dict[str, list]]) -> rx.Component:
    return rx.box(
        rx.hstack(rx.foreach(project["technologies"], get_badge)),
    )

def projects_example() -> rx.Component:
    return rx.box(rx.foreach(NestedStateFE.projects, project_item))
```

Ako želite primer gde nisu sve vrednosti u `dict`-u istog tipa, pogledajte primer var operacija koristeći `foreach`.

Ovde je sledeći primer kako koristiti `foreach` sa umetnutim strukturama podataka.

```py
class NestedDictIterState(rx.State):
    color_chart: dict[str, list[str]] = {
        "purple": ["red", "blue"],
        "orange": ["yellow", "red"],
        "green": ["blue", "yellow"],
    }

def display_colors(color: rx.Var[tuple[str, list[str]]]):
    return rx.vstack(
        rx.text(color[0], color=color[0]),
        rx.hstack(
            rx.foreach(
                color[1],
                lambda x: rx.box(rx.text(x, color="black"), bg=x),
            )
        ),
    )

def nested_dict_foreach():
    return rx.grid(
        rx.foreach(
            NestedDictIterState.color_chart,
            display_colors,
        ),
        columns="3",
    )
```

## Foreach sa cond

Takođe možemo koristiti `foreach` sa `cond` komponentom.

U ovom primeru definišemo funkciju "render_item". Ova funkcija uzima "item", koristi `cond` da proveri da li je "item" "is_packed". Ako je "is_packed" ona vraća "item_name" sa ✔, a ako nije vraća "item_name". Koristimo `foreach` za iteriranje preko svih "items" u "to_do_list" koristeći "render_item" funkciju.

Sammy's Packing List

- Space suit ✔
- Helmet ✔
- Back Pack

```py
import dataclasses

@dataclasses.dataclass
class ToDoListItem:
    item_name: str
    is_packed: bool

class ForeachCondState(rx.State):
    to_do_list: list[ToDoListItem] = [
        ToDoListItem(item_name="Space suit", is_packed=True),
        ToDoListItem(item_name="Helmet", is_packed=True),
        ToDoListItem(item_name="Back Pack", is_packed=False),
    ]

def render_item(item: rx.Var[ToDoListItem]):
    return rx.cond(
        item.is_packed,
        rx.list.item(item.item_name + " ✔"),
        rx.list.item(item.item_name),
    )

def packing_list():
    return rx.vstack(
        rx.text("Sammy's Packing List"),
        rx.list(rx.foreach(ForeachCondState.to_do_list, render_item)),
    )
```

[Sadržaj](00_sadrzaj.md)
