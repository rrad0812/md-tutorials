
# Metode

[Kanali][0208] | [Sadržaj][00] | [Interfejsi][0210]

Go podržava neke funkcije objektno orijentisanog programiranja. Metod je jedna od tih funkcija. Ovaj članak će predstaviti koncepte vezane za metode u Gou.

## Deklaracije metoda

U Go-u, možemo (eksplicitno) deklarisati metod za tip Ti *T, gde Tmora da zadovolji 4 uslova:

- T mora biti definisanog tipa ;
- T mora biti definisan u istom paketu kao i deklaracija metode;
- T ne sme biti tipa pokazivača;
- T ne sme biti tip interfejsa. Tipovi interfejsa biće objašnjeni u sledećem članku .

Tip T i *T se nazivaju tipom prijemnika odgovarajućih metoda deklarisanih za njih. Tip Tse naziva osnovnim tipovima prijemnika svih metoda deklarisanih za oba tipa T i \*T.

Imajte na umu da takođe možemo deklarisati metode za alijase tipova T i \*T navedenih gore. Efekat je isti kao i deklarisanje metoda za same tipove T i \*T.

Ako je metod deklarisan za tip, možemo reći da tip ima (ili poseduje) metod.

Iz gore navedenih uslova, dobićemo zaključke da nikada ne možemo (eksplicitno) deklarisati metode za:

- unapred deklarisane tipove, kao što su inti string, jer ne možemo deklarisati metode u builtinstandardnom paketu.
- tipovi interfejsa. Ali tip interfejsa može da poseduje metode. Molimo vas da pročitate sledeći članak za detalje.
- neimenovani tipovi . osim pokazivačkih tipova sa oblikom *Tkoji su gore opisani.

Deklaracija metode je slična deklaraciji funkcije, ali ima dodatni deo za deklaraciju parametra. Dodatni deo parametra može da sadrži jedan i samo jedan parametar tipa prijemnika metode. Taj jedini parametar se naziva parametar prijemnika deklaracije metode. Parametar prijemnika mora biti zatvoren unutar ()i deklarisan između funcključne reči i imena metode.

Evo nekoliko primera deklaracije metoda:

```go
// Age and int are two distinct types. We
// can't declare methods for int and *int,
// but can for Age and *Age.
type Age int
func (age Age) LargerThan(a Age) bool {
    return age > a
}
func (age *Age) Increase() {
    *age++
}

// Receiver of custom defined function type.
type FilterFunc func(in int) bool
func (ff FilterFunc) Filter(in int) bool {
    return ff(in)
}

// Receiver of custom defined map type.
type StringSet map[string]struct{}
func (ss StringSet) Has(key string) bool {
    _, present := ss[key]
    return present
}
func (ss StringSet) Add(key string) {
    ss[key] = struct{}{}
}
func (ss StringSet) Remove(key string) {
    delete(ss, key)
}

// Receiver of custom defined struct type.
type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) SetPages(pages int) {
    b.pages = pages
}
```

Iz gore navedenih primera znamo da osnovni tipovi prijemnika ne mogu biti samo strukturni tipovi, već mogu biti i druge vrste tipova, kao što su osnovni tipovi i kontejnerski tipovi, sve dok osnovni tipovi prijemnika zadovoljavaju 4 gore navedena uslova.

U nekim drugim programskim jezicima, imena parametara prijemnika su uvek implicitni this, što nije preporučeni identifikator za imena parametara prijemnika u jeziku Go.

Prijemnik tipa *Tse naziva pokazivački prijemnik , dok se prijemnici koji nisu pokazivački nazivaju prijemnici vrednosti . Lično, ne preporučujem da se pokazivač terminologije posmatra kao suprotnost vrednosti terminologije , jer su vrednosti pokazivača samo posebne vrednosti. Ali, nisam protiv korišćenja terminologija pokazivački prijemnik i prijemnik vrednosti ovde. Razlog će biti objašnjen u nastavku.

Imena metoda mogu biti prazan identifikator _. Tip može imati više metoda sa praznim identifikatorom kao imenom. Ali takve metode se nikada ne mogu pozivati. Samo izvezene metode se mogu pozivati iz drugih paketa. Pozivi metoda biće predstavljeni u kasnijem odeljku.

## Svaka metoda odgovara implicitnoj funkciji

Za svaku deklaraciju metode, kompajler će deklarisati odgovarajuću implicitnu funkciju za nju. Za poslednje dve metode deklarisane za tip Booki tip *Booku poslednjem primeru u poslednjem odeljku, kompajler implicitno deklariše sledeće dve funkcije:

```go
func Book.Pages(b Book) int {
    // The body is the same as the Pages method.
    return b.pages
}

func (*Book).SetPages(b *Book, pages int) {
    // The body is the same as the SetPages method.
    b.pages = pages
}
```

U svakoj od dve implicitne deklaracije funkcija, parametar prijemnika se uklanja iz odgovarajuće deklaracije metode i ubacuje u normalnu listu parametara kao prvi. Tela funkcija dve implicitno deklarisane funkcije su ista kao i njihova odgovarajuća eksplicitna tela metode.

Implicitna imena funkcija , Book.Pagesi (*Book).SetPages, su oba oblika TypeDenotation.MethodName. Pošto identifikatori u jeziku Go ne mogu da sadrže specijalne karaktere tačke, dva implicitna imena funkcija nisu legalni identifikatori, tako da se ove dve funkcije ne mogu eksplicitno deklarisati. Mogu ih implicitno deklarisati samo kompajleri, ali se mogu pozivati u korisničkom kodu:

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var book Book
    // Call the two implicit declared functions.
    (*Book).SetPages(&book, 123)
    fmt.Println(Book.Pages(book)) // 123
}
```

U stvari, kompajleri ne samo da deklarišu dve implicitne funkcije, već i prepisuju dve odgovarajuće eksplicitno deklarisane metode kako bi omogućili da te dve metode pozivaju dve implicitne funkcije u telima metoda (barem možemo pretpostaviti da se to dešava), baš kao što pokazuje sledeći kod:

```go
func (b Book) Pages() int {
    return Book.Pages(b)
}

func (b *Book) SetPages(pages int) {
    (*Book).SetPages(b, pages)
}
```

## Implicitne metode sa prijemnicima pokazivača

Za svaku metodu deklarisanu za tip prijemnika vrednosti T, odgovarajuća metoda sa istim imenom će biti implicitno deklarisana od strane kompajlera za tip *T. U gornjem primeru, Pagesmetoda je deklarisana za tip Book, tako da je metoda sa istim imenom Pagesimplicitno deklarisana za tip \*Book:

```go
// Note: this is not a legal Go syntax.
// It is shown here just for explanation purpose.
// It indicates that the expression (&aBook).Pages
// is evaluated as aBook.Pages (see below sections).
func (b *Book) Pages = (*b).Pages
```

Zato ne odbacujem upotrebu terminologije prijemnika vrednosti (kao suprotnosti od terminologije prijemnika pokazivača). Na kraju krajeva, kada eksplicitno deklarišemo metodu za tip koji nije pokazivač, u stvari se deklarišu dve metode, eksplicitna je za tip koji nije pokazivač, a implicitna je za odgovarajući tip pokazivača.

Kao što je pomenuto u poslednjem odeljku, za svaku deklarisanu metodu, kompajleri će takođe deklarisati odgovarajuću implicitnu funkciju za nju. Dakle, za implicitno deklarisanu metodu, kompajler deklariše sledeću implicitnu funkciju.

```go
func (*Book).Pages(b *Book) int {
    return Book.Pages(*b)
}
```

Drugim rečima, za svaku eksplicitno deklarisanu metodu sa prijemnikom vrednosti, istovremeno će biti deklarisane i dve implicitne funkcije i jedna implicitna metoda.

## Specifikacije metoda i skupovi metoda

Specifikacija metode može se posmatrati kao prototip funkcije bez funcključne reči. Možemo videti da je svaka deklaracija metode sastavljena od funcključne reči, deklaracije parametra prijemnika, specifikacije metode i tela metode (funkcije).

Na primer, specifikacije metoda za Pagesi SetPagesmetode prikazane gore su

```go
Pages() int
SetPages(pages int)
```

Svaki tip ima skup metoda. Skup metoda tipa koji nije interfejs sastoji se od svih specifikacija metoda deklarisanih, eksplicitno ili implicitno, za taj tip, osim onih čija su imena prazan identifikator _. Tipovi interfejsa biće objašnjeni u sledećem članku .

Na primer, skupovi metoda tipa Bookprikazanog u prethodnim odeljcima su

```go
Pages() int
```

a skup metoda tipa *Bookje

```go
Pages() int
SetPages(pages int)
```

Redosled specifikacija metoda u skupu metoda nije važan za sam skup metoda.

Za skup metoda, ako se svaka specifikacija metode u njemu nalazi i u drugom skupu metoda, onda kažemo da je prvi skup metoda podskup drugog, a drugi je nadskup prvog. Ako su dva skupa metoda podskupovi (ili nadskupovi) jedan drugog, onda kažemo da su dva skupa metoda identična.

Za dat tip T, pretpostavimo da nije ni tip pokazivača ni tip interfejsa, iz razloga pomenutog u poslednjem odeljku, skup metoda tipa Tje uvek podskup skupa metoda tipa \*T. Na primer, skup metoda tipa Bookprikazan gore je podskup skupa metoda tipa \*Book.

Imajte u vidu da će neeksportovana imena metoda, koja počinju malim slovima, iz različitih paketa uvek biti posmatrana kao dva različita imena metoda, čak i ako su ta dva imena metoda bukvalno ista.

Skupovi metoda igraju važnu ulogu u polimorfizmu u jeziku Go. O polimorfizmu, pročitajte sledeći članak (interfejsi u jeziku Go) za detalje.

Skupovi metoda sledećih tipova su uvek prazni:

- ugrađeni osnovni tipovi.
- definisani tipovi pokazivača.
- tipovi pokazivača čiji su osnovni tipovi interfejsni ili pokazivački tipovi.
- neimenovani tipovi nizova, kriški, mapa, funkcija i kanala.

## Vrednosti metoda i pozivi metoda

Metode su zapravo specijalne funkcije. Metode se često nazivaju članske funkcije. Kada tip poseduje metodu, svaka vrednost tipa će posedovati nepromenljivog člana funkcionalnog tipa. Ime člana je isto kao i ime metode, a tip člana je isti kao i funkcija deklarisana sa oblikom deklaracije metode, ali bez dela prijemnika.

Poziv metode je samo poziv takvoj članskoj funkciji. Za vrednost v, njena metoda mmože biti predstavljena selektorskim oblikom v.m, što je vrednost funkcije.

Primer koji sadrži neke pozive metoda:

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var book Book

    fmt.Printf("%T \n", book.Pages)       // func() int
    fmt.Printf("%T \n", (&book).SetPages) // func(int)
    // &book has an implicit method.
    fmt.Printf("%T \n", (&book).Pages) // func() int

    // Call the three methods.
    (&book).SetPages(123)
    book.SetPages(123) // equivalent to the last line
    fmt.Println(book.Pages())    // 123
    fmt.Println((&book).Pages()) // 123
}
```

> [!Note]
> Za razliku od C jezika, u Go-u ne postoji -> operator za pozivanje metoda sa
> pokazivačkim prijemnicima, tako da (&book)->SetPages(123)je nedozvoljeno u
> Go-u.

Čekajte! Zašto se linija book.SetPages(123) u gornjem primeru kompajlira ispravno? Na kraju krajeva, metoda SetPagesnije deklarisana za taj Booktip. S jedne strane, ovo se može posmatrati kao sintaksički šećer radi lakšeg programiranja. Ovaj šećer radi samo za adresabilne prijemnike vrednosti. Kompajler će implicitno uzeti adresu adresabilne vrednosti bookkada se ona prosledi kao argument prijemnika SetPagespoziva metode. S druge strane, trebalo bi da mislimo aBookExpression.SetPagesda je uvek legalan selektor (sa stanovišta sintakse), čak i ako aBookExpressionse izraz proceni kao neadresabilna Bookvrednost, u kom slučaju je selektor aBookExpression.SetPagesnevažeći (ali legalan).

Kao što je gore pomenuto, kada se metod deklariše za tip, svaka vrednost tog tipa će posedovati funkciju članicu. Nulte vrednosti nisu izuzeci, bez obzira da li su nulte vrednosti tipova predstavljene sa ili ne nil.

Primer:

```go
package main

type StringSet map[string]struct{}
func (ss StringSet) Has(key string) bool {
    // Never panic here, even if ss is nil.
    _, present := ss[key]
    return present
}

type Age int
func (age *Age) IsNil() bool {
    return age == nil
}
func (age *Age) Increase() {
    *age++ // If age is a nil pointer, then
           // dereferencing it will panic.
}

func main() {
    _ = (StringSet(nil)).Has   // will not panic
    _ = ((*Age)(nil)).IsNil    // will not panic
    _ = ((*Age)(nil)).Increase // will not panic

    _ = (StringSet(nil)).Has("key") // will not panic
    _ = ((*Age)(nil)).IsNil()       // will not panic

    // This following line will panic. But the
    // panic is not caused by invoking the method.
    // It is caused by the nil pointer dereference
    // within the method body.
    ((*Age)(nil)).Increase()
}
```

## Argumenti prijemnika se prenose kopijom

Baš kao i opšti argumenti funkcije, argumenti prijemnika se takođe prenose kopiranjem. Dakle, modifikacije direktnog dela argumenta prijemnika u pozivu metode neće se odraziti izvan metode.

Primer:

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) SetPages(pages int) {
    b.pages = pages
}

func main() {
    var b Book
    b.SetPages(123)
    fmt.Println(b.pages) // 0
}
```

Još jedan primer:

```go
package main

import "fmt"

type Book struct {
    pages int
}

type Books []Book

func (books Books) Modify() {
    // Modifications on the underlying part of
    // the receiver will be reflected to outside
    // of the method.
    books[0].pages = 500
    // Modifications on the direct part of the
    // receiver will not be reflected to outside
    // of the method.
    books = append(books, Book{789})
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{500} {456}]
}
```

Malo van teme, ako se dva reda u redosledu gore navedene Modifymetode zamene, onda se obe modifikacije neće odraziti izvan tela metode.

```go
func (books Books) Modify() {
    books = append(books, Book{789})
    books[0].pages = 500
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{123} {456}]
}
```

Razlog je taj što appendće poziv dodeliti novi blok memorije za čuvanje elemenata kopije prosleđenog argumenta prijemnika sloja. Dodela se neće odraziti na sam prosleđeni argument prijemnika sloja.

Da bi se obe modifikacije odrazile izvan tela metode, prijemnik metode mora biti pokazivač jedan.

```go
func (books *Books) Modify() {
    *books = append(*books, Book{789})
    (*books)[0].pages = 500
}

func main() {
    var books = Books{{123}, {456}}
    books.Modify()
    fmt.Println(books) // [{500} {456} {789}]
}
```

## Normalizacija vrednosti metode

Tokom kompajliranja, kompajleri će normalizovati svaki izraz vrednosti metode, promenom implicitnog uzimanja adrese i dereferenciranja pokazivača u eksplicitne u tom izrazu vrednosti metode.

Assume vje vrednost tipa Ti v.mje legalan izraz vrednosti metode,

- Ako mje metoda eksplicitno deklarisana za tip *T, onda će je kompajleri normalizovati kao (&v).m;

- Ako mje metoda eksplicitno deklarisana za tip T, onda je izraz vrednosti metode v.mveć normalizovan.

Assume pje vrednost tipa *Ti p.mje legalan izraz vrednosti metode,

- Ako mje metoda eksplicitno deklarisana za tip T, onda će je kompajleri normalizovati kao (*p).m;

- Ako mje metoda eksplicitno deklarisana za tip *T, onda je izraz vrednosti metode p.mveć normalizovan.

Normalizacija vrednosti promovisane metode biće objašnjena u sledećem članku o ugrađivanju tipova .

## Procena vrednosti metode

Assume v.mje normalizovani izraz vrednosti metode, tokom izvršavanja, kada v.mse vrednost metode procenjuje, argument prijemnika vse procenjuje i kopija rezultata procene se čuva i koristi u kasnijim pozivima vrednosti metode.

Na primer, u sledećem kodu,

- Izraz vrednosti metode b.Pagesje već normalizovan. Tokom izvršavanja, kopija argumenta prijemnika bse čuva. Kopija je ista kao Book{pages: 123}, naknadna modifikacija vrednosti bnema uticaja na ovu kopiju. Zato poziv f1()ispisuje 123.

- Izraz vrednosti metode p.Pagesje normalizovan kao (\*p).Pagesu vreme kompajliranja. U vreme izvršavanja, argument prijemnika \*p se procenjuje na trenutnu bvrednost, koja je Book{pages: 123}. Kopija rezultata procene se čuva i koristi u kasnijim pozivima vrednosti metode, zato poziv f2()takođe ispisuje 123.

- Izraz vrednosti metode p.Pages2je već normalizovan. Tokom izvršavanja, kopija argumenta prijemnika pse čuva. Sačuvana vrednost je adresa vrednosti b, stoga će se sve promene u bodraziti kroz dereferenciranje sačuvane vrednosti, zato poziv g1()ispisuje 789.

- Izraz vrednosti metode b.Pages2je normalizovan kao (&b).Pages2u vreme kompajliranja. U vreme izvršavanja, kopija rezultata evaluacije &bse čuva. Sačuvana vrednost je adresa vrednosti b, stoga će se sve promene na bodraziti kroz dereferenciranje sačuvane vrednosti, zato poziv g2()ispisuje 789.

```go
package main

import "fmt"

type Book struct {
    pages int
}

func (b Book) Pages() int {
    return b.pages
}

func (b *Book) Pages2() int {
    return (*b).Pages()
}

func main() {
    var b = Book{pages: 123}
    var p = &b
    var f1 = b.Pages
    var f2 = p.Pages
    var g1 = p.Pages2
    var g2 = b.Pages2
    b.pages = 789
    fmt.Println(f1()) // 123
    fmt.Println(f2()) // 123
    fmt.Println(g1()) // 789
    fmt.Println(g2()) // 789
}
```

Definisani tip ne dobija metode eksplicitno deklarisane za izvorni tip koji se koristi u njegovoj definiciji

Na primer, u sledećem kodu, za razliku od definisanog tipa MyInt, definisani tip Agenema IsOddmetodu.

```go
package main

type MyInt int
func (mi MyInt) IsOdd() bool {
    return mi%2 == 1
}

type Age MyInt

func main() {
    var x MyInt = 3
    _ = x.IsOdd() // okay
    
    var y Age = 36
    // _ = y.IsOdd() // error: y.IsOdd undefined
    _ = y
}
```

Da li metod treba deklarisati sa prijemnikom pokazivača ili prijemnikom vrednosti?

Prvo, iz poslednjeg odeljka znamo da ponekad moramo deklarisati metode sa pokazivačkim prijemnicima.

U stvari, uvek možemo deklarisati metode sa pokazivačkim prijemnicima bez ikakvih logičkih problema. Samo je pitanje performansi programa da je ponekad bolje deklarisati metode sa vrednosnim prijemnicima.

Za slučajeve kada su i prijemnici vrednosti i prijemnici pokazivača prihvatljivi, evo nekih faktora koje treba uzeti u obzir pri donošenju odluka.

- Previše kopija pokazivača može prouzrokovati veće opterećenje za sakupljač smeća.

- Ako je veličina tipa prijemnika vrednosti velika, onda troškovi kopiranja argumenta prijemnika možda nisu zanemarljivi. Tipovi pokazivača su svi tipovi male veličine .

- Deklarisanje metoda i prijemnika vrednosti i prijemnika pokazivača za isti osnovni tip verovatnije će izazvati trke podataka ako se deklarisane metode pozivaju istovremeno u više gorutina.

- Vrednosti tipova u syncstandardnom paketu ne treba kopirati, tako da je deklarisanje metoda sa prijemnicima vrednosti za strukturne tipove čije je ugrađivanje tipova u syncstandardni paket problematično.

Ako je teško doneti odluku da li metod treba da koristi prijemnik pokazivača ili prijemnik vrednosti, onda jednostavno izaberite način prijema pokazivača.

[Kanali][0208] | [Sadržaj][00] | [Interfejsi][0210]

[0208]: 0208_Kanali.md
[00]: 00_Sadrzaj.md
[0210]: 0210_Interfejsi.md
