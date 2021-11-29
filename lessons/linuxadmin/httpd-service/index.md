## Webový server

> [warning]
> Toto jsou jen poznámky, ne materiály k samostudiu

Už víš, jak a odkud instalovat, tak je vhodný čas na dnešní úkol.
Nainstalujeme si tradiční webový server, [Apache](http://httpd.apache.org/) neboli `httpd`.
Existují různé alternativy k Apachi, dnes se často používá `nginx`, který je rychlejší a štíhlejší;
Apache je starší a robustnější.
Je to dobrý příklad toho, jak konfigurovat systémové služby a jak se o takové procesy starat.

## Služby a démoni

Webový server je služba (angl. *service*): něco co lze zapnout a vypnout,
spouštět při zapnutí počítače, u čeho lze zkontrolovat stav (běží/neběží)
a podobně.
Kromě webových serverů existují služby pro e-mailové servery,
grafické systém pro vykreslování okýnek, řízení tiskových úloh, atd.

Většina služeb spouští takzvaný *démon* (angl. *dæmon*),
dlouho běžící proces, který je spuštěný „na pozadí“.
Nepouští se ze shellu, který by čekal na to, až takový proces skončí.

Démoni jsou typicky „odpoutaní“
od shellu a terminálu který ho spustil (aby např. nešel ovládat
pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd> nebo `kill %1`
nebo aby se nevypnul po ukončení terminálu).
Historicky se o tohle každý démon staral sám; dnes se o to většinou stará
proces který služby spouští.

Na Fedoře se o služby stará systémový proces, který se jmenuje `systemd`. 
My jako uživatelé máme k dispozici příkaz `systemctl`,
který říká `systemd` co má dělat.

Když spustíš jen příkaz `systemctl`, uvidíš dlouhý seznam věcí,
které na počítači „běží“ a o které se `systemd` stará: nejen služby
(*service*) ale i, zařízení (device), připojené souborové systémy (mount),
logování, nastavení automatického času, internetové připojení, atd.

Proces `systemd` se spouští jako první po zapnutí počítače
a stará se o to, aby se ve správném pořadí zapnuly všechny služby,
připojily všechny disky a podobně.
Ověř si, že má PID 1:

```console
$ ps -A | head
    PID TTY          TIME CMD
      1 ?        00:00:06 systemd
```

Příkaz `pstree` ti ukáže, že všechny ostatní procesy jsou mu „podřízené“.
Zkus si v `pstree -T | less` najít `pstree`: zjistíš, že `pstree` byl zpuštěn
z Bashe, který byl spuštěn z `gnome-terminal`, který byl spuštěn
(možná s několika mezikroky) ze `systemd`.
Kdybys `gnome-terminal` zavřel{{a}}, `less` i `pstree` se ukončí.

Ve výstupu `pstree` najdeš i démony: třeba `firewalld` je démon
(dlouho běžící proces), který se stará o firewall; `cupsd` je démon který
se stará o tisk a tiskárny (CUPS je zkratka Common Unix Printing System),
`sssd` se stará o přihlašovací účty (SSS je zkratka System Security Services),
`chronyd` zařizuje automatické nastavení hodin počítače podle internetu,
Všimni si konvence démony pojmenovávat s `d` na konci.
`systemd` tady není výjimkou, ale např. `NetworkManager`, který se stará o síť,
je démon bez `d` v názvu.

Webový server ale potřebuje víc než jen démona (tedy dlouho běžící proces,
který něco sdílí na internetu). Například:
* nastavení např. toho, co se má na webu zveřejnit
* služby na kterých závisí (např. webový server potřebuje funkční síť)
* nástroj na ovládání (např. vypnutí a zapnutí) démona
* nástroj na zjištění, jestli démon běží
* nějaký způsob na hlášení chyb nebo zjišťování návštěvnosti
* a tak dále.

Všechno tohle je nastavené v rámci *služby* (angl. *service*).
Jako administrátoři budeme spouštět, nastavovat a ovládat *služby*;
což typicky (ale ne vždy) zahrnuje spouštění, nastavování a ovládání
nějakého démona – procesu který danou službu zajišťuje.

> [note]
> Termín *Daemon* nemá nic společného s okultismem.
> Je to jméno bytosti která pracuje v pozadí,
> podobně jako fyzikům známý Maxwellův démon.
> Více viz [Wikipedia](https://en.wikipedia.org/wiki/Daemon_%28computing%29#Terminology).


## Instalace httpd

Nainstaluj `httpd` pomocí příkazu `dnf install httpd`.

Když se podíváš na všechny soubory, které tento balíček obsahuje, uvidíš dlouhý seznam:

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

Některé linuxové distribuce (např. Ubuntu) typicky službu po instalaci rovnou
spustí s nějakým výchozím nastavením, nebo umožňují základní nastavení
už při instalaci.

Fedora má filosofii jinou. Nově nainstalovaný software je
sice k dispozici, ale většinou se automaticky nespouští.
Správce si ho musí nastavit a spustit sám.

Protože budeme nastavovat systémovou službu – a tedy měnit nastavení
celého systému – musíš být přihlášen{{a}} jako superuživatel (`root`).
Udělej to pomocí `sudo -i`.
Příklady níže na nutnost být superuživatel upozorňují
použitím výzvy `#` místo `$`.

Software který spravuje systémové služby na Fedoře se jmenuje `systemd`.
Ten je dnes zdaleka nejrozšířenější, ale existují i jiné
(např. Upstart nebo Sys-V init), které fungují jinak.


### Start, stop, enable, disable

Spusť službu `httpd`:
```console
# systemctl start httpd
```
To spustí proces (démona) `httpd`.
Ověř si, že proces opravdu běží:

```console
# ps -A | grep httpd
   3955 ?        00:00:00 httpd
   3956 ?        00:00:00 httpd
   3958 ?        00:00:00 httpd
   3959 ?        00:00:00 httpd
   3960 ?        00:00:00 httpd
```
Vida, je jich hned několik.

V čem je tohle lepší než pouštět proces `httpd` přímo?
* `systemd` se stará o „démonizaci“ – odpoutání od terminálu a shellu,
  nastavení aktuálního adresáře, spuštění pod příslušným uživatelským účtem
  a podobně,
* `systemd` proces kontrolovat, takže kdyby např. skončil s chybou,
  `systemd` se ho pokusí znovu nastartovat.

Můžeš si nechat zobrazit stav služby – jestli běží,
jaké má PID, seznam spuštěných procesů, kolik využívá paměti, atp:
```console
# systemctl status httpd
```
Nyní můžeš načíst webovou stránku webového serveru přímo ve virtuálce.
Jako adresu zadej svoji IP adresu.
Máš-li dost silný počítač, můžeš ji otevřít v prohlížeči.
Ale jde to i v terminálu: zadej příkaz `curl` a IP adresu uveď jako argument.

`curl` je příkaz, který umí stahovat webové stránky.
Jako argument mu můžeš dát libovolnou webovou adresu,
např. `curl https://github.com`.

> [note]
> `curl` ukáže jen HTML kód, stránku „nevykreslí“.
> Jestli ale chceš, můžeš si nainstalovat do virtuálního počítače textový
> webový prohlížeč, např. `links`: `dnf install links`,
> spustí se pak příkazem `links <adresa>`.
> a ukončí se pomocí <kbd>Q</kbd>.

Webová stránka by ale měla být k dispozici i z jiného počítače –
„venku“ z toho virtuálního.
Jestli máš správně nastavenou síť a firewall, mělo by to jít:
zadej IP adresu virtuálního počítače do prohlížeče na svém počítači a
měla by se ti načíst testovací webová stránka.

Webserver budeš mít k dispozici tak dlouho, dokud máš zapnutou službu `httpd`.
Jakmile ji vypneš, služba se ukončí.
Zkus si to:

```console
# systemctl stop httpd
```

A samozřejmě musíš mít zapnutý i (virtuální) počítač, na kterém služba běží.
Když ho vypneš a znovu zapneš, webserver se zatím nezapne sám.
Na to potřebuješ ještě jeden příkaz,
který nastaví že se služba zapne vždy po startu systému:

```console
# systemctl enable httpd
```

Otestovat jestli máš službu korektně nastavenou můžeš restartováním počítače
(příkazem `reboot`).
Pokud potom běží tak jak jsi ji naplánoval{{a}}, je vše v pořádku.

> [note]
> Existuje příkaz, který „stav po zapnutí počítače“ nastolí bez restartu.
> Bohužel ale vypne nesouvisející procesy, takže nejspíš ukončí terminál
> do kterého příkaz zadáváš. Je to příkaz `systemctl isolate graphical`,
> resp. `systemctl isolate multi-user` pro systém bez grafického prostředí.

Zapínání při startu systému lze vypnout pomocí `systemctl disable httpd`.

> [note]
> Příkaz `systemctl start` a `systemctl stop` se týkají jen *aktuálního stavu*:
> nastartují/vypnou službu tady a teď.
> Příkazy `syatemctl enable` a `syatemctl disable` službu nespouští:
> jen nastavují co se zapne při startu systému.

### Restart a reload

Se službami se pojí ještě dva příkazy, které se hodí znát. Jsou to:
- `systemctl restart httpd` 
- `systemctl reload httpd`

Když změníš konfiguraci, server se o tom nedozví automaticky za svého běhu. 
Musíš mu říct, že se má restartovat a při startu si načte aktualizovanou konfiguraci.
Rozdíl mezi `restart` a `reload` je, že v případě `reload` se pouze načtou změny v konfiguračních souborech, ale existující připojení k webserveru zůstanou aktivní.
V případě restartu se celá služba ukončí (a připojení se zavřou) a znovu spustí.

A kde najdeš nastavení webového serveru? 
Je v `/etc/httpd/conf.d/`, kde najdeš několik souborů. 
Můžeš provést změnu v `/etc/httpd/conf.d/welcome.conf`, například vše zakomentovat (s použitím znaku `#`). 
Tento soubor definuje obsah stránky, kterou dostaneš, když se zeptáš na IP adresu počítače s běžícím webovým serverem.
Po zakomentování by se mělo začít zobrazovat něco jiného.
Když se ale teď podíváš do prohlížeče, nebude žádná změna.
Server si zatím změny nevšiml.
Můžeš teď buď vypnout a zapnout server, nebo mu říct, aby si načetl nové nastavení. 
Půjdeme druhou variantou, použij `systemctl reload httpd` (příp. `restart`) pro zobrazení změn.

### Log

Když je se službou něco špatně, je dobré se podívat do *logu*
(*log* je anglicky *lodní deník* – místo kam se zapisují záznamy o stavu).
Poslední část logu – několik posledních hlášek, které server ohlásil –
je vidět ve výstupu `systemctl status httpd`.
Více jich ale uloženo do systému, který se jmenuje `journal`.
Prohlíží se příkazem `journalctl`; přepínač `-u` filtruje hlášky z dané služby:

```console
$ journalctl -u httpd
-- Journal begins at Mon 2021-10-04 16:34:05 CEST, ends at Mon 2021-11-29 16:34:26 CET. --
lis 29 15:40:22 fedora systemd[1]: Starting The Apache HTTP Server...
lis 29 15:40:22 fedora httpd[3955]: AH00558: httpd: Could not reliably determine the server's fully qualified domai>
lis 29 15:40:22 fedora httpd[3955]: Server configured, listening on: port 80
lis 29 15:40:22 fedora systemd[1]: Started The Apache HTTP Server.
lis 29 15:59:24 fedora systemd[1]: Stopping The Apache HTTP Server...
lis 29 15:59:25 fedora systemd[1]: httpd.service: Deactivated successfully.
lis 29 15:59:25 fedora systemd[1]: Stopped The Apache HTTP Server.
lis 29 15:59:25 fedora systemd[1]: httpd.service: Consumed 1.087s CPU time.
lis 29 16:13:58 fedora systemd[1]: Starting The Apache HTTP Server...
lis 29 16:13:58 fedora httpd[8265]: AH00558: httpd: Could not reliably determine the server's fully qualified domai>
lis 29 16:13:58 fedora httpd[8265]: Server configured, listening on: port 80
lis 29 16:13:58 fedora systemd[1]: Started The Apache HTTP Server.
```


### A jiné služby

Příkazy `systemctl` fungují i pro ostatní služby na systému.
Až budeš místo webového serveru nastavovat server e-mailový,
spouštění a podobná správa služby bude fungovat obdobně.
Ale nastavení bude jinde – a v jiném formátu – než `/etc/httpd`.


## DocumentRoot a UserDir

Zpět ke konkrétní službě – webovému serveru.

To, co a jak dává `httpd` (Apache) k dispozici, se dá nastavit.

V `/etc/httpd/conf/httpd.conf` je základní nastavení.
To hlavní v něm je `DocumentRoot`: adresář kde jsou uloženy dokumenty které server servíruje.
Typicky je to `/var/www/html/`.
Vytvoř si v něm soubor `hello.txt` a napiš do něj *Hello, World!*
Podívej se na stránku a zkontroluj, že se ten soubor objeví v prohlížeči.
Na rozdíl od konfugurace (jako u `welcome.conf` výše) si obsah adresáře `/var/www/html` server „nepamatuje“ mezi jednotlivými připojeními.
Když tedy přijde nový dotaz, zobrazí se aktuální obsah, i kdyby se od posledního `reload` změnil.
Zkus to: změň `hello.txt` a pošli nový dotaz (obnovením stránky – klávesou <kbd>Ctrl</kbd>+<kbd>R</kbd>; v grafickém prohlížeči i <kbd>F5</kbd>).

V souboru `/etc/httpd/conf.d/autoindex.conf` je nastavení toho, jak se generují výpisy adresářů.
Zkus zakomentovat `IndexOptions` a kouknout jak se změní „hlavní“ stránka serveru po `systemctl reload`.

V `/etc/httpd/conf/httpd.conf` je ještě jedno nastavení, a to `UserDir`. 
Nastavuje, že pokud má uživatel vytvořený adresář `public_html`, pak tento adresář lze zobrazit pod `/~user/`, kde *user* je jméno uživatelského účtu.
Na prvních unixových systémech to bylo skvělé – každý si mohl jednoduše dát na web co chtěl.
Pak se zjistilo, že je to bezpečnostní riziko, protože se podle toho dá velmi jednoduše zjistit, jestli uživatel daného jména existuje na tom počítači a zkoušet se na ně připojit.
Z tohoto důvodu je to standardně vypnuté, ale v případě potřeby můžeš tuto funkcionalitu opět zapnout.

