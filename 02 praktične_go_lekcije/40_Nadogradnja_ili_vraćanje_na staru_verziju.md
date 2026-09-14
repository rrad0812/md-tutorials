
# 40 Nadogradnja ili vraćanje na stariju verziju programa Go

[39 OOP jezik][39] | [00 Sadržaj][00] | [41 Preporuke za dizajn][41]

**Šta ćete naučiti u ovom poglavlju?**

- Kako nadograditi (ili smanjiti) verziju programa Go na računaru.

**Obrađeni tehnički koncepti**:

- SHA256

## Provera vaše verzije

Da biste dobili trenutno instaliranu verziju Go-a, samo otkucajte:

```sh
go version
```

Ovo će izvesti nešto poput ovoga:

```sh
go version go1.11.1 darwin/amd64
```

Verzija je odštampana sa referencom vašeg sistema (u mom slučaju Apple MacOS)

## Gde je vaš binarni fajl Go

Da biste nadogradili sistem na novu verziju programa Go, morate da pronađete gde se Go nalazi na vašem sistemu.

```sh
which go
```

`which` će locirati programsku datoteku u korisničkoj putanji (uporedi `man which`)

Obično (korisnici Linuksa i Meka), ako ovo otkucate na svom sistemu, prikazaće se:

```sh
/usr/local/go/bin/go
```

Za korisnike Windows-a, putanja gde se čuvaju binarne datoteke go-a je obično.

```sh
C:\Go\bin
```

Da biste bili sigurni, možete pokrenuti komandu:

```sh
c:\> where go
```

Uspešno smo locirali binarne datoteke; sada možemo da ih nadogradimo.

## Preuzmite novu verziju

Nove verzije Goa se nalaze na veb-sajtu golang.org: <https://golang.org/dl/>. Uvek preuzimajte svoju Go verziju sa ove zvanične URL adrese.

Da biste ažurirali trenutnu verziju, preuzmite ciljanu verziju sa ove veb stranice.

Da biste bili sigurni da ste preuzeli validnu binarnu datoteku, morate izračunati SHA256 sumu preuzete datoteke. Zatim morate da se uverite da je ona strogo jednaka onoj koja je navedena na veb lokaciji.

**Na macOS-u**:

Da biste proverili sha256 sumu, otvorite terminal i otkucajte:

```sh
shasum -a 256 go1.11.2.linux-amd64.tar.gz
```

Zastavica „a“ se koristi da naznači koji algoritam treba koristiti (ovde je to SHA256)

**Na Linuksu**:

Komanda je kraća:

```sh
sha256sum go1.11.2.linux-amd64.tar.gz
```

**Na Windows-u**:

Možete koristiti certutil da biste to uradili u komandnoj liniji

```sh
certutil -hashfile go1.11.2.linux-amd64.tar.gz SHA256
```

## Instalirajte (novu/staru) verziju

**Na macOS-u**:

Preuzeli ste instalater. On će automatski instalirati izabranu verziju. Samo dvaput kliknite na preuzetu datoteku.

**Na Linuksu**:

- **Raspakivanje arhive**

  Prvo, morate raspakovati preuzetu arhivu sledećom komandom:
  
  ```sh
  sudo tar -C /usr/local -xzf go1.11.2.linux-amd64.tar.gz
  ```
  
  Izdvojiće sadržaj datoteke tar.gz u "/usr/local"

  Direktorijum bin sadrži dve izvršne datoteke u trenutku pisanja ovog teksta:

  - go
  - gofmt

- **Modifikacija putanje**:

  Da biste mogli da pokrenete komande go i gofmt, potrebno je da dodate ovaj direktorijum u svoju putanju (PATH). Da biste to uradili, dodajte ovu liniju na vrh datoteke ".bashrc" koja se nalazi u vašem početnom direktorijumu (ili ".zshrc" ako koristite zsh):
  
  ```sh
  export PATH=$PATH:/usr/local/go/bin
  ```
  
  Nakon toga, moraćete ponovo pokrenuti terminal da bi promena stupila na snagu.

**Na Windows-u**:

Preuzeli ste instalater. On će automatski instalirati izabranu verziju. Samo dvaput kliknite na preuzetu datoteku.

[39 OOP jezik][39] | [00 Sadržaj][00] | [41 Preporuke za dizajn][41]

[39]: 39_Objektno_orijentisani_programski_jezik.md
[00]: 00_Sadržaj.md
[41]: 41_Preporuke_za_dizajn.md
