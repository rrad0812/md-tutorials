
# Frappe Framework tutorijal

[Sadržaj][00]

## 05 Anatomija Frappea

### `frappe-bench` direktorijum

```sh
frappe-bench/
├── apps/
├── config/
├── env/
├── logs/
├── sites/
├── Procfile
└── patches.txt
```

Hajde da prođemo kroz ovaj dir, ali ne samo da nabrojimo direktorijume, već da razumemo njihovu ulogu.

- **apps/**  
  Ovo je verovatno najvažniji direktorijum. Ovde živi izvorni kod aplikacija. Na primer:
  
  ```sh
  apps/
      frappe/
      erpnext/
      payments/
      crm/
      moja_aplikacija/
  ```
  
  Svaka aplikacija je praktično jedan Git projekat. To znači da ćeš kasnije moći da uradiš nešto poput:
  
  ```sh
  apps/
      frappe/
          .git
  
      moja_aplikacija/
          .git
  ```
  
  i svaka će imati svoju istoriju. To je jedna od lepih osobina Frappe-a.
  
- **env/**  
  Ovo je Python virtual environment. Drugim rečima:

  ```sh
  python
  pip
  bench
  frappe
  gunicorn
  psycopg
  ...
  ```

  sve živi ovde. Zbog toga ne zagađuješ sistemski Python. To je potpuno isto kao kada napraviš
  
  ```bash
  python -m venv
  ```

  ili

  ```sh
  uv venv
  ```

  samo što Bench to radi sam.
  
- **sites/**  
  Ovo je direktorijum koji početnicima pravi najveću zabunu. Ovde nije izvorni kod. Ovde su podaci i konfiguracija sajtova. Na primer:
  
  ```sh
  sites/
    site1.local/
      site_config.json
      private/
      public/
      locks/
      ...
    assets/
    apps.txt
    common_site_config.json
  ```
  
  Drugim rečima,
  
  - **Aplikacija ≠ Sajt**.  
  - **Jedna aplikacija može biti instalirana na više sajtova.**
  
- **config/**  
  Ovo je Bench konfiguracija. Ovde se nalaze konfiguracioni fajlovi koje Bench generiše. Na primer:
  
  - Redis
  - Nginx
  - Supervisor
  - Procfile konfiguracije
  - razni JSON fajlovi
  
  Većinu vremena nećeš ručno menjati ove fajlove.
  
- **logs/**  
  Vrlo koristan direktorijum. Ako nešto ne radi... ovde prvo gledaš.
  
  Npr.
  
  ```sh
  web.log
  worker.log
  redis.log
  schedule.log
  ```
  
  Kasnije ćeš dosta vremena provoditi upravo ovde.

- **Procfile**  
  Ovo je zanimljiv fajl. Bench ne pokreće jedan proces. Pokreće ih više. Na primer:
  
  - web server
  - scheduler
  - worker
  - socketio
  - watch
  - ...
  
  `Procfile` govori Bench-u: "Ovo su procesi koje treba pokrenuti."
  
  Ako si radio sa Heroku ili Foreman, koncept će ti biti poznat.

- **patches.txt**  
  Ovo nije nešto što ćeš često dirati. Koristi se tokom migracija i nadogradnji kako bi Bench znao koje su zakrpe (patches) već primenjene.

**Globalna organizacija**:  

Primeti da Bench veoma lepo odvaja tri stvari:

```sh
Code -> apps/
Data -> sites/
Enviroment -> env/
```

To je veoma elegantna organizacija.

#### apps/ i sites/ direktorijumi

**Vizuelna slika najvažnijeg dela organizacije Frappe**:

```text
               Bench
                 │
       ┌─────────┴─────────┐
       │                   │
    Applications         Sites
    (izvorni kod)   (podaci i konfiguracija)
```

Idemo redom:

- **apps/**  
  Kod tebe trenutno postoji samo jedna aplikacija:

  ```text
  apps/
  └── frappe/
  ```

  To znači da si instalirao samo Frappe Framework. To je potpuno očekivano.
  Da si instalirao ERPNext, izgledalo bi otprilike ovako:

  ```text
  apps/
  ├── frappe/
  └── erpnext/
  ```

  A kasnije, kada budeš pravio svoju aplikaciju:

  ```text
  apps/
  ├── frappe/
  ├── erpnext/
  └── moja_aplikacija/
  ```

  Primeti jednu stvar: `apps/` ne zna ništa o `site1.local` sajtu.
  Tu nema:
  - konfiguracije sajta,
  - nema baze,
  - nema korisnika,
  - nema podataka.
  
  Samo kod. To je veoma lepo odvajanje odgovornosti.

- **sites/**  
  Ovde se već nalazi mnogo zanimljivijih stvari.
  
  **common_site_config.json**  
  Ovo je globalna konfiguracija Bench-a. Ona važi za sve sajtove. Na primer:
  
  ```txt
  site1.local
  site2.local
  demo.local
  ```
  
  Svi će koristiti ono što je definisano ovde, osim ako neki sajt ne prepiše (override) određenu vrednost.
  
  To je isti koncept koji postoji u mnogim frameworcima: **globalna podešavanja + lokalna podešavanja.**
  
  **site1.local/**  

  Ovo je jedan konkretan Frappe sajt. Vrlo je važno da ga ne posmatraš kao "projekat". On je više nalik instanci aplikacije. Na primer:
  
  - ima svoju bazu,
  - svoje korisnike,
  - svoje dokumente,
  - svoje fajlove,
  - svoja podešavanja.
  
  Ako sutra napraviš:
  
  ```bash
  bench new-site firma2.local
  ```
  
  dobićeš još jedan direktorijum:
  
  ```text
  sites/
  ├── site1.local/
  └── firma2.local/
  ```
  
  Oba će koristiti isti kod iz `apps/frappe/`, ali će imati potpuno odvojene podatke.  
  
  To je jedna od najvećih prednosti Frappe arhitekture.

  **assets/ dir**  

  Ovo često zbuni početnike. Ovde Bench smešta izgrađene (built) statičke resurse. Ne originalni JavaScript. Ne originalni CSS. Već ono što frontend alat napravi nakon build procesa. Drugim rečima:
  
  ```sh
  apps/
      ... source JS ...
    ↓
  bench build
    ↓
  sites/assets/
  ```

  Ako dolaziš iz sveta Vite-a, Webpack-a ili Rollup-a, ovo će ti biti poznato.

  **apps.txt**  

  Ovaj fajl izgleda bezazleno. Verovatno sadrži samo:
  
  ```text
  frappe
  ```
  
  ali je veoma važan. On govori Bench-u: "Ove aplikacije postoje u ovom Bench okruženju."
  Kasnije ćeš ovde videti i:
  
  ```text
  frappe
  erpnext
  moja_aplikacija
  ```

  **apps.json**  

  Ovo je noviji mehanizam koji Bench koristi za dodatne informacije o aplikacijama. U praksi ga retko menjaš ručno; Bench ga održava.

#### Jedna aplikacija - više sajtova

Po mom mišljenju, ovo je najvažnija slika do sada:

```sh
apps/ -> Izvorni kod
sites/ -> Podaci o sajtovima
```

Zamisli sledeće:

```sh
apps/
    frappe/
    erpnext/
    warehouse_app/
```

i

```sh
sites/
    firmaA.local
    firmaB.local
    firmaC.local
```

Sve tri firme koriste:

- isti Frappe,
- isti ERPNext,
- istu tvoju aplikaciju.

Ali svaka ima:

- svoju PostgreSQL bazu,
- svoje korisnike,
- svoje fakture,
- svoje dokumente.

To je veoma elegantan način za **multi-tenant** arhitekturu.

#### Jedan sajt - više aplikacija

Pre nego pređemo na sledeći korak može pitanje:

"Kažeš da jedna aplikacija može da bude na više sajtova. Da li jedan sajt može da ima više aplikacija?

Odgovor je: Da. Zapravo, to je jedan od osnovnih principa Frappe-a.

U stvari, odnos je: **više aplikacija ↔ više sajtova**.  

Možeš to posmatrati kao matricu.

| Aplikacija | site1.local | firmaA.local | firmaB.local |
| ---------- | :---------: | :----------: | :----------: |
| frappe | ✅ | ✅ | ✅ |
| erpnext | ✅ | ✅ | ❌ |
| crm | ❌ | ✅ | ✅ |
| warehouse | ✅ | ❌ | ✅ |

Dakle:

- jedan sajt može imati više aplikacija,
- jedna aplikacija može biti instalirana na više sajtova.

**Frappe je uvek prva instalirana aplikacija na sajtu**  
Svaki sajt mora imati instaliranu aplikaciju `frappe`. Ona je osnova svega.
Na nju se "kače" ostale aplikacije. Na primer:

```txt
site1.local/
├── frappe
├── erpnext
├── payments
└── moja_aplikacija
```

**A šta svaka aplikacija donosi?**

Svaka može da doda:

- nove DocType-ove,
- nove stranice,
- nove API-je,
- nove izveštaje,
- nove Workspaces,
- nove hook-ove,
- nove JavaScript fajlove,
- nove Python module,
- nove migracije.

Drugim rečima, aplikacije se "ugrađuju" u isti Frappe sistem.

Za svaki sajt postoji informacija koje su aplikacije na njemu instalirane.

To možeš čak odmah da proveriš:

```sh
bench --site site1.local list-apps
```

Pošto si napravio potpuno nov sajt, očekujem da će rezultat biti:

```txt
frappe
```

Kasnije, kada instaliraš ERPNext:

```sh
bench --site site1.local install-app erpnext
```

onda će:

```sh
bench --site site1.local list-apps
```

vratiti:

```txt
frappe
erpnext
```

A kada jednog dana napraviš svoju aplikaciju:

```txt
frappe
erpnext
moja_aplikacija
```

#### sites/site1.local/ direktorijum

Ovde ćemo videti sadržaj `sites/site1.local` direktorijuma.

```sh
site1.local/
├── locks
├── logs
├── private
├── public
└── site_config.json
```

Hajde da ga "rastavimo" na delove. Odmah možeš da primetiš jednu zanimljivu stvar.

Ovde nema Python koda. Nema:

- `.py`
- `.js`
- `.html`
- DocType definicija

Zašto? Zato što je sav kod u `apps/`, a "site1.local" sadrži samo ono što pripada toj konkretnoj instanci.

- **site_config.json**  
  Ovo je, po mom mišljenju, najvažniji fajl jednog sajta.
  
  On odgovara na pitanja:
  
  - na koju bazu se povezujem?
  - koji Redis koristim?
  - gde su fajlovi?
  - koje su specifične postavke ovog sajta?
  
  Praktično, kada Frappe "otvori" sajt, prvo pročita ovaj fajl.
  
  Možemo ga detaljno analizirati za nekoliko minuta.

- **private/**  
  Ime govori dosta. Ovde završavaju stvari koje nisu javno dostupne.  
  
  Na primer:
  
  ```txt
    private/
    backups/
    files/
  ```
  
  Kasnije ćeš ovde imati:
  
  - privatne priloge
  - backup fajlove
  - eksportovane podatke
  - razne interne fajlove
  
  Ovo nikada ne treba direktno servirati preko web servera.

- **public/**  
  Suprotno od `private`. Ovde završavaju fajlovi koji mogu biti javno dostupni.
  Najčešće:
  
  ```text
  public/files
  ```
  
  Na primer:
  
  - logo firme
  - slike proizvoda
  - PDF koji je dozvoljeno preuzeti
  - slike koje korisnici vide

- **logs/**  
  
  ```txt
  database.log
  database.log.1
  ```
  
  To znači da svaki sajt ima svoje logove.
  
  Dakle, ako sutra imaš:
  
  ```text
  site1.local
  firmaA.local
  firmaB.local
  ```
  
  svaki može imati sopstvene logove.
  
  To mnogo olakšava administraciju.

- **locks/**  
  Ovaj direktorijum mnogi ni ne primete. Koristi se za razne mehanizme zaključavanja (locking).
  
  Na primer:
  
  - da se dva procesa ne sudare tokom migracije,
  - da scheduler ne pokrene isti posao dva puta,
  - da se spreče paralelne operacije koje bi dovele do nekonzistentnog stanja.
  
  Većinu vremena će biti prazan. I to je potpuno normalno.

**Zanimljivost**  
Ako pogledaš direktorijum `site1.local/` kao celinu on uopšte ne izgleda kao aplikacija. Više liči na profil jednog korisnika sistema.

I to je upravo ono što jeste. Kod je negde drugde.

Ovde su samo:

- konfiguracija,
- podaci,
- fajlovi,
- logovi.

Do sada smo pričali o arhitekturi **Bench → Apps → Sites**, i mislim da je to bio pravi redosled. Međutim, od sledećeg koraka počećemo da povezujemo te delove u jednu celinu. Videćeš da Bench nije "aplikacija", već pre **orkestrator** koji upravlja Python okruženjem, aplikacijama i sajtovima.  

Kada to shvatiš, većina `bench` komandi će postati vrlo intuitivna, jer ćeš razumeti **šta** rade, a ne samo **kako** se koriste.
  
#### sites/site1.local/site_config.json

```sh
cat sites/site1.local/site_config.json
```

```json
{
 "db_name": "_c9eb2d89e08e8728",
 "db_password": "Oq69qLS1RcrKcUxg",
 "db_type": "postgres"
}
```
  
Odmah možemo da izvedemo nekoliko zaključaka:

- **Zašto baza nema ime `site1.local`?**  

  Verovatno si očekivao nešto poput: "site1.local", ili "site1_local". Međutim, Frappe radi drugačije.  On generiše ime baze: "_c9eb2d89e08e8728".

- **Gde je korisničko ime?**

  Primeti nešto zanimljivo. Ovde nema: "db_user": "...". Zašto? Zato što kod PostgreSQL-a Frappe koristi isto ime za bazu i korisnika.
  
  Drugim rečima: db_name: "_c9eb2d89e08e8728" i "role": "_c9eb2d89e08e8728" imaju isto ime.

- **Lozinka**

  Ovu lozinku nisi ti birao. Bench ju je napravio automatski. To je veoma dobra praksa. Svaki sajt dobija:
  
  - svog korisnika
  - svoju bazu
  - svoju nasumičnu lozinku

- **db_type**

  Ovde piše "db_type":"postgres". To znači da ostatak Frappe-a zna koji backend koristi. Da si radio sa MariaDB, ovde bi bilo drugačije.
  
**Kako Frappe pristupa PostgreSQL?**  

Ako ovde nema `db_host` i `db_port` definicije kako Frappe zna da koristi:  127.0.0.1, 5432?

Odgovor je: Ne zna iz ovog fajla. Te informacije dolaze iz drugih delova konfiguracije (globalnih podešavanja Bench-a i podrazumevanih vrednosti).

To znači da jedan `site_config.json` sadrži samo ono što je specifično za taj sajt.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
