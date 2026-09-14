
# Instalacija

[Sadržaj](00_sadrzaj.md)

> [!Note]  
> Reflex zahteva Python 3.10+.

## Razvojno okruženje

Toplo preporučujemo da kreirate `virtuelno okruženje` za vaš projekat.

`venv` je standardna opcija. `Conda` i `poetry` su neke alternative.

- **Kreiranje projektnog direktorijuma**

  Zamenite `my_app_name` sa imenom vašeg projekta. Pređite u novi direktorijum.

  ```sh
  mkdir my_app_name
  cd my_app_name
  ```

- **Kreiranje i podešavanje virtual environmenta**

  ```sh
  python3 -m venv .venv
  source .venv/bin/activate
  ```

- **Instalacija Reflex paketa**

  Reflex je raspoloživ kao `pip` paket.

  ```sh
  pip install reflex
  ```

- ** Inicializacija projekta

  ```sh
  reflex init
  ```

  Komanda će vratiti nekolko opcije šablona za izbor iz navedenog u nastavku:

- **Inicijalizacija veb drektorijuma**

  ```sh
  Get started with a template:
  (0) blank (https://blank-template.reflex.run) - A minimal template
  (1) dashboard (https://dashboard-new.reflex.run/) - A dashboard with tables and graphs
  (2) sales (https://sales-new.reflex.run/) - An app to manage sales and customers
  (3) ai_image_gen (https://ai-image-gen.reflex.run/) - An app to generate images using AI
  (4) ci_template (https://cijob.reflex.run/) - A template for continuous integration
  (5) api_admin_panel (https://api-admin-panel.reflex.run/) - An admin panel for an api.
  (6) nba (https://nba-new.reflex.run/) - A data visualization app for NBA data.
  (7) customer_data_app (https://customer-data-app.reflex.run/) - An app to manage customer data.
  Which template would you like to use? (0): 
  ```
  
  Odavde izaberite šablon.

- **Pokretanje aplikacije**

  Pokretanje u dev modu:
  
  ```sh
  reflex run
  ```
  
  Vaša aplikacija je pokrenuta na <http://localhost:3000>.
  
  Refleks štampa na terminal. Da biste povećali verbosity dnevnika za pomoć u vezi sa uklanjanjem pogrešaka, koristite `--loglevel` zastavu:
  
  ```sh
  reflex run --loglevel debug
  ```
  
  Refleks će na vruće ponovno uvesti bilo koju promenu koda u realnom vremenu prilikom rada u režimu razvoja. Izmena vašeg koda prikazaće se na <http://localhost:3000> automatski.
  
[Sadržaj](00_sadrzaj.md)
