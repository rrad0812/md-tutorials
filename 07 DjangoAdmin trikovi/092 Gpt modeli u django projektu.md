
# OpenAI GPT modeli u Djangu projektu

## Uvod

OpenAI GPT modeli su stekli popularnost zbog svoje široke upotrebe u generisanju tekstualnog sadržaja za razne zadatke kao što su pisanje imejlova, odgovaranje na često postavljana pitanja korisničkoj službi i prevođenje jezika, da nabrojimo samo neke.

Ovi GPT modeli se generalno koriste putem ChatGPT-a, četbota koji je objavio OpenAI, ili putem API-ja i biblioteka koje pružaju veću kontrolu. Ovaj tutorijal će vas voditi kroz korišćenje ovih modela pomoću OpenAI API-ja u vašem Django veb projektu. Naučićete kako da pozovete ChatCompletion API koristeći različite parametre i kako da formatirate i koristite njegove odgovore.

Do kraja ovog tutorijala, kreiraćete Django krajnju tačku koja, kada se pozove, šalje zahtev OpenAI-u da konstruiše kratku priču koristeći date reči i vraća svoj odgovor.

## Preduslovi

Da biste završili ovaj tutorijal, biće vam potrebno:

- Postojeći Django projekat: Ako počinjete od nule, možete podesiti Django
  projekat prateći vodič za podešavanje Django razvojnog okruženja.

- OpenAI nalog: Idite na veb stranicu OpenAI platforme i potražite dugme
  "Registruj se". Nakon registracije, morate da potvrdite svoju adresu e-pošte i unesete lične podatke.

- OpenAI API ključ: Kada podesite nalog, prijavite se i idite do odeljka
  API ključevi sa vaše kontrolne table. Kliknite na "Kreiraj novi tajni ključ". Vaš API ključ će biti generisan i izgledaće otprilike ovako "sk-abcdefghijklmnop". Obavezno sačuvajte ovaj ključ na bezbednoj lokaciji, jer ga više nećete moći videti.

- OpenAI Python paket: Ako ste pratili tutorijal u prvom preduslovu,
  trebalo bi da već imate virtuelno okruženje pod nazivom envactive unutar direktorijuma pod nazivom django-apps. Uverite se da je vaše virtuelno okruženje aktivno tako što ćete potvrditi da se njegovo ime pojavljuje u zagradama na početku terminalne komande. Ako nije aktivno, možete ga ručno aktivirati pokretanjem komande:

  ```sh
  sammy@ubuntu:$ .env/bin/activate
  ```

  u vašem terminalu iz `django-apps` direktorijuma. Kada je vaše okruženje aktivno, pokrenite sledeće da biste instalirali OpenAI Python paket:

  ```sh
  (env)sammy@ubuntu:$ pip install openai
  ```

## Upućivanje poziva ka OpenAI-u

U ovom koraku, dodaćete svoj OpenAI API ključ OpenAI klijentu i napraviti jednostavan API poziv ChatCompletion API-ju. Takođe ćete ispitati odgovor koji dobijete od API-ja.

Da biste započeli, otvorite svoj Pajton interpreter:

```sh
(env)sammy@ubuntu:$ python
```

Prvo, uvezite OpenAI klijent i dodajte svoj API ključ klijentu:

```py
from openai import OpenAI
client = OpenAI(api_key="your-api-key")
```

Zamenite "your-api-key"stvarnim API ključem koji ste dobili od OpenAI platforme.

Sada, hajde da napravimo API poziv ChatCompletion API-ju. Koristimo `chat.completions.create()` metod:

```py
response = client.chat.completions.create(
  model="gpt-3.5-turbo", 
  messages=[{"role": "user", "content": "count 1 to 10"}]
)
```

U gornjem kodu, naveli smo model koji treba koristiti kao gpt-3.5-turbo, dodali smo jedan objekat poruke koji sadrži ulogu "user" (ostale opcije su "system" i "assistant") i sadržaj/prompt "count 1 to 10".

Da biste videli odgovor API poziva, možete odštampati poruku odgovora, koja treba da sadrži brojeve od 1 do 10 u lepoj maloj listi:

```py
print(response.choices[0].message.content)

Output
1, 2, 3, 4, 5, 6, 7, 8, 9, 10
```

Čestitamo! Uspešno ste napravili jednostavan API poziv ka OpenAI-ju i dobili odgovor. Formatiraćemo i iskoristiti API odgovor za kreiranje kratke priče u sledećim koracima.

## Rad sa parametrima

Sada kada ste uspešno izvršili jednostavan API poziv ChatCompletion API-ju, hajde da istražimo kako da radimo sa parametrima da bismo prilagodili ponašanje modela. Dostupno je nekoliko parametara koji vam omogućavaju da kontrolišete generisanje teksta. Pogledaćemo tri u nastavku.

1. Temperatura : Parametar temperature određuje koliko je generisani sadržaj slučajan. Viša vrednost temperature, kao što je 0,8 , daće raznovrsnije i kreativnije odgovore, dok će niža vrednost temperature, kao što je 0,1, proizvesti sličnije odgovore. Na primer:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    temperature=0.1
)

print(response.choices[0].message.content)

Output
1. Apple
2. Elephant
3. Sunshine
4. Adventure
5. Serenity
```

Hajde da pokušamo `temperature=0.1` ponovo da vidimo novogenerisani tekst:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    temperature=0.1
)
print(response.choices[0].message.content)

Output
1. Apple
2. Elephant
3. Sunshine
4. Adventure
5. Serenity
```

Tekst je, ispostavilo se isti. Sada, pokušajmo `temperature=0.8` dva puta:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    temperature=0.8
)
print(response.choices[0].message.content)

Output
1. Cat, 
2. Apple, 
3. Guitar, 
4. Sky, 
5. Book

response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    temperature=0.8
)
print(response.choices[0].message.content)

Output
1. Apple
2. Sunshine
3. Happiness
4. Love
5. Technology
```

2.Maksimalan broj tokena: Ovo vam omogućava da ograničite dužinu generisanog teksta. Postavljanje određene vrednosti osigurava da odgovor ne prelazi određeni broj tokena. Tokeni su proporcionalni broju reči u odgovoru. Na primer:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    max_tokens=10
)
print(response.choices[0].message.content)

Output
1. Apple
2. Car
```

Promena vrednosti na 20:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    max_tokens=20
)
print(response.choices[0].message.content)

Output
1. Apple
2. Car
3. Music
4. Ocean
5. Love
```

3.Strimovanje: Ovo određuje da li odgovori treba da se strimuju ili vraćaju. Kada je podešeno na True, API odgovor će biti strimovan, što znači da ćete dobijati izlaz u delovima kako se generiše. Ovo je korisno za duge razgovore ili aplikacije u realnom vremenu. Da biste omogućili strimovanje, dodajte `stream` parametar sa vrednošću `True` API pozivu. Na primer:

```py
response = client.chat.completions.create(
    model="gpt-3.5-turbo", 
    messages=[{"role": "user", "content": "mention five words"}], 
    stream=True
)

collected_messages = []

for chunk in response:
    chunk_message = chunk.choices[0].delta.content
    if chunk_message is not None:
        collected_messages.append(chunk_message)

print(collected_messages)

Output
['', 'cat', '\n', 'book', '\n', 'computer', '\n', 'sun', '\n', 'water']
```

U gornjem kodu, `chunk_message` promenljiva sadrži sadržaj poruke u svakom bloku koji vraća API. Pre nego što dodamo svaki na `collected_messages` listu, proveravamo da li je blok `None` kakav je sadržaj poslednjeg bloka obično None.

Korišćenje ovih parametara vam omogućava da prilagodite ponašanje modela i kontrolišete generisane odgovore kako bi bolje odgovarali vašoj aplikaciji ili projektu. Eksperimentišite sa različitim vrednostima da biste postigli željene rezultate.

U sledećem koraku, pružićemo kontekst modelu u obliku sistemskog upita.

## Kreiranje sistemskog upita

U ovom koraku, kombinovaćemo sve informacije koje smo naučili i kreirati sistemski upit koji pruža kontekst GPT modelu, govoreći mu o njegovoj svrsi i određujući njegova pravila.

Prvo, hajde da kreiramo Pajton modul koji sadrži funkciju za obavljanje ovog zadatka. Zatvorite interpreter i kreirajte novu datoteku pod nazivom `story_generator.py` u direktorijumu vašeg Django projekta.

```sh
(env)sammy@ubuntu:$ touch ~/my_blog_app/blog/blogsite/story_generator.py
```

Zatim, možete dodati OpenAI API ključ svojim promenljivim okruženja kako ga ne biste direktno dodavali u Python datoteku:

```sh
(env)sammy@ubuntu:$ export OPENAI_KEY="your-api-key"
```

Otvorite ga `story_generator.py` i unutar njega kreirajte openai klijent i definišite funkciju generate_storykoja uzima kolekciju reči kao ulaz:

`~/mi_blog_app/blog/blogsite/stori_generator.py`

```py
import os
from openai import OpenAI

client = OpenAI(api_key=os.environ["OPENAI_KEY"])
def generate_story(words):
    # Call the OpenAI API to generate the story
    response = get_short_story(words)
    # Format and return the response
    return format_response(response)
```

U ovoj funkciji pozivamo posebnu funkciju `get_short_story` da bismo uputili API poziv OpenAI-ju za priču, a zatim drugu funkciju `format_response`,da bismo formatirali odgovor iz API-ja.

Sada, hajde da se fokusiramo na `get_short_story` funkciju. Dodajte sledeće na dno vaše `story_generator.py` datoteke:

`~/mi_blog_app/blog/blogsite/stori_generator.py`

```py
def get_short_story(words):
    # Construct the system prompt
    system_prompt = f"""You are a short story generator.
    Write a short story using the following words: {words}.
    Do not go beyond one paragraph."""
    # Make the API call
    response = client.chat.completions.create(
        model="gpt-3.5-turbo",
        messages=[{
            "role": "user",
            "content": system_prompt
        }],
        temperature=0.8,
        max_tokens=1000
    )

    # Return the API response
    return response
```

U ovoj funkciji, prvo podešavamo sistemski upit, koji obaveštava model o zadatku koji treba da izvrši i određuje koliko dugačka treba da bude priča. Zatim prosleđujemo ovaj sistemski upit ChatCompletion API-ju i vraćamo njegov odgovor.

Konačno, možemo implementirati `format_response` funkciju. Dodajte sledeće na dno vaše `story_generator.py` datoteke:

`~/mi_blog_app/blog/blogsite/stori_generator.py`

```py
def format_response(response):
    # Extract the generated story from the response
    story = response.choices[0].message.content
    # Remove any unwanted text or formatting
    story = story.strip()
    # Return the formatted story
    return story
```

Sada možete testirati ove funkcije tako što ćete pozvati `generate_story` funkciju, proslediti joj kolekciju reči kao argument i ispisati njen odgovor. Dodajte sledeće na dno datoteke `story_generator.py`:

```py
print(generate_story("cat, book, computer, sun, water"))
```

Sada sačuvajte i zatvorite datoteku. Pokrenite skriptu da biste videli generisanu priču:

```sh
(env) sammy@ubuntu:$ python ~/my_blog_app/blog/blogsite/story_generator.py
```

Izlaz:

```sh
In a cozy corner of a sunlit room, a fluffy cat named Whiskers lounged lazily next to a towering bookshelf. Amongst the rows of books, a curious computer hummed softly. As the sun streamed through the window, casting a warm glow, Whiskers noticed a small water stain on the shelf. Intrigued, the cat pawed at the book closest to the mark. As if guided by fate, the book opened to reveal a hidden compartment containing a glittering diamond necklace. With the secret now unveiled, Whiskers embarked on an unexpected adventure, where the sun, water, and the power of knowledge merged into a thrilling tale of mystery and discovery.
```

Prilično zanimljivo! Hajde da obrišemo red sa `print` izjavom pošto ćemo `generate_story` funkciju pozivati iz Django prikaza. Uklonite označeni red iz vaše `story_generator.py` datoteke:

```py
print(generate_story("cat, book, computer, sun, water"))
```

Slobodno eksperimentišite sa sistemskim upitom i dodajte više konteksta i pravila kako biste poboljšali generisane priče.

Pređite na sledeći korak da biste integrisali `story_generator` modul u vaš Django projekat.

## Integracija sa bekend prikazom

Morate kreirati Django prikaz i URL rutu da biste integrisali story_generatormodul u svoj Django projekat. U prikazu ćete izdvojiti očekivane reči iz zahteva, pozvati generate_storyfunkciju i vratiti odgovor.

Prvo, otvorite datoteku views.pyu direktorijumu vaše Django aplikacije. Uvezite potrebne module i dodajte funkciju pregleda pod nazivom `generate_story_from_words`:

`~/mi_blog_app/blog/blogsite/vievs.py`

```py
from django.http import JsonResponse
from .story_generator import generate_story

def generate_story_from_words(request):
    words = request.GET.get('words') # Extract the expected words from the request
    story = generate_story(words) # Call the generate_story function with the extracted words
    return JsonResponse({'story': story}) # Return the story as a JSON response
```

Zatim, otvorite urls.pydatoteku i dodajte URL obrazac za generate_story_from_wordsprikaz:

`~/mi_blog_app/blog/blogsite/urls.py`

```py
urlpatterns = [
    # Other URL patterns...
    path('generate-story/', views.generate_story_from_words, name='generate-story'),
]
```

Sada možete da zatražite `/generate-story/` krajnju tačku. Na primer, da biste testirali koristeći curl, možete da napravite GET zahtev krajnjoj tački sa očekivanim rečima kao parametrom upita. Otvorite terminal i pokrenite sledeću komandu:

```sh
(env)sammy@ubuntu:$ curl "http://your_domain/generate-story/?words=cat,book,computer,sun,water"
```

Obavezno zamenite "http://your_domain" stvarnim domenom na kojem se nalazi vaš Django projekat. Reči "cat,book,computer,sun,water" predstavljaju očekivane reči pomoću kojih želite da generišete priču. Možete ih promeniti u bilo koje reči koje želite.

Nakon pokretanja komande, trebalo bi da vidite odgovor servera, koji će sadržati generisanu priču:

```sh
(env)sammy@ubuntu:$ curl "http://your_domain/generate-story/?words="cat,book,computer,sun,water"
```

Izlaz:

```sh
{"story": "Once upon a time, in a cozy little cottage nestled amidst a dense forest, a curious cat named Whiskers sat by the window, basking in the warm rays of the sun. As Whiskers lazily flicked his tail, his eyes caught sight of a dusty book lying on a nearby shelf. Intrigued, he carefully jumped onto the shelf, causing a cascade of books to tumble down, one opening up to reveal a hidden compartment. Inside, Whiskers discovered an ancient computer, its screen flickering to life as he brushed against the power button. Mesmerized by the glowing screen, Whiskers ventured into a world of virtual landscapes, where he roamed freely, chasing digital fish and pausing to admire breathtaking waterfalls. Lost in this newfound adventure, Whiskers discovered the wonders of both the tangible and virtual worlds, realizing that true exploration knows no bounds."}
```
