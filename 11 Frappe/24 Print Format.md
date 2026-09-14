# Print Format u Frappe-u

Print Format je jedan od najkorisnijih delova Frappea kad želiš da dokument "izgleda" kao pravi poslovni papir.

**Za šta se koristi?**

Najčešće se koristi za:

- fakture
- otpremnice
- ponude
- naloge
- izveštaje
- sažetke taskova

**Šta je Print Format?**

Print Format je zapravo HTML + CSS + Jinja template koji prikazuje podatke iz jednog Frappe dokumenta.

Kada ga otvoriš, Frappe uzme podatke iz DocType-a i napravi:

- pregled na ekranu
- štampu
- PDF

**Zašto je važan?**

Ovo nije samo "lepši prikaz".

Print Format ti daje poslovnu vrednost jer omogućava da:

- korisnik dobije spreman dokument
- podaci budu jasno organizovani
- isti dokument može da se koristi u štampi i PDF-u

**Kako funkcioniše?**

Frappe Print Format radi na sledeći način:

1. Uzmeš dokument iz DocType-a
2. U template-u referenciraš polja tog dokumenta
3. Frappe renderuje HTML
4. Dobiješ gotov print/PDF

**Najvažnija ideja!**

U Print Formatu ne misliš "HTML stranica".

Misliš "poslovni dokument".

To znači:

- šta korisnik treba da vidi
- koje podatke su važni
- kako ih uredno prikazati

## Primer

Ako imaš Task dokument, možeš prikazati:

- naslov taska
- status
- assignee
- due date
- opis

Primer template-a:

```html
<h2>{{ doc.subject }}</h2>

<p><strong>Status:</strong> {{ doc.status }}</p>
<p><strong>Assigned To:</strong> {{ doc.assigned_to }}</p>
<p><strong>Due Date:</strong> {{ doc.expiry_date }}</p>

{% if doc.description %}
<p><strong>Description:</strong><br>
{{ doc.description }}</p>
{% endif %}
```

**Pristupa poljima dokumenta:**

Najčešće koristiš:

```html
{{ doc.ime_polja }}
```

Na primer:

```html
{{ doc.customer_name }}
{{ doc.total_amount }}
{{ doc.status }}
```

**Ako imaš child table:**

Ako dokument ima stavke, možeš ih prikazati kroz petlju:

```html
<table class="table table-bordered">
  <thead>
    <tr>
      <th>Item</th>
      <th>Qty</th>
      <th>Amount</th>
    </tr>
  </thead>
  <tbody>
    {% for row in doc.items %}
    <tr>
      <td>{{ row.item_code }}</td>
      <td>{{ row.qty }}</td>
      <td>{{ row.amount }}</td>
    </tr>
    {% endfor %}
  </tbody>
</table>
```

**Dobra praksa:**

- prvo napravi logičan raspored dokumenta
- zatim dodaj polja
- tek onda stilizuj
- drži format jednostavnim na početku

**Prvi primer za našu aplikaciju?**

Za aplikaciju sa taskovima, odličan prvi Print Format bi bio:

- Task Summary
- naslov taska
- opis
- assignee
- due date
- status
- lista svih time entries ili povezanih podataka

Tako odmah vidiš da Print Format nije samo "štampanje", nego stvarna poslovna funkcionalnost.

**Najvažnije stvari da zapamtiš!**

- Print Format koristi Jinja template
- `doc` predstavlja trenutni dokument
- možeš prikazivati polja i child table-e
- možeš dodati CSS za izgled
- odličan je za PDF i štampu

## Kako to uradiš u Frappe-u?

Evo vrlo praktičnog postupka.

- **Otvori Desk**

  Uđeš u svoj Frappe site i otvoriš Desk.

- **Idi na Print Formats**

  U glavnom meniju tražiš **Print Format** ili otvoriš modul ako ga već imaš u aplikaciji.

- **Kreiraj novi Print Format**

  Klikneš na:
  
  - New
  
  i popuniš osnovne podatke:
  
  - Name: Task Summary
  - DocType: Task
  - Standard: Yes ili No, zavisno od potrebe

- **Napiši template**

  U polje za HTML template ubaciš nešto ovako:
  
  ```html
  <h2 style="margin-bottom: 20px;">{{ doc.subject }}</h2>
  
  <p><strong>Status:</strong> {{ doc.status }}</p>
  <p><strong>Assigned To:</strong> {{ doc.assigned_to }}</p>
  <p><strong>Due Date:</strong> {{ doc.expiry_date }}</p>
  
  {% if doc.description %}
  <hr>
  <p><strong>Description:</strong></p>
  <p>{{ doc.description }}</p>
  {% endif %}
  ```

- **Sačuvaj i testiraj**

  Nakon što sačuvaš, otvoriš jedan Task i u dropdown-u za Print proveriš da li se pojavljuje tvoj format.
  
  Ako je sve dobro, videćeš dokument u formatu koji si definisao.

**Najčešća greška početnika!**

Najčešća greška je da pokušaš odmah da napraviš "previše".

Počni od ovoga:

- naslov
- status
- datum
- opis

Kad to radi, onda dodaješ stil i dodatna polja.

**Mali trik za bolje prikazivanje!**

Ako želiš da format izgleda profesionalno, koristiš jednostavan CSS:

```html
<style>
  body { font-family: Arial, sans-serif; }
  h2 { color: #1f2937; }
  .box { border: 1px solid #ddd; padding: 15px; margin-top: 20px; }
</style>

<div class="box">
  <h2>{{ doc.subject }}</h2>
  <p><strong>Status:</strong> {{ doc.status }}</p>
</div>
```

**Kako razmišljati o Print Formatu?**

Misli na ovo:

- šta će korisnik tačno videti?
- koje informacije su bitne?
- da li format treba da bude za štampu, PDF ili samo pregled?

Ako napraviš format koji je jasan i jednostavan, već ćeš imati dobar rezultat.

## Praktičan primer za našu aplikaciju

U aplikaciji sa taskovima, dobar prvi Print Format bi bio:

- Task Summary
- Subject
- Status
- Assigned To
- Due Date
- Description

To je idealan početni primer jer pokazuje osnovnu logiku bez previše komplikacija.

## Još nekoliko važnih stvari

**Uslovni prikaz:**

Možeš prikazati deo teksta samo ako postoji podatak:

```html
{% if doc.description %}
<p>{{ doc.description }}</p>
{% endif %}
```

**Petlje za više stavki:**

Ako imaš više stavki, koristiš petlju:

```html
{% for row in doc.items %}
<tr>
  <td>{{ row.item_code }}</td>
  <td>{{ row.qty }}</td>
</tr>
{% endfor %}
```

**Formatiranje vrednosti:**

Neke vrednosti možeš formatirati na pregledan način:

```html
{{ frappe.format(doc.total_amount, {'fieldtype': 'Currency'}) }}
```

**Standard vs custom Print Format:**

- Standard Print Format je onaj koji dolazi sa sistemom ili aplikacijom
- Custom Print Format je tvoj prilagođeni format

U praksi, obično kreiraš custom format za specifične poslovne potrebe.

**Kad Print Format postane „pravi“ dokument:**

Print Format je najkorisniji kada želiš da dokument:

- izgleda profesionalno
- ima jasnu strukturu
- može da se štampa ili šalje kao PDF

## Zaključak

Print Format nije samo dekoracija.

On je jedan od načina da iz Frappe podataka napraviš nešto što ima stvarnu poslovnu vrednost.

Ako razmišljaš u tom smeru, onda si već otišao korak dalje od samog tutoriala.
