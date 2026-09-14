# Učenje Frappe Framework

[Sadržaj][00]

## 11 Korišćenje Frappe frameworka

Ne znam da li je ovo najblji način za jedno običnog programera da shvati kako treba da koristi FF ali sam ja izgubio najvažniju nit: **Koji način koristim kada kreiram dokument u kojoj situaciji?**
Nisi mi baš najjasniji bio do kraja.

### Šta programer treba da koristi

A to je zapravo ono zbog čega učiš Frappe.

Postoje četiri osnovna scenarija rada sa dokumentima:

1. **Kreiranje novog dokumenta**

   Koristiš:

   ```python
   doc = frappe.new_doc("Customer")
   ```

   ili

   ```python
   doc = frappe.get_doc({
       "doctype": "Customer",
       "customer_name": "ABC Ltd"
   })
   ```

   kada praviš potpuno novi dokument koji još ne postoji u bazi.

   Razlika između ova dva je uglavnom u praktičnosti:

   ```py
   doc = frappe.new_doc("Customer")
   doc.customer_name = "ABC Ltd"
   doc.customer_group = "Commercial"
   ```

   naspram:

   ```python
   doc = frappe.get_doc({
       "doctype": "Customer",
       "customer_name": "ABC Ltd",
       "customer_group": "Commercial"
   })
   ```

   Na kraju oba završe sa:

   ```python
   doc.insert()
   ```

2. **Učitavanje postojećeg dokumenta**

   Koristiš:

   ```python
   doc = frappe.get_doc("Customer", "CUST-00001")
   ```

   Ovde Frappe odlazi u bazu, pročita podatke i vrati objekat.

3. **Brzo čitanje podataka**

   Ako ti ne treba ceo objekat:

   ```python
   name = frappe.db.get_value(...)
   ```

   ili

   ```python
   rows = frappe.get_all(...)
   ```

   Ovako je mnogo brže. Ne pravi `Document`, ne pokreće dodatnu logiku.

4. **Direktan SQL**

   Tek kada ORM nije dovoljan.

### Hijerarhija pristupa

Treba mi dokument?

Ja bih to danas nacrtao ovako:

```text
│
├── Ne
│      │
│      └── get_all()
│          get_value()
│          SQL
│
└── Da
       │
       ├── već postoji?
       │        │
       │        └── get_doc()
       │
       └── novi?
                │
                ├── new_doc()
                └── get_doc({...})
```

### Filozofija Frappea

- sve je `Document`,
- svaki `Document` ima `Meta`,
- ponašanje dolazi iz kombinacije metapodataka i Python klase,
- gotovo sve prolazi kroz isti životni ciklus (`insert`, `save`, `submit`, `cancel`...),
- hook-ovi i događaji omogućavaju proširenje bez menjanja jezgra framework-a.

Zamisli da praviš aplikaciju za servis računara.

Pitanje nije: Kako radi `get_controller()`?  
Pitanje je: Gde pišem poslovnu logiku?  
Odgovor je: na više mesta, i svako ima svoju namenu.

- **Controller (DocType klasa)**

  Ovo je prirodno mesto za logiku jednog DocType-a.
  
  Na primer:
  
  ```python
  class ServiceOrder(Document):
  
      def validate(self):
          ...
  
      def before_save(self):
          ...
  
      def on_submit(self):
          ...
  ```
  
  Ovde pišeš sve što je sastavni deo tog DocType-a.
  
  Na primer:
  
  - izračunavanje ukupne cene,
  - provera da li je serijski broj unet,
  - automatsko postavljanje statusa,
  - validacija datuma.
  
  Drugim rečima: Ako ta logika pripada tom dokumentu, piše se u kontroleru.

- **Hooks**

  Hook nije zamena za kontroler. Hook kaže: "Ne mogu ili ne želim da menjam postojeći DocType, ali želim da reagujem na njegove događaje."
  
  Na primer, ERPNext već ima:
  
  ```text
  Sales Invoice
  ```
  
  Ti nećeš da menjaš ERPNext. Ali želiš: "Posle svakog submit-a pošalješ podatke u eksterni ERP."
  
  Ne diraš:
  
  ```python
  SalesInvoice(Document)
  ```
  
  nego u `hooks.py` napišeš:
  
  ```python
  doc_events = {
      "Sales Invoice": {
          "on_submit": "my_app.api.export_invoice"
      }
  }
  ```
  
  To je potpuno druga filozofija.

- **Controller i Hook nisu konkurenti**

  Mislim da je ovo najvažnija rečenica.
  
  Ja sam ih dugo posmatrao kao:
  
  ```text
  Controller ili Hook
  ```
  
  A sada mislim da je ispravno:
  
  ```text
  Controller + Hook
  ```
  
  Controller je vlasnik DocType-a. Hook je posmatrač.
  
- **Override**
  
  Treća mogućnost.
  
  Ako kažeš:
  
  ```python
  override_doctype_class = {
      "Customer": ...
  }
  ```
  
  onda praktično kažeš: Nemoj koristiti Frappe-ov ili ERPNext-ov kontroler Koristi moj. Ovo je ozbiljna intervencija.
  
  Koristi se retko.
  
- **Server Script**

  Ako želiš malu logiku bez pravljenja aplikacije.
  
  Na primer:
  
  - automatski popuni polje,
  - zabrani nešto,
  - pošalji mail.
  
  To nije zamena za ozbiljan razvoj, ali je zgodno za manje prilagođavanje.
  
- **Ja bih to danas nacrtao ovako**
  
  ```text
  Da li je ovo logika mog DocType-a?
  
          │
          ├── DA
          │      │
          │      └── Controller
          │
          └── NE
                 │
                 ├── samo reagujem?
                 │         │
                 │         └── Hook
                 │
                 ├── menjam tuđi DocType?
                 │         │
                 │         └── Override
                 │
                 └── mala administrativna logika
                           │
                           └── Server Script
  ```

### Kako izgleda razvoj jedne Frappe aplikacije

Drugim rečima, svaku temu ćemo posmatrati kroz tri pitanja:

1. **Šta želim da uradim?**
2. **Koji Frappe API koristim?**
3. **Kada se izvršava moj kod?**

Na primer, za dokumente:

| Želim da...                            | Koristim                                       | Moj kod pišem u... |
| -------------------------------------- | ---------------------------------------------- | ------------------ |
| Napravim novi dokument                 | `frappe.new_doc()` ili `frappe.get_doc({...})` | Controller         |
| Učitam postojeći                       | `frappe.get_doc()`                             | Controller         |
| Reagujem na događaj drugog DocType-a   | `hooks.py` (`doc_events`)                      | Hook               |
| Zamenim ponašanje postojećeg DocType-a | `override_doctype_class`                       | Novi Controller    |
| Samo pročitam podatke                  | `frappe.db.get_value()`, `frappe.get_all()`    | Nema Controller-a  |

Po mom mišljenju, ovakva tabela je vrednija za svakodnevni rad nego deset stranica analize `BaseDocument`.

**Kada koristim Controller, a kada Hook?**

Recimo da pravimo DocType:

```text
Service Order
```

i onda rešavamo pitanja jedno po jedno:

- Gde računam ukupnu cenu?
- Gde proveravam da li je serijski broj obavezan?
- Gde šaljem e-mail nakon `submit`?
- Gde sinhronizujem podatke sa drugim sistemom?
- Gde dodajem logiku ako ne smem da menjam ERPNext?

### Prva velika ideja Frappe-a

Frappe je framework u kome se skoro sve vrti oko Document objekta i njegovog životnog ciklusa.

Sve ostalo se vrti oko njega.

Kao programer, šta zapravo radiš?

- Praviš novi Document

  ```python
  doc = frappe.new_doc("Customer")
  doc.customer_name = "ABC"
  doc.insert()
  ```
  
  ili
  
  ```python
  doc = frappe.get_doc({
      "doctype": "Customer",
      "customer_name": "ABC"
  })
  doc.insert()
  ```

- Menjaš postojeći

  ```python
  doc = frappe.get_doc("Customer", "CUST-0001")
  
  doc.mobile_no = "064..."
  
  doc.save()
  ```

- Čitaš podatke

  ```python
  frappe.get_all(...)
  ```
  
  ili
  
  ```python
  frappe.db.get_value(...)
  ```

- Reaguješ na događaje

  Gde pišem kod?
  
  ```text
                      Želim da napišem kod
  
                             │
            ┌────────────────┴────────────────┐
            │                                 │
        Logika pripada                  Logika NE pripada
        ovom DocType-u                 ovom DocType-u
            │                                 │
            ▼                                 ▼
       Controller                        Hook
  ```
  
  To je osnovno pravilo.
  
  **Primer**
  
  Napravio si DocType:
  
  ```text
  Service Order
  ```
  
  Sada postavimo nekoliko pitanja.
  
  - **Treba izračunati ukupnu cenu.**  
    Gde? U:
  
    ```python
    class ServiceOrder(Document):
    ```
  
    Zašto? Jer je to **sastavni deo Service Order-a**.
    Bez toga dokument nije ispravan.

  - **Treba poslati e-mail**  
    Posle Submit-a.  
    Ako je slanje e-maila deo samog poslovnog procesa Service Order-a, može ići u kontroler.  Ali... Ako praviš aplikaciju koja šalje SMS za više različitih DocType-ova:

    - Sales Invoice
    - Purchase Invoice
    - Service Order

    onda je to već:

    ```text
    Hook
    ```

    Zašto? Jer više nije logika jednog dokumenta.
    To je logika aplikacije.

    **Pravilo**
    Controller odgovara na pitanje: **Kako se ponaša ovaj dokument?**.  
    Hook odgovara na pitanje: **Šta moja aplikacija želi da uradi kada se nešto dogodi?**  

    **Jedan primer**
  
    Controller:

    ```python
    class ServiceOrder(Document):
    
        def validate(self):
            if self.total == 0:
                frappe.throw(...)
    ```

    To je prirodno.

    Hook:

    ```python
    doc_events = {
        "Service Order": {
            "on_submit": "my_app.sms.send_sms"
        }
    }
    ```

    Ovo nije deo dokumenta. Ovo je dodatna funkcionalnost aplikacije.

  - **A šta je Override?**

    Override kaže: "Ne sviđa mi se postojeći kontroler." Koristi moj.
    To je ozbiljna intervencija.

    Većina projekata ga koristi veoma retko.

### Redosled evaulacije koda

Do sada smo pratili:

```text
    get_doc() -> get_controller() ->
...
```

To je bilo korisno jednom. Ali više ne. Od sada ćemo pratiti:

```text
    insert() -> before_insert -> validate -> before_save -> SQL -> after_insert ->
    ...
```

To je ono što programer zaista mora da zna.

[Sadržaj][00]

[00]: 00%20Frape%20framework%20tutorijal.md
