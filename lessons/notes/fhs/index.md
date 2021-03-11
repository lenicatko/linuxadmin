# Adresářová struktura

Nyní se podíváme na strukturu souborového systému.

V grafickém programu zobrazujícím soubory (např. `nautilus`) otevři adresář `/`. 
Je dobré mít v nastavení (*Předvolby*, *Preferences*) zaškrtnutou možnost
*Umožnit rozbalování složek* (*Allow files to expand*),
která u adresářů ukáže malé rozbalovací trojúhelníčky.

V Linuxu jsou všechny soubory uložené v jednom adresářovém stromu,
pod adresářem `/`, což je hlavní rozdíl oproti Windows,
kde jsou disky `C:`, `D:`, atd.

V *Nautilu* se ke kořenovému adresáři nedá dostat úplně přímočaře.
Použij klávesovou zkratku <kbd>Ctrl</kbd>+<kbd>L</kbd>, která ti umožní zadat
jakoukoli cestu, zadej `/` a <kbd>Enter</kbd>.
Nautilus pak `/` zobrazí jako *Počítač*; na to můžeš kliknout a přidat
do záložek, ať se sem příště dostaneš rychleji.

V adresáři `/` najdeš všechny soubory, které existují na disku. 
Pro strukturu tohoto adresáře existuje standard,
pojmenovaný [Filesystem Hiearchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard),
který popisuje, kam ukládat jaké soubory. 
Základní struktura systémových souborů tak bývá na všech unixových systémech stejná.
Je univerzální, navržená nejen pro osobní počítače,
ale různé jiné stroje (servery, sdílené pracovní stanice na univerzitách, apod.).
Je potřeba ty soubory rozdělovat podle toho jak se k nim v těchto případech přistupuje:
* které jsou pro čtení, a které i pro zápis,
* které jsou přístupné pro všechny uživatele, a které pro každého uživatele zvlášť,
* které můžou být sdílené mezi více stejnými počítači,
* které můžou být sdílené mezi více různými počítači,
* které jsou součástí systému, a které doinstalované/vytvořené „ručně“.

Půjdeme na to podle abecedy.

## /bin

Jeden z adresářů, který patří do `PATH`.
Obsahuje naprostou většinu příkazů, které můžeš spustit, 
a které jsou nainstalované jako součást systému.
Najdeš tu `cat`, `echo`, `bash` i `ls`.

Na Fedoře je `/bin` *symlink* na `/usr/bin`.
Historicky totiž byly příkazy v `/bin` nainstalované na každém počítači zvlášť,
zatímco `/usr` sis mohl{{a}} zpřístupnit i přes síť z jiného počítače.
Příkazy, které administrátor potřeboval na tohle zpřistupnění, byly v `/bin`;
zbytek (např. okýnkové grafické prostředí) v `/usr/bin`

V dnešní době relativně velkých disků toto rozdělení nedává smysl,
a tak je `/bin` a `/usr/bin` to samé.


## /boot

Zde jsou soubory, které konfigurují to, co se stane po zapnutí počítače:
odkud se načte linuxové jádro, jak se připraví zbytek disku
a jak se jádro spustí.

Když máš nainstalovaných víc operačních systémů,
tady je např. konfigurace menu pro jejich výběr.

Na Fedoře tu (zvlášť po aktualizaci systému) najdeš hned několik konfigurací.
Kdyby jedna z nějakého důvodu nefungovala,
nemusíš celý systém přeinstalovat – můžeš vybrat jinou.


## /dev

Název `/dev` je od *device* - zařízení. 
Tady jsou soubory, které ovládají či řídí zařízení připojená k tvému počítači.

Vzpomínáš si, jak jsme říkali,
že soubor je cokoli, z čeho se dá číst nebo kam se dá zapisovat?
`/dev` je plný velice zvláštních souborů, které tohle dovádějí k absurditě.

Třeba v `/dev/input` jsou zařízení pro klávesnici a myš.

```console
$ ls /dev/input
by-id    event1  event4  js0     mouse1
by-path  event2  event5  mice    mouse2
event0   event3  event6  mouse0  mouse3
```

Zkus si vypsat (pomocí `sudo cat`) soubor `/dev/input/mouse3` a hýbat myší.
(Kdyby to nic nevypisovalo, zkus jiný soubor `/dev/input/mouse*`.)

Ze souboru pro myš se dá *číst* – dostaneš informace o tom, jak se myš hýbe
(i když formát ve kterém je dostaneš není přes `cat` příliš čitelný.)

Podobné soubory v `/dev` existují pro:
* procesory,
* zvukové karty,
* disky,
* paměť počítače,
* terminály (`/dev/tty*`),
* video karty,
* a tak dále.

Jedná o nastavení celého počítače, takže aby sem mohl{{a}} něco zapsat,
musíš být superuživatelem.
A když tam zapíšeš něco špatně, můžeš si tím zničit systém.


## /etc

Obsahuje konfigurační soubory.
Najdeš spoustu adresářů rozmanitých programů, kde je uloženo jejich nastavení.
Když jsi systémový administrátor a chceš nastavit uložené heslo k WiFi,
nebo že `~/bin` má být v `$PATH`,
nebo aby všichni uživatelé měli na ploše červené pozadí,
nastavíš to tady - pro celý systém.

Většinou se konfigurace ukládá jako textový soubor.
Může ji měnit jen superuživatel, ale stačí k tomu obyčejný textový editor.

Informace o všech uživatelích systému jsou, jak už víš, v `/etc/passwd`
a `/etc/shadow`.

`/etc/hosts` obsahuje seznam jmen počítačů na síti.
Mělo by to říkat, že 127.0.0.1 je `localhost`, což znamená,
že pokud se přípojíš „přes internet“ (např. v prohlížeči) k adrese `localhost`,
připojení bude navázáno s adresou 127.0.0.1 (která označuje tvůj počítač).
Ostatní webové adresy se hledají na internetu – jak, to je zadáno v souboru
`/etc/resolv.conf`.


Soubor `/etc/group` obsahuje seznam všech uživatelských skupin na počítači.
Místo `usermod` tak můžeš ručně přidat řádek sem – pokud si věříš, že zadáš
vše správně.


Soubor `/etc/fstab` obsahuje nastavení disků – respektive které disky z `/dev`
jsou na tomto počítači vidět jako souborové systémy.
Tady se dá nastavit třeba,
z jakého vzdáleného počítače se má vzít adresář `/home`.

Samotný `/home` je další v abecedě, ale podívejme se nejdřív na `lib*`.


## /lib a /lib64

Obě ty složky jsou odkazy na `/usr/lib` a `/usr/lib64`.
Je to ze stejného důvodu jako u `/bin` a `/usr/bin`:
původně v adresářích `/lib` a `/lib64` byly obsaženy systémové knihovny
nutné pro administraci; dnes jsou sjednoceny se systémovými knihovnami
pro běžnou práci.


### Věc o architekturách - x86_64, ARM atd.

Ty nejrychlejší programy na systému jsou zkompilované do *binárních souborů*.
Když je otevřeš pomoci například `less`, uvidíš nečitelný binární výstup.
Zkus:

```console
less /bin/cat
```

Soubor obsahuje instrukce pro *procesor* – řídící jednotku počítače.
Máš-li procesor od Intelu, instrukce „přičti 1“ má číslo řekněme 8634369.
Máš-li procesor ARM, stejná instrukce bude mít třeba číslo 3800109057.

Pro různé druhy procesorů tak je potřeba mít různé programy.
Programy v Pythonu tento problém nemají: instrukce jsou zapsané jako text,
který jiný program čte a vykonává.
(A ten jiný program už je zkompilovaný – viz `less /usr/bin/python`.)

Programy v Pythonu umí importovat knihovny.
Podobně fungují *knihovny* (angl. *libraries*) i pro zkompilované programy:
užitečné funkce jsou uloženy ve zvláštním souboru, který se dá připojit
(angl. *link*) k různým programům.
Třeba knihovna `libc.so` poskytuje základní funkce jako otevírání a čtení
souborů a knihovna `libm.so` matematické funkce jako sinus nebo odmocninu.
Další knihovny jsou k dispozici jako `lib*.so`, občas s číslem verze na konci
a občas v podadresáři `/lib64`.

```console
$ ll /lib64/libc.so
-rw-r--r--. 1 root root 253 16. čec 04.26 /lib64/libc.so
$ ll /lib64/libm.so
-rw-r--r--. 1 root root 110 16. čec 04.26 /lib64/libm.so
```

Procesory Intel (a AMD) mají zvláštnost, že rozumí dvěma „jazykům“,
tedy kódování instrukcí: starší (32-bitový) `x86`
a novější (64-bitový) `x86_64`.
Aby fungovaly programy pro obě varianty, musí pro ně být na systému knihovny.
Ty pro starší variantu jsou v `/lib` (protože tam je starší programy hledají;
novější v `/lib64`.
Na tvém systému je adresář `/lib` téměř prázdný, protože dnes se programy
kompilují jen pro tu novější verzi.


## /home

Místo pro uživatelské domovské adresáře.
Uvnitř je adresář <code>home/<var>tvojejmeno</var></code>, zkráceně `~`,
do nějž si můžeš jako uživatel dát, co jen chceš :)

Některé programy hledají konkrétně pojmenované soubory v domovské složce,
třeba Bash tu má `.bashrc`, `.bash_profile` a `.bash_history`.
Když si zobrazíš skryté soubory, zjistíš, že souborů začínajících tečkou je v ní poměrně dost.
Protože je pak orientace ve složce složitější,
moderní programy se snaží ukládat svou konfiguraci do adresáře `.config`
místo přímo do `~`.
V něm si své potřebné informace ukládají do složek podle svého jména.

Soubor `/home/petr/.config/user-dirs.dirs` například obsahuje jména adresářů,
do kterých grafické programy ukládají data:

```console
$ cat /home/petr/.config/user-dirs.dirs
...
XDG_DOCUMENTS_DIR="$HOME/Dokumenty"
XDG_MUSIC_DIR="$HOME/Hudba"
XDG_PICTURES_DIR="$HOME/Obrázky"
XDG_VIDEOS_DIR="$HOME/Videa"
```

Kromě toho by v `~` měl být adresář `.local`, kde můžou být složky jako:

* `bin`
* `etc`
* `lib`
* `share`
* `var`

Jména těch adresářů mají stejný význam, jako přímo v kořenovém adresáři `/`,
s tím rozdílem, že tady jsou dostupné jen pro toho konkrétního uživatele.
Když si nainstaluješ program jen pro sebe, spustitelný program se uloží do
`~/.local/bin`, knihovny do `~/.local/lib` a nastavení do `~/.local/etc`.
jako by se instaloval do systému (čili do stejných složek), ale pouze pro tebe.

Pythonní programy můžeš pro uživatele instalovat pomocí `pip install --user`:

```console
$ python -m pip install --user cowsay
...
Successfully installed cowsay-2.0.3
$ type cowsay
cowsay je /home/petr/.local/bin/cowsay
[petr@localhost link]$ cowsay Ahoj!
  _____
< Ahoj! >
  =====
...
```

(Většinou je ale lepší použít virtuální prostředí. Když budeš instalovat
pomocí `--user` do `~/.local`, po lěkolika měsících či letech tam budeš mít
pořádný nepořádek – spoustu nekompatibilních verzí knihoven a příkazů.)


## /libexec
Možná máš tuto složku jen jako součást `/usr`.
Jsou zde programy, které by se neměly spouštět přímo z příkazové řádky.
Najdeš zde skripty, které využívají přímo programy nainstalované,
ale nejsou vhodné na to, aby je spouštěl člověk.
Takové skryté prográmky dávají vývojaři programů do `/libexec`.

Třeba `/usr/libexec/git-core/git-diff` je program, který spustí `git` když
zadáš `git diff`.
Nikdo a nic jiného by ho spouštět neměl.


## /lost+found
Když spadne počítač a zhavaruje disk, systém se pak z toho vzpamatuje a může
najít nějaké soubory, které nejsou zařazené do žádného adresáře.
Nalezené soubory jsou přesunuty sem.
V ideálním případě je tento adresář prázdný a neměl by být přístupný pro každého,
protože některé z těch souborů můžou obsahovat citlivé údaje.

## /media
Když vložíš flashku či CD, v `/media` tady se automaticky
vytvářely adresáře s obsahem toho zařízení.
Dneska zas to tak moc nedělá a je pro to lepší místo.
Občas tu z historických najdeš symbolický odkaz na to lepší místo.

## /mnt
Sem se připojovují externí disky.

Na osobních počítačích se s tím už tolik nepotkáš – když vložíš do počítače
*flashku*, systém ji automaticky najde a zpřístupní (na jiném místě).
Ale na serverech, kde se disky připojují „ručně“, jsou často v `/mnt`.
Stejně tak třeba na univerzitě většinou najdeš sdílené disky právě tady.

(Víse o pcipojování příště.)

## /opt

Tady se dávají věci, které nejsou dělané přímo pro tvou distribuci.
Programy, které jsou napsány přímo pro ten systém, dávají jakousi záruku, že všechno bude dobře spolupracovat.
Tyto ukládají své soubory do hlavního souborového systému.
Pro ostatní případy se to dává do `/opt`, do složek se jménem autora, firmy nebo softwaru.

Pokud si nainstaluješ Chrome od Googlu, VS Code od Microsoftu nebo Zoom od… Zoomu,
měly by se objevit zde.

## /proc

Obsahuje informace o aktuálně běžících procesech. 
Každý proces tady má svou složku a v ní informace například o paměti, kterou používá.
Existují pak třeba programy, které se podívají dovnitř a jsou podle obsahu paměti schopné říct,
jaké funkce daný proces zrovna volání a kolik času stráví v této funkci.
Nebo třeba otevřené soubory (jako to dělá `lsof`).

* `mem` – paměť využívaná procesem
* `fd` – *file descriptors*, otevřené soubory
* `cwd` – odkaz na aktuální pracovní adresář

`/proc/self` je symbolický odkaz na adresář s aktuálně běžícím procesem.
Můžeš díky němu zjistit číslo právě běžícího procesu:
```console
$ ls -l /proc/self
lrwxrwxrwx 1 root root 0 říj 17 09:48 /proc/self -> 6692
```
6692 je zde číslo procesu `ls`.

## /root

Domovský adresář pro superuživatele.

Proč `root` nemá svou složku v `/home`? 
Jde o to, že běžné uživatelské účty bývají občas sdílené.
Představ si počítačovou učebnu na univerzitě:
Připojíš se k libovolnému počítači a dostaneš k dispozici své soubory.

Nebo se může stát, že se něco pokazí a adresáře v `/home` nejsou k dispozici.
Zavolá se administrátor, přihlásí se jako *root* a vyřeší problém.
Domovská složka *roota* musí být proto k dispozici vždy, i když je nějaký
problém s `/home`.


## /run
Obsahuje informace potřebné za běhu.
Když daný program skončí, informace v `/run` se dají smazat.
Je to novější standard, ne všechny programy ho používají – často podobné
informace najdeš v `/lib`.

V <code>/run/media/<var>jméno</var></code> se objeví obsah např. připojené
flashky.
Disk je tak k dispozici tomu, kdo byl přihlášený když se flashka objevila.

Ve virtuálním počítači je možné virtuálně vložit CD (jeho obsah v souboru .ISO);
když to uděláš, obsh se objeví v <code>/run/media/*</code>.

## /sbin

Tady jsou tu programy pro superuživatele, které by neměl spouštět běžný
uživatel – nástroje na opravy disků, analýzu chyb, nastavení síťování
a spousta jiných.

Mezi `/bin` (`/usr/bin`) a `/sbin` (`/usr/sbin`) není v praxi moc velký rozdíl.
V `PATH` máš normálně oba.


## /srv
Pokud na tvém počítači má běžet webový server, sem se tradičně dávají věci
které takový server zpřístupní.
Nejspíš tam teď nic nemáš.

## /sys
Informace o systému.
Když je potřeba dát k dispozici nějaké informace o systému, nebo zařídit,
aby se dal ovládat, je zde na to soubor.
Dájí se tu nastavit např. detaily správy paměti nebo (na opravdovém počítači)
třeba rychlost větráčků.

Je podobný jako `/dev`: soubory v `/dev` se tvoří na základě
informací v `/sys`.

## /tmp

Sem se dávají dočasné soubory.
Na moderních systémech se tato složka automaticky smaže při startu systému.

Historicky se sem ukládaly např. stažené soubory z prohlížeče nebo
rozbalené archivy, které pak člověk mohl zkopírovat někam jinam.

Pokud v automatických testech testuješ čtení a ukládání nějakých souborů,
je dobrý nápad vytvářet složku s testovacími soubory v `/tmp`.
Kdyby se soubory nepodařilo smazat, smažou se aspoň po vypnutí systému.

Některé programy si do `/tmp` dávají soubory potřebné pro jednorázovou akci.
Například když v prohlížeči zmáčkneš u stahování přílohy *otevři* místo
*stáhni*, tak se příloha stáhne do `/tmp`.

Krom toho existuje i `/var/tmp`, kam se dávají dočasné soubory
které nechceš smazat po restartu počítače.


## /usr
Složky pro uživatele, kopírují strukturu kořenového adresáře.

## /var
Tady jsou soubory, které se mění za běhu systému.
`/var/mail` je systémová pošta. E-mail je standard starší než internet – zprávy
se dají posílat i mezi uživateli v rámci jednoho počítače.

`/var/cache/` - tady se dávají soubory, které zrychlují práci,
ale dají se vygenerovat kdykoli znova.
Když si nějaký program stáhne něco z internetu,
dá si to sem a příště to nemusí stahovat zase.
Pokud se to smaže, nic se neděje, stáhne se to prostě zas.

`/var/run/` - další adresář pro soubory, které se mění za běhu počítače.
`/var/run/` může být *symlink* na `/var` nebo `/run`, všechna tři místa mají totiž stejný účel.

