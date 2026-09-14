
# Frappe Framework tutorijal

[Sadržaj][00]

## Instalacija third-party aplikacije

Što se tiče tvog pitanja: da, instalacija third-party open source aplikacija je možda baš sledeća dobra lekcija.

U Frappe svetu se to radi preko `bench get-app` sistema.

Recimo da hoćeš neku javnu aplikaciju:

Primer:

```sh
cd ~/frappe-bench
bench get-app https://github.com/neki-user/neka-app.git
```

Ovo preuzima aplikaciju u:

```txt
apps/
   neka-app/
```

Zatim instalacija na tvoj site:

```bash
bench --site site1.local install-app neka_app
```

i provera:

```sh
bench --site site1.local list-apps
```

trebalo bi:

```txt
frappe
frappe_lab
work_manager
neka_app
```

Posle:

```sh
bench restart
```

ili ako si u developer modu:

```sh
bench start
```

Ali ovde dolazi veoma važna lekcija:

Frappe app nije samo Python paket.

Kada instaliraš app, on može da donese:

```txt
    neka_app
    ├── doctypes
    │    └── nove poslovne tabele
    ├── fixtures
    │    └── unapred definisana podešavanja
    ├── hooks.py
    ├── reports
    ├── pages
    ├── patches
    └── permissions
```

Znači instalacija aplikacije može promeniti:

- bazu,
- DocType-e,
- role,
- menije,
- Desk,
- background procese.

Zato je Frappe app mnogo bliža "pluginu za poslovni sistem" nego običnom Python paketu.

Za tvoju trenutnu fazu ja bih predložio jednu vrlo zanimljivu vežbu:

Uzmi neku malu open-source Frappe aplikaciju i uradi:

- `bench get-app`

- pogledaj strukturu:

  ```txt
  apps/neka_app/
  ```

- instaliraj je na `site1.local`

- pogledaj:
  - koje DocType je dodala,
  - koje hook-ove ima,
  - koji Workspace je napravila,
  - šta je promenila u bazi.

Par poznatijih open-source Frappe aplikacija koje možeš pogledati:

- ERPNext — najveći primer Frappe aplikacije (ceo ERP izgrađen na Frappe-u)
- Frappe HR — HR sistem kao posebna Frappe aplikacija
- Frappe Helpdesk — ticketing/helpdesk aplikacija

Sve su javno dostupne i upravo su dobar materijal za proučavanje arhitekture.

I slažem se sa tvojom završnom procenom: Frappe ima mnogo urađenih stvari i vrlo ozbiljnu infrastrukturu, ali nosi i određene filozofske odluke koje nisu za svakoga. Kod ovakvih framework-a nije pitanje da li su "dobri" ili "loši", nego da li se njihov način razmišljanja poklapa sa problemima koje rešavaš.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
