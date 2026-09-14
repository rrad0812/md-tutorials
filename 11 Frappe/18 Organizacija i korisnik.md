# Učenje Frappe Framework

[Sadržaj][00]

## 18 Organizacija i korisnik

---

Mučim se oko unutrašnje organizacije. Da li je ovakvo razmišljanje OK?

- Kreiranjem DocType, odrđujemo njegovu strukturu, pisanjem koda i ponašanje. Skoro kao klasa.
- Model se implicitno određuje i samim time kreira ako ne postoji prilikom kreiranja DocTypea.
- Workspace je način kako obezbedjujemo Look End Fill krajnjem korisniku.
- Svaki modul ? i/ili DocType ? mora biti dodat da bi se pojavio kao link u workspace-u.
- Ako je sve ovo tačno, nije mi jasno zašto je adresna struktura nehomogena, tj ne vidim kom linku
  pristupa korisnik kada želi da poče da radi sa aplikacijom?  
  Tada se valjda pojavljuje Logon prozor itd...

---

Da, razmišljanje ti je veoma dobro. Rekao bih 85-90% tačno, uz par važnih nijansi.

1. Kreiranjem DocType određuješ strukturu i ponašanje, skoro kao klasa.
   Tačno. U Frappe-u je DocType istovremeno:

   - metadata model (polja, tipovi, permisioni),
   - UI definicija forme/liste,
   - server objekat (Python kontroler),
   - API resurs.

2. Model se implicitno kreira ako ne postoji.  
   Uglavnom tačno, ali praktično: tabela i kolone se sigurno usklađuju kroz migrate/sync mehanizam (najčešće posle save + migrate u dev toku). Dakle, nije “ručni SQL model”, nego Frappe sync.

3. Workspace je Look and Feel za korisnika.
   Delimično tačno. Preciznije:

   - Desk daje globalni look and feel,
   - Workspace je početna navigaciona površina za određeni poslovni domen (shortcut-i, chart-ovi, linkovi, brojke).

4. Svaki modul/DocType mora biti dodat da bi bio link u Workspace-u.
   Tačno za vidljivost unutar tog konkretnog Workspace-a.
   Nije tačno da mora da bude u Workspace-u da bi “postojao” u sistemu.
   DocType može postojati i biti dostupan kroz search/Awesome Bar/direct route, ako korisnik ima dozvole.

5. Zašto je adresna struktura “nehomogena”?
   Odlično pitanje. Imaš 2 različita sveta:

   - filesystem struktura (apps, moduli, doctype folderi),
   - runtime URL struktura (Desk rute za korisnika).

   Oni namerno nisu 1:1 isti.

   Mentalna mapa:
   - Kod i fajlovi: tehnička organizacija developera.
   - URL rute: korisnička navigacija kroz Desk.

**Tipičan tok korisnika**:

- Otvori site URL.
- Ako nije ulogovan, ide na login ekran.
- Posle logina ulazi u Desk, tipično na /app.
- Odatle ide na Workspace ili direktno na DocType list/formu (preko linka, search-a ili bookmark-a).

Zato ti deluje “nejasno kom linku korisnik pristupa”: ulazni link je skoro uvek /app (nakon logina), a dalje navigaciju određuju Workspace + dozvole + shortcut-i.

### Mini mapa rutiranja za Work Manager

Evo mini routing mape baš za tvoj Work Manager, praktično i bez teorije viška.

Osnovna ideja:

- Fajl struktura služi developeru.
- URL ruta služi korisniku.
- Ne moraju biti 1:1 iste, i to je normalno u Frappe.

Mini routing mapa (tipičan tok):

1. Ulaz na sajt: `<https://tvoj-site>`

2. Ako korisnik nije prijavljen: preusmerenje na `login` ekran (forma za prijavu)

3. Posle uspešnog logina ulaz u Desk na: `/app`

4. Iz Desk-a korisnik ide na Workspace  
   Primer: `/app/workspace/work-manager`  
   (naziv na kraju je slug Workspace-a)

5. Iz Workspace-a klik na shortcut vodi na DocType listu  
   Primer za projekat:
   `/app/wmproject`  
   primer za task:
   `/app/wmTask`

6. Klik na konkretan zapis vodi na formu tog zapisa  
   Primer:
   `/app/wmproject/WMP-00001`  
   `/app/wmtask/TASK-00012`

7. Novi zapis se obično otvara kroz `+Add DocType` u listi i prelazi na formu novog dokumenta

Kako to povezati sa tvojim pitanjima:

1. DocType je `centralna` jedinica: model + forma + lista + API + permissions.
2. Workspace nije “obavezna tehnička stepenica”, već UX ulaz za korisnika.
3. DocType može postojati i bez shortcut-a u Workspace-u, ali je tada manje vidljiv korisniku.
4. Zato korisnik “ne pogađa” putanje iz foldera, nego ulazi kroz:
   `login -> app -> workspace -> link/doc`.

Ako želiš, sledeće mogu da ti napišem i mini mapu “koji fajl na disku utiče na koji ekran u Desk-u” (1:1 cheat sheet).

### Mini mapa 1:1: fajl na disku -> ekran u Desk-u

Ispod je praktična mapa za Work Manager aplikaciju.

| Fajl/objekat na disku | Na sta utice | Gde korisnik to vidi |
| --- | --- | --- |
| `apps/work_manager/work_manager/modules.txt` | Koji moduli postoje u aplikaciji | Indirektno kroz Module/Workspace organizaciju |
| `.../doctype/wmproject/wmproject.json` | Definicija DocType-a: polja, dozvole, prikaz | Forma i lista na ruti `/app/wmproject` |
| `.../doctype/wmproject/wmproject.py` | Server logika (`validate`, hook metode) | Korisnik vidi posledicu pri Save/Submit (validacije, greske) |
| `.../doctype/wmproject/wmproject.js` | Client logika forme | Korisnik vidi ponasanje na formi (auto-popuna, dugmad, upozorenja) |
| `.../doctype/wmproject/wmproject_list.js` | List view podesavanja | Kolone i indikatori na listi `/app/wmproject` |
| Workspace DocType zapis (u Desk-u, eksport JSON u app) | Shortcut-i, kartice, sekcije | Pocetni ekran Workspace-a, npr. `/app/workspace/work-manager` |
| `work_manager/hooks.py` | Integracija aplikacije (events, scheduler, fixtures, override) | Uglavnom indirektno; korisnik vidi kroz funkcionalnost koja se aktivira |

Napomena o rutama:

1. Korisnik gotovo uvek ulazi kroz `/app` (posle logina).
2. Workspace je navigacioni sloj, ne "vlasnik" DocType-a.
3. DocType moze da radi i bez shortcut-a u Workspace-u, ako korisnik ima dozvole i zna rutu.

### Brzi flow za proveru

1. Menjas `wmproject.json` (npr. dodas polje).
2. Pokrenes migrate/sync u bench okruzenju.
3. Otvoris `/app/wmproject` i proveris da li se promena vidi u listi/formi.
4. Ako menjas `wmproject.js` ili `wmproject_list.js`, uradis `clear-cache` i refresh.

Ako ovako razdvojis "fajl koji menjam" i "ekran koji ocekujem", navigacija kroz Frappe postaje mnogo jasnija.

### Mini dijagnostika: promena se ne vidi u UI

Kada uradis izmenu, a u Desk-u deluje kao da se nista nije promenilo, idi ovim redosledom.

1. Proveri da li gledas pravi site

   - U bench okruzenju potvrdi da radis nad pravim sajtom (`site1.local` ili drugi).
   - Ako imas vise sajtova, najcesca greska je da menjas app na jednom, a gledas drugi.

2. Proveri da li je metadata sinhronizovana

   - Ako si menjao DocType strukturu, pokreni `migrate/sync`.
   - Bez toga, JSON i baza mogu da budu van sinhronizacije.

3. Proveri cache i asset-e

   - Za JS/list view izmene (`*_list.js`, `*.js`) uradi `clear-cache`.
   - Zatim osvezi browser (hard refresh) da ucita nove asset-e.

4. Proveri permissions

   - Ako polje ili DocType ne vidis, cesto je problem u `role/permission` pravilima.
   - Proveri da li korisnik ima `read` prava nad DocType-om i poljem.

5. Proveri Workspace sadrzaj

   - DocType moze da postoji, ali da nema shortcut u Workspace-u.
   - Dodaj shortcut ili otvori direktnu rutu (`/app/wmproject`) radi potvrde.

6. Proveri da li je problem u nazivu rute

   - Route koristi DocType `naming`, ne naziv foldera.
   - Ako sumnjas, nadji DocType preko `Awesome Bar` pretrage pa otvori odatle.

7. Proveri server validacije

   - Ako forma ne cuva izmene, gledaj validacije iz `*.py` kontrolera.
   - `frappe.throw(...)` moze da blokira save i ostavi utisak da UI "ne radi".

8. Proveri da li je promena na pravom mestu

   - Forma ponasanje: `wmdoc.js`
   - Lista kolone/indikatori: `wmdoc_list.js`
   - Struktura polja i permisioni: `wmdoc.json`
   - Poslovna pravila: `wmdoc.py`

Pravilo koje najvise stedi vreme:

- Ako je promena u metadata/modelu -> prvo `migrate`.
- Ako je promena u klijentskom JS-u -> prvo `clear-cache` + `hard refresh`.
- Ako je promena u vidljivosti -> prvo `permissions` + `workspace shortcut`.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
