# Frappe Framework tutorijal

[Sadržaj][00]

## 06 Pokretanje razvojnog okruženja

Iz `frappe-bench` direktorijuma pokreni:

```bash
bench start
```

```sh
15:31:35 system        | redis_cache.1 started (pid=1419)
15:31:35 system        | redis_queue.1 started (pid=1422)
15:31:35 system        | socketio.1 started (pid=1429)
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # Redis version=7.0.15, bits=64, commit=00000000, modified=0, pid=1420, just started
15:31:35 redis_cache.1 | 1420:C 13 Jul 2026 15:31:35.033 # Configuration loaded
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.033 * Increased maximum number of open files to 10032 (it was originally set to 1024).
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.033 * monotonic clock: POSIX clock_gettime
15:31:35 system        | web.1 started (pid=1432)
15:31:35 system        | worker.1 started (pid=1441)
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 * Running mode=standalone, port=13000.
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 # Server initialized
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
15:31:35 redis_cache.1 | 1420:M 13 Jul 2026 15:31:35.042 * Ready to accept connections
15:31:35 system        | watch.1 started (pid=1427)
15:31:35 system        | schedule.1 started (pid=1431)
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # oO0OoO0OoO0Oo Redis is starting oO0OoO0OoO0Oo
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # Redis version=7.0.15, bits=64, commit=00000000, modified=0, pid=1426, just started
15:31:35 redis_queue.1 | 1426:C 13 Jul 2026 15:31:35.053 # Configuration loaded
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.053 * Increased maximum number of open files to 10032 (it was originally set to 1024).
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.053 * monotonic clock: POSIX clock_gettime
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 * Running mode=standalone, port=11000.
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 # Server initialized
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.054 # WARNING Memory overcommit must be enabled! Without it, a background save or replication may fail under low memory condition. Being disabled, it can can also cause failures without low memory condition, see https://github.com/jemalloc/jemalloc/issues/1328. To fix this issue add 'vm.overcommit_memory = 1' to /etc/sysctl.conf and then reboot or run the command 'sysctl vm.overcommit_memory=1' for this to take effect.
15:31:35 redis_queue.1 | 1426:M 13 Jul 2026 15:31:35.057 * Ready to accept connections
15:31:36 watch.1       | 
15:31:36 web.1         | /home/radosav/frappe-bench/env/lib/python3.12/site-packages/passlib/utils/__init__.py:854: DeprecationWarning: 'crypt' is deprecated and slated for removal in Python 3.13
15:31:36 web.1         |   from crypt import crypt as _crypt
15:31:36 socketio.1    | Realtime service listening on: ws://0.0.0.0:9000
15:31:36 watch.1       | yarn run v1.22.22
15:31:36 watch.1       | $ node esbuild --watch --live-reload
15:31:37 web.1         | WARNING: This is a development server. Do not use it in a production deployment. Use a production WSGI server instead.
15:31:37 web.1         |  * Running on all addresses (0.0.0.0)
15:31:37 web.1         |  * Running on http://127.0.0.1:8000
15:31:37 web.1         |  * Running on http://192.168.122.74:8000
15:31:37 web.1         | Press CTRL+C to quit
15:31:37 web.1         |  * Restarting with stat
15:31:37 web.1         | /home/radosav/frappe-bench/env/lib/python3.12/site-packages/passlib/utils/__init__.py:854: DeprecationWarning: 'crypt' is deprecated and slated for removal in Python 3.13
15:31:37 web.1         |   from crypt import crypt as _crypt
15:31:37 web.1         |  * Debugger is active!
15:31:37 web.1         |  * Debugger PIN: 972-324-418
15:31:43 watch.1       | Browserslist: caniuse-lite is outdated. Please run:
15:31:43 watch.1       |   npx update-browserslist-db@latest
15:31:43 watch.1       |   Why you should do it regularly: https://github.com/browserslist/update-db#readme
15:31:54 watch.1       | Watching for changes...
```

Ono što je najlepše: **Sve radi**.

Sada ćemo objasniti zašto se pokreće baš ovih sedam procesa.

Prva važna stvar je: **Bench nije server** . Mnogi početnici misle: "Pokrenuo sam Bench."  Ne. Pokrenuo si **orkestrator**. `bench start` ne izvršava Frappe direktno. On pokreće više nezavisnih procesa i nadgleda ih.

Praktično radi nešto ovako:

```text
    Vizuelni prikaz pokrenutih procesa

                bench start
                     │
     ┌───────────────┼────────────────┐
     │               │                │
   web.py       redis_cache      redis_queue
     │
  socketio
     │
  scheduler
     │
   worker
     │
   watch
```

Ako jedan proces padne, Bench to vidi i ispisuje u terminal. To je razlog zbog kog su svi logovi objedinjeni na jednom mestu.

To je praktično mapa Frappe-a.

Svaki od procesa rešava jedan veoma specifičan problem.
  
- **WEB**:

  Ovo je najlakši.
  
  ```txt
  web.1
  ```
  
  Kasnije vidiš
  
  ```txt
  Running on
  127.0.0.1:8000
  192.168.122.74:8000
  ```
  
  To je HTTP server. Browser priča sa njim. Ako otvoriš <http://192.168.122.74:8000> ili preko port forwardinga sa hosta, prvo se javlja upravo **web proces**. Ali... web ne radi sve. On samo prima zahtev.
  
- **SOCKETIO**:

  ```txt
  Realtime service listening
  ws://0.0.0.0:9000
  ```
  
  Ovo je sasvim drugi server. Ne HTTP. Već **WebSocket**. Njegova svrha je: "server → browser" bez refresh-a.
  
  Na primer:
  
  - notifikacije
  - chat
  - progress bar
  - live dashboard
  - background job završen
  
  Sve to dolazi preko SocketIO.

- **REDIS CACHE**:

  Prvi Redis.
  
  ```text
  port 13000
  ```
  
  Ovaj Redis služi kao memorijski keš.
  
  Na primer:
  
  ```txt
  Korisnik -> Permissions -> Redis Cache -> sledeći zahtev -> ne čita bazu ponovo
  ```
  
  Time se štedi mnogo SQL upita.

- **REDIS QUEUE**:

  Drugi Redis.
  
  ```txt
  port 11000
  ```
  
  Ovo je potpuno druga uloga. Ovde se ne čuvaju podaci. Ovde se čuvaju zadaci.  
  
  Na primer:
  
  ```text
  Pošalji 500 emailova.
  ```
  
  Browser neće čekati. Web kaže: "Stavi ovo u Queue". Redis Queue zapamti posao. Worker ga kasnije izvrši.
  
- **WORKER**:

  Jedan od mojih omiljenih procesa.
  
  ```text
  worker.1
  ```
  
  Njegov posao je veoma jednostavan.  
  
  Beskonačna petlja:
  
  ```txt
  Ima li nešto u Queue? -> nema -> čekaj -> ima -> izvrši -> čekaj
  ```
  
  To je sve. Ali zahvaljujući njemu browser ostaje brz.

- **SCHEDULER**:
  
  Ovo je nešto kao cron.
  
  Na primer:
  
  ```txt
  svakih 5 minuta -> pokreni cleanup
  ```
  
  ili
  
  ```text
  svake noći -> backup
  ```
  
  ili
  
  ```text
  svakih sat vremena -> sync
  ```
  
  Scheduler ne izvršava posao. On samo kaže: "Vreme je."  
  Posao zatim ubaci u Queue. Worker ga izvrši.  
  Primeti kako se procesi lepo nadovezuju.

- **WATCH**:

  Ovo koriste programeri.
  
  Kod tebe se lepo vidi
  
  ```txt
  esbuild --watch
  ```
  
  i kasnije
  
  ```text
  Watching for changes
  ```
  
  Šta to znači?
  
  Ako promeniš
  
  ```txxt
  apps/
  frappe/
  ...    
  ...
  ...
  some.js
  ```
  
  Watch odmah vidi izmenu. Automatski pokrene build. Ne moraš ručno.  
  To je ogromna ušteda vremena.

**Jedan primer**  

Recimo da klikneš u browseru
  
```txt
Create Customer
```

Šta se dešava?

Otprilike ovo:

```txt
Browser -> HTTP -> WEB -> Python -> PostgreSQL -> vrati rezultat -> Browser
```

Ali ako taj Customer treba da pošalje email dobrodošlice:

```txt
WEB -> Queue -> Redis Queue -> Worker -> SMTP -> Email poslat
```

Browser u međuvremenu već prikazuje stranicu.  
Ne čeka email.

**A ako email traje 20 sekundi?**  
  
Nema problema. Worker radi. Browser radi.

SocketIO može da kaže:

```txt
Email sent.
```

bez refresh-a.

**Šta je zapravo Frappe?**  
Ja bih odgovorio ovako:

Osnovni princip rada Frappe frameworka

```text
                 Browser
                     │
                HTTP/WebSocket
                     │
         ┌───────────┴───────────┐
         │                       │
       Web                  SocketIO
         │
         ▼
      Frappe Framework
         │
 ┌───────┼──────────────────┐
 │               │          │
 ▼               ▼          ▼
PostgreSQL Redis Queue Redis Cache
           │
           ▼
         Worker
```

To je mnogo bliže stvarnoj slici nego jednostavno reći "Frappe je Python web framework".

**Dve zanimljive sitnice iz tvog izlaza**:

- Prva je upozorenje:

  ```txt
  Memory overcommit must be enabled!
  ```
  
  Ovo nije greška. Redis samo preporučuje da Linux ima:
  
  ```bash
  vm.overcommit_memory = 1
  ```
  
  Za razvoj možeš da ga ignorišeš. Za produkciju ga je dobro podesiti.

- Druga je:

  ```text
  passlib
  DeprecationWarning: crypt is deprecated
  ```
  
  To dolazi od Python-a 3.12 i biblioteke `passlib`. Nije do tvog sistema niti do Frappe konfiguracije. Očekujem da će biti rešeno u novijim verzijama zavisnosti.

## Prva poseta sajtu

Sada prelazimo na ono što ja smatram prvim pravim susretom sa Frappe-om. Do sada smo bili "ispod haube". Sada ćemo prvi put pogledati kako izgleda sistem iz ugla korisnika, ali ćemo ga posmatrati iz ugla programera.

**Otvori Frappe u browseru**:

Pošto radiš u VM-u, trebalo bi da možeš da otvoriš:

```http
http://site1.local:8000
```

ili, ako koristiš SSH tunel ili port forwarding, odgovarajuću adresu na hostu. Trebalo bi da dobiješ login ekran. Nemoj još ništa da istražuješ. Samo potvrdi da se stranica otvara.

**Jedan HTTP zahtev**:

Pre nego što se uloguješ, hajde da ispratimo jedan jedini HTTP zahtev.

- Browser šalje:

  ```txt
  GET /
  ```

- Web server prima zahtev.

- Frappe kaže: "Koji sajt je tražen?". Pošto imaš samo jedan sajt (`site1.local`), odgovor je
  jednostavan.

- Frappe učitava:  

  ```txt
  sites/site1.local/site_config.json
  ```

- Povezuje se na PostgreSQL.  

- Pronalazi da korisnik nije prijavljen.  

- Generiše HTML login stranice.  

- Browser je prikazuje.  

To je ceo prvi ciklus.

**Kako Frappe zna da treba da koristi baš `site1.local`?**:

Na produkcionom serveru odgovor je jednostavan:

```sh
erp.firma.rs
```

- Host zaglavlje (HTTP Host header)
- site1.local ili firma.rs ili erp.example.com
- Svaki domen odgovara jednom sajtu.

Ali ti nemaš domen.  
I nemaš Nginx.

Imaš samo:

```txt
<http://192.168.122.74:8000>
```

Pa kako onda zna?  
Hajde da pogledamo sledeći važan fajl u Bench arhitekturi.
  
### `common_site_config.json` - zajednička definicija za sve sajtove

```sh
cat sites/common_site_config.json
```

```json
{
 "background_workers": 1,
 "file_watcher_port": 6787,
 "frappe_user": "radosav",
 "gunicorn_workers": 5,
 "live_reload": true,
 "rebase_on_pull": false,
 "redis_cache": "redis://127.0.0.1:13000",
 "redis_queue": "redis://127.0.0.1:11000",
 "redis_socketio": "redis://127.0.0.1:13000",
 "restart_supervisor_on_update": false,
 "restart_systemd_on_update": false,
 "serve_default_site": true,
 "shallow_clone": true,
 "socketio_port": 9000,
 "use_redis_auth": false,
 "webserver_port": 8000
}
```

Sada imamo praktično kompletnu sliku kako Bench funkcioniše. Po mom mišljenju, `common_site_config.json` je **kontrolni centar** Bench-a, dok je `site_config.json` lična karta jednog sajta.

Hajde da ga rastavimo.

- **Dva nivoa konfiguracije**

  Već sada možeš da vidiš da postoje dva nivoa:
  
  ```txt
  sites/
      common_site_config.json [ 1. -> (važi za sve sajtove)]
      site1.local/
          site_config.json    [ 2. -> (samo za site1.local)]
  ```
  
  To je veoma elegantan dizajn. Globalne stvari pišu se jednom. Specifične stvari pišu se po sajtu.
  
- **Redis**

  Odmah vidiš tri Redis konekcije:
  
  ```json
  "redis_cache": "redis://127.0.0.1:13000",
  "redis_queue": "redis://127.0.0.1:11000",
  "redis_socketio": "redis://127.0.0.1:13000"
  ```
  
  Odmah možemo da povežemo sa onim što smo videli juče.
  
  ```txt
  Redis Cache -> port:13000
  Redis Queue -> port: 11000
  SocketIO -> koristi isti Redis kao Cache.
  ```
  
  Zašto? Zato što SocketIO koristi Redis kao **message broker** između procesa.  
  To ćemo detaljnije videti kasnije kada budemo pričali o realtime događajima.

- **Web server**

  Ovde stoji
  
  ```json
  "webserver_port": 8000
  ```
  
  To je upravo ono što si video u izlazu:
  
  ```sh
  Running on http://127.0.0.1:8000
  ```
  
  Dakle Bench nije "pogodio" port. On ga je pročitao odavde.

- **SocketIO**

  Ovde piše
  
  ```json
  "socketio_port": 9000
  ```
  
  A u logovima si video
  
  ```sh
  Realtime service listening
  ws://0.0.0.0:9000
  ```
  
  Opet se sve poklapa.

- **File watcher**

  ```json
  "file_watcher_port": 6787
  ```
  
  Ovo koristi Watch proces.

- **Background workers**

  ```json
  "background_workers": 1
  ```
  
  To znači da trenutno imaš jednog Worker-a.
  
  Kasnije možeš imati u zavisnosti od opterećenja.

- **Gunicorn workers**

  ```json
  "gunicorn_workers": 5
  ```
  
- **Najzanimljivija stavka**

  ```json
  "serve_default_site": true
  ```
  
  Kako Frappe zna da otvori baš `site1.local` kada u URL-u pišem samo IP adresu?
  
  Evo odgovora.
  
  Pošto je
  
  ```json
  serve_default_site = true
  ```
  
  Bench kaže: Ako ne mogu da odredim sajt na osnovu HTTP Host zaglavlja, posluži podrazumevani sajt. Pošto imaš samo jedan sajt, to je upravo `site1.local`.

  Kasnije, kada budeš imao:
  
  ```text
  site1.local
  firmaA.local
  firmaB.local
  ```
  
  više neće biti dovoljno da pristupiš preko IP adrese.
  
  HTTP Host zaglavlje ili Nginx će odlučivati koji sajt treba otvoriti.

- **Korisnik Bench-a**

  Ovde piše
  
  ```json
  "frappe_user": "radosav"
  ```
  
  To znači da je ceo Bench napravljen pod tvojim Linux korisnikom. Bench treba da radi kao običan korisnik.
  
  To je veoma dobra praksa.
  
### Oba conf fajla zajedno

Pogledaj sada zajedno oba fajla.
  
- **Globalno - common_site_config.json**
  - redis
  - portovi
  - worker
  - socketio

- **Lokalno - site1.local/site_config.json**
  - db_name
  - db_password
  - db_type

Odjednom postaje jasno zašto su odvojeni.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
