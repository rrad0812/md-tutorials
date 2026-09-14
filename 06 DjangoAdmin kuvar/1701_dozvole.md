
# Dozvole

[Sadržaj](00_sadrzaj.md)

Django pruža dobar okvir za autentifikaciju uz usku integraciju sa Django adminom. Django admin ne primenjuje posebna ograničenja na korisnika admina. To može dovesti do opasnih scenarija koji mogu ugroziti vaš sistem.

Da li ste znali da korisnici koji pripadaju `staff`-u i koji upravljaju drugim korisnicima u admin-u mogu da uređuju svoje dozvole? Da li ste znali da oni takođe moguda naprave superuser-e? U administratoru Django ne postoji ništa što to sprečava, pa sve je na vama!

Na kraju ovog vodiča znaćete kako da zaštitite svoj sistem:

- Od eskalacije dozvola sprečavanjem korisnika da uređuju sopstvene dozvole.
- Održavajte dozvole urednim i održivim, prisiljavanjem korisnike da koriste samo grupe.
- Sprečite curenje dozvola kroz prilagođene akcije izričitim sprovođenjem potrebnih dozvola.

Dozvole su škakljive. Ako ne podesite dozvole, sistem izlažete riziku od uljeza,curenja podataka i ljudskih grešaka. Ako zloupotrebite dozvole ili ih previše koristite, rizikujete da ometate svakodnevne operacije.

[Sadržaj](00_sadrzaj.md)
