
# Props

[Sadržaj](00_sadrzaj.md)

Props menjaju ponašanje i izgled komponente. Oni se prosleđuju kao argumenti ključne reči komponenti.

## Props komponenti

Postoje props koji se dele između svih komponenti, ali svaka komponenta takođe može da definiše sopstvene rekvizite.

Na primer, `rx.image` komponenta ima `src` prop koji specificira URL slike za prikaz i `alt` prop koji specificira alternativni tekst za sliku.

```py
rx.image(
    src="https://reflex.dev/logo.jpg",
    alt="Reflex Logo",
)
```

Vidi `rx.image` referencnu stranu za primer.

### Zajednički props

Komponente podržavaju mnoge standardne HTML svojstva kao props. Na primer: HTML `id` imovine je izložena direktno kao prop `id`. HTML `className` imovina je izložen kao prop `class_name`.

```py
rx.box(
    "Hello World",
    id="box-id",
    class_name=[
        "class-name-1",
        "class-name-2",
    ],
)
```

U gornjem primeru, `class_name` prop `rx.box` komponente je dodeljena lista naziva klasa. Ovo znači da `rx.box` komponenta će biti stilizovana sa CSS klasama "class-name-1" i "class-name-2".

### Stil props

Pored props specifičnih za komponente, većina ugrađenih komponenti podržava čitav niz stilskih props. Možete koristiti bilo koje CSS svojstvo za stilizovanje komponente.

```py
rx.button(
    "Fancy Button",
    border_radius="1em",
    box_shadow="rgba(151, 65, 252, 0.8) 0 15px 30px -10px",
    background_image="linear-gradient(144deg,#AF40FF,#5B42F3 50%,#00DDEB)",
    box_sizing="border-box",
    color="white",
    opacity=1,
)
```

### Povezivanje props sa state

Reflex aplikacije definišu `State` klase koje drže varijable koje se mogu promeniti u vremenu.

Stanje može biti izmenjeno:

- kao odgovor na stvari kao što je unos korisnika,
- kao što je klik na dugme ili
- kao odgovor na događaje kao što je učitavanje stranice.

`State vars` može biti vezan za komponente, tako da korisnički interfejs uvek odražava trenutno stanje aplikacije.

Pokušajte da kliknete na dugme ispod da promenite njenu boju.

```py
class PropExampleState(rx.State):
    text: str = "Hello World"
    color: str = "red"

    @rx.event
    def flip_color(self):
        if self.color == "red":
            self.color = "blue"
        else:
            self.color = "red"

def index():
    return rx.button(
        PropExampleState.text,
        color_scheme=PropExampleState.color,
        on_click=PropExampleState.flip_color,
    )
```

U ovom primeru "color_scheme" prop je povezan sa "color" `state var`.

Kada "flip_color" event handler se pozove, "color" var je ažuriran, i `color_scheme` prop je ažuraran da odgovara.

[Sadržaj](00_sadrzaj.md)
