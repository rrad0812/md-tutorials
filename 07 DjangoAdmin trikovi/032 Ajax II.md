Podnošenje AJAX forme u Django

Idemo na posao:

1. Preuzmite komprimovani pre-ajax Django projekat iz repo-a 
2. Aktivirajte virtualenv 
3. Instalirajte zahteve 
4. Sinhronizujte bazu podataka 
5. Pokrenite server 

Kada se prijavite, testirajte formu. Ovde imamo jednostavnu aplikaciju za komunikaciju sa pravima samo za kreiranje. Izgleda lepo, ali postoji jedan dosadan problem: 
Osvežavanje stranice.

Kako da ga se otarasimo? Ili, kako da ažuriramo samo deo veb stranice bez potrebe da osvežavamo celu stranicu?

AJAX je tehnologija na strani klijenta koja se koristi za pravljenje asinhronih zahteva na strani servera – tj. zahtevanje ili slanje podataka – gde naredni odgovori 
ne izazivaju osvežavanje cele stranice.

Ovaj vodič pretpostavlja da imate radno znanje o Djangu, kao i neko iskustvo sa JavaScript/jQuery. Takođe bi trebalo da budete upoznati sa osnovnim HTTP metodama, 
posebno GET i POST. Trebate da ubrzate? Nabavite pravi Python.

Koristi zaštitu

Bez obzira da li koristite AJAX ili ne, obrasci su izloženi riziku od napada falsifikatom zahteva na više lokacija (CSRF). ( Pročitajte više o CSRF napadima na blogu 
Coding Horror . Imaju sjajan članak. )

Da biste sprečili takve napade, morate da dodate {% csrf_token %} oznaku šablona u obrazac, koji dodaje skriveno polje za unos koje sadrži token koji se šalje sa 
svakim POST zahtevom.

Ako pogledate šablon talk/indek.html, možete videti da smo već uključili ovaj token. Međutim, kada su u pitanju AJAX zahtevi, moramo da dodamo još malo koda, jer ne 
možemo da prosledimo taj token koristeći JavaScript objekat pošto su skripte statičke.

Da bismo ovo zaobišli, moramo da kreiramo prilagođeno zaglavlje koje uključuje token da nam čuva leđa. Jednostavno uzmite kod ovde i dodajte ga na kraj datoteke 
main.js. Da, mnogo je koda. Mogli bismo to da prođemo red po red, ali to nije poenta ovog posta. Samo nam verujte da funkcioniše.

Rukovanje događajima

Pre nego što dodirnemo AJAX kod, moramo da dodamo rukovalac događaja u našu JavaScript datoteku koristeći jQuery.

Imajte na umu da je jQuery JavaScript. To je jednostavno JavaScript biblioteka koja se koristi za smanjenje količine koda koji treba da napišete. Ovo je uobičajena
oblast konfuzije, pa samo imajte na umu ovo dok budete prolazili kroz ostatak ovog uputstva.

Koji događaj(e) treba da „obradimo“? Pošto u ovom trenutku samo radimo na kreiranju posta, samo treba da dodamo jedan rukovalac u main.js :

// Submit post on submit
$('#post-form').on('submit', function(event){
    event.preventDefault();
    console.log("form submitted!")  // sanity check
    create_post();
});

Ovde, kada korisnik pošalje obrazac, ova funkcija se pokreće, što

1. Sprečava podrazumevano ponašanje pregledača za podnošenje obrasca, 
2. Dnevnici „obrazac poslat!” na konzolu i 
3. Poziva funkciju koja se zove create_post() gde će AJAX kod živeti. 

Obavezno dodajte ID post-form u obrazac u datoteci index.html :

<form action="/create_post/" method="POST" id="post-form">

I dodajte vezu do JavaScript datoteke na dno šablona:

<script src="static/scripts/main.js"></script>

Testirajte ovo. Pokrenite server, a zatim otvorite svoju JavaScript konzolu. Trebalo bi da vidite sledeće kada pošaljete obrazac:

form submitted!
Uncaught ReferenceError: create_post is not defined

Upravo ovo treba da vidimo: podnošenje obrasca je ispravno obrađeno, pošto je „obrazac dostavljen!“ prikazuje se i create_post funkcija se poziva. Sada samo treba
da dodamo tu funkciju.

Dodavanje AJAX-a

Hajde da razvijemo poslednju iteraciju pre nego što dodamo stvarni AJAX kod.

Ažurirajte main.js :

Dodajte create_postfunkciju:

// AJAX for posting
function create_post() {
    console.log("create post is working!") // sanity check
    console.log($('#post-text').val())
};

Ponovo smo izvršili proveru ispravnosti da bismo se uverili da je funkcija ispravno pozvana, a zatim zgrabimo ulaznu vrednost obrasca. Da bi ovo ispravno 
funkcionisalo, potrebno je da dodamo id u polje obrasca:

Uklonite oglase 

Ažurirajte forms.py :

class PostForm(forms.ModelForm):
    class Meta:
        model = Post
        # exclude = ['author', 'updated', 'created', ]
        fields = ['text']
        widgets = {
            'text': forms.TextInput(attrs={
                'id': 'post-text', 
                'required': True, 
                'placeholder': 'Say something...'
            }),
        }

Obratite pažnju kako smo takođe dodali čuvar mesta u polje i učinili ga obaveznim zajedno sa ID-om. Mogli bismo da dodamo neke rukovaoce greškama u šablon obrasca 
ili jednostavno dozvolimo HTML5 da to upravlja. Hajde da iskoristimo ovo drugo.

Testirajte ponovo. Pošaljite obrazac sa rečju „test“. Trebalo bi da vidite sledeće u svojoj konzoli:

form submitted!
create post is working!
test

Sveet. Dakle, potvrdili smo da ispravno pozivamo funkciju, create_post()kao i da smo uhvatili vrednost unosa obrasca. Hajde sada da ubacimo neki AJAX da bismo
poslali POST zahtev.

Ažurirajte main.js :

// AJAX for posting
function create_post() {
    console.log("create post is working!") // sanity check
    $.ajax({
        url : "create_post/", // the endpoint
        type : "POST", // http method
        data : { the_post : $('#post-text').val() }, // data sent with the post 
request

        // handle a successful response
        success : function(json) {
            $('#post-text').val(''); // remove the value from the input
            console.log(json); // log the returned json to the console
            console.log("success"); // another sanity check
        },

        // handle a non-successful response
        error : function(xhr,errmsg,err) {
            $('#results').html("<div class='alert-box alert radius' data-alert>Oops! 
We have encountered an error: "+errmsg+
                " <a href='#' class='close'>&times;</a></div>"); // add the error to 
the dom
            console.log(xhr.status + ": " + xhr.responseText); // provide a bit more 
info about the error to the console
        }
    });
};

Šta se dešava? Pa, šaljemo podatke obrasca do create_post/krajnje tačke, a zatim čekamo jedan od dva odgovora - ili uspeh ili neuspeh.... Pratite komentare koda za
detaljnije objašnjenje.

Ažurirajte poglede

Sada hajde da ažuriramo naše poglede da bismo ispravno obradili POST zahtev:

def create_post(request):
    if request.method == 'POST':
        post_text = request.POST.get('the_post')
        response_data = {}

        post = Post(text=post_text, author=request.user)
        post.save()

        response_data['result'] = 'Create post successful!'
        response_data['postpk'] = post.pk
        response_data['text'] = post.text
        response_data['created'] = post.created.strftime('%B %d, %Y %I:%M %p')
        response_data['author'] = post.author.username

        return HttpResponse(
            json.dumps(response_data),
            content_type="application/json"
        )
    else:
        return HttpResponse(
            json.dumps({"nothing to see": "this isn't happening"}),
            content_type="application/json"
        )

Ovde preuzimamo tekst posta zajedno sa autorom i ažuriramo bazu podataka. Zatim kreiramo odgovor dict, serijalizujemo ga u JSON, a zatim ga šaljemo kao odgovor - 
koji se evidentira na konzoli u obrađivaču uspeha: console.log(json), kao što ste videli u create_post()funkciji u JavaScript datoteci iznad.

Testirajte ovo ponovo.

Trebalo bi da vidite objekat u konzoli:

form submitted!
create post is working!
Object {text: "hey!", author: "michael", postpk: 15, result: "Create post 
successful!", created: "August 22, 2014 10:55 PM"}
success

Kako bi bilo da dodamo JSON u DOM!

Ažuriranje DOM-a

Ažurirajte šablon

Jednostavno dodajte id „razgovora“ u <ul>:

<ul id="talk">

Zatim ažurirajte obrazac tako da se dodaju greške:

<form method="POST" id="post-form">
    {% csrf_token %}
    <div class="fieldWrapper" id="the_post">
        {{ form.text }}
    </div>
    <div id="results"></div> <!-- errors go here -->
    <input type="submit" value="Post" class="tiny button">
</form>

Ažurirajte main.js

Sada možemo da dodamo JSON u DOM gde je taj novi „razgovor“ id:

success : function(json) {
    $('#post-text').val(''); // remove the value from the input
    console.log(json); // log the returned json to the console
    $("#talk").prepend("<li><strong>"+json.text+"</strong> - <em> 
"+json.author+"</em> - <span> "+json.created+"</span></li>");
    console.log("success"); // another sanity check
},

Spremni da vidite ovo na delu? Testirajte to!

Ako želite da vidite kako izgleda greška, onda komentarišite sav CSRF Javascript u main.js i pokušajte da pošaljete obrazac.

Isperite, ponovite

Tvoj red. Moramo da sredimo još neke događaje. Sa svojim novim saznanjima o jKuery-u i AJAX-u, možete ih postaviti na svoje mesto. Dodao sam kod u konačnu 
aplikaciju - koju možete preuzeti ovde - koja uključuje vezu za brisanje. Potrebno je samo da dodate događaj za rukovanje klikom, koji zatim poziva funkciju koja 
koristi AJAX da pošalje POST zahtev na pozadinu za brisanje objave iz baze podataka. Pratite isti tok rada kao što sam ja uradio u ovom vodiču. Sledeći put ćemo 
objaviti odgovor na ovo.

Zaključak

Kako izgleda vaša aplikacija? Spremni za više?

1. AJAX je tako juče. Možemo učiniti mnogo više sa mnogo manje koda koristeći AngularJS. 
2. U većini slučajeva, standard je JavaScript na strani klijenta, bilo da je u pitanju AJAX ili Angular  ili neki drugi okvir, sa RESTful API-jem na strani servera. 
3. Gde su testovi? 

Srećno kodiranje!

Podnošenje forme za Django i AJAX II

Poslednji put smo dodali AJAX u našu osnovnu Django komunikacionu aplikaciju da bismo poboljšali korisničko iskustvo. Krajnji rezultat je aplikacija koja bolje 
reaguje i funkcionalnija za krajnjeg korisnika pošto smo eliminisali osvežavanje stranice.

Dakle, u smislu CRUD-a, moramo da dodamo mogućnosti DELETE. Koristićemo sledeći tok rada:

1. Podešavanje obrađivača događaja 
2. Kreirajte AJAX zahtev 
3. Ažurirajte Django View 
4. Obradi povratni poziv 
5. Ažurirajte DOM 

Setup Event Handler

Kada korisnik klikne na vezu za brisanje, ovaj "događaj" treba da se "rukuje" u JavaScript datoteci:

// Delete post on click
$("#talk").on('click', 'a[id^=delete-post-]', function(){
    var post_primary_key = $(this).attr('id').split('-')[2];
    console.log(post_primary_key) // sanity check
    delete_post(post_primary_key);
});

Na klik, zgrabimo primarni ključ posta, koji smo dodali id u isto vreme kada se novi post dodaje u DOM:

$("#talk").prepend("<li><strong>" + json.text + "</strong> - 
<em> " + json.author + "</em> - 
<span> " + json.created + "</span> - 
<a id='delete-post-" + json.postpk + "'>delete me</a></li>");

Takođe prosleđujemo primarni ključ kao argument funkciji delete_post(), koju treba da dodamo...

Kreirajte AJAX zahtev

Kao što ste verovatno pretpostavili, delete_post()funkcija obrađuje AJAX zahtev:

function delete_post(post_primary_key){
    if (confirm('are you sure you want to remove this post?')==true){
        $.ajax({
            url : "delete_post/", // the endpoint
            type : "DELETE", // http method
            data : { postpk : post_primary_key }, // data sent with the delete request
            success : function(json) {
                // hide the post
              $('#post-'+post_primary_key).hide(); // hide the post on success
              console.log("post deletion successful");
            },

            error : function(xhr,errmsg,err) {
                // Show an error
                $('#results').html("<div class='alert-box alert radius' data-alert>"+
                "Oops! We have encountered an error. <a href='#' 
                class='close'>&times;</a></div>"); // add error to the dom
                console.log(xhr.status + ": " + xhr.responseText); // provide a bit 
                more info about the error to the console
            }
        });
    } else {
        return false;
    }
};

Uporedite ovaj kod sa create_post() funkcijom. Šta je drugačije?

1. Obratite pažnju na kondicional. Metoda confirm()prikazuje okvir za dijalog, gde korisnik mora 
da klikne ili 'OK' ili 'Cancel'. Pošto će ovaj proces zapravo ukloniti objavu iz baze podataka, samo 
želimo da se uverimo da korisnik nije slučajno kliknuo na delete. Ovo im samo daje priliku da 
otkažu pre nego što se zahtev pošalje.

Ovi dijaloški okviri nisu najelegantniji način da se ovo reši, ali je funkcionalan. Testirajte to.

2. Takođe, pošto brišemo post, koristimo 'DELETE' za HTTP metod.

Imajte na umu da neki stariji pretraživači podržavaju samo GET i POST zahteve. Ako znate 
da će se vaša aplikacija koristiti na nekim starijim verzijama Internet Ekplorer-a, možete da 
koristite POST tuneliranje kao rešenje.

Ažurirajte Django View

Sada, hajde da se okrenemo serverskoj strani i ažuriramo Django URL-ove i poglede. Zahtev se šalje 
serverskoj strani, koja prvo obrađuje urls.py :

# Talk urls
from django.conf.urls import patterns, url

urlpatterns = patterns(
    'talk.views',
    url(r'^$', 'home'),
    url(r'^create_post/$', 'create_post'),
    url(r'^delete_post/$', 'delete_post'),
)

Sa podešenim URL-om, zahtev se zatim usmerava na odgovarajući Django pogled:

def delete_post(request):
    if request.method == 'DELETE':

        post = Post.objects.get(
            pk=int(QueryDict(request.body).get('postpk')))

        post.delete()

        response_data = {}
        response_data['msg'] = 'Post was deleted.'

        return HttpResponse(
            json.dumps(response_data),
            content_type="application/json"
        )
    else:
        return HttpResponse(



            json.dumps({"nothing to see": "this isn't happening"}),
            content_type="application/json"
        )

Ako je metod zahteva 'DELETE' onda objavu uklanjamo iz baze podataka. Šta se dešava ako taj primarni ključ ne postoji? Ovo će dovesti do neočekivanog neželjenog 
efekta. Drugim rečima, dobićemo grešku koja nije pravilno obrađena. Pogledajte da li možete da shvatite kako da uhvatite grešku, a zatim da je pravilno rešite 
koristeći naredbu try/except.

Kada se objava izbriše, kreiramo odgovor dict, serijalizujemo ga u JSON, a zatim ga šaljemo kao odgovor nazad na strani klijenta.

Uporedite kod sa create_post() funkcijom. Zašto ne možemo request.DELETE.get? Pa, zato što Django ne pravi rečnik za DELETE (ili PUT) zahteve kao što to radi za 
GET i POST zahteve. Tako smo konstruisali sopstveni rečnik koristeći get metod iz klase QueryDict.

NAPOMENA : Ova aplikacija bi mogla da bude više RESTful (a kod bi mogao da bude DRY) ako bismo koristili jedan pogled za rukovanje različitim scenarijima 
zasnovanim na HTTP metodama (GET, POST, PUT, DELETE). Trenutno imamo različite poglede za metode GET, POST i DELETE. Ovo bi se lako moglo refaktorisati u jedan 
jedini pogled:

def index(request):
   if request.method == 'GET':
       # do something
   elif request.method == "POST":
       # do something
   elif request.method == "DELETE":
       # do something
   else:
       # do something

O ovome ćemo se detaljnije pozabaviti sledeći put kada dodamo Django Rest Framevork u projekat.

Upravljajte povratnim pozivom

Vraćajući se na delete_post() funkciju u main.js , kako postupamo sa uspešnim povratnim pozivom?

success : function(json) {
    // hide the post
  $('#post-'+post_primary_key).hide(); // hide the post on success
  console.log("post deletion successful");
},

Ovde sakrivamo oznaku sa određenim ID-om, a zatim beležimo poruku o uspehu na konzoli. Na primer, ako izbrišemo objavu sa ID-om 20, tada će oznaka povezana sa 
ID-om post-20 biti sakrivena. Ovo radi? Testirajte to. Ne bi trebalo. Hajde da to popravimo…

Ažurirajte DOM

Otvorite šablon index.html. Vidite li negde taj ID? Jok. Hajde da ga dodamo u početnu <li>oznaku:

<li id='post-{{post.pk}}'>

Sada testirajte. Trebalo bi da radi.

Međutim, pre nego što to nazovemo danom, takođe moramo da napravimo promenu na jednom drugom mestu. Šta se dešava ako dodate post, a zatim odmah pokušate da ga 
izbrišete? Ne bi trebalo da funkcioniše pošto ne ažuriramo DOM ispravno. Vratite se na create_post() funkciju i ažurirajte kod koji se koristi za dodavanje 
novog posta u DOM:

$("#talk").prepend("<li id='post-"+json.postpk+"'><strong>"+json.text+
    "</strong> - <em> "+json.author+"</em> - <span> "+json.created+
    "</span> - <a id='delete-post-"+json.postpk+"'>delete me</a></li>");

Primetite razliku? Testirajte to. Sve je dobro.