
# Konfiguracija

[Sadržaj](00_sadrzaj.md)

Refleks aplikacije se mogu konfigurisati pomoću

- konfiguracione datoteke,
- promenljivih okruženja i
argumenata komandne linije.

## Konfiguraciona datoteka

Pokretanje `reflex init` komande će kreirati `rxconfig.py` datoteku u vašem korenskom direktorijumu projekta. Odavde možete proslediti ključne argumente klasi `Config` da biste konfigurisali svoju aplikaciju.

Na primer:

```py
# rxconfig.py
import reflex as rx

config = rx.Config(
    app_name="my_app_name",

    # Connect to your own database.
    db_url="postgresql://user:password@localhost:5432/my_db",

    # Change the frontend port.
    frontend_port=3001,
)
```

Pogledajte referencu za konfiguraciju za sve dostupne parametre.

## Promenljive okruženja

Možete zameniti konfiguracionu datoteku podešavanjem promenljivih okruženja. Na primer, da biste zamenili podešavanje `frontend_port`, možete podesiti promenljivu okruženja `FRONTEND_PORT`.

```sh
FRONTEND_PORT=3001 reflex run
```

## Argumenti komandne linije

Konačno, možete prepisati konfiguracionu datoteku i promenljive okruženja tako što ćete proslediti argumente komandne linije u `reflex run`.

```sh
reflex run --frontend-port 3001
```

Pogledajte CLI referencu za sve dostupne argumente.

## Prilagodljivi direktorijum podataka aplikacije

Promenljiva okruženja `REFLEX_DIR` može se podesiti, što omogućava korisnicima da podese lokaciju gde Reflex piše pomoćne alate poput Bun i NodeJS.

Podrazumevano koristimo direktorijume specifične za platformu:

- Na Windows-u `C:/Users/<username>/AppData/Local/reflex` se koristi.
- Na macOS-u `~/Library/Application Support/reflex` se koristi.
- Na Linuksu `~/.local/share/reflex` se koristi.

[Sadržaj](00_sadrzaj.md)
