
# Delovi Vrednosti

[Strukture u Gou][0203] | [Sadržaj][00] | [Nizovi isečci i mape u Gou][0205]

Članci koji slede nakon ovog će predstaviti više vrsta Go tipova. Da biste lako i dublje razumeli te članke, najbolje je prvo pročitati sledeći sadržaj u trenutnom članku pre čitanja tih članaka.

## Dve kategorije Go tipova

Go se može posmatrati kao jezik iz C-familije, što se može potvrditi iz prethodna dva članka o pokazivačima u Gou i strukturama u Gou. Memorijske strukture struktura i pokazivač tipova u Gou i Cu su veoma slične.

S druge strane, Go se takođe može posmatrati kao C jezički okvir. To se uglavnom ogleda u činjenici da Go podržava nekoliko vrsta tipova čije strukture memorije vrednosti nisu potpuno transparentne, dok je glavna karakteristika C tipova to što su memorijske strukture C vrednosti transparentne. Svaka C vrednost u memoriji zauzima jedan memorijski blok (jedan kontinuirani memorijski segment). Međutim, vrednost nekih vrsta Go tipova često može biti smeštena na više od jednog memorijskog bloka.

Kasnije, delove vrednosti (koji su raspoređeni po različitim memorijskim blokovima) nazivamo delovima vrednosti. Vrednost koja se nalazi na više od jednog memorijskog bloka sastoji se od jednog direktnog dela vrednosti i nekoliko osnovnih indirektnih delova na koje se poziva taj direktni deo vrednosti.

Gore navedeni pasusi opisuju dve kategorije Go tipova:

| Tipovi čije se vrednosti nalaze samo na jednom bloku memorije | Tipovi čije vrednosti mogu biti smeštene na više memorijskih blokova |
| ------------------------------------------------------------- | ------------------------------- |
| ( **solo direct value part** ) | ( **direct part -> underlying part** ) |
| Bulovski tipovi | slice tipovi |
| Numerički tipovi | Map tipovi |
| Pokazivački tipovi | Kanal tipovi |
| Nebezbedni pokazivački tipovi | Funkcijski tipovi |
| Strukture | Interfejs tipovi |
| Niz tipovi | String tipovi |

Sledeći članci o programu Go 101 daće detaljna objašnjenja za mnoge vrste tipova navedenih u gornjoj tabeli. Ovaj članak je samo da bi se napravila priprema za lakše razumevanje tih objašnjenja.

> [!Note]
>
> - Da li vrednosti interfejsa i stringova mogu da sadrže osnovne delove zavisi
> od kompajlera. Za standardnu implementaciju Go kompajlera, vrednosti
> interfejsa i stringova mogu da sadrže osnovne delove.
> - Teško je, čak i nemoguće, dokazati da li vrednosti funkcija mogu sadržati
> osnovne delove. U Go 101, videćemo da vrednosti funkcija mogu sadržati osnovne
> delove.

Vrste tipova u drugoj kategoriji donose mnogo pogodnosti Go programiranju tako što obuhvataju mnoge detalje implementacije. Različiti Go kompajleri mogu usvojiti različite interne implementacije za ove tipove, ali eksterno ponašanje vrednosti ovih tipova mora zadovoljiti zahteve navedene u Go specifikaciji.

Tipovi u drugoj kategoriji nisu baš fundamentalni tipovi za jezik, možemo ih implementirati od nule koristeći tipove iz prve kategorije. Međutim, kapsuliranjem nekih uobičajenih ili jedinstvenih funkcionalnosti i podržavanjem ovih tipova kao osnovnih komponenti u Gou, iskustva Go programiranja postaju prijatna i produktivna.

S druge strane, ove enkapsulacije usvojene u implementaciji tipova u drugoj kategoriji kriju mnoge interne definicije ovih tipova. Ovo sprečava Go programere da sagledaju celu sliku ovih tipova i ponekad stvara neke prepreke za bolje razumevanje Goa.

Da bi korisnici bolje razumeli tipovi u drugoj kategoriji i njihove vrednosti, u nastavku ovog članka biće uvedene definicije unutrašnje strukture ovih vrsta tipova. Detaljne implementacije ovih tipova ovde neće biti objašnjene. Objašnjenja u ovom članku su zasnovana na, ali nisu potpuno ista kao, implementacije koje koristi standardni Go kompajler.

## Dve vrste pokazivača u programu Go

Pre nego što prikažemo definicije unutrašnje strukture tipova u drugoj kategoriji, hajde da razjasnimo više o pokazivačima i referencama.

U pretposlednjem članku smo učili o pokazivačima u Gou. Tipovi pokazivača u tom članku su tipovi pokazivača sigurni po tipu. U stvari, Go takođe podržava tipove pokazivača koji nisu sigurni po tipu. **unsafe.PointerTip** koji je dat u **unsafe** standardnom paketu je kao **void\*** u C jeziku.

U većini ostalih članaka u Go 101, ako nije posebno navedeno, kada se pominje tip pokazivača, to se odnosi na tip pokazivača bezbedan po tipu. Međutim, u narednim delovima ovog članka, kada se pominje pokazivač, to može biti ili pokazivač bezbedan po tipu ili pokazivač nebezbedan po tipu.

Vrednost pokazivača čuva memorijsku adresu druge vrednosti, osim ako vrednost pokazivača nije pokazivač vrednosti **nil**. Možemo reći da vrednost pokazivača upućuje na drugu vrednost ili da je druga vrednost upućena vrednošću pokazivača. Vrednosti se takođe mogu indirektno referencirati.

- Ako vrednost strukture *a* ima polje pokazivača *b* koje referencira na vrednost *c*, onda možemo reći da vrednost strukture *a* takođe referencira na vrednost *c*.
- Ako vrednost *x* referencira (direktno ili indirektno) na vrednost *y*, i vrednost *y* referencira (direktno ili indirektno) na vrednost *z*, onda možemo reći i da vrednost *x* (indirektno) referencira na vrednost *z*.

U nastavku, tip strukture sa poljima tipova pokazivača nazivamo **tipom omotača pokazivača**, a tip čije vrednosti mogu da sadrže (direktno ili indirektno) pokazivače nazivamo tipom **držača pokazivača**. **Tipovi pokazivača** i **tipovi omotača pokazivača** su svi **tipovi držača pokazivača**. Tipovi nizova sa tipovima elemenata držača pokazivača su takođe tipovi držača pokazivača. (Tipovi nizova će biti objašnjeni u sledećem članku.)

## (Moguće) interne definicije tipova u drugoj kategoriji

Da bismo bolje razumeli ponašanje vrednosti druge kategorije tokom izvršavanja, nije loša ideja da pomislimo da su ovi tipovi interno definisani kao tipovi u prvoj kategoriji, koji su prikazani u nastavku. Ako niste mnogo koristili sve vrste Go tipova, trenutno ne morate da pokušavate da jasno shvatite ove definicije. Umesto toga, u redu je da samo steknete grub utisak o ovim definicijama i ponovo pročitate ovaj članak kada kasnije steknete više iskustva u Go programiranju. Poznavanje definicija u grubim crtama je dovoljno da pomogne Go programerima da razumeju tipove objašnjene u sledećim člancima.

### Interne definicije tipova mapa, kanala i funkcija

Interne definicije tipova mapa, kanala i funkcija su slične:

```go
// map types
type _map *hashtableImpl

// channel types
type _channel *channelImpl

// function types
type _function *functionImpl
```

Dakle, interno, tipovi ove tri vrste su samo pokazivački tipovi. Drugim rečima, direktni delovi vrednosti ovih tipova su interni pokazivači. Za svaku vrednost ovih tipova koja nije nula, njen direktni deo (pokazivač) upućuje na njen indirektni osnovni implementacioni deo.

Uzgred, standardni Go kompajler koristi heštabele za implementaciju mapa.

### Interna definicija tipova isečaka

Interna definicija tipova isečaka je ovakva:

```go
type _slice struct {
    // referencing underlying elements
    elements unsafe.Pointer
    
    // number of elements and capacity
    len, cap int
}
```

Dakle, interno, tipovi isečaka su tipovi struktura omotača pokazivača. Svaka vrednost isečka koja nije nula ima indirektni osnovni deo koji čuva vrednosti elemenata vrednosti isečka. Polje elements direktnog dela referencira indirektni osnovni deo vrednosti isečka.

### Interna definicija tipova stringova

Ispod je interna definicija za tipove stringova:

```go
type _string struct {
    elements *byte // referencing underlying bytes
    len      int   // number of bytes
}
```

Dakle, tipovi stringova su interno takođe tipovi struktura omotača pokazivača. Svaka vrednost stringa ima indirektni osnovni deo koji čuva bajtove vrednosti stringa, a indirektni deo je referenciran poljem **elements** te vrednosti stringa.

### Interna definicija tipova interfejsa

Ispod je interna definicija za opšte tipove interfejsa:

```go
type _interface struct {
    dynamicType  *_type         // the dynamic type
    dynamicValue unsafe.Pointer // the dynamic value
}
```

Interno, tipovi interfejsa su takođe tipovi struktura omotača pokazivača. Interna definicija tipa interfejsa ima dva polja pokazivača. Svaka vrednost interfejsa koja nije nula ima dva indirektna osnovna dela koja čuvaju dinamički tip i dinamičku vrednost te vrednosti interfejsa. Na dva indirektna dela se pozivaju polja **dynamicType** i **dynamicValue** te vrednosti interfejsa.

U stvari, za standardni Go kompajler, gornja definicija se koristi samo za prazne tipove interfejsa. Prazni tipovi interfejsa su tipovi interfejsa koji ne određuju nikakve metode. Više o interfejsima možemo saznati kasnije u članku interfejsi u Go-u. Za tipove interfejsa koji nisu prazni, koristi se definicija kao sledeća.

```go
type _interface struct {
    dynamicTypeInfo *struct {
        dynamicType *_type       // the dynamic type
        methods     []*_function // method table
    }
    dynamicValue unsafe.Pointer  // the dynamic value
}
```

Polje **methods** polja **dynamicTypeInfo** vrednosti interfejsa čuva implementirane metode dinamičkog tipa vrednosti interfejsa za tip (interfejs) vrednosti interfejsa.

### Delovi osnovne vrednosti se ne kopiraju u dodele vrednosti

Sada smo saznali da su interne definicije tipova u drugoj kategoriji tipova držača pokazivača (pokazivača ili omotača pokazivača). Poznavanje ovoga je veoma korisno za razumevanje ponašanja kopiranja vrednosti u Go-u.

U jeziku Go, svaka dodela vrednosti (uključujući prosleđivanje parametara itd.) je **plitka kopija** vrednosti ako uključene vrednosti odredišta i izvora imaju isti tip (ako su im tipovi različiti, možemo pretpostaviti da će vrednost izvora biti implicitno konvertovana u tip odredišta pre nego što se izvrši ta dodela). Drugim rečima, samo se direktni deo vrednosti izvora kopira u vrednost odredišta prilikom dodele vrednosti. Ako vrednost izvora ima deo(love) osnovne vrednosti, onda će se direktni delovi vrednosti odredišta i izvora pozivati na isti deo(love) osnovne vrednosti, drugim rečima, vrednosti odredišta i izvora će deliti isti deo(love) osnovne vrednosti.

U stvari, gore navedeni opisi nisu 100% tačni u teoriji, za stringove i interfejse. Zvanični Go FAQ kaže da osnovni deo dinamičke vrednosti vrednosti interfejsa treba da se kopira takođe kada se kopira vrednost interfejsa. Međutim, pošto je dinamička vrednost vrednosti interfejsa samo za čitanje, standardni Go kompajler/okruženje za izvršavanje ne kopira osnovne delove dinamičke vrednosti prilikom kopiranja vrednosti interfejsa. Ovo se može posmatrati kao optimizacija kompajlera. Ista situacija je i za string vrednosti i ista optimizacija (koju je napravio standardni Go kompajler/okruženje za izvršavanje) je napravljena za kopiranje string vrednosti. Dakle, za standardni Go kompajler/okruženje za izvršavanje, opisi u poslednjem odeljku su 100% tačni, za vrednosti bilo kog tipa.

Pošto indirektni osnovni deo ne mora da pripada isključivo nijednoj vrednosti, on ne doprinosi veličini koju vraća funkcija **unsafe.Sizeof**.

## O terminologiji „Tip reference“ i „Referentna vrednost“

Reč referenca u Go svetu je velika zbrka. Donosi mnogo zabune Go zajednici. Neki članci, uključujući i neke zvanične, koriste referencu kao kvalifikatore tipova i vrednosti ili tretiraju referencu kao suprotnost vrednosti. Ovo se snažno ne preporučuje u Go 101. Zaista ne želim da se raspravljam o ovoj tački. Ovde ću samo navesti neke apsolutne zloupotrebe reference :

- Samo tipovi slice, map, channel i function su referentni tipovi u Go-u. (Ako nam je potrebna terminologija referentnih tipova u Go-u, onda ne bi trebalo da isključujemo tipove držača pokazivača iz referentnih tipova).
- Reference su suprotnosti vrednosti. (Ako nam je potrebna terminologija referentnih vrednosti u Go-u, onda referentne vrednosti posmatrajte kao posebne vrednosti, umesto kao suprotnosti vrednosti.)
- Neki parametri se prosleđuju po referenci. (Žao nam je, svi parametri se prosleđuju kopiranjem, odnosno direktnim delovima, u Go-u.)

Ne mislim da su terminologije referentnog tipa ili referentne vrednosti potpuno beskorisne za Go, samo mislim da nisu baš bitne i da donose mnogo zabune pri korišćenju Goa. Ako nam ove terminologije i trebaju, više volim da ih definišem kao držače pokazivača. I, moje lično mišljenje je da je najbolje ograničiti reč reference samo na predstavljanje odnosa između vrednosti koristeći je kao glagol ili imenicu, a nikada je ne koristiti kao pridev. Ovo će izbeći mnoge zabune u učenju, podučavanju i korišćenju Goa.

[Strukture u Gou][0203] | [Sadržaj][00] | [Nizovi isečci i mape u Gou][0205]

[0203]: 0203_Strukture.md
[00]: 00_Sadrzaj.md
[0205]: 0205_Nizovi_isečci_i_mape.md
