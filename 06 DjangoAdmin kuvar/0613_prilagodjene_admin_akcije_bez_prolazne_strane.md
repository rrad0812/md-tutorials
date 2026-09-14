
# Prilagodjene admin akcije bez prolazne strane

[Sadržaj](00_sadrzaj.md)

Jedan od mojih projekata ima deo za parsiranje. Ponekad moram ručno ponovo da parsiram plejlistu (na primer kada ažuriram kod parsera nakon ispravke greške). Tako sam pronašao kul način da dodam metodu u Django ModelAdmin koja se može pokrenuti iz admina.

`your_app/admin.py`

```py
from django.contrib import admin
from .models import Playlist
from .tasks import parse_playlist

@admin.register(Playlist)
class PlaylistAdmin(admin.ModelAdmin):
    actions = ['parse']
    
    def parse(self, request, queryset):
        queryset.update(is_parsed=False)
        for playlist in queryset.all():
            parse_playlist.delay(url=playlist.url)
            self.message_user(request, "Playlists will be reparsed soon")
```

Interesantni delovi:

- Menjam "Playlist.is_parsed" polje na `False`, jer će moja lista biti parsirana uskoro.

    ```py
    queryset.update(is_parsed=False)
    ```

- Celery task "parse_playlist" i u toj liniji se task odlaže sa `.delay()` metodom.

    ```py
    parse_playlist.delay(url=playlist.url)
    ```

- Poruka korisniku koja če se pojaviti kada se parsiranje završi.

    ```py
    self.message_user()
    ```

[Sadržaj](00_sadrzaj.md)
