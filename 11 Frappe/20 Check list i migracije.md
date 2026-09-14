# Frappe Framework tutorijal

[Sadržaj][00]

## 20 Check list i migracije

Ovo poglavlje je kratki operativni vodič: kada pokrećeš migraciju, šta Frappe radi i kako da brzo proveriš da li je sve zdravo.

### Kada pokrećeš migraciju

Pokreni migraciju svaki put kada menjaš model ili preuzimaš izmene koje menjaju model.

1. Posle izmene DocType strukture: novo polje, promena tipa polja, obaveznost, novi DocType.
2. Posle pull-a izmena koje uključuju DocType JSON, patch-eve ili schema promene.
3. Posle update-a app-a na sajtu.
4. Pre testova, ako je model menjan od poslednjeg pokretanja.

### Šta Frappe radi tokom migracije

Migracija je usklađivanje stanja baze sa stanjem koda.

1. Učita DocType definicije i metadata izmene iz app-a.
2. Uskladi tabele i kolone u bazi sa DocType strukturom.
3. Izvrši patch-eve (jednokratne skripte za data/schema promene).
4. Osveži cache i metadata reference.
5. Pripremi sistem da UI i backend vide nova pravila i polja.

### Osnovna komanda

```bash
bench --site <tvoj_sajt> migrate
```

### Pre migrate check lista

Pre pokretanja migracije proveri ove tačke:

1. Exportovao si DocType izmene iz Desk-a u app fajlove.
2. Nema otvorenih konflikata u JSON/Python fajlovima.
3. Znaš koje novo obavezno polje uvodiš i kako postojeći podaci to preživljavaju.
4. Imaš backup ako je u pitanju važna ili produkciona baza.
5. Radiš na pravom site-u.

### Posle migrate check lista

Odmah posle migracije proveri:

1. Komanda je završila bez error poruke.
2. U Desk-u se vide nova polja i očekivani DocType.
3. Kreiranje jednog testnog dokumenta prolazi kroz osnovni tok.
4. Validacije rade kao što je planirano.
5. Pokretanje test modula prolazi.

### Brzi test posle migracije

```bash
bench --site <tvoj_sajt> run-tests --module work_manager.work_manager.doctype.wm_task.test_wm_task
```

Po potrebi pokreni i Time Entry test modul:

```bash
bench --site <tvoj_sajt> run-tests --module work_manager.work_manager.doctype.wm_time_entry.test_wm_time_entry
```

### Najčešće greške i kako da ih prepoznaš

1. Missing mandatory field u starim zapisima.
Linija greške obično pominje koji field je obavezan; dodaj default ili data patch.

2. Status vrednosti ne odgovaraju Select opcijama.
Tipično vidiš ValidationError pri insert/save i poruku oko statusa.

3. Patch pao zbog pretpostavke o podacima.
U traceback-u traži naziv patch fajla i proveri uslov koji nije ispunjen.

4. Promenjen field type bez plana za postojeće podatke.
Greška često pominje cast/convert; reši kroz bezbedan prelaz ili patch.

### Preporučeni mini workflow

U praksi koristi ovaj redosled:

1. Izmena modela u DocType-u.
2. Export promena.
3. Migrate.
4. Jedan ručni smoke test kroz Desk.
5. Jedan automatski test modul.

Ako ovo pratiš, većina problema se hvata rano i lako se rešava.

### Migracija u produkciji bez stresa

Za produkciju koristi ovih 5 pravila svaki put.

1. Backup pre bilo čega.
Napravi svež backup baze i fajlova pre pull/migrate koraka.

2. Kratak maintenance window.
Planiraj migraciju u periodu manjeg opterećenja i obavesti korisnike unapred.

3. Jasna verzija i plan promena.
Tačno zapiši koji commit/deploy ide i koje DocType/schema promene očekuješ.

4. Rollback plan unapred.
Pre starta odluči kako vraćaš stanje ako migracija padne: backup restore i povratak na prethodnu verziju koda.

5. Post-migrate smoke test odmah.
Odmah proveri login, list view, create/save za ključne DocType-ove i barem jedan kritičan izveštaj.

Minimalni produkcijski redosled:

1. Backup.
2. Pull deploy verzije.
3. bench --site <tvoj_sajt> migrate
4. Smoke test.
5. Tek onda otvori sistem korisnicima.

Ako migrate padne, ne improvizuj na brzinu.

1. Sačuvaj traceback i tačan korak gde je stalo.
2. Vrati bazu i kod na poslednje stabilno stanje.
3. Ispravi problem na staging-u.
4. Ponovi deploy tek kada staging migracija prođe čisto.

### Staging pre produkcije check lista

Pre svakog produkcionog deploy-a prođi ovih 6 tačaka na staging okruženju:

1. Staging je na istom commit-u kao planirani produkcioni deploy.
2. Pokrenut je `bench --site <staging_sajt> migrate` bez grešaka.
3. Prošli su ključni test moduli (`wmTask` i `wmTime Entry`).
4. Proveren je barem jedan end-to-end poslovni tok (project -> task -> time entry).
5. Provereni su role i permission tokovi za glavne korisničke uloge.
6. Zabeležen je rezultat provere (ko, kada, koji commit, šta je testirano).

Ako bilo koja tačka padne, produkcioni deploy se pauzira dok staging ne bude zelen.

### Template za deploy zapisnik

Koristi ovaj zapis posle svakog staging/prod deploy-a.

```text
Datum i vreme:
Okruženje (staging/production):
Site:
Commit / tag:
Odgovorna osoba:

Pre-check:
- Backup uradjen: DA/NE
- Staging migrate prosao: DA/NE
- Kljucni testovi prosli: DA/NE

Komande koje su pokrenute:
1)
2)
3)

Rezultat migracije:
- Status: USPESNO / NEUSPESNO
- Trajanje:
- Kratka napomena:

Smoke test rezultat:
- Login: OK/FAIL
- Create Task: OK/FAIL
- Create Time Entry: OK/FAIL
- Kljucni izvestaj: OK/FAIL

Ako je bilo problema:
- Error poruka (sazetak):
- Rollback uradjen: DA/NE
- Sledeci korak:
```

Minimalna praksa: cuvaj ovaj zapis uz svaki deploy commit ili u internom runbook dokumentu.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
