English

Poslednje ažuriranje 24. avgusta 2021.

Rad sa AJAX-om u Django-u
Jasin Ruizi

Istaknuti kurs
 Tviter  Redit  Hakerske vesti  Fejsbuk

Razvoj vođen testiranjem

AJAX , što je skrać pomoću Django-a, Django
enica od asinhroni JavaScript i XML, je skup tehnologija koje se

REST Framework-a i
koriste na strani klijenta za asinhrono slanje i preuzimanje podataka sa servera.

Docker-a
AJAX nam omogućava da menjamo sadržaj veb stranice, bez potrebe da korisnik

U ovom kursu ćete naučiti kako da
ponovo učita celu stranicu. Ovo može biti korisno, na primer, za automatsko podesite razvojno okruženje sa Dokerom
dovršavanje u traci za pretragu ili validaciju obrasca. Pravilno korišćeno, možete kako biste izgradili i implementirali RESTful

poboljšati performanse vašeg sajta, smanjiti opterećenje servera i poboljšati API koji pokreću Pajton, Django i Django

celokupno korisničko iskustvo. REST Framework.

U ovom članku ćemo pogledati primere kako se izvršavaju GET, POST, PUT i Kupi odmah 30 dolara
DELETE AJAX zahtevi u Django-u. Iako će fokus biti na Fetch API-ju , prikazaćemo
i jQuery primere. Pogledajte kurs

Šta je AJAKS? Претражи све туторијале

AJAX je programska praksa koja koristi XMLHttpRequest (XHR) objekat za
asinhronu komunikaciju sa serverom i kreiranje dinamičkih veb stranica. Iako se TEME TUTORIJALA

AJAX i XMLHttpRequest često koriste naizmenično, to su različite stvari.
api arhitektura avs devops

Da bi slao i primao podatke na i sa veb servera, AJAX koristi sledeće korake: Django django rest framework

doker brzi api termos
1. Napravite objekat XMLHttpRequest.

prednji deo heroku kubernetes

2. Koristite objekat XMLHttpRequest za asinhronu razmenu podataka između mašinsko učenje pajton reaguj
klijenta i servera.

red zadataka testiranje vju

3. Koristite Javaskript i DOM za obradu podataka. veb skrejping

AJAX se može koristiti sa jQuery-jem korišćenjem ajax metode, ali je nativni Fetch
API mnogo bolji jer ima čist interfejs i ne zahteva biblioteku treće strane. SADRŽAJ

Opšta struktura Fetch API-ja izgleda ovako: Šta je AJAKS?

Kada treba koristiti AJAX?
fetch('http://some_url.com')

.then(response => response.json()) // converts the response to JSON CRUD resurs

.then(data => {

console.log(data); Zahtev za dobijanje
// do something (like update the DOM with the data)

POST zahtev
});

Zahtev za stavljanje

Zahtev za brisanje
Pogledajte odeljak Korišćenje Fetch i WindowOrWorkerGlobalScope.fetch() iz
MDN dokumentacije za više primera, zajedno sa svim dostupnim opcijama za Rezime

fetch metodu.

Povratne informacije



Kada treba koristiti AJAX?
Ponovo, AJAX može pomoći u poboljšanju performansi vašeg sajta, uz smanjenje
opterećenja servera i poboljšanje celokupnog korisničkog iskustva. Uz to, dodaje
mnogo složenosti vašoj aplikaciji. Zbog toga, osim ako ne koristite jednostranu
aplikaciju (SPA) - kao što su React, Angular ili Vue - trebalo bi da koristite AJAX
samo kada je apsolutno neophodan.

Neki primeri kada biste mogli razmisliti o korišćenju AJAX-a:

1. Automatsko dovršavanje pretrage

2. Validacija formulara

3. Sortiranje i filtriranje tabela

4. Kapče
5. Ankete i ankete

Generalno, ako je potrebno često ažurirati sadržaj na osnovu interakcija korisnika,
možda biste želeli da razmotrite korišćenje AJAX-a za upravljanje ažuriranjem
delova veb stranice umesto cele stranice osvežavanjem stranice.

CRUD resurs
Primeri u ovom članku mogu se primeniti na bilo koji CRUD resurs. Primer Django
projekta koristi todos kao svoj resurs:

Metod URL Opis

DOBIJ /todos/ Vraća sve zadatke

POŠTA /todos/ Dodaje zadatak

PUT /todos/<todo-id>/ Ažurira zadatak

OBRIŠI /todos/<todo-id>/ Briše zadatak

Primer projekta se može naći na GitHabu:

1. Preuzmi verziju: https://github.com/testdrivenio/django-ajax-xhr

2. Verzija jKueri-ja: https://github.com/testdrivenio/django-ajax-xhr/tree/jquery

Zahtev za dobijanje
Počnimo sa jednostavnim GET zahtevom za preuzimanje podataka.

API za preuzimanje
Primer:

Povratne informacije



fetch(url, {

method: "GET",

headers: {

"X-Requested-With": "XMLHttpRequest",

}

})

.then(response => response.json())

.then(data => {

console.log(data);

});

Jedini potrebni argument je URL adresa resursa sa kog želite da preuzmete
podatke. Ako URL adresa zahteva ključne reči ili nizove upita, možete koristiti
Django-ovu {% url %} oznaku.

Da li ste primetili X-Requested-With zaglavlje? Ovo je neophodno da biste obavestili
server da šaljete AJAX zahtev.

fetch vraća obećanje koje sadrži HTTP odgovor. Koristili smo .then metodu da
prvo izdvojimo podatke u JSON formatu iz odgovora (preko response.json() ), a
zatim da pristupimo vraćenim podacima. U gornjem primeru, samo smo prikazali
podatke u konzoli.

https://github.com/testdrivenio/django-ajaks-khr/blob/main/static/main.js#L19-L39

jQuery AJAX
Ekvivalentni jQuery kod:

$.ajax({

url: url,

type: "GET",

dataType: "json",

success: (data) => {

console.log(data);

},

error: (error) => {

console.log(error);

}

});

https://github.com/testdrivenio/django-ajaks-khr/blob/jkueri/static/main.js#L19-L41

Django Vju
Što se tiče Django-a, iako postoji nekoliko načina za rukovanje AJAX zahtevima u
prikazima, najjednostavniji je pomoću prikaza zasnovanog na funkcijama:

from django.http import HttpResponseBadRequest, JsonResponse

from todos.models import Todo

def todos(request):

# request.is_ajax() is deprecated since django 3.1

is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'

if is_ajax:

if request.method == 'GET':

todos = list(Todo.objects.all().values())

return JsonResponse({'context': todos})

return JsonResponse({'status': 'Invalid request'}, status=400)

else:

return HttpResponseBadRequest('Invalid request')

Povratne informacije



U ovom primeru, naš resurs je lista zadataka (todos). Dakle, pre nego što smo
preuzeli liste zadataka iz baze podataka, proverili smo da li imamo posla sa AJAX
zahtevom i da je metod zahteva GET. Ako su oba vrednosti tačna, serijalizovali smo
podatke i poslali odgovor koristeći JsonResponse klasu . Pošto QuerySet objekat nije
JSON serijalizabilan (u ovom slučaju lista zadataka), koristili smo values metodu
da vratimo naš QuerySet kao rečnik, a zatim smo ga umotali u list . Konačni
rezultat je lista rečnika.

https://github.com/testdrivenio/django-ajax-xhr/blob/main/todos/views.py#L13-L28

POST zahtev
Zatim, hajde da vidimo kako se obrađuju POST zahtevi.

API za preuzimanje
Primer:

fetch(url, {

method: "POST",

credentials: "same-origin",

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"),

},

body: JSON.stringify({payload: "data to send"})

})

.then(response => response.json())

.then(data => {

console.log(data);

});

Moramo navesti kako želimo da pošaljemo akreditive u zahtevu.

U gornjem kodu, koristili smo vrednost "same-origin" (podrazumevana) da bismo
pregledaču naznačili da pošalje akreditive ako je traženi URL na istom poreklu kao i
poziv za preuzimanje.

U slučaju da se frontend i bekend nalaze na različitim serverima, morali biste da
podesite credentials na "include" (što uvek šalje akreditive sa svakim zahtevom)
i omogućite deljenje resursa između izvora u bekendu. Možete koristiti paket
django-cors-headers da biste dodali CORS zaglavlja odgovorima u Django
aplikaciji.

Želite da saznate više o tome kako da obrađujete AJAX zahteve na istom
domenu i na više domena? Pregledajte članak o Django Session-based Auth
for Single Page Apps .

Ovog puta smo poslali podatke serveru u body zahtevu.

Obratite pažnju na X-CSRFToken zaglavlje. Bez njega biste dobili odgovor 403
forbidden od servera u terminalu:

Forbidden (CSRF token missing or incorrect.): /todos/

To je zato što je obavezno uključiti CSRF token prilikom podnošenja POST zahteva
kako bi se sprečili napadi falsifikovanja zahteva između sajtova .

Možemo uključiti CSRF token tako što ćemo X-CSRFToken zaglavlje na svakom od
njih podesiti XMLHttpRequest na vrednost CSRF tokena. Povratne informacije



Django dokumentacija nam pojednostavljuje život pružajući nam lepu funkciju koja
nam omogućava da dobijemo token :

function getCookie(name) {

let cookieValue = null;

if (document.cookie && document.cookie !== "") {

const cookies = document.cookie.split(";");

for (let i = 0; i < cookies.length; i++) {

const cookie = cookies[i].trim();

// Does this cookie string begin with the name we want?

if (cookie.substring(0, name.length + 1) === (name + "=")) {

cookieValue = decodeURIComponent(cookie.substring(name.length +

1));

break;

}

}

}

return cookieValue;

}

https://github.com/testdrivenio/django-ajaks-khr/blob/main/static/main.js#L42-L56

jQuery AJAX
AJAX POST zahtev sa jQuery-jem je prilično sličan GET zahtevu:

$.ajax({

url: url,

type: "POST",

dataType: "json",

data: JSON.stringify({payload: payload,}),

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"), // don't forget to include the 

'getCookie' function

},

success: (data) => {

console.log(data);

},

error: (error) => {

console.log(error);

}

});

https://github.com/testdrivenio/django-ajaks-khr/blob/jkueri/static/main.js#L44-L61

Django Vju
Na strani servera, prikaz treba da dobije podatke iz zahteva u JSON formatu, tako
da ćete morati da koristite json modul da biste ga učitali.

Povratne informacije



import json

from django.http import HttpResponseBadRequest, JsonResponse

from todos.models import Todo

def todos(request):

# request.is_ajax() is deprecated since django 3.1

is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'

if is_ajax:

if request.method == 'POST':

data = json.load(request)

todo = data.get('payload')

Todo.objects.create(task=todo['task'],

completed=todo['completed'])

return JsonResponse({'status': 'Todo added!'})

return JsonResponse({'status': 'Invalid request'}, status=400)

else:

return HttpResponseBadRequest('Invalid request')

Nakon što smo proverili da li imamo posla sa AJAX zahtevom i da je metod zahteva
POST, deserijalizovali smo objekat zahteva i izvukli objekat korisnog tereta. Zatim
smo kreirali novi todo i poslali nazad odgovarajući odgovor.

https://github.com/testdrivenio/django-ajax-xhr/blob/main/todos/views.py#L13-L28

Zahtev za stavljanje

API za preuzimanje
Primer:

fetch(url, {

method: "PUT",

credentials: "same-origin",

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"), // don't forget to include the 

'getCookie' function

},

body: JSON.stringify({payload: "data to send"})

})

.then(response => response.json())

.then(data => {

console.log(data);

});

Ovo bi trebalo da izgleda slično POST zahtevu. Jedina razlika je oblik URL-a:

1. POŠTA - /todos/

2. STAVI - /todos/<todo-id>/

https://github.com/testdrivenio/django-ajaks-khr/blob/main/static/main.js#L59-L73

jQuery AJAX
Ekvivalent jQuery-ju:

Povratne informacije



$.ajax({

url: url,

type: "PUT",

dataType: "json",

data: JSON.stringify({payload: payload,}),

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"), // don't forget to include the 

'getCookie' function

},

success: (data) => {

console.log(data);

},

error: (error) => {

console.log(error);

}

});

https://github.com/testdrivenio/django-ajaks-khr/blob/jkueri/static/main.js#L64-L81

Django Vju
Primer:

import json

from django.http import HttpResponseBadRequest, JsonResponse

from django.shortcuts import get_object_or_404

from todos.models import Todo

def todo(request, todoId):

# request.is_ajax() is deprecated since django 3.1

is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'

if is_ajax:

todo = get_object_or_404(Todo, id=todoId)

if request.method == 'PUT':

data = json.load(request)

updated_values = data.get('payload')

todo.task = updated_values['task']

todo.completed = updated_values['completed']

todo.save()

return JsonResponse({'status': 'Todo updated!'})

return JsonResponse({'status': 'Invalid request'}, status=400)

else:

return HttpResponseBadRequest('Invalid request')

https://github.com/testdrivenio/django-ajaks-khr/blob/main/todos/vievs.pi#L31-L53

Zahtev za brisanje

API za preuzimanje
Primer:

Povratne informacije



fetch(url, {

method: "DELETE",

credentials: "same-origin",

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"), // don't forget to include the 

'getCookie' function

}

})

.then(response => response.json())

.then(data => {

console.log(data);

});

https://github.com/testdrivenio/django-ajaks-khr/blob/main/static/main.js#L76-L89

jQuery AJAX
jQuery kod:

$.ajax({

url: url,

type: "DELETE",

dataType: "json",

headers: {

"X-Requested-With": "XMLHttpRequest",

"X-CSRFToken": getCookie("csrftoken"),

},

success: (data) => {

console.log(data);

},

error: (error) => {

console.log(error);

}

});

https://github.com/testdrivenio/django-ajaks-khr/blob/jkueri/static/main.js#L84-L100

Django Vju
Prikaz:

from django.http import HttpResponseBadRequest, JsonResponse

from django.shortcuts import get_object_or_404

from todos.models import Todo

def todo(request, todoId):

# request.is_ajax() is deprecated since django 3.1

is_ajax = request.headers.get('X-Requested-With') == 'XMLHttpRequest'

if is_ajax:

todo = get_object_or_404(Todo, id=todoId)

if request.method == 'DELETE':

todo.delete()

return JsonResponse({'status': 'Todo deleted!'})

return JsonResponse({'status': 'Invalid request'}, status=400)

else:

return HttpResponseBadRequest('Invalid request')

https://github.com/testdrivenio/django-ajaks-khr/blob/main/todos/vievs.pi#L31-L53

Rezime
Povratne informacije



AJAX nam omogućava da izvršavamo asinhrone zahteve za promenu delova
stranice bez potrebe za ponovnim učitavanjem cele stranice.

U ovom članku ste detaljno videli primere kako se izvršavaju GET, POST, PUT i
DELETE AJAX zahtevi u Django-u pomoću Fetch API-ja i jQuery-ja.

Primer projekta se može naći na GitHabu:

1. Preuzmi verziju: https://github.com/testdrivenio/django-ajax-xhr

2. Verzija jKueri-ja: https://github.com/testdrivenio/django-ajax-xhr/tree/jquery

 Django

Jasin Ruizi
Jasin je samouki veb programer i bloger iz Alžira.
Fasciniran je mašinskim učenjem. U slobodno vreme
uživa u čitanju o istoriji, psihologiji i fizici. Možete uživati u
njegovim drugim delima na Don't Repeat Yourself .

   

PODELI OVAJ TUTORIJAL

 Tviter  Redit  Hakerske vesti  Fejsbuk

PREPORUČENI TUTORIJALI



Budite u toku sa ažuriranjima kursa
Pridružite se našoj mejling listi da biste bili obavešteni o

ažuriranjima i novim izdanjima.

Унесите своју имејл адресу Pretplatite se

UČI

Kursevi Paketi Blog  TestDriven.io je
ponosni pristalica
otvorenog koda

VODIČI Povratne informacije



Kompletan Pajton Django i celer Dubinski zaron u Flask 10% profita od svakog našeg FastAPI
kursa i našeg Flask kursa za veb
razvoj biće donirano FastAPI i Flask

O TESTDRIVEN.IO timovima, respektivno.

Podrška i konsultacije Šta je razvoj vođen testiranjem? Svedočanstva
Pratite naše doprinose

Donacije otvorenog koda O nama Upoznajte autore Saveti i trikovi

© Autorska prava 2017–2025 TestDriven Labs.
Razvio Majkl Herman .

Прати @testdrivenio

Povratne informacije