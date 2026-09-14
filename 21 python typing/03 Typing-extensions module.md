# Uvod u modul `typing-extensions` u Pajtonu

Modul `typing-extensions` pruža bekportove najnovijih funkcija za tipiziranje kako bi se osiguralo da programeri koji rade sa starijim verzijama Pajtona i dalje mogu da koriste ove napredne alate. Ovaj modul deluje kao most između budućih izdanja Pajtona i postojećih baza koda, omogućavajući nam da budemo u toku sa modernim praksama nagoveštavanja tipova bez ograničenja naše verzije Pajtona.

U ovom članku ćemo istražiti modul `typing-extensions`, njegove osnovne karakteristike i kako se razlikuje od standardnog modula za tipiziranje, i pružiti praktične primere kako bismo demonstrirali njegovu korisnost u realnim scenarijima.

## Šta je modul `typing-extensions`

### Definicija i svrha

Modul `typing-extensions` je neophodna biblioteka u Pajtonu koja omogućava programerima da koriste napredne funkcije nagoveštavanja tipova pre nego što budu zvanično dodate standardnom modulu za tipiziranje. Kako se Pajtonov sistem tipova razvija, nove funkcije i uslužni programi se uvode u novije verzije Pajtona. Međutim, mnogi programeri rade sa starijim verzijama Pajtona, a nadogradnja možda nije uvek izvodljiva.

Tu dolazi do izražaja `typing-extensions` — on pruža bekportove ovih novih karakteristika tipova, čineći ih dostupnim u starijim verzijama Pajtona. Bez obzira da li radimo sa starijim sistemima ili treba da osiguramo kompatibilnost između različitih Pajton okruženja, `typing-extensions` nam omogućava da usvojimo najnovije prakse nagoveštavanja tipova bez ograničenja našom verzijom Pajtona.

### Vodič za instalaciju

Da biste koristili modul `typing-extensions`, instalirajte ih putem pip-a :

```sh
pip install typing_extensions
```

## Osnovne karakteristike `typing-extensions`

Modul `typing-extensions` pruža nekoliko moćnih tipova i uslužnih programa koji poboljšavaju mogućnosti nagoveštavanja tipova u Pajtonu. Ispod je pregled najznačajnijih tipova i funkcija koje uvodi.

Pregled dostupnih tipova i uslužnih programa

- **Annotaded**: Dodaje metapodatke savetima za tip.
- **TypedDict**: Definiše rečnike sa određenim ključevima i tipovima vrednosti.
- **Literal**: Ograničava promenljivu na skup unapred definisanih vrednosti.
- **Protocol**: Omogućava strukturno podtipizovanje.
- **Final**: Osigurava da se promenljiva ili metod ne može prepisati.
- **TypeAlias**: Omogućava kreiranje alijas imena za tipove.
- **Self**: Predstavlja instancu klase u anotacijama metoda.

1. **Annotaded** tip

   Tip `Annotated` omogućava programerima da dodaju metapodatke postojećim tipovima. Ovo je korisno kada želimo da proširimo informacije o tipu dodatnim anotacijama koje ne utiču na ponašanje tokom izvršavanja, ali mogu pružiti vredne savete za alate za statičku analizu ili druge okvire.

   ```py
   from typing_extensions import Annotated
   
   # Metadata added to specify units
   def calculate_distance(speed: Annotated[int, 'km/h'], time: Annotated[int, 'hours']) -> int:
       return speed * time
   ```

2. **TypedDict** i njegove varijante

   **TypedDict** omogućava programerima da odrede strukturu rečnika, primenjujući i ključeve i njihove povezane tipove. Pomaže u osiguravanju da strukture podataka slične rečnicima odgovaraju očekivanim tipovima, što je posebno korisno za API-je i konfiguracione datoteke.

   **Required i NotRequired**: Ove vrednosti pomažu u određivanju da li su određena polja u TypedDict - u obavezna ili opcionalna.

    ```py
    from typing_extensions import NotRequired
   
    class EmployeeOptional(TypedDict):
        name: str
        age: NotRequired[int]  # Optional field
   
    employee = EmployeeOptional(name="Arun")
    print(employee)
    ```

3. **Literal** tip

   Tip `Literal` ograničava promenljivu na određeni skup konstantnih vrednosti. Ovo je korisno kada želimo da ograničimo moguće vrednosti koje promenljiva može da uzme, obezbeđujući bolju bezbednost tipa.

    ```py
    from typing_extensions import Literal
    
    def set_environment(env: Literal['development', 'production']) -> None:
        if env not in ['development', 'production']:
            raise Exception("Unknown env value!")
        print(f"Setting environment to {env}")
    
    set_environment('development')
    set_environment('staging') # Raises an error
    ```

4. **Protocol** tip i strukturni podtipovi

   `Protocol` omogućava strukturno podtipizovanje, što znači da se klasa može smatrati podtipom protokola ako implementira potrebne metode, bez obzira na to da li eksplicitno nasleđuje iz protokola.

    ```py
    from typing_extensions import Protocol
    
    class Drivable(Protocol):
        def drive(self) -> None:
            pass
    
    class Car:
        def drive(self) -> None:
            print("Driving a car")
    
    def test_drive(vehicle: Drivable) -> None:
        vehicle.drive()
    
    
    car = Car()
    test_drive(car)
    
    # Izlaz:
    # Driving a car
    ```

5. **Final** ključna reč za konstante i klase

   Ključna reč Final ukazuje na to da promenljiva, metoda ili klasa ne treba da budu konstante i govori da se ne prepisuju ili ponovo dodeljuju, osiguravajući nepromenljivost i štiteći ključne delove našeg koda od nenamernih promena.

    ```py
    from typing_extensions import Final
    
    PI: Final = 3.14159
    ```

6. **TypeAlias** ​​za kreiranje alijasa

   TypeAlias ​​nam omogućava da kreiramo smislene alijase za složene naznake tipova, poboljšavajući čitljivost našeg koda. Ovo je posebno korisno kada se radi sa složenim tipovima koji se ponovo koriste na više mesta.

    ```py
    from typing_extensions import TypeAlias

    # Alias for a tuple of two integers
    Coordinate: TypeAlias = tuple[int, int]
    
    def move_to(position: Coordinate) -> None:
        print(f"Moving to {position}")
    
    move_to((10, 20))
    
    # Izlaz:
    # Moving to (10, 20)
    ```

7. **Self** tip za metode koje vraćaju instance

   Tip Self pojednostavljuje anotacije metoda tako što nam omogućava da naznačimo da metod vraća instancu klase kojoj pripada. Ovo je posebno korisno za fluentne interfejse, gde metode vraćaju isti objekat za ulančavanje metoda.

    ```py
    from typing_extensions import Self
    
    class Builder:
        def set_name(self, name: str) -> Self:
            self.name = name
            return self
    
        def build(self) -> dict:
            return {'name': self.name}
    
    builder = Builder().set_name('Example').build()
    print(builder)
    
    Izlaz:
    
    {'name': 'Example'}
    ```

Ove osnovne karakteristike `typing-extensions` značajno proširuju mogućnosti nagoveštavanja tipova u Pajtonu, omogućavajući programerima da pišu čistiji i robusniji kod, a istovremeno osiguravaju kompatibilnost sa starijim verzijama Pajtona.

## Poređenje između typing i typing-extensions

| Aspekt | Modul typing | Modul typing-extensions |
| ------ | ------------ | ----------------------- |
| **Svrha** | Pruža ugrađene savete za tipove za Pajton 3.5+ | Nudi rezervne kopije novih funkcija kucanja koje još nisu dostupne u kucanju |
| **Dostupnost** | Standardna Pajton biblioteka, dostupna u svim Pajton 3.5+ verzijama | Spoljni paket koji treba instalirati zasebno |
| **Osnovne karakteristike** | Uobičajeni tipovi saveta kao što su List, Dict, Union, Optional | Dodatni tipovi kao što su Literal, TypedDict, Protocol, Final |
| **Kompatibilnost unazad** | Karakteristike vezane za verziju Pajtona koja se koristi | Vraća funkcije tipiziranja na starije verzije Pajtona |
| **Slučaj upotrebe za uobičajene savete o tipovima** | Koristi se za osnovno nagoveštavanje tipova u modernim Pajton kodnim bazama | Koristite za napredne ili eksperimentalne savete o tipovima u starijim bazama koda |
| **Strukturno podtipizovanje** | Ograničena podrška putem typing.Protocol-a (Python 3.8+) | Puna podrška putem typing_extensions.Protocol za starije verzije |
| **Nepromenljive promenljive/metode** | Podržava Final od Pajtona 3.8+ | Pruža finalnu verziju za verzije pre Pajtona 3.8 |
| **Anotacije metapodataka tipa** | Nije dostupno | Podržava anotiranje za dodavanje metapodataka savetima za kucanje |
| **TypedDict** | Dostupno kao TypedDict u Pajtonu 3.8+ | Obezbeđuje TypedDict za verzije pre Pajtona 3.8 |
| **Saveti za literalni tip** | Dostupno kao literal u Pajtonu 3.8+ | Pruža Literal za verzije pre Pajtona 3.8 |
| **Kada se koristi** | Koristi se u većini slučajeva gde je Python 3.8+ ili noviji osnovna verzija | Koristite kada radite sa starijim verzijama Pajtona ili kada su vam potrebne buduće funkcije |
| **Kodeks za budućnost** | Već integrisano u standardnu biblioteku Pajtona | Koristite za rano usvajanje novih funkcija pre nego što se integrišu |
| **Instalacija** | Nije potrebna instalacija, deo je Pajtona | Zahteva instalaciju `pip install typing-extensions` |

## Efikasna upotreba typing i typing-extensions zajedno

Moduli `typing` i `typing-extensions` se često koriste zajedno, posebno u projektima kojima je potrebna podrška i starijim i novijim verzijama Pajtona. Cilj je iskoristiti najnaprednije funkcije nagoveštavanja tipova uz očuvanje kompatibilnosti sa unazadnim verzijama. U nastavku su navedene najbolje prakse za zajedničko korišćenje ovih modula, primeri scenarija gde su oba neophodna i strategije za prelazak na `typing` kako se nove funkcije integrišu u standardnu biblioteku Pajtona.

### Najbolje prakse

- Podrazumevano za typing: Koristite ugrađeni modul typing za standardne savete o tipovima u Pajtonu 3.5 i novijim verzijama.
- Koristite `typing-exstensions` za napredne funkcije: Koristite `typing-exstensions` za funkcije kao što su `TypedDict`, `Literal` i `Final` u starijim verzijama Pajtona.
- Alias uvoza radi jasnoće: Razmotrite aliasiranje uvoza da biste naznačili njihov izvorni modul.
- Uslovni uvoz: Proverite dostupnost funkcija pre uvoza iz `typing` ili `typing-extensions`.

### Primeri scenarija

- Kompatibilnost unazad: Koristite oba modula kada podržavate više verzija Pajtona.
- Pre usvajanja novih funkcija: Počnite da koristite `typing-exstensions` za funkcije koje se očekuju u budućim izdanjima Pajtona.

## Prelazak sa `typing-exstensions` na `typing`

- Pratite ažuriranja Pajtona: Redovno proveravajte beleške o izdanjima za nove verzije Pajtona da biste videli koje su funkcije iz `typing-extensions` integrisane u `typing`.
- Koristite uslovni uvoz: Implementirajte uslovni uvoz da biste prešli sa `typing-exstensions` na `typing` kada je funkcija integrisana u Pajton.
- Postepeno refaktorisanje: Kada naš projekat bude na verziji koja izvorno podržava potrebne funkcije, ažurirajte uvoz iz `typing-extensions` u `typing`.
- Ažuriranje zavisnosti: Uklonite `typing-extensions` iz vaše datoteke `requirements.txt` ili datoteka za upravljanje zavisnostima ako više nisu potrebne.
- Testiranje i validacija: Pokrenite testove nakon tranzicije kako biste osigurali da nagoveštavanje tipova i dalje funkcioniše kako se očekuje. Koristite alate poput `mypy` za validaciju.

### Primer

Evo primera korišćenja `TypedDict` i `Literal`:

```py
from typing_extensions import TypedDict, Literal

class AppConfig(TypedDict):
    app_name: str
    version: str
    environment: Literal['development', 'production']

def show_app_config(config: AppConfig) -> None:
    print(f"App: {config['app_name']}, Version: {config['version']}, Environment: {config['environment']}")

config = {'app_name': 'MyApp', 'version': '1.0', 'environment': 'development'}
show_app_config(config)

# Izlaz:
# App: MyApp, Version: 1.0, Environment: development
```

Ovaj kod pokazuje kako se koristi `TypedDict` za sprovođenje određene strukture rečnika i kako se koristi `Literal` za ograničavanje vrednosti određenog ključa. Funkcija `show_app_config` zatim koristi ove strukturirane podatke za ispis konfiguracije aplikacije u jasnom formatu. Ovaj pristup poboljšava jasnoću koda i smanjuje verovatnoću grešaka povezanih sa netačnim strukturama podataka ili vrednostima.

### Uobičajene zamke

Uobičajena greška je mešanje anotacija pri tipiziranju. Uvek se pobrinite da dosledno koristimo naznake tipova u celom projektu kako bismo izbegli sukobe. Takođe, moramo se pobrinuti da verzija Pajtona u našem kodu radi kako bismo izbegli probleme sa kompatibilnošću.

Izbegavajte preterano komplikovanje naših tipova saveta. Neka budu jasni i jednostavni, vodeći računa da služe čitljivijem, a ne složenijem kodu.

## Zaključak

Integracijom modula `typing-extensions`, dobijamo pristup moćnim funkcijama za nagoveštavanje tipova koje poboljšavaju bezbednost i čitljivost koda. Ove funkcije su posebno vredne kada se radi na projektima koji zahtevaju unazadnu kompatibilnost između različitih verzija Pajtona.
