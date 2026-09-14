
# Struktura projekta

[Sadržaj](00_sadrzaj.md)

## Struktura direktorijuma

Kreirajmo novu aplikaciju pod imenom "hello".

```sh
mkdir hello
cd hello
reflex init
```

Ovo će kreirati strukturu direktorijuma:

```sh
hello
├── .web
├── assets
├── hello
│   ├── __init__.py
│   └── hello.py
└── rxconfig.py
```

Idemo preko svake stvake ovih direktorijuma i datoteka.

- **.web**

  Ovde će se sačuvati kompajlirane JavaScript datoteke. Nikada nećete trebati dodirnuti ovaj direktorijum, ali može biti korisno za uklanjanje pogrešaka.
  
  Svaka Refleks strana kompajliraće se u odgovarajuću `.js` datoteku u imeniku `.web/pages`.

- **assets**

  Direktorijum `assets` je mesto gde možete da čuvate bilo koju statičku imovinu koju želite da bude javno dostupna. Ovo uključuje slike, fontove i druge datoteke.
  
  Na primer, ako sačuvate sliku u `assets/image.png`, možete ga prikazati  iz aplikacije ovako:
  
  ```py
  rx.image(src="/image.png")
  ```

- **hello - glavni projektni direktorijum**

  Inicijalizacija vašeg projekta stvara direktorijum sa istim imenom kao i vaša aplikacija. Ovde ćete pisati logiku svoje aplikacije.
  
  Refleks generiše zadanu aplikaciju u okviru `hello/hello.py` datoteke. Možete da izmenite ovu datoteku da biste prilagodili svoju aplikaciju.
  
- **Konfiguracija**

  `rxconfig.py` datoteka se može koristiti za konfigurisanje vaše aplikacije. Podrazumevano izgleda ovako nekako:
  
  ```py
  import reflex as rx
  
  config = rx.Config(
      app_name="hello",
  )
  ```

Komentarisaćemo strukturu projekata i konfiguracije detaljnije u naprednoj dokumentaciji strukture projekta.

[Sadržaj](00_sadrzaj.md)
