
# Završno o paginaciji

[Sadržaj](00_sadrzaj.md)

U ovom članku ste saznali kako funkcioniše paginacija u Djangu. Iako naivna paginacija dobro funkcioniše za male tabele, ova metoda brzo smanjuje performanse kako vaša tabela raste na milione redova. Štaviše, skok do zapisa duboko u tabeli biće veoma spor u upitu koji koristi `OFFSET`.

Dobra vest je da možete ubrzati stvari promenom `COUNT` upita. Pored toga, prelazak na paginaciju skupova ključeva poboljšaće performanse pretraživanja stranica i učiniće ih da rade u realnom vremenu. Django olakšava promenu podrazumevane konfiguracije, dajući vam moć da napravite efikasno rešenje za paginaciju u Djangu.

[Sadržaj](00_sadrzaj.md)
