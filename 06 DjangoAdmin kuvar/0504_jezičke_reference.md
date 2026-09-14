
# Jezičke reference

[Sadržaj](00_sadrzaj.md)

`DjangoQL` se dobija sa izuzetnim `SyntaxHelp`, koji se može naći u Django admin (vidi `Syntax Help` link na auto-completion popup-u). Ovde je kratak zaključak:

> DjangoQL syntaksa je Python sintaksa, sa malim razlikama. U osnovi vi samo referencirate model polja kao što bi i u Pajton kodu, potom primenjujete komparacijske  i logičke operatore i zagrade.

DjangoQL je osetljiv na veličinu slova.

- Polja modela: tačno ista kao što su definisana u Pajton kodu.
- Pristup ugnježdenim osobinama preko dot operatora, na primer: `author.last_name`;
- Stringovi moraju biti double-quoted. Jednostruki navodnici nisu podržani.
- Da bi uradili escape treba koristiti duple navodnike \";
- Logičke i null vrednosti: True, False, None. Primetite da oni mogu biti u kombinaciji samo sa operatorima jednakosti, tako da možete pisati `published = False` ili `date_published = None`, ali `published >` False će izazvati grešku;
- Logički operatori: `and`, `or`;
- Operatori poredjenja: `=`, `!=`, `<`, `<=`, `>`, `>=` rade kao što se očekuje.
- `~` i `!~` test da li string ima ili nema substring (`icontains`);
- testna vrednost naspram liste: `in`, `not in`. Primer: `pk in (2, 3)`.

[Sadržaj](00_sadrzaj.md)
