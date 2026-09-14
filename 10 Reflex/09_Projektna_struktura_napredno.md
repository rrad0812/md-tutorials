
# Struktura projekta (napredno)

[Sadržaj](00_sadrzaj.md)

## Modul aplikacije

Refleks uvozi glavni modul aplikacije na osnovu `app_name` iz konfiguracije, koja mora definisati global na nivou modula nazvan `app` kao instanca `rx.App`.

Glavni modul aplikacije je odgovoran za uvoz svih ostalih modula koji čine aplikaciju i definisanje `app = rx.App()`.

Svi ostali moduli koji sadrže stranice, stanje i modele MORAJU biti uvezeni od strane glavnog modula ili paketa aplikacije da bi ih Reflex uključio u kompajlirani izlaz.

## Razbijanje aplikacije na manje delove

Kako se aplikacije skaliraju, efikasna organizacija je ključna. To se postiže razbijanjem aplikacije na manje, upravljive module i njihovim organizovanjem u logičke pakete koji izbegavaju kružne zavisnosti.

U sledećoj dokumentaciji biće aplikacija sa `app_name` = `example_big_app` Glavni modul bi bio `example_big_app/example_big_app.py`.

U odeljku "Sve zajedno“ nalazi se vizuelni prikaz strukture direktorijuma projekta koji pomaže u praćenju, zajedno sa primerima ispod.

### Paket stranica

Sve složene aplikacije će imati više stranica, pa se preporučuje da se kreira `example_big_app/pages` kao paket.

- Ovaj paket treba da sadrži jedan modul po stranici u aplikaciji.
- Ako određena stranica zavisi od stanja, podstanje treba da bude definisano u
  istom modulu kao i stranica.
- Funkcija vraćanja stranice treba da bude ukrašena `rx.page()` kako bi se
  dodala kao ruta u aplikaciji.

```py
import reflex as rx
from ..state import AuthState

class LoginState(AuthState):
    @rx.event
    def handle_submit(self, form_data):
        self.logged_in = authenticate(
            form_data["username"], form_data["password"]
        )

def login_field(name: str, **input_props):
    return rx.hstack(
        rx.text(name.capitalize()),
        rx.input(name=name, **input_props),
        width="100%",
        justify="between",
    )

@rx.page(route="/login")
def login():
    return rx.card(
        rx.form(
            rx.vstack(
                login_field("username"),
                login_field("password", type="password"),
                rx.button("Login"),
                width="100%",
                justify="center",
            ),
            on_submit=LoginState.handle_submit,
        ),
    )
```

### Šabloniranje

Većina aplikacija održava dosledan raspored i strukturu na svim stranicama. Definisanje ove zajedničke strukture u posebnom modulu olakšava deljenje i ponovnu upotrebu prilikom kreiranja pojedinačnih stranica.

Najbolje prakse

- Izdvojite uobičajene elemente korisničkog interfejsa u funkciju koja vraća komponentu.
- Ako funkcija prihvata funkciju koja vraća komponentu, može se koristiti kao dekorator kao što je prikazano ispod.

```py
from typing import Callable
import reflex as rx

from .components.menu import menu
from .components.navbar import navbar

def template(
    page: Callable[[], rx.Component],
) -> rx.Component:
    return rx.vstack(
        navbar(),
        rx.hstack(
            menu(),
            rx.container(page()),
        ),
        width="100%",
    )
```

Dekorator `@template` treba da se pojavi ispod `@rx.page` dekoratora i iznad funkcije za vraćanje stranice. Pogledajte primer koda stranice "Post".

### Upravljanje stanjem

Većina stranica će koristiti `State` u nekom svojstvu. Trebalo bi da izbegavate dodavanje promenljivih (var) deljenom stanju koje će se koristiti samo na jednoj stranici. Umesto toga, definišite novu podklasu `state` rx.Statei držite je u istom modulu kao i stranica.

### Pristup drugim klasama stanja

Od verzije Reflex 0.4.3, svaki obrađivač događaja može dobiti pristup instanci bilo kog drugog podstanja putem `get_state` API-ja. Sa praktične perspektive, to znači da se stanje može podeliti na manje delove bez potrebe za složenom hijerarhijom nasleđivanja za deljenje pristupa drugim stanjima.

U prethodnim izdanjima, ako je aplikacija želela da sačuva podešavanja `SettingsState` sa stranicom ili komponentom radi njihove izmene, svako drugo stanje sa obrađivačem događaja kojem je potreban pristup tim podešavanjima moralo bi da nasleđuje od `SettingsState`, čak i ako je drugo stanje uglavnom ortogonalno. Drugo stanje bi sada uvek moralo da učita podešavanja, čak i za obrađivače događaja kojima nije potreban pristup.

Bolja strategija je da se željeno stanje učita na zahtev samo od obrađivača događaja kome je potreban pristup podstanju.

### Komponenta podešavanja

```py
import reflex as rx

class SettingsState(rx.State):
    refresh_interval: int = 15
    auto_update: bool = True
    prefer_plain_text: bool = True
    posts_per_page: int = 20


def settings_dialog():
    return rx.dialog(...)
```

### Stranica Post

Ova stranica se učitava kako `SettingsState` bi se odredilo koliko objava treba prikazati po stranici i koliko često treba osvežavati.

```py
import reflex as rx

from ..models import Post
from ..template import template
from ..components.settings import SettingsState

class PostsState(rx.State):
    refresh_tick: int
    page: int
    posts: list[Post]

    @rx.event
    async def on_load(self):
        settings = await self.get_state(SettingsState)
        if settings.auto_update:
            self.refresh_tick = (
                settings.refresh_interval * 1000
            )
        else:
            self.refresh_tick = 0

    @rx.event
    async def tick(self, _):
        settings = await self.get_state(SettingsState)
        with rx.session() as session:
            q = (
                Post.select()
                .offset(self.page * settings.posts_per_page)
                .limit(settings.posts_per_page)
            )
            self.posts = q.all()

    @rx.event
    def go_to_previous(self):
        if self.page > 0:
            self.page = self.page - 1

    @rx.event
    def go_to_next(self):
        if self.posts:
            self.page = self.page + 1

@rx.page(route="/posts", on_load=PostsState.on_load)
@template
def posts():
    return rx.vstack(
        rx.foreach(PostsState.posts, post_view),
        rx.hstack(
            rx.button(
                "< Prev", on_click=PostsState.go_to_previous
            ),
            rx.button(
                "Next >", on_click=PostsState.go_to_next
            ),
            justify="between",
        ),
        rx.moment(
            interval=PostsState.refresh_tick,
            on_change=PostsState.tick,
            display="none",
        ),
        width="100%",
    )
```

### Zajednička stanja

Zajednička stanja i podstanja koja dele više stranica ili komponenti treba implementirati u posebnom modulu kako bi se izbegao kružni uvoz. Ovaj modul ne bi trebalo da uvozi druge module u aplikaciji.

### Ponovna upotreba komponenti

Primarni mehanizam za ponovnu upotrebu komponenti u Reflex-u je definisanje funkcije koja vraća komponentu, a zatim jednostavno pozivanje te funkcije tamo gde je ta funkcionalnost potrebna.

Funkcije komponenti obično ne bi trebalo da uzimaju klase stanja kao argumente, već bi radije uvozile potrebno stanje i direktno pristupale promenljivim na klasi.

### Funkcije za pamćenje radi poboljšanja performansi

U velikoj aplikaciji, ako komponenta ima mnogo podkomponenti ili se koristi na velikom broju mesta, može poboljšati performanse kompajliranja i izvršavanja kako bi se funkcija memorizovala sa dekoratorom `@lru_cache`.

Da biste memoizovali foo komponentu i izbegli njeno ponovno kreiranje više puta, jednostavno dodajte `@lru_cache` u definiciju funkcije, a komponenta će biti kreirana samo jednom po jedinstvenom skupu argumenata.

## Primer_velike_aplikacije/komponente

Ovaj paket sadrži delove aplikacije koji se mogu ponovo koristiti, na primer zaglavlja, podnožja i menije. Ako određena komponenta zahteva stanje, podstanje može biti definisano u istom modulu za lokalnost. Bilo koje podstanje definisano u modulu komponente treba da sadrži samo polja i obrađivače događaja koji se odnose na tu pojedinačnu komponentu.

### Spoljne komponente

Reflex 0.4.3 je uveo podršku za reflex componentCLI komande , što olakšava objedinjavanje uobičajenih funkcionalnosti za objavljivanje na PyPI kao samostalni Python paket koji se može instalirati i koristiti u bilo kojoj Reflex aplikaciji.

Prilikom pakovanja npm komponenti ili drugih samostalnih delova funkcionalnosti, može biti korisno premestiti ovu složenost van same aplikacije radi lakšeg održavanja i ponovne upotrebe u drugim aplikacijama.

### Modeli baza podataka

Preporučuje se implementacija svih modela baze podataka u jednoj datoteci kako bi se lakše definisali odnosi i razumela cela šema.

Međutim, ako je šema veoma velika, možda bi imalo smisla imati modelspaket sa pojedinačnim modelima definisanim u njihovim sopstvenim modulima.

U svakom slučaju, definisanje modela odvojeno omogućava bilo kojoj stranici ili komponenti da ih uveze i koristi bez kružnog uvoza.

### Paket najvišeg nivoa

Ovo je odlično mesto za uvoz svih stanja, modela i stranica koje bi trebalo da budu deo aplikacije. Tipično, komponente i pomoćne elemente nije potrebno uvoziti, jer će ih uvesti stranice koje ih koriste (inače bi bili neiskorišćeni).

```py
from . import state, models
from .pages import (
    index,
    login,
    post,
    product,
    profile,
    schedule,
)

__all__ = [
    "state",
    "models",
    "index",
    "login",
    "post",
    "product",
    "profile",
    "schedule",
]
```

Ako neke stranice nisu ovde uvezene, one neće biti kompajlirane kao deo aplikacije.
primer_velike_aplikacije/primer_velike_aplikacije.py

Ovo je glavni modul aplikacije. Pošto je sve ostalo definisano u drugim modulima, ova datoteka postaje veoma jednostavna.

```py
import reflex as rx

app = rx.App()
```

### Upravljanje datotekama

Postoje dve kategorije nekodnih sredstava (mediji, fontovi, stilski listovi, dokumenti) koje se obično koriste u Reflex aplikaciji.

### Sredstva

Direktorijum `assets` se koristi za statičke datoteke kojima bi trebalo da se pristupi u odnosu na koren frontenda (podrazumevani port 3000). Kada se aplikacija rasporedi u produkcijskom režimu, promene u direktorijumu resursa NEĆE biti dostupne tokom izvršavanja!

Prilikom referenciranja sredstva, uvek koristite kosu crtu koja počinje unapred, kako bi se sredstvo moglo razrešiti bez obzira na rutu stranice na kojoj se može pojaviti.

### Otpremljene_datoteke

Ako aplikacija treba da dinamički učini datoteke dostupnim tokom izvršavanja, preporučuje se da se ciljni direktorijum podesi preko `REFLEX_UPLOADED_FILES_DIR` promenljive okruženja (podrazumevano `./uploaded_files`), da se datoteke zapišu relativno u odnosu na putanju koju vraća `rx.get_upload_dir()` i da se kreiraju radne veze preko `rx.get_upload_url` (`relative_path`).

Otpremljene datoteke se prikazuju sa bekenda (podrazumevani port 8000) putem `/_upload/<relative_path>`

### Sve to zajedno

Na osnovu prethodne diskusije, preporučeni raspored projekta izgleda ovako.

```sh
example-big-app/
├─ assets/
├─ example_big_app/
│  ├─ components/
│  │  ├─ __init__.py
│  │  ├─ auth.py
│  │  ├─ footer.py
│  │  ├─ menu.py
│  │  ├─ navbar.py
│  ├─ pages/
│  │  ├─ __init__.py
│  │  ├─ index.py
│  │  ├─ login.py
│  │  ├─ posts.py
│  │  ├─ product.py
│  │  ├─ profile.py
│  │  ├─ schedule.py
│  ├─ __init__.py
│  ├─ example_big_app.py
│  ├─ models.py
│  ├─ state.py
│  ├─ template.py
├─ uploaded_files/
├─ requirements.txt
├─ rxconfig.py
```

## Zaključak

- Kao i svaki drugi Pajton projekat, podelite aplikaciju na module i pakete
  kako bi baza koda bila organizovana i laka za upravljanje.
- Korišćenje manjih modula i paketa olakšava ponovnu upotrebu komponenti i
  stanja u aplikaciji bez uvođenja kružnih zavisnosti.
- Kreirajte pojedinačne funkcije da biste obuhvatili jedinice funkcionalnosti i
  ponovo ih koristili gde je potrebno.

[Sadržaj](00_sadrzaj.md)
