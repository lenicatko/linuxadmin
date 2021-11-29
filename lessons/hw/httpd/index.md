# Úkol – httpd

Nastav webový server tak, aby soubory z adresáře `public_html` ve tvém domovském adresáři byly k dispozici pod `http//:<adresa>/~<jméno>/`, např. `http://192.168.122.133/~petr/`.
Budeš s tím dva problémy navíc:

Instrukce jsou v souboru `/etc/httpd/conf.d/userdir.conf`. Komentář v tomhle souboru používá číselnou reprezentaci práv:
* 711 znamená `rwx--x--x`
* 755 znamená `rwxr-xr-x`
* příkaz `stat -c '%a %A %n' <soubor>` ti pro daný soubor vypíše práva v číselné i „normální“ reprezentaci
* Obecně každá číslice ukazuje jednu trojici práv (uživatel, skupina, ostatní) jako součet čísel 1 (`x`), 2 (`w`) a 4 (`x`).


Na Fedoře je navíc zapnutý systém SELinux (Security Enhanced Linux), který z bezpečnostních důvodů znemožňuje přístup systémových služeb k uživatelským souborům. Budeou tedy potřeba ještě dva příkazy:
* `sudo setsebool -P httpd_enable_homedirs on` povolí přístup `apache` k `/home/~/public_html`
* `sudo restorecon -R /home` aktualizuje informace u souborů v `/home`

Zajisti, aby ostatní uživatelé na systému neměli přístup ke tvému domovskému adresáři (kromě `public_html`).


---

Zbývá jen dodat, že takhle se to dnes už nedělá – webové servery bývají
na zvláštních (např. virtuálních) počítačích, kde běží pouze server.
Uživatelé na takovém stroji nemají celé své domovské adresáře, ale jen
obsah ke sdílení přes web.
