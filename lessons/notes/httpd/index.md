# Webový server

Vyzkoušejme si, jak na Linuxu nastavit zprovoznit a službu.
Typicky příklad takové služby je webový server, tedy program,
který odpovídá prohlížečům na HTTP dotazy.

> [note]
> Základní webový server posílá jen soubory z disku;
> kdybys chtěl{{a}} zprovoznit webovou *aplikaci* (např. s přihlášováním
> uživatelů), budeš potřebovat spustit ještě *aplikační server*
> napsaný např. v Pythonu, a propojit ho s HTTP serverem.
> To je nad rámec této lekce.


## Firewall

Ještě než půjdeme na náš dnešní úkol, musíme pozměnit nastavení firewallu.
Firewall je služba, která z bezpečnostních důvodů povoluje nebo blokuje
připojení.

Budeš chtít spustit webový server, který bude poslouchat na portu 80.
Port je číslo, které říká, k jaké službě se chceš připojit.
Internetové komunikační protokoly, tedy způsoby jak se spolu počítače „baví“
mají často přiřazený výchozí port, na kterém pracují.
Například:

- webserver používají porty 80 (protokol HTTP) nebo 443 (HTTPS);
- e-mail používá porty 25 (SMTP), 109 (POP2), 110 (POP3), 465 (SMTPS), 995 (POP3S);
- vzdálené připojení používá port 22 (SSH).

Komunikovat lze i přes jiné porty než ty oficiální: jde jenom o konvenci.
Když otevřeš v prohlížeči `https://seznam.cz/`, použije se port 443:
tahle adresa je zkratkou pro `https://seznam.cz:443/`.
Kdybys otevřel{{a}} `https://seznam.cz:12345/`, použije se port 12345
(na kterém ovšem v tomhle případě na druhé straně nic neposlouchá).

Porty do čísla 1023 jsou vyhrazeny pro systémové služby; smí je používat
jen superuživatel (`root`), resp. systémové služby které `root` spravuje.
Vyšší porty může používat kdokoli.
(Znáš-li webové frameworky jako Flask, všimni si že jejich vývojové servery
často poslouchají na portech jako 8000, 8008, 8080 – čísla co lidem připomínají
80, ale jsou vyšší než 1023.)

Ale zpátky k webovému serveru.
My budeme používat oficiální port pro HTTP a je tedy potřeba zajistit,
aby ho firewall neblokoval.
To uděláš příkazem:

```console
$ sudo firewall-cmd --add-service=http
Warning: ALREADY_ENABLED: 'http' already in 'FedoraWorkstation'
success
``` 

Pokud máš firewall už nastavený, příkaz vypíše výše uvedené varování,
že port už je otevřen.
Když varování nevidíš, tím lépe – v tom případě bylo otevření portu nutné :)

> [note]
> Kdybys chtěl{{a}} otevřít konrétní číslo portu, použil{{a}} bys místo
> `--add-service=http` přepínač `--add-port=80/tcp`.
> Funguje to stejně.

Dále nastav, že se má port pro `http` povolit i po startu počítače.
Ten samý příkaz spust s přepínačem `--permanent` navíc:

```console
$ sudo firewall-cmd --permanent --add-service=http
Warning: ALREADY_ENABLED: http
success
```

Varování opět buď uvidíš nebo ne.

Při administraci služeb často narazíš na rozdíl mezi tím, jaké má služba
nastavení *teď* a jak bude nastavená *po restartu*.
Jsou to dvě odlišné věci a bývají na ně různé sady/varianty příkazů.


## Instalace

Už víš, jak a odkud instalovat, tak je vhodný čas na dnešní úkol.
Nainstalujeme si tradiční webový server, `Apache`, neboli `httpd`.
Je to dobrý příklad služby, na kterém si ukážeme,
jak systémové služby konfigurovat a jak se o takové procesy starat.

Jméno `httpd` je zkratka pro *HTTP dæmon* – služba,
která se stará o komunikaci přes protokol HTTPD.
Čili webový server.

> [note]
> K Apachi existují různé alternativy.
> Dnes se často používá `nginx`, který je novější, rychlejší a štíhlejší.
> Apache je starší a robustnější.

Nainstaluj `httpd` pomocí příkazu `sudo dnf install httpd`.

Zkus se podívat dlouhý seznam souborů, které balíček obsahuje:

```console
$ rpm -q -l httpd
/etc/httpd/conf
/etc/httpd/conf.d/autoindex.conf
/etc/httpd/conf.d/userdir.conf
/etc/httpd/conf.d/welcome.conf
/etc/httpd/conf.modules.d
/etc/httpd/conf.modules.d/00-base.conf
/etc/httpd/conf.modules.d/00-dav.conf
...
```

## Správa služby httpd

Fedora se řídí filozofií, že nově nainstalované balíčky jsou sice k dispozici,
ale nespouští se automaticky.
Správce si je musí nastavit a spustit sám.
Existují linuxové distribuce (jako Ubuntu), které se zaměřují na to,
aby systém vyžadoval malé množství konfigurace.
Na takovém systému by se služba po instalaci rovnou spustila
s nějakou výchozí konfigurací.


### Start, stop

`httpd` je démon. O démonech jsme si povídali minule.
O démony se stará systémový proces, který se jmenuje `systemd`. 
K ovládání `systemd`, tedy spouštění a správu démonů,
slouží příkaz `systemctl`.

Když spustíš jen příkaz `sudo systemctl`, uvidíš velký seznam věcí,
co běží na počítači (procesy, zařízení, připojené souborové systémy, logování,
nastavení automatického času, internetové připojení, atd.).

> [note]
> Používáme *Fedora Workstation*, systém přednastavený pro osobní počítače,
> kde by mělo pro uživatele “všechno fungovat” – tiskárny, připojování
> CD/DVD/Flash disků, kalibrace barev monitoru, Bluetooth, síť, Wi-Fi,
> synchronizace času a tak dále.
> Proto v základním systému běží služeb spousta.
> Kdyby sis nainstaloval{{a}} *Fedora Server*, běželo by jich mnohem míň;
> každou službu navíc bys musel{{a}} nastavit a zapnout.
> Takový systém je pak lépe přizpůsobený konkrétnímu účelu – třeba servírování
> webových stránek.

Pro nás bude důležitá služba `httpd`, uvedená jako `httpd.service`.
Můžeš ji buď najít v dlouhém seznamu (rolovátko je `less`,
hledá se přes `/`), nebo výpis filtrovat.
Příkaz `sudo systemctl` bez argumentů je zkratka pro `sudo systemctl list-units`,
výpis služeb.
Když chceš výpis filtrovat, musíš podpříkaz `list-units` uvést:

```
$ sudo systemctl list-units                 # všechno
$ sudo systemctl list-units '*.service'     # jen služby
$ sudo systemctl list-units httpd.service   # jen služba httpd
```

U jedné služby je ale lepší se podívat na detailní výpis stavu (jestli běží,
na jakém portu poslouchá, seznam procesů, poslední zprávy atp.):

```console
$ sudo systemctl status httpd
```

V tomto a následujících příkazech `systemctl` je `httpd` zkratka pro
`httpd.service` – když nezadáš „druh” věci co spravuješ, `systemctl`
předpokládá že jde o službu.

Službu `httpd` spusť a znovu se podívej na její stav:

```console
$ sudo systemctl start httpd
$ sudo systemctl status httpd
```

Nastartovanou službu máš teď k dispozici.
Kdyby proces spadl s chybou, `systemd` ho nastartuje znova.

> [note]
> Kdyby proces padal opakovaně `systemd` to po nějaké době vzdá.
> Přesné chování jde pak nastavit.

Nyní můžeš načíst webovou stránku z prohlížeče (ať už virtuálního nebo na reálném počítači).
Jako adresu zadej IP adresu virtuálního počítače.

Kdyby to nežlo, použij příkaz `curl localhost | head`.
Měl by stáhnout a vypsat HTML kód se značkami jako `<html ...>` a `<head>`.

Pro vypnutí služby `httpd` existuje příkaz `stop`:

```console
$ sudo systemctl stop httpd
```

Zkus se podívat na `status` vypnuté služby.
Pak `http` zase zapni.

Až budeš nastavovat systém bez grafického prohlížeče, bude se ti hodit umět se
připojit z příkazové řádky.
Pomůže ti to zjistit, jestli je problém ve službě nebo ve špatně nastaveném
firewallu, který blokuje spojení „zvenčí“.
HTTP dotazy umí posílat `curl`; jako argument uvedeš adresu jako v prohlížeči.
Zkus to.

Můžeš si dokonce nainstalovat i textového prohlížeč,
např. `links`: po `sudo dnf install links` se spustí příkazem `links <adresa>`.
Ukončuje se pomocí <kbd>Q<kbd>.

Pomocí klávesy <kbd>q</kbd> `links` opustíš.


## Enable, disable

Webserver budeš mít k dispozici tak dlouho, dokud poběží služba `httpd`.
To můžeš zařídit přes `systemctl`, ale služba se ukončí i když vypneš počítač.
Po opětovném startu systému se webserver sám nezapne.
Na to potřebuješ ještě nastavit, aby se služba zapnula vždy po startu systému:

```console
$ sudo systemctl enable httpd
```

Příkaz `start` se týká jen tohoto běhu počítače, nastartuje službu pro tebe tady a teď. 
Příkaz `enable` naopak řídí jen zapínání při startu systému, nespouští službu přímo.

Podobně fungují opačné příkazy `stop` a `disable`:
`stop` službu vypne, `disable` nastaví, aby se automaticky nespouštěla.

Abys otestoval{{a}}, jestli máš službu korektně nastavenou,
je dobré restartovat počítač (příkazem `sudo reboot`).
Pokud potom běží tak, jak jsi ji naplánoval{{a}}, je vše v pořádku.


### Restart a reload

Když změníš konfiguraci, server na to nebude reagovat automaticky.
Musíš ho restartovat, tedy vypnout a zapnout.
Při startu si konfiguraci načte.
Místo `start` a `stop` na to použij příkaz `restart`,
který řeší potřebné detaily (např. počká na ukončení, než službu znovu zapne):

```console
$ sudo systemctl restart httpd
```

Některým službám můžeš místo restartu rovnou říct,
že mají načíst novou konfiguraci.
K tomu slouží příkaz `reload`:

```console
$ sudo systemctl reload httpd
```

Na rozdíl od `restart` se v tomto případě nepřeruší aktivní spojení,
které `httpd` obsluhuje.
Je to tedy příjemnější varianta.

Když už jsme u nastavení webového serveru, kde ho najdeš?
Možná tě nepřekvapí, že je v `/etc/httpd`, respektive hlavně v `/etc/httpd/conf.d/`.
Je v `/etc/httpd/conf.d/`, kde najdeš několik souborů. 
Můžeš zkusit změnit `/etc/httpd/conf.d/welcome.conf`.
Tento soubor definuje obsah stránky, kterou dostaneš, když se zeptáš na IP adresu počítače s běžícím webovým serverem.
Když tu všechno zakomentuješ (s použitím znaku `#`),
uděláš `reload` a znovu načteš stránku v prohlížeči,
mělo by se začít zobrazovat něco jiného.


### DocumentRoot a UserDir

Apache servíruje obsah adresáře určitým způsobem.
V souboru `/etc/httpd/conf.d/autoindex.conf` je nastavení toho, jak se generují výpisy adresářů.
V `/etc/httpd/conf/httpd.conf` je základní nastavení a to hlavní v něm je `DocumentRoot`, kde by mělo být řečeno, kde jsou uloženy dokumenty, kterými disponuje webový server.
Typicky je to `/var/www/html/`.
Vytvoř si v něm soubor `hello.txt` a napiš do něj *Hello, World!*
Podívej se na stránku a zkontroluj, že se ten soubor objeví v prohlížeči.
Obsah `/var/www/html` už není nastavení serveru. Když přijde dotaz, co je na té hlavní stránce, zobrazí se obsah toho adresáře. Tohle bude fungovat i bez reloadu serveru, stačí obnovení stránky (klávesou <kbd>F5</kbd>.

V `/etc/httpd/conf/httpd.conf` je ještě jedno nastavení, a to `UserDir`. 
Nastavuje, že pokud má uživatel vytvořený adresář `public_html`, pak tento adresář lze zobrazit pod `/~user/`, kde *user* je jméno uživatelského účtu. 
Na prvních unixových systémech to bylo skvělé, každý si mohl jednoduše dát na web, co chtěl.
Pak se zjistilo, že je to bezpečnostní riziko, protože se podle toho dá velmi jednoduše zjistit, jestli uživatel daného jména existuje na tom počítači a zkoušet se na ně připojit.
Z tohoto důvodu je to standardně vypnuté, ale v případě potřeby můžeš tuto funkcionalitu opět zapnout .
