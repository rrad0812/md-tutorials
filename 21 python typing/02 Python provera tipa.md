
# Python provera tipa

> [!Warning]  
> Ovaj tekst je relativno star, iz 2016. god.

## Pozadina

### Zašto bi me trebalo brinuti?

Možda ste čuli da su naznake tipova dodate u Pajton 3.5 i pitali se „zašto bi mene to zanimalo?“. Pa, odgovor je uglavnom isti kao i za dokumentovanje vašeg koda: provera tipova vam štedi vreme sprečavanjem grešaka i uklanjanjem nagađanja. Do sada, najbolje što smo imali u pogledu specifikacija tipova bilo je nekoliko konvencija koje su u najboljem slučaju bile dvosmislene, a sada imamo standard oko kojeg možemo graditi alate.

Uzmimo za primer konvenciju o dokumentacionim stringovima u numpy-ju . LJubazno nam daju neke osnovne primere kucanja, kao što je ovaj:

```py
Parameters
----------
filename : str
copy : bool
dtype : data-type
iterable : iterable object
shape : int or tuple of int
files : list of str
```

Ali nema smernica o tome kako ih kombinovati u složenije recepte. Kao rezultat toga, često viđam dvosmislene specifikacije tipa koje izgledaju ovako:

```py
list of str or int
```

Da li je int na listi ili nije ?

Štaviše, ne postoje primeri kako se rukuje složenim torkama, tipovima ključeva i vrednosti rečnika, pozivljivim potpisima ili kako se specificiraju tipovi koji su klase, a ne instance.

Rezultat je da je, uz ovoliku dvosmislenost, programska provera tipova prilično nepouzdana. Da bi se situacija poboljšala, neka IDE-a poput PyCharm-a su predložila sopstvenu konvenciju, što je korak napred, ali da li zaista želite da vaši moduli budu specifični za IDE?

### PEP 484

Sa PEP 484 , Gvido i ekipa su sve ovo rešili za vas i kreirali standard za anotacije tipova koji je sada deo Python-a 3.5. Ako koristite Python 3.5 ili noviji, možete pisati definicije funkcija ovako:

```py
def  func ( inputs :  Union [ str ,  List [ str ]], enabled :  Dict [ str ,  bool ])  
    ->  Iterable [ str ]: 
    ...
```

Odlično. Ali šta je sa onima od nas koji su još uvek zaglavljeni na 2.x?

Ovde treba da stanemo i razjasnimo razliku između anotacija tipova i provere tipova. Dodavanjem pep484, Python 3.5 je dobio dve stvari:

- standard za opisivanje tipova (npr .)Union[str, List[str]]
- sintaksna podrška za anotiranje argumenata funkcija i povratnih vrednosti opisima tipova

Primetno je da ovde nedostaje stvarna provera tipova , tj. inspekcija koda kako bi se osiguralo da argumenti i dodele odgovaraju svojim deklarisanim tipovima. Programeri Pajtona su tu ulogu prepustili alatima trećih strana.

Nazad na Pajton 2.x: Standard za opisivanje tipova nedvosmislenim terminima je velika stvar čak i bez sintaksičke podrške dodate u verziji 3.5, a dobra vest je da je sredstvo za kreiranje ovih definicija tipova, modul za tipizaciju , dostupno za Pajton 2.7. Međutim, nedostatak sintaksičke podrške znači da proveravači tipova moraju da obezbede sopstvene konvencije za povezivanje opisa tipova sa argumentima i povratnim vrednostima u Pajtonu 2.7 kodu.

Dakle, bez daljeg odlaganja, hajde da pređemo na alate.

## Alati

Postoji nekoliko alata za obavljanje provere tipova kompatibilnih sa pep484. Svaki ima svoje prednosti i mane. Oba vrše statičku analizu koda, što znači da zapravo ne uvoze i ne pokreću vaše module: umesto toga, oni parsiraju i analiziraju vaš kod. Ovo je bezbednije, ali znači da se dinamički generisani objekti ne mogu pregledati.

### mypy

**mypy** je alat komandne linije, sličan linteru, koji skenira vaš kod i ispisuje greške. Programeri **mypy**-ja predvode proveru tipova koda. PEP 484 je prvobitno inspirisan **mypy**-jem, a sam Gvido je trenutno uključen u njegov razvoj.

Ispod su vaše opcije koje **mypy** podržava za dodavanje anotacija tipa funkcijama u Python-u 2.7 (više informacija ovde ):

#### Jednolinijski

```py
def  doit ( inputs ,  enabled ):
    # (Union[str, List[str]], Dict[str, bool]) -> Iterable[str]
    "Uradi nešto sa tim ulazima"
    ...
```

Nezgodno je što može postati veoma dugačko i teško vizuelno povezati argument sa tipom.

#### Višelinijski

```py
def doit ( inputs ,  # type: Union[str, List[str]]
           enabled   # type: Dict[str, bool]
    ):
    # type: (...) -> Iterable[str]
    "Uradi nešto sa tim ulazima"
    ...
```

Malo opširnije, ali čitljivije.

Jedan aspekt **mypy**-ja koji vam može otežati integraciju u vaš ciklus izgradnje/objavljivanja jeste da je za njegovo pokretanje potreban Python 3.5+, čak i ako analizirate Python 2.7 kod.

### PyCharm

**PyCharm** je moj novi omiljeni IDE. Njegova analiza koda ide duboko i svakodnevno mi spasava život. Sada kada sam ga obavestio o tipovima argumenata i povratnih vrednosti, to je u osnovi SkyNet (ili će biti, uz samo još nekoliko nadogradnji...).

U trenutku pisanja ovog teksta, **PyCharm** podržava i jednoredne i višeredne stilove navedene gore, kao i PEP 484 - kompatibilne tipove koji se isporučuju putem dokumentacionih stringova. Za ove druge morate biti prilično pažljivi u vezi sa formatiranjem: Ako ste previše labavi, parser će odustati.**PyCharm**može da parsira četiri stila dokumentacionih stringova.

#### Stil restrukturiranog teksta

```py
def doit(inputs, enabled):
    """Do something with those inputs

    :param inputs: input names
    :type inputs:  Union[str, List[str]]
    :param enabled: mapping of input names to enabled status
    :type enabled: Dict[str, bool]
    :rtype: Iterable[str]
    """
    ...
```

Ružno, ali obavlja posao. Dokumentacioni stringovi u stilu Epydoc-a su isti, ali sa `@` umesto vodećeg `:`.

#### Gugl stil

```py
def doit(inputs, enabled):
    """Do something with those inputs

    Args:
        inputs (Union[str, List[str]]):  input names
        enabled (Dict[str, bool]):  mapping of input names to enabled status

    Returns:
        Iterable[str]: enabled inputs
    """
    ...
```

Kompaktno, ali čitljivo.

#### numpy stil

```py
def doit(inputs, enabled):
    """Do something with those inputs

    Args:
        inputs (Union[str, List[str]]):  input names
        enabled (Dict[str, bool]):  mapping of input names to enabled status

    Returns:
        Iterable[str]: enabled inputs
    """
    ...
```

Moj lični favorit.

Glavna mana PyCharm-a za proveru tipova u stilu PEP 484 je to što i dalje pokušava da sustigne **mypy**. Neke prilično osnovne funkcije i dalje nedostaju:

- Type
- Type aliases
- TypeVar
- Generics lekovi

Pored toga, voleo bih da vidim više vizuelnih povratnih informacija.

Ako ništa drugo ne proiziđe iz pisanja ovoga, vredeće ako nekoliko ljudi klikne na gornje linkove i podigne malo buke o tim pitanjima.

### pytype

Uključujem pytype sa Gugla radi potpunosti. To je alat komandne linije poput **mypy**. Glavna stvar koju ima je to što se može pokrenuti pomoću Python-a 2.7, za razliku od **mypy**-ja koji se može pokrenuti samo pomoću Python-a 3.5+ (oba alata mogu analizirati Python 3.x kod).

### Poređenje

PyCharm vam daje gotovo trenutne povratne informacije o nekompatibilnostima tipova u kontekstu vašeg koda, što stvara zavisnost koja podstiče sve više nagoveštavanja tipova. **mypy**, s druge strane, je pomalo mučan. Morate ga pokrenuti ručno, a zatim pretraživati njegov zagonetni izlaz i tražiti odgovarajuće brojeve linija. Zapravo je namenjen da bude integrisan u vaš proces izgradnje/objavljivanja.

Takođe mi se jako sviđa što mi**PyCharm**dozvoljava da nastavim da određujem tipove unutar docstring-ova. Za postojeći kod, osnovni tipovi već rade unutar PyCharm-a, tako da samo treba da nadogradim egzotičnije recepte na novi standard. Takođe, više volim da se informacije o tipu nalaze pored opisa tipa.

Glavna mana PyCharm-a je što nije tako temeljan kao **mypy** i još uvek postoji niz izuzetno važnih funkcija koje trenutno nisu implementirane, mada sam uveren da će se poboljšati u kratkom roku. **mypy** je takođe sposoban da statički tipizira pojedinačne promenljive, ne samo argumente funkcija i povratne vrednosti.

Ništa vas ne sprečava da koristite oba zajedno –**PyCharm**kao neposrednu prvu liniju odbrane i **mypy** kao temeljitiju proveru koju sprovodi kontinuirana integracija.

## Klase tipova

Prvo što treba razumeti je da su anotacije tipova zapravo Pajton klase.

Morate ih uvesti iz `typing` da biste ih koristili. Ovo je, priznajem, pomalo smetnja, ali ima više smisla kada uzmete u obzir da integracija sintakse u Pajtonu 3.5 znači da pridajete objekte definicijama funkcija baš kao što to radite kada dajete podrazumevanu vrednost argumentu. U stvari, možete koristiti `typing.get_type_hints()` funkciju da biste pregledali objekte nagoveštaja tipa na funkciji tokom izvršavanja, baš kao što biste pregledali podrazumevane vrednosti argumenta sa `inspect.getargspec()`.

Klase tipova se dele u nekoliko kategorija, koje ćemo razmotriti u nastavku.

### Osnovni tipovi

Osnovni skup tipova je prilično dobro obrađen u **mypy** dokumentaciji, ali ću dati kratak pregled u nastavku.

#### Any

Predstavlja bilo koji tip.

Ako funkcija vraća vrednost `None`, trebalo bi da ovo eksplicitno navedete, jer ako se izostavi, podrazumevano se koristi `Any`, što je popustljivije.

> Za razliku od `Any`, `object` je običan statički tip i za `object` vrednosti se prihvataju samo operacije važeće za sve tipove.

`Any` je stoga popustljiviji.

#### Callable

Koristi se za označavanje funkcije ili vezane metode sa određenim potpisom.

Evo jednostavne funkcije i kako je kodirati kao anotaciju tipa:

```py
def repeat(s, count):
    # type: (str, int) -> str
    return s * count
```

```py
Callable[[str, int], str]
```

Ili, ako vas zanima samo povratni rezultat:

```py
Callable[..., str]
```

#### Union

Koristi se kada postoji više od jednog validnog tipa.

```py
Union[str, List[str]]
```

#### Optional

Skraćenica za tip koji je dozvoljeno da bude `None`.

Ova dva izraza su ekvivalentna:

```py
Optional[int]
Union[int, None]
```

U **mypy** `None` je podrazumevano validna vrednost za svaki tip, ali zbog popularnih zahteva to će se promeniti, mada nisam siguran u kom vremenskom okviru. Već je moguće promeniti ponašanje provere tipa pomoću zastavice. Stoga, ako sada počinjete, najbolje je da steknete naviku dodavanja `Optional` modifikatora tipa da biste označili tip koji uključuje `None`.

#### Type

Koristi se da označi da tip treba da bude neinstancirana klasa.

```py
Type[MyClass]
Type[Union[MyClass, OtherClass]]
```

#### Type aliases

Ovo je tehnika, a ne tip. Sećate se kako smo razgovarali o tome da su definicije tipova regularni Pajton objekti? Pa, to znači da ih možete dodeliti promenljivim na nivou modula i koristiti te promenljive u svojim anotacijama. Ovo je korisno ako imate mnogo funkcija koje prihvataju isti složeni recept.

(pokvareno u PyCharm-u)

```py
from typing import Dict, List, Union
PropertiesType = Dict[str, List[str]]
PropertiesListType = List[Dict[str, PropertiesType]]

def process_properties(props):
    # type: (PropertiesListType) -> None
    ...
```

### Generic

Ovo je osnovna klasa za sve klase kolekcija obrađene u nastavku. To im daje sintaksu zagrada za specijalizaciju tipa (npr. Container[int]). Moje prosvetljenje u vezi sa nagoveštavanjem tipova došlo je kada sam shvatio da podklase od Generic nisu samo za definisanje nagoveštaja tipova. Korišćenjem `Generic` kao alternativne osnovne klase `object` pri  kreiranju sopstvenih klasa kolekcija, vaše klase se mogu koristiti i kao kolekcija (instanciranjem kao što biste to obično uradili) i kao anotacija tipa (korišćenjem `[]` na samoj klasi). Pogledajte primer `Stack` u **mypy** dokumentaciji da biste videli primer.

(pokvareno u PyCharm-u)

#### TypeVar

`TypeVar` vam omogućava da kreirate odnose i ograničenja između argumenta i drugih argumenata ili vraćenih vrednosti.

Na primer, recimo da imate funkciju koja prihvata vrednost bilo kog tipa, a vraća vrednost istog tipa.

Ako koristimo, `Any` onda nećemo uspeti da uspostavimo tu vezu:

```py
def  passthrough ( input ):
    # type: (Any) -> Any
    return  input
```

I ulaz i rezultat mogu biti bilo kog tipa, ali ništa ne ukazuje na to da će uvek biti **istog** tipa.

Da bismo proveri tipova dali više konteksta, kreiramo `TypeVar` i delimo ga između anotacija.

```py
T = TypeVar('T')

def passthrough(input):
    # type: (T) -> T
    return input
```

Ovo se naziva generičkom funkcijom. Naravno, postaje zanimljivije od ovoga. Vrednost `TypeVar` može biti ograničena na isti način kao i bilo koja druga vrednost:

```py
TypeVar('T', bound=Callable[[int, str], bool])
```

`TypeVars` se često koriste sa `Generic` kolekcijama (o čemu će biti više reči u nastavku) da bi se formirala veza između kolekcije i drugog argumenta ili povratnih vrednosti. Evo jednog dobrog primera iz dokumentacije o generičkim tipovima:

```py
from typing import TypeVar, Sequence

T = TypeVar('T')

def first(seq: Sequence[T]) -> T:
    return seq[0]
```

### Konkretni tipovi kolekcija

Konkretni tipovi kolekcija su namenjeni da se koriste kao zamena za određene ključne kolekcije u svrhu nagoveštavanja tipova. Njihova instancija nije moguća: za to je potrebno da nastavite da koristite njihove „stvarne“ ekvivalente.

U idealnom svetu, sve kolekcije u standardnoj biblioteci Pajtona bi bile podklase od Generic, što bi omogućilo da ista klasa služi i kao implementacija i kao anotacija tipa. Možda će se ovo jednog dana rešiti, ako nagoveštavanje tipova postane popularno, a u međuvremenu imamo ovu podelu.

Vrste konkretnih kolekcija:

- Tuple
- Dict
- DefaultDict
- List
- Set

Prilično su jednostavni za korišćenje. Sve što vam je potrebno možete saznati iz nekoliko jednostavnih primera:

| Primer | Objašnjenje |
| ------ | ----------- |
| list | lista bilo koje vrste, moguće heterogena |
| List[Any] | isto kao gore |
| List[int] | lista koja sadrži samo cele brojeve |
| dict | rečnik sa bilo kojim ključem ili vrednošću |
| Dict[Any, Any] | isto kao gore |
| Dict[str, int] | rečnik čiji su ključevi stringovi, a vrednosti celi brojevi |
| tuple | torka sa bilo kojom količinom bilo kog tipa |
| Tuple[Any, ...] | isto kao gore |
| Tuple[int] | torka sa jednim celim brojem. npr.:(1,) |
| Tuple[int, ...] | torka sa bilo kojim brojemint |
| Tuple[int, str] | torka čiji je prvi element ceo broj, a drugi je string |

### Imenovana torka

`typing.NamedTuple` je alternativa `collections.namedtuple` koja podržava proveru tipa.

Ispod haube, ona obavija `collections.namedtuple` i označava rezultujuću klasu atributom za praćenje tipova polja, ali u stvarnosti to nije ni potrebno jer statička analiza koda neće imati pristup tome.

Evo primera adaptiranog iz dokumentacije. Klasa Point definisana u sledećem kodu je neprozirna za proveru tipa:

```py
from collections import namedtuple

Point = namedtuple('Point', ['x', 'y'])
p = Point(x=1, y='x')
p.y / 2.0  # fails at runtime
```

Zamenom sa `typing.NamedTuple`, Point klasa se sada može koristiti kao anotacija tipa u funkcijama i instanciranje tipa može biti pravilno validirano.

```py
from typing import NamedTuple

Point = NamedTuple('Point', [('x', int), ('y', int)])
p = Point(x=1, y='x')  # issue detected by mypy
p.y / 2.0
```

### Prava klasa

Kao što možete očekivati, bilo koja klasa može se koristiti kao identifikator tipa. Ovo ograničava objekte na instance ove klase i njenih podklasa.

Dva alata – **mypy** i **PyCharm** – razlikuju se po načinu na koji pronalaze objekte navedene u anotacijama tipa.

Sa **mypy**, dato ime mora biti važeći identifikator za taj objekat u trenutnom modulu. Na primer, ovo funkcioniše:

```py
import zipfile

def zipit(arg):
    # type: (zipfile.ZipFile) -> None
    return
```

Ali ovo ne:

```py
import zipfile

def zipit(arg):
    # type: (ZipFile) -> None
    return
```

To je zato što ZipFile ne identifikuje nijedan objekat u opsegu funkcije zipit (iskreno, nisam baš sasvim siguran kako funkcioniše određivanje opsega u **mypy**-ju, ali sigurno ima opseg modula). Ovo ponašanje ima smisla ako razmišljate o komentarima tipa kao o rezervisanim mestima za dodatke sintakse Python 3.5. Ponovo, pomaže da razmišljate o savetima tipa na isti način kao što biste razmišljali o podrazumevanim argumentima. U tom svetlu, mislim da je intuitivno da ne bi funkcionisalo bez prethodnog uvoza zipfile:

```py
def zipit(arg: zipfile.ZipFile) -> None:
    return
```

Ovo pravilo se zapravo primenjuje na bilo koji objekat definisan spolja za anotaciju tipa zasnovanu na komentarima, kao što su alijasi tipova, ali najčešće dolazi do izražaja kod prilagođenih klasa.

**PyCharm** je malo popustljiviji od **mypy**. Ako ispred objekta dodate naziv modula ili paketa sa tačkom, on će pronaći objekat unutar tog modula, pod pretpostavkom da su putanje pretrage projekta pravilno podešene. Naravno, ako planirate da koristite oba alata zajedno, moraćete da ciljate na najmanji zajednički imenilac, a to je **mypy**.
