
# 33 Konfiguracija aplikacije

[32 Šabloni][32] | [00 Sadržaj][00] | [34 Referentne vrednosti][34]

**Šta ćete naučiti u ovom poglavlju?**

- Kako konfigurisati aplikaciju?
- Kako dodati opciju komandne linije programu?
- Kako kreirati i analizirati promenljive okruženja?

**Obrađeni tehnički koncepti**:

- Promenljiva okruženja
- Demaršal
- Opcije komandne linije

## Uvod

Kada pišemo programe za sebe, (često) direktno u izvorni kod upisujemo neke važne opcije, kao što su, na primer, port na kojem će naš server slušati, ime baze podataka koju koristimo... Ali šta ako pokušate da delite svoj program sa drugima? Postoji značajna verovatnoća da vaši korisnici neće koristiti isto ime baze podataka kao što vi koristite u fazi razvoja; oni će takođe želeti da koriste drugi port za svoj server!

A šta je sa bezbednošću! Da li želite da svi znaju lozinku vaše baze podataka? Ne bi trebalo da unosite akreditive u svoj kod.

U ovom odeljku ćemo objasniti kako možete da konfigurišete svoje aplikacije.

### Šta je to konfigurabilnost aplikacije

Aplikacija je konfigurabilna kada korisnicima nudi mogućnost kontrole aspekata izvršavanja. Na primer, korisnici mogu definisati port na kojem će server slušati, akreditive baze podataka, trajanje vremenskog ograničenja HTTP zahteva...

### Kako to napraviti

U mnogim aplikacijama otvorenog koda, konfiguracija se obrađuje putem strukture podataka ključ-vrednost. Opcije se često označavaju jedinstvenim imenom. Juniks sistemi koriste stringove kao imena opcija. Vindous koristi strukturu stabla.

U stvarnoj aplikaciji često se nalazi klasa (ili tip strukture) koja otkriva opcije konfiguracije. Ova klasa često analizira datoteku kako bi dodelila vrednosti svakoj opciji. Aplikacije često predlažu opcije komandne linije za podešavanje konfiguracija.

## Opcija komandne linije

Da bismo definisali imenovanu opciju komandne linije, možemo koristiti standardni paket `flag`.

Uzmimo primer. Pravite REST API. Vaša aplikacija će izložiti veb server; želite da korisnik može da konfiguriše port za slušanje.

Prvi korak je definisanje nove zastavice:

```go
// configuration/cli/main.go 

port := flag.Int("port", 4242, "the port on which the server will listen")
```

- Prvi argument je naziv zastavice;
- drugi je podrazumevana vrednost;
- treći je tekst pomoći.
- Vraćena promenljiva će biti pokazivač na ceo broj.

Postoji alternativna sintaksa. Prvo definišete promenljivu, a zatim prosleđujete pokazivač na tu promenljivu kao argument funkcije IntVar:

```go
var port int
flag.IntVar(&port, "port", 4242, "the port on which the server will listen")
```

Sledeći korak je analiziranje zastavica:

```go
flag.Parse()
```

Funkcija `Parse` mora biti pozvana pre korišćenja promenljivih koje ste definisali. Interno će iterirati kroz sve argumente date u komandnoj liniji i dodeliti vrednosti zastavicama koje ste definisali.

U prvoj verziji ( `flag.Int` ) promenljiva će biti pokazivač na ceo broj ( `*int` ). U drugoj verziji ( `flag.Int` ) će biti tipa ceo broj ( `int` ). Ova specifičnost menja način na koji koristite promenljivu nakon toga:

```go
// version 1 : Int

port := flag.Int("port", 4242, "the port on which the server will listen")
flag.Parse()
fmt.Printf("%d",*port)

// version 2 : IntVar

var port2 int   flag.IntVar(&port2, "port2", 4242, "the port on which the server will listen") flag.Parse()
fmt.Printf("%d\n",port2)
```

### Ostali tipovi

Paket `flag` otkriva funkcije za dodavanje zastavica različitih tipova. Evo liste tipova sa leve strane i odgovarajućih funkcija paketa `flag`.

- Int64, Int64Var
- String, StringVar
- Uint, UintVar
- Uint64, Uint64Var
- Float64, Float64Var
- Duration, DurationVar
- Bool, BoolVar

### Upotreba u komandnoj liniji

Da biste prosledili navedene zastavice vašem programu, imate dve opcije:

```sh
myCompiledGoProgram -port=4242
```

ili

```sh
myCompiledGoProgram -port 4242
```

Ova poslednja upotreba je zabranjena za bulean zastavice. Umesto toga, korisnik vaše aplikacije može postaviti zastavicu na vrednost "true" jednostavnim pisanjem:

```sh
myCompiledGoProgram -myBoolFlag
```

Na taj način, promenljiva iza zastavice će biti postavljena na vrednost "true".

Da biste videli sve dostupne zastavice, možete koristiti zastavicu pomoći (koja je podrazumevano dodata vašem programu!).

```sh
./myCompiledGoProgram -help
Usage of./myCompiledProgram:
  -myBool
        a test boolean
  -port int
        the port on which the server will listen (default 4242)
  -port2 int
        the port on which the server will listen (default 4242)
```

## Promenljive okruženja

Neki servisi u oblaku podržavaju konfiguraciju putem promenljivih okruženja. Promenljive okruženja se lako podešavaju pomoću komande `export`. Da biste kreirali promenljivu okruženja pod nazivom MYVAR, samo otkucajte sledeću komandu u terminalu:

```sh
export MYVAR=value
```

Ako želite da prosledite promenljive okruženja vašoj aplikaciji, koristite sledeću sintaksu

```sh
export MYVAR=test && export MYVAR2=test2 &&./myCompiledProgram
```

Ovde podešavamo dve promenljive okruženja, a zatim pokrećemo program "myCompiledProgram".

### Kako dobiti vrednost promenljive okruženja

Da biste dobili vrednost promenljive okruženja, možete koristiti `os` standardni paket. Paket pruža funkciju pod nazivom `Getenv`. Ova funkcija uzima ime promenljive kao argument i vraća njenu vrednost (kao string):

```go
// configuration/env/main.go 
myvar := os.Getenv("MYVAR")
myvar2 := os.Getenv("MYVAR2")
fmt.Printf("myvar : '%s'\n", myvar)
fmt.Printf("myvar2 :'%s'\n", myvar2)
```

Druga metoda koja vam omogućava da preuzmete promenljive okruženja je `LookupEnv`. Bolja je od prethodne jer će vas obavestiti ako promenljiva nije prisutna:

```go
// configuration/env/main.go 
port, found := os.LookupEnv("DB_PORT")
if !found {
    log.Fatal("impossible to start up, DB_PORT env var is mandatory")
}
portParsed, err := strconv.ParseUint(port, 10, 8)
if err != nil {
    log.Fatalf("impossible to parse db port: %s", err)
}
log.Println(portParsed)
```

`LookupEnv` će proveriti da li promenljiva okruženja postoji.

- Ako ne postoji, drugi rezultat ( found ) će biti jednak `false`.
- U prethodnom primeru, analiziramo preuzeti port u uint16 (baza 10) sa `strconv.ParseUint`.

### Kako dobiti SVE promenljive okruženja

Takođe možete dobiti sve promenljive okruženja pomoću funkcije `Environ()`, koja će vratiti isečak stringova:

```go
fmt.Println(os.Environ())
```

Vraćeni isečak je sastavljen od svih promenljivih u formatu "ključ=vrednost". Svaki element isečka je par ključ-vrednost. Paket ne izoluje vrednost od ključa. Morate da analizirate vrednosti isečka.

### Kako podesiti promenljivu okruženja

Paket `os` otkriva `os.Setenv` funkciju:

```go
err := os.Setenv("MYVAR3","test3")
if err != nil {
    panic(err)
}
```

`os.Setenv` prihvata dva argumenta:

- ime promenljive koju treba podesiti
- njegova vrednost.

Imajte na umu da ovo može generisati grešku. Morate da rešite tu grešku.

## Konfiguracija zasnovana na datotekama

Konfiguracija se takođe može sačuvati u datoteci i aplikacija je može automatski analizirati.

Format datoteke zavisi od potreba aplikacije. Ako vam je potrebna složena konfiguracija sa nivoima i hijerarhijom (poput Windows registra), onda bi možda trebalo da razmislite o formatima kao što su JSON, YAML ili XML.

Uzmimo primer JSON konfiguracione datoteke:

```json
{
    "server": {
        "host": "localhost",
        "port": 80
    },
    "database": {
        "host": "localhost",
        "username": "myUsername",
        "password": "abcdefgh"
    }
}
```

Ovu datoteku ćemo postaviti negde na mašinu koja hostuje aplikaciju. Moramo da je analiziramo unutar aplikacije da bismo dobili te vrednosti konfiguracije.

Da bismo ga analizirali, prvo moramo da kreiramo tip strukture u našem programu:

```go
// configuration/file/main.go 
type Configuration struct {
    Server   Server `json:"server"`
    Database DB     `json:"database"`
}
type Server struct {
    Host string `json:"host"`
    Port int    `json:"port"`
}
type DB struct {
    Host     string `json:"host"`
    Username string `json:"username"`
    Password string `json:"password"`
}
```

Kreiramo tri tipa struktura:

- Configuration koji će grupisati druga dva tipa
  - Server koji sadrži JSON svojstva servera
  - DB koji čuvaju JSON svojstva baze podataka

Te strukture tipova će vam omogućiti da demaršalujete JSON konfiguracionu datoteku. Sledeći korak je otvaranje datoteke i čitanje njenog sadržaja:

```go
// configuration/file/main.go 

confFile, err := os.Open("myConf.json")
if err != nil {
    panic(err)
}
defer confFile.Close()
conf, err := ioutil.ReadAll(confFile)
if err != nil {
    panic(err)
}
```

Ovde pozivamo `os.Open` da otvorimo datoteku "myConf.json" koja se za primer nalazi u istom direktorijumu kao i izvršna datoteka. U stvarnom životu, trebalo bi da otpremite ovu datoteku na odgovarajuću lokaciju na mašini.

Dobijamo ceo sadržaj datoteke koristeći `ioutil.ReadAll`. Ova funkcija vraća isečak bajtova. Sledeći korak je korišćenje ovog isečka sirovih bajtova sa `json.Unmarshal`:

```go
// configuration/file/main.go 

myConf := Configuration{}
err = json.Unmarshal(conf, &myConf)
if err != nil {
    panic(err)
}
fmt.Printf("%+v",myConf)
```

Kreiramo promenljivu `myConf` koja je inicijalizovana praznom strukturom `Configuration{}`. Zatim prosleđujemo promenljivu `conf` koja sadrži isečak bajtova izvučen iz datoteke i (kao drugi argument) pokazivač na `json.Unmarshal` "myConf".

Na kraju, naš program izbacuje sledeće:

{Server:{Host:localhost Port:80} Database:{Host:localhost Username:myUsername Password:abcdefgh}}

Zatim možete deliti ovu promenljivu sa celom aplikacijom.

## Modul github.com/spf13/viper

Modul `github.com/spf13/viper` je u vreme pisanja ovog teksta veoma popularan među Go zajednicom. Ima više od 14.000 zvezdica i više od 1,3 hiljade forkova.

Ovaj paket vam omogućava da jednostavno definišete konfiguraciju za vašu aplikaciju iz datoteka, promenljivih okruženja, komandne linije, bafera... Takođe podržava zanimljivu funkciju: ako se vaša konfiguracija promeni tokom životnog veka vaše aplikacije, ona će biti ponovo učitana.

### Čitanje iz datoteke

Hajde da pokušamo:

```go
// configuration/viper/main.go 
//...
// import "github.com/spf13/viper"
viper.SetConfigName("myConf")
viper.AddConfigPath(".")
err := viper.ReadInConfig()
if err != nil {
    log.Panicf("Fatal error config file: %s\n", err)
}
```

Prvo, morate definisati ime konfiguracione datoteke koju želite da učitate: `viper.SetConfigName("myConf")`. Ovde učitavamo datoteku pod nazivom "myConf". Imajte na umu da Viper-u ne dajemo ekstenziju datoteke. On će preuzeti ispravno kodiranje i analizirati datoteku.

U sledećem redu moramo reći viperu da pretražuje pomoću funkcije `AddConfigPath`. Tačka znači "traži u trenutnom direktorijumu" datoteku pod nazivom "myConf". Podržane ekstenzije u vreme pisanja ovog teksta su:

- json,
- toml,
- jaml,
- yml,
- properties,
- props,
- prop,
- hcl

Imajte na umu da smo mogli dodati još jednu putanju za pretragu konfiguracije. Funkcija `AddConfigPath` će dodati putanju isečku.

Kada pozovemo `ReadInConfig`, paket će tražiti navedenu datoteku.

Zatim možete koristiti učitanu konfiguraciju:

```go
fmt.Println(viper.AllKeys())
//[database.username database.password server.host server.port database.host]
fmt.Println(viper.GetString("database.username"))
// myUsername
```

Pomoću `AllKeys` funkcije možete da preuzmete sve postojeće konfiguracione ključeve vaše aplikacije. Da biste pročitali određenu konfiguracionu promenljivu, možete koristiti funkciju `GetString`.

Funkcija `GetString` kao parametar uzima ključ konfiguracione opcije. Ključ je odraz hijerarhije konfiguracione datoteke. Ovde "database.username" znači da pristupamo vrednosti svojstva korisničko ime iz baze podataka.

Objekat baze podataka takođe ima svojstvo host. Da biste mu pristupili, ključ je jednostavno "database.host".

Viper nudi "GetXXX" funkcije za tipove bool, float64, int, string, map[string]interface{}, map[string]string, isečak stringova, vreme i trajanje. Takođe je moguće jednostavno koristiti Get za preuzimanje vrednosti konfiguracije. Tip povratka ove poslednje funkcije je prazan interfejs. Ne preporučujem to jer može dovesti do grešaka tipa. U našem slučaju, očekujemo string, ali ako korisnik unese ceo broj u konfiguracionu datoteku, vratiće se int. Vaš program neće raditi kako se očekuje i mogao bi da paniči.

### Čitanje iz promenljivih okruženja

Viper može automatski da analizira promenljive okruženja sa prefiksom:

```go
viper.SetEnvPrefix("myapp")
viper.AutomaticEnv()
```

Sa te dve linije, svaki put kada ćete pozvati metod za dobijanje ( GetString, GetBool,... ), Viper će tražiti promenljive okruženja sa prefiksom MYAPP_ :

```go
fmt.Println(viper.GetInt("timeout"))
```

tražiće vrednost promenljive okruženja pod nazivom MYAPP_TIMEOUT.

## Problemi i kako ih izbeći

### Nedostajuća dokumentacija

Mnogi projekti pate od nedostatka dokumentacije koja se tiče njihovih opcija konfiguracije.

Dokumentovanjem svake dostupne opcije u posebnoj datoteci, smanjujete prepreku usvajanja vašeg softvera.

Unutar kompanije, možda niste osoba koja će se baviti implementacijom vaše aplikacije. Stoga, u svoje procene morate uključiti vreme potrebno za dokumentaciju. Bolja dokumentacija znači kraće vreme potrebno za plasman vašeg rešenja na tržište.

Zanimljiva studija Ariel Rabkin i Rendija Kaca sa Univerziteta Berkli pokazala je da je čak i za velike projekte otvorenog koda (proučavali su sedam velikih projekata poput Kasandre, Hadupa, HBasea...) dokumentacija konfiguracije bila netačna:

- Pominju se opcije konfiguracije koje ne postoje u izvornom kodu aplikacije. Primetili su da su ponekad opcije jednostavno komentarisane u izvornom kodu aplikacije.

- Neke opcije postoje u izvornom kodu aplikacije koje nisu čak ni dokumentovane.

Rešenje je jednostavno: kreirajte preciznu dokumentaciju o opcijama konfiguracije i redovno je ažurirajte kako se projekat razvija.

### Greške u konfiguraciji

Tokom jedne studije od tehničkih operatera je zatraženo da izvrše operacije na realističnom internet sistemu (aplikacija za aukciju sa tri nivoa). Primećeno je da je 50% grešaka bilo greške u konfiguraciji!

Ova brojka je impresivna i veoma zanimljiva. Sistemi mogu da otkažu zbog grešaka u konfiguraciji. Ne možete se zaštititi od korisnika koji su uneli pogrešan broj porta u konfiguraciju. Ali vaša aplikacija može da se zaštiti od pogrešne konfiguracije.

- Detekcijom i upozoravanjem korisnika.
- Nezamenom lažne vrednosti podrazumevanom.

```go
// configuration/error-detection/main.go 
//...
dbPortRaw := os.Getenv("DATABASE_PORT")
dbPort, err := strconv.ParseUint(port, 10, 16)
if err != nil {
    log.Panicf("Impossible to parse database port number '%s'. Please double check the env variable DATABASE_PORT",dbPortRaw)
}
```

U prethodnom kodu, preuzimamo vrednost promenljive okruženja DATABASE_PORT. Zatim pokušavamo da je konvertujemo u uint16. Ako dođe do greške, odbijamo pokretanje aplikacije i obaveštavamo korisnika o tome šta nije u redu.

Imajte na umu da ovde kažemo korisniku šta da uradi da bi ispravio ovu grešku. Ovo je dobra praksa, potrebno vam je samo 10 sekundi da napišete, ali može uštedeti korisniku 1 sat istraživanja.

Takođe proverite da li program efikasno koristi sve vaše konfiguracione promenljive.

## Bezbednost

Promenljive konfiguracije se često sastoje od tajni: lozinki, pristupnih tokena, ključeva za šifrovanje... Potencijalno curenje tih tajni može prouzrokovati štetu. Nijedno od gore navedenih rešenja nije savršeno za čuvanje i obezbeđivanje proizvodnih tajni. Rotacija tih tajni takođe nije jednostavna sa prethodno pomenutim rešenjem.

Postoje neka rešenja otvorenog koda koja se bave ovim veoma specifičnim problemom. Ona nude veliki skup funkcija za zaštitu, praćenje i reviziju korišćenja vaših tajni. U vreme pisanja ovog teksta, čini se da je Vault, koji uređuje Hashicorp, najpopularniji. 3 I napisan je u Go! jeziku.

## Testirajte sebe

1. Kako dodati opciju komandne linije tipa string u program?

   ```go
   // define other flags
   var password stringflag.StringVar(&password,"password", "default","the db password")
   // parse input flags
   flag.Parse()

   password2 := flag.String("password2", "default", "lozinka za bazu podataka")
   ```

2. Kako proveriti da li promenljiva okruženja postoji i dobiti njenu vrednost u istom pozivu?

   - Sa funkcijom `LookupEnv` iz paketa `os`

     ```go
     port, ok := os.LookupEnv("DB_PORT”) 
     if !ok { 
         log.Fatal("nemoguće pokretanje, promenljiva okruženja DB_PORT je obavezna”) 
     }
     ```

3. Kako podesiti promenljivu okruženja?

   - Sa funkcijom `SetEnv` iz paketa `os`

     ```go
     error := os.Setenv("MYVAR3”, "test3”) 
     if error != nil { panic(error) }
     ```

4. Kako konfigurisati aplikaciju pomoću JSON datoteke?

   - Kreirajte određeni tip sa svim vašim konfiguracionim promenljivim.
   - Pri bootstrap-u, otvorite JSON datoteku
   - Raspakuj datoteku.

## Ključno

- Konfiguracija aplikacije može se izvršiti pomoću opcija komandne linije.
- Standardni paket `flag` pruža metode za dodavanje takvih opcija.

  ```go
  // define other flags
  var password stringflag.StringVar(&password,"password", "default","the db password")
  // parse input flags
  flag.Parse()
  ```

- Opcije konfiguracije se daju pri pokretanju programa putem komandne linije:
  
  ```sh
  myCompiledGoProgram -password insecuredPass
  ```

- Konfiguracija se generalno učitava pri pokretanju aplikacije (u glavnom)
- Konfiguracija aplikacije se takođe može obaviti putem promenljivih okruženja
- Da biste preuzeli promenljivu okruženja, možete koristiti ospaket

  ```go
  dbHost := os.GetEnv("DB_HOST”)

  dbHost, pronađeno := os.LookupEnv("DB_HOST”)
  ```

. Takođe možete da upravljate konfiguracijom putem datoteke (JSON, YAML,...). Ideja je da se kreira struktura tipa i raspakuje sadržaj datoteke.

- Modul github.com/spf13/viper je popularna referenca za rukovanje konfiguracijom aplikacije.

- Programeri treba pažljivo da dokumentuju opcije konfiguracije. Ažurirana dokumentacija je konkurentska prednost.

- Posebno rešenje za upravljanje tajnim podacima trebalo bi da obrađuje tajne aplikacija.

[32 Šabloni][32] | [00 Sadržaj][00] | [34 Referentne vrednosti][34]

[32]: 32_Šabloni.md
[00]: 00_Sadržaj.md
[34]: 34_Referentne_vrednosti.md
