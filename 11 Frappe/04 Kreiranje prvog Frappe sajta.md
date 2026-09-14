# Frappe Framework tutorijal

[Sadržaj][00]

## 04 Kreiranje prvog Frappe sajta

Pošto koristimo PostgreSQL, moramo da uradimo jednu brzu pripremu pre same komande za kreiranje sajta. Frappe sa PostgreSQL-u zahteva posebne ekstenzije (poput `pg_trgm` i `btree_gin`). Da bi ih Frappe-ov korisnik uspešno kreirao tokom instalacije, najsigurnije je da ih tvoj `postgres` superuser ima već aktivne na sistemskom nivou (u šablonu `template1` iz kojeg se prave sve nove baze).

- **Priprema PostgreSQL šablona baze podataka**

  Pokreni ovu komandu da omogućiš potrebne ekstenzije u podrazumevanom šablonu baze:
  
  ```sh
  sudo -u postgres psql -d template1 -c "CREATE EXTENSION IF NOT EXISTS pg_trgm; 
  CREATE EXTENSION IF NOT EXISTS btree_gin;"
  ```
  
- **Kreiranje novog sajta**

  Sada, dok si unutar "frappe-bench" direktorijuma, pokreni komandu za kreiranje sajta. Nazvaćemo
  ga npr. "site1.local" (možeš staviti bilo koje ime koje se završava sa .localhost ili .local za
  lokalni razvoj):

  ```sh
  bench new-site site1.local --db-type postgres
  ```
  
  Tokom izvršavanja ove komande, Bench će te pitati za dve stvari:
  
  - **MySQL root password**  
    Iako piše MySQL, pošto si stavio `--db-type postgres`, ovde unosiš lozinku za `postgres` korisnika koju si postavio u Fazi 1.
  
  - **Administrator password**  
    Ovo je lozinka za glavnog admina samog Frappe web interfejsa. Postavi neku lozinku po želji (npr. admin123 za lokalni rad) i zapamti je.
  
    Kako da proveriš da li je uspelo? Kada komanda završi, proveri da li baza zaista postoji u PostgreSQL-u:
  
    ```sh
    sudo -u postgres psql -c "\l"
    ```
  
    Trebalo bi da vidiš novu bazu sa čudnim, nasumičnim imenom koje počinje sa podvlakom (npr. _1a2b3c4d5e...), jer Frappe namerno generiše hešovana imena baza radi bezbednosti.
  
> [!Note]
> **Rešenje problema sa konekcijom na PostgreSQL**
>
> Moramo da dozvolimo svim korisnicima (`all`) da se povežu na sve baze (`all`) preko 127.0.0.1
> (localhost-a) interfejsa koristeći lozinku.
>
> Podrazumevano, PostgeSQL je podešen na `peer` konkcije (`socket`) sa localhosta, i to je način na
> koji pristupa psql. Za Frappe pristup, preko mrežnog interfejsa, potrebno je prepodesiti u `/etc/
> postgresql/16/main/pg_hba.conf` konfiguracionm fajlu IPv4 pravilo, tako da glasi:
>
> ```txt
> # IPv4 local connections:
> host    all             all             127.0.0.1/32            md5
> ```
>
> Promenili smo tip protkola lozinke, jer se u praksi pokazalo da radi!
>
> **Restart PostgreSQL servisa**
>
> Posle promene konfig. fajllova, restartuj PostgreSQL servis:
>
> ```sh
> sudo systemctl restart postgresql
> ```

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
