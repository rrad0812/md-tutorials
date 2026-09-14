
# Izrazi, iskazi i jednostavni iskazi

[Paketi i uvoz paketa][0107] | [Sadržaj][00] | [Osnovna kontrola toka][0109]

Ovaj članak će predstaviti izraze i iskaze u jeziku Go.

Jednostavno govoreći, izraz predstavlja vrednost, a iskaz predstavlja operaciju. Međutim, u stvari, neki specijalni izrazi mogu biti sastavljeni od nekoliko vrednosti i predstavljaju ih, a neki iskazi mogu biti sastavljeni od nekoliko podoperacija/izraza. U zavisnosti od konteksta, neki iskazi se takođe mogu posmatrati kao izrazi.

Jednostavni iskazi su neke posebni iskazi. U jeziku Go, neki delovi svih vrsta tokova upravljanja moraju biti jednostavni iskazi, a neki delovi moraju biti izrazi. Tokovi upravljanja biće predstavljeni u sledećem članku o programu Go 101.

Ovaj članak neće davati tačne definicije za izraze i iskaze. Teško je to postići. Ovaj članak će navesti samo neke slučajeve izraza i iskaza. Neće biti obuhvaćene sve vrste izraza i iskaza u ovom članku, ali će biti navedene sve vrste jednostavnih iskaza.

## Neki slučajevi izraza

Većina izraza u jeziku Go su izrazi sa jednom vrednošću. Svaki od njih predstavlja jednu vrednost. Drugi izrazi predstavljaju više vrednosti i nazivaju se izrazi sa više vrednosti.

U okviru ovog dokumenta, kada se pominje izraz, mislimo da je to izraz sa jednom vrednošću, osim ako nije drugačije naznačeno.

Vrednosni literali, promenljive i imenovane konstante su svi izrazi sa jednom vrednošću, koji se nazivaju i elementarni izrazi.

Operacije (bez delova dodele) koje koriste operatore predstavljene u članku, "Uobičajeni operatori", su svi izrazi sa jednom vrednošću.

Ako funkcija vrati barem jedan rezultat, onda su njeni pozivi (bez delova dodele) izrazi. Konkretno, ako funkcija vrati više od jednog rezultata, onda njeni pozivi pripadaju izrazima sa više vrednosti. Pozivi funkcija bez rezultata nisu izrazi.

Metode se mogu posmatrati kao posebne funkcije. Dakle, gore navedeni slučajevi funkcija važe i za metode. Metode će biti detaljno objašnjene u članku o metodama u Go-u kasnije.

U stvari, kasnije ćemo saznati da su prilagođene funkcije, uključujući metode, sve vrednosti funkcija, tako da su one takođe (jednovrednosni) izrazi. Više o tipovima funkcija i vrednostima ćemo saznati kasnije.

Operacije prijema kanala (bez delova dodele) su takođe izrazi. Operacije kanala biće objašnjene kasnije u članku o kanalima u Go-u.

Neki izrazi u jeziku Go, uključujući operacije prijema kanala, mogu imati opcione rezultate u jeziku Go. Takvi izrazi mogu se predstaviti i kao izrazi sa jednom i kao izrazi sa više vrednosti, u zavisnosti od različitih konteksta. Takve izraze možemo kasnije pročitati u drugim člancima o programu Go 101.

## Jednostavni slučajevi iskaza

Postoji šest vrsta jednostavnih iskaza.

- kratke forme za deklaraciju promenljivih
- čiste dodele vrednosti (ne mešanje sa deklaracijama promenljivih), uključujući **x op= y** operacije.
- pozivi funkcija/metoda i operacije prijema kanala. Kao što je pomenuto u poslednjem odeljku, ovi jednostavni iskazi se takođe mogu koristiti kao izrazi.
- operacije slanja na kanal.
- ništa (tj. prazni iskazi). Naučićemo neke upotrebe praznih iskaza u sledećem članku.
- x++ i x--.

Ponovo, operacije prijema i slanja kanala biće predstavljene u članku o kanalima u programu Go.

Imajte na umu da se **x++** i **x--** ne mogu koristiti kao izrazi. Go ne podržava sintaksne oblike **++x** i **--x**.

## Neki slučajevi nejednostavnih iskaza

Nepotpuna lista nejednostavnih iskaza:

- standardni obrasci deklaracije promenljivih. Da, kratke deklaracije promenljivih su jednostavni iskazi, ali standardne nisu.
- deklaracije imenovanih konstanti.
- deklaracije prilagođenih tipova.
- deklaracije za uvoz paketa.
- eksplicitni blokovi koda. Eksplicitni blok koda počinje sa { i završava se sa }. Blok koda može da sadrži mnogo podizrazas.
- deklaracije funkcija. Deklaracija funkcije može da sadrži mnogo podizraza.
- kontrolni tokovi i skokovi izvršavanja koda.
- return linije u deklaracijama funkcija.
- Odloženi pozivi funkcija i kreiranje gorutina.

Primeri izraza i iskaza

```go
// Some non-simple statements.
import "time"
var a = 123
const B = "Go"
type Choice bool
func f() int {
    for a < 10 {
        break
    }

    // This is an explicit code block.
    {
        //...
    }
    return 567
}

// Some simple statements:
c := make(chan bool) // channels will be explained later
a = 789
a += 5
a = f() // here f() is used as an expression
a++
a--
c <- true // a channel send operation
z := <-c  // a channel receive operation used as the
          // source value in an assignment statement.

// Some expressions:
123
true
B
B + " language"
a - 789
a > 0 // an untyped boolean value
f     // a function value of type "func ()"

// The following ones can be used as both
// simple statements and expressions.
f()
<-c // a channel receive operation
```

[Paketi i uvoz paketa][0107] | [Sadržaj][00] | [Osnovna kontrola toka][0109]

[0107]: 0107_Paketi_i_uvoz_paketa.md
[00]: 00_Sadrzaj.md
[0109]: 0109_Osnovna_kontrola_toka.md
