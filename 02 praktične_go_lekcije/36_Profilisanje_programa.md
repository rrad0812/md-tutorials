
# 36 Profilisanje programa

[35 Izgradnja HTTP klijenta][35] | [00 Sadržaj][00] | [37 Kontekst][37]

**Šta ćete naučiti u ovom poglavlju?**

- Šta je profilisanje?
- Kako kreirati profil procesora vašeg programa.
- Kako kreirati Heap profil vašeg programa.
- Kako se koristi pprofalat za analizu profila.
- Neke uobičajene tehnike optimizacije korišćenja procesora.

**Obrađeni tehnički koncepti**:

- Profil
- pprof
- Protokolski baferi
- protokol
- Procesor
- Profil gomile

## Šta je profilisanje

Profilisanje je tehnika optimizacije programa. "Profilisati program" znači prikupljanje detaljnih statistika o tome kako program radi. Te statistike mogu biti korišćenje procesora, alokacija memorije, vreme provedeno na programskim rutinama, broj poziva funkcija... itd.

Ali koja je razlika u odnosu na benčmark? Benčmark prikuplja informacije o određenoj funkciji tokom izvršavanja. Profilisanje je prikupljanje statistike za ceo program.

## Zašto profilisanje

Profilisanje se često koristi kada se primeti pad performansi. Alat se koristi da bi se razumelo zašto program ne radi kako treba.

Statička analiza kodne baze može biti nedovoljna da otkrije zašto se program loše ponaša. Benčmarkovi testiraju performanse izolovane funkcije; oni nisu dovoljni da bi se razumela cela slika.

Profilisanje takođe može biti alat za poboljšanje programskog inženjeringa. Go je relativno efikasan jezik, ali loše dizajnirani programi mogu patiti od problema sa performansama. Ti problemi se mogu lako razumeti i ispraviti uz ljubaznu pomoć profilisanja.

## Početak rada sa profilisanjem

Da bismo profilisali program, možemo koristiti paket `runtime/pprof` koji pruža potreban API za pokretanje i zaustavljanje profilisanja.

U ovom odeljku ćemo predstaviti program koji sabira cele brojeve. Program se sastoji samo od glavnog paketa sa jednom funkcijom pod nazivom "doSum". Ova funkcija će sabirati cele brojeve od 0 do 787766776:

```go
package main

import (
    "fmt"
)

func main() {
    result := doSum()
    fmt.Println(result)
}

func doSum()int{
    sum := 0
    for i := 0; i < 787766777; i++ {
        sum += i
    }
    return sum
}
```

Sledeći korak je dodavanje poziva pprof API-ju unutar naše glavne funkcije:

```go
// profiling/getting-started/main.go 

f, err := os.Create("profile.pb.gz")
if err != nil {
    log.Fatal(err)
}
err = pprof.StartCPUProfile(f)
if err != nil {
    log.Fatal(err)
}
defer pprof.StopCPUProfile()
```

Ovaj blok koda mora biti postavljen unutar `main`. Kreiraće datoteku pod nazivom "profile.pb.gz". Zatim će pokrenuti profilisanje procesora i upisati rezultat profila u ovu datoteku ( pprof.`StartCPUProfile(f))`. Na kraju glavne funkcije, pozvaćemo `pprof.StopCPUProfile()`. Da bismo poboljšali čitanje, koristili smo odloženu naredbu.

Zatim moramo da izgradimo naš program (nazvaćemo ga binarni "gettingstarted"):

```sh
go build -o gettingstarted main.go
```

Napraviće binarnu datoteku u trenutnom direktorijumu. Da bismo prikupili podatke, moramo pokrenuti naš program:

```sh
./gettingstarted
```

Možete videti da je kreirana datoteka "profile.pb.gz". Možete pokušati da otvorite datoteku da biste vizualizovali rezultat, ali to nije baš dobra ideja. Ova datoteka je kompresovana, a drugo. Moraćemo da koristimo alat za vizualizaciju rezultata našeg profilisanja!

Ovaj program je `pprof`, Gugl ga je razvio da bi omogućio "vizuelizaciju i analizu podataka profilisanja". Pprof može da čita profile i generiše izveštaje o njima na lako čitljiv način. Za početak, koristićemo interfejs komandne linije `pprof` da bismo vizuelizovali našu statistiku profilisanja (videćemo druge tehnike vizuelizacije kasnije u ovom poglavlju):

```sh
go tool pprof gettingstarted cpu.profile
```

Jednostavno pozivamo `go tool pprof`, navodeći kao prvi argument putanju do binarne datoteke ( "gettingstarted" ) našeg programa, a zatim datoteku profila ( "cpu.profile" ). Ova komanda će pokrenuti interaktivni režim. Moraćete da unesete komande da biste prikazali statistiku:

```sh
go tool pprof gettingstarted profile.pb.gz
File: gettingstarted
Type: cpu
Time: Jan 1, 2021 at 9:27pm (CET)
Duration: 413.54ms, Total samples = 220ms (53.20%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

U standardnom izlazu možete videti da pprof prikazuje:

- ime binarne datoteke (ovde: gettingstarted)
- vrsta profila (ovde cpu)
- datum kada smo generisali profil
- ukupno trajanje izvršavanja programa ( 413ms)
- ukupan broj uzoraka (vratićemo se kasnije na to šta je uzorak jer može biti zbunjujuće)

## Čitanje datoteka profila

Da bismo mogli da čitamo datoteke profila, potrebno je da instaliramo program koji je obezbedio Google: `protoc`. Zašto? Pošto datoteke profila imaju određeni format, to su `protobuf` datoteke.

### Reč o protokolskim baferima (protobuf)

Protokol baferi su razvijeni interno od strane kompanije Google i napravljeni su kao otvoreni kod. Oni mogu transformisati strukturirane podatke u lagan format koji se može čuvati i prenositi preko mreže. Proces transformacije strukturiranih podataka u određeni format naziva se serijalizacija. Podaci serijalizovani (ili kodirani) ovom metodom su veoma lagan. Vraćeni format se naziva "binarna žica".

Za razliku od XML-a ili JSON-a, imena polja se ne prevode u serijalizovanu verziju podataka. Stoga je veličina poruke manja. Potrebna vam je neka vrsta specifikacije da biste pročitali serijalizovanu poruku. Specifikacija serijalizacije je "proto datoteka". Proto datoteke imaju ekstenziju .proto.

Guglov tim je razvio alate u mnogim jezicima (C++, C#, Dart, Go, Java, Python, Ruby, Objective-C...) za laku serijalizaciju i deserijalizaciju podataka. U sledećem odeljku, koristićemo jedan od ovih alata za deserijalizaciju datoteke profila.

### Instalirajte kompajler protokolskog bafera (protoc)

#### Preuzmite najnovije izdanje

Prvo morate da preuzmete najnoviju verziju kompajlera. U vreme pisanja ovog teksta, najnovija verzija je 3.15.2. U sledećim redovima ću vam dati komandu za preuzimanje ove specifične verzije. Molimo vas da uvek preuzmete najnoviju verziju! Da biste dobili broj najnovije verzije, posetite GitHub stranicu <https://github.com/protocolbuffers/protobuf/releases>.

Preuzećemo već kompajliranu verziju softvera. Naravno, možete ga sami kompajlirati ako imate instaliran C++ kompajler na računaru, ali to nije najlakše rešenje.

Možete direktno preuzeti zip datoteke na <https://github.com/protocolbuffers/protobuf/releases>.

Takođe možete koristiti ove terminalne komande:

- Za računare sa Linux-om (64 bita):

  ```sh
  curl -OL <https://github.com/protocolbuffers/protobuf/releases/download/v3.15.2/protoc-3.15.2-linux-x86_64.zip>
  ```

- Za korisnike MacOs-a :

  ```sh
  curl -OL https://github.com/protocolbuffers/protobuf/releases/download/v3.15.2/protoc-3.15.2-osx-x86_64.zip
  ```

Za Mac i Linux, koristili smo cURL, CLI alatku za pravljenje HTTP zahteva. Komandi prosleđujemo dve zastavice:

- -O će zapisati preuzeti sadržaj u datoteku (nazvanu isto kao i datoteka na serveru)

- -L će pratiti preusmeravanje jer GitHub čuva izdanja na Amazon AWS S3 baketu (servis za
  skladištenje u oblaku)

Za korisnike Windows-a, URL adresa za preuzimanje izdanja je:
<https://github.com/protocolbuffers/protobuf/releases/download/v3.6.1/protoc-3.6.1-win32.zip>

Raspakujte objavljene datoteke.

Za korisnike Linuksa i Maka, možete koristiti komandnu liniju za raspakivanje datoteka zahvaljujući uslužnom programu unzip :

```sh
cd where/you/downloaded/the/zip/file
unzip protoc-3.15.2-osx-x86_64.zip -d protoc-3.15.2
```

Zastavica -d će staviti naduvane datoteke u navedeni direktorijum (biće kreiran ako ne postoji).

Za korisnike Windows-a, savetujem vam da koristite grafički interfejs za raspakivanje datoteka.

#### Dodajte binarnu datoteku svojoj putanji

Za korisnike Linuksa i Meka, postoji jedno pogodno mesto gde možete da smestite svoje binarne datoteke: "/usr/local/bin". Direktorijum "/usr/local" se koristi za lokalnu instalaciju softvera. "bin" folder će sadržati lokalne binarne datoteke. Ako ste radoznali u vezi sa hijerarhijom fajl sistema Unixa, pogledajte specifikaciju: <http://refspecs.linuxfoundation.org/FHS_2.3/fhs-2.3.html>

```sh
sudo mv protoc-3.15.2/bin/protoc /usr/local/bin
```

Moraćete da koristite sudo da biste premestili izvršnu datoteku u dir "/usr/local/bin".

### /bin/protocusr

Za korisnike Windows-a, moraćete da dodate binarnu datoteku u promenljivu okruženja PATH.

### Preuzmite dekodiranu verziju profila

Da biste dobili dekodiranu verziju datoteke profila, prvi korak je da nabavite ".proto" datoteku. Kodirani protokolski baferi ne sadrže imena polja kako bi se smanjila njihova veličina.

Datoteku.proto možemo pronaći na pprof GitHub repozitorijumu na sledećoj adresi: <https://github.com/google/pprof/blob/master/proto/profile.proto>. Go se isporučuje sa verzijom pprof koju možete pronaći u direktorijumu vendor unutar Go foldera sa izvornim kodom (u "src/cmd/vendor/GitHub.com/google/pprof/"). Međutim, izgleda da profile.proto nije uključen u verziju koju ste dobili u vreme pisanja ovog teksta.

Jedno rešenje je kloniranje `pprof` u vaš "src" folder

```sh
cd /path/to/your/dev/directory
git clone https://github.com/google/pprof.git
```

Zatim možemo da koristimo preuzetu proto datoteku. Vraćeni profil je gzip-ovan (komprimovan je). Prvo ga moramo raspakovati. Da bismo to uradili, koristićemo komandu gunzip (dostupnu za korisnike Linux-a i Mac-a):

```sh
gunzip profile.pb.gz
```

Ovim će se obrisati datoteka profile.pb.gz i kreirati nova datoteka pod nazivom profile.pb (što je raspakovana verzija datoteke profile.pb.gz). Korisnici Windows-a mogu koristiti alatku sa grafičkim korisničkim interfejsom da bi ovo postigli.

Onda možemo dekodirati datoteku protokolskog bafera.

```sh
protoc --decode perftools.profiles.Profile /path/to/your/dev/directory/pprof/proto/profile.proto --proto_path /path/to/your/dev/directory/pprof/proto < profile.pb
```

Na mom računaru:

```sh
protoc --decode perftools.profiles.Profile /Users/maximilienandile/Documents/DEV/pprof/proto/profile.proto --proto_path /Users/maximilienandile/Documents/DEV/pprof/proto <  profile.pb
```

#### Detalji o komandi protoc

Hajde da detaljno objasnimo elemente te komande da bismo to razjasnili:

- **--decode**: ova zastavica čeka tip poruke. Naš profil je poruka u terminologiji protokolskog
  bafera. Svaka poruka ima određeni tip. U našem slučaju, naša profilna poruka ima tip. Ovaj string ne dolazi niotkuda. Ako prikažete prve redove datoteke profile.proto, videćete da ima smisla:perftools.profiles.Profile

  ```go
  // github.com/google/pprof/blob/master/proto/profile.proto
  //...
  syntax = "proto3";
  
  package perftools.profiles;
  
  option java_package = "com.google.perftools.profiles";
  option java_outer_classname = "ProfileProto";
  
  message Profile {
  //...
  ```
  
  Ovo je naziv paketa, a zatim naziv poruke:perftools.profiles.Profile
  
  Sledeći argument komande protoc je putanja do datoteke proto

- **--proto_path**: ova zastavica je ovde da ukaže protoc-u gde može da pronađe.
  proto datoteke. Ova putanja mora da sadrži našu.proto datoteku. U suprotnom, protoc neće moći da obavi svoj posao.

Zatim prosleđujemo kodiranu poruku sa u `protoc < profile.pb`. Podaci iz datoteke `profile.pb` biće prosleđeni kao standardni ulaz u `protoc` program.

Evo rezultata ove komande:

```go
sample_type {
  type: 1
  unit: 2
}
sample_type {
  type: 3
  unit: 4
}
sample {
  location_id: 1
  location_id: 2
  location_id: 3
  value: 14
  value: 140000000
}
sample {
  location_id: 4
  location_id: 2
  location_id: 3
  value: 7
  value: 70000000
}
sample {
  location_id: 5
  location_id: 2
  location_id: 3
  value: 1
  value: 10000000
}
mapping {
  id: 1
  has_functions: true
}
location {
  id: 1
  mapping_id: 1
  address: 17482189
  line {
    function_id: 1
    line: 24
  }
}
location {
  id: 2
  mapping_id: 1
  address: 17482037
  line {
    function_id: 2
    line: 18
  }
}
location {
  id: 3
  mapping_id: 1
  address: 16946166
  line {
    function_id: 3
    line: 201
  }
}
location {
  id: 4
  mapping_id: 1
  address: 17482182
  line {
    function_id: 1
    line: 24
  }
}
location {
  id: 5
  mapping_id: 1
  address: 17482186
  line {
    function_id: 1
    line: 25
  }
}
function {
  id: 1
  name: 5
  system_name: 5
  filename: 6
}
function {
  id: 2
  name: 7
  system_name: 7
  filename: 6
}
function {
  id: 3
  name: 8
  system_name: 8
  filename: 9
}
string_table: ""
string_table: "samples"
string_table: "count"
string_table: "cpu"
string_table: "nanoseconds"
string_table: "main.doSum"
string_table: "/Users/maximilienandile/go/src/go_book/profiling/gettingstarted/main.go"
string_table: "main.main"
string_table: "runtime.main"
string_table: "/usr/local/go/src/runtime/proc.go"
time_nanos: 1546864261276935000
duration_nanos: 417027943
period_type {
  type: 3
  unit: 4
}
period: 10000000
```

Možemo videti da ova datoteka definiše:

- Vrste uzoraka
- Uzorci
- Mapiranja
- Lokacije
- Funkcije
- "Tabela stringova"
- Svojstva time_nanos, tip perioda i svojstvo period.

U narednim odeljcima, razumećete upotrebu tih svojstava.

## Šta je stek poziva

### Šta je stek poziva?

Stek je gomila predmeta. U stvarnom životu, možete napraviti stek sa drvetom, sa čašama šampanjca ili sa svim što se može složiti.

U računarstvu, slažemo pozive funkcija. Kada se program izvršava, počinje sa funkcijom. Glavna funkcija je prva funkcija koja se izvršava. A zatim ćemo pozivati druge funkcije, koje će pozivati druge funkcije...

Kada se vaš program pokrene, stek poziva će rasti... Možete dobiti stek poziva koristeći `debug` paket.

Stek poziva programa je uređena lista funkcija koje se trenutno izvršavaju.

### Primer steka poziva: kako odštampati stek

Uzmimo primer. U sledećem prikazu možete videti primer aplikacije koja definiše glavnu funkciju i dve druge funkcije "firstFunctionToBeCalled" i "secondFunctionToBeCalled". Glavna funkcija poziva, "firstFunctionToBeCalled" koja poziva "secondFunctionToBeCalled". U ovoj poslednjoj funkciji, ispisaćemo stek.

```go
// profiling/stack/main.go 
package main

import "runtime/debug"

func main() {
    firstFunctionToBeCalled()
}

func firstFunctionToBeCalled(){
    secondFunctionToBeCalled()
}

func secondFunctionToBeCalled(){
    debug.PrintStack()
}
```

Prethodni program je izbacio:

```go
$ go run main.go
goroutine 1 [running]:
runtime/debug.Stack(0xc00000e1b0, 0x1, 0x1)
    /usr/local/go/src/runtime/debug/stack.go:24 +0xa7
runtime/debug.PrintStack()
    /usr/local/go/src/runtime/debug/stack.go:16 +0x22
main.secondFunctionToBeCalled()
    /Users/maximilienandile/go/src/go_book/profiling/stack/main.go:14 +0x20
main.firstFunctionToBeCalled()
    /Users/maximilienandile/go/src/go_book/profiling/stack/main.go:10 +0x20
main.main()
    /Users/maximilienandile/go/src/go_book/profiling/stack/main.go:6 +0x20
```

Možete čitati stek od kraja do početka (od "main.main" do "runtime/debug.Stack"). Sve počinje sa "main". Zatim stek raste...

Stek pozivi se zatim analiziraju i koriste da bi se dao smisao merenju koje je napravio profiler.

## Procesorsko vreme

U ovom odeljku pokušaću da vam objasnim šta je vreme procesora. Bez jasnog razumevanja ovog pojma, sledeće odeljke će biti teško razumeti.

Procesorsko vreme predstavlja vreme centralne procesorske jedinice (CPU) potrebno za izvršavanje skupa instrukcija definisanih u vašem programu. Mikroprocesor obrađuje te instrukcije. Što je vaš program složeniji i vrši intenzivne proračune, to vam je potrebno više procesorskog vremena.

Procesorsko vreme možemo podeliti u dve podkategorije:

- Vreme korisnika procesora
- Sistemsko vreme procesora

Uzmimo primere da bismo bolje razumeli te koncepte.

```go
// profiling/understanding-CPU-profile/main.go
package main

import (
    "fmt"
    "log"
    "os"
    "runtime/pprof"
)

func main() {
    // set CPU profiling (1)
    f, err := os.Create("profile.pb.gz")
    if err != nil {
        log.Fatal(err)
    }
    err = pprof.StartCPUProfile(f)
    if err != nil {
        log.Fatal(err)
    }
    defer pprof.StopCPUProfile()
    // CPU intensive operation (2)
    test := 0
    for i := 0; i < 1000000000; i++ {
        test = i
    }
    fmt.Println(test)
}
```

U prethodnom prikazu, počinjemo sa standardnom sintaksom za pokretanje profilisanja procesora u našem programu. Zatim smo razvili for petlju koja iterira na brojevima od 0 do 1000000000 (isključeno). Unutar for petlje menjamo vrednost promenljive test na vrednost i (što je promenljiva brojača).

Hajde da napravimo naš program, a zatim ga pokrenemo:

```go
go build main.go
./main
999999999
```

Koliko vremena je potrebno da se program pokrene? Možemo ga ponovo pokrenuti pomoću uslužnog programa time (za korisnike Mac-a i Linux-a). Komanda će izvršiti program i prikazati statistiku:

```sh
./main
real    0m0.629s
user    0m0.518s
sys     0m0.005s
```

Kako protumačiti ovaj izlaz?

- 0m0. 629​​​s: ukupno vreme, što je vreme proteklo između pozivanja i završetka
  programa
- 0m0. 518​​​s sekunde odgovaraju vremenu korisničkog procesora. Ovo vreme odgovara vremenu koje
  je procesor (CPU) bio zauzet izvršavanjem instrukcija van jezgra (korisničkog prostora).
- 0m0.005​s ssekunde odgovaraju vremenu sistemskog procesora. Ova vremenska mera odgovara vremenu
  koje je potrebno procesoru da izvrši komande u prostoru jezgra, na primer, sistemske pozive (na primer, otvaranje datoteke).

Možete videti da sistemski pozivi zauzimaju samo 1 procenat procesorskog vremena. 99% procesorskog vremena se dešava u korisničkom prostoru.

Da bismo videli kako možemo promeniti te procente, ovog puta možemo napraviti program koji izvršava mnogo sistemskih poziva. Slučajno, imamo na raspolaganju syscall paket:

```go
// profiling/system-call/main.go
package main

import (
    "fmt"
    "io/ioutil"
    "log"
    "syscall"
    "time"
)

func main() {
    for i := 0; i < 100; i++ {
        fileName := fmt.Sprintf("/tmp/file%d", time.Now().UnixNano())
        err := ioutil.WriteFile(fileName, []byte("test"), 0644)
        if err != nil {
            log.Fatal(err)
        }
        err = syscall.Chmod(fileName, 0444)
        if err != nil {
            log.Fatal(err)
        }
    }
}
```

Ovde imamo for petlju koja će kreirati 100 datoteka. Putanju datoteke gradimo sa `fmt.Sprintf`. Svaka putanja je "/tmp/fileXXX" gde XXX predstavlja broj nanosekundi od UNIX epohe (broj nanosekundi proteklih od 00:00:00 četvrtka, 1. januara 1970. godine).

Zatim kreiramo datoteku uz pomoć `ioutil.Writefile`. Ovaj uslužni program će izvršiti dva sistemska poziva (`Open` i `Write`). Zatim, kada se datoteka kreira, promenićemo njen režim u "0444" uz pomoć sistemskog poziva `Chmod`.

Ovo je mnogo sistemskih poziva! Da vidimo šta se dešava sa našom statistikom vremena:

```sh
go build main.go
time./main

real    0m0.031s
user    0m0.004s
sys     0m0.023s
```

Korisnička linija odgovara samo 0,004 sekunde, dok sistemska linija odgovara 0,023 sekunde. Na slici možete vizuelno videti preraspodelu. CPU vreme za sistemske pozive sada predstavlja 85,2% ukupnog CPU vremena!

### Jezgro

U ovom odeljku koristili smo termin "jezgro". Jezgro se odnosi na centralnu komponentu operativnog sistema. Jezgro upravlja sistemskim resursima. Takođe upravlja različitim hardverskim komponentama računara. Kada vršimo sistemski poziv, koristimo mogućnosti jezgra. Na primer, otvaranje datoteke u programu Go pokrenuće sistemski poziv koji će jezgro obraditi.

## Šta je uzorak

Videli ste da profil sadrži brojne uzorke u dekodiranom profilu. Pojam uzorka može biti zbunjujući, pa želim da ga razjasnim.

Uzorak je merenje. Ovo merenje se vrši u određenom trenutku tokom procesa profilisanja. Kada profilišemo program, prikupljamo merenja, a ta merenja se materijalizuju u profilu pomoću uzoraka. Na slici 3 možete videti da je profil sastavljen od uzoraka koji sadrže:

- mera
- lokacija i
- opcione dodatne informacije.

Mera nije ista ako imate profil procesora ili profil memorije.

Kada aktivirate profilisanje procesora, vaš program će se zaustavljati svakih 100 milisekundi.

Svaki put kada profiler zaustavi program:

- prikuplja podatke
- podaci se analiziraju, mera se ekstrahuje
- Kreira se uzorak.

Prikupljeni podaci se sastoje od steka poziva. Kao što ste videli u poslednjem odeljku, trag će dati informacije o pozvanim funkcijama. Vreme procesora se takođe meri za svaki uzorak.

Alat `pprof` nam omogućava da vizualizujemo uzorke onako kako se pojavljuju u profilu. Da biste dobili ove podatke, jednostavno unesite sledeću komandu u terminal (ući ćete u interaktivni režim pprof-a)

```sh
go tool pprof main profile.pb.gz
File: main
Type: cpu
Time: Jan 13, 2019 at 7:07pm (CET)
Duration: 720.43ms, Total samples = 470ms (65.24%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

Zatim samo otkucajte "traces":

```sh
(pprof) traces
File: main
Type: cpu
Time: Jan 13, 2019 at 7:07pm (CET)
Duration: 720.43ms, Total samples = 470ms (65.24%)
-----------+-------------------------------------------------------
     240ms   main.main
             runtime.main
-----------+-------------------------------------------------------
     190ms   main.main
             runtime.main
-----------+-------------------------------------------------------
      30ms   runtime.nanotime
             runtime.sysmon
             runtime.mstart1
             runtime.mstart
-----------+-------------------------------------------------------
      10ms   main.main
             runtime.main
-----------+-------------------------------------------------------
```

Uzorci profila su napisani u tekstualnom režimu. Možete videti da u našem slučaju imamo četiri uzorka. Prva kolona predstavlja vrednosti uzoraka, vreme procesora. Druga kolona je trag.

Možemo videti da je za izradu našeg profila bilo potrebno 720,43 ms i da ukupan broj uzoraka čini 470 ms, što predstavlja 65,24% od 720,43 ms. Termin trajanje može biti zbunjujući. Ovo nije trajanje programa već trajanje profila. U našem slučaju, dve brojke su bliske, ali nisu jednake.

Ukupno vreme izvršavanja programa je manje od trajanja profila. To je zato što se profilisanje pokreće nakon početka programa, a zaustavlja se kada se završi glavna funkcija.

Prvi primer je tabela koja ima vrednost od 240ms CPU vremena. Stek pozivi kada su podaci prikupljeni imaju dužinu 2. Prva funkcija koja se poziva je "runtime.main", druga koja se poziva je "main.main". Vrednosti se ne akumuliraju u ovom prikazu.

## Pprof komandna linija

U ovom odeljku ćemo se fokusirati na komandnu liniju `pprof`. Kao što vidite, komandna linija pprof se sastoji od 4 elementa:

- Izlazni format za vizuelizaciju rezultata (npr. pdf, SVG, web...)
- Opcije koje mogu da preciziraju vizuelizaciju (na primer, zastavica -show će
  predstavljati samo čvorove koji se podudaraju sa regularnim izrazom )
- Putanja do binarne datoteke programa
- Putanja do izvora profila (što je datoteka protokolskog bafera)

U vreme pisanja ovog teksta, dostupna su 24 izlazna formata. Nećemo obraditi svaki pojedinačni format. Češći su `-web` koji će generisati SVG grafikon i prikazati ga u veb pregledaču, `-svg` i `-pdf` koji će vam omogućiti da lako delite svoje profile sa kolegama. U sledećem odeljku ćemo videti neke uobičajene slučajeve upotrebe.

## Prikaz profila u veb pregledaču

### Preduslov: Instalacija Graphviz-a

Moraćete da instalirate Graphviz (<https://gitlab.com/graphviz/graphviz>) na vaš računar da biste pokrenuli sledeću komandu.

- Graphviz je dostupan na homebrew-u za korisnike MacOS-a: `brew install graphviz`
- Za korisnike Linuksa: `apt-get install graphviz`
- Za Windows, uputstva za instalaciju za korisnike nalaze se na zvaničnoj veb stranici.

#### Napomena za korisnike Mac-a

Za korisnike Mac-a, možda ćete prvo morati da definišete podrazumevanu aplikaciju za otvaranje SVG-a. Na mom računaru, na primer, podrazumevana aplikacija je bila Latex. Ako se nađete u ovoj situaciji, možete promeniti podrazumevanu aplikaciju za otvaranje SVG-a u vašem omiljenom pregledaču. Kliknite desnim tasterom miša na SVG datoteku, a zatim kliknite na "Get Info". Otvoreni prikaz vam omogućava da izaberete podrazumevanu aplikaciju koju ćete koristiti za otvaranje. Izaberite onu koju želite, a zatim potvrdite izmene.

### Izvršite komandu

Da biste generisali SVG datoteku iz svog profila i otvorili je u veb pregledaču, možete koristiti veb zastavicu:

```sh
go tool pprof -web gettingstarted profile.pb
```

Ovo će kreirati SVG datoteku i otvoriti vaš pregledač da je prikaže.

Hajde da detaljno objasnimo šta je na ovom grafikonu.

Na slici možete videti da je proizvedeni SVG sastavljen od dva glavna dela.

Prvi deo (pravougaona kutija) sadrži detalje o profilu:

- Ime binarnog profilisanog
- Tip profila
- Vreme kada je profil generisan
- Vreme beleži profil (trajanje)
- Indikacija ukupnog broja uzoraka, koja se ovde vraća kao trajanje, može biti
  zbunjujuća.
- Indikacija o reprezentativnosti trenutnog pogleda:
  - označeno kao "Prikaz čvorova čini X, Y% od Z."
  - X predstavlja broj uzoraka koje prikazani čvorovi predstavljaju.
  - Z ukupan broj uzoraka dostupnih u profilu
  - Y predstavlja procenat X/Z %

Zatim na ovoj slici možete videti blokove koji se nazivaju čvorovi i strelice. Imamo usmereni graf (koji je takođe acikličan, što znači da ne sadrži cikluse) ako koristimo ispravnu terminologiju. Svaki čvor u ovom grafu predstavlja poziv funkcije. Na slici detaljno prikazujemo šta svaki element čvora znači:

Značenja čvorova u Pprof SVG formatu

Dobili ste informacije o paketu, pozvanu funkciju, vrednost uzorka i ukupan broj uzoraka profila na svakom čvoru. Ova poslednja statistika je veoma važna. Pomoću ove statistike možete uporediti težinu svakog čvora i videti gde se dešava pad performansi.

## Interaktivni veb interfejs

Generisane informacije o profilu mogu se pregledati u lepom veb interfejsu. Da biste ga pokrenuli, otkucajte:

```sh
go tool pprof -http localhost:9898 yourBinaryName profile.pb
```

Pokrenuće veb server na http:localhost:9898, gde možete videti statistiku.

Početna stranica interaktivnog veb interfejsa Pprof-a

Veb lista je zanimljiv prikaz. Omogućava vam da vidite izvorni kod vaše aplikacije sa statistikom profilisanja. Pored toga, dobijate i dizasemblovanu verziju profilisanog izvornog koda!

### Rastavljeno? Šta to znači

Kada kompajlirate svoj Go program, dobijate ono što se naziva "izvršna datoteka". Možete izvršiti izvršnu datoteku pomoću računara. Izvršna datoteka je napisana mašinskim jezikom. Mašinski jezik nije nemoguće pročitati, ali je zaista teško. Dizasembler je program koji može transformisati mašinski jezik u asemblerski jezik. Asemblerski jezik nije Go kod. To je skup instrukcija koje su veoma bliske arhitekturi računara (procesora), ali i njegovom operativnom sistemu.

Vizuelizacijom proizvedenog asemblerskog koda, možete bolje razumeti šta se dešava "ispod haube". Asembler nije baš popularan među programerima. Ako uzmemo anketu programera na Stackoverflow-u iz 2018. godine, samo 7,8% naših kolega kaže da ga koristi, dok je za "profesionalne" programere ta brojka niža: 6,8%.

## Šta je optimizacija koda?

Ovo je proces "modifikacije softverskog sistema kako bi neki njegov aspekt radio efikasnije ili koristio manje resursa".

Glavni cilj optimizacije koda je smanjenje vremena izvršavanja programa. Profilisanje je zanimljiv alat koji možemo koristiti za smanjenje vremena izvršavanja i potrošnje memorije programa.

Da biste optimizovali kod, potrebno ga je izmeniti, a ponekad ćete morati da napravite važne izmene da biste poboljšali performanse. Čang, Malke i Hvu, u radu objavljenom 1991. godine, napravili su zanimljivu razliku između dve vrste optimizacije koda:

- Prva je modifikacija delova koda koja smanjuje vreme izvršavanja ovog određenog dela bez modifikacije vremena izvršavanja bilo kojih drugih instrukcija u programu. Na primer, brisanje nekog starog koda će smanjiti vreme izvršavanja dela koji ste optimizovali; ova modifikacija neće uticati na ostale instrukcije.

- Drugi je optimizacija delova koda koji će povećati vreme izvršavanja drugih delova koda. Uzmimo primer, zamislite da imate for petlju, i unutar ove petlje pravite beskorisno izračunavanje. Možete izdvojiti ovo izračunavanje van petlje. Ako to uradite, smanjićete vreme izvršavanja petlje, ali ćete povećati vreme izvršavanja van petlje.

## Uobičajenih tehnika optimizacije

Optimizacija koda se uglavnom stiče iskustvom. Ali postoje neke uobičajene tehnike optimizacije koje bi trebalo da znate napamet!

### Uklanjanje mrtvog koda

Uvođenje mrtvog koda nije lako sa go-om. go kompajler će blokirati ako deklarišete promenljivu bez njenog korišćenja. To je veoma dobra stvar. Ali to ne znači da ne možete uvesti neke fragmente mrtvog koda.

Evo jednog primera:

```go
// profiling/classic-opti/deadCode/main.go
package main

import "fmt"

func main() {
    fmt.Println(multiply(1, 9))
}

func multiply(x, y int) int {
    a := x + 1
    b := a + 2*y
    b = b * b
    return x * y
}
```

Ovde imamo funkciju množenja koja prihvata dva cela broja (x i y). Ova funkcija će pomnožiti oba cela broja i vratiti rezultat.

U telu ove funkcije definišemo a i b. Te dve promenljive su beskorisne i možda su nasleđe starih izračunavanja koja su ranije rađena. Ne budite sentimentalni, obrišite te beskorisne redove:

```go
// profiling/classic-opti/deadCodeV2/main.go 
//...
func multiply(x,y int) int {
    return x * y
}
```

### Optimizacija izlaza iz petlje

Ovu tehniku optimizacije je veoma lako primeniti u praksi. Ako se suočite sa novim kodom, pažljivo ispitajte petlje i njihove izlazne uslove.

Petlje se često koriste za iteraciju kroz nizove ili isecanje da bi se pronašao određeni element kada ne znate njegov indeks. Tražite beskorisne iteracije: kada pronađete ono što tražite, nema potrebe da nastavljate petlju!

Uzmimo primer. Zamislite da je vaš zadatak da pronađete element u segmentu i prijavite njegov indeks. Mogli biste da napravite ovaj program

```go
// profiling/classic-opti/loopExit/main.go 
//...

// s is a slice of uint32
var search uint32 = 4285020338
var foundIndex int
var found bool
for j := 0; j < len(s); j++ {
    if s[j] == search {
        found = true
        foundIndex = j
    }
}
if found {
    fmt.Printf("found index is %d", foundIndex)
} else {
    fmt.Println("index not found")
}
```

U ovom listingu počinjemo sa definicijom 3 promenljive:

- search - koja će sadržati broj koji pokušavamo da pronađemo u isečku s,
- foundIndex koja će sadržati indeks pronađenog elementa i
- found, što je bulova vrednost true ako je broj pronađen.

Zatim pažljivo pogledajte petlju for... Ponavljamo sve elemente isečka. U svakoj iteraciji proveravamo da li je element koji se nalazi na trenutnom indeksu jednak celom broju koji tražimo. Ako jeste, postavljamo promenljive found i foundIndexex na njihove očekivane vrednosti.

Koji je interes nastaviti petlju ako pronađemo element? Napravićemo neka dodatna poređenja koja će oduzeti neko vreme. Možemo direktno izaći iz petlje ako pronađemo element:

```go
// profiling/classic-opti/loopExitV2/main.go 
//...

for j := 0; j < len(s); j++ {
    if s[j] == search {
        found = true
        foundIndex = j
        break
    }
}
```

Koristimo `break` naredbu koja će završiti petlju for.

### Uklanjanje koda nepromenljivog u petlji

Ova tehnika optimizacije se sastoji u pomeranju invarijantnih instrukcija izvan petlji. Te instrukcije imaju izvorni operand koji se ne menja unutar petlje.

Da bismo ovo razjasnili, uzmimo primer. Program koji moramo da napravimo mora da izračuna ukupan promet lanca prodavnica. Prodavnice dostavljaju holdingu svoje podatke o prometu. Za svaku prodavnicu troškovi moraju se oduzeti od poslatog prometa:

```go
// profiling/classic-opti/loopInvariant/main.go 
//...
var turnover uint32
for i := 0; i < len(s); i++ {
    var costs = 950722 + 12*uint32(len(s))
    turnover = turnover + s[i] - costs
}
fmt.Println(turnover)
```

Možemo primetiti da unutar for petlje, u svakoj iteraciji definišemo promenljivu `costs`. Ova promenljiva se podešava rezultatom složenog izračunavanja:

```go
var costs = 950722 + 12*uint32(len(s))
```

Izračunavanje troškova je i dalje isto za svaku prodavnicu. Ne zavisi od prometa prodavnica. Izvorni operand ( 950722 + 12*uint32(len(s)) ) se ne menja; on je nepromenljiv. Ovo izračunavanje možemo izdvojiti van ove for petlje. Program će ga izvršiti samo jednom, i stoga ćemo uštedeti malo procesorskog vremena:

```go
// profiling/classic-opti/loopInvariantV2/main.go 
//...

var turnover uint32
var costs = 950722 + 12*uint32(len(s))
for i := 0; i < len(s); i++ {
    turnover = turnover + s[i] - costs
}
fmt.Println(turnover)
```

### Fuzija petlji

Ako je vaš program sastavljen od dve petlje (ili više) koje:

- se izvršavaju pod istim uslovima
- su nezavisne (izvršavanje jedne petlje ne zavisi od izvršavanja druge)
- imaju isti broj pogubljenja

Možemo spojiti te petlje u jednu jedinstvenu petlju. Evo malog primera:

```go
// profiling/classic-opti/loopFusion/main.go 
//...

var costs float64
for i := 0; i < len(s); i++ {
    costs += 0.2*float64(s[i])
}
var turnover uint32
for i := 0; i < len(s); i++ {
    turnover += s[i]
}
```

Te dve petlje imaju isti broj iteracija, nezavisne su i obe se izvršavaju. Možemo ih spojiti:

```go
// profiling/classic-opti/loopFusionV2/main.go 
//...

var costs float64
var turnover uint32
for i := 0; i < len(s); i++ {
    costs += 0.2*float64(s[i])
    turnover += s[i]
}
```

Uštedećemo vreme procesora (umesto da petljom prelazimo preko isečka dva puta, vrednosti s ćemo preći samo jednom).

### Konstantno savijanje

Konstantno savijanje je tehnika optimizacije kompajlera koja se sastoji u proceni vrednosti konstanti tokom kompajliranja programa, a ne tokom njegovog izvršavanja.

Kompilator već radi ovo za nas. Ali korisno je zapamtiti! Ako vaš program definiše promenljivu koja čuva rezultat operacije, razmislite o mogućnosti kreiranja konstante umesto toga. Operacija će se izvršiti tokom kompajliranja i stoga ćete uštedeti dragoceno vreme procesora.

Uzmimo primer funkcije otpremanja koja je odgovorna za prenos datoteka sa jednog servera na drugi:

```go
// profiling/classic-opti/constantFolding/main.go 
//...

func upload(files []File)(error){
    uploadLimit := 10*2048/2
    for _,file := range files{
        if file.Size > uploadLimit {
            return errors.New("the upload limit has been reached")
        }
        // upload the file
    }
    return nil
}
```

Ovde je promenljiva "uploadLimit" podešena na: 10*2048/2

Problem je u tome što svaki put kada otpremamo datoteku na server, ponovo ćemo izračunati "uploadLimit", koji je konstanta. Možemo ga zameniti konstantom kako bi Go to izračunao kada se program kompajlira.

```go
// profiling/classic-opti/constantFoldingV2/main.go 
//....

const uploadLimit =10*2048/2

func upload(files []File)(error){
    for _,file := range files{
        if file.Size > uploadLimit {
            return errors.New("the upload limit has been reached")
        }
        // upload the file
    }
    return nil
}
```

## Problem optimizacije u stvarnom svetu

Mnogo smo govorili o profilisanju u prethodnim odeljcima; vreme je da primenimo ono što smo naučili na problem optimizacije iz stvarnog sveta.

### Prezentacija programa za optimizaciju

Zamislite da ste programer u velikoj svetskoj hotelskoj grupi. Vaša kompanija ima 30.000 hotela širom sveta. Od vas se traži da napravite program za izračunavanje globalnog prometa grupe. Svaki hotel šalje izveštaj centralnoj kancelariji na kraju svakog meseca. Izveštaj je JSON datoteka. Evo primera:

```go
{
    "name": "The Ultra Big Luxury Hotel",
    "reservations": [
      {
        "id": 0,
        "duration": 1,
        "rateId": 0
      },
      {
        "id": 1,
        "duration": 5,
        "rateId": 1
      }
    ],
    "rates": [
      {
        "id": 0,
        "price": 300
      },
      {
        "id": 1,
        "price": 244
      }
    ]
}
```

Ovo je izveštaj za "Ultra veliki luksuzni hotel". Hotel ima dve rezervacije. Svaka rezervacija ima interni ID; trajanje izraženo u noćenjima i ID tarife. Svaki hotel održava svoj cenovnik. Rezervacija broj 0 ima trajanje od 1 dana sa ID tarife 0, što znači 300 dolara po noćenju. Druga rezervacija ima trajanje od 5 dana za cenu jednaku 244 dolara po noćenju.

Izveštaji se agregiraju u veliku JSON datoteku. Moramo da razvijemo program koji će čitati JSON datoteku, a zatim izračunati ukupan promet cele grupe.

```go
// profiling/program-optimization/cmd/main.go 
//...

type Reporting struct {
    HotelReportings []HotelReporting
    ExchangeRates   []ExchangeRate
}

type HotelReporting struct {
    HotelId      int
    HotelName    string
    Reservations []Reservation
    Rates        []Rate
}

type ExchangeRate struct {
    RateUSD float64
    HotelId int
}
type Reservation struct {
    Id       uint
    Duration uint
    RateId   uint
}
type Rate struct {
    Id    uint
    Price uint
}
```

Prvo kreiramo našu strukturu tipa da bismo raspakovali JSON datoteku:

```go
data, err := ioutil.ReadFile("/Users/maximilienandile/go/src/go_book/profiling/programOptimization/cmd/output.json")
if err != nil {
    panic(err)
}
reportings := make([]Reporting, 0)
err = json.Unmarshal(data, &reportings)
if err != nil {
    panic(err)
}
```

Zatim učitavamo datoteku i raspakujemo JSON u reportings, što je deo vrednosti Reporting-a. Zatim ćemo implementirati glavnu logiku:

```go
var groupTurnover float64
for _, hotelReport := range r.HotelReportings {
    turnover, err := getTurnover(hotelReport,r.ExchangeRates)
    if err != nil {
        panic(err)
    }
    groupTurnover += turnover
}
fmt.Printf("Group Turnover %f\n", groupTurnover)
```

Ponavljamo izveštaje, a zatim ćemo za svaki izveštaj pozvati funkciju "getTurnover" koja će izračunati promet za određeni hotelski izveštaj.

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate) (float64, error) {
    turnoverPerReservation := []float64{}
    for _, reservation := range reporting.Reservations {
        ratePerNight, err := getRatePerNight(reporting.Rates, reservation.RateId)
        if err != nil {
            panic(err)
        }
        xr,err := getExchangeRate(reporting.HotelId,exchangeRates)
        if err != nil {
            panic(err)
        }
        turnoverResaUSD := float64(ratePerNight*reservation.Duration)*xr

        turnoverPerReservation = append(turnoverPerReservation, turnoverResaUSD)

    }
    return computeTotalTurnover(turnoverPerReservation), nil
}

func computeTotalTurnover(turnoverPerReservation []float64) float64 {
    var totalTurnover float64
    for _, t := range turnoverPerReservation {
        totalTurnover += t
    }
    return totalTurnover
}

func getRatePerNight(rates []Rate, rateId uint) (uint, error) {
    var found bool
    var price uint
    for _, rate := range rates {
        if rate.Id == rateId {
            found = true
            price = rate.Price
        }
    }
    if found {
        return price, nil
    } else {
        return 0, errors.New("Impossible to retrieve rate per night")
    }
}

func getExchangeRate(hotelId int, exchangeRates []ExchangeRate) (float64, error) {
    var found bool
    var rate float64
    for _, xr := range exchangeRates {
        if xr.HotelId == hotelId {
            found = true
            rate = xr.RateUSD
        }
    }
    if found {
        return rate, nil
    } else {
        return 0, errors.New("Impossible to retrieve exchange rate")
    }
}
```

U prethodnom spisku imamo tri glavne funkcije koje će izračunati promet:

- "getTurnover" funkcija će kreirati deo koji sadrži promet za svaku rezervaciju hotela. Da bi
  dobila ovaj promet, funkcija će morati da preuzme cenu po noćenju koju primenjuje hotel (pozvaće "getRatePerNight") i kurs valute hotela da bi konvertovala iznos u američke dolare ( "getExchangeRate").

- Zatim će pomnožiti broj noćenja (trajanje) sa cenom da bi dobio iznos novca generisan
  rezervacijom. Ovaj iznos je u lokalnom novcu; program će ga pomnožiti sa kursom američkih dolara.

- Sledeći korak je sumiranje iznosa koji se nalaze u turnoverPerReservation isečku. Da bismo to
  uradili, pozivamo computeTotalTurnover.

### Prva vožnja

Da bismo testirali naš program, generisali smo lažne podatke:

- 5.000 hotela
- 70 cena po hotelu
- 1.000 rezervacija po hotelu.

Hajde da ga izgradimo:

```sh
go build main.go
```

A onda ga možemo pokrenuti (pomoću uslužnog programa time)

```sh
time./main --rates 70 --hotels 5000 --resas 1000
real    0m27.659s
user    0m27.423s
sys     0m0.124s
```

Vreme izvršavanja programskog alata je 27.659 sekundi (uključujući vreme generisanja uzorka podataka)

### pprof analiza

Kada se program pokrene, profil je zapisan u gzip datoteku pod nazivom profile.pb.gz. Možemo pokrenuti pprof da bismo videli podatke profilisanja:

```sh
go tool pprof main profile.pb.gz
File: main
Type: cpu
Time: Jan 15, 2019 at 1:42pm (CET)
Duration: 27.18s, Total samples = 25.72s (94.63%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

Poslednja komanda će otvoriti interaktivni režim. Možete videti da prikupljeni uzorci pokrivaju 90,81% trajanja profila. Počećemo prikazivanjem 10 najvažnijih funkcija u vezi sa vremenom procesora.

#### Najviši komandni tim

```sh
(pprof) top
Showing nodes accounting for 25.58s, 99.46% of 25.72s total
Dropped 25 nodes (cum <= 0.13s)
Showing top 10 nodes out of 23
      flat  flat%   sum%        cum   cum%
    16.87s 65.59% 65.59%     16.87s 65.59%  main.getExchangeRate
     6.15s 23.91% 89.50%      6.15s 23.91%  runtime.memclrNoHeapPointers
     2.03s  7.89% 97.40%      2.03s  7.89%  runtime.nanotime
     0.24s  0.93% 98.33%      0.24s  0.93%  main.getRatePerNight
     0.21s  0.82% 99.14%      0.21s  0.82%  runtime.(*mspan).init (inline)
     0.07s  0.27% 99.42%     23.62s 91.84%  main.getTurnover
     0.01s 0.039% 99.46%      6.44s 25.04%  runtime.growslice
         0     0% 99.46%     23.63s 91.87%  main.main
         0     0% 99.46%      6.42s 24.96%  runtime.(*mcache).nextFree
         0     0% 99.46%      6.42s 24.96%  runtime.(*mcache).nextFree.func1
```

Na slici možete videti objašnjenja o izlazu komande. Imajte na umu značenje reči cum i flat.

- **flat** - vreme funkcije odgovara zbiru procesorskog vremena koje je potrošeno tokom
  izvršavanja funkcije tokom profilisanja programa.
  
- Cum označava kumulativno. Ovo je zbir vremena procesora koje je potrebno za izvršavanje
  funkcije, a takođe i vreme koje funkcija potroši kada čeka da se druga funkcija vrati.

Uzmimo primer. Možemo prikazati sve tragove koji sadrže funkciju "main.getTurnover" sledećom komandom (koristeći interaktivni režim pprof):

```sh
(pprof) traces focus main.getTurnover
```

U sledećem odeljku možete videti izvod 3 prikazane tragove.

```sh
-----------+-------------------------------------------------------
     1.58s   runtime.memmove
             runtime.growslice
             main.getTurnover
             main.main
             runtime.main
-----------+-------------------------------------------------------
      70ms   runtime.memclrNoHeapPointers
             runtime.growslice
             main.getTurnover
             main.main
             runtime.main
-----------+-------------------------------------------------------
      20ms   main.getTurnover
             main.main
             runtime.main
```

Prvi trag iznosi 1,58 sekundi.

Funkcija "main.getTurnover" je na trećoj poziciji (počevši od početka steka poziva). Ovaj trag pokazuje da "main.main" je funkcija "main()" pozvala funkciju "main.getTurnover" koja je pozvala funkciju "runtime.growslice".… U ovom primeru, funkcija "getTurnover" čeka da se funkcija "runtime.grow" vrati (koja čeka da se "runtime.memove" vrati). Ovo možemo dodati kumulativnom trajanju.

Za drugi stek poziva, funkcija "getTurnover" je takođe deo steka poziva, takođe na trećoj poziciji. Moramo dodati trajanje kumulativnom trajanju.

Za treći stek poziva imamo našu funkciju na vrhu steka. Možemo ovo dodati trajanju ravnog broja. Hajde da uradimo malo matematike:

Ako dodamo samo te stekove poziva da bismo generisali naš profil

- ravno vreme funkcije "main.getTurnover" bi bilo jednako 20ms
- dok bi kumulativno vreme "main.getTurnover" bilo jednako:

  1,58s+0,07s=1,65s

Ako analiziramo sve tragove profila, možemo doći do istih zapažanja.

### Prva runda optimizacije: izlaz iz petlje

U ovoj prvoj rundi fokusiraćemo se na funkciju main.getExchangeRatekoja zbirno sabira 16,87 sekundi zabeleženog CPU vremena profila. To predstavlja 65,59% kumuliranog CPU vremena. Moramo poboljšati ovu funkciju:

```go
func getExchangeRate(hotelId int, exchangeRates []ExchangeRate) (float64, error) {
    var found bool
    var rate float64
    for _, xr := range exchangeRates {
        if xr.HotelId == hotelId {
            found = true
            rate = xr.RateUSD
        }
    }
    if found {
        return rate, nil
    } else {
        return 0, errors.New("Impossible to retrieve exchange rate")
    }
}
```

- Ova funkcija će iterirati kroz segment pod nazivom exchangeRateskoji sadrži sve kurseve valuta
  za svaki hotel.
- Svaki hotel ima kurs. U našem uzorku postoji 5.000 hotela. Svaki put kada se ova funkcija
  pozove, petlja for će iterativno preći preko 5.000 kurseva da bi pronašla pravi.
- Zamislite da tražimo devizni kurs za hotel sa ID-om 42. Ako su elementi segmenta poređani po
  hotelId-u, u 42. iteraciji smo završili, devizni kurs je pronađen.
- Ali naša funkcija će nastaviti da iterira kroz isečak. To je 5.000 - 42 = 4958 beskorisnih
  iteracija.

Mogli bismo da zaustavimo iteraciju kada pronađemo naš devizni kurs:

```go
if xr.HotelId == hotelId {
    found = true
    rate = xr.RateUSD
    break
}
```

Ovde ključna reč break izlazi iz petlje for i nastavlja izvršavanje od zatvarajuće zagrade funkcije for.

Hajde ponovo da pokrenemo profil da vidimo da li je poboljšao naše performanse.

### Vremenska statistika

real    0m7.690s
user    0m7.498s
sys     0m0.085s

Impresivno! Sa ovom jednostavnom naredbom break, postigli smo 20sekundi!

### Profil procesora

Da vidimo efekat na statistiku profila:

Showing nodes accounting for 6.70s, 99.41% of 6.74s total
Dropped 5 nodes (cum <= 0.03s)
Showing top 10 nodes out of 25
      flat  flat%   sum%        cum   cum%
     4.29s 63.65% 63.65%      4.29s 63.65%  main.getExchangeRate
     1.48s 21.96% 85.61%      1.48s 21.96%  runtime.memmove
     0.57s  8.46% 94.07%      0.57s  8.46%  runtime.nanotime
     0.18s  2.67% 96.74%      0.18s  2.67%  main.getRatePerNight
    0.10s  1.48% 98.22%      0.10s  1.48%  runtime.(*mspan).init (inline)
     0.06s  0.89% 99.11%      0.06s  0.89%  runtime.memclrNoHeapPointers
     0.02s   0.3% 99.41%      6.15s 91.25%  main.getTurnover
         0     0% 99.41%      6.15s 91.25%  main.main
         0     0% 99.41%      0.18s  2.67%  runtime.(*mcache).nextFree
         0     0% 99.41%      0.18s  2.67%  runtime.(*mcache).nextFree.func1

Funkcija "main.getExchangeRate" je i dalje na vrhu CPU profila. NJeno vreme izvršavanja je sada samo 4,29 sekundi, u poređenju sa 16,87 sekundi u prethodnoj verziji; ovo je poboljšanje!

### Druga runda optimizacije: invarijante petlji (V2)

Prethodni profil procesora pokazuje da main.getExchangeRateje funkcija i dalje jedna od funkcija koje najviše troše procesorsko vreme. Da vidimo u kom kontekstu se koristi:

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate) (float64, error) {
    turnoverPerReservation := []float64{}
    for _, reservation := range reporting.Reservations {
        //..
        xr,err := getExchangeRate(reporting.HotelId,exchangeRates)
        //...
    }
    //...
}
```

- Možete videti da će funkcija getTurnoverpreuzeti promet za određeni hotel.
- Preći će preko svih hotelskih rezervacija.
- I za svaku rezervaciju hotela, funkcija će preuzeti kurs hotela. To nije baš optimalno jer se
  kurs ne menja. Stabilan je za svaku rezervaciju jer je definisan za svaki hotel.

U ovom isečku koda imamo savršen primer invarijante petlje. Promenljiva xrće uvek biti ista, ali je stalno preuzimamo za svaku iteraciju petlje.

Možemo jednostavno izdvojiti poziv funkcije getExchangeRateiz petlje for. Postavićemo ga pre početka petlje:

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate) (float64, error) {
    turnoverPerReservation := []float64{}
    xr,err := getExchangeRate(reporting.HotelId,exchangeRates)
    for _, reservation := range reporting.Reservations {
        //..
        //...
    }
    //...
}
```

Da vidimo uticaj ove jednostavne modifikacije na performanse našeg programa.

```go
go build main
time./main --rates 70 --hotels 20000 --resas 1000
```

Rezultat je impresivan:

```sh
real    0m1.090s
user    0m0.847s
sys     0m0.072s
```

Programu je potrebno ne 1.090 sekundi za izvršavanje! Da vidimo rezultate profilisanja:

```sh
$ go tool pprof main profile_1547640425.pb.gz
File: main
Type: cpu
Time: Jan 16, 2019 at 1:07pm (CET)
Duration: 614.36ms, Total samples = 400ms (65.11%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

Prvo, primećujemo da uzorci čine samo 400ms (približno četiri uzorka). Ovo je sasvim normalno jer naš program ima vreme izvršavanja od samo jedne sekunde. Profiler je izvukao samo četiri uzorka, što čini 65,11% trajanja profila.

Želimo to da poboljšamo. Rešenje je generisanje više uzoraka podataka. Pomnožićemo broj hotela sa 4 (sada 20.000) da bismo povećali broj uzoraka. Imam jednostavan skript koji generiše lažne podatke; namerno ga krijem da biste se vi koncentrisali samo na optimizaciju koda.

Ponovo gradimo program i pokrećemo ga da bismo dobili profil.

```sh
$ go tool pprof main profile_1547641211.pb.gz
File: main
Type: cpu
Time: Jan 16, 2019 at 1:20pm (CET)
Duration: 2.06s, Total samples = 1.86s (90.28%)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof)
```

Uzorci sada čine 90,28% trajanja profila! Zbir vremena uzorkovanja je sada 1,86 sekundi. Broj uzoraka je sada 24! Podaci našeg profila su tačniji. Pogledajmo pogled odozgo:

```sh
(pprof) top
Showing nodes accounting for 1.83s, 98.39% of 1.86s total
Showing top 10 nodes out of 45
      flat  flat%   sum%        cum   cum%
     1.48s 79.57% 79.57%      1.48s 79.57%  runtime.memmove
     0.12s  6.45% 86.02%      0.12s  6.45%  main.getRatePerNight
     0.06s  3.23% 89.25%      0.06s  3.23%  runtime.(*mspan).refillAllocCache
     0.05s  2.69% 91.94%      0.05s  2.69%  runtime.memclrNoHeapPointers
     0.04s  2.15% 94.09%      0.04s  2.15%  runtime.(*mspan).init (inline)
     0.02s  1.08% 95.16%      0.02s  1.08%  main.getExchangeRate
     0.02s  1.08% 96.24%      1.81s 97.31%  main.getTurnover
     0.02s  1.08% 97.31%      0.02s  1.08%  runtime.mmap
     0.01s  0.54% 97.85%      0.01s  0.54%  main.computeTotalTurnover
     0.01s  0.54% 98.39%      0.01s  0.54%  runtime.findrunnable
```

### Treća runda optimizacije: mape (V3)

U prethodnom prikazu odozgo, možemo primetiti da je funkcija main.getRatePerNightdrugi najveći potrošač procesorskog vremena.

Hajde da analiziramo ovu funkciju:

```go
func getRatePerNight(rates []Rate, rateId uint) (uint, error) {
    var found bool
    var price uint
    for _, rate := range rates {
        if rate.Id == rateId {
            found = true
            price = rate.Price
        }
    }
    if found {
        return price, nil
    } else {
        return 0, errors.New("Impossible to retrieve rate per night")
    }
}
```

Ovde tražimo cenu noći.

- Za svaki hotel, cene se prenose u izveštaju.
- Svaka cena je dobila id, a svaka rezervacija ima rateId, što nam omogućava da pronađemo cenu
  primenjenu na rezervaciju.
- Funkcija se sastoji od for petlje koja će iterativno pregledati sve hotelske cene da bi pronašla
  pravu.
- Ovde imamo prvu optimizaciju koju možemo da napravimo; moramo da zaustavimo petlju kada
  pronađemo brzinu. (videti odeljak.

- Samo dodajemo naredbu za prekid:

  ```go
  if rate.Id == rateId {
      found = true
      price = rate.Price
      break
  }
  ```

Sa ovim poboljšanjem, programu je sada potrebno 3,088 sekundi za izvršavanje (u poređenju sa 3,097 sekundi ranije).

Takođe možemo poboljšati pretragu.

Označimo sa n broj cena za hotel. U najgorem slučaju, moramo izvršiti n iteracija da bismo pronašli cenu.

Možemo koristiti mapu da poboljšamo vreme pretrage. To će zahtevati važnu modifikaciju koda. Prvo, potrebno je da kreiramo funkciju za generisanje mape cena.

```sh
identifikacioni broj​​​​→cena​​​​
ID rate→cena
```

Mapa će povezati rateIdsa cenom:

```go
func RateMap(reporting HotelReporting)map[uint]uint{
    m := make(map[uint]uint)
    for _,rate := range reporting.Rates{
        m[rate.Id] = rate.Price
    }
    return m
}
```

Prethodna funkcija će kao argument uzeti an HotelReportingi vratiti a map(ključevi i vrednosti su neoznačeni celi brojevi). Ponavljaćemo stope izveštavanja o hotelima i progresivno puniti mapu.,

Zatim moramo da izmenimo funkciju getRatePerNight. Sada će pretraživati mapu da bi dobila cenu po noćenju:

```go
func getRatePerNight(rateMap map[uint]uint, rateId uint) (uint, error) {
    price, found := rateMap[rateId]
    if found {
        return price, nil
    } else {
        return 0, errors.New("Impossible to retrieve rate per night")
    }
}
```

Sledeća modifikacija se sastoji u pozivanju ovog generatora mapa unutar naše glavne funkcije. Za svaki izveštaj o hotelu, napravićemo našu mapu, a zatim je proslediti funkciji getTurnover:

```go
func main(){
    //...
    for _, hotelReport := range r.HotelReportings {
        rateMap := RateMap(hotelReport)
        turnover, err := getTurnover(hotelReport,r.ExchangeRates,rateMap)
        if err != nil {
            panic(err)
        }
        groupTurnover += turnover
    }
    fmt.Printf("Group Turnover %f\n", groupTurnover)
}
```

Uzgred, funkciju getTurnoverje takođe potrebno izmeniti. Mora da prihvati treći parametar: mapu brzina:

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate, rateMap map[uint]uint) (float64, error) {
    //...
    for _, reservation := range reporting.Reservations {
        ratePerNight, err := getRatePerNight(rateMap, reservation.RateId)
        //...
    }
    //...
}
```

Završili smo. Sada možemo da kompajliramo, profilišemo i merimo vreme izvršavanja:

```sh
$ go build main.go
$ time./main --rates 70 --hotels 20000 --resas 1000
real    0m3.242s
user    0m2.774s
sys     0m0.321s
```

To nije baš očekivano. Ukupno vreme izvršavanja je veće nego kod prethodne verzije! Pomnožili smo ga sa 3. Šta nije u redu? Ovaj pad performansi se objašnjava činjenicom da kreiranje mape generiše dodatno opterećenje (okruženje za izvršavanje mora da izračuna heševe i da ih sačuva).

Uticaj dodavanja hešmapa na vreme izvršavanja

| Broj koraka | 70 koraka | 70 koraka | 600 koraka | 600 koraka | 10000 koraka |
| ---------- | -------- | --------- | ----------- | ------ | ----- |
| Verzija programa | v2 | v3 (mapa) | v2 | v3 (mapa) | v3 (mapa) |
| Ukupno vreme izvršenja | 1.090s | 3.242s | 7,698s | 4.209s | 26,628s |
| Vreme korisnika procesora | 0,847s | 2,774s | 7,493s | 3,705s | 23.218s |

U našem uzorku podataka imamo samo 70 koraka. Hajde da ponovo testiramo naš program sa 600 koraka. U tabeli možete videti da se relativne performanse v3 povećavaju kada se broj stopa povećava.

Hajde da pogledamo najveće potrošače procesorskog vremena pomoću pprof-a (za 10.000):

Showing nodes accounting for 12.02s, 94.50% of 12.72s total

Dropped 51 nodes (cum <= 0.06s)

Showing top 10 nodes out of 36

| flat | flat% | sum% | cum | cum% | . |
| --- | ------ | ---- | --- | ---- | - |
| 4.51s | 35.46% | 35.46% | 9.71s | 76.34% | runtime.mapassign_fast64 |
| 4.19s | 32.94% | 68.40% | 4.24s | 33.33% | runtime.(*hmap).newoverflow |
| 1.53s | 12.03% | 80.42% | 1.53s | 12.03% | runtime.memclrNoHeapPointers |
| 0.57s | 4.48% | 84.91% | 0.57s | 4.48% | runtime.aeshash64 |
| 0.54s | 4.25% | 89.15% | 11.86s | 93.24% | main.RateMap |
| 0.21s | 1.65% | 90.80% | 0.26s | 2.04% | runtime.mapaccess2_fast64 |
| 0.13s | 1.02% | 91.82% | 0.62s | 4.87% | main.getTurnover |
| 0.13s | 1.02% | 92.85% | 0.13s | 1.02% | runtime.nanotime |
| 0.11s | 0.86% | 93.71% | 0.14s | 1.10% | runtime.overLoadFactor (inline) |
| 0.10s | 0.79% | 94.50% | 0.10s | 0.79% | main.getExchangeRate |

U prikazu odozgo na pprof-u možete videti da funkcija "runtime.mapassign_fast642 zauzima veliki deo ukupnog vremena procesora (76,34% kumulativnog vremena). To znači da je korak koji odgovara popunjavanju mape parovima ključ-vrednost intenzivan za procesor, ali smo ipak uštedeli dosta vremena procesora njihovom implementacijom.

### Četvrta runda optimizacije: pojednostavljivanje koda

Ako pažljivo pogledate kod našeg programa, videćete da imamo beskorisnu funkciju: "computeTotalTurnover". Ova funkcija uzima isečak `float64` i vraća zbir elemenata koji se nalaze u tom delu:

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate, rateMap map[uint]uint) (float64, error) {

    turnoverPerReservation := []float64{}
    // is this slice necessary
    //... for loop
    return computeTotalTurnover(turnoverPerReservation), nil
    // this function call is useless
}
```

Ne treba nam funkcija da bismo to uradili! Funkcija computeTotalTurnoverse može ažurirati na sledeću verziju:

```go
func getTurnover(reporting HotelReporting, exchangeRates []ExchangeRate, rateMap map[uint]uint) (float64, error) {
    var totalTurnover float64
    xr,err := getExchangeRate(reporting.HotelId,exchangeRates)
    if err != nil {
        panic(err)
    }
    for _, reservation := range reporting.Reservations {
        ratePerNight, err := getRatePerNight(rateMap, reservation.RateId)
        if err != nil {
            panic(err)
        }
        totalTurnover += float64(ratePerNight*reservation.Duration)*xr
    }
    return totalTurnover, nil
}
```

Ovde počinjemo definisanjem promenljive totalTurnover(tipa float64) koja će sadržati promet za ceo hotelski izveštaj. Umesto da delimični promet za rezervaciju stavimo u deo, a na kraju saberemo deo, jednostavno dodajemo promet na.totalTurnover

Hajde da pokrenemo ovu verziju:

```sh
$ go build main.go
$ time./main --rates 600 --hotels 20000 --resas 1000 --profileName 600rates
real    4.14s
user    3.58s
sys     0.44s
```

Ovo malo poboljšanje ima mali (ali ne i zanemarljiv) uticaj na performanse. Verziji 3 programa je bilo potrebno 3,705 sekundi (vreme korisnika procesora), a novoj verziji je sada potrebno samo 3,58 sekundi.

### Rezime

Poboljšanje učinka (600 stopa, 20.000 hotela, 1.000 rezervacija)

| ... | v0 | v1 | v2 | v3 | v4 |
| --- | -- | -- | -- | -- | -- |
| Ukupno vreme izvršenja | 463,21s | 136,60s | 16,51s | 4,79s | 4,57s |
| Vreme korisnika procesora | 460,93s | 135,12s | 15,83s | 4,13s | 3,91 s |
| Sistemsko vreme procesora | 1,92s | 1,02s | 0,57s | 0,51s | 0,55s |

U tabeli možete videti da sam detaljno opisao vreme izvršavanja i vreme procesora svake verzije programa.

Naši napori su se isplatili jer je svaka runda optimizacije poboljšala vreme izvršavanja i vreme procesora. Od početne verzije do konačne verzije (koju još uvek možemo poboljšati), vreme korisnika procesora je smanjeno sa približno 115 puta.

## Profilisanje hipa

U "gornjoj" tabeli možemo videti da se većina vremena troši na funkciju `runtime.memmove`. Ako pogledamo izvorni kod Go runtime-a, ova funkcija se koristi za premeštanje bajtova memorije sa jedne lokacije na drugu. Takođe možete primetiti da se u gornjoj tabeli korišćenja procesora, druge funkcije runtime-a koje se bave memorijom zovu: `runtime.(*mspan).refillAllocCache` i `runtime.memclrNoHeapPointers`.

Možemo zaključiti da smo negde u našem programu napravili grešku sa memorijom... Još nismo uradili profilisanje memorije, ali možemo otkriti problem sa potrošnjom memorije.

Pogledajmo zaglavlja naših tri funkcija:

```go
func getTurnover(reporting HotelReporting) (uint, error)
func computeTotalTurnover(turnoverPerReservation []uint) uint
func getRatePerNight(rates []Rate, rateId uint) (uint, error)
```

Ovde prosleđujemo promenljive po vrednosti, a ne po referenci. Kopiramo podatke tokom izvršavanja programa. Jednostavna optimizacija je prosleđivanje promenljivih po referenci.

### Šta je profil hipa

Profil hipa omogućava programeru da precizno razume korišćenje memorije programa. Dakle, profil hipa će prikupljati podatke o tačkama alokacije i dealokacije u vašem programu. Profiler će prikupljati trag steka vašeg programa koji se pokreće i preciznu statistiku memorije određenom brzinom.

Usput, takođe možete prikupiti trenutnu statistiku memorije u bilo kom delu vašeg programa pozivanjem:

```go
memStats = new(runtime.MemStats)
runtime.ReadMemStats(memStats)
fmt.Println("cumulative count of heap objects allocated: %d",memStats.Mallocs)
//...
```

### Preduslov: sačuvajte profil memorije

Moraćemo da promenimo kod naše aplikacije kako bismo beležili potrošnju memorije. Dodaćemo sledeće linije da bismo zatražili od paketa pprof da sačuva profil memorije naše aplikacije:

```go
f, err := os.Create("mem_profile.pb.gz")
if err != nil {
    log.Fatal(err)
}
runtime.GC() // get up-to-date statistics
if err := pprof.WriteHeapProfile(f); err != nil {
    log.Fatal("memory profile cannot be gathered", err)
}
defer f.Close()
```

Prvo što treba da uradimo je da kreiramo datoteku koja će sadržati podatke našeg profila. Koristićemo pogodan metod `os.Create`.

Zatim ćemo zatražiti od izvršnog okruženja da pokrene sakupljanje smeća (ovo će blokirati izvršavanje dok se sakupljanje ne završi). Zašto? Zato što želimo da naša statistika memorije bude preciznija. Sakupljač smeća će se rešiti memorije koja je prethodno dodeljena i trenutno se ne koristi. Stoga se ova neiskorišćena memorija neće pojaviti u našoj statistici.

Zatim pokrećemo funkciju WriteHeapProfilekoja će upisati profil u datu datoteku.

### Sakupljač smeća i njegov uticaj na profilisanje memorije

Profil memorije koji generiše paket `pprof` će zabeležiti alokacije memorije uživo "od poslednjeg završenog sakupljanja smeća".

Ako je izvršno okruženje pokrenulo proces sakupljanja smeća u programu tri puta, podaci prijavljeni u profil bi se odnosili na alokacije memorije između drugog i trećeg pokretanja GC-a.

Ako nema sakupljanja smeća tokom programa, profil će sadržati podatke o svim alokacijama.

Da biste onemogućili GC, možete dodati ovu liniju u svoj go program:

```go
debug.SetGCPercent(-1)
```

### Stopa profilisanja

Kao i profilisanje procesora, profilisanje memorije se dešava određenom brzinom.

### Naš prvi profil memorije

Hajde da napravimo naš modifikovani program i pokrenemo ga:

```go
go build main.go
./main --rates 600 --hotels 20000 --resas 1000
```

Naš memorijski profil je zapisan na disk. Sada možemo analizirati naš profil pomoću komandne linije pprof:

```go
go tool pprof main mem_profile.pb.gz
File: main
Type: inuse_space
Time: Jan 21, 2019 at 8:49pm (CET)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 791.52MB, 100% of 791.52MB total
      flat  flat%   sum%        cum   cum%
  791.52MB   100%   100%   791.52MB   100%  main.data
         0     0%   100%   791.52MB   100%  main.main
         0     0%   100%   791.52MB   100%  runtime.main
```

Prvo, možemo primetiti da su profilisane alokacije od 791,52 MB.

Unošenjem komande `top`, vidimo prvih 10 čvorova sa najvećom veličinom u našem profilu. Ovde imamo samo tri čvora! To je jedini čvor našeg profila (to objašnjava 100%).

Tada možemo primetiti da je tip `profila.inuse_space`

### Različite vrste statistike memorije

Podrazumevano inuse_spacese prikazuje tip statistike, ali to nije jedini tip dostupnih indikatora memorije:

Memorijski prostor koji je dodeljen, ali nije oslobođen. Ovo je memorija koja se koristi u trenutku uzorkovanja.

Da biste dobili profil u tom režimu, možete otkucati sledeću komandu:

```sh
go tool pprof -sample_index=inuse_space main profile.pb.gz
```

**alloc_space** - Memorijski prostor koji je trenutno dodeljen + memorija koja je oslobođena.

```sh
go tool pprof -sample_index=alloc_space main profile.pb.gz
```

**inuse_objects** - Prikazaće broj objekata koji su dodeljeni, ali nisu oslobođeni.

```sh
go tool pprof -sample_index=inuse_objects main profile.pb.gz
```

**alloc_objects** - Ovo je brojač objekata koji su dodeljeni + svi objekti koji su oslobođeni.

```sh
go tool pprof -sample_index=alloc_space main profile.pb.gz
```

Morate imati na umu da se sve te statistike čuvaju u istom profilu. Ne postoje četiri vrste memorijskih profila, već samo jedan, koji se naziva profil hipa.

### Upotreba različitih vrsta statistike

Svaka vrsta statistike će prikupljati različite vrste informacija:

- Statistika upotrebe će dati pregled memorije koja se koristi u određenom trenutku tokom
  izvršavanja programa.

- Statistika alokacije je korisna da se vidi koji deo programa alokuje najviše memorije (jer
  takođe uzima u obzir memoriju koju je oslobodio sakupljač smeća).

### Glavna mesta alokacije memorije

Ono što možemo videti u prvom profilu koji smo generisali jeste da se memorija koristi 100% u funkciji main.data. Ova funkcija je napravljena da generiše podatke koje koristimo u našem programu.

To nije iznenađenje. Da vidimo statistiku alloc_space.

```sh
go tool pprof -sample_index=alloc_space main profile.pb.gz
File: main
Type: alloc_space
Time: Jan 21, 2019 at 9:10pm (CET)
Entering interactive mode (type "help" for commands, "o" for options)
(pprof) top
Showing nodes accounting for 1.53GB, 100% of 1.53GB total
      flat  flat%   sum%        cum   cum%
    1.53GB   100%   100%     1.53GB   100%  main.data
         0     0%   100%     1.53GB   100%  main.main
         0     0%   100%     1.53GB   100%  runtime.main
```

Ovaj profil daje informacije o glavnim delovima našeg programa koji alokuju memoriju.

### Istražite korišćenje memorije pomoću interaktivnog prikaza

Da biste istražili svoj memorijski profil, koristite sledeću komandu:

```sh
go tool pprof -http localhost:9898 profile.pb.gz
```

## Završna napomena

- Pprof je odličan alat za razumevanje kako vaši programi funkcionišu i kako se izvode.
- Pomoću profilisanja možete otkriti probleme sa performansama.
- Međutim, jasan i razumljiv program je uvek lakše održavati nego nejasan ultra-efikasan program.

## Testirajte sebe

1. Kako se zove paket koji generiše profile?
   - "vreme izvršavanja/pprof"

2. Šta su protokolski baferi?
   - Ovo je metoda koja se koristi za serijalizaciju strukturiranih podataka.
   - Serijalizovani podaci su manji.

3. Koji program možete koristiti za dekodiranje poruke serijalizovane pomoću protokolskih bafera?
   - protokol
   - Možete ga preuzeti ovde <https://github.com/protocolbuffers/protobuf/releases>

4. Šta je "stek poziva"?
   - Stek poziva programa je lista trenutno izvršavanih funkcija.
   - Na početku steka, naći ćete funkciju mainiz paketa main (ulazna tačka programa).

5. U izlazu UNIX `time` komande, koja je razlika između vremena "user" i "sys"?
   - user: Ovo vreme odgovara vremenu koje je procesor (CPU) bio zauzet izvršavanjem instrukcija
     van jezgra (korisničkog prostora).
   - sys: Ova vremenska mera odgovara vremenu koje je potrebno procesoru da izvrši komande u
     prostoru jezgra, na primer, sistemske pozive.

6. Navedite jednu tehniku optimizacije za smanjenje potrošnje procesora.
   - Na primer, uklanjanje mrtvog koda (pogledajte odeljak "Ključni zaključci" za skraćenu listu)

## Ključno

- Profilisanje = prikupljanje detaljnih statistika o tome kako program radi
- Profilisanje se razlikuje od benčmarkova. Benčmark meri performanse jedne funkcije.
- Da biste kreirali profil procesora, dodajte ove linije u svoju glavnu funkciju:

  ```go
  f, err := os.Create("profile.pb.gz")
  if err != nil {
      log.Fatal(err)
  }
  pprof.StartCPUProfile(f)
  defer pprof.StopCPUProfile()
  ```

- "Protokolski baferi" je metoda koja se koristi za serijalizaciju strukturiranih podataka.
- Može transformisati strukturirane podatke u lagan format koji se može čuvati i prenositi preko
  mreže.
- Datoteka profila koju generiše go se serijalizuje pomoću Protocol Buffers-a.
- Stek poziva je uređena lista aktivnih funkcija u programu. Stek poziva može da raste ili da se
  smanjuje u zavisnosti od programa.
- Vreme procesora predstavlja vreme koje centralna procesorska jedinica (CPU) koristi za
  izvršavanje skupa instrukcija definisanih u vašem programu.
- Vreme izvršavanja programa može se meriti komandom timena UNIX sistemima
- Da biste prikazali profil u vašem veb pregledaču, možete koristiti sledeću komandu

  ```sh
  go tool pprof -web yourBinary profile.pb
  ```

- Uzorak je merenje. Ovo merenje se vrši u određenom trenutku tokom procesa profilisanja.
- Neke uobičajene tehnike optimizacije korišćenja procesora su:
  - Uklanjanje mrtvog koda
  - Optimizacija izlaska iz petlje: izađite iz petlje što je pre moguće
  - Uklanjanje koda nepromenljivog u petlji: uklonite sve instrukcije koje ne zavise od lokalnih
    promenljivih petlje.
  - Spajanje petlji: ako imate dve petlje koje ponavljaju istu kolekciju, možda ćete želeti da ih
    spojite.
  - Kada rezultat operacije ne varira, možda ćete želeti da kreirate konstantu.
- Da biste kreirali profil memorije, koristite sledeći kod:

  ```go
  f, err := os.Create("mem_profile.pb.gz")
  if err != nil {
      log.Fatal(err)
  }
  if err := pprof.WriteHeapProfile(f); err != nil {
      log.Fatal("memory profiling is cannot be gathered", err)
  }
  defer f.Close()
  ```

- Da biste pokrenuli interaktivni veb interfejs pprof, koristite ovu komandu:
  
  ```sh
  go tool pprof -http localhost:9898 profile.pb.gz
  ```

[35 Izgradnja HTTP klijenta][35] | [00 Sadržaj][00] | [37 Kontekst][37]

[35]: 35_Izgradnja_HTTP_klijenta.md
[00]: 00_Sadržaj.md
[37]: 37_Kontekst.md
