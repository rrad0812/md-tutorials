
# Puni prilagodjeni search lookup

[Sadržaj](00_sadrzaj.md)

`.get_lookup_name()` i `.get_lookup_value(value)` hooks pokrivaju više jednostavnih slučajeva korišćenja. Ali ponekad nisu dovoljni i vi želite punu prilagodjenu search logiku. U tim slučajevima možete nadjačati glavni `get_lookup()` metod polja.

Primer ispod demonstrira pretragu po "years" korisnika:

```py
from djangoql.schema import DjangoQLSchema, IntFieldclass 

class UserAgeField(IntField):
    """
    Search by given number of full years
    """
    model = Username = 'age'

    def get_lookup_name(self):
        """
        We'll be doing comparisons vs. this model field
        """
        return 'date_joined'

    def get_lookup(self, path, operator, value):
        """
        The lookup should support with all operators compatible with IntField
        """
        if operator == 'in':
            result = None
            for year in value:
                condition = self.get_lookup(path, '=', year)
                result = condition if result is None else result | condition
            return result

        elif operator == 'not in':result = None
            for year in value:
                condition = self.get_lookup(path, '!=', year)
                result = condition if result is None else result & condition
            return result
        
        value = self.get_lookup_value(value)
        search_field = ' '.join(path + [self.get_lookup_name()])
        year_start =  self.years_ago(value + 1)
        year_end = self.years_ago(value)
        
        if operator == '=':
            return ( Q(**{'%s gt' % search_field: year_start}) &Q(**{'%s lte' % search_field: year_end})
        )
        elif operator == '!=':
            return (Q(**{'%s lte' % search_field: year_start}) |Q(**{'%s gt' %
                search_field: year_end})
        )
        elif operator == '>':
            return Q(**{'%s lt' % search_field: year_start})
        elif operator == '>=':
            return Q(**{'%s lte' % search_field: year_end})
        elif operator == '<':
            return Q(**{'%s gt' % search_field: year_end})
        elif operator == '<=':
            return Q(**{'%s gte' % search_field: year_start})

    def years_ago(self, n):
        timestamp = now() 
        
        try:
            return timestamp.replace(year=timestamp.year - n)
        except ValueError:
            February 29
        return timestamp.replace(month=2, day=28, year=timestamp.year - n)

class UserQLSchema(DjangoQLSchema):
    
    def get_fields(self, model):
        fields = super(UserQLSchema, self).get_fields(model)
        if model == User:
            fields += [UserAgeField()]return fields

@admin.register(User)
class CustomUserAdmin(DjangoQLSearchMixin, UserAdmin):
    djangoql_schema = UserQLSchema
```

[Sadržaj](00_sadrzaj.md)
