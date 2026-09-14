
# Reverzne veze

[Sadržaj](00_sadrzaj.md)

## related_name

`related_name` atribut u `ForeignKey` poljima je jako koristan. Daje nam mogućnost da definišemo značajno ime za reverznu vezu.

> [!Note]
>
> **Pravilo**: Ako niste sigurni šta bi trebalo da bude `related_name`, koristite množinu imena modela.

```py
class Company:
    name = models.CharField(max_length=30)

class Employee:
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    company = models.ForeignKey(Company,
        on_delete=models.CASCADE, related_name='employees')
```

To znači da "Company" model će imati poseban atribut "employees", koji će vraćati `QuerySet` sa svim zaposlenim instancama povezanih sa kompanijom.

```py
google = Company.objects.get(name='Google')
google.employees.all()
```

Možete koristiti reverznu vezu za modifikovanje company polja na Employee instancama:

```py
vitor = Employee.objects.get(first_name='Vitor')
google = Company.objects.get(name='Google')
google.employees.add(vitor)
```

[Sadržaj](00_sadrzaj.md)

## related_query_name

Ova se vrsta veze primenjuje i na filterske upite. Na primer, ako želim da izlistam sve kompanije koji zapošljavaju ljude sa imenom "Vitor", uradiću sledeće:

```py
companies = Company.objects.filter(employees__first_name='Vitor')
```

Ako želite sa promenite ime veze, evo kako da to uradite:

```py
class Employee:
    first_name = models.CharField(max_length=30)
    last_name = models.CharField(max_length=30)
    company = models.ForeignKey(Company, on_delete=models.CASCADE,
        related_name='employees', related_query_name='person'
)
```

Tada bi korišćenje bilo:

```py
companies = Company.objects.filter(person__first_name='Vitor')
```

Da bi bilo konzistentno `related_name` ide u množini a `related_query_name` ide u jednini.

[Sadržaj](00_sadrzaj.md)
