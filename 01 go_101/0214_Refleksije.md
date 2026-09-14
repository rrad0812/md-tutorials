
# Refleksije u Gou

[Generici][0213] | [Sadržaj][00] | [Pravila za prelom reda][0301]

Go je statički jezik sa dobrom podrškom za refleksiju. Ostatak ovog članka će objasniti funkcionalnosti refleksije koje su obezbeđene u **reflect** standardnom paketu.

Veoma je korisno pročitati pregled sistema tipova i interfejsa Go u Go člancima pre čitanja ostatka trenutnog članka.

## Pregled Go Reflection

Refleksija u Gou donosi mnoge dinamičke funkcionalnosti u Go programiranje. Mnogi standardni paketi koda, kao što su paketi **fmt** i **encoding**, u velikoj meri se oslanjaju na funkcionalnosti refleksije.

Možemo ispitati Go vrednosti kroz vrednosti tipova **Type** i **Value** definisanih u **reflect** standardnom paketu. U ostatku ovog članka biće prikazani neki primeri o tome kako se koriste vrednosti ova dva tipa.

Jedan od ciljeva dizajna refleksije u Go-u je da svaka operacija bez refleksije bude moguća i putem refleksijskih načina. Iz raznih razloga, ovaj cilj nije 100% postignut. Međutim, većina operacija bez refleksije sada se može primeniti putem refleksijskih načina. S druge strane, putem refleksijskih načina možemo da izvršimo neke operacije koje je nemoguće postići putem nerefleksijskih načina. Operacije koje se ne mogu i mogu postići samo putem refleksijskih načina biće pomenute u narednim odeljcima.

### Tip reflect.Type i vrednosti

U programskom jeziku Go možemo da kreiramo **reflect.Type** vrednost iz proizvoljne vrednosti koja nije interfejs pozivanjem funkcije **reflect.TypeOf**. Rezultatna **reflect.Type** vrednost predstavlja tip vrednosti koja nije interfejs. Naravno, možemo i da prosledimo vrednost interfejsa pozivu **reflect.TypeOf** funkcije, ali poziv će vratiti **reflect.Type** vrednost koja predstavlja dinamički tip vrednosti interfejsa. U stvari, funkcija **reflect.TypeOf** ima samo jedan parametar tipa **interface{}** i uvek vraća **reflect.Type** vrednost koja predstavlja dinamički tip jedinog parametra interfejsa. Da bismo dobili **reflect.Type** vrednost koja predstavlja tip interfejsa korišćenjem **reflect.TypeOf** funkcije, moramo da koristimo indirektne načine koji će biti predstavljeni u nastavku, da bismo postigli ovaj cilj.

Od verzije Go 1.22, možemo koristiti **reflect.TypeFor** funkciju za dobijanje **reflect.Type** vrednosti koja predstavlja tip poznat u vreme kompajliranja. Tip poznat u vreme kompajliranja može biti ili interfejsni ili neinterfejsni.

Tip **reflect.Type** je tip interfejsa. On određuje nekoliko metoda. Možemo pozvati ove metode da bismo pregledali informacije o tipu koji predstavlja **reflect.Type** vrednost prijemnika. Neke od ovih metoda se primenjuju na sve vrste tipova, neke od njih su specifične za jednu ili nekoliko vrsta. Molimo vas da pročitate dokumentaciju svake metode za detalje. Pozivanje jedne od metoda preko nepravilne **reflect.Type** vrednosti prijemnika izazvaće paniku.

Primer:

```go
package main

import "fmt"
import "reflect"

func main() {
    type A = [16]int16
    var c <-chan map[A][]byte
    tc := reflect.TypeOf(c)
    fmt.Println(tc.Kind())    // chan
    fmt.Println(tc.ChanDir()) // <-chan
    tm := tc.Elem()
    ta, tb := tm.Key(), tm.Elem()
    // The next line prints: map array slice
    fmt.Println(tm.Kind(), ta.Kind(), tb.Kind())
    tx, ty := ta.Elem(), tb.Elem()

    // byte is an alias of uint8
    fmt.Println(tx.Kind(), ty.Kind()) // int16 uint8
    fmt.Println(tx.Bits(), ty.Bits()) // 16 8
    fmt.Println(tx.ConvertibleTo(ty)) // true
    fmt.Println(tb.ConvertibleTo(ta)) // false

    // Slice and map types are incomparable.
    fmt.Println(tb.Comparable()) // false
    fmt.Println(tm.Comparable()) // false
    fmt.Println(ta.Comparable()) // true
    fmt.Println(tc.Comparable()) // true
}
```

U Gou postoji 26 vrsta tipova.

U gornjem primeru, koristimo metodu **Elem** da dobijemo tipove elemenata nekih tipova kontejnera (tip kanala, tip mape, tip isečka i tip niza). U stvari, ovu metodu možemo koristiti i da dobijemo osnovni tip tipa pokazivača. Na primer,

```go
package main

import "fmt"
import "reflect"

type T []interface{m()}
func (T) m() {}

func main() {
    tp := reflect.TypeOf(new(interface{}))
    tt := reflect.TypeOf(T{})
    fmt.Println(tp.Kind(), tt.Kind()) // ptr slice

    // Get two interface Types indirectly.
    ti, tim := tp.Elem(), tt.Elem()
    // The next line prints: interface interface
    fmt.Println(ti.Kind(), tim.Kind())

    fmt.Println(tt.Implements(tim))  // true
    fmt.Println(tp.Implements(tim))  // false
    fmt.Println(tim.Implements(tim)) // true

    // All types implement any blank interface type.
    fmt.Println(tp.Implements(ti))  // true
    fmt.Println(tt.Implements(ti))  // true
    fmt.Println(tim.Implements(ti)) // true
    fmt.Println(ti.Implements(ti))  // true
}
```

Gore navedeni primer takođe pokazuje kako (indirektno) dobiti **reflect.Type** vrednost koja predstavlja tip interfejsa.

Refleksijom možemo dobiti sve tipove polja (strukturnog tipa) i informacije o metodi tipa. Takođe možemo dobiti informacije o parametru i tipu rezultata funkcionalnog tipa putem refleksije.

```go
package main

import "fmt"
import "reflect"

type F func(string, int) bool
func (f F) m(s string) bool {
    return f(s, 32)
}
func (f F) M() {}

type I interface{m(s string) bool; M()}

func main() {
    var x struct {
        F F
        i I
    }
    tx := reflect.TypeOf(x)
    fmt.Println(tx.Kind())        // struct
    fmt.Println(tx.NumField())    // 2
    fmt.Println(tx.Field(1).Name) // i
    // Package path is an intrinsic property of
    // non-exported selectors (fields or methods).
    fmt.Println(tx.Field(0).PkgPath) // 
    fmt.Println(tx.Field(1).PkgPath) // main

    tf, ti := tx.Field(0).Type, tx.Field(1).Type
    fmt.Println(tf.Kind())               // func
    fmt.Println(tf.IsVariadic())         // false
    fmt.Println(tf.NumIn(), tf.NumOut()) // 2 1
    t0, t1, t2 := tf.In(0), tf.In(1), tf.Out(0)
    // The next line prints: string int bool
    fmt.Println(t0.Kind(), t1.Kind(), t2.Kind())

    fmt.Println(tf.NumMethod(), ti.NumMethod()) // 1 2
    fmt.Println(tf.Method(0).Name)              // M
    fmt.Println(ti.Method(1).Name)              // m
    _, ok1 := tf.MethodByName("m")
    _, ok2 := ti.MethodByName("m")
    fmt.Println(ok1, ok2) // false true
}
```

Iz gornjeg primera, mogli smo da otkrijemo da,

- Za tipove koji nisu interfejsi, **reflect.Type.NumMethod** vraća samo broj izvezenih metoda (uključujući implicitno deklarisane) tipa. Ne možemo da dobijemo informacije o neizvezenoj metodi korišćenjem metode **reflect.Type.MethodByName**. Za interfejsne tipove, ograničenja ne postoje (činjenica nije pomenuta u dokumentaciji dve metode pre verzije Go 1.16). Takva situacija važi i za odgovarajuće metode tipa **reflect.Value** predstavljene u sledećem odeljku.
- Iako **reflect.Type.NumField** poziv metode vraća broj svih polja (uključujući i ona koja nisu izvezena) strukturnog tipa, nije dobra ideja koristiti **reflect.Type.FieldByName** metodu za dobijanje informacija o polju koje nije izvezeno.

Možemo pregledati oznake strukturnih polja putem refleksije. Tipovi oznaka strukturnih polja su **reflect.StructTag**, koja ima dve metode, **Get** i **Lookup**, za pregled parova ključ-vrednost navedenih u oznakama polja. Primer pregleda oznaka strukturnih polja:

```go
package main

import "fmt"
import "reflect"

type T struct {
    X    int  `max:"99" min:"0" default:"0"`
    Y, Z bool `optional:"yes"`
}

func main() {
    t := reflect.TypeOf(T{})
    x := t.Field(0).Tag
    y := t.Field(1).Tag
    z := t.Field(2).Tag
    fmt.Println(reflect.TypeOf(x)) // reflect.StructTag
    // v is a string
    v, present := x.Lookup("max")     
    fmt.Println(len(v), present)      // 2 true
    fmt.Println(x.Get("max"))         // 99
    fmt.Println(x.Lookup("optional")) //  false
    fmt.Println(y.Lookup("optional")) // yes true
    fmt.Println(z.Lookup("optional")) // yes true
}
```

**Napomena**:

- Ključevi oznaka ne smeju da sadrže razmak (vrednost Unikoda 32), navodnike (vrednost Unikoda 34) i dvotačku (vrednost Unikoda 58).
- Da bi se formirao važeći par ključ-vrednost, nije dozvoljeno da se razmak stavi iza tačke-zareza u pretpostavljenom paru ključ-vrednost. Dakle,
`optional: "yes"`ne formira par ključ-vrednost.
- Razmaci u vrednostima oznaka su važni (ne smeju se ignorisati). Dakle
  `json:"author, omitempty"`, ,
  `json:" author,omitempty"` i
  `json:"author,omitempty"`se razlikuju jedan od drugog.
- Svaka oznaka strukturnog polja treba da se predstavi kao jedan red da bi bila u potpunosti smislena.

Pored **reflect.TypeOf** funkcije, možemo koristiti i neke druge funkcije iz **reflect** standardnom paketu da bismo kreirali **reflect.Type** vrednosti koje predstavljaju neke neimenovane kompozitne tipove.

```go
package main

import "fmt"
import "reflect"

func main() {
    ta := reflect.ArrayOf(5, reflect.TypeOf(123))
    fmt.Println(ta) // [5]int
    tc := reflect.ChanOf(reflect.SendDir, ta)
    fmt.Println(tc) // chan<- [5]int
    tp := reflect.PtrTo(ta)
    fmt.Println(tp) // *[5]int
    ts := reflect.SliceOf(tp)
    fmt.Println(ts) // []*[5]int
    tm := reflect.MapOf(ta, tc)
    fmt.Println(tm) // map[[5]int]chan<- [5]int
    tf := reflect.FuncOf([]reflect.Type{ta},
                []reflect.Type{tp, tc}, false)
    fmt.Println(tf) // func([5]int) (*[5]int, chan<- [5]int)
    tt := reflect.StructOf([]reflect.StructField{
        {Name: "Age", Type: reflect.TypeOf("abc")},
    })
    fmt.Println(tt)            // struct { Age string }
    fmt.Println(tt.NumField()) // 1
}
```

Postoje još **reflect.Type** metode koje nisu korišćene u gornjim primerima, molimo vas da pročitate dokumentaciju **reflect** paketa za njihovu upotrebu.

**Napomena, do sada (Go 1.25)**:

- Ne postoje načini za kreiranje tipova interfejsa putem refleksije.
- Iako možemo da kreiramo strukturni tip sa ugrađivanim poljima putem refleksije, taj način ima mnogo zamki.

### Tip reflect.Value i vrednosti

Slično tome, možemo kreirati **reflect.Value** vrednost iz proizvoljne vrednosti koja nije interfejs pozivanjem funkcije **reflect.ValueOf**. Rezultatna **reflect.Value** vrednost predstavlja vrednost koja nije interfejs. Isto kao i **reflect.TypeOf** funkcija, **reflect.ValueOf** funkcija takođe ima samo jedan parametar tipa **interface{}**. Kada se argument tipa interfejsa prosledi pozivu **reflect.ValueOf** funkcije, poziv će vratiti **reflect.Value** vrednost koja predstavlja dinamičku vrednost argumenta interfejsa. Da bismo dobili **reflect.Value** vrednost koja predstavlja vrednost interfejsa, moramo koristiti indirektne načine koji će biti predstavljeni u nastavku da bismo postigli ovaj cilj.

Vrednost koju predstavlja neka **reflect.Value** vrednost često se naziva osnovna vrednost od v.

Postoji mnogo metoda deklarisanih za ovaj **reflect.Value** tip. Možemo pozvati ove metode da bismo pregledali informacije o (i manipulisali) osnovnoj vrednosti **reflect.Value** prijemnika vrednosti. Neke od ovih metoda se primenjuju na sve vrste vrednosti, neke od njih su specifične za jednu ili više vrsta. Molimo vas da pročitate dokumentaciju **reflect** standardnog paketa za detalje. Pozivanje metode specifične za vrstu sa nepravilnom **reflect.Value** vrednošću prijemnika izazvaće paniku.

Metoda **CanSet** vrednosti **reflect.Value** vraća da li je osnovna vrednost vrednosti **reflect.Value** modifikovana (može li se dodeliti). Ako je vrednost Go **modifikovana**, možemo pozvati metodu **Set** odgovarajuće **reflect.Value** vrednosti da bismo modifikovali vrednost Go. Imajte na umu da su **reflect.Value** vrednosti vraćene direktno **reflect.ValueOf** pozivima funkcija uvek samo za čitanje.

Primer:

```go
package main

import "fmt"
import "reflect"

func main() {
    n := 123
    p := &n
    vp := reflect.ValueOf(p)
    fmt.Println(vp.CanSet(), vp.CanAddr()) // false false
    vn := vp.Elem() // get the value referenced by vp
    fmt.Println(vn.CanSet(), vn.CanAddr()) // true true
    vn.Set(reflect.ValueOf(789)) // <=> vn.SetInt(789)
    fmt.Println(n)               // 789
}
```

Neeksportovana polja strukturnih vrednosti ne mogu se menjati putem refleksija.

```go
package main

import "fmt"
import "reflect"

func main() {
    var s struct {
        X interface{} // an exported field
        y interface{} // a non-exported field
    }
    vp := reflect.ValueOf(&s)
    // If vp represents a pointer. the following
    // line is equivalent to "vs := vp.Elem()".
    vs := reflect.Indirect(vp)
    // vx and vy both represent interface values.
    vx, vy := vs.Field(0), vs.Field(1)
    fmt.Println(vx.CanSet(), vx.CanAddr()) // true true
    // vy is addressable but not modifiable.
    fmt.Println(vy.CanSet(), vy.CanAddr()) // false true
    vb := reflect.ValueOf(123)
    vx.Set(vb)     // okay, for vx is modifiable
    // vy.Set(vb)  // will panic, for vy is unmodifiable
    fmt.Println(s) // {123 <nil>}
    fmt.Println(vx.IsNil(), vy.IsNil()) // false true
}
```

Iz gornja dva primera možemo saznati da postoje dva načina da se dobije **reflect.Value** vrednost čija je osnovna vrednost referencirana osnovnom vrednošću (vrednošću pokazivača) druge **reflect.Value** vrednosti.

- Jedan je pozivanje **Elem** metode vrednosti **reflect.Value** koja predstavlja vrednost pokazivača.
- Drugi je da se pozivu funkcije prosledi **reflect.Value** vrednost koja predstavlja vrednost pokazivača **reflect.Indirect**. (Ako argument prosleđen **reflect.Indirect** pozivu funkcije ne predstavlja vrednost pokazivača, onda poziv vraća kopiju argumenta.)

Treba napomenuti da se **reflect.Value.Elem** metoda može koristiti i za dobijanje **reflect.Value** vrednosti koja predstavlja dinamičku vrednost vrednosti interfejsa. Na primer,

```go
package main

import "fmt"
import "reflect"

func main() {
    var z = 123
    var y = &z
    var x interface{} = y
    v := reflect.ValueOf(&x)
    vx := v.Elem()
    vy := vx.Elem()
    vz := vy.Elem()
    vz.Set(reflect.ValueOf(789))
    fmt.Println(z) // 789
}
```

Standardni **reflect** paket takođe deklariše neke **reflect.Value** povezane funkcije. Svaka od ovih funkcija odgovara ugrađenoj funkciji ili funkcionalnosti koja ne zahteva refleksiju. Sledeći primer pokazuje kako povezati (vrstu) prilagođene generičke funkcije sa različitim vrednostima funkcije.

```go
package main

import "fmt"
import "reflect"

func InvertSlice(args []reflect.Value) []reflect.Value {
    inSlice, n := args[0], args[0].Len()
    outSlice := reflect.MakeSlice(inSlice.Type(), 0, n)
    for i := n-1; i >= 0; i-- {
        element := inSlice.Index(i)
        outSlice = reflect.Append(outSlice, element)
    }
    return []reflect.Value{outSlice}
}

func Bind(p interface{},
        f func ([]reflect.Value) []reflect.Value) {
    // invert represents a function value.
    invert := reflect.ValueOf(p).Elem()
    invert.Set(reflect.MakeFunc(invert.Type(), f))
}

func main() {
    var invertInts func([]int) []int
    Bind(&invertInts, InvertSlice)
    fmt.Println(invertInts([]int{2, 3, 5})) // [5 3 2]

    var invertStrs func([]string) []string
    Bind(&invertStrs, InvertSlice)
    fmt.Println(invertStrs([]string{"Go", "C"})) // [C Go]
}
```

Ako je osnovna vrednost od **reflect.Value** vrednost funkcije, onda možemo pozvati **Call** metod od **reflect.Value** da bismo pozvali osnovnu funkciju.

```go
package main

import "fmt"
import "reflect"

type T struct {
    A, b int
}

func (t T) AddSubThenScale(n int) (int, int) {
    return n * (t.A + t.b), n * (t.A - t.b)
}

func main() {
    t := T{5, 2}
    vt := reflect.ValueOf(t)
    vm := vt.MethodByName("AddSubThenScale")
    results := vm.Call([]reflect.Value{reflect.ValueOf(3)})
    fmt.Println(results[0].Int(), results[1].Int()) // 21 9

    neg := func(x int) int {
        return -x
    }
    vf := reflect.ValueOf(neg)
    fmt.Println(vf.Call(results[:1])[0].Int()) // -21
    fmt.Println(vf.Call([]reflect.Value{
        vt.FieldByName("A"), // panic on changing to "b"
    })[0].Int()) // -5
}
```

Imajte u vidu da se neeksportovana polja ne smeju koristiti kao argumenti refleksivnih poziva. Ako se red *vt.FieldByName("A")* u gornjem primeru zameni sa *vt.FieldByName("b")*, doći će do panike.

Primer refleksije za vrednosti mape.

```go
package main

import "fmt"
import "reflect"

func main() {
    valueOf := reflect.ValueOf
    m := map[string]int{"Unix": 1973, "Windows": 1985}
    v := valueOf(m)
    // A zero second Value argument means to delete an entry.
    v.SetMapIndex(valueOf("Windows"), reflect.Value{})
    v.SetMapIndex(valueOf("Linux"), valueOf(1991))
    for i := v.MapRange(); i.Next(); {
        fmt.Println(i.Key(), "\t:", i.Value())
    }
}
```

Imajte na umu da je **MapRange** metoda podržana od verzije Go 1.12.

Primer refleksije za vrednosti kanala.

```go
package main

import "fmt"
import "reflect"

func main() {
    c := make(chan string, 2)
    vc := reflect.ValueOf(c)
    vc.Send(reflect.ValueOf("C"))
    succeeded := vc.TrySend(reflect.ValueOf("Go"))
    fmt.Println(succeeded) // true
    succeeded = vc.TrySend(reflect.ValueOf("C++"))
    fmt.Println(succeeded) // false
    fmt.Println(vc.Len(), vc.Cap()) // 2 2
    vs, succeeded := vc.TryRecv()
    fmt.Println(vs.String(), succeeded) // C true
    vs, sentBeforeClosed := vc.Recv()
    fmt.Println(vs.String(), sentBeforeClosed) // Go true
    vs, succeeded = vc.TryRecv()
    fmt.Println(vs.String()) // <invalid Value>
    fmt.Println(succeeded)   // false
}
```

Metode **TrySend** i **TryRecv** odgovaraju **select** blokovima koda toka upravljanja po principu jedan slučaj-jedan podrazumevani parametr.

Možemo koristiti **reflect.Select** funkciju za simulaciju **select** bloka koda sa dinamičkim brojem **case** grana tokom izvršavanja.

```go
package main

import "fmt"
import "reflect"

func main() {
    c := make(chan int, 1)
    vc := reflect.ValueOf(c)
    succeeded := vc.TrySend(reflect.ValueOf(123))
    fmt.Println(succeeded, vc.Len(), vc.Cap()) // true 1 1

    vSend, vZero := reflect.ValueOf(789), reflect.Value{}
    branches := []reflect.SelectCase{
        {Dir: reflect.SelectDefault, Chan: vZero, Send: vZero},
        {Dir: reflect.SelectRecv, Chan: vc, Send: vZero},
        {Dir: reflect.SelectSend, Chan: vc, Send: vSend},
    }
    selIndex, vRecv, sentBeforeClosed := reflect.Select(branches)
    fmt.Println(selIndex)         // 1
    fmt.Println(sentBeforeClosed) // true
    fmt.Println(vRecv.Int())      // 123
    vc.Close()
    // Remove the send case branch this time,
    // for it may cause panic.
    selIndex, _, sentBeforeClosed = reflect.Select(branches[:2])
    fmt.Println(selIndex, sentBeforeClosed) // 1 false
}
```

Odgovarajuće osnovne vrednosti nekih **reflect.Value** vrednosti mogu biti **nil**. Na primer, nulte **reflect.Value** vrednosti.

```go
package main

import "reflect"
import "fmt"

func main() {
    var z reflect.Value // a zero Value value
    fmt.Println(z)      // <invalid reflect.Value>
    v := reflect.ValueOf((*int)(nil)).Elem()
    fmt.Println(v)      // <invalid reflect.Value>
    fmt.Println(v == z) // true
    var i = reflect.ValueOf([]interface{}{nil}).Index(0)
    fmt.Println(i)             // <nil>
    fmt.Println(i.Elem() == z) // true
    fmt.Println(i.Elem())      // <invalid reflect.Value>
}
```

Za Go vrednost, možemo koristiti **reflect.ValueOf** funkciju da kreiramo **reflect.Value** vrednost koja predstavlja Go vrednost, uz pomoć **interface{}**. Inverzni proces je sličan, možemo pozvati **Interface** metodu vrednosti **reflect.Value** da bismo dobili **interface{}** vrednost, a zatim otkucati **assert** na vrednosti **interface{}** da bismo dobili Go vrednost predstavljenu sa (tj. osnovnu vrednost ) vrednosti **reflect.Value**. Ali imajte na umu da pozivanje **Interface** metode **reflect.Value** vrednosti koja predstavlja neizvezeno polje izaziva paniku.

```go
package main

import (
    "fmt"
    "reflect"
    "time"
)

func main() {
    vx := reflect.ValueOf(123)
    vy := reflect.ValueOf("abc")
    vz := reflect.ValueOf([]bool{false, true})
    vt := reflect.ValueOf(time.Time{})

    x := vx.Interface().(int)
    y := vy.Interface().(string)
    z := vz.Interface().([]bool)
    m := vt.MethodByName("IsZero").Interface().(func() bool)
    fmt.Println(x, y, z, m()) // 123 abc [false true] true

    type T struct {x int}
    t := &T{3}
    v := reflect.ValueOf(t).Elem().Field(0)
    fmt.Println(v)             // 3
    fmt.Println(v.Interface()) // panic
}
```

Metod **reflect.Value.IsZero** je predstavljen u Go 1.13. Koristi se za proveru da li **reflect.Value** osnovna vrednost jednaka vrednosti nula vrednosti.

Od verzije Go 1.17, isečak se može konvertovati u pokazivač niza. Međutim, takva konverzija može izazvati paniku ako je dužina pokazivača osnovnog tipa niza prevelika. Metod **reflect.Value.CanConvert(T reflect.Type)** predstavljen u Go 1.17 se koristi za proveru da li će konverzija biti uspešna ili ne.

Primer korišćenja CanConvertmetode:

```go
package main

import (
    "fmt"
    "reflect"
)

func main() {
    s := reflect.ValueOf([]int{1, 2, 3, 4, 5})
    ts := s.Type()
    t1 := reflect.TypeOf(&[5]int{})
    t2 := reflect.TypeOf(&[6]int{})
    fmt.Println(ts.ConvertibleTo(t1)) // true
    fmt.Println(ts.ConvertibleTo(t2)) // true
    fmt.Println(s.CanConvert(t1))     // true
    fmt.Println(s.CanConvert(t2))     // false
}
```

Postoje i druge **reflect.Value** srodne funkcije i metode koje se ne koriste u gornjim primerima, molimo vas da pročitate dokumentaciju **reflect** paketa za njihovu upotrebu. Pored toga, imajte na umu da postoje neki detalji vezani za refleksiju pomenuti u detaljima o programu Go 101.

[Generici][0213] | [Sadržaj][00] | [Pravila za prelom reda][0301]

[0213]: 0213_Generici.md
[00]: 00_Sadrzaj.md
[0301]: 0301_Pravila%20za%20prelom%20reda.md
