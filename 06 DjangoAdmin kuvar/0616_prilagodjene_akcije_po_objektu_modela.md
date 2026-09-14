
# Prilagodjene akcije po objektu modela

[Sadržaj](00_sadrzaj.md)

Ponekad želite da izvršite odredjenu akciju samo na jednom objektu. ‘actions’ padajućalista pravi to mogućim, al je potrebno nekoliko klikova da bi se izvršila. Postoji i jednostavniji način, implementiranje hiperlinka za svaku vrstu pojedinačno sa željenom akcijom:

```py
class PictureAdmin(admin.ModelAdmin):
    list_fields = (..., 'mail_link', )

    def mail_link(self, obj):
        dest = reverse('admin:myapp_pictures_mail_author',
        kwargs={'pk': obj.pk})
        return format_html('<a href="{url}">{title}</a>', url = dest, title='send mail')
    
    mail_link.short_description = 'Show some love'
    mail_link.allow_tags = True


def get_urls(self):
    urls = [
        url('^(?P<pk>\d+)/sendaletter/?$',
        self.admin_site.admin_view(self.mail_view),
        name='myapp_pictures_mail_author'),
    ]
    return urls + super(PictureAdmin, self).get_urls()

def mail_view(self, request, *args, **kwargs):
    obj = get_object_or_404(Picture, pk=kwargs['pk'])
    send_mail('Feel the granny\'s love', 'Hey, she loves your pet!',
        'granny@yoursite.com', [obj.author.email])
    self.message_user(request, 'The letter is on its way')
    return redirect(reverse('admin:myapp_picture_changelist'))
```

Link se pojavljuje pored svake vrste, na kraju, klikom na njega biće poslat mejl.

[Sadržaj](00_sadrzaj.md)
