
# 03 Terminal

[02 Jezik Go][02] | [00 Sadržaj][00] | [04 Podesite svoje okruženje][04]

**Šta ćete naučiti u ovom poglavlju?**

- Šta su šel, bash i terminal?
- Kako pokrenuti terminalnu aplikaciju?
- Kako izdavati osnovne komande pomoću bash-a?
- Kako poboljšati svoje "iskustvo za programere" na Windows-u?

**Obrađeni tehnički koncepti!**

- GUI: Grafički korisnički interfejs
- CLI: Interfejs komandne linije
- Školjka
- Bash
- WSL: Windows podsistem za Linux

## Uvod

Svakodnevni život Go programera zahteva korišćenje terminala. Ovo poglavlje će objasniti šta je terminal i kako se koristi. Ako ste već upoznati sa njim, možete preskočiti ovo poglavlje.

## Grafički korisnički interfejs (GUI)

Operativni sistemi poput macOS-a, Windows-a ili Linux-a nude bogate grafičke korisničke interfejse (GUI). Da biste pokrenuli program instaliran na vašem računaru, obično dvaput kliknete na ikonu koja se nalazi na radnoj površini. Programi će biti prikazani u prozorima sa interaktivnim interfejsom: meniji, bočne trake, dugmad...

Kažemo da ti programi nude grafički korisnički interfejs (GUI). Velika većina korisnika će koristiti programe koji pružaju ove interfejse. Grafički interfejsi su jednostavni za korišćenje i intuitivni.

## Interfejs komandne linije (CLI)

Grafički korisnički interfejsi nisu oduvek postojali. Prvi računari nisu imali takve mogućnosti. Ali kako su korisnici tih računara uspevali da pokreću i koriste programe? Računari su isporučeni sa interfejsom komandne linije. Ovaj interfejs se takođe naziva "školjka". Shell je program koji može da prosleđuje komande operativnom sistemu. Shell je generički termin koji označava takve programe. Najpoznatija školjka je `bash` (Bourne Again Shell). Bash se podrazumevano isporučuje sa macOS-om i velikom većinom Linux distribucija. Windows se takođe podrazumevano isporučuje sa školjkom (ali to nije bash).

### Kako komunicirati sa školjkom: terminal

Na starim računarima, školjka se prikazivala korisniku direktno nakon pokretanja. Na modernim računarima, moramo pokrenuti program da bismo komunicirali sa školjkom. Ovaj program se često naziva terminal. Videćemo kako da otvorimo terminal na MacO-u, Linux-u (GNOME) i Windows-u.

#### macOS

- Otvorite aplikaciju Finder
- Zatim u traci menija kliknite na "Go", a zatim na "Utilities"
- Otvoriće se prozor. Kliknite na "Terminal"
- Otvoriće se novi terminal
  Imajte na umu da koristim prilagođeni terminal. Stoga možete videti drugačiji izlaz; to je sasvim normalno.

### Linux (Ubuntu)

- Na Ubuntuu možete koristiti prečicu `Ctrl+Alt+T`
- Terminal možete pokrenuti i pomoću Ubuntu Dash-a. Unesite "Terminal" i aplikacija bi trebalo da se pojavi.

### Windows

- Kliknite na dugme "Start", a zatim u tekstualno polje unesite "cmd" (za komandnu liniju).
- Zatim kliknite na cmd aplikaciju.
- Trebalo bi da se pojavi crni prozor; ovo je vaš terminal!

#### Komanda cmder

Terminal i Windows ljuska nisu baš praktični u današnjem vremenu. Savetujem vam da instalirate `cmder` kako biste olakšali život programerima na Windows-u. Cmder je emulator koji će vam omogućiti da koristite komande dostupne na Linux-u/MacOS-u. Proces instalacije je jednostavan (preuzmite najnoviju verziju sa GitHub-a), a zatim pokrenite čarobnjaka za instalaciju.

Nakon instaliranja cmder-a, pokrenite program "Cmder" da biste otvorili svoj novi terminal.

#### Baš za Windows

Podrazumevano, ne možete koristiti bash na Windows računaru. To nije problem, ali znači da ćete morati da pronađete Windows ekvivalent za svaku macOS/Linux komandu. To može biti komplikovano u nekom trenutku jer mnogi primeri i tutorijali na vebu ne pružaju uvek ekvivalent za Windows.

Majkrosoft je objavio da sada možete instalirati "Windows podsistem za Linux" (WSL) na svoj Windows računar. Ovo je dobra vest jer ćete koristiti bash. Uputstva za instalaciju možete pronaći na Majkrosoftovoj veb stranici: <https://docs.microsoft.com/en-us/windows/wsl/install-win10> (za Windows 10).

Toplo vam savetujem da ovo instalirate, jer će vam olakšati život čak i ako pokušam da dam Windows ekvivalent za osnovne komande u sledećim odeljcima.

## Kako se koristi terminal

Nakon što otvorite terminal, videćete crni prozor. Ovo je interfejs gde možete da kucate komande. Ovo nije intuitivno jer da biste kucali komande, morate ih prethodno znati! Svaka komanda ima ime. Da biste pokrenuli komandu, ukucaćete njeno ime, na kraju neke opcije, a zatim pritisnuti enter. Uzmimo primer.

### O simbolu dolara

U primerima ćete videti red koji počinje simbolom dolara. Ovo je konvencija; znači "unesite u terminal", kada želite da reprodukujete primere ne unosite $ (dolar), već sve posle njega.

#### MacOS / Linux

Recimo da želimo da prikažemo sadržaj vaše radne površine. Unesite sledeću komandu (zamenite maximilienandile svojim imenom).

- Za macOS

  ```sh
  ls /Users/maximilienandile/Desktop
  ```

- Za Linux

  ```sh
  ls /home/maximilienandile/Desktop
  ```

  Zatim pritisnite Enter. Videćete listu datoteka i direktorijuma.

  Sada otkucajte sledeću komandu:

  ```sh
  pwd
  ```
  
  Pritisnite enter. Rezultat je sledeći:
  
  ```sh
  /Users/maximilienandile
  ```

  Ovde smo koristili dve komande:
  
  ```sh
  ls  # omogućava vam da prikažete sadržaj direktorijuma
  pwd # omogućava vam da ispišete radni direktorijum (ispišite ime direktorijuma u ​​kojem se nalazite )
  ```
  
- Windows (bez Linux-a za Windows)

  Komanda za prikaz sadržaja trenutnog direktorijuma je dir.

  Svaki red će predstavljati ili datoteku ili direktorijum. U trećoj koloni se nalazi `<DIR>`, ako je direktorijum. Ništa ako je datoteka.

- Windows (sa WSL-om i Cmder-om)

  Ako ste instalirali WSL i cmder, prvi korak je pokretanje cmder-a (na primer, preko Windows menija). Zatim možete pokrenuti `bash` kucanjem ove komande:
  
  ```sh
  bash
  ```
  
  I pritisnite Enter. Sada koristite bash! (čestitam). Da biste naveli elemente u vašem početnom direktorijumu, jednostavno otkucajte:
  
  ```sh
  ls /mnt/c/Users/maxou123
  ```

  (gde je maxou123 vaše korisničko ime).

## Testirajte sebe

### Pitanja i odgovori

1. Koja je komanda za prikaz sadržaja direktorijuma?
   - `ls` za UNIX sisteme
   - `dir` za Windows
2. Šta je terminal?
   - Terminal je program koji nudi interfejs za korišćenje ljuske  
     (shell-a).
3. Navedite naziv dobro poznate školjke
   - `bash`
   - `zsh`

### Ključno

- Grafički korisnički interfejsi nisu oduvek postojali
- Za interakciju sa računarima možemo koristiti grafički interfejs ili interfejs komandne linije (CLI).
- Da bismo koristili CLI, moramo da otvorimo terminalnu aplikaciju koja nudi interfejs za interakciju sa ljuskom.
- Bash je školjka.
- Možete pokrenuti komande tako što ćete uneti ime komande i eventualno opcije, a zatim pritisnuti enter.
- Koristićemo terminal za pokretanje komandi specifičnih za go.

[02 Jezik Go][02] | [00 Sadržaj][00] | [04 Podesite svoje okruženje][04]

[02]: 02_Jezik_Go.md
[00]: 00_Sadržaj.md
[04]: 04_Podesite_svoje_okružnje.md
