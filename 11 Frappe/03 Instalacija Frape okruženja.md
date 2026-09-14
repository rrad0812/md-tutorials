# Frappe Framework tutorijal

[Sadržaj][00]

## 03 Instalacija Frappe okruženja

- **Instalacija git**

  ```sh
  sudo apt install git -y
  ```

- **Instalacija Redisa**

  ```sh
  sudo apt install redis-server -y
  ```

- **Instalacija PostgreSQL**

  ```sh
  sudo apt install postgresql postgresql-contrib libpq-dev -y
  ```

  - Provera instalirane verzije
  
    ```sh
    psql --version
    ```

  - Prelazak na `postgres` nalog  

    To je podrazumevani admin korisnik PostgeSQL i instaliran je na sistem za vreme instalacije PostgrSQL-a.

    ```sh
    sudo -i -u postgres
    ```

  - Pokretanje PostgreSQL shela

    ```sh
    psql
    ```

  - Izlazak iz shela:

    ```sh
    \q
    ```

  - Na psql možeš doći kao `postgres` bez prelaska sa svog naloga:

    ```sh
    sudo -u postgres psql
    ```

  - Dodela passworda postgres korisniku:

    ```sql
    ALTER USER postgres WITH PASSWORD 'postgres_password';
    ```
  
- **Instalacija web servera - nginx**

  ```sh
  sudo apt install nginx
  ```

- **Instalacija wkhtmltopdf**
  
  - Instalacija zavisnosti

    ```sh
    sudo apt install xvfb libfontconfig
    ```

  - Preuzmi wkhtmltopdf paket sa <https://wkhtmltopdf.org/downloads.html>,
    potom pokreni sledeću komandu za instalaciju:

    ```sh
    sudo apt install ./wkhtmltox_file.deb
    ```
  
- **Instalacija frontend zavisnosti**

  - Instalacija **nvm**

    ```sh
    curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.3/install.sh | bash
    ```

  - Instalacija **nodejs**

    ```sh
    nvm install 24
    ```

    - Provera instaliranosti
  
      ```sh
      node -v
      ```

  - Instalacija **yarn**

    ```sh
    npm install -g yarn
    ```
  
- **Instalacija Bench-a (Frappe v15)**  

  ```sh
  uv tool install frappe-bench --with setuptools
  ```

  **Napomena**:  
  Dodali smo sa `--with setuptools` jer Frappe u pozadini još uvek koristi neke starije Python mehanizme za pakete.
  
  - Sada proveri da li sistem vidi komandu:

    ```sh
    bench --version
    ```

- **Instalacija i inicijalizacija frappe-bench direktorijuma**

  Sada kada imamo bench alat, pravimo glavni folder gde će ti biti svi sajtovi i aplikacije (izaberi verziju 15):
  
  ```sh
  bench init frappe-bench --frappe-branch version-15
  ```
  
  Uđi u kreirani direktorijum:
  
  ```sh
  cd frappe-bench
  ```
  
  Sada se nalaziš u glavnom upravljačkom čvorištu svog projekta.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
