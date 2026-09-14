
# Pregled sistema tipova u Go

[Gorutine, Odloženi pozivi funkcija i Panic/Recover][0110] | [Sadržaj][00] | [Pokazivači][0202]

Ovaj članak će predstaviti sve vrste tipova u jeziku Go i koncepte vezane za sistem tipova u jeziku Go. Bez poznavanja ovih osnovnih koncepata, teško je imati temeljno razumevanje jezika Go.

## Osnovni tipovi

Ugrađeni osnovni tipovi u Go-u su takođe predstavljeni kao ugrađeni osnovni tipovi i osnovni vrednosni literali. Radi potpunosti ovog članka, ovi ugrađeni osnovni tipovi su ponovo navedeni ovde.

- Ugrađeni tip stringa: **string**.
- Ugrađeni bulov tip: **bool**.
- Ugrađeni numerički tipovi:
  - **int8**,
  - **uint8( byte )**,
  - **int16**,
  - **uint16**,
  - **int32( rune )**,
  - **uint32**,
  - **int64**,
  - **uint64**,
  - **int**,
  - **uint**,
  - **uintptr**.
  - **float32**,
  - **float64**.
  - **complex64**,
  - **complex128**.

Imajte na umu da je **byte** ugrađeni alias od **uint8**, i da je **rune** ugrađeni alias od **int32**. Takođe možemo deklarisati prilagođene aliase tipa (videti dole).

Osim tipova stringova, serija članaka o Go 101 neće pokušati da objasni više o drugim osnovnim tipovima.

17 ugrađenih osnovnih tipova su unapred deklarisani tipovi.

## Složeni tipovi

Go podržava sledeće kompozitne tipove:

- tipovi pokazivača - slični C pokazivačima.
- tipovi struktura - slično C strukturama.
- tipovi funkcija - funkcije su tipovi prve klase u Go-u.
- tipovi kontejnera :
  - tipovi nizova - tipovi kontejnera fiksne dužine.
  - tip isečaka - tipovi kontejnera dinamičke dužine i dinamičkog kapaciteta.
  - tipovi mapa - mape su asocijativni nizovi (ili rečnici). Standardni Go kompajler implementira
    mape kao heš tabele.
  - tipovi kanala - kanali se koriste za sinhronizaciju podataka između gorutina (zelene niti u
    Go-u).
  - tipovi interfejsa - interfejsi igraju ključnu ulogu u refleksiji i polimorfizmu.

Neimenovani kompozitni tipovi mogu biti označeni njihovim odgovarajućim literalima tipa. U nastavku su navedeni neki primeri literalnog predstavljanja svih vrsta neimenovanih kompozitnih tipova (imenovani i neimenovani tipovi biće objašnjeni u nastavku).

```go
// Assume T is an arbitrary type and Tkey is
// a type supporting comparison (== and !=).

*T         // a pointer type
[5]T       // an array type
[]T        // a slice type
map[Tkey]T // a map type

// a struct type
struct {
    name string
    age  int
}

// a function type
func(int) (bool, string)

// an interface type
interface {
    Method0(string) int
    Method1() (int, bool)
}

// some channel types
chan T
chan<- T
<-chan T
```

Uporedivi i neuporedivi tipovi biće objašnjeni u nastavku.

## Vrste tipova

Svaki od gore pomenutih osnovnih i kompozitnih tipova odgovara jednoj vrsti tipova. Pored ovih vrsta, nesigurni tipovi pokazivača predstavljeni u **unsafe** standardnom paketu takođe pripadaju jednoj vrsti tipova u Gou. Dakle, do sada (Go 1.25), Go ima 26 vrsta tipova.

### Definicije tipova

( Definicija tipa, ili deklaracija definicije tipa, prvobitno nazvana deklaracija tipa, bila je jedina deklaracija tipa mnogo pre Go 1.9. Od Go 1.9, definicija tipa je postala jedan od dva oblika deklaracija tipa. Novi oblik se naziva deklaracija aliasa tipa, što će biti predstavljeno u odeljku ispod.)

U programu Go, možemo definisati nove tipove koristeći sledeći oblik. U sintaksi, **type** je ključna reč.

```go
// Define a solo new type.
type NewTypeName SourceType

// Define multiple new types together.
type (
    NewTypeName1 SourceType1
    NewTypeName2 SourceType2
)
```

Nova imena tipova moraju biti identifikatori. Ali imajte na umu da imena tipova deklarisana na nivou paketa ne mogu biti init.

Druga deklaracija tipa u gornjem primeru uključuje dve specifikacije tipa. Ako deklaracija tipa sadrži više od jedne specifikacije tipa, specifikacije tipa moraju biti obuhvaćene parom ().

Napomena,

- Novi definisani tip i njegov odgovarajući izvorni tip u definicijama tipova su dva različita tipa.
- Dva tipa definisana u dve definicije tipa su uvek dva različita tipa.
- Novo definisani tip i izvorni tip će deliti isti osnovni tip (videti dole definiciju osnovnih tipova), a njihove vrednosti se mogu konvertovati jedna u drugu.
- Tipovi se mogu definisati unutar tela funkcija.

Neki primeri definicija tipa:

```go
// The following new defined and source types are all
// basic types. The source types are all predeclared.
type (
    MyInt int
    Age   int
    Text  string
)

// The following new defined and source types are all
// composite types. The source types are all unnamed.
type IntPtr *int
type Book struct{author, title string; pages int}
type Convert func(in0 int, in1 bool)(out0 int, out1 string)
type StringArray [5]string
type StringSlice []string

func f() {
    // The names of the three defined types
    // can be only used within the function.
    type PersonAge map[string]int
    type MessageQueue chan string
    type Reader interface{Read([]byte) int}
}
```

Imajte na umu da je, od Go 1.9 do Go 1.17, Go specifikacija uvek smatrala da su unapred deklarisani ugrađeni tipovi definisani tipovi. Ali od Go 1.18, Go specifikacija pojašnjava da unapred deklarisani ugrađeni tipovi nisu definisani tipovi.

### Prilagođeni generički tipovi i instancirani tipovi

Od verzije Go 1.18, Go podržava prilagođene generičke tipove (i funkcije). Generički tip mora biti instanciran da bi se koristio kao vrednosni tip.

**Generički tip** je **definisani tip**, a njegovi instancirani tipovi su **imenovani tipovi** (imenovani tipovi su objašnjeni u sledećem odeljku).

Druga dva važna koncepta u prilagođenim generičkim tipovima su **ograničenja** i **parametri tipa**.

Ova knjiga ne govori detaljno o ​​prilagođenim generičkim tipovima. Molimo vas da pročitate knjigu Go Generics 101 o tome kako deklarisati i koristiti generičke tipove i funkcije.

### Imenovani tipovi u odnosu na neimenovane tipove

Pre Go 1.9, terminologija imenovanog tipa je precizno definisana u Go specifikaciji. Imenovani tip je definisan kao tip koji je predstavljen identifikatorom. Zajedno sa funkcijom prilagođenog alijasa tipa uvedenom u Go 1.9 (videti sledeći odeljak), terminologija imenovanog tipa je uvek uklonjena iz Go specifikacije i zamenjena je terminologijom definisanog tipa. Od Go 1.18, zajedno sa uvođenjem prilagođenih generičkih tipova, terminologija imenovanog tipa je vraćena u Go specifikaciju.

Imenovani tip može biti

- unapred deklarisani tip ( ne uključujući alias tipa );
- definisani (neprilagođeni-generički) tip;
- instancirani tip (generičkog tipa);
- parametar tipa tip (koristi se u prilagođenim generičkim tipovima).

Drugi vrednosni tipovi nazivaju se **neimenovani tipovi**. **Neimenovani tip** mora biti kompozitni tip (ne obrnuto).

### Deklaracije alijasa tipa

Od verzije Go 1.9, možemo deklarisati prilagođene alijase tipova koristeći sledeću sintaksu. Sintaksa deklaracije alijasa je veoma slična definiciji tipa, ali imajte na umu da postoji **=** specifikacija alijasa tipa u svakoj verziji.

```go
type (
    Name = string
    Age  = int
)

type table = map[string]int
type Table = map[Name]Age
```

Alijasi tipova moraju biti identifikatori. Kao i definicije tipova, alijasi tipova se takođe mogu deklarisati unutar tela funkcija.

Ime tipa (ili literala) i njegovi alijasi označavaju identičan tip. Prema gornjim deklaracijama, Name je alijas od string, tako da oba označavaju isti tip. Isto važi i za ostala tri para notacija tipa (ili imena ili literale):

```sh
Age i int
table i map[string]int
Table i map[Name]Age
```

U stvari, literali **map[string]int** i **map[Name]Age** takođe označavaju isti tip. Dakle, slično, alijasi *table* i *Table* takođe označavaju isti tip.

Imajte na umu da, iako alias tipa uvek ima ime, on može označavati neimenovani tip. Na primer, aliasi *table* i *Table* označavaju isti neimenovani tip **map[string]int**.

### Neka od pravila za osnovne tipove

U programu Go, svaki tip ima osnovni tip. Pravila:

- Za ugrađene tipove, odgovarajući osnovni tipovi su oni sami.
- Za **Pointer** tip definisan u **unsafe** standardnom paketu koda, njegov osnovni tip je on sam (barem tako možemo misliti. U stvari, osnovni tip tipa **unsafe.Pointer** nije dobro dokumentovan. Takođe možemo misliti da je osnovni tip **\*T**, gde **T** predstavlja proizvoljni tip). **unsafe.Pointer** se takođe posmatra kao ugrađeni tip.
- Osnovni tip neimenovanog tipa, koji mora biti kompozitni tip, je on sam.
- U deklaraciji tipa, novodeklarisani tip i izvorni tip imaju isti osnovni tip.

Primeri:

```go
// The underlying types of the following ones are both int.
type (
    MyInt int
    Age   MyInt
)

// The following new types have different underlying types.
type (
    IntSlice   []int   // underlying type is []int
    MyIntSlice []MyInt // underlying type is []MyInt
    AgeSlice   []Age   // underlying type is []Age
)

// The underlying types of []Age, Ages, and AgeSlice
// are all the unnamed type []Age.
type Ages AgeSlice
```

Kako se može pratiti osnovni tip na osnovu korisnički deklarisanog tipa? Pravilo je da kada se ispuni ugrađeni osnovni tip ili neimenovani tip, praćenje treba zaustaviti. Uzmimo gore prikazane deklaracije tipova kao primere, hajde da pratimo njihove osnovne tipove.

```go
MyInt → int
Age → MyInt → int
IntSlice → []int
MyIntSlice → []MyInt → []int
AgeSlice → []Age → []MyInt → []int
Ages → AgeSlice → []Age → []MyInt → []int
```

U programu Go,

- tipovi čiji su osnovni tipovi **bool** nazivaju se bulovi tipovi;
- tipovi čiji su osnovni tipovi bilo koji od ugrađenih **int** tipova nazivaju se celobrojni tipovi;
- tipovi čiji su osnovni tipovi ili **float32** ili **float64** nazivaju se tipovi sa pokretnim zarezom;
- tipovi čiji su osnovni tipovi ili **complex64** ili **complex128** nazivaju se složeni tipovi;
- celobrojni, brojevi sa pokretnim zarezom i kompleksni tipovi se takođe nazivaju numerički tipovi;
- tipovi čiji su osnovni tipovi **string** nazivaju se string tipovi.

Koncept osnovnog tipa igra važnu ulogu u konverzijama vrednosti, dodelama i poređenjima u Go jeziku.

### Vrednosti

Instanca tipa naziva se "vrednost" tog tipa. Vrednosti istog tipa dele neka zajednička svojstva. Tip može imati mnogo vrednosti. Jedna od njih je **nulta vrednost** tog tipa.

Svaki tip ima **nultu vrednost**, koja se može posmatrati kao **podrazumevana vrednost tipa**. Unapred deklarisani **nil** identifikator može se koristiti za predstavljanje nultih vrednosti:

- isečaka,
- mapa,
- funkcija,
- kanala,
- pokazivača (uključujući pokazivače koji nisu sigurni po tipu) i
- interfejsa.

Za više informacija o nil, pročitajte nil u Go-u.

Postoji nekoliko vrsta oblika predstavljanja vrednosti u kodu, uključujući:

- literale,
- imenovane konstante,
- promenljive i
- izraze,

mada se prva tri mogu posmatrati kao posebni slučajevi drugog.

Vrednost može biti tipizirana ili netipizirana.

Sve vrste osnovnih vrednosnih literala su predstavljene u članku "osnovni tipovi i osnovni vrednosni literali". Postoje još dve vrste literala u Go-u:

- kompozitni literali i
- funkcionalni literali.

Literali funkcije, kao što i samo ime kaže, koriste se za predstavljanje vrednosti funkcije. Deklaracija funkcije se sastoji od literala funkcije i identifikatora (ime funkcije).

Kompozitni literali se koriste za predstavljanje vrednosti struktura i kontejnera (nizova, isečaka i mapa). Za više detalja pročitajte strukture u Gou i kontejnere u Gou.

Ne postoje literali za predstavljanje vrednosti pokazivača, kanala i interfejsa.

#### Delovi vrednosti

Tokom izvršavanja, mnoge vrednosti se čuvaju negde u memoriji. U programu Go, svaka takva vrednost ima direktan deo. Međutim, neke od njih imaju jedan ili više indirektnih delova. Svaki deo vrednosti zauzima kontinuirani segment memorije. Indirektni osnovni delovi vrednosti se referenciraju njenim direktnim delom putem ( bezbednih ili nebezbednih ) pokazivača.

Deo sa terminološkom vrednošću nije definisan u Go specifikaciji. Koristi se samo u Go 101 da bi se pojednostavila neka objašnjenja i pomoglo Go programerima da bolje razumeju Go tipove i vrednosti.

#### Veličine vrednosti

Kada se vrednost čuva u memoriji, broj bajtova koje zauzima direktni deo vrednosti naziva se veličina vrednosti. Pošto sve vrednosti istog tipa imaju istu veličinu vrednosti, često to jednostavno nazivamo veličinom tipa.

Možemo koristiti Sizeoffunkciju u unsafestandardnom paketu da bismo dobili veličinu bilo koje vrednosti.

Go specifikacija ne određuje zahteve za veličinu vrednosti za nenumeričke tipove. Zahtevi za veličine vrednosti svih vrsta osnovnih numeričkih tipova navedeni su u članku osnovni tipovi i osnovni vrednosni literali.

### Osnovni tip pokazivača

Za tip pokazivača, pretpostavimo da se njegov osnovni tip može označiti kao **\*T** u literalu, tada se **T** naziva osnovni tip tipa pokazivača.

Više informacija o tipovima i vrednostima pokazivača možete pronaći u članku pokazivači u Gou.

### Polja strukturnog tipa

Strukturni tip se sastoji od kolekcije deklaracija promenljivih članova. Svaka od deklaracija promenljivih članova naziva se „polje“ tipa strukture. Na primer, sledeći tip strukture *Book* ima tri polja *author*, *title* i *pages*.

```go
type Book struct {
    author string
    title  string
    pages  int
}
```

Više informacija o tipovima i vrednostima struktura možete pronaći u članku "strukture u Gou".

### Potpis tipova funkcija

Signatura tipa funkcije sastoji se od liste definicija ulaznih parametara i liste definicija izlaznih rezultata funkcije.

Ime i telo funkcije nisu delovi potpisa funkcije. Tipovi parametara i rezultata su važni za potpis funkcije, ali imena parametara i rezultata nisu važna.

Za više detalja o tipovima funkcija i vrednostima funkcija, pročitajte funkcije u programu Go.

### Metode i skup metoda jednog tipa

U jeziku Go, neki tipovi mogu imati metode. Metode se takođe mogu nazvati funkcijama članicama. Skup metoda tipa sastoji se od svih metoda tog tipa.

### Dinamički tip i dinamička vrednost vrednosti interfejsa

Vrednosti interfejsa su vrednosti čiji su tipovi tipovi interfejsa.

Svaka vrednost interfejsa može da uokviri vrednost koja nije interfejs. Vrednost uokvirena u vrednost interfejsa naziva se **dinamička vrednost vrednosti interfejsa**. Tip dinamičke vrednosti naziva se **dinamički tip vrednosti interfejsa**. Vrednost interfejsa koja ne uokviruje ništa je nulta vrednost interfejsa.

**Nulta vrednost interfejsa** nema ni dinamičku vrednost ni dinamički tip.

Tip interfejsa može da odredi nula ili više metoda, koje formiraju skup metoda tipa interfejsa.

Ako je skup metoda tipa, koji je ili tip interfejsa ili tip koji nije interfejs, nadskup skupa metoda tipa interfejsa, kažemo da tip implementira tip interfejsa.

Za više informacija o tipovima i vrednostima interfejsa, pročitajte o "interfejsima u Go-u".

### Konkretna vrednost i konkretan tip vrednosti

Za (tipizovanu) vrednost koja nije interfejs, njena konkretna vrednost je ona sama, a njen konkretni tip je tip vrednosti.

Nulta vrednost interfejsa nema ni konkretan tip ni konkretnu vrednost.

Za vrednost interfejsa koja nije nula, njegova konkretna vrednost je njegova dinamička vrednost, a njegov konkretan tip je njegov dinamički tip.

### Tipovi kontejnera

**Nizovi**, **isečci** i **mape** mogu se posmatrati kao formalni tipovi kontejnera.

Ponekad se tipovi **stringova** i **kanala** mogu neformalno posmatrati i kao tipovi kontejnera.

Svaka vrednost formalnog ili neformalnog tipa kontejnera ima **dužinu**.

Više informacija o formalnim tipovima i vrednostima kontejnera možete pronaći u članku "kontejneri u Go jeziku".

### Tip ključa mape

Ako se osnovni tip tipa mape može označiti kao **map[Tkey]T**, onda se **Tkey** naziva tipom ključa tipa mape. **Tkey** mora biti uporediv tip (videti dole).

### Tip elementa tipa kontejnera

Tipovi elemenata sačuvanih u vrednostima kontejnerskog tipa moraju biti **identični**. Identičan tip elemenata naziva se **ip elementa kontejnerskog tipa**.

- Za tip niza, ako je njegov osnovni tip **[N]T**, onda je tip njegovog elementa **T**.
- Za tip isečka, ako je njegov osnovni tip **[]T**, onda je njegov tip elementa **T**.
- Za tip mape, ako je njegov osnovni tip **map[Tkey]T**, onda je tip njegovog elementa **T**.
- Za tip kanala, ako je njegov osnovni tip **chan T**, **chan<- T** ili **<-chan T**, onda je njegov
  tip elementa **T**
- Tip elementa bilo kog tipa stringa je uvek **byte** (tj. **uint8**).

### Tipovi kanala

Vrednosti kanala mogu se posmatrati kao sinhronizovani redovi redova „prvi uđe, prvi izađe“ (FIFO). Tipovi i vrednosti kanala imaju smernice.

- Vrednost kanala koja se može i slati i primati naziva se **dvosmerni kanal**. NJegov tip se naziva **tip dvosmernog kanala**. Osnovni tipovi dvosmernih tipova kanala označeni su literalom **chan T**.
- Vrednost kanala koja se može samo slati naziva se **kanal samo za slanje**. NJegov tip se naziva **tip kanala samo za slanje**. Tipovi kanala samo za slanje označeni su literalom **chan<- T**.
- Vrednost kanala koja se može samo primiti naziva se **kanal samo za prijem**. NJegov tip se naziva **tip kanala samo za prijem**. Tipovi kanala samo za prijem označeni su literalom **<-chan T**.

Više informacija o tipovima i vrednostima kanala možete pronaći u članku "kanali u Gou".

### Tipovi koji podržavaju ili ne podržavaju poređenja

Trenutno (Go 1.25), Go ne podržava poređenja (sa operatorima **==** i **!=**) za vrednosti sledećih tipova:

- tipovi isečaka
- tipovi mapa
- tipovi funkcija*
- bilo koji tip strukture sa poljem čiji je tip neuporediv i
- bilo koji tip niza čiji je tip elementa neuporediv.

Gore navedeni tipovi se nazivaju neuporedivi tipovi. Svi ostali tipovi se nazivaju uporedivi tipovi. Kompilatori zabranjuju poređenje dve vrednosti neuporedivih tipova.

Imajte na umu da se neuporedivi tipovi u mnogim člancima nazivaju i neuporedivi tipovi.

Ključni tip bilo kog tipa mape mora biti uporedivi tip.

Više o detaljnim pravilima poređenja možemo saznati iz članka "konverzije vrednosti, dodele i poređenja u Gou".

## Objektno orijentisano programiranje u Go-u

Go nije objektno orijentisan programski jezik sa svim funkcionalnostima, ali Go zaista podržava neke elemente objektno orijentisanog programiranja. Molimo vas da pročitate sledeće članke za detalje:

- metode u programu Go.
- implementacije u Gou.
- ugrađivanje tipova u Go-u.

## Generici u Go-u

Pre Go 1.18, generičke funkcionalnosti u Go-u su bile ograničene na ugrađene tipove i funkcije. Od Go 1.18, prilagođeni generici su već podržani. Molimo vas da pročitate članak o genericima u Go -u za ugrađene generike i knjigu Go Generics 101 za prilagođene generike.

[Gorutine, Odloženi pozivi funkcija i Panic/Recover][0110] | [Sadržaj][00] | [Pokazivači][0202]

[0110]: 0110_Goroutine_Odloženi_Funkcijski_pozivi_i_Panic-Recover.md
[00]: 00_Sadrzaj.md
[0202]: 0202_Pokazivači.md
