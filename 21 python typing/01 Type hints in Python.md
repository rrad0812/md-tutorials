
# Nagoveštavanje tipova u Pajtonu

Jedan od moćnih alata koje Pajton pruža za promociju jasnog i pouzdanog koda je koncept "savetovanja tipova".

Možda se pitate: "Pajton je jezik sa dinamičkim tipiziranjem, pa zašto bih se mučio sa tipovima?"

Kao inženjer podataka ili početnik u Pajtonu zainteresovan za najbolje prakse kodiranja, razumevanje i primena tipova saveta u vašem Pajton kodu može biti prava prednost.

U ovom članku ćemo se detaljnije pozabaviti tipovima, njihovim primenama i njihovim prednostima u programiranju u Pajtonu. Pošto je Dagster frejmvork sa anotacijama tipova, takođe ćemo objasniti kako se tipovi mogu koristiti u procesima inženjeringa podataka kako bi se poboljšala njihova čitljivost i smanjila sklonost greškama. To je kao da pružate mapu sebi u budućnosti i drugim programerima koji mogu da interaguju sa vašim kodom - mapu koja detaljno prikazuje tipove podataka koji ulaze i izlaze iz vaših funkcija i klasa.

## Šta je dinamičko tipiziranje

Pajton je dinamički tipiziran jezik. U statički tipiziranim jezicima kao što su Java ili C++, morate deklarisati tip promenljivih pre nego što ih upotrebite. Na primer, potrebno je da navedete da li je promenljiva ceo broj, broj sa pokretnim decimalom, string itd. U Pajtonu možete kodirati bez razmišljanja o tipovima podataka sve do izvršavanja programa – što je jedna od karakteristika koje Pajton čine posebno prilagođenim početnicima.

Na primer, možete deklarisati promenljivu i direktno joj dodeliti vrednost bez navođenja njenog tipa, otuda i termin "dinamički tipiziran". Pajton interpreter implicitno povezuje vrednost i njen tip tokom izvršavanja.

```py
x = 10  # x is an integer
x = "Hello"  # now x is a string
```

U prvom redu x je ceo broj. U drugom redu, isti x postaje string. Pajton besprekorno obrađuje ovu tranziciju zahvaljujući svojoj prirodi dinamičkog tipiziranja.

Međutim, ova dinamična priroda može dovesti i do grešaka koje je teško otkloniti, posebno u velikim bazama koda ili složenim cevovodima za obradu podataka, gde protok podataka možda nije odmah očigledan.

Nagoveštaji tipa, uvedeni u Pajtonu 3.5 kao deo standardne biblioteke preko PEP 484, omogućavaju vam da odredite očekivani tip promenljive, parametra funkcije ili povratne vrednosti.

### Zašto koristiti naznake tipova

Iako dinamičko tipiziranje nudi fleksibilnost, ono takođe stvara prostor za potencijalne greške. Tu dolaze do izražaja saveti za tipove. Oni mogu značajno poboljšati čitljivost koda i sprečiti greške povezane sa tipovima.

- **Poboljšana čitljivost koda**: Nagoveštaji tipova deluju kao oblik dokumentacije koji pomaže programerima da razumeju tipove argumenata koje funkcija očekuje i šta vraća. Ova poboljšana jasnoća čini kod čitljivijim i lakšim za razumevanje.

- **Detekcija grešaka**: Alati poput **pyright** i **mypy** mogu se koristiti za statičku analizu vašeg Pajton koda. Proverava konzistentnost tipova u vašem kodu na osnovu naznaka tipova i upozorava vas na greške vezane za tip pre izvršavanja. Saznajte zašto Dagster tim preporučuje mypy potpuno preskakanje i korišćenje samo pyright.

- **Bolja podrška za IDE**: Mnoga integrisana razvojna okruženja (IDE) i linteri mogu da koriste naznake tipova kako bi obezbedili bolje dovršavanje koda, proveru grešaka i refaktorisanje.

- **Olakšava velike projekte**: Za veće projekte sa više programera, naznake tipova mogu biti veoma korisne za razumevanje strukture i toka podataka kroz bazu koda. Objavili smo vodič o tome kako uključiti i održavati anotacije tipova za javne Pajton projekte.

### Ograničenja

- **Ne primenjuje se tokom izvršavanja**: Pajtonovi nagoveštaji tipova se ne primenjuju već su samo nagoveštaji, a Pajton interpreter neće izbaciti greške ako se navedeni tipovi ne podudaraju sa stvarnim vrednostima. Ovo može dovesti do pogrešnog shvatanja da nagoveštaji tipova mogu da primene na bezbednost tipova, što ne mogu.

- **Prekomplikovano**: Za male ili jednostavne skripte, saveti za tipove mogu delovati preterano i potencijalno bi mogli da iskomplikuju kod koji bi trebalo da bude jasan i jednostavan.

- **Nije fleksibilan**: Jedan od razloga za popularnost Pajtona je njegova dinamička priroda, a naznake tipova mogu to ograničiti.

## Osnovni saveti za tipove

Pajtonov `typing` modul sadrži nekoliko funkcija i klasa koje se koriste za pružanje naznaka tipova za vaš Pajton kod. Evo kako možete primeniti naznake tipova u različitim scenarijima.

### Deklarisati tipove za promenljive

Da biste dali savete o tipu promenljivih, možete koristiti simbol dvotačke `:` praćen tipom. Evo primera:

```py
age: int = 20
name: str = "Alice"
is_active: bool = True
```

Ovde "age" je nagovešteno kao `int`, "name" kao `string` i "is_active" kao `bool` vrednost.

### Anotacije funkcija

Možete dati savete o tipovima za parametre funkcije i povratne vrednosti. Ovo pomaže drugim programerima da razumeju koje tipove argumenata funkcija očekuje i koji tip funkcija vraća.

```py
def greet(name: str) -> str: 
    return f"Hello, {name}"
```

U ovom primeru, funkcija greetočekuje nameda bude string i vratiće string.

## Ugrađeni tipovi u Pajtonu

Pajton ima nekoliko ugrađenih tipova. Najčešće korišćeni su:

- **int**: Predstavlja ceo broj
- **float**: Predstavlja broj sa pokretnim zarezom
- **bool**: Predstavlja bulovsku vrednost (`True` ili `False`)
- **str**: Predstavlja string

Postoje i složeni tipovi kao što su `list`, `tuple` i `dict` koji se mogu koristiti za pružanje detaljnijih naznaka tipova koje ćemo kasnije pogledati.

Takođe ćete pronaći listu glavnih tipova Pajtona u dodatku.

### Atomski naspram kompozitnih tipova

U Pajtonu postoji razlika između atomskih i kompozitnih tipova kada je u pitanju nagoveštavanje tipova. Atomski tipovi, kao što su `int`, `float` i `str`, su jednostavni i nedeljivi, a njihove anotacije tipa mogu se dati direktno korišćenjem samog tipa, kao što je `str`.

```py
def my_function(my_string: str) -> int:
    return len(my_string)
```

S druge strane, kompozitni tipovi poput `List` i `Dict` su sastavljeni od drugih tipova, i pre Pajtona 3.9, često su zahtevali uvoz specifičnih definicija iz modula `typing`, kao što je `typing.List[int]` za listu celih brojeva.

```py
from typing import List

def my_function(numbers: List[int]) -> int:
    return sum(numbers)
```

U novijim verzijama Pajtona, možete pisati `list[int]` umesto `typing.List[int]`.

## Annotiranje funkcija u detaljima

Nagoveštaji tipova mogu biti posebno korisni kada se ugrade u potpise funkcija. Ovo ne samo da omogućava programerima da razumeju koje tipove argumenata funkcija očekuje, već im daje i predstavu o tome šta će funkcija vratiti.

### Kako odrediti tipove argumenata i tip povratka funkcije

Možete odrediti tipove argumenata i tip povratka funkcije koristeći simbol `:` za argumente i simbol `->` za tip povratka. Evo opšte sintakse:

```py
def function_name(arg1: type1, arg2: type2, ...) -> return_type:
    # function body
```

U ovoj sintaksi, "arg1", "arg2", itd. su argumenti funkcije, a "type1", "type2", itd. su tipovi tih argumenata. "return_type" je tip vrednosti koju funkcija vraća.

### Primeri korišćenja naznaka tipova u potpisima funkcija

Razmotrimo funkciju koja izračunava površinu pravougaonika:

```py
def area_rectangle(length: float, breadth: float) -> float:
    return length * breadth
```

U ovoj funkciji, očekuje se da su "length" i "breadth" pokretni brojevi, a funkcija takođe vraća pokretni broj. Funkcija će i dalje raditi ako prosledite cele brojeve ili čak stringove koji se mogu konvertovati u pokretni broj, ali naznaka tipa jasno stavlja do znanja da je dizajnirana za rukovanje brojevima sa pokretnim zarezom.

Drugi primer može biti funkcija koja prihvata listu celih brojeva i vraća njihov zbir kao ceo broj:

```py
def sum_elements(numbers: list[int]) -> int:
    return sum(numbers)
```

U‍ ovom primeru, "numbers" parametar je naznačen kao lista celih brojeva, a tip povratka je ceo broj.

Imajte na umu da ovi saveti za tipove ne primenjuju proveru tipova tokom izvršavanja programa. To su saveti za programere i Pajton neće podići `TypeError` ako se stvarni tipovi ne podudaraju sa navedenim tipovima.

## Složeni tipovi

Modul `typing` u Pajtonu pruža nekoliko klasa koje se mogu koristiti za pružanje složenijih saveta tipova. U nastavku su navedene neke od najčešće korišćenih klasa:

### Lista, rečnik, torka, skup

Klase `list`, `dict`, `tuple` i `set` mogu se koristiti za pružanje naznaka tipova za liste, rečnike, torke i skupove, respektivno. Mogu se parametrizovati da bi se pružile još detaljnije naznake tipova.

- **Lista celih brojeva**:

  ```py
  numbers: list[int] = [1, 2, 3]
  ```

- **Rečnik sa string ključevima i float vrednostima**:

  ```py
  weights: dict[str, float] = {"apple": 0.182, "banana": 0.120}
  ```

- **Torka sa intom i stringom**:

  ```py
  student: tuple[int, str] = (1, "John")
  ```

- **Skup stringova**:

  ```py
  flags: set[str] = {"apple", "banana", "cherry"}
  ```

U ovim primerima,

- "numbers" je označeno kao lista celih brojeva,
- "weights" je rečnik sa string ključevima i float vrednostima,
- "student" je torka sa celim brojevima i stringom, i
- "flags" je skup stringova.

### Optional

Nagoveštaj tipa `Optional` može se koristiti da naznači da promenljiva može biti ili određenog tipa ili `None`.

```py
from typing import Optional

def find_student(student_id: int) -> Optional[dict[str, str]]:
    # If the student is found, return a dictionary containing their data
    # If the student is not found, return None
```

### Union

Nagoveštaj tipa `Union` se koristi da naznači da promenljiva može biti jednog od nekoliko tipova. Na primer, ako promenljiva može biti ili `str` ili `int`, možete dati nagoveštaj tipa ovako:

```py
from typing import Union

def process(data: Union[str, int]) -> None:
    # This function can handle either a string or an integer
```

U novijim verzijama Pajtona, možete koristiti operator vertikalne crte (`|`) da biste označili tip koji može biti jedna od nekoliko opcija, zamenjujući potrebu za `Union`:

```py
def process(data: str | int) -> None:
    # This function can handle either a string or an integer
```

### Any

Klasa `Any` se koristi da naznači da promenljiva može biti bilo kog tipa. Ovo je ekvivalentno kao da uopšte ne navedete naznaku tipa.

```py
from typing import Any

def process(data: Any) -> None:
    # This function can handle data of any type
```

Ovi alati iz `typing` modula mogu vam pomoći da pružite detaljne savete o tipovima koji olakšavaju razumevanje i debagovanje vašeg koda.

Međutim, imajte na umu da su Pajtonovi saveti za tipove opcioni i da se ne primenjuju tokom izvršavanja. Oni su namenjeni kao alat za programere, a ne kao način za sprovođenje bezbednosti tipova.

## Korisnički definisani tipovi

U Pajtonu možete definisati sopstvene tipove koristeći klase, što je osnovni mehanizam za kreiranje prilagođenih tipova. Možete koristiti ove klase u naznakama tipova baš kao što biste koristili ugrađene tipove. Modul `typing`takođe pruža dodatne alate za kreiranje specifičnijih tipova, uključujući `Type` i `NewType`.

### Definisanje sopstvenih tipova pomoću klasa

Možete kreirati klasu i koristiti je kao naznaku tipa. Evo primera:

```py
class Student:
    def __init__(self, name: str, age: int):
        self.name = name
        self.age = age

def print_student_details(student: Student) -> None:
    print(student.name, student.age)
```

Studentje korisnički definisan tip i koristi se kao naznaka tipa u print_student_detailsfunkciji.

### Korišćenje Type za nagoveštavanje tipa

Klasa Typeiz typingmodula može se koristiti da naznači da će promenljiva biti klasa, a ne instanca klase. Ovo se obično koristi kada se očekuje da argument funkcije bude klasa, na primer u fabričkim funkcijama.

```py
from typing import Type

def create_student(cls: Type[Student], name: str, age: int) -> Student:
    return cls(name, age)
```

U ovom primeru, create_studentočekuje Studentklasu (ili podklasu od Student) kao svoj prvi argument.

### Korišćenje NewType za kreiranje različitih tipova

NewTypese koristi za kreiranje različitih tipova. Korisno je kada želite da razlikujete dva tipa koja bi inače bila ista.

Na primer, recimo da u svom programu obrađujete studentske identifikacione brojeve i identifikacione brojeve kursa i želite da budete sigurni da ih ne mešate. Oba su predstavljena kao celi brojevi, tako da možete koristiti NewTypeda biste kreirali dva različita tipa:

```py
from typing import NewType

StudentID = NewType('StudentID', int)
CourseID = NewType('CourseID', int)

def get_student(student_id: StudentID) -> None:
    # Fetch student data...

def enroll_in_course(student_id: StudentID, course_id: CourseID) -> None:
    # Enroll the student in the course...
```

Iako su StudentIDi CourseIDceli brojevi, smatraju se različitim tipovima i ne mogu se koristiti naizmenično. Međutim, imajte na umu da se ova provera ne sprovodi u vreme izvršavanja, već tokom statičke provere tipa pomoću alata kao što je mypy.

## Generički tipovi

Generici vam omogućavaju da definišete funkciju, klasu ili strukturu podataka koja radi sa različitim tipovima. Klasa Generici TypeVarfunkcija iz typingmodula se koriste za definisanje generičkih tipova. Na primer, lista je generička struktura podataka jer može da sadrži elemente bilo kog tipa.

### TypeVar

TypeVar se koristi za definisanje promenljive tipa, koja može biti bilo kog tipa, a određeni tip određuje klijentski kod. Evo primera:

```py
from typing import TypeVar

T = TypeVar('T')

def first_element(lst: List[T]) -> T:
    return lst[0]
```

Ovde je T promenljiva tipa koja može biti bilo kog tipa. Funkcija first_element radi sa listom bilo kog tipa i vraća element tog tipa. Konkretan tip Tbi bio određen listom koju prosleđujete funkciji.

### Generic

Genericse koristi za definisanje generičkih klasa. Generička klasa može biti inicijalizovana različitim tipovima, a ti tipovi se koriste u naznakama tipova unutar klase.

```py
from typing import Generic, TypeVar

T = TypeVar('T')

class Box(Generic[T]):
    def __init__(self, value: T):
        self.value = value

    def get(self) -> T:
        return self.value
```

Ovde Boxje generička klasa koja radi sa bilo kojim tipom T. Kada kreirate instancu Box, možete navesti tip T, a taj tip se koristi u valueatributu i get​​metodi.

```py
box1 = Box[int](10)
box2 = Box[str]("Hello")
```

box1je Boxkoji sadrži ceo broj, i box2je Boxkoji sadrži string.

## Provera tipa sa `pyright`

Provera tipova poput . pyrightje alat koji se koristi za sprovođenje nagoveštavanja tipova u Pajtonu. U Dagsteru nam se zaista sviđa **pyright** jer je brži od drugih alternativa kao što je **mypy**.

Sam Pajton je dinamički tipiziran jezik, što znači da se provere tipova dešavaju tokom izvršavanja programa i da ne primenjuje pravila nagoveštavanja tipova. Ako pokušate da izvršite operaciju koja nije podržana za dati tip podataka, Pajton će izazvati grešku tokom izvršavanja programa. Na primer, pozivanje nedefinisane metode na objektu će pokrenuti grešku samo tokom izvršavanja programa.

Međutim, prilikom razvoja velikih ili složenih sistema, sprovođenje konzistentnosti tipova može pomoći u ranom otkrivanju potencijalnih grešaka. pyrightvrši statičku proveru tipova, što znači da proverava tipove vaših promenljivih, argumenata funkcija i povratnih vrednosti pre nego što se kod zapravo pokrene. Za ovo koristi naznake tipova koje ste naveli u svom kodu. Važno je razumeti da pyrightne izvršava niti pokreće vaš kod; on ga jednostavno čita i analizira.

### Kako koristiti proveru tipova da biste proverili svoje tipove

Da biste ga koristili **pyright**, prvo ga morate instalirati:

```sh
pip install pyright
```

Zatim, da biste proverili Pajton datoteku, pokrećete **pyright** sa datotekom kao argumentom:

```sh
pyright my_file.py
```

Pyright će zatim analizirati datoteku i prijaviti sve pronađene greške u tipu.

Na primer, ako imate funkciju koja je anotirana da primi strkao argument i prosledite int, pyrightfunkcija će ovo prepoznati.

## Statička naspram dinamičke provere tipa

Statička provera tipova je proces provere bezbednosti tipova programa na osnovu analize teksta programa (izvornog koda). Statička provera tipova se vrši u vreme kompajliranja (pre nego što se program pokrene). Jezici koji sprovode statičku proveru tipova uključuju C++, Java i Rust.

S druge strane, dinamička provera tipova je proces provere bezbednosti tipova programa tokom izvršavanja. Dinamička provera tipova se dešava dok se program pokreće. Jezici koji koriste dinamičku proveru tipova uključuju Pajton, Rubi i Javaskript.

Kod statičke provere tipova, tipovi se proveravaju pre nego što se program pokrene, što olakšava otkrivanje i sprečavanje grešaka u tipu. Ovo čini program bezbednijim za pokretanje, jer je većina grešaka povezanih sa tipovima otkrivena tokom kompajliranja. Međutim, to takođe zahteva od programera da eksplicitno deklariše tipove svih promenljivih i povratnih vrednosti funkcija, što se može smatrati smanjenjem fleksibilnosti.

Dinamička provera tipa pruža veću fleksibilnost, jer ne morate eksplicitno deklarisati tip svake promenljive. Međutim, to takođe znači da se greške u tipu mogu pojaviti tokom izvršavanja, što bi potencijalno moglo dovesti do pada programa.

Pajton je jezik sa dinamičkim tipiziranjem, ali takođe podržava opcionu statičku proveru tipova putem alata kao što su pyrighti type hints. Ovo pruža Pajton programerima jedinstvenu fleksibilnost, omogućavajući im da biraju kada žele bezbednost statičke provere tipova, a kada preferiraju fleksibilnost dinamičkog tipiziranja.

## Saveti za tipove i dokumentacione strngove

Nagoveštaji tipova, kao što smo već pomenuli, ukazuju na tipove promenljivih, parametara funkcija i povratnih vrednosti. Oni mogu pomoći drugim programerima da razumeju koje tipove podataka vaša funkcija očekuje i šta će vratiti.

S druge strane, dokumentacioni stringovi se koriste za pružanje opisa onoga što vaša funkcija, klasa ili modul radi. Dokumentacioni string može da sadrži opis svrhe funkcije, njene argumente, njenu povratnu vrednost i sve izuzetke koje može da izazove.

Evo primera kako možete zajedno koristiti naznake tipova i dokumentacione stringove:

```py
def filter_and_sort_products(products: list[dict[str, int]], attribute: str, min_value: int) -> list[dict[str, int]]:
    """
    Filters a list of products by a given attribute and minimum value, and then sorts the filtered products by the attribute.

    Args:
        products (list[dict[str, int]]): A list of products represented as dictionaries.
        attribute (str): The attribute to filter and sort by.
        min_value (int): The minimum acceptable value of the specified attribute.

    Returns:
        list[dict[str, int]]: A list of filtered and sorted products.

    Raises:
        KeyError: If the specified attribute is not found in any product.

    Examples:
        >>> products = [{"name": "Apple", "price": 10}, {"name": "Banana", "price": 5}]
        >>> filter_and_sort_products(products, "price", 6)
        [{"name": "Apple", "price": 10}]
    """
    filtered_products = [product for product in products if product[attribute] >= min_value]
    return sorted(filtered_products, key=lambda x: x[attribute])
```

Ovde, potpis funkcije pokazuje da funkcija uzima listu rečnika koji predstavljaju proizvode, string koji predstavlja atribut i ceo broj koji predstavlja minimalnu vrednost. Vraća listu filtriranih i sortiranih rečnika.

Dokumentacija objašnjava svrhu funkcije, njene parametre, povratnu vrednost, moguće izuzetke (kao što je slučaj KeyErrorako dati atribut nije prisutan) i uključuje primer kako pozvati funkciju.

Ova kombinacija tipova saveta i dokumentacionih nizova može značajno poboljšati čitljivost i održavanje vašeg koda.

## Dodatak: Tipovi Pajtona

Ukratko, evo najčešćih ugrađenih tipova podataka u Pajtonu. Takođe možete naići na ili koristiti prilagođene tipove podataka iz eksternih biblioteka ili one koje su definisali drugi programeri.

- **Numerički tipovi**

  | Vrsta | Naziv | Opis | Primeri |
  | :---: | ----- | ---- | ------- |
  | Numerički | **int** | Celi brojevi | 5 , -3, 42 |
  | Numerički | **float** | Brojevi sa pokretnim zarezom | 3.14 , -0.001, 2.71 |
  | Numerički | **complex** | Kompleksni brojevi | 3+4j,2-5j |
  | Tekst | **str** | String | "Hello, World!",'Python' |
  | Sequence | **list** | Lista | [1, 2, 3], ["a", "b", "c"] |
  | Sequence | **tuple** | Torka | (1, 2, 3), ("a", "b", "c") |
  | Sequence | **range** | Opseg | range(5), range(0, 5, 2) |
  | Mapa | **dict** | Rečnik | {"name": "John", "age": 30} ,{1: "one", 2: "two"} |
  | Skup | **set** | Skup | {1, 2, 3},{"apple", "banana", "cherry"} |
  | Skup | **frozenset** | Nepromenljivi skup | frozenset(["a", "b"]) |
  | Bool | **bool** | Bool | True, False |
  | Binarni | **bytes** | Nepromenljivi niz bajtova | b'hello', bytes([65, 66, 67]) |
  | Binarni | **bytearray** | Promenljivi niza bajtova | bytearray([65, 66, 67]) |
  | Binarni | **memoryview** | Objekat prikaza memorije | memoryview(b'abc') |
  | Nijedan | **NoneType** | Predstavlja odsustvo vrednosti | None |

## Ostali korisni moduli i tipovi

1. **datetime** Modul

    datetime.date       – Predstavlja datum  
    datetime.datetime   – Predstavlja datum i vreme  
    datetime.time       – Predstavlja doba dana  
    datetime.timedelta  – Trajanje ili razlika između dva datuma/vremena  
    datetime.tzinfo     – Osnovna klasa za informacije o vremenskoj zoni

2. **collections** Modul

    namedtuple  – Fabrička funkcija za kreiranje podklasa korke sa imenovanim poljima  
    deque       – Red sa dva kraja  
    Counter     – Podklasa dict za brojanje hešovanih objekata  
    OrderedDict – Rcent koji pamti redosled umetanja  
    defaultdict – Rcent koji pruža podrazumevane vrednosti za nedostajuće ključeve  

3. **array** Modul

    array.array – Prostorno efikasan niz sa specifikacijom tipa  

4. **struct** Modul

    Koristi se za pakovanje i raspakivanje binarnih podataka

5. **json** Modul

    Alati za kodiranje i dekodiranje JSON podataka

6. **enum** Modul

    Enum    – Osnovna klasa za kreiranje nabrojanih konstanti
    IntEnum – Nabrajanja koja su takođe podklase odint
