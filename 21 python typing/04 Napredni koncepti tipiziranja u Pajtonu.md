
# Neki napredni koncepti tipiziranja u Pajtonu

Za početak, dozvolite mi da priznam da Python jeste dinamički ( postepeno? ) tipiziran jezik. Međutim, sa modernim Pajtonom (3.6+, a zapravo i sa 3.10+) i statičkim alatima (npr. mypy , pyright/pylance ), to je prilično robustan statički tipiziran jezik. Većina biblioteka trećih strana je ili tipizirana ili imaju odvojeno distribuirane tipove. Iz perspektive programera, ova verzija jezika koja koristi potpuno strogo statičko tipiziranje je u suštini drugačiji jezik od Pajtona iz davnina.

Neću ovde u potpunosti iznositi argument, ali tretiranje Pajtona kao statički tipiziranog jezika značajno povećava čitljivost i izražajnost koda, značajno smanjuje greške, pa čak i količinu testova koje treba da napišete, i olakšava prelazak na "pravi" statički tipizirani jezik kao što je Rust (uradite to, odličan je!) ili Go (uradite to, prilično je dobar!) ili čak stare jezike poput C/C++/Java.

Iako postoji dobar materijal o naprednijim konceptima tipiziranja u Pajtonu ( mypy dokumentacija je zapravo prilično dobra), većina tutorijala koje pronađete na mreži su primeri nivoa "Zdravo svete", koji samo uvode tipove ljudima koji nikada nisu koristili Pajton ili su iz mračnih dana Pajtona pre verzije 3.6. Ovaj post pretpostavlja da dobro razumete Pajton i već znate osnove anotacija tipova, ili da dolazite sa drugog statički tipiziranog jezika i samo želite da vidite da li je Pajton dorastao zadatku. U redu, kraj tirada, hajde da se pozabavimo nekim polunaprednim stvarima u tipiziranju.

Svi primeri u ovom postu su dostupni ovde ako želite da se poigrate sa njima. Predložio bih korišćenje vscode-a i "striktnog" Pylance režima provere tipova.

## Anotacije tipa i princip segregacije interfejsa

Setite se principa segregacije interfejsa iz SOLID principa ujaka Boba. Pa, ako ga ne se setite, to u osnovi znači da korisnici vašeg koda ne bi trebalo da budu primorani da se oslanjaju na interfejse/tipove/klase koji im nisu potrebni. Videćemo kako anotacije tipova u Pajtonu mogu pomoći u tome.

### Neke opšte dobre prakse

Prvo, pogledajmo kako možemo učiniti naš kod robusnijim definisanjem labavih tipova za ulazne parametre i strogih tipova za izlaze. Recimo da pišemo funkciju za filtriranje određene reči ili fraze iz nekih stringova. Mogli biste napisati svoju funkciju ovako.

```py
from typing import List

def filter_things(things: List[str], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```

Ovo je u redu i funkcionisalo bi ovako

```py
things = ["badger", "snake", "mushroom"]
filtered_things = filter_things(things, "mushroom")
# returns ["badger", "snake"]
```

Ali da li nam zaista trebaju ulazi things u obliku liste, sve što radimo jeste da ih prelazimo preko njih. Mnogo Pajton tipova to radi. Mogli bismo da pretpostavimo i uradimo nešto poput

```py
from typing import Any, Dict, List, Set, Tuple

def filter_things(things: List[str] | Tuple[str,...] | Set[str] | Dict[str, Any], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```

ali to bi bilo užasno, a niste samo još više zbunili korisnika. Umesto toga, možemo koristiti tip koji određuje minimalno ograničenje za ono što radimo. Pošto samo iteriramo, možemo da thingsuradimo

```py
from typing import Iterable, List

def filter_things(things: Iterable[str], filter_string: str) -> List[str]:
    return [t for t in things if filter_string not in t]
```

Korišćenje `Iterable` tipa samo navodi da je things kolekcija stringova koja se može iterirati. Sada naša funkcija može da radi sa mnogo različitih tipova ulaza kao što su torke, skupovi, reči, čak i generatori, jer su svi ti tipovi iterabilni. Imajte na umu da ovde vraćamo konkretan tip `List[str]`. Uvek je dobra ideja da tip povratka bude što je moguće uži kako bi ponašanje bilo determinističko bez obzira na to kakvu vrstu ulaza date funkciji.

Da bismo ovo pojasnili, pogledajmo još jedan primer.

```py
from typing import List

def get_fifth_element(things: List[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]
```

Ponovo, ovde nam zapravo ne treba lista, ali ne iteriramo. Umesto toga, proveravamo dužinu i hvatamo vrednost na datom indeksu. Dakle, to znači da moramo biti u mogućnosti da dobijemo veličinu ulazne kolekcije i moramo biti u mogućnosti da dobijemo stavku u kolekciji na datom indeksu. Ako pogledamo dokumentaciju, možemo videti da je ispravan tip koji treba koristiti ovde `Sequence` jer implementira metode `__len__` i `__getitem__` koje nam omogućavaju da lenje pozovemo i indeksiramo je. Dakle, imamo

```py
from typing import Sequence 

def get_fifth_element(things: Sequence[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]

# good with a tuple
elements = ("earth", "wind", "water", "fire", "multipass")
fifth_element = get_fifth_element(elements)

# or a list
elements = ["earth", "wind", "water", "fire", "multipass"]
fifth_element = get_fifth_element(elements)
```

Kao što ćemo videti kasnije, čak je i ovaj tip malo preuzak, ali bez definisanja sopstvenog tipa ovo će obično biti u redu.

### Protokoli

Imao sam prethodni post o protokolima, ali ovde ćemo ih razmotriti u kontekstu principa segregacije interfejsa i dati malo više detalja o tome zašto su korisni.

Pa, počnimo sa starim primerom definisanja Animal

```py
import abc

class Animal(abc.ABC):
    @abc.abstractmethod
    def make_sound(self) -> None:
        pass

    @abc.abstractmethod
    def act(self) -> None:
        pass

    @property
    @abc.abstractmethod
    def num_legs(self) -> int:
        pass
```

Napomena, ovde sam koristio ABC. Mogao sam da koristim protokol za ovo, ali pokušavam da falsifikujem biblioteku treće strane koja možda već ima mnogo različitih implementacija klase koristeći standardnu hijerarhiju klasa. Sada hajde da implementiramo neke životinje.

```py
class Dog(Animal):
    def make_sound(self) -> None:
        print("woof")

    def act(self) -> None:
        print("dog is walking")

    @property
    def num_legs(self) -> int:
        return 4


class Fish(Animal):
    def make_sound(self) -> None:
        print("blub")

    def act(self) -> None:
        print("fish is swimming")

    @property
    def num_legs(self) -> int:
        return 0
```

i napravimo funkciju da bismo mogli da čujemo neke životinjske zvuke

```py
from typing import Iterable

def get_animal_sounds(animals: Iterable[Animal]) -> None:
    for animal in animals:
        animal.make_sound()

animals = [Dog(), Fish()]
get_animal_sounds(animals)
# woof
# blub
```

Dakle, ova funkcija prihvata Iterable(vidite, naučili smo iz prvog dela!) broj Animals. Ako pokrenemo ovu funkciju sa ovim ulazima, dobićemo woofi blub. Super!, sada želimo da znamo šta lisica kaže, pa implementiramo lisicu

```py
class Fox:
    def make_sound(self) -> None:
        print("redacted")
```

Super, hajde da ovo uključimo

```py
animals = [Dog(), Fish(), Fox()]
get_animal_sounds(animals)
```

Uh, dobijamo grešku od našeg proveravača tipa ovako

```sh
Argument of type "list[Dog | Fish | Fox]" cannot be assigned to parameter "animals" of type "Iterable[Animal]" in function "get_animal_sounds"
  "list[Dog | Fish | Fox]" is incompatible with "Iterable[Animal]"
    TypeVar "_T_co@Iterable" is covariant
      Type "Dog | Fish | Fox" cannot be assigned to type "Animal"
        "Fox" is incompatible with "Animal" PylancereportGeneralTypeIssues
```

Ah, da, zato što smo upravo implementirali interfejs make_soundza Foxživotinje, a ne ceo. Pa, to je glupo, samo želim da znam šta lisica kaže, zašto moram da implementiram ceo "interfejs". Protokoli u pomoć

```py
from typing import Iterable, Protocol

class CanMakeSound(Protocol):
    def make_sound(self) -> None:
        ...

def get_animal_sounds(animals: Iterable[CanMakeSound]) -> None:
    for animal in animals:
        animal.make_sound()
```

Sada možemo saznati šta lisica kaže, pošto sve tri životinje sada primenjuju Protokol CanMakeSound.

Dakle, naravno, ovo je glup primer, ali postoji mnogo situacija iz stvarnog života gde naše funkcije, klase itd. treba da znaju samo o nekoliko metoda objekta, a ne moraju da znaju o svima njima. Ako koristimo protokole samo da bismo naveli šta nam je potrebno, to znači da korisnik može da implementira sopstvene objekte samo da bi zadovoljio ovaj protokol.

Ako vam se ovo zaista dopada, evo kako bismo mogli učiniti našu get_fifth_elementfunkciju još fleksibilnijom korišćenjem prilagođenog protokola koji zahteva samo minimum.

```py
from typing import Any, Protocol, TypeVar

T = TypeVar("T", covariant=True)

class MySequence(Protocol[T]):
    def __getitem__(self, __idx: Any) -> T:
        ...

    def __len__(self) -> int:
        ...

def get_fifth_element(things: MySequence[str]) -> str:
    if len(things) < 5:
        raise ValueError(f"Length of input must be >= 5, found len={len(things)}")
    return things[4]
```

## Generici sa ograničenjima protokola

U gornjim primerima koristili smo Protokole da bismo naše funkcije učinili veoma opštim (može se čak reći i generičkim!); međutim, koristili smo ih samo u argumentima, a ne u tipu povratka. Generalno, nema smisla vraćati Protokol. Ponekad je potrebno koristiti generike sa prilagođenim ograničenjima. To možemo ilustrovati implementacijom jednostavne maxfunkcije koja vraća maksimum od dva ulaza. Prvo, hajde da to uradimo za ints

```py
def max(a: int, b: int) -> int:
    if a < b:
        return b
    return a
```

U redu, jednostavno, ali ugrađena max funkcija u Pajtonu radi na mnogim drugim tipovima, kao što su float, str, pa čak i liste i sekvence. Ako želimo generičku max funkciju, pogledajmo gornju funkciju i utvrdimo koji je minimalni zahtev. Ako pogledamo, jedini zahtev je da se ai bmogu uporediti pomoću operatora "manje od". U Pajtonu to znači da oba moraju implementirati __lt__magičnu metodu. Dakle, hajde da napravimo ograničenu generičku funkciju i prepišemo našu funkciju.

```py
from typing import Protocol, TypeVar
from typing_extensions import Self

class HasLessThan(Protocol):
    def __lt__(self, __other: Self) -> bool:
        ...

T = TypeVar("T", bound=HasLessThan)

def max(a: T, b: T) -> T:
    if a < b:
        return b
    return a
```

Sada, ako koristimo ovu funkciju, dobićemo ispravne tipove povratnih vrednosti i to će omogućiti mnogo različitih poređenja.

```py
m = max(3, 4) # m is of type: int
m = max("hello", "world") # m is of type: string
m = max([4, 5, 6], [1, 2]) # m is of type: List[int]
```

Kao i uvek, mogli bismo čak definisati i sopstveni tip. Hajde da definišemo klasu linije gde želimo maxda vratimo duži objekat linije

```py
from dataclasses import dataclass
from typing_extensions import Self

@dataclass
class Line:
    x_min: float
    x_max: float

    def __lt__(self, other: Self) -> bool:
        """Compare based on line length"""
        return (self.x_max - self.x_min) < (other.x_max - other.x_min)

m = max(Line(0, 3), Line(0, 5)) # returns Line(0, 5)
```

Ovo je veoma moćan metod za definisanje generičkih funkcija/klasa koje određuju samo minimalna ograničenja tipa. Ako dolazite sa Rusta, ovo je slično generičkim funkcijama sa ograničenjima osobina.

## Parametar Specifikacija Promenljive

Pošto je Pajton jezik sa postepenim tipiziranjem, on ne može ili nije mogao uvek imati statičko tipiziranje u svim situacijama. Do pojave Parameter Specification Variables ili samo "ParamSpec" nismo mogli dobiti dobro statičko tipiziranje na funkcijama koje su dekorisane. Kada bi funkcija bila dekorisana, gubili bismo informacije o tipu ulaznih i izlaznih parametara te funkcije. Da vidimo kako to sada možemo popraviti saParamSpec

```py
import time
from typing import Callable, ParamSpec, TypeVar

T = TypeVar("T")
P = ParamSpec("P")

def timer(func: Callable[P, T]) -> Callable[P, T]:
    def inner(*args: P.args, **kwargs: P.kwargs) -> T:
        tstart = time.perf_counter()
        out = func(*args, **kwargs)
        print(f"function: {func.__name__} ran in {(time.perf_counter()-tstart)} seconds")
        return out

    return inner

@timer
def add(a: float, b: float) -> float:
    """
    Add two numbers
    """
    return a + b

out = add(3.4, 4.5) 
# function: add ran in 6.902000677655451e-06 seconds
```

Ovde smo kreirali timerdekorator koji jednostavno ispisuje vreme potrebno za poziv dekorisane funkcije. Kada dodamo dekorator jednostavnoj addfunkciji, on će ispisati vreme izvršavanja. Ovo je jednostavan primer, ali hajde da pogledamo šta se dešava. Primetite da spoljašnja funkcija dekoratora prihvata dekorisanu funkciju funci mi koristimo anotaciju tipa Callable[P, T]. Tje samo generička promenljiva koja navodi da funkcija može da vrati bilo koji tip. Promenljiva Pje ParamSpec. Ovo je ono što omogućava da se te informacije o tipu zadrže u dekorisanoj funkciji. U funkciji innerkoristimo *argsi **kwargskao što biste uvek koristili za nepoznate ulaze, ali sada ih možemo anotirati sa P.argsi P.kwargs, respektivno. I to je magija koja poštuje informacije o tipu dekorisane funkcije. Vaš uređivač koda (ja koristim vscode i on je neverovatan za ovo i zapravo sve stvari...) trebalo bi da vam prikaže definiciju funkcije kada pređete mišem preko nje, a provera tipova treba da poštuje tipove ulaza i izlaza. Ovu funkciju sam prilično intenzivno koristio u prethodnom postu o implementaciji potpuno tipiziranog lru-cache-a .

### Preopterećenja

Preopterećenja vam omogućavaju da odredite različite kombinacije ulaza i/ili izlaza za jednu funkciju. Ovo može dovesti do nekih zaista preopterećenih funkcija koje rade previše, ali, kao i većina stvari, sve je umereno.

Generalno govoreći, postoje dva slučaja u kojima biste želeli da razmotrite korišćenje preopterećenja

- Opcioni dekoratori drugog reda (tj. dekoratori koji mogu, ali i ne moraju da prihvataju argumente)
- Funkcije gde postoji nekoliko različitih kombinacija ulaznih parametara koje mogu, ali i ne moraju dovesti do različitih tipova izlaza. Imajte na umu da preopterećenja treba koristiti samo tamo gde se posao ne može obaviti generičkim tipovima.

Prvo, pogledajmo slučaj dekoratora drugog reda modifikujući naš gornji primer. Šta ako želimo mogućnost formatiranja naše poruke koja ispisuje vreme izvršavanja, ili šta ako želimo da koristimo loger umesto samog ispisivanja, ali i dalje želimo da imamo podrazumevano ponašanje bez prosleđivanja bilo kakvih argumenata. Možemo ga modifikovati na sledeći način.

```py
import time
from typing import Callable, Optional, ParamSpec, TypeVar, overload

T = TypeVar("T")
P = ParamSpec("P")

def _default_display_fn(function_name: str, call_time: float) -> None:
    print(f"function: {function_name} ran in {call_time} seconds")

@overload
def timer(__func: Callable[P, T]) -> Callable[P, T]:
    ...

@overload
def timer(*, display_fn: Callable[[str, float], None]) -> Callable[[Callable[P, T]], Callable[P, T]]:
    ...

def timer(
    __func: Optional[Callable[P, T]] = None,
    *,
    display_fn: Optional[Callable[[str, float], None]] = None,
) -> Callable[[Callable[P, T]], Callable[P, T]] | Callable[P, T]:

    display_fn = display_fn or _default_display_fn

    def decorator(func: Callable[P, T]) -> Callable[P, T]:
        def inner(*args: P.args, **kwargs: P.kwargs) -> T:
            tstart = time.perf_counter()
            out = func(*args, **kwargs)
            display_fn(func.__name__, time.perf_counter() - tstart)
            return out

        return inner

    if __func is not None:
        return decorator(__func)

    return decorator
```

Ovde ima dosta detalja, ali glavna poenta su dva dekoratora preopterećenja. Oni određuju dva potpisa dekoratora timer. Prvi je jednostavan dekorator baš kao što smo imali ranije, gde ga možete koristiti bez zagrada. Drugi slučaj je naš dekorator drugog reda koji možemo proslediti u funkciji da bismo prikazali vreme izvršavanja. Mogli bismo ga koristiti na sledeći način.

```py
import logging

def logging_display_fn(function_name: str, call_time: float) -> None:
    logging.info(f"function: {function_name} ran in {call_time} seconds")

@timer(display_fn=logging_display_fn)
def add(a: float, b: float) -> float:
    """
    Add two numbers
    """
    return a + b
```

Sada, umesto korišćenja podrazumevane vrednosti, printkoristićemo loger. U oba slučaja, potpis tipa originalne addfunkcije se čuva. Samo posmatranjem složenosti stvarne timerimplementacije možemo početi da vidimo zašto preopterećenje može biti opasno, jer funkcija može postati veoma komplikovana i možda je bolje podeliti je na odvojene funkcije. Ipak, korišćenje preopterećenja kao načina da se imaju dekoratori i prvog i drugog reda je generalno dobra ideja.

Da bismo demonstrirali drugi slučaj preopterećenja, proširimo funkcionalnost naše maxfunkcije tako da korisnik može da prosledi tipove koji nemaju izvornu podršku za <, ali im dozvoljavamo da proslede ključnu funkciju kako bi odredili kako da uporede vrednosti ai b:

```py
from typing import Callable, Protocol, TypeVar, overload
from typing_extensions import Self

class HasLessThan(Protocol):
    def __lt__(self, __other: Self) -> bool:
        ...

T = TypeVar("T", bound=HasLessThan)
S = TypeVar("S")

def _default_key(__val):
    return __val

@overload
def max(a: T, b: T) -> T:
    ...

@overload
def max(a: S, b: S, *, key: Callable[[S], T]) -> S:
    ...

def max(a, b, *, key=None):
    key_func = key or _default_key
    if key_func(a) < key_func(b):
        return b
    return a
```

U gornjem primeru imamo dva preopterećenja. Prvo preopterećenje je upravo ono što smo imali ranije, gde ulazi moraju da podržavaju operator <. Drugo preopterećenje sada omogućava korisniku da prosledi bilo koji tip, ali je potrebno da prosledi ključnu funkciju koja vraća vrednost koja će se koristiti za poređenje dva ulaza.

Na primer, vratimo se na naš prilagođeni tip linije, ali ovaj put nećemo implementirati metodu __lt__već ćemo koristiti ključnu funkciju umax

```py
from dataclasses import dataclass

@dataclass
class Line:
    x_min: float
    x_max: float

m = max(Line(0, 3), Line(0, 5)) # this will fail the type check
m = max(Line(0, 3), Line(0, 5), key=lambda line: (line.x_max - line.x_min)) # returns Line(0, 5)
```

Dakle, ovde nam naša preopterećenja omogućavaju da budemo fleksibilniji u našim tipovima. Imajte na umu da u ovom slučaju nisam uključio tipove u stvarnu implementaciju. Na vama je da li ćete to uraditi ili ne, jer stvarni potpis implementacije neće biti izložen krajnjem korisniku, već samo preopterećenja.

Funkcija u maxPajtonu zapravo ima još nekoliko preopterećenja ako želite da je proverite.
Završavanje

U ovom postu smo obradili neke naprednije slučajeve upotrebe anotacija tipova. TLDR verzija je

- Koristite opšte tipove unosa navodeći samo najmanji deo onoga što je potrebno
- Koristite stroge konkretne tipove izlaza
- Koristite prilagođene protokole koji definišu minimalni interfejs potreban za funkcije/metode/klase
- Koristite prilagođene protokole kao granice generičkih tipova da biste kod učinili opštijim
- Koristite specifikaciju parametra (npr. ParamSpec) prilikom pravljenja dekoratera
- Koristite preopterećenja štedljivo kada generički tipovi ne mogu da obave posao. Dajte prednost preopterećenjima u odnosu na upotrebu Union tipova, posebno u povratnim tipovima.

Kao što sam pomenuo u uvodu, ništa od ovoga nije potrebno za validan Pajton, ali korišćenje sistema anotacija tipova će učiniti vaš kod čitljivijim i učiniće ga mnogo prijatnijim za rad.
