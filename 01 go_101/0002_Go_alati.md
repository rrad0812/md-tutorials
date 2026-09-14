
# Go alati

[Uvod u Go][0001] | [Sadržaj][00] | [Elementi koda][0101]

Trenutno, alati u zvaničnom Go Toolchain-u (kasnije nazvanom Go Toolchain) su najkorišćeniji alati za razvoj Go projekata. U seriji članaka Go 101, svi primeri su kompajlirani i verifikovani standardnim Go kompajlerom koji je dostupan u Go Toolchain-u.

Ovaj članak će predstaviti kako se podešava Go razvojno okruženje i kako se koristi go komanda koja se nalazi u Go Toolchain-u. Takođe će biti predstavljeni i neki drugi alati iz Go Toolchain-a.

## Instaliranje Go Toolchain

Preuzmite Go Toolchain i instalirajte ga prateći uputstva prikazana na stranici za preuzimanje.

Takođe možemo koristiti alat "GoTV" za upravljanje instalacijama više verzija Go alata i njihovo harmonično i praktično korišćenje.

Verzija izdanja Go Toolchain-a je u skladu sa najvišom jezičkom verzijom koju izdanje podržava. Na primer, Go Toolchain 1.25.x podržava sve jezičke verzije Go-a od 1.0 do Go 1.25.

Putanja do "bin" podfoldera u korenskoj putanji instalacije Go Toolchain-a mora biti uneta u PATH promenljivu okruženja da bi se alati (poput "go" komande) koji su obezbeđeni u Go Toolchain-u izvršili bez unosa njihovih punih putanja. Ako je vaš Go Toolchain instaliran putem instalera ili pomoću menadžera paketa, putanja do "bin" podfoldera je možda već automatski podešena u PATH promenljivoj okruženja.

Novije verzije Go Toolchain-a podržavaju funkciju pod nazivom moduli za upravljanje zavisnostima projekata. Funkcija je eksperimentalno predstavljena u verziji 1.11 i podrazumevano je omogućena od verzije 1.16.

- Postoji promenljiva okruženja, GOPATH, koje bi trebalo da budemo svesni, mada nam ne treba previše obraćati pažnju na nju. Podrazumevano je da je to putanja do dira "go" u početnom direktorijumu trenutnog korisnika. Promenljiva GOPATH okruženja može da navede više putanja ako je ručno navedena. Kasnije, kada se pomene GOPATH dir, to znači dir na prvoj putanji u GOPATH promenljivoj okruženja.

- Subdir "pkg" ispod GOPATH dira se koristi za čuvanje keširanih verzija Go modula (Go modul je kolekcija Go paketa) koji zavise od lokalnih Go projekata.

- Postoji GOBIN promenljiva okruženja koja kontroliše gde će se čuvati binarne datoteke Go programa generisane potkomandom "go install". Vrednost GOBIN promenljive okruženja je podrazumevano podešena na putanju "bin" subdira unutar GOPATH dira. Putanja GOBIN treba da bude podešena u PATH promenljivoj okruženja ako želite da pokrenute generisane binarne datoteke Go programa bez navođenja njihovih punih putanja.

## Najjednostavniji program Go

Hajde da napišemo jednostavan primer i naučimo kako da pokrenemo jednostavne Go programe.

Sledeći program je najjednostavniji Go program:

```go
package main

func main() {
}
```

- **package main** navodi ime paketa ( "main" ovde ) koome pripada datoteka sa kodom.

- Drugi red je prazan radi bolje čitljivosti.

- **main** funkcija u **main** paketu određuje ulaznu tačku programa.

> [!Note]
> **Redosled izvršavanja koda**  
> Imajte na umu da se neki drugi korisnički kod može izvršiti pre nego što se funkcija **main**
> pozove.

### Pokretanje Go programa

Go Toolchain zahteva da Go datoteka izvornog koda ima ekstenziju ".go". Ovde pretpostavljamo da je gore navedeni izvorni kod sačuvan u datoteci pod nazivom "simplest-go-program.go".

Otvorite terminal i promenite trenutni direktorijum u direktorijum koji sadrži gore navedenu izvornu datoteku, a zatim pokrenite komandu

```go
go run simplest-go-program.go
```

Ništa se ne prikazuje? Da, ovaj program ne prikazuje ništa, jer je telo funkcije "main" prazno.

Ako postoje neke sintaksne greške u izvornom kodu, onda će te greške biti prijavljene pri kompajliranju.

Ako se u paketu programa nalazi više izvornih datoteka u paketu "main", onda bi trebalo da pokrenemo program sledećom komandom

```go
go run .
```

> [!Note]
> **Pokretanje go programa**
>
> - Komanda **go run** se ne preporučuje za kompajliranje i pokretanje velikih Go projekata. To je samo zgodan način za pokretanje jednostavnih Go programa, poput onih u člancima Go 101. Za velike Go projekte, koristite komande **go build** ili **go install** da biste izgradili, a zatim pokrenuli izvršne binarne datoteke.
>
> - Svaki formalni Go projekat koji podržava Go module zahteva **go.mod** datoteku koja se nalazi u korenskom folderu tog projekta. **go.mod** datoteku može generisati potkomanda **go mod init** (videti dole).
>
> - Go Toolchain ignoriše izvorne datoteke koje počinju sa **_** ili **..**.

## Više o go subkomandama

- Tri komande, **go run**, **go build** i **go install** samo prikazuju sintaksičke greške u kodu (ako ih ima). One ne (pokušavaju da) prikazuju upozorenja u kodu (tj. moguće logičke greške u kodu).
  
- Možemo koristiti komandu **go vet** da proverimo i prijavimo takva upozorenja.

- Možemo (i često bi trebalo) da koristimo **go fmt** komandu za formatiranje izvornog koda Go-a sa doslednim stilom kodiranja.

- Možemo koristiti **go test** komandu za pokretanje testova i benčmarkova.

- Možemo koristiti **go doc** komandu za pregled Go dokumentacije u terminalnim prozorima.

- Toplo se preporučuje da omogućite svojim Go projektima da podrže funkciju Go modula kako bi se olakšalo upravljanje zavisnostima. Za projekte koji podržavaju Go module komanda **go mod init example.com/myproject** se koristi za generisanje **go.mod** datoteke u trenutnom direktorijumu, koja će se posmatrati kao korenski direktorijum modula pod nazivom "example.com/myproject". "go.mod" datoteka će se koristiti za snimanje zavisnosti modula. Datoteku "go.mod" možemo ručno uređivati ili dozvoliti go podkomandama da je ažuriraju.

- Komanda **go mod tidy** se koristi za dodavanje nedostajućih zavisnosti modula u datoteku i uklanjanje neiskorišćenih zavisnosti modula iz "go.mod" datoteke analiziranjem celog izvornog koda trenutnog projekta.

- Komanda **go get** se koristi za dodavanje/nadogradnju/snižavanje nivoa/uklanjanje jedne zavisnosti. Ova komanda se koristi ređe od **go mod tidy** komande.

- Od verzije Go Toolchain 1.16, možemo pokrenuti komandu **go install example.com/program@latest** da bismo instalirali najnoviju verziju Go programa treće strane (u GOBIN folder). Pre verzije Go Toolchain 1.16, odgovarajuća komanda je bila **go get -u example.com/program**, koja je sada zastarela.

- Možemo koristiti **go help SubCommand** komandu da bismo videli poruku pomoći za određenu subkomandu.

- Pokrenite **go** komandu bez ikakvih argumenata da biste prikazali podržane subkomande.

Serija članaka Go 101 neće mnogo više objašnjavati kako se koriste alati koji su dostupni u Go Toolchain-u. Molimo vas da pročitate zvaničnu dokumentaciju za detalje.

## Pogledajte dokumentaciju Go paketa u pregledačima

Mogli bismo da koristimo alatku za pregled dokumentacije i izvornog koda Golds ( go101.org/golds ) da bismo lokalno pregledali dokumentaciju određenog Go projekta. Ovo nije zvanični alat. Razvio sam ga ja (Tapir Liu). Golds podržava bogata iskustva čitanja koda (kao što je navođenje odnosa implementacije tipova). Nakon instaliranja Golds-a , možemo da pokrenemo **golds std ./...** da bismo pregledali dokumentaciju standardnih paketa i paketa u trenutnom diru.

Za zvaničnu Go dokumentaciju, posetite **go.dev**.

[Uvod u Go][0001] | [Sadržaj][00] | [Elementi koda][0101]

[0001]: 0001_Uvod.md
[00]: 00_Sadrzaj.md
[0101]: 0101_Elementi_koda.md
