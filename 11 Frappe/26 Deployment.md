# Frappe Framework tutorijal

[Sadržaj][00]

## 26 Deployment

Do sada smo govorili o tome kako napraviti aplikaciju, kako dodati modele, forme, dozvole, report-e i automatizaciju.

Ali jedna stvar je presudna za svaku aplikaciju: kako je staviti u produkciju.

Deployment znači da tvoj lokalni razvoj postaje stvarna aplikacija na serveru, dostupna korisnicima.

U praksi deployment treba da reši:

- server,
- domen,
- bazu,
- web server,
- workers/scheduler,
- backup-e,
- SSL,
- i proces nadogradnje aplikacije.

U Frappe-u deployment nije samo "pokrenuti aplikaciju".

To je skup sledećih stvari:

1. instalacija okruženja na serveru,
2. kreiranje bench-a,
3. kreiranje site-a,
4. instalacija aplikacije,
5. migracija i pokretanje servisa,
6. konfiguracija domena i SSL-a.

### Osnovna arhitektura produkcije

U produkciji ti obično imaš:

- server ili VPS,
- Linux operativni sistem,
- Python i Node.js,
- MariaDB ili PostgreSQL,
- Redis,
- Nginx,
- Supervisor ili systemd,
- Frappe bench i site.

Najjednostavnije rečeno:

```text
Korisnik → Nginx → Frappe / Gunicorn / workers → baza podataka
```

### Razlika između development i production

U development-u obično radiš ovako:

```bash
bench start
```

To je dobro za lokalni rad.

U production-u ne koristiš samo `bench start`.

Umesto toga koristiš:

- Nginx za HTTP/HTTPS,
- worker procese za pozadinske zadatke,
- scheduler za scheduled jobs,
- supervisor/systemd za pokretanje servisa.

### Ključni koraci deployment-a

#### 1. Priprema servera

Na serveru moraš imati:

- Ubuntu ili sličan Linux sistem,
- Python,
- pip,
- build tools,
- bazu podataka,
- Redis,
- Nginx,
- bench.

#### 2. Kreiranje bench-a

Bench je osnovno okruženje za Frappe aplikacije.

```bash
bench init frappe-bench --frappe-branch version-15
```

#### 3. Dodavanje aplikacije

Ako imaš sopstvenu aplikaciju, dodaješ je u bench:

```bash
cd frappe-bench
bench get-app myapp /put/do/repositorijuma/myapp
```

#### 4. Kreiranje site-a

Svaki Frappe sajt je jedan site.

```bash
bench new-site example.com
```

#### 5. Instalacija aplikacije na site-u

```bash
bench --site example.com install-app myapp
```

#### 6. Migracija

Kada promeniš modele ili strukturu aplikacije, moraš uraditi migraciju:

```bash
bench --site example.com migrate
```

### Nginx i domain

Da bi korisnici mogli da pristupe sajtu, potrebno je povezati domen sa serverom.

Najčešće radiš:

```bash
bench setup nginx
```

Onda dodaješ domen u konfiguraciji i omogućavaš SSL.

### Workers i scheduler

Frappe aplikacije često rade sa:

- web procesima,
- background workerima,
- schedulerom.

To je važno zato što scheduled jobs i neke pozadinske operacije ne smeju da zavise od jednog terminala.

U production-u to obično pokrećeš kroz Supervisor ili systemd.

### Primer deployment procesa

Jednostavan tok izgleda ovako:

```bash
cd frappe-bench
git pull
bench --site example.com migrate
bench restart
```

Ako je aplikacija promenjena i dodata nova verzija, često se radi:

```bash
bench --site example.com migrate
bench restart
```

### Backup i sigurnost

Deployment nije kompletan bez backup-a.

Dobra praksa je:

- backup baze podataka,
- backup aplikacije,
- backup konfiguracije,
- i plan za povratak sistema.

### Najčešće greške pri deployment-u

1. Nije pokrenut scheduler
2. Nije konfigurisan Nginx
3. Site nije pravilno instaliran
4. Migracija nije urađena
5. SSL i domen nisu povezani
6. Nepostojanje backup-a

### Šta je u praksi najvažnije

Za početak, ne treba odmah da misliš na "kompletnu enterprise infrastrukturu".

Najvažniji prvi korak je:

- server,
- bench,
- site,
- aplikacija,
- migrate,
- nginx,
- scheduler.

To je već dovoljan osnovni deployment.

### Deployment na VPS-u

Jedan od najčešćih načina je deployment na VPS-u, na primer Ubuntu serveru.

Tipičan tok izgleda ovako:

1. kupiš VPS,
2. povežeš domen,
3. instaliraš potrebne dependency-je,
4. postaviš bench,
5. napraviš site,
6. instaliraš aplikaciju,
7. podesiš Nginx i SSL,
8. pokreneš scheduler i worker-e.

To je vrlo praktičan pristup kada želiš da aplikacija bude dostupna stalno i da imaš kontrolu nad serverom.

### Deployment sa Docker-om

Docker je još jedan način da se Frappe aplikacija pokrene u kontrolisanom okruženju.

Prednosti su:

- isti environment na lokalnom i produkcionom serveru,
- lakše upravljanje servisima,
- jednostavnije skaliranje i ponovnu instalaciju.

U praksi to znači da imaš kontejnere za:

- web aplikaciju,
- bazu,
- Redis,
- i eventualno scheduler/worker-e.

Ovo je dobar izbor ako želiš stabilnu i prenosivu instalaciju.

### Primer deployment-a za Work Manager aplikaciju

Recimo da si napravio aplikaciju `work_manager` i želiš je staviti na server.

Tok bi bio:

1. napraviš bench,
2. dodadeš aplikaciju u bench,
3. napraviš site, npr. `workmanager.example.com`,
4. instaliraš aplikaciju na taj site,
5. uradiš migraciju,
6. podesiš Nginx i domen,
7. pokreneš scheduler,
8. proveriš da li aplikacija radi kroz browser.

### Deployment checklist

Kad želiš da napraviš prvi ozbiljan deployment, ovaj checklist ti može pomoći:

- [ ] server je pripremljen i pristupačan,
- [ ] Python, bench i potrebni paketi su instalirani,
- [ ] baza podataka i Redis su dostupni,
- [ ] bench je inicijalizovan,
- [ ] aplikacija je dodata u bench,
- [ ] site je kreiran,
- [ ] aplikacija je instalirana na site,
- [ ] migracija je urađena,
- [ ] Nginx je konfigurisan,
- [ ] SSL i domen su povezani,
- [ ] scheduler i worker-i su pokrenuti,
- [ ] backup sistema postoji,
- [ ] aplikacija je proverena kroz browser.

### Zaključak

Deployment je ono što pravi razliku između "imam aplikaciju" i "aplikacija radi za korisnike".

Za Frappe to znači da ne radiš samo razvoj, nego i:

- pripremu servera,
- instalaciju bench-a,
- kreiranje site-a,
- migracije,
- pokretanje servisa,
- i održavanje produkcije.

To je poslednji korak u razvoju aplikacije, a u isto vreme jedan od najvažnijih.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
