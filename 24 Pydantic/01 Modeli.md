
# Modeli

Jedan od glavnih načina definisanja šeme u Pydantic-u je putem modela. Modeli su jednostavno klase koje nasleđuju `BaseModel` polja i definišu ih kao anotirane atribute.

Možete zamisliti modele kao slične strukturama u jezicima kao što je C, ili kao zahteve jedne krajnje tačke u API-ju.

Modeli dele mnoge sličnosti sa Pajtonovim `dataclasses`, ali su dizajnirani sa nekim suptilnim, ali važnim razlikama koje pojednostavljuju određene tokove rada vezane za:

- **validaciju**, 
- **serijalizaciju** i 
- **generisanje JSON šeme**.
  
Više o tome možete pronaći u odeljku "Klase podataka" u dokumentaciji.

Nepouzdani podaci mogu se proslediti modelu i, nakon parsiranja i validacije, Pydantic garantuje da će polja rezultujuće instance modela biti u skladu sa tipovima polja definisanim na modelu.

> [!Note]Napomena
>
> Validacija — namerno pogrešan naziv  
>
> Termin "validacija" koristimo da označimo proces instanciranja modela (ili drugog tipa) koji se pridržava određenih tipova i ograničenja. Ovaj zadatak, po kome je Pydantic dobro poznat, najčešće se naziva "validacija" u kolokvijalnom smislu, iako u drugim kontekstima termin "validacija" može biti restriktivniji.
>
> Potencijalna zabuna oko termina "validacija" proizilazi iz činjenice da se, strogo govoreći, primarni fokus Pidantika ne poklapa precizno sa rečničkom definicijom "validacije":
>
> Validacija - **imenica, radnja provere ili dokazivanja validnosti ili tačnosti nečega.
>
> U Pydantic-u, termin "validacija" se odnosi na proces stvaranja instanci modela (ili drugog tipa) koji se pridržava određenih tipova i ograničenja. Pydantic garantuje tipove i ograničenja izlaza, a ne ulaznih podataka. Ova razlika postaje očigledna kada se uzme u obzir da se Pydantic-ova `ValidationError` greška pokreće kada se podaci ne mogu uspešno raščlaniti u instancu modela.
>
> Iako ova razlika u početku može delovati suptilno, ona ima praktičan značaj. U nekim slučajevima, "validacija" ide dalje od samog kreiranja modela i može uključivati kopiranje i pretvaranje podataka u druge. Ovo može uključivati kopiranje argumenata prosleđenih konstruktoru kako bi se izvršilo pretvaranje u novi tip bez mutiranja originalnih ulaznih podataka. Za detaljnije razumevanje implikacija za vašu upotrebu, pogledajte odeljke "Konverzija podataka" i "Kopiranje atributa" u nastavku.
>
> U suštini, primarni cilj Pydantic-a je da osigura da rezultujuća struktura, nakon naknadne obrade (nazvana "validacija"), precizno odgovara primenjenim tipovima. S obzirom na široko rasprostranjeno usvajanje "validacije" kao kolokvijalnog termina za ovaj proces, mi ćemo ga dosledno koristiti u našoj dokumentaciji.
>
> Iako su se termini "parsiranje" i "validacija" ranije koristili naizmenično, ubuduće cilj nam je da isključivo koristimo "validiranje", pri čemu je "parsiranje" rezervisano posebno za diskusije vezane za parsiranje JSON-a.

## Osnovna upotreba modela

> [!Note]Napomena
>
> Pydantic se u velikoj meri oslanja na postojeće konstrukcije za kucanje u Pajtonu za definisanje modela. Ako niste upoznati sa njima, sledeći resursi mogu biti korisni:
>
>    [Vodiči za sistem tipova](https://typing.readthedocs.io/en/latest/guides/index.html)  
>    [mypy dokumentacija](https://mypy.readthedocs.io/en/latest/)

```py
from pydantic import BaseModel, ConfigDict

class User(BaseModel):
    id: int
    name: str = 'Jane Doe'
    model_config = ConfigDict(str_max_length=10)
```

U ovom primeru, User je model sa dva polja:

- **id**, što je ceo broj (definisan pomoću int tipa) i obavezan je
- **name**, što je string (definisan pomoću str tipa) i nije obavezan (ima podrazumevanu vrednost).

Dokumentacija o [tipovima](https://pydantic.dev/docs/validation/latest/concepts/types/) proširuje podržane tipove.

Polja se mogu prilagoditi na više načina pomoću `Field()` funkcije. Više informacija potražite u dokumentaciji o poljima.

Model se zatim može instancirati:

```py
user = User(id='123')
```

user je instanca od User. Inicijalizacija objekta će izvršiti svu analizu i validaciju. Ako se ne izazove `ValidationError` izuzetak, znate da je rezultujuća instanca modela validna.

Poljima modela se može pristupiti kao normalnim atributima objekta user:

```py
assert user.name == 'Jane Doe'  
assert user.id == 123  
assert isinstance(user.id, int)
```

Instanca modela može biti serijalizovana korišćenjem `model_dump()` metode:

```py
assert user.model_dump() == {'id': 123, 'name': 'Jane Doe'}
```

Pozivanje funkcije `dict`  na instanci će takođe obezbediti rečnik, ali ugnežđena polja neće biti rekurzivno konvertovana u rečnike. `model_dump()` takođe pruža brojne argumente za prilagođavanje rezultata serijalizacije.

Podrazumevano, modeli su promenljivi i vrednosti polja se mogu menjati dodeljivanjem atributa:

```py
user.id = 321
assert user.id == 321
```

> [!Warning]Oprez
>
> Prilikom definisanja modela, pazite na kolizije imenovanja između imena polja i njegove anotacije tipa.
>
> Na primer, sledeće se neće ponašati kako se očekuje i dovešće do greške validacije:
>
> ```py
> from typing import Optional
> from pydantic import BaseModel
>
> class Boo(BaseModel):
>     int: Optional[int] = None
>
> m = Boo(int=123)  # Will fail to validate.
> ```
> 
> Zbog načina na koji Pajton procenjuje anotirane naredbe dodele, naredba je ekvivalentna sa 
>
> ```py
> int: None = None
> ```
>
> što dovodi do greške u validaciji.

## Metode i svojstva modela

Gore navedeni primer pokazuje samo vrh ledenog brega onoga što modeli mogu da urade. Klase modela poseduju sledeće metode i atribute:

- **model_validate()**: Validira dati objekat u odnosu na Pydantic model. Vidite Validacija podataka.
- **model_validate_json()**: Validira date JSON podatke u odnosu na Pydantic model. Vidite Validacija podataka.
- **model_construct()**: Kreira modele bez pokretanja validacije. Pogledajte Kreiranje modela bez validacije.
- **model_dump()**: Vraća rečnik polja i vrednosti modela. Videti Serijalizacija.
- **model_dump_json()**: Vraća JSON string reprezentaciju model_dump(). Pogledajte Serijalizaciju.
- **model_copy()**: Vraća kopiju (podrazumevano, plitku kopiju) modela. Pogledajte Kopija modela.
- **model_json_schema()**: Vraća jsonable rečnik koji predstavlja JSON šemu modela. Vidite JSON šemu.
- **model_fields**: Mapiranje između imena polja i njihovih definicija ( FieldInfoinstanci).
- **model_computed_fields**: Mapiranje između izračunatih imena polja i njihovih definicija ( ComputedFieldInfoinstanci).
- **model_parametrized_name()**: Izračunava ime klase za parametrizacije generičkih klasa.
- **model_post_init()**: Izvršava dodatne radnje nakon što je model instanciran i svi validatori polja su primenjeni.
- **model_rebuild()**: Rekonstrukcija šeme modela, koja takođe podržava izgradnju rekurzivnih generičkih modela. Vidite Rekonstrukcija šeme modela.

Instance modela poseduju sledeće atribute:

- **model_extra**: Dodatna polja podešena tokom validacije.
- **model_fields_set**: Skup polja koja su eksplicitno navedena prilikom inicijalizacije modela.

> [!Note]Napomena
>
> Pogledajte API dokumentaciju `BaseModel` za definiciju klase, uključujući kompletnu listu metoda i atributa.  

> [!Note] Info  
>   Pogledajte "Izmene pydantic.BaseModel" u "Vodiču za migraciju" za detalje o promenama u odnosu na Pydantic V1.

## Konverzija podataka

Pydantic može da konvertuje ulazne podatke kako bi ih primorao da se prilagode tipovima polja modela, a u nekim slučajevima to može dovesti do gubitka informacija. 

Na primer:

```py
from pydantic import BaseModel

class Model(BaseModel):
    a: int
    b: float
    c: str

print(Model(a=3.000, b='2.72', c=b'binary data').model_dump())

#> {'a': 3, 'b': 2.72, 'c': 'binary data'}
```

Ovo je namerna odluka kompanije Pydantic i često je najkorisniji pristup. Pogledajte ovo izdanje za dužu diskusiju o ovoj temi.

Ipak, Pydantic pruža [strict mode](https://pydantic.dev/docs/validation/latest/concepts/strict_mode/), gde se ne vrši konverzija podataka. Ulazne vrednosti moraju biti istog tipa kao i deklarisani tip polja.

To je slučaj i sa **kolekcijama**. U većini slučajeva, ne bi trebalo da koristite apstraktne kontejnerske klase, već samo konkretan tip, kao što je **list**:

```py
from pydantic import BaseModel

class Model(BaseModel):
    items: list[int]  

print(Model(items=(1, 2, 3)))
#> items=[1, 2, 3]
```

Osim toga, korišćenje ovih apstraktnih tipova može dovesti i do loših performansi validacije, a generalno korišćenje konkretnih tipova kontejnera će izbeći nepotrebne provere.

## Dodatni podaci

Podrazumevano, Pydantic modeli neće prijaviti grešku kada navedete dodatne podatke, a ove vrednosti će jednostavno biti ignorisane:

```py
from pydantic import BaseModel

class Model(BaseModel):
    x: int

m = Model(x=1, y='a')
assert m.model_dump() == {'x': 1}
```

Vrednost **extra** konfiguracije može se koristiti za kontrolu ovog ponašanja:

```py
from pydantic import BaseModel, ConfigDict

class Model(BaseModel):
    x: int
    model_config = ConfigDict(extra='allow')

m = Model(x=1, y='a')  
assert m.model_dump() == {'x': 1, 'y': 'a'}
assert m.__pydantic_extra__ == {'y': 'a'}
```

Konfiguracija može uzeti tri vrednosti:

- `'ignore'` - Dostavljanje dodatnih podataka se ignoriše (podrazumevano).
- `'forbid'` -Dostavljanje dodatnih podataka nije dozvoljeno.
- `'allow'`- Dozvoljeno je pružanje dodatnih podataka i oni se čuvaju u `__pydantic_extra__` atributu rečnika. `__pydantic_extra__` se mogu eksplicitno anotirati radi validacije dodatnih polja.

Metode validacije (npr. `model_validate()`) imaju opcioni `extra` argument koji će poništiti `extra` vrednost konfiguracije modela za taj poziv validacije.

Za više detalja, pogledajte extraAPI dokumentaciju.

Pajdantičke klase podataka takođe podržavaju dodatne podatke (pogledajte odeljak o konfiguraciji klase podataka).

## Ugnežđeni modeli

Složenije hijerarhijske strukture podataka mogu se definisati korišćenjem samih modela kao tipova u anotacijama.

```py
from typing import Optional
from pydantic import BaseModel

class Foo(BaseModel):
    count: int
    size: Optional[float] = None

class Bar(BaseModel):
    apple: str = 'x'
    banana: str = 'y'

class Spam(BaseModel):
    foo: Foo
    bars: list[Bar]

m = Spam(foo={'count': 4}, bars=[{'apple': 'x1'}, {'apple': 'x2'}])

print(m)

# """
# foo=Foo(count=4, size=None) bars=[Bar(apple='x1', banana='y'), Bar(apple='x2', banana='y')]
# """

print(m.model_dump())

# """
# {
#     'foo': {'count': 4, 'size': None},
#     'bars': [{'apple': 'x1', 'banana': 'y'}, {'apple': 'x2', 'banana': 'y'}],
# }
# """
```

Podržani su modeli sa samoreferenciranjem. Za više detalja pogledajte dokumentaciju vezanu za anotacije unapred.

## Rekonstrukcija šeme modela

Kada definišete klasu modela u svom kodu, Pydantic će analizirati telo klase kako bi prikupio razne informacije potrebne za izvršenje validacije i serijalizacije, prikupljene u osnovnoj šemi. Primetno je da se anotacije tipova modela procenjuju kako bi se razumeli važeći tipovi za svako polje (više informacija možete pronaći u dokumentaciji o arhitekturi). Međutim, može se desiti da se anotacije odnose na simbole koji nisu definisani kada se klasa modela kreira. Da bi se zaobišao ovaj problem, može se koristiti `model_rebuild()` metoda:

```py
from pydantic import BaseModel, PydanticUserError

class Foo(BaseModel):
    x: 'Bar'

try:
    Foo.model_json_schema()
except PydanticUserError as e:
    print(e)

# """
# `Foo` is not fully defined; you should define `Bar`, then call `Foo.model_rebuild()`.
# For further information visit https://errors.pydantic.dev/2/u/class-not-fully-defined
# """

class Bar(BaseModel):
    pass

Foo.model_rebuild()

print(Foo.model_json_schema())

# """
# {
#     '$defs': {'Bar': {'properties': {}, 'title': 'Bar', 'type': 'object'}},
#     'properties': {'x': {'$ref': '#/$defs/Bar'}},
#     'required': ['x'],
#     'title': 'Foo',
#     'type': 'object',
# }
# """
```

Pajdantik pokušava automatski da utvrdi kada je ovo neophodno i da prikaže grešku ako nije urađeno, ali možda ćete želeti da proaktivno pozovete `model_rebuild()` funkciju kada radite sa rekurzivnim modelima ili generičkim modelima.

U V2, `model_rebuild()` zamenjuje `update_forward_refs()` iz V1. Postoje neke manje razlike u novom ponašanju. Najveća promena je to što se prilikom pozivanja `model_rebuild()` najudaljenijeg modela gradi osnovna šema koja se koristi za validaciju celog modela (ugnežđeni modeli i sve ostalo), tako da svi tipovi na svim nivoima moraju biti spremni pre nego što se `model_rebuild()` pozove.

## Validacija podataka

Pydantic može da validira podatke u tri različita režima: Python, JSON i strings.

- Pajton režim se koristi kada se koristi :

  - Konstruktor **__init__()** modela.  
    Vrednosti polja moraju biti navedene korišćenjem ključnih argumenata.
  
  - **model_validate()**: podaci se mogu dati ili kao rečnik ili kao instanca modela  
    (podrazumevano se pretpostavlja da su instance validne; pogledajte podešavanje revalidate_instances). Proizvoljni objekti se takođe mogu dati ako su eksplicitno omogućeni.

- JSON i string režimi mogu se koristiti sa namenskim metodama:

  - **model_validate_json()**: podaci se validiraju kao **JSON** **string** ili **bytes** objekat. 
    Ako su vaši dolazni podaci JSON korisni teret, ovo se generalno smatra bržim (umesto ručnog raščlanjivanja podataka kao rečnika). Saznajte više o JSON raščlanjivanju u JSON dokumentaciji.
  - **model_validate_strings()**: podaci se validiraju kao rečnik (mogu se ugnezditi) sa 
    ključevima i vrednostima stringova i validira podatke u JSON režimu tako da se navedeni stringovi mogu pretvoriti u ispravne tipove.

U poređenju sa korišćenjem konstruktora modela, moguće je kontrolisati nekoliko parametara validacije prilikom korišćenja metoda `model_validate_*()` (strogost, dodatni podaci, kontekst validacije itd.).

> [!Note]Napomena
>
> U zavisnosti od tipova i konfiguracije modela, Python i JSON režimi mogu imati različito ponašanje validacije (npr. sa strogošću ). Ako imate podatke koji dolaze iz izvora koji nije JSON, ali želite isto ponašanje validacije i greške koje biste dobili iz JSON režima, naša preporuka za sada je da ili prebacite svoje podatke u JSON (npr. koristeći json.dumps()), ili koristite model_validate_strings() ako podaci imaju oblik (potencijalno ugnežđenog) rečnika sa string ključevima i vrednostima. Napredak za ovu funkciju može se pratiti u ovom izdanju.

```py
from datetime import datetime
from typing import Optional
from pydantic import BaseModel, ValidationError

class User(BaseModel):
    id: int
    name: str = 'John Doe'
    signup_ts: Optional[datetime] = None

m = User.model_validate({'id': 123, 'name': 'James'})
print(m)

# > id=123 name='James' signup_ts=None

try:
    m = User.model_validate_json('{"id": 123, "name": 123}')
except ValidationError as e:
    print(e)

# """
# 1 validation error for User
# name
#   Input should be a valid string [type=string_type, input_value=123, input_type=int]
# """

# m = User.model_validate_strings({'id': '123', 'name': 'James'})
print(m)

# > id=123 name='James' signup_ts=None

m = User.model_validate_strings(
    {'id': '123', 'name': 'James', 'signup_ts': '2024-04-01T12:00:00'}
)
print(m)

# > id=123 name='James' signup_ts=datetime.datetime(2024, 4, 1, 12, 0)

try:
    m = User.model_validate_strings(
        {'id': '123', 'name': 'James', 'signup_ts': '2024-04-01'}, strict=True
    )
except ValidationError as e:
    print(e)

# """
# 1 validation error for User
# signup_ts
#   Input should be a valid datetime, invalid datetime separator, expected `T`, `t`, `_` or space [type=datetime_parsing, input_value='2024-04-01', input_type=str]
# """
```

## Kreiranje modela bez validacije

Pydantic takođe pruža `model_construct()` metod koji omogućava kreiranje modela bez validacije. Ovo može biti korisno u barem nekoliko slučajeva:

- pri radu sa složenim podacima za koje se već zna da su validni (zbog performansi)
- kada jedna ili više validatorskih funkcija nisu idempotentne
- kada jedna ili više funkcija validatora imaju neželjene efekte koje ne želite da se pokrenu.

> [!Warning]Oprez
>
> `model_construct()` ne vrši nikakvu validaciju, što znači da može da kreira modele koji su nevažeći. Metodu `model_construct()` treba koristiti samo sa podacima koji su već validirani ili kojima definitivno verujete.

> [!Note]Napomena
>
> U Pydantic V2, razlika u performansama između validacije (bilo direktnom instanciracijom ili `model_validate*` metodama) i `model_construct()` je znatno smanjena. Za jednostavne modele, korišćenje validacije može biti čak i brže. Ako ga koristite `model_construct()` iz razloga performansi, možda ćete želeti da profilišete svoj slučaj upotrebe pre nego što pretpostavite da je zapravo brži.

Imajte na umu da se za korenske modele, vrednost korena može proslediti `model_construct()` poziciono, umesto korišćenja ključnog argumenta.

Evo još nekih napomena o ponašanju model_construct():

- Kada kažemo "ne vrši se validacija" — to uključuje pretvaranje rečnika u instance modela. Dakle, ako imate polje koje se odnosi na tip modela, moraćete sami da konvertujete unutrašnji rečnik u model.
- Ako ne prosledite ključne argumente za polja sa podrazumevanim vrednostima, podrazumevane vrednosti će se i dalje koristiti.
- Za modele sa privatnim atributima, `__pydantic_private__` rečnik će biti popunjen na isti način kao što bi bio prilikom kreiranja modela sa validacijom.
- Nijedna `__init__` metoda iz modela ili bilo koje od njegovih roditeljskih klasa neće biti pozvana, čak ni kada je `__init__` definisana prilagođena metoda.

Uključeno dodatni podaci ponašanje sa `model_construct()`

- Za modele sa `extra` podešenim na `'allow'`, podaci koji ne odgovaraju poljima biće ispravno sačuvani u `__pydantic_extra__` rečniku i sačuvani u atributu modela `__dict__`.
- Za modele sa `extra` podešenim na `'ignore'`, podaci koji ne odgovaraju poljima biće ignorisani — to jest, neće se čuvati u `__pydantic_extra__` ili `__dict__` na instanci.
- Za razliku od instanciranja modela sa validacijom, poziv metode ` model_construct()` sa `extra` postavljenim na  `'forbid'` ne izaziva grešku u prisustvu podataka koji ne odgovaraju poljima. Umesto toga, navedeni ulazni podaci se jednostavno ignorišu.

## Definisanje prilagođenog načina `__init__()`

Pajdantik pruža podrazumevanu `__init__()` implementaciju za Pajdantik modele, koja se poziva samo kada se koristi konstruktor modela (a ne sa `model_validate_*()` metodama). Ova implementacija delegira validaciju na pydantic-core.

Međutim, moguće je definisati prilagođenu funkciju `__init__()` na vašim modelima. U ovom slučaju, ona će biti bezuslovno pozvana iz svih metoda validacije, bez izvršavanja validacije (i zato bi trebalo da je pozovete `super().__init__(**kwargs)` u vašoj implementaciji).

Definisanje prilagođenog podešavanja `__init__()` se ne preporučuje, jer će svi parametri validacije (strogost, ponašanje dodatnih podataka, kontekst validacije) biti izgubljeni. Ako je potrebno da izvršite radnje nakon što je model inicijalizovan, možete koristiti validatore polja ili modela posle ili definisati implementaciju `model_post_init()`.

```py
import logging
from typing import Any
from pydantic import BaseModel

class MyModel(BaseModel):
    id: int
    def model_post_init(self, context: Any) -> None:
        logging.info("Model initialized with id %d", self.id)
```

## Obrada grešaka

Pydantic će izazvati izuzetak `ValidationError` kad god pronađe grešku u podacima koje validira.

Jedan izuzetak će biti podignut bez obzira na broj pronađenih grešaka, a ta greška validacije će sadržati informacije o svim greškama i kako su se dogodile.

Pogledajte odeljak "Obrada grešaka" za detalje o standardnim i prilagođenim greškama.

Kao demonstracija:

```py
from pydantic import BaseModel, ValidationError

class Model(BaseModel):
    list_of_ints: list[int]
    a_float: float

data = {
    'list_of_ints': ['1', 2, 'bad'],
    'a_float': 'not a float',
}

try:
    Model(**data)
except ValidationError as e:
    print(e)

# """
# 2 validation errors for Model
# list_of_ints.2
#   Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='bad', input_type=str]
# a_float
#   Input should be a valid number, unable to parse string as a number [type=float_parsing, input_value='not a float', input_type=str]
# """
```

U primeru poput ovog, problem data je upravo u kodu. `ValidationError` uključuje vrednost odbijenu na svakoj neuspešnoj lokaciji, ali u pokrenutoj aplikaciji vam mogu biti potrebni i ti detalji u kontekstu zahteva ili posla. Logfire beleži neuspele validacije sa oba.

## Proizvoljne instance klase

(*Ranije poznato kao "ORM režim"/ from_orm()*).

Kada koristi `model_validate()` metodu, Pydantic takođe može da validira proizvoljne objekte, dobijanjem atributa na objektu koji odgovaraju imenima polja. Jedna uobičajena primena ove funkcionalnosti je integracija sa objektno-relacionim mapiranjima (ORM).

Ovu funkciju je potrebno ručno omogućiti, bilo podešavanjem `from_attributes` vrednosti konfiguracije ili korišćenjem `from_attributes` parametra na `model_validate()`.

Primer ovde koristi SQLAlchemy, ali isti pristup bi trebalo da funkcioniše za bilo koji ORM.

```py
from typing import Annotated
from sqlalchemy import ARRAY, String
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column
from pydantic import BaseModel, ConfigDict, StringConstraints

class Base(DeclarativeBase):
    pass

class CompanyOrm(Base):
    __tablename__ = 'companies'
    id: Mapped[int] = mapped_column(primary_key=True, nullable=False)
    public_key: Mapped[str] = mapped_column(
        String(20), index=True, nullable=False, unique=True
    )
    domains: Mapped[list[str]] = mapped_column(ARRAY(String(255)))

class CompanyModel(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    id: int
    public_key: Annotated[str, StringConstraints(max_length=20)]
    domains: list[Annotated[str, StringConstraints(max_length=255)]]

co_orm = CompanyOrm(
    id=123,
    public_key='foobar',
    domains=['example.com', 'foobar.com'],
)

print(co_orm)

# > <__main__.CompanyOrm object at 0x0123456789ab>

co_model = CompanyModel.model_validate(co_orm)
print(co_model)

#> id=123 public_key='foobar' domains=['example.com', 'foobar.com']
```

## Ugnežđeni atributi

Kada se atributi koriste za validaciju modela, instance modela će biti kreirane i iz atributa najvišeg nivoa i iz dublje ugnežđenih atributa, prema potrebi.

Evo primera koji ilustruje princip:

```py
from pydantic import BaseModel, ConfigDict

class PetCls:
    def __init__(self, *, name: str) -> None:
        self.name = name

class PersonCls:
    def __init__(self, *, name: str, pets: list[PetCls]) -> None:
        self.name = name
        self.pets = pets

class Pet(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    name: str

class Person(BaseModel):
    model_config = ConfigDict(from_attributes=True)
    name: str
    pets: list[Pet]

bones = PetCls(name='Bones')
orion = PetCls(name='Orion')
anna = PersonCls(name='Anna', pets=[bones, orion])
anna_model = Person.model_validate(anna)

print(anna_model)

# > name='Anna' pets=[Pet(name='Bones'), Pet(name='Orion')]
```

## Kopija modela

Metoda `model_copy()` omogućava dupliranje modela (uz opcione nadogradnje), što je posebno korisno pri radu sa zamrznutim modelima.

```py
from pydantic import BaseModel

class BarModel(BaseModel):
    whatever: int

class FooBarModel(BaseModel):
    banana: float
    foo: str
    bar: BarModel

m = FooBarModel(banana=3.14, foo='hello', bar={'whatever': 123})
print(m.model_copy(update={'banana': 0}))
# > banana=0 foo='hello' bar=BarModel(whatever=123)

# normal copy gives the same object reference for bar:
print(id(m.bar) == id(m.model_copy().bar))
# > True

# deep copy gives a new object reference for `bar`:
print(id(m.bar) == id(m.model_copy(deep=True).bar))

#> False
```

## Generički modeli

Pydantic podržava kreiranje generičkih modela kako bi se olakšala ponovna upotreba uobičajene strukture modela. Podržane su i nova sintaksa parametra tipa (uvedena od strane PEP 695 u Python 3.12) i stara sintaksa (pogledajte Python dokumentaciju za više detalja).

Evo primera korišćenja generičkog Pydantic modela za kreiranje lako ponovo upotrebljivog HTTP omotača korisnog opterećenja odgovora:

- Pajton 3.9 i noviji
- Pajton 3.12 i noviji (nova sintaksa)

Evo primera za novu verziju:

```py
from pydantic import BaseModel, ValidationError

class DataModel(BaseModel):
    number: int

class Response[DataT](BaseModel):  
    data: DataT  

print(Response[int](data=1))
#> data=1

print(Response[str](data='value'))
#> data='value'

print(Response[str](data='value').model_dump())
#> {'data': 'value'}

data = DataModel(number=1)
print(Response[DataModel](data=data).model_dump())
#> {'data': {'number': 1}}

try:
    Response[int](data='value')
except ValidationError as e:
    print(e)
# 
# """
# 1 validation error for Response[int]
# data
# Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='value', input_type=str]
# """
# """
```

> [!Note]Novo u verziji 2.11
>
> Puna podrška za sintaksu parametra tipa i podrazumevane vrednosti promenljivih tipa.

> [!Warning]Oprez
>
> Prilikom parametrizacije modela sa konkretnim tipom, Pydantic ne validira da li je dati tip dodeljiv promenljivoj tipa ako ima gornju granicu.

Bilo koja logika konfiguracije, validacije ili serijalizacije podešena na generičkom modelu biće primenjena i na parametrizovane klase, na isti način kao prilikom nasleđivanja iz klase modela. Sve prilagođene metode ili atributi će takođe biti nasleđeni.

Generički modeli se takođe pravilno integrišu sa proveravačima tipova, tako da dobijate svu proveru tipova koju biste očekivali ako biste deklarisali poseban tip za svaku parametrizaciju.

> [!Note]Napomena
>
> Interno, Pydantic kreira podklase generičkog modela tokom izvršavanja programa kada je klasa generičkog modela parametrizovana. Ove klase su keširane, tako da bi korišćenje generičkih modela trebalo da stvori minimalno opterećenje.

Da bi nasledila generički model i sačuvala činjenicu da je on generički, podklasa takođe mora naslediti od Generic:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel

TypeX = TypeVar('TypeX')

class BaseClass(BaseModel, Generic[TypeX]):
    X: TypeX

class ChildClass(BaseClass[TypeX], Generic[TypeX]):
    pass

# Parametrize `TypeX` with `int`:
print(ChildClass[int](X=1))

# > X=1
```

Takođe možete kreirati generičku podklasu modela koja delimično ili potpuno zamenjuje promenljive tipa u nadklasi:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel

TypeX = TypeVar('TypeX')
TypeY = TypeVar('TypeY')
TypeZ = TypeVar('TypeZ')

class BaseClass(BaseModel, Generic[TypeX, TypeY]):
    x: TypeX
    y: TypeY

class ChildClass(BaseClass[int, TypeY], Generic[TypeY, TypeZ]):
    z: TypeZ

# Parametrize `TypeY` with `str`:
print(ChildClass[str, int](x='1', y='y', z='3'))

# > x=1 y='y' z=3
```

Ako je ime konkretnih podklasa važno, možete takođe poništiti podrazumevano generisanje imena tako što ćete poništiti metodu model_parametrized_name():

```py
from typing import Any, Generic, TypeVar
from pydantic import BaseModel

DataT = TypeVar('DataT')

class Response(BaseModel, Generic[DataT]):
    data: DataT
    @classmethod
    def model_parametrized_name(cls, params: tuple[type[Any],...]) -> str:
        return f'{params[0].__name__.title()}Response'

print(repr(Response[int](data=1)))
#> IntResponse(data=1)

print(repr(Response[str](data='a')))
#> StrResponse(data='a')
```

Možete koristiti parametrizovane generičke modele kao tipove u drugim modelima:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel

T = TypeVar('T')

class ResponseModel(BaseModel, Generic[T]):
    content: T

class Product(BaseModel):
    name: str
    price: float

class Order(BaseModel):
    id: int
    product: ResponseModel[Product]

product = Product(name='Apple', price=0.5)
response = ResponseModel[Product](content=product)
order = Order(id=1, product=response)

print(repr(order))

# """
# Order(id=1, product=ResponseModel[Product](content=Product(name='Apple', price=0.5)))
# """
```

Korišćenje promenljive istog tipa u ugnežđenim modelima vam omogućava da sprovedete odnose kucanja na različitim tačkama u vašem modelu:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel, ValidationError

T = TypeVar('T')

class InnerT(BaseModel, Generic[T]):
    inner: T

class OuterT(BaseModel, Generic[T]):
    outer: T
    nested: InnerT[T]

nested = InnerT[int](inner=1)

print(OuterT[int](outer=1, nested=nested))
#> outer=1 nested=InnerT[int](inner=1)

try:
    print(OuterT[int](outer='a', nested=InnerT(inner='a')))  
except ValidationError as e:
    print(e)

# """
# 2 validation errors for OuterT[int]
# outer
#   Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='a', input_type=str]

# nested.inner
#   Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='a', input_type=str]
# """
```

> [!Warning]Oprez
>
> Iako možda neće izazvati grešku, toplo savetujemo da ne koristite parametrizovane generike u `isinstance()` proverama.
>
> Na primer, ne bi trebalo da radite `isinstance(my_model, MyGenericModel[int])`. Međutim, u redu je da uradite `isinstance(my_model, MyGenericModel`)(imajte na umu da bi, za standardne generičke klase, provera podklase sa parametrizovanom generičkom klasom izazvala grešku).
>
> Ako treba da izvršite `isinstance()` provere parametrizovanih generičkih klasa, to možete učiniti tako što ćete podklasirati parametrizovanu generičku klasu:
>
> ```py
> class MyIntModel(MyGenericModel[int]):...
> isinstance(my_model, MyIntModel)
> ```

## Detalji implementacije

### Validacija neparametrizovanih promenljivih tipa

Kada ostavi promenljive tipa neparametrizovane, Pydantic tretira generičke modele slično kao što tretira ugrađene generičke tipove kao što su listi dict:

- Ako je promenljiva tipa vezana ili ograničena na određeni tip, ona će biti korišćena.
- Ako promenljiva tipa ima podrazumevani tip (kako je navedeno u PEP 696 ), ona će biti korišćena.
- Za neograničene ili neograničene promenljive tipa, Pydantic će se vratiti na Any.

```py
from typing import Generic
from typing_extensions import TypeVar
from pydantic import BaseModel, ValidationError

T = TypeVar('T')
U = TypeVar('U', bound=int)
V = TypeVar('V', default=str)

class Model(BaseModel, Generic[T, U, V]):
    t: T
    u: U
    v: V

print(Model(t='t', u=1, v='v'))
#> t='t' u=1 v='v'

try:
    Model(t='t', u='u', v=1)
except ValidationError as exc:
    print(exc)

# """
# 2 validation errors for Model
# u
#   Input should be a valid integer, unable to parse string as an integer [type=int_parsing, input_value='u', input_type=str]
# v
#   Input should be a valid string [type=string_type, input_value=1, input_type=int]
# """
```

> [!Warning]Oprez
>
> U nekim slučajevima, validacija u odnosu na neparametrizovani generički model može dovesti do gubitka podataka. Konkretno, ako se koristi podtip promenljive tipa gornja granica, ograničenja ili podrazumevana vrednost, a model nije eksplicitno parametrizovan, rezultujući tip neće biti onaj koji je dat:
>
> ```py
> from typing import Generic, TypeVar
> from pydantic import BaseModel
> 
> ItemT = TypeVar('ItemT', bound='ItemBase')
> 
> class ItemBase(BaseModel):...
> 
> class IntItem(ItemBase):
>     value: int
> 
> class ItemHolder(BaseModel, Generic[ItemT]):
>     item: ItemT
> 
> loaded_data = {'item': {'value': 1}}
> print(ItemHolder(**loaded_data))  
> #> item=ItemBase()
> 
> print(ItemHolder[IntItem](**loaded_data))  
> #> item=IntItem(value=1)
> ```

### Serijalizacija neparametrizovanih promenljivih tipa

Ponašanje serijalizacije se razlikuje kada se koriste promenljive tipa sa gornjim granicama, ograničenjima ili podrazumevanom vrednošću:

Ako se Pydantic model koristi u gornjoj granici promenljive tipa i promenljiva tipa nikada nije parametrizovana, onda će Pydantic koristiti gornju granicu za validaciju, ali će vrednost tretirati kao `Any` u smislu serijalizacije:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel

class ErrorDetails(BaseModel):
    foo: str

ErrorDataT = TypeVar('ErrorDataT', bound=ErrorDetails)

class Error(BaseModel, Generic[ErrorDataT]):
    message: str
    details: ErrorDataT

class MyErrorDetails(ErrorDetails):
    bar: str

# serialized as Any
error = Error(
    message='We just had an error',
    details=MyErrorDetails(foo='var', bar='var2'),
)

assert error.model_dump() == {
    'message': 'We just had an error',
    'details': {
        'foo': 'var',
        'bar': 'var2',
    },
}

# serialized using the concrete parametrization
# note that `'bar': 'var2'` is missing
error = Error[ErrorDetails](
    message='We just had an error',
    details=ErrorDetails(foo='var'),
)

assert error.model_dump() == {
    'message': 'We just had an error',
    'details': {
        'foo': 'var',
    },
}
```

Evo još jednog primera gore navedenog ponašanja, nabrajajući sve permutacije u vezi sa ograničenom specifikacijom i parametrizacijom generičkog tipa:

```py
from typing import Generic, TypeVar
from pydantic import BaseModel

TBound = TypeVar('TBound', bound=BaseModel)
TNoBound = TypeVar('TNoBound')

class IntValue(BaseModel):
    value: int

class ItemBound(BaseModel, Generic[TBound]):
    item: TBound

class ItemNoBound(BaseModel, Generic[TNoBound]):
    item: TNoBound

item_bound_inferred = ItemBound(item=IntValue(value=3))
item_bound_explicit = ItemBound[IntValue](item=IntValue(value=3))
item_no_bound_inferred = ItemNoBound(item=IntValue(value=3))
item_no_bound_explicit = ItemNoBound[IntValue](item=IntValue(value=3))
# calling `print(x.model_dump())` on any of the above instances results in the following:
#> {'item': {'value': 3}}
```

Međutim, ako se koriste ograničenja ili podrazumevana vrednost (prema PEP 696 ), onda će se podrazumevani tip ili ograničenja koristiti i za validaciju i za serijalizaciju ako promenljiva tipa nije parametrizovana. Ovo ponašanje možete poništiti pomoću SerializeAsAny:

```py
from typing import Generic
from typing_extensions import TypeVar
from pydantic import BaseModel, SerializeAsAny

class ErrorDetails(BaseModel):
    foo: str

ErrorDataT = TypeVar('ErrorDataT', default=ErrorDetails)

class Error(BaseModel, Generic[ErrorDataT]):
    message: str
    details: ErrorDataT

class MyErrorDetails(ErrorDetails):
    bar: str

# serialized using the default's serializer
error = Error(
    message='We just had an error',
    details=MyErrorDetails(foo='var', bar='var2'),
)

assert error.model_dump() == {
    'message': 'We just had an error',
    'details': {
        'foo': 'var',
    },
}

# If `ErrorDataT` was using an upper bound, `bar` would be present in `details`.
class SerializeAsAnyError(BaseModel, Generic[ErrorDataT]):
    message: str
    details: SerializeAsAny[ErrorDataT]

# serialized as Any
error = SerializeAsAnyError(
    message='We just had an error',
    details=MyErrorDetails(foo='var', bar='baz'),
)

assert error.model_dump() == {
    'message': 'We just had an error',
    'details': {
        'foo': 'var',
        'bar': 'baz',
    },
}
```

## Kreiranje dinamičkog modela

Postoje neke prilike kada je poželjno kreirati model koristeći informacije o izvršavanju za određivanje polja. Pydantic pruža funkciju `create_model()` koja omogućava dinamičko kreiranje modela:

```py
from pydantic import BaseModel, create_model

DynamicFoobarModel = create_model('DynamicFoobarModel', foo=str, bar=(int, 123))

# Equivalent to:
class StaticFoobarModel(BaseModel):
    foo: str
    bar: int = 123
```

Definicije polja su navedene kao ključni argumenti i treba da budu:

- Jedan element, koji predstavlja anotaciju tipa polja.
- Dvostruki skup, gde je prvi element tip, a drugi element dodeljena vrednost (ili podrazumevana vrednost ili Field()funkcija).

> [!Note]↻ Promenjeno u verziji 2.11
>
> Prilikom navođenja jednog elementa za definicije polja, može se koristiti bilo koji tip (ranije Annotatedje mogao biti naveden samo obrazac).

Evo jednog naprednijeg primera:

```py
from typing import Annotated
from pydantic import BaseModel, Field, PrivateAttr, create_model

DynamicModel = create_model(
    'DynamicModel',
    foo=(str, Field(alias='FOO')),
    bar=Annotated[str, Field(description='Bar field')],
    _private=(int, PrivateAttr(default=1)),
)

class StaticModel(BaseModel):
    foo: str = Field(alias='FOO')
    bar: Annotated[str, Field(description='Bar field')]
    _private: int = PrivateAttr(default=1)
```

Posebne ključne reči, argumenti `__config__`, i, `__base__` mogu se koristiti za prilagođavanje novog modela. Ovo uključuje proširivanje osnovnog modela dodatnim poljima.

```py
from pydantic import BaseModel, create_model

class FooModel(BaseModel):
    foo: str
    bar: int = 123

BarModel = create_model(
    'BarModel',
    apple=(str, 'russet'),
    banana=(str, 'yellow'),
    __base__=FooModel,
)

print(BarModel)
#> <class '__main__.BarModel'>

print(BarModel.model_fields.keys())
#> dict_keys(['foo', 'bar', 'apple', 'banana'])
```

Validatore možete dodati i tako što ćete `__validators__` argumentu proslediti rečnik.

```py
from pydantic import ValidationError, create_model, field_validator

def alphanum(cls, v):
    assert v.isalnum(), 'must be alphanumeric'
    return v

validators = {
    'username_validator': field_validator('username')(alphanum)  
}

UserModel = create_model(
    'UserModel', username=(str,...), __validators__=validators
)

user = UserModel(username='scolvin')
print(user)
#> username='scolvin'

try:
    UserModel(username='scolvi%n')
except ValidationError as e:
    print(e)

# """
# 1 validation error for UserModel
# username
#   Assertion failed, must be alphanumeric [type=assertion_error, input_value='scolvi%n', input_type=str]
# """
```

> [!Note]Napomena
>
> Da biste pickle dinamički kreirani model:
>
> - model mora biti definisan globalno
> - argument __module__mora biti naveden

> [!Warning]Oprez
>
> Ova funkcija može da izvrši proizvoljni kod sadržan u anotacijama polja, ako je potrebno da se procene reference na nizove.
> 
> Više informacija potražite u odeljku "Bezbednosne implikacije introspektivnih anotacija".

Vidite takođe: primer dinamičkog modela, koji pruža smernice za izvođenje opcionog modela iz drugog.

## RootModel i prilagođene tipove korena

Pidantični modeli mogu se definisati sa "prilagođenim tipom korena" podklasiranjem `pydantic.RootModel`.

Tip korena može biti bilo koji tip koji podržava Pydantic i određuje se generičkim parametrom `RootModel`. Vrednost korena može se proslediti model `__init__` ili `model_validate` preko prvog i jedinog argumenta.

Evo primera kako ovo funkcioniše:

```py
from pydantic import RootModel

Pets = RootModel[list[str]]
PetsByName = RootModel[dict[str, str]]

print(Pets(['dog', 'cat']))
#> root=['dog', 'cat']

print(Pets(['dog', 'cat']).model_dump_json())
#> ["dog","cat"]

print(Pets.model_validate(['dog', 'cat']))
#> root=['dog', 'cat']

print(Pets.model_json_schema())
"""
{'items': {'type': 'string'}, 'title': 'RootModel[list[str]]', 'type': 'array'}
"""

print(PetsByName({'Otis': 'dog', 'Milo': 'cat'}))
#> root={'Otis': 'dog', 'Milo': 'cat'}

print(PetsByName({'Otis': 'dog', 'Milo': 'cat'}).model_dump_json())
#> {"Otis":"dog","Milo":"cat"}

print(PetsByName.model_validate({'Otis': 'dog', 'Milo': 'cat'}))
#> root={'Otis': 'dog', 'Milo': 'cat'}
```

Ako želite direktno da pristupite root stavkama u polju ili da iterirate kroz stavke, možete implementirati prilagođene funkcije `__iter__` i `__getitem__`, kao što je prikazano u sledećem primeru.

```py
from pydantic import RootModel

class Pets(RootModel):
    root: list[str]
    def __iter__(self):
        return iter(self.root)
    def __getitem__(self, item):
        return self.root[item]

pets = Pets.model_validate(['dog', 'cat'])

print(pets[0])
#> dog

print([pet for pet in pets])
#> ['dog', 'cat']
```

Takođe možete direktno kreirati podklase parametrizovanog korenskog modela:

```py
from pydantic import RootModel

class Pets(RootModel[list[str]]):
    def describe(self) -> str:
        return f'Pets: {", ".join(self.root)}'

my_pets = Pets.model_validate(['dog', 'cat'])

print(my_pets.describe())
#> Pets: dog, cat
```

## Lažna nepromenljivost

Modeli se mogu konfigurisati da budu nepromenljivi putem `model_config['frozen'] = True`. Kada je ovo podešeno, pokušaj promene vrednosti atributa instance će izazvati greške. Pogledajte API referencu za više detalja.

> [!Note]Napomena
>
> Ovo ponašanje je postignuto u Pydantic V1 putem podešavanja konfiguracije `allow_mutation = False`. Ova konfiguraciona zastavica je zastarela u Pydantic V2 i zamenjena je sa `frozen`.

> [!Warning]Oprez
>
> U Pajtonu, nepromenljivost se ne nameće. Programeri imaju mogućnost da modifikuju objekte koji se konvencionalno smatraju "nepromenljivim" ako to žele.

```py
from pydantic import BaseModel, ConfigDict, ValidationError

class FooBarModel(BaseModel):
    model_config = ConfigDict(frozen=True)
    a: str
    b: dict

foobar = FooBarModel(a='hello', b={'apple': 'pear'})

try:
    foobar.a = 'different'
except ValidationError as e:
    print(e)

# """
# 1 validation error for FooBarModel
# a
#   Instance is frozen [type=frozen_instance, input_value='different', input_type=str]
# """

print(foobar.a)
#> hello

print(foobar.b)
#> {'apple': 'pear'}

foobar.b['apple'] = 'grape'

print(foobar.b)
#> {'apple': 'grape'}
```

Pokušaj promene "a" je izazvao grešku i "a" ostaje nepromenjen. Međutim, rečnik "b" je promenljiv, a nepromenljivost "foobar" ne sprečava "b" da se menja.

## Apstraktne osnovne klase

Pidantični modeli se mogu koristiti zajedno sa Pajtonovim apstraktnim osnovnim klasama (ABC).

```py
import abc
from pydantic import BaseModel

class FooBarModel(BaseModel, abc.ABC):
    a: str
    b: int
    @abc.abstractmethod
    def my_abstract_method(self):
        pass
```

## Redosled polja

Redosled polja utiče na modele na sledeće načine:

- Redosled polja je sačuvan u JSON šemi modela
- Redosled polja je očuvan kod grešaka validacije
- Redosled polja se čuva prilikom serijalizacije podataka

```py
from pydantic import BaseModel, ValidationError

class Model(BaseModel):
    a: int
    b: int = 2
    c: int = 1
    d: int = 0
    e: float

print(Model.model_fields.keys())
#> dict_keys(['a', 'b', 'c', 'd', 'e'])

m = Model(e=2, a=1)
print(m.model_dump())
#> {'a': 1, 'b': 2, 'c': 1, 'd': 0, 'e': 2.0}

try:
    Model(a='x', b='x', c='x', d='x', e='x')
except ValidationError as err:
    error_locations = [e['loc'] for e in err.errors()]

print(error_locations)
#> [('a',), ('b',), ('c',), ('d',), ('e',)]
```

## Automatski isključeni atributi

### Promenljive klase

Atributi označeni sa `ClassVar` se pravilno tretiraju od strane Pydantic-a kao promenljive klase i neće postati polja na instancama modela:

```py
from typing import ClassVar
from pydantic import BaseModel

class Model(BaseModel):
    x: ClassVar[int] = 1
    y: int = 2

m = Model()
print(m)
#> y=2

print(Model.x)
#> 1
```

### Privatni atributi modela

Atributi čije ime ima vodeću donju crtu, Pydantic ne tretira kao polja i nisu uključeni u šemu modela. Umesto toga, oni se konvertuju u `private atribut`" koji se ne validira niti postavlja tokom poziva `__init__`, `model_validate`, itd.

Evo primera upotrebe:

```py
from datetime import datetime
from random import randint
from typing import Any
from pydantic import BaseModel, PrivateAttr

class TimeAwareModel(BaseModel):
    _processed_at: datetime = PrivateAttr(default_factory=datetime.now)
    _secret_value: str
    
    def model_post_init(self, context: Any) -> None:
        # this could also be done with `default_factory`:
        self._secret_value = randint(1, 5)

m = TimeAwareModel()
print(m._processed_at)
#> 2032-01-02 03:04:05.000006

print(m._secret_value)
#> 3
```

Imena privatnih atributa moraju početi donjom crtom kako bi se sprečili sukobi sa poljima modela. Međutim, imena "dunder" atributa (kao što je `__attr__`) nisu podržana i biće potpuno ignorisana iz definicije modela.

> [!Note]Novo u verziji 2.13
>
> Podrazumevane fabrike mogu da prihvate validirane podatke modela kao argument.

## Potpis modela

Svi Pydantic modeli će imati generisan potpis na osnovu svojih polja:

```py
import inspect
from pydantic import BaseModel, Field

class FooModel(BaseModel):
    id: int
    name: str = None
    description: str = 'Foo'
    apple: int = Field(alias='pear')

print(inspect.signature(FooModel))
#> (*, id: int, name: str = None, description: str = 'Foo', pear: int) -> None
```

Tačan potpis je koristan za potrebe introspekcije i biblioteka poput "FastAPI" ili "hypothesis".

Generisani potpis će takođe poštovati prilagođene `__init__` funkcije:

```py
import inspect
from pydantic import BaseModel

class MyModel(BaseModel):
    id: int
    info: str = 'Foo'

    def __init__(self, id: int = 1, *, bar: str, **data) -> None:
        """My custom init!"""
        super().__init__(id=id, bar=bar, **data)

print(inspect.signature(MyModel))
#> (id: int = 1, *, bar: str, info: str = 'Foo') -> None
```

Da bi se uključilo u potpis, alias ili ime polja mora biti važeći Pajton identifikator. Pydantic će dati prioritet alijasu polja u odnosu na njegovo ime prilikom generisanja potpisa, ali može koristiti ime polja ako alias nije važeći Pajton identifikator.

Ako i alias i ime polja nisu validni identifikatori (što je moguće egzotičnom upotrebom `create_model`), biće dodat `**data` argument. Pored toga, `**data` argument će uvek biti prisutan u potpisu ako je `model_config['extra'] == 'allow'`.
Strukturno podudaranje obrazaca

Pydantic podržava strukturno uparivanje obrazaca za modele, kako je predstavljeno od strane PEP 636 u Python 3.10.

```py
from pydantic import BaseModel

class Pet(BaseModel):
    name: str
    species: str

a = Pet(name='Bones', species='dog')

match a:
    # match `species` to 'dog', declare and initialize `dog_name`
    case Pet(species='dog', name=dog_name):
        print(f'{dog_name} is a dog')

#> Bones is a dog

    # default case
    case _:
        print('No dog matched')
```

> [!Note]Napomena

> Izjava za podudaranje velikih i malih slova može izgledati kao da kreira novi model, ali nemojte se zavaravati; to je samo sintaksički šećer za dobijanje atributa i njegovo upoređivanje ili deklarisanje i inicijalizaciju.

## Kopije atributa

U mnogim slučajevima, argumenti prosleđeni konstruktoru biće kopirani kako bi se izvršila validacija i, gde je potrebno, prinuda.

U ovom primeru, imajte na umu da se ID liste menja nakon što je klasa konstruisana jer je kopirana tokom validacije:

```py
from pydantic import BaseModel

class C1:
    arr = []

    def __init__(self, in_arr):
        self.arr = in_arr

class C2(BaseModel):
    arr: list[int]

arr_orig = [1, 9, 10, 3]
c1 = C1(arr_orig)
c2 = C2(arr=arr_orig)

print(f'{id(c1.arr) == id(c2.arr)=}')
#> id(c1.arr) == id(c2.arr)=False
```

> [!Note]Napomena
>
> Postoje neke situacije gde Pydantic ne kopira atribute, kao na primer prilikom prosleđivanja modela — koristimo model kakav jeste. Ovo ponašanje možete poništiti podešavanjem `model_config['revalidate_instances'] = 'always'`.
