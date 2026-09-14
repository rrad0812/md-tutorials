
# 04 Podesite svoje razvojno okruženje

[03 Terminal][03] | [00 Sadržaj][00] | [05 Prva Go aplikacija][05]

**Šta ćete naučiti u ovom poglavlju?**

- Kako instalirati Go na vaš računar.
- Kako izračunati kontrolnu sumu (SHA 256).

**Obrađeni tehnički koncepti!**

- Arhitektura računara
- Heš funkcija
- Kompresija / Dekompresija
- Profil školjke
- Putanja

## Uvod

Da biste pisali Go programe, potreban vam je editor. Da biste testirali, dokumentovali, kompajlirali i formatirali svoje Go programe, moraćete da instalirate binarne datoteke Go alata.

Ovo poglavlje će vas voditi kroz proces instalacije.

## Arhitektura računara

U sledećim odeljcima, moraćete da znate arhitekturu vašeg računara. Ovaj tehnički termin ponekad donosi zabunu početnicima. Šta je to tačno?

Računari dolaze u različitim oblicima, cenama i modelima. Mnogi potrošači se fokusiraju na brend računara i njegove glavne specifikacije (kao što su RAM memorija, količina memorije na čvrstom disku). Marketing ponekad zaobilazi pitanje procesora, ali je on fundamentalni deo računara. Procesor se naziva i CPU ( Central Processing Unit - centralna procesorska jedinica ).

Procesor je odgovoran za pokretanje sistemskih instrukcija, ali i programa. Procesor je opšti pojam. Postoje različite vrste procesora.

Programi koriste skup unapred definisanih instrukcija da bi komunicirali sa mašinom. Ovu listu naredbi nazivamo skupom instrukcija. Neki procesori dele iste skupove instrukcija. Drugi imaju potpuno različite. Pojam arhitekture obuhvata skup instrukcija i fizičku organizaciju, kao i implementaciju računara.

Sada razumete zašto je ovaj pojam važan na mašinskom nivou. Go toolchain je kolekcija programa koje ćemo koristiti za izgradnju naših programa.

- Ti programi su napisani u programskom jeziku Go i kompajlirani.
- Go tim pruža različite verzije alata za svaki podržani operativni sistem i arhitekturu.
  - Go ima minimalne sistemske i OS zahteve.
  - Možete ih proveriti na ovoj veb stranici: <https://github.com/golang/go/wiki/MinimumRequiriments>.  Stranica se često ažurira.

## Instalacija Go toolchain-a

Da biste započeli razvoj Go programa, prvo morate da instalirate Go na svoj računar. Da biste ga instalirali, moraćete da preuzmete datoteku. Postupak instalacije se malo razlikuje u zavisnosti od vašeg operativnog sistema. Samo pratite korake koji odgovaraju vašem operativnom sistemu.

Prvi korak (koji ne zavisi od vašeg operativnog sistema) je da odete na zvaničnu stranicu za preuzimanje: <https://golang.org/dl/>. Inače, `golang.org` je zvanična Go veb stranica. Toplo vam savetujemo da preuzimate Go samo sa ove URL adrese.

U ovom odeljku ćemo obraditi Linux, macOS i Windows. Imajte na umu da je go dostupan i za FreeBSD.

### Linux

#### Odredite svoju arhitekturu

Prvo morate da odredite arhitekturu svog računara. To će vam omogućiti da izaberete pravu datoteku za preuzimanje.

Otvorite terminal i otkucajte sledeću komandu:

```sh
uname -p
```

Uname je program koji prikazuje karakteristike vašeg sistema. Zastavica -p će ispisati naziv arhitekture procesora mašine.

Verzije Golang Linux-a su dostupne za sledeće arhitekture:

- x86
- x86-64
- ARMv6
- ARMv8
- ppc64le
- s390x

#### Preuzmite arhivu

Kada znate svoju arhitekturu, sada možete preuzeti odgovarajuću kompresovanu fasciklu sa Go veb stranice.

Samo kliknite na plavi link i preuzimanje će početi.

U narednim odeljcima, instaliraćemo verziju 1.12.8 za arhitekturu x86-64. Uputstva su ista za ostale arhitekture i ostale verzije. Samo će se ime arhive promeniti.

#### Proverite heš arhive

Ovaj korak se toplo preporučuje. Cilj je proveriti da li imate oštećenu verziju arhive. Da biste to uradili, otvorite terminal i otkucajte:

```sh
cd /where/you/have/downloaded/the/file
sha256sum go1.12.8.linux-amd64.tar.gz
```

Prva komanda koristi cd (promena direktorijuma) da bi se otišlo do direktorijuma gde se nalazi datoteka koju ste preuzeli. Na primer, ako se datoteka nalazi u /home/max/, potrebno je da izvršite:

```sh
cd /home/max
```

Sledeća komanda će izračunati SHA256 heš (što je kriptografska heš funkcija) datoteke go1.12.8.linux-amd64.tar.gz.

**Kriptografska heš funkcija**: Funkcija koja će kao ulaz prihvatiti podatke promenljive veličine (datoteku, reč, rečenicu) i izbaciti blob podataka fiksne veličine. Imajući izlaz, gotovo je nemoguće vratiti ulaz funkcije. Rezultat se naziva „heš“ ili "sažetak poruke". Ovde ga koristimo da bismo se uverili da datoteka nije izmenjena na putu do našeg računara. Da bismo to uradili, upoređujemo heš koji smo izračunali na našem računaru sa hešem koji je dat na Go veb stranici. Ako su oba jednaka, nije bilo izmene.

Kada se izvrši sha256sum, na ekranu ćete videti skup znakova koje morate uporediti sa hešom prikazanim na veb lokaciji. Ako nisu identični, nešto je pošlo naopako, nemojte koristiti preuzetu datoteku. Ako su heš koji ste izračunali i heš prikazan na Go veb lokaciji jednaki, spremni ste.

#### Dekompresujte arhivu

Arhiva je gzipovana (komprimovana pomoću gzip-a, često se naziva tarball ). Da biste je deflatovali, imate dve opcije:

- Koristite grafički interfejs (ako ga imate): dvaput kliknite na arhivu i otvoriće se prozor. Pratite uputstva. Morate raspakovati i deflatovati datoteke u `/usr/local`
- Koristite terminal.

Radićemo opciju 2.

Otvorite terminal i unesite komandu:

```sh
sudo tar -C /usr/local -xzf go1.12.8.linux-amd64.tar.gz
```

Koristićemo aplikaciju tar (u sudo režimu)

- `-C /usr/local` : znači da ćemo promeniti direktorijum za izvršavanje u /usr/local

- `-xzf go1.12.8.linux-amd64.tar.gz`: znači da želimo da raspakujemo (x) arhivu go1.12.8.linux-amd64.tar.gz koja je kompresovana pomoću gzip-a.

#### Podesite svoju PUTANJU

Hajde da pogledamo prikaz stabla fascikle `/usr/local/go`.

Imamo osam foldera: API, bin, doc, lib, misc, pkg, src i tests.

Folder bin sadrži tri izvršna fajla:

- go : ovo je glavni izvršni fajl
- godoc : ovo je program koji se koristi za generisanje dokumentacije 1
- gofmt : ovaj program će formatirati vaše izvorne datoteke u skladu sa jezičkim konvencijama

Ako ste u direktorijumu bin, možete pokrenuti go. Probajte ovu komandu:

```sh
./go version
```

Ispisaće go verziju. Ali ovo nije zadovoljavajuće. Želimo da budemo u mogućnosti da otvorimo terminal i pokrenemo `go –version` svuda. Da bismo to uradili, potrebno je da dodamo ovu fasciklu u PATH.

`PATH`: promenljiva okruženja pod nazivom PATH koja sadrži listu direktorijuma gde će školjke tražiti izvršne datoteke kada korisnik izda komandu. Često ćete čuti "dodaj ovaj direktorijum putanji". Ovaj izraz znači da morate da dodate adresu direktorijuma listi direktorijuma u ​​promenljivoj okruženja PATH.

Sledeća komanda će dodati "/usr/local/go/bin" promenljivoj PATH:

```sh
export PATH=$PATH:/usr/local/go/bin
```

Promenljivoj PATH (komandom export) postavljamo vrednost koja je već sadržana u njoj (označena sa $PATH ) i dodajemo joj karaktere ":/usr/local/go/bin". Ako držite isti terminal otvoren i otkucate:

```sh
go version
```

videćete prikazanu verziju programa Go.

#### Izmenite svoj profil školjke

Sada, ako zatvorite terminal i otvorite drugi, primetićete da više ne radi. Promenljiva okruženja PATH je izmenjena samo za vašu trenutnu sesiju. Kada otvorite novi terminal, kreirate novu sesiju. Izmena napravljena na PATH se proširuje na nove sesije.

Da biste izmenili promenljivu PATH za svaku novu sesiju, morate da promenite svoj profil školjke. Potrebno je da dodate:

```sh
export PATH=$PATH:/usr/local/go/bin“ na početak profila.
```

- Ako koristite bash, datoteka za izmenu je `~/.bashrc`. Pokreće se svaki put kada se pokrene sesija.
- Ako koristite zsh, datoteka za izmenu je `~/.zshrc`. Pokreće se svaki put kada se pokrene sesija.
- Za ostale školjke, pogledajte priloženu dokumentaciju, ali bi trebalo da radi isto.

Kada se `.bashrc` ili .zshrc` modifikuju, promena se ne primenjuje odmah. Potrebno je da pokrenete novu sesiju da biste videli rezultat.

Ako ne želite da započnete novu sesiju, možete otkucati ovu komandu i pritisnuti enter:

```sh
source ~/.bashrc
```

#### Testirajte instalaciju

Otvorite terminal i pitajte Go za trenutnu verziju instalacije:

```sh
go version
```

Broj verzije bi trebalo da se pojavi!

### macOS

Imate dve glavne opcije za instaliranje go-a:

- Koristite instalater
- Preuzmite binarnu datoteku i instalirajte je sami.
- Treća opcija bi bila kompajliranje go-a iz izvornog koda. Radi jednostavnosti, pokazaćemo vam kako da koristite instalater.

#### Preuzmite instalater

Idite na veb stranicu <https://golang.org/dl/> i preuzmite odgovarajući instaler za željenu verziju. Na slici 1 smo označili liniju koja odgovara verziji instalera 1.12.8 za macOS.

#### Proverite heš instalatera

Da bismo proverili integritet preuzete datoteke, izračunaćemo SHA256 kriptografski heš. Da biste to uradili, otvorite prozor terminala. Zatim otkucajte komande:

```sh
cd Downloads
shasum -a 256 go1.12.8.darwin-amd64.pkg    SHA_SUM  go1.12.8.darwin-amd64.pkg
```

Prva komanda će promeniti trenutni direktorijum (cd) u `/Downloads`. Zatim ćemo koristiti alatku shasum. Dodajemo zastavicu a da bismo ukazali da želimo da koristimo SHA256 funkciju heširanja. Rezultat će biti sastavljen od kriptografskog heša praćenog imenom heširane datoteke.

Da biste proverili heš, idite na stranicu za preuzimanje i proverite kolonu sa SHA256 kontrolnom sumom. Potrebno je da proverite da li je heš koji dobijete isti kao onaj koji je dao go tim.

Zamenite SHA_SUM vrednošću koju ste kopirali na veb lokaciji.

Hešovi koji se ne podudaraju znače da je datoteka koju ste preuzeli nekako izmenjena. Ne bi trebalo da je koristite.

#### Pokrenite instalater

Dvaput kliknite na preuzetu datoteku. Automatski će se pokrenuti čarobnjak za instalaciju. Pratite proces instalacije. Tokom instalacije biće vam zatražena lozinka. Instalater će datoteke potrebne za pokretanje Go-a smestiti u direktorijum `/usr/local/go`.

#### Izmenite svoj profil školjke na macOS

Otvorite terminal i otkucajte:

```sh
go version
```

Ako je prikazana verzija komande go, onda je sve u redu! U suprotnom, moraćete da dodate Go svojoj promenljivoj okruženja PATH.

Možete pratiti uputstva u prethodnom odeljku. Ista su i za macOS.

### Windows

U ovom odeljku ćemo vam pokazati kako da instalirate Go pomoću.msi instalera. Ovo je najlakši metod. Ali prvo, pogledajte specifične zahteve za Windows operativne sisteme: <https://github.com/golang/go/wiki/MinimumRequirements#windows>.

#### Odredite svoju arhitekturu na Windows OS-u

Go za Windows je dostupan za dve arhitekture:

- x86, što odgovara 32-bitnim sistemima
- x86-64, što odgovara 64-bitnim sistemima

Da biste saznali da li je vaš sistem 32-bitni ili 64-bitni, moraćete da proverite svojstva sistema:

- Za Windows 7
  - Kliknite na dugme Start
  - Kliknite desnim tasterom miša na Računar
  - Zatim kliknite na Svojstva
  - Prozor će prikazati tip sistema
- Za Windows 10 i Windows 8.1
  - Kliknite na dugme Start
  - Zatim idite na Podešavanja, zatim na Sistem i O nama
  - Prozor će prikazati tip sistema

#### Preuzmite instalater za Windows OS

Sada kada znate svoju arhitekturu, možete preuzeti odgovarajući instaler sa zvanične Go veb stranice:

#### Proverite heš instalatera za Windows OS

Kada preuzmete ispravan instalater, morate proveriti njegov integritet. Da bismo to uradili, izračunaćemo SHA256 heš preuzete datoteke.

Windows ima ugrađeni program pod nazivom `certutil.exe`. Ovaj program je deo "Servisa sertifikata". Ovaj program se nalazi u `C:\Windows\System32\certutil.exe`. Ako ga ne pronađete, predlažem vam da pokrenete pretragu u Windows folderu.

Da biste izračunali SHA256 heš, otvorite terminal i zatim otkucajte sledeće komande.

```sh
cd C:\users\max\downloads
certutil -hashfile go1.12.8.windows-amd64.msi SHA256
SHA256 hash of file go1.12.8.windows-amd64.msi:
98 39 25 49 90 de dc 56 47 bf 38 61 1f 7c 1a 7a fc 65 f7 fd 6a af f9 77 e0 5b 17 d4 ec 21 fe a6
CertUtil: -hash file command completed successfully.
```

Zatim morate uporediti heš koji je izračunao certutil sa onim prikazanim na veb-sajtu golang.org. String koji izvede certutil ima razmake između svakog heksadecimalnog broja. Morate kopirati string i ukloniti razmake.

Ako su oba niza jednaka, možete preći na sledeći korak. Ako nisu, dobijate oštećenu verziju. Nemojte je koristiti.

#### Pokrenite instalater na Windows OS-u

Dvaput kliknite na instalater i pratite uputstva. Na kraju procesa instalacije, testirajte instalaciju. Otvorite terminal i otkucajte sledeću komandu:

```sh
go version
```

Ako vidite odštampanu verziju, završili ste instalaciju

## Pregled promenljivih okruženja u programu Go

Koristite promenljive okruženja za njegovu konfiguraciju. U ovom odeljku ćemo detaljno opisati neke od njih:

- **GOBIN** - Podrazumevano, Go će smeštati kompajlirane programe u **$GOPATH/bin**. Ako želite da poništite ovo ponašanje, možete podesiti ovu promenljivu.

- **GOROOT** - Apsolutna putanja gde je instalirana Go distribucija (za korisnike Linux-a i macOS-a, podrazumevano je `/usr/local/go` ).

- **GOHOSTOS** - Ovo je operativni sistem Go alata

- **GOHOSTARCH** - Ovo je sistemska arhitektura binarnih datoteka Go toolchain-a

Da biste ispisali sve promenljive okruženja Go, možete koristiti ovu komandu:

```sh
go env
```

Koristite ovu komandu kada imate problema sa podešavanjima programa Go. Evo rezultata komande:

```sh
GO111MODULE=""
GOARCH="amd64"
GOBIN=""
GOCACHE="/Users/maximilienandile/Library/Caches/go-build"
GOENV="/Users/maximilienandile/Library/Application Support/go/env"
GOEXE=""
GOFLAGS=""
GOHOSTARCH="amd64"
GOHOSTOS="darwin"
GOINSECURE=""
GOMODCACHE="/Users/maximilienandile/go/pkg/mod"
GONOPROXY=""
GONOSUMDB=""
GOOS="darwin"
GOPATH="/Users/maximilienandile/go"
GOPRIVATE=""
GOPROXY="https://proxy.golang.org,direct"
GOROOT="/usr/local/go"
GOSUMDB="sum.golang.org"
GOTMPDIR=""
GOTOOLDIR="/usr/local/go/pkg/tool/darwin_amd64"
GOVCS=""
GOVERSION="go1.16"
GCCGO="gccgo"
AR="ar"
CC="clang"
CXX="clang++"
CGO_ENABLED="1"
GOMOD="/dev/null"
CGO_CFLAGS="-g -O2"
CGO_CPPFLAGS=""
CGO_CXXFLAGS="-g -O2"
CGO_FFLAGS="-g -O2"
CGO_LDFLAGS="-g -O2"
PKG_CONFIG="pkg-config"
GOGCCFLAGS="-fPIC -arch x86_64 -m64 -pthread -fno-caret-diagnostics -Qunused-arguments -fmessage-length=0 -fdebug-prefix-map=/var/folders/lm/9l2tk4811x32rmw4407f9h3m0000gn/T/go-build744222834=/tmp/go-build -gno-record-gcc-switches -fno-common"
```

## Uobičajene greške

U ovom odeljku možete pronaći listu uobičajenih problema koji se javljaju tokom instalacije Go-a.

### Greška „komanda nije pronađena“ ili „nije prepoznata“ kada želim da pokrenem binarnu datoteku go

#### Simptom

Kada koristite zsh shell (na Linux-u ili Mac-u):

```sh
go version
zsh: command not found: go
```

Kada koristite bash shell (na Linux-u ili Mac-u):

```sh
$ go version
bash: command not found: go
```

Za Windows:

```sh
$ go version
'go' is not recognized as an internal or external command, operable program Or batch file.
```

#### Objašnjenje

Ova poruka o grešci znači da vaša školjka nije pronašla izvršnu datoteku „go“. Kada otvorite terminal, školjka će učitati promenljivu okruženja PATH (pogledajte definiciju u prethodnom odeljku). Ova promenljiva sadrži listu adresa direktorijuma na fajl sistemu. Šelk će pretraživati te direktorijume ako može da pronađe izvršnu datoteku pod nazivom „go“. Ako se ne može pronaći izvršna datoteka pod nazivom „go“, izveštaće se o grešci.

#### Rešenja

- Proverite da li je fascikla koja sadrži binarnu datoteku „go“ (go/bin) dodata u vašu putanju.
  - Za korisnike Linuksa i Meka:
    - Otvorite terminal i otkucajte „echo $PATH“
    - Proverite da li se direktorijum go/bin nalazi u rezultatu izlaza

      ```sh
      echo $PATH /usr/bin:/bin:/usr/sbin:/sbin:/usr/local/go/bin
      ```

      Ovde možete videti da se `/usr/local/go/bin` nalazi u putanji
      Ako nije, dodajte `/usr/local/go/bin` putanji (pratite uputstva data u prethodnom odeljku)
  - Za korisnike Windows-a:

    ```sh
    C:> echo %PATH% C:;C:;C:
    ```

    Ovde je podešavanje ispravno. Fascikla bin je u putanji

    Ako nije, dodajte ga u PATH. Da biste to uradili:

    - otvori Sistem > Napredna podešavanja sistema
    - zatim kliknite na dugme Promenljive okruženja. Trebalo bi da se pojavi prozor koji sadrži sve
      promenljive okruženja.
    - Kliknite na „PATH“
    - Dodajte tačku-zarez, a zatim „C:\Go\bin“
    - Kliknite na OK, pa na OK.
    - Zatvorite prozor terminala i otvorite drugi. Trebalo bi da bude sve u redu.

    Ako prethodno rešenje nije funkcionisalo, trebalo bi da proverite da li je binarna datoteka "go" prisutna tamo gde mislite da jeste.

  - Za korisnike Linuksa i Meka:
    - Idite na `/usr/local/go/bin` i proverite da li imate datoteku pod nazivom "go"

  - Za korisnike Windows-a:
    - Idite na `C:\Go\bin` i proverite da li imate datoteku pod nazivom "go".

    Na primer, kada fascikla Go (ili go) čak i ne postoji, moraćete da je ponovo instalirate. Možda ste instaliranu binarnu datoteku premestili u drugi direktorijum?

### Greška „greška u formatu izvršne datoteke“

#### Simptom 02

Binarna datoteka go je pronađena u vašem sistemu, ali kada pokrenete Go komandu dobijate grešku:

```sh
go version
zsh: exec format error: go
```

#### Objašnjenje 02

Verovatno ste preuzeli verziju koja ne odgovara vašem sistemu. Na primer, preuzeli ste binarnu datoteku za Linuk i pokrenuli je na svom Meku.

#### Rešenje 02

Preuzmite verziju koja odgovara vašem operativnom sistemu i arhitekturi vašeg računara.

## Testirajte sebe

### Pitanje i odgovori

1. Kako prikazati broj verzije programa Go?
   - Otvorite terminal
   - Tip
   - $ go version
   - Pritisnite enter

2. Gde pronaći najnoviju verziju programa Go?
   - Najnovije verzije Go-a dostupne su na zvaničnoj Go veb stranici:
     <https://golang.org/dl/>

### Ključno

- Go alati su dostupni na zvaničnoj Go veb stranici
- Go alatni lanac je dostupan na operativnim sistemima Windows, macOS i Linux
- Trebalo bi da preuzmete verziju koja odgovara vašem operativnom sistemu i arhitekturi.
- Nakon instaliranja Go alata, možete pokrenuti go komandu pomoću svog omiljenog terminala.
  - Možda ćete morati da dodate binarnu datoteku go u svoju promenljivu okruženja PATH

[03 Terminal][03] | [00 Sadržaj][00] | [05 Prva Go aplikacija][05]

[03]: 03_Terminal.md
[00]: 00_Sadržaj.md
[05]: 05_Prva_aplikacija.md
