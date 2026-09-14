
# Tutorijal - Aplikacija AI Chat

[Sadržaj](00_sadrzaj.md)

Ovaj vodič će vas provesti kroz pravljenje AI aplikacije za ćaskanje sa Reflex-om. Ova aplikacija je prilično složena, ali ne brinite - podelićemo je na male korake.

U ovom vodiču ćete naučiti kako da:

- Instalirajte refleks i podesite svoje razvojno okruženje.
- Kreirajte komponente za definisanje i stilizovanje korisničkog interfejsa.
- Koristite stanje da biste svojoj aplikaciji dodali interaktivnost.
- Postavite svoju aplikaciju da biste je delili sa drugima.

## Postavljanje vašeg projekta

Počećemo kreiranjem novog projekta i postavljanjem našeg razvojnog okruženja. Prvo napravite novi direktorijum za svoj projekat i idite do njega.

```sh
mkdir chatapp
cd chatapp
```

Zatim ćemo kreirati virtuelno okruženje za naš projekat. Ovo je opciono, ali se preporučuje. U ovom primeru, koristićemo `venv` da kreiramo naše virtuelno okruženje.

```sh
chatapp $ python3 -m venv venv
$ source venv/bin/activate
```

Sada ćemo instalirati Reflex i kreirati novi projekat. Ovo će kreirati novu strukturu direktorijuma u ​​direktorijumu našeg projekta.

> [!Note]  
> Kada se od vas zatraži da izaberete šablon, izaberite opciju 0 za prazan projekat.

```sh
chatapp $ pip install reflex
chatapp $ reflex init
────────────────────────── Initializing chatapp ──────────────────────────
Success: Initialized chatapp
chatapp $ ls
assets          chatapp         rxconfig.py     venv
```

Možete pokrenuti aplikaciju šablona da biste bili sigurni da sve radi.

```sh
chatapp $ reflex run
────────────────────────── Starting Reflex App ───────────────────────────
Compiling:  ━━━━━━━━━━━━━━━━━━━━━━━━━━━ 100% 1/1 0:00:00
────────────────────────── App Running ───────────────────────────────────
App running at: <http://localhost:3000>
```

Trebalo bi da vidite da vaša aplikacija radi na `http://localhost:3000`.

Reflex takođe pokreće backend server koji upravlja svim stanjima i komunikacijom sa frontendom. Možete testirati da pozadinski server radi tako što ćete otići na `http://localhost:8000/ping`.

Sada kada smo postavili naš projekat, u sledećem odeljku ćemo početi da pravimo našu aplikaciju!

## Osnovni frontend

Počnimo sa definisanjem frontenda za našu aplikaciju za ćaskanje. U Reflex-u, frontend se može podeliti na nezavisne komponente za višekratnu upotrebu. Pogledajte dokumentaciju o komponentama za više informacija.

### Prikažite pitanje i odgovor

Izmenićemo funkciju indeksa u datoteci "chatapp/chatapp.py" da bismo vratili komponentu koja prikazuje jedno pitanje i odgovor.

"chatapp.py"

```py
import reflex as rx

def index() -> rx.Component:
    return rx.container(
        rx.box(
            "What is Reflex?",
            # The user's question is on the right.
            text_align="right",
        ),
        rx.box(
            "A way to build web apps in pure Python!",
            # The answer is on the left.
            text_align="left",
        ),
    )

# Add state and page to the app.
app = rx.App()
app.add_page(index)
```

Komponente mogu biti ugnežđene jedna unutar druge da bi se kreirali složeni rasporedi. Ovde kreiramo roditeljski kontejner koji sadrži dva polja, za pitanje i odgovor.

Takođe dodajemo neke osnovne stilove komponentama. Komponente uzimaju argumente ključne reči, zvane `props`, koji menjaju izgled i funkcionalnost komponente. Koristimo `text_align` prop za poravnavanje teksta levo i desno.

### Ponovna upotreba komponenti

Sada kada imamo komponentu koja prikazuje jedno pitanje i odgovor, možemo je ponovo koristiti za prikaz više pitanja i odgovora. Premestićemo komponentu u zasebnu funkciju `question_answer` i pozvati je iz funkcije indeksa.

```py
def qa(question: str, answer: str) -> rx.Component:
    return rx.box(
        rx.box(question, text_align="right"),
        rx.box(answer, text_align="left"),
        margin_y="1em",
    )

def chat() -> rx.Component:
    qa_pairs = [
        (
            "What is Reflex?",
            "A way to build web apps in pure Python!",
        ),
        (
            "What can I make with it?",
            "Anything from a simple website to a complex web app!",
        ),
    ]
    return rx.box(
        *[
            qa(question, answer)
            for question, answer in qa_pairs
        ]
    )

def index() -> rx.Component:
    return rx.container(chat())
```

### Unos ćaskanja

Sada želimo način da korisnik unese pitanje. Za ovo ćemo koristiti komponentu za unos da bi korisnik dodao tekst i komponentu dugmeta za slanje pitanja.

```py
def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(placeholder="Ask a question"),
        rx.button("Ask"),
    )

def index() -> rx.Component:
    return rx.container(
        chat(),
        action_bar(),
    )
```

### Stilizovanje

Hajde da dodamo malo stilova aplikaciji. Više informacija o stilizovanju možete pronaći u dokumentaciji o stilovima. Da bismo održali naš kod čistim, premestićemo stilove u zasebnu datoteku "chatapp/style.py".

"style.py":

```py
import reflex as rx

# Common styles for questions and answers.
shadow = "rgba(0, 0, 0, 0.15) 0px 2px 8px"
chat_margin = "20%"
message_style = dict(
    padding="1em",
    border_radius="5px",
    margin_y="0.5em",
    box_shadow=shadow,
    max_width="30em",
    display="inline-block",
)

# Set specific styles for questions and answers.
question_style = message_style | dict(
    margin_left=chat_margin,
    background_color=rx.color("gray", 4),
)
answer_style = message_style | dict(
    margin_right=chat_margin,
    background_color=rx.color("accent", 8),
)

# Styles for the action bar.
input_style = dict(
    border_width="1px",
    padding="0.5em",
    box_shadow=shadow,
    width="350px",
)

button_style = dict(
    background_color=rx.color("accent", 10),
    box_shadow=shadow,
)
```

Uvešćemo stilove u "chatapp.py" i koristićemo ih u komponentama. Za sada, aplikacija bi trebalo da izgleda ovako:

"chatapp.py":

```py
import reflex as rx
from chatapp import style

def qa(question: str, answer: str) -> rx.Component:
    return rx.box(
        rx.box(
            rx.text(question, style=style.question_style),
            text_align="right",
        ),
        rx.box(
            rx.text(answer, style=style.answer_style),
            text_align="left",
        ),
        margin_y="1em",
        width="100%",
    )

def chat() -> rx.Component:
    qa_pairs = [
        (
            "What is Reflex?",
            "A way to build web apps in pure Python!",
        ),
        (
            "What can I make with it?",
            "Anything from a simple website to a complex web app!",
        ),
    ]
    return rx.box(
        *[
            qa(question, answer)
            for question, answer in qa_pairs
        ]
    )

def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(
            placeholder="Ask a question",
            style=style.input_style,
        ),
        rx.button("Ask", style=style.button_style),
    )

def index() -> rx.Component:
    return rx.center(
        rx.vstack(
            chat(),
            action_bar(),
            align="center",
        )
    )

app = rx.App()
app.add_page(index)
```

Aplikacija izgleda dobro, ali još uvek nije baš korisna! U sledećem odeljku dodaćemo neke funkcionalnosti aplikaciji.

## Stanje

Sada hajde da učinimo aplikaciju za ćaskanje interaktivnom dodavanjem stanja. Stanje je mesto gde definišemo sve promenljive koje se mogu menjati u aplikaciji i sve funkcije koje ih mogu menjati. Možete naučiti više o stanju u dokumentaciji o stanju.

### Definisanje stanja

Napravićemo novu datoteku pod nazivom "state.py" u direktorijumu "chatapp". Naše stanje će pratiti trenutno pitanje koje se postavlja i istoriju ćaskanja. Takođe ćemo definisati rukovaoca događaja "answer" koji će obraditi trenutno pitanje i dodati odgovor u istoriju ćaskanja.

"state.py":

```py
import reflex as rx

class State(rx.State):

    # Trenutno pitanje koje se postavlja.
    question: str

    # Keep track of the chat history as a list of (question, answer) tuples.
    chat_history: list[tuple[str, str]]

    @rx.event
    def answer(self):
        # Our chatbot is not very smart right now...
        answer = "I don't know!"
        self.chat_history.append((self.question, answer))
```

### Povezivanje stanja sa komponentama

Sada možemo uvesti stanje u "chatapp.py" i referencirati ga u našim frontend komponentama. Izmenićemo komponentu za ćaskanje da koristi stanje umesto trenutnih fiksnih pitanja i odgovora.

"chatapp.py":

```py
from chatapp.state import State

def chat() -> rx.Component:
    return rx.box(
        rx.foreach(
            State.chat_history,
            lambda messages: qa(messages[0], messages[1]),
        )
    )

def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(
            placeholder="Ask a question",
            on_change=State.set_question1,
            style=style.input_style,
        ),
        rx.button(
            "Ask",
            on_click=State.answer,
            style=style.button_style,
        ),
    )
```

Obične Python `for petlje` ne funkcionišu za iteraciju preko promenljivih stanja jer

- se ove vrednosti mogu menjati i
- nisu poznate u vreme kompajliranja.

Umesto toga, koristimo `foreach` komponentu za iteraciju kroz istoriju ćaskanja.

Takođe povezujemo `on_change` događaj unosa sa rukovaocem događaja "set_question", koji će ažurirati promenljivu stanja "question" dok korisnik kuca u unosu. Povezujemo `on_click` događaj dugmeta sa rukovaocem događaja "answer", koji će obraditi pitanje i dodati odgovor u istoriju ćaskanja. Rukovaoc događaja "set_question" je ugrađeni implicitno definisan rukovaoc događaja. Svaka osnovna promenljiva ima jedan. Saznajte više u dokumentaciji o događajima u odeljku "Setters".

### Čišćenje unosa

Trenutno se unos ne čisti nakon što korisnik klikne na dugme. Možemo to popraviti povezivanjem vrednosti unosa sa question, pomoću "value=State.question", i čišćenjem kada pokrenemo rukovaoca događaja za answer, sa "self.question = ''".

**chatapp.py**:

```py
def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(
            value=State.question,
            placeholder="Ask a question",
            on_change=State.set_question2,
            style=style.input_style,
        ),
        rx.button(
            "Ask",
            on_click=State.answer,
            style=style.button_style,
        ),
    )

# state.py
@rx.event
def answer(self):
    # Our chatbot is not very smart right now...
    answer = "I don't know!"
    self.chat_history.append((self.question, answer))
    self.question = ""
```

### Strimovanje teksta

Obično se ažuriranja stanja šalju na frontend kada se rukovaoc događaja vrati. Međutim, želimo da strimujemo tekst iz čatbota dok se generiše. Možemo to učiniti yieldom iz rukovaoca događaja. Pogledajte dokumentaciju `yield events` za više informacija.

**state.py**:

```py
import asyncio

async def answer(self):
    # Our chatbot is not very smart right now...
    answer = "I don't know!"
    self.chat_history.append((self.question, ""))

    # Clear the question input.
    self.question = ""
    # Yield here to clear the frontend input before continuing.
    yield

    for i in range(len(answer)):
        # Pause to show the streaming effect.
        await asyncio.sleep(0.1)
        # Add one letter at a time to the output.
        self.chat_history[-1] = (
            self.chat_history[-1][0],
            answer[: i + 1],
        )
        yield
```

U sledećem odeljku, završićemo naš čatbot dodavanjem AI!

## Finalna aplikacija

Koristićemo OpenAI API da bismo dali našem čatbotu malo inteligencije.

### Konfigurisanje OpenAI API ključa

Prvo, uverite se da imate aktivnu OpenAI pretplatu. Zatim instalirajte najnoviji openai paket:

pip install --upgrade openai

### Direktna konfiguracija API-ja u kodu

Ažurirajte datoteku "state.py" da direktno uključi vaš API ključ:

**state.py**:

```py
import os
from openai import AsyncOpenAI

import reflex as rx

# Inicijalizuj OpenAI klijenta
client = AsyncOpenAI(
    api_key="YOUR_OPENAI_API_KEY"
)  # Replace with your actual API key
```

### Korišćenje API-ja

Da biste učinili svoj čatbot inteligentnim, potrebno je povezati se sa API-jem jezičkog modela. Ova sekcija objašnjava kako se integrisati sa OpenAI API-jem da bi se omogućili odgovori vašeg čatbota.

- Prvo, korisnik kuca prompt koji se ažurira preko rukovaoca događaja on_change.
- Zatim, kada je prompt spreman, korisnik može izabrati da ga pošalje klikom
  na dugme Ask što zauzvrat pokreće metodu State.answer unutar naše datoteke state.py.
- Konačno, ako se metoda pokrene, prompt se šalje putem zahteva
  OpenAI klijentu i vraća odgovor koji možemo iskoristiti za ažuriranje istorije ćaskanja!

**chatapp.py**:

```py
def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(
            value=State.question,
            placeholder="Ask a question",
            # on_change event updates the input as the user types a prompt.
            on_change=State.set_question3,
            style=style.input_style,
        ),
        # on_click event triggers the API to send the prompt to OpenAI.
        rx.button(
            "Ask",
            on_click=State.answer,
            style=style.button_style,
        ),
    )
```

**state.py**:

```py
import os

from openai import AsyncOpenAI


@rx.event
async def answer(self):
    # Our chatbot has some brains now!
    client = AsyncOpenAI(
        api_key=os.environ["OPENAI_API_KEY"]
    )

    session = await client.chat.completions.create(
        model="gpt-4o-mini",
        messages=[
            {"role": "user", "content": self.question}
        ],
        stop=None,
        temperature=0.7,
        stream=True,
    )

    # Add to the answer as the chatbot responds.
    answer = ""
    self.chat_history.append((self.question, answer))

    # Clear the question input.
    self.question = ""
    
    # Yield here to clear the frontend input before continuing.
    yield

    async for item in session:
        if hasattr(item.choices[0].delta, "content"):
            if item.choices[0].delta.content is None:
                # presence of 'None' indicates the end of the response
                break
            answer += item.choices[0].delta.content
            self.chat_history[-1] = (
                self.chat_history[-1][0],
                answer,
            )
            yield
```

Konačno, imamo naš čatbot!

## Finalni kod

Ova aplikacija je jednostavan, interaktivni čatbot napravljen sa Reflex-om koji koristi OpenAI API za inteligentne odgovore. Čatbot ima čist interfejs sa strimovanim odgovorima za prirodno iskustvo razgovora.

### Ključne karakteristike

- Odgovori u realnom vremenu sa strimovanjem
- Čisti, vizuelno jasni balončići za razgovor za pitanja i odgovore
- Jednostavan interfejs za unos sa poljem za pitanje i dugmetom za slanje

### Struktura projekta

Ispod je kompletan kod čatbota sa komentarima koji odgovaraju imenima datoteka.

```sh
chatapp/
├── chatapp.py    # UI komponente i podešavanje aplikacije
├── state.py      # Upravljanje stanjem i API integracija
└── style.py      # Definicije stilova
```

Fajl chatapp.py:

```py
import reflex as rx
from chatapp import style
from chatapp.state import State


def qa(question: str, answer: str) -> rx.Component:
    return rx.box(
        rx.box(
            rx.text(question, style=style.question_style),
            text_align="right",
        ),
        rx.box(
            rx.text(answer, style=style.answer_style),
            text_align="left",
        ),
        margin_y="1em",
    )


def chat() -> rx.Component:
    return rx.box(
        rx.foreach(
            State.chat_history,
            lambda messages: qa(messages[0], messages[1]),
        )
    )


def action_bar() -> rx.Component:
    return rx.hstack(
        rx.input(
            value=State.question,
            placeholder="Ask a question",
            on_change=State.set_question,
            style=style.input_style,
        ),
        rx.button(
            "Ask",
            on_click=State.answer,
            style=style.button_style,
        ),
    )


def index() -> rx.Component:
    return rx.center(
        rx.vstack(
            chat(),
            action_bar(),
            align="center",
        )
    )


app = rx.App()
app.add_page(index)
```

Fajl state.py:

```py
import os
from openai import AsyncOpenAI
import reflex as rx


class State(rx.State):
    question: str
    chat_history: list[tuple[str, str]] = []

    async def answer(self):
        client = AsyncOpenAI(
            api_key=os.environ["OPENAI_API_KEY"]
        )

        # Start streaming completion from OpenAI
        session = await client.chat.completions.create(
            model="gpt-4o-mini",
            messages=[
                {"role": "user", "content": self.question}
            ],
            temperature=0.7,
            stream=True,
        )

        # Initialize response and update UI
        answer = ""
        self.chat_history.append((self.question, answer))
        self.question = ""
        yield

        # Process streaming response
        async for item in session:
            if hasattr(item.choices[0].delta, "content"):
                if item.choices[0].delta.content is None:
                    break
                answer += item.choices[0].delta.content
                self.chat_history[-1] = (
                    self.chat_history[-1][0],
                    answer,
                )
                yield
```

Fajl style.py:

```py
import reflex as rx

# Common style base
shadow = "rgba(0, 0, 0, 0.15) 0px 2px 8px"
chat_margin = "20%"
message_style = dict(
    padding="1em",
    border_radius="5px",
    margin_y="0.5em",
    box_shadow=shadow,
    max_width="30em",
    display="inline-block",
)

# Styles for questions and answers
question_style = message_style | dict(
    margin_left=chat_margin,
    background_color=rx.color("gray", 4),
)

answer_style = message_style | dict(
    margin_right=chat_margin,
    background_color=rx.color("accent", 8),
)

# Styles for input elements
input_style = dict(
    border_width="1px",
    padding="0.5em",
    box_shadow=shadow,
    width="350px",
)

button_style = dict(
    background_color=rx.color("accent", 10),
    box_shadow=shadow,
)
```

### Sledeći koraci

Čestitamo! Napravili ste svoj prvi čatbot. Odavde možete pročitati ostatak dokumentacije da biste naučili više o Reflex-u detaljnije. Najbolji način za učenje je da napravite nešto, pa pokušajte da napravite svoju aplikaciju koristeći ovo kao polaznu tačku!

### Još jedna stvar

Sa našim hosting servisom, možete implementirati ovu aplikaciju jednom komandom za par minuta. Pogledajte naš Hosting Quick Start.

[Sadržaj](00_sadrzaj.md)
