# 17 Korišćenje asemblera

[prev][f16] [content][f0]

FreePaskal podržava upotrebu asemblera u kodu, ali ne i inline asemblerske makroe. Za više informacija o sintaksi asemblera specifičnoj za procesor i njenim ograničenjima, pogledajte Vodič za programere.

## 17.1 Asemblerske izjave

Sledi primer uključivanja asemblera u Paskal kod.

 ```pascal
 ...  
 Statements;  
 ...  
 Asm  
   the asm code here  
   ...  
 End;  
 ...  
 Statements;
```

Asemblerske instrukcije između ključnih reči `Asm` i `End` biće umetnute u asembler koji generiše kompajler. Uslovni izrazi se mogu koristiti u asemblerskom kodu, kompajler će ih prepoznati i tretirati kao bilo koji drugi uslovni izraz.

## 17.2 Asemblerske procedure i funkcije

Asemblerske procedure i funkcije se deklarišu pomoću asemblerske direktive. Ovo omogućava generatoru koda da izvrši brojne optimizacije generisanja koda.

Generator koda ne generiše nikakav stek frejm (ulazni i izlazni kod za rutinu) ako ne sadrži lokalne promenljive i parametre. U slučaju funkcija, ordinalne vrednosti moraju biti vraćene u akumulatoru. U slučaju vrednosti sa pokretnim zarezom, one zavise od ciljnog procesora i opcija emulacije.

[prev][f16] [content][f0]

[f0]: 00_sadrzaj.md
[f16]: 16_izuzeci.md
