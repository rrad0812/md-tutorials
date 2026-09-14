
# CSV Export preko komandne linije

[Sadržaj](00_sadrzaj.md)

Napravimo fajl:

U `app_name/management/commands/dump_app_name_csv.py`

```py

import os
import csv
from django.conf import settings
from academy.models import Invite
from django.core.management.base import BaseCommand

class Command(BaseCommand):
    
    def handle(self, *args, **options):
        print "Dumping CSV"
        csv_path = os.path.join(settings.BASE_DIR, "dump_invites.csv")
        csv_file = open(csv_path, 'wb')
        csv_writer = csv.writer(csv_file)
        csv_writer.writerow(['name', 'branch', 'gender', 'date_of_birth',
            'race', 'notes', 'reporter'])

        for obj in Invite.objects.all():
            row = [obj.name, obj.branch, obj.gender, obj.date_of_birth,
                obj.race,obj.notes, obj.reporter]
            csv_writer.writerow(row)
```

Pokrenimo gornji fajl:

```shell
python manage.py dump_app_name_csv
```

Podaci će biti izvezeni u fajl `dump_invites.csv`.

[Sadržaj](00_sadrzaj.md)
