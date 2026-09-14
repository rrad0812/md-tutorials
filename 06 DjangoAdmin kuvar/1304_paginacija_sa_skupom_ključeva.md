
# Paginacija sa skupom ključeva

[Sadržaj](00_sadrzaj.md)

Da biste rešili spor `OFFSET` problem, možete ga zameniti paginacijom skupa ključeva (ili pretrage). U paginaciji skupa ključeva, svaku stranicu preuzima poređano polje poput `id` ili `created_at` datuma. Umesto ponavljanja brojanja stranica, kao što se `OFFSET` to dešava, paginacija ključeva filtrira direktno poredano polje.

**Primer korišćenja paginacije skupa ključeva**:

```sql
SELECT *
  FROM user
  WHERE id < 60 -- The last item in the previous page
  ORDER BY id DESC
  LIMIT 10
```

Ovaj pristup se malo razlikuje od `OFFSET` zbog toga što morate znati vrednost na kojoj počinjete. Zamislite da ste na četvrtoj strani tabele, a znate prvu `id` na petoj. Umesto da prebrojavate sve zapise, možete ići direktno na taj `id` i vratiti sledećih deset stavki.

Ovo funkcioniše jer indeksi mogu efikasno podržati ovakav upit. Traženje indeksa B-stabla da vrati 10 id unosa pre određenog id zahteva samo učitavanje 10 unosa indeksa. Uporedite to sa upotrebom `OFFSET`, gde se moraju učitati svi unosi do pomaka, a zatim i navedena granica, što čini velike pomake veoma skupim.

Kada koristite paginaciju ključeva zajedno sa pravim indeksom, videćete značajno povećanje performansi. Kompleksnost vremena da nađe neki zapis u bazi podataka je konstantna. Na primer, traženje poslednje stranice velike tabele generisane gore traje samo 78 ms.

Ova metoda takođe štiti od oskudnih podataka. Ako je korisnik izbrisan i redosled više nije uzastopan u tabeli, to ne utiče na paginaciju skupova ključeva; bez problema će preskočiti nedostajuću vrednost.

[Sadržaj](00_sadrzaj.md)

## Kompromisi paginacije skupova ključeva

Paginacija ključeva dolazi sa nekoliko kompromisa. Bez `OFFSET`-a, ne znate tačno koliko stranica ima u tabeli niti na kom se broju stranice trenutno nalazite. Osim toga, za paginaciju skupova ključeva potrebno je sortirati polje na vašem modelu. Sekvencijalni ID-ovi i polja datuma dobro funkcionišu.

[Sadržaj](00_sadrzaj.md)
