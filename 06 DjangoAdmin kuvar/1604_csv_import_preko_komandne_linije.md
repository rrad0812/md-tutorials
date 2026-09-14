
# CSV Import preko komandnde linije

[Sadržaj](00_sadrzaj.md)

Napravimo sledeću šemu direktorijuma aplikacije:

```shell
app_name/
  init .py
  admin.py
  apps.py
  models.py
  management/
    init .py
    commands/
      init .py
  migrations/
  tests.py
  views.py
```

Napravimo fajl `app_name/management/commands/load_app_name_csv.py` sa sadržajem:

```py
import csv
from app_name.models import Invite
from django.core.management.base import BaseCommand

class Command(BaseCommand):

    def handle(self, *args, **options):
        print "Loading CSV"
        csv_path = "./app_name_invites.csv" Iz ovog csv se učitavaju podaci
        csv_file = open(csv_path, 'rb')
        csv_reader = csv.DictReader(csv_file)
        for row in csv_reader:
            obj = Invite.objects.create(name=row['Name'], branch=row['Branch'])
            print obj
```

Pokrenimo gornji fajl:

```shell
python manage.py load_app_name_csv
```

Podaci će biti učitani u Invite tabelu iz fajla `./app_name_invites.csv`.

[Sadržaj](00_sadrzaj.md)
