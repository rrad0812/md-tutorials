
# Kako funkcioniše Reflex

Koristićemo sledeću osnovnu aplikaciju koja prikazuje slike profila sa Githaba kao primer da objasnimo različite delove arhitekture.

![alt text](images/kako_funkc_reflex.png)

```py
import requests
import reflex as rx

class GithubState(rx.State):
    url: str = "https://github.com/reflex-dev"
    profile_image: str = "https://avatars.githubusercontent.com/u/104714959"

    @rx.event
    def set_profile(self, username: str):
        if username == "":
            return
        try:
            github_data = requests.get(
                f"https://api.github.com/users/{username}"
            ).json()
        except:
            return
        self.url = github_data["url"]
        self.profile_image = github_data["avatar_url"]


def index():
    return rx.hstack(
        rx.link(
            rx.avatar(src=GithubState.profile_image),
            href=GithubState.url,
        ),
        rx.input(
            placeholder="Your Github username",
            on_blur=GithubState.set_profile,
        ),
    )
```

## Arhitektura Reflex-a

Full-stek veb aplikacije se sastoje od frontenda i bekenda. Frontend je korisnički interfejs i služi se kao veb stranica koja se pokreće u korisničkom pregledaču. Bekend se bavi logikom i upravljanjem stanjem (kao što su baze podataka i API-ji) i pokreće se na serveru.

U tradicionalnom veb razvoju, to su obično dve odvojene aplikacije i često su napisane u različitim frejmvorcima ili jezicima. Na primer, možete kombinovati Flask bekend sa React frontendom. Sa ovim pristupom, morate da održavate dve odvojene aplikacije i na kraju pišete mnogo šablonskog koda da biste povezali frontend i bekend.

Želeli smo da pojednostavimo ovaj proces u Reflex-u definisanjem i frontenda i bekenda u jednoj kodnoj bazi, koristeći Pajton za sve. Programeri bi trebalo da brinu samo o logici svoje aplikacije, a ne o detaljima implementacije niskog nivoa.

U suštini, Reflex aplikacije se kompajliraju u `React` frontend aplikaciju i `FastAPI` bekend aplikaciju. Samo se korisnički interfejs kompajlira u `Javascript`; sva logika aplikacije i upravljanje stanjem ostaju u `Pajtonu` i pokreću se na serveru. Reflex koristi `WebSockets` za slanje događaja sa frontenda na bekend i za slanje ažuriranja stanja sa bekenda na frontend.

Dijagram ispod pruža detaljan pregled načina rada aplikacije Reflex. U narednim odeljcima ćemo detaljnije proći kroz svaki deo.

![alt text](images/nacin_rada.png)

### Frontend

Želeli smo da Reflex aplikacije izgledaju i deluju kao tradicionalne veb aplikacije za krajnjeg korisnika, a da istovremeno budu jednostavne za kreiranje i održavanje za programera. Da bismo to postigli, gradili smo na vrhu zrelih i popularnih veb tehnologija.

Kada pokrenete Reflex aplikaciju, Reflex kompajlira frontend u jednostranu `Next.js` aplikaciju i služi je na portu (podrazumevano `3000`) kojem možete pristupiti u svom pregledaču.

Zadatak frontenda je da "odražava" stanje aplikacije i šalje događaje bekendu kada korisnik interaguje sa korisničkim interfejsom. Na frontendu se ne pokreće nikakva stvarna logika.

### Komponente

Refleks frontendovi se grade korišćenjem komponenti koje se mogu sastaviti zajedno da bi se kreirali složeni korisnički interfejsi. Umesto korišćenja šablonskog jezika koji kombinuje HTML i Pajton, mi jednostavno koristimo Pajton funkcije za definisanje korisničkog interfejsa.

```py
def index():
    return rx.hstack(
        rx.link(
            rx.avatar(src=GithubState.profile_image),
            href=GithubState.url,
        ),
        rx.input(
            placeholder="Your Github username",
            on_blur=GithubState.set_profile,
        ),
    )
```

U našoj aplikaciji, imamo komponente kao što su `rx.hstack`, `rx.avatar` i `rx.input`. Ove komponente mogu imati različite osobine - **props** koje utiču na njihov izgled i funkcionalnost - na primer, `rx.input` komponenta ima `placeholder` **prop** za prikaz podrazumevanog teksta.

Možemo naterati naše komponente da reaguju na interakcije korisnika događajima kao što je `on_blur`, o čemu ćemo više govoriti u nastavku.

Ispod haube, ove komponente se kompajliraju u React komponente. Na primer, gornji kod se kompajlira u sledeći React kod:

```js
<HStack>
    <Link href={GithubState.url}>
        <Avatar src={GithubState.profile_image}/>
    </Link>
    <Input
        placeholder="Your Github username"
        // This would actually be a websocket call to the backend.
        onBlur={GithubState.set_profile}
    >
</HStack>
```

Mnoge naše osnovne komponente su bazirane na `Radix`-u , popularnoj `React` biblioteci komponenti. Takođe imamo mnoge druge komponente za grafičko predstavljanje, tabele podataka i još mnogo toga.

Izabrali smo `React` jer je popularna biblioteka sa ogromnim ekosistemom. Naš cilj nije da ponovo stvorimo veb ekosistem, već da ga učinimo dostupnim programerima koji koriste Python.

Ovo takođe omogućava našim korisnicima da donesu sopstvene komponente ako nemamo komponentu koja im je potrebna. Korisnici mogu sami da „umotaju“ svoje React komponente, a zatim ih objave da bi ih drugi koristili. Vremenom ćemo izgraditi naš ekosistem komponenti treće strane tako da korisnici mogu lako da pronađu i koriste komponente koje su drugi napravili.

### Stilizovanje

Želeli smo da se uverimo da Reflex aplikacije izgledaju dobro odmah po pokretanju, a da programerima i dalje daju potpunu kontrolu nad izgledom svoje aplikacije.

Imamo osnovni sistem tematskog kreiranja koji vam omogućava da podesite opcije stilizovanja visokog nivoa, kao što su tamni režim i akcentna boja, u celoj aplikaciji kako biste joj dali jedinstven izgled i osećaj.

Pored ovoga, Reflex komponente se mogu stilizovati koristeći punu snagu CSS-a. Koristimo `Emotion` biblioteku kako bismo omogućili stilizovanje „CSS-in-Python“, tako da možete proslediti bilo koji CSS **prop** kao ključni argument komponenti. Ovo uključuje i responzivne prop-ove prosleđivanjem liste vrednosti.

### Bekend

Sada da pogledamo kako smo dodali interaktivnost našim aplikacijama.

U Refleksu se samo frontend kompajlira u Javaskript i pokreće na korisničkom pregledaču, dok celokupno stanje i logika ostaju u Pajtonu i pokreću se na serveru. Kada pokrenete **reflex run**, pokrećemo FastAPI server (podrazumevano na portu [**8000**]) sa kojim se frontend povezuje preko `web socketa`.

Sva stanja i logika su definisani unutar `State` klase.

```py
class GithubState(rx.State):
    url: str = "https://github.com/reflex-dev"
    profile_image: str = (
        "https://avatars.githubusercontent.com/u/104714959"
    )

    def set_profile(self, username: str):
        if username == "":
            return
        github_data = requests.get(
            f"https://api.github.com/users/{username}"
        ).json()
        self.url = github_data["url"]
        self.profile_image = github_data["avatar_url"]
```

Stanje se sastoji od promenljivih i obrađivača događaja.

Promenljive su bilo koje vrednosti u vašoj aplikaciji koje se mogu menjati tokom vremena. One su definisane kao atributi klase `State` i mogu biti bilo koji Pajton tip koji se može serijalizovati u JSON. U našem primeru, "url" i "profile_image" su promenljive.

Rukovaoci događaja su metode u vašoj `State` klasi koje se pozivaju kada korisnik interaguje sa korisničkim interfejsom. One su jedini način na koji možemo da menjamo promenljive u Refleksu i mogu se pozivati kao odgovor na radnje korisnika, kao što je klik na dugme ili kucanje u tekstualno polje. U našem primeru, "set_profile" je rukovalac događaja koji ažurira promenljive "url" i "profile_image".

Pošto se obrađivači događaja pokreću na serverskom delu, možete koristiti bilo koju Pajton biblioteku unutar njih. U našem primeru, koristimo biblioteku `requests` za upućivanje API poziva Githabu kako bismo dobili sliku profila korisnika.

### Obrada događaja (event handler)

Sada prelazimo na zanimljiv deo - kako se nosimo sa događajima i ažuriranjima stanja.

Obično kada pišete veb aplikacije, morate da napišete mnogo šablonskog koda da biste povezali frontend i bekend. Sa Refleksom, ne morate da brinete o tome - mi se bavimo komunikacijom između frontenda i bekenda umesto vas. Programeri samo treba da napišu logiku za rukovanje događajima, a kada se promenljive ažuriraju, korisnički interfejs se automatski ažurira.

Možete pogledati gornji dijagram za vizuelni prikaz procesa. Hajde da ga prođemo kroz primer slike našeg profila na Githabu.

### Okidači događaja (event trigers)

Korisnik može da interaguje sa korisničkim interfejsom na mnogo načina, kao što je kliktanje na dugme, kucanje u polje za tekst ili zadržavanje pokazivača miša iznad elementa. U Reflex-u, ove događaje nazivamo okidačima događaja.

rx.input(
    placeholder="Your Github username",
    on_blur=GithubState.set_profile,
)

U našem primeru povezujemo `on_blur` okidač događaja sa "set_profile" obrađivačem događaja. To znači da kada korisnik ukuca u polje za unos, a zatim klikne dalje, poziva se "set_profile" obrađivač događaja.

### Red događaja (event que)

Na frontendu održavamo redosled događaja sa svim događajima koji čekaju na realizaciju. Događaj se sastoji od tri glavna dela podataka:

- **klijentski token** : Svaki klijent (kartica pregledača) ima jedinstveni
  token koji ga identifikuje. Ovo omogućava serverskom delu da zna koje stanje da ažurira.
- **obrađivač događaja** : Obradavač događaja koji se pokreće na serveru.
- **argumenti** : Argumenti koji se prosleđuju obrađivaču događaja.

Pretpostavimo da ukucam svoje korisničko ime "picklelo“ u polje za unos. U ovom primeru, naš događaj bi izgledao otprilike ovako:

```json
{
  "client_token": "abc123",
  "event_handler": "GithubState.set_profile",
  "arguments": ["picklelo"]
}
```

Na frontendu održavamo redosled događaja sa svim događajima koji čekaju na realizaciju.

Kada se događaj pokrene, on se dodaje u red čekanja. Imamo `processing` zastavicu koja osigurava da se istovremeno obrađuje samo jedan događaj. Ovo osigurava da je stanje uvek konzistentno i da nema uslova trke sa dva obrađivača događaja koji istovremeno menjaju stanje.

> [!Note]  
> Postoje izuzeci od ovoga, kao što su pozadinski događaji koji vam omogućavaju
  da pokrećete događaje u pozadini bez blokiranja korisničkog interfejsa.

Kada je događaj spreman za obradu, šalje se na bekend putem WebSocket veze.

### Menadžer stanja (State manager)

Kada se događaj primi, on se obrađuje na bekendu.

Refleks koristi menadžer stanja koji održava mapiranje između klijentskih tokena i njihovog stanja. Podrazumevano, menadžer stanja je samo rečnik u memoriji, ali se može proširiti da koristi bazu podataka ili keš. U produkciji koristimo Redis kao naš menadžer stanja.

### Obrada događaja

Kada dobijemo stanje korisnika, sledeći korak je pokretanje obrađivača događaja sa argumentima.

```py
def set_profile(self, username: str):
    if username == "":
        return
    github_data = requests.get(
        f"https://api.github.com/users/{username}"
    ).json()
    self.url = github_data["url"]
    self.profile_image = github_data["avatar_url"]
```

U našem primeru, "set_profile" obrađivač događaja se pokreće iz klase stanja korisnika. Ovo upućuje API poziv Githab-u da bi se dobila slika profila korisnika, a zatim ažurira stanje "url" i "profile_image" promenljive.

### Ažuriranja stanja

Svaki put kada se obrađivač događaja vrati (ili vrati vrednost ), čuvamo stanje u menadžeru stanja i šaljemo ažuriranja stanja frontendu da bismo ažurirali korisnički interfejs.

Da bi održao performanse kako vaše stanje raste, Reflex interno prati promenljive koje su ažurirane tokom obrade događaja ( prljave promenljive ). Kada obrada događaja završi, pronalazimo sve prljave promenljive i kreiramo ažuriranje stanja koje se šalje frontendu.

U našem slučaju, ažuriranje stanja može izgledati otprilike ovako:

```json
{
  "url": "https://github.com/picklelo",
  "profile_image": "https://avatars.githubusercontent.com/u/104714959"
}
```

Novo stanje čuvamo u našem menadžeru stanja, a zatim šaljemo ažuriranje stanja frontendu. Frontend zatim ažurira korisnički interfejs kako bi odražavao novo stanje. U našem primeru, prikazuje se nova slika profila na Githabu.
