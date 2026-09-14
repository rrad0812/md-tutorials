
# Ključne reči i identifikatori u Go-u

[Elementi koda][0101] | [Sadržaj][00] | [Osnovni tipovi i osnovni vrednosni literali][0103]

Ovaj članak će predstaviti ključne reči i identifikatore u Go-u.

## Ključne reči

Ključne reči su posebne reči koje pomažu kompajlerima da razumeju i analiziraju korisnički kod.

Do sada (Go 1.25), Go ima samo 25 ključnih reči:

  . | . | . | . | . |
 ----- | ------- | ---- | --------- | ------ |
 break | default | func | interface | select |
 case | defer | go | map | struct |
 chan | else | goto | package | switch |
 const | fallthrough | if | range | type |
 continue | for | import | return | var |

Mogu se svrstati u četiri grupe:

- **const**, **func**, **import**, **package**, **type** i **var** se koriste za deklarisanje svih vrsta elemenata koda u Go programima.
- **chan**, **interface**, **map** i **struct** se koriste kao delovi u nekim denotacijama složenog tipa.
- **break**, **case**, **continue**, **default**, **else**, **fallthrough**, **for**, **goto**, **if**, **range**, **return**, **select** i **switch** se koriste za kontrolu toka koda.
- **defer** i **go** takođe su ključne reči za kontrolu toka, ali na druge specifične načine. One modifikuju pozive funkcija, o čemu ćemo govoriti u ovom članku.

Ove ključne reči će biti detaljnije objašnjene u drugim člancima.

## Identifikatori

Identifikator je token koji mora biti sastavljen od **Unicode slova**, **Unicode cifara** i **_**
(donje crte), i mora početi ili sa **Unicode slovom** ili sa **_**. Ovde,

- **Unikod slova** označavaju znakove definisane u kategorijama slova Lu , Ll , Lt , Lm ili Lo.
- **Unikod cifre** označavaju znakove definisane u kategoriji brojeva Nd Unikod standarda.

Ključne reči se ne mogu koristiti kao identifikatori.

Identifikator **_** je poseban identifikator, naziva se prazan identifikator.

Kasnije ćemo naučiti da sva imena tipova, promenljivih, konstanti, oznaka, imena paketa i imena importovanih paketa moraju biti identifikatori.

Identifikator koji počinje velikim slovom Unikoda naziva se **exported** identifikator. Reč **exported** može se tumačiti kao **public** u mnogim drugim jezicima. Identifikatori koji ne počinju velikim slovom Unikoda nazivaju se **non-exported** identifikatori. Reč **non-exported** može se tumačiti kao **private** u mnogim drugim jezicima. Trenutno (Go 1.25), **eastern** znakovi se smatraju neizvezenim slovima. Ponekad se neizvezeni identifikatori nazivaju i neizvezeni identifikatori.

Evo nekih legalnih izvezenih identifikatora:

```go
Player_9
DoSomething
VERSION
Ĝo
Π
```

Evo nekih legalnih identifikatora koji se ne izvoze:

```go
_
_status
memStat
book
π
一个类型
변수
エラー
```

A evo i nekih tokena koji se ne smeju koristiti kao identifikatori:

```go
// Starting with a Unicode digit.
123
3apples

// Containing Unicode characters not
// satisfying the requirements.
a.b
*ptr
$name
a@b.c

// These are keywords.
type
range
```

[Elementi koda][0101] | [Sadržaj][00] | [Osnovni tipovi i osnovni vrednosni literali][0103]

[0101]: 0101_Elementi_koda.md
[00]: 00_Sadrzaj.md
[0103]: 0103_Osnovni_tipovi_%20i_osnovni_vrednosni_literali.md
