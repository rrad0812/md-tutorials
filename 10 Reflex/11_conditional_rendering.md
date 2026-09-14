
# Uslovni rendering

[Sadržaj](00_sadrzaj.md)

Ako se prisetimo osnova Reflex-a, ne možemo koristiti Python `if/else` naredbe when referenciramo `state vars` u Reflex-u. Umesto toga, koristimo `rx.cond` komponentu za uslovni rendering komponenti ili postavimo props na osnovu vrednosti `state var`-a.

Ispod je jednostavan primer koji pokazuje kako da prelazite između dve tekstualne komponente proverom vrednosti prikaza var stanja "show".

```py
class CondSimpleState(rx.State):
    show: bool = True

    @rx.event
    def change(self):
        self.show = not (self.show)

def cond_simple_example():
    return rx.vstack(
        rx.button("Toggle", on_click=CondSimpleState.change),
        rx.cond(
            CondSimpleState.show,
            rx.text("Text 1", color="blue"),
            rx.text("Text 2", color="red"),
        ),
    )
```

Ako je prikazano `True` tada se prva komponenta renderuje (u ovom slučaju plavi tekst). Inače druga komponenta se renderuje (u ovom slučaju crveni tekst).

## Uslovni props

Takođe možete postaviti props uslovno koristeći `rx.cond`. U ovom primeru postavljamo "color_sheme" prop `slider` komponente na osnovu `state var` "value".

```py
class PropCondState(rx.State):
    value: int

    @rx.event
    def set_end(self, value: list[int | float]):
        self.value = value[0]


def cond_prop():
    return rx.slider(
        default_value=[50],
        on_value_commit=PropCondState.set_end,
        color_scheme=rx.cond(
            PropCondState.value > 50, 
            "green", "pink"),
        width="100%",
    )
```

## Var operacije

Možete koristiti var operacije sa `cond` komponentom za složeije odnose.

## Višestruke uslovne izjave

`rx.match` komponenta u Reflex-u pruža moćnu alternativu ugneždjenim `rx.cond` za rukovanje višestrukim uslovnim izjavama i podudaranje strukturnih obrazaca. Ova komponenta vam omogućava da rukujete sa više uslova i njihovih povezanih komponenti na čišći i čitljiviji način u poređenju sa ugnežđenim `rx.cond` strukturama.

```py
from typing import List
import reflex as rx

class MatchState(rx.State):
    cat_breed: str = ""
    animal_options: List[str] = [
        "persian",
        "siamese",
        "maine coon",
        "ragdoll",
        "pug",
        "corgi",
    ]

    @rx.event
    def set_cat_breed(self, breed: str):
        self.cat_breed = breed

def match_demo():
    return rx.flex(
        rx.match(
            MatchState.cat_breed,
            ("persian", rx.text("Persian cat selected.")),
            ("siamese", rx.text("Siamese cat selected.")),
            ("maine coon", rx.text("Maine Coon cat selected.")),
            ("ragdoll", rx.text("Ragdoll cat selected.")),
            rx.text("Unknown cat breed selected."),
        ),
        rx.select(
            [
                "persian",
                "siamese",
                "maine coon",
                "ragdoll",
                "pug",
                "corgi",
            ],
            value=MatchState.cat_breed,
            on_change=MatchState.set_cat_breed,
        ),
        direction="column",
        gap="2",
    )
```

[Sadržaj](00_sadrzaj.md)
