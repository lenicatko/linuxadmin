
# 5. Linuxová administrace - souborové systémy


## Jak pustit terminál s příkazem, aniž bychom sáhli na myš?

Nejprve zjisti název svého terminálu. 
Může to být například `gnome-terminal`, `konsole` nebo `xterm`. 
Na tvém systému může jich být víc než jeden, vyber si kterýkoli z dostupných.

Když se podíváš do nápovědy vybraného terminálu, dozvíš se další krok:

```console
$ gnome-terminal --help
Usage:
  gnome-terminal [OPTION…] [-- COMMAND …]
```

Zkus třeba příkaz:

```console
$ gnome-terminal -- top
```
Zavřením okna s terminálem ukončíš daný proces.

> [note] 
> Spouštět programy můžeš také z systémového Shellu, který je dostupný pod zkratkou <kbd>ALT</kbd>+<kbd>F2</kbd>. 
> Když do něj napíšeš jméno terminálu a zmáčkneš <kbd>Enter</kbd>, zafunguje to taky. 
Je to asi nejrychlejší cesta ke spouštění programů na počítači s Linuxem.


## Odkazy

### Tvrdý odkaz
Vytvoříme si nový soubor `basnicka.txt`

```
Když byl Pepa ještě mladý,
nevědel si s ničím rady.
Chlastal pivo, jedl ovar,
byl duševní polotovar.
```

Podívej se do obsahu stávající složky pomocí příkazu `ls -li`. 
Všimni si, že oproti příkazu `ls -l`, je zde nový první sloupec. 
Jak myslíš, co znamenají tyto hodnoty?

```console
$ ls -l -i
540792 drwxr-xr-x. 2 guest guest 4096 Mar 13  2016 basnicka.txt
557103 drwxrwxr-x. 4 guest guest 4096 Feb  1  2019 dev
557071 drwxr-xr-x. 2 guest guest 4096 Mar 13  2016 Documents
```
V prvním sloupci je identifikační číslo souboru.
Každý soubor v systému má své unikátní číslo. 
Adresář funguje podobně jako slovník, kde jsou klíče názvy souborů a hodnoty - jejich identifikace.
Když se podíváš na adresáře, zjistíš, že i adresáře mají svá identifikační čísla.

```console
$ ls -l -i ..
total 1884220
2885155 drwxr-xr-x 3 guest guest       4096 zář 19  2019 01
3408256 drwxr-xr-x 2 guest guest       4096 říj  5  2019 02
2885201 drwxr-xr-x 2 guest guest       4096 říj 24  2019 03
2884113 drwxr-xr-x 1 guest guest       4096 říj 24  2019 04
2884334 drwxr-xr-x 1 guest guest       4096 říj 19  2019 05
```

Co by se stalo, kdybys měl{{a}} v adresáři dvě jména, která by označovala stejný soubor?

Takto vytvoříš druhý název, který bude označovat stejný soubor.
```console
$ ln basnicka.txt basen.txt
$ ls -l -i
534799 -rw-rw-r--. 2 guest guest    0 Oct 24 18:23 basen.txt
534799 -rw-rw-r--. 2 guest guest    0 Oct 24 18:23 basnicka.txt
```
Oba soubory jsou zcela stejné. Když změníš jeden, změní se i druhý.
Tomu se říká **hardlink** (*tvrdý odkaz*). A k čemu je to užitečné? Popravdě, moc není. 
Tvrdé odkazy se používají často na to, aby se identifikovaly stejné soubory na disku a jeden mohl nahradit kopií toho druhého. 

Souborový systém dokonce kontroluje, kolikrát je daný soubor v tomto do systému, a to pomocí čísla, na které se můžeš podívat ve třetím sloupci výpisu.
```console
534799 -rw-rw-r--. 2 guest guest    0 Oct 24 18:23 basen.txt
534799 -rw-rw-r--. 2 guest guest    0 Oct 24 18:23 basnicka.txt
                   ^
                   |
                   +-- toto počítadlo existence souboru na disku
                       (= kolikrát se na disku nachází)   
```
Když smažeš jeden ze souborů, číslo se sníží o 1. 
Když smažeš druhý, číslo spadne na nulu a systém bude vědět, že daný soubor není už potřeba, pak ho jednou smaže.

*Hardlink* není úplně šikovný, ale občas se s ním setkáš. Je dobře vědět, že existuje.


### Symbolický odkaz

Užitečnější a frekventovanější je **symlink** (symbolický odkaz).
Můžeš si představit, že tento druh odkazu říká: "obsah tohoto souboru zde není, podívej se jinam".

Symbolický odkaz vytvoříš pomocí přepínače `-s`:
```console
$ ln -s basnicka.txt BASEN
$ ls -li
534947 lrwxrwxrwx. 1 guest guest    9 Oct 24 18:26 BASEN -> basnicka.txt
534799 -rw-rw-r--. 2 guest guest    0 Oct 24 18:23 basen.txt
```

Podívej se na výstup v terminálu. `BASEN -> basnicka.txt` je zobrazení symbolického odkazu. 
Čísla obou souborů jsou jiná.
Soubor `BASEN` nemá v sobě obsah naší básničky. 
Představ si, že je v něm uložená jen informace, že obsah se nachází v souboru `basnicka.txt`. 
S odkazem `BASEN` můžeš pracovat jako s jinými soubory. 
Když ho otevřeš pro čtení, systém sáhne po místě, na které odkazuje, tedy `basnicka.txt`, a vypíše jeho obsah.

Dá se udělat i odkaz někam, kde nic není.
```console
$ ln -s nic.txt NIC
$ ls -li
2885258 lrwxrwxrwx 1 guest guest          7 říj 24  2019 NIC -> nic.txt
```

Když ho budeš chtít otevřit, systém ale řekne, že takový soubor neexistuje.

```console
$ cat NIC
cat: NIC: No such file or directory
```
Symbolické odkazy jsou velmi užitečné.

* Nemusíš nic kopírovat
Když potřebuješ mít soubor na určitém místě na disku, ale nechceš ho kopírovat - uděláš symbolický odkaz.
Typicky se symbolické odkazy použijí, pokud máš soubory na externím disku a chceš s nimi pracovat na svém počítači. 
Vytvoří se *symlinky* na určitá místa externího disku, počítač je zpracuje a po odpojení disku *symlinky* zůstanou k dispozici do budoucna.

* *Symlink* může odkazovat i na adresář
Můžeš si například nastavit *symlink* na svůj domovský adresář:

```console
ln -s ~ tady-je-doma
ls -li
2884841 lrwxrwxrwx 1 guest guest         10 zář 28 12:46 tady-je-doma -> /home/guest
```

Na to je dobré dávat pozor, až budeš psát skript, který něco čte a ukládá v adresářové struktuře - vždy si ověř, že odkazuješ na správné místo na disku. 

> [note] 
> Když se podíváš na soubory v grafickém programu, většinou poznáš podle ikonky s šipečkou, že se jedná o symbolický odkaz.


# Uspořádání souborů na disku

Nyní se podíváme na strukturu souborového systému.

V grafickém programu zobrazujícím soubory (např. `nautilus`) otevři adresář `/`. 
Je dobré mít v tuto chvíli zaškrtnutou možnost *Allow files to expand*, která rozbalí obsah adresáře po kliknutí.

V Linuxu jsou všechny soubory uložené v jednom adresářovém stromu, pod adresářem `/`, což je hlavní rozdíl oproti Windows, kde jsou disky `C:`, `D:`, atd.

V adresáři `/` najdeš všechny soubory, které existují na disku. 
Pro strukturu tohoto adresáře existuje standard, pojmenovaný [Filesystem Hiearchy Standard](https://en.wikipedia.org/wiki/Filesystem_Hierarchy_Standard), který popisuje, kam ukládat jaké soubory. 
Základní struktura systémových souborů bývá na všech unixových systémech stejná.
To jak vypadá, vychází z historických důvodů. 
Je totiž univerzální, navržená nejen pro osobní počítače, ale různé jiné stroje (sdílené počítače, pracovní stanice apod.).
Je potřeba ty soubory rozdělovat podle toho, jak se k nim v těchto případech přistupuje: které jsou pro čtení, a které pro zápis, které jsou přístupné pro všechny uživatele, a které pro každého uživatele zvlášť. 
Jsou rozděleny soubory, které se instalují do systému jakožto jeho součást (např. software, který se instaluje přes nějaký systémový instalátor), a programy, které si uživatel nainstaluje ručně.

Půjdeme na to podle abecedy.

## /bin
Je to jeden z adresářů, který patří do `PATH`.
Obsahuje všechny příkazy, které můžeš spustit.

Možná to budeš mít jako *symlink* na `/usr/bin`. Je to tak opět z historických důvodů. 
Historicky totiž `/bin` byl na každém počítači zvlášť, zatímco `/usr` sis mohl{{a}} zpřístupnit z jiného počítače nebo disku. 
Z toho vyplývá rozdíl mezi příkazy, které potřebuje samotný systém, potažmo systémový administrátor, a příkazy, které jsou záležitostí samotného uživatele. 
`bash`, `cat`, `ls` - všechno to patří k `/bin`. 
Do `/usr/bin` patří třeba grafické editory nebo okénkové prostředí (to, co není nezbytně potřeba, aby tvůj systém dokázal fungovat).
V `/bin` jsou věci, které potřebuje systémový administrátor, aby mohl počítač opravit.

Na spoustě moderních systémů (ač ne všech) jsou již tyto adresáře propojeny.

## /boot

Obsahuje soubory na inicializaci systému. 
Zde jsou soubory, které konfigurují to, co se stane po zapnutí počítače. 
Načte se linuxové jádro, připraví se zbytek disku, aby se dal používat a jádro se pustí. 
Když máš nainstalovaných víc operačních systému, tady je např. konfigurace menu pro jejich výběr.

## /cd-rom
V dřívější době se zde objevil obsah vloženého CD. 
Dneska se to ale už dává jinam. 
Ne na každém systému najdeš tento adresář.

## /dev

`dev` je od *device* - zařízení. 
Tady jsou soubory, které ovládají, či řídí zařízení přípojená k tvému počítači.

Je zde například `/dev/input/mouse0`, který dostává informace o tom, jak se hýbe tvoje myš. A takové soubory jsou k dispozici v `/dev`. 
Soubory, které komunikují dost přímo s věcmi připojenými k počítači. Mezi nimi jsou:
* procesory
* sdílené paměti,
* disky,
* zvukové karty,
* spousta tty s číslem (tty0, tty1 atd.) - čili terminály,
* video karty,

Je potřeba být superuživatelem, aby bylo možné něco zapsat do těch souborů, protože se jedná o nastavení celého počítače. 
A když tam zapíšeš něco špatného,  můžeš si tím zničit systém.

## /etc

Obsahuje konfigurační soubory. 
Když jsi systémový administrátor a chceš, aby všichni uživatelé měli například červené pozadí, nastavíš to tady - pro celý systém.
Většinou zde najdeš spoustu adresářů rozmanitých programů, kde je uloženo jejich nastavení.

Je konvencí, že konfigurace se ukládá jako textový soubor.
Může je měnit jen superuživatel, ale pro změnu stačí obyčejný textový editor.

Informace o všech uživatelích systému jsou v `/etc/passwd`. 
Je jich zde asi víc, než jsi čekal{{a}}}. 
Typicky pro každou aplikaci existuje jeden uživatel, kterému se povolí dělat jen tu či onu činnost.
Jako první je uživatel `root`. Někde mezi údaji (nejspíš blíž konce) bude i tvoje uživatelské jméno.

```console
                   + -- domovský adresář
  jmého uživatele  |      shell uživatele
  |                |         |
root:x:0:0:root:/root:/bin/bash
       | |   |
       | |   + -- jméno hlavní skupiny
       +-+ -- číslo uživatele a skupiny, do které patří       

bin:x:1:1:bin:/bin:/sbin/nologin
daemon:x:2:2:daemon:/sbin:/sbin/nologin
```

Proč `root` nemá svou složku v `/home`? 
Jde o to, že běžné uživatelské účty bývají sdílené (jsou k dispozici z více počítačů - třeba přes síť). 
Představ si třeba počítačovou učebnu. 
Připojíš se k libovolnému počítači svými údaji a dostáváš k dispozici své soubory. 
Může se stát, že se něco pokazí a domovská složka není k dispozici. 
Zavolá se administrátor, přihlásí se jako *root* a vyřeší problém. 
Domovská složka *rootu* musí být proto vždy k dispozici, ne jen tehdy, když je k dispozici `/home`.

Když se přihlásíš do systému, tak se pustí tzv. *login shell*. 
U tebe to bude Bash, ale můžeš si zde nastavit i jiný Shell.
Když je jako *login shell* uvedeno u uživatele `nologin`, takový uživatel se nemůže přihlásit do systému.

A kde jsou hesla v tak slibně pojmenovaném souboru? 
Ano, jsou to ty `x` ve druhém sloupečku údajů uživatele.
Když Unix vznikal v akademickém prostředí, vůbec nevadilo, že se heslo ukládalo přímo do toho souboru. 
Jenže jak se systém vyvíjel, nebylo to udržitelné řešení. 
Hesla se proto přenesla jinam. 
Z důvodu provázanosti v systému se ale soubor nemohl přejmenovat a je to dodnes `/etc/passwd`, i když hesla neobsahuje.

Reálná hesla jsou uložená v souboru `shadow`. 
Ten soubor nemůže nikdo přečíst (když se podíváš na oprávnění - opravdu nikdo), jsou zahashovaná.

Přidání/smazání uživatele se udělá právě v `/passwd`. 
Když se ve správném formátu vyplní nový řádek, vytvoří se tím systémový uživatel. Může to udělat jen superuživatel. 
Na správu uživatelů existují ale lepší nástroje, takže to nedělej tímto způsobem :). 
Protože je tento soubor tak důležitý, vytváří se k němu i záloha s pomlčkou v názvu. 
Jen pro případ, že by se něco rozbilo v originálu.
Když se za jméno souboru dá vlnovka `~`, tak je to záloha. 
Ne na všech souborových systémech se ale vlnovka dá použít, a proto základní soubory používají pro označení zálohy pomlčku.

### hosts
Obsahuje seznam jmen počítačů na síti.
Mělo by to říkat, že 127.0.0.1 je `localhost`, což znamená, že pokud se přípojíš k adrese `localhost`, připojení bude navázáno s adresou 127.0.0.1.


### group
Obsahuje seznam všech uživatelských skupin na počítači.
Přidáním řádku vytváříš skupiny.
```console
jméno  číslo skupiny
|      |
root:x:0  
wheel:x:10:user
            |
            členové této skupiny
```
Nejrychlejší způsob, jak se přidat do skupiny, je přidání svého jména na konci řádku. Lepší je ale použít příkaz `usermod`.


### fstab
Zde je nastavení, které disky na tomto počítači chceš vidět jako připojené souborové systémy.
Dá se zde nastavit třeba, z jakého vzdáleného počítače se má vzít složka `/home`.

## /home
Místo pro uživatelské domovské složky.
Je to složka, do níž si můžeš dát, co jen chceš :). 
Některé programy hledají konkrétně pojmenované soubory v domovské složce, třeba `.bashrc`. 
Když si zobrazíš skryté soubory, zjistíš, že souborů začínajících tečkou je v ní poměrně dost.
Protože je pak orientace ve složce složitější, moderní programy se spíš snaží ukládat svou konfiguraci do adresáře `.config`. 
V něm si své potřebné informace ukládají do složek podle svého jména.

Kromě toho by tam měl být adresář `.local`, kde jsou složky
* `bin`
* `etc`
* `lib`
* `share`
* `var`

Jména těch adresářů mají stejný význam, jako přímo v kořenovém adresáři `/`, s tím rozdílem, že tady jsou dostupné jen pro toho konkrétního uživatele.
Když si nainstaluješ program jen pro sebe, nainstaluje se úplně stejně, jako by se instaloval do systému (čili do stejných složek), ale pouze pro tebe.
Příklad může být třeba příkaz: `pip --user install nejakypythonnnastroj`, který se dá do složky `.local/bin`.


## /lib a /lib64

Obě ty složky jsou odkazy na `/usr/lib` a `/usr/lib64`, a je to ze stejného důvodu, jako `/bin` a `/usr/bin`. 
Původně v adresářích `/lib` a `/lib64` byly obsaženy systémové knihovny pro administrátora, dnes jsou již sjednoceny s adresářem pro knihovny doinstalované uživatelem.

### Věc o architekturách - x86_64, ARM atd. 
Můžeš mít procesor Intel, který je 64bitový, což znamená, že instrukce pro něj mají nějakou velikost. 
Existuje sada instrukcí pro tu konkrétní architekturu. 
Vedle toho existují procesory ARM, které vevnitř fungují jinak, a taky potřebují jiné programy.
Program napsaný v Pythonu bude fungovat na obou, protože pythonní interpreter umí ho přečíst na jakémkoli systému.
Ty nejrychlejší programy na tvém systému jsou ale zkompilované do binárních souborů. 
Ty soubory poznáš velice jednoduše, když je otevřeš pomoci například `less`, uvidíš nečitelný binární výstup.
Zkus:
```console
less /bin/cat
```
To jsou už přímo instrukce pro ten procesor. Pro každý procesor bude ten *stejný* soubor vypadat jinak.
Když máme vícero počítačů na síti, rozlišuje se mezi soubory, které jsou jen pro tento konkrétní procesor, a soubory, jež jsou použitelné kdekoli.
V `/lib` jsou knihovny zkompilované jen pro daný procesor, a v `/usr/share` se pak nachází ty obecné, které se můžou použit na jakémkoli počítači.

V `/lib64` uvidíš spoustu souborů, které mají příponu `.so`.
Najdeš zde například `libc.so`, což je standardní knihovna jazyka C.
Tu knihovnu používá téměř každý program, který máš na počítači, umožňuje totiž komunikaci s operačním systémem.

Spousta těch souborů má na konci `.1` či jiné číslo. 
Jedno z toho je typicky symbolický odkaz na ten druhý soubor.
Když nějaký program používá konkrétní verzi knihovny, řekne si: *já chci libc.so6*, a dostane přímo tuto verzi. 
Když je programu nezáleží na verzi knihovny, vyžádá si základní název a dostane zpravidla nejnovější verzi, která je nainstalovaná na daném počítači.
Jsou to tedy různá jména stejného souboru, podle toho, jestli chci nejnovější nebo konkrétní verzi. 
A díky tomu, že je to odkaz, neduplikují se na disku žádné informace.


Možná si teď říkáš, k čemu vlastně ty odkazy jsou, proč je jeden soubor v tolika verzích?

Podívejme se na Python.
```console
$ ls -l /usr/bin/python
/usr/bin/python --> python2
$ ls -l /usr/bin/python2
/usr/bin/python2 --> python2.7
$ ls -l /usr/bin/python3
/usr/bin/python3 --> python3.7
```
Když napíšeš příkaz `python`, v této distribuci Linuxu dostaneš k dispozici verzi 2.7. Když napíšeš `python3` - verzi 3.7. 
Pokud píšeš program v Pythonu a víš, že se může provést kteroukoli verzi Pythonu, nemusíš specifikovat verzi, stačí použit obecný odkaz. 
Pokud ale víš, že chceš použít specifické vlastností konkrétní verze, například Python 3.8, zadáš do terminálu přímo: `python3.8`.
Odkaz je způsob, jak říct, co je ta výchozí verze, v tuto chvíli nejlepší k použití.

Proč je `/lib` a `/lib64`? To je vlastnost nejpoužívanějších procesorů Intel, že umí dvě instrukční sady. 
Rozumí jak 32bitovým, tak 64bitovým instrukcím. 
Daný program ale může používat jen jednu z knihoven, musí si vybrat.

## /libexec
Možná máš tuto složku jen jako součást `/usr`.
Jsou zde programy, které by se neměly spouštět přímo z příkazové řádky.
Najdeš zde skripty, které využívají přímo programy nainstalované v tvém počítači, ale nejsou vhodné na to, aby je spouštěl člověk. Takové skryté prográmky dávají vývojaři programů do `/libexec`.

## /lost+found
Když spadne počítač a zhavaruje disk, systém se pak z toho vzpamatuje a může najít nějaké soubory, které nejsou zařazené do žádného adresáře. Nalezené soubory jsou přesunuty sem. 
V ideálním případě je tento adresář prázdný a neměl by být přístupný pro každého, protože některé z těch souborů můžou obsahovat citlivé údaje.

## /media
Když vložíš flashku či CD, v `/media` tady se automaticky vytvářely adresáře s obsahem toho zařízení. 
Dneska zas to tak moc nedělá a je pro to lepší místo. Sem se dávají symbolické odkazy na jiné adresáře, ale je to tak z historických důvodů. 

## /mnt
Tady se připojovaly disky, v dávných časech, když se to dělávalo ještě ručně. Dnes, když vložíš do počítače flashku, systém ji automaticky najde a zpřístupní. 
Dneska se s tím už tolik nepotkáte, ale třeba klasicky na univerzitách, kde všechny počítače jsou připojené do jedné sítě, sdílené disky jsou k dispozici právě z `/mnt`.

## /opt
Tady se dávají věci, které nejsou dělané přímo pro tvou distribuci. 
Programy, které jsou napsány přímo pro ten systém, dávají jakousi záruku, že všechno bude dobře spolupracovat. 
Tyto ukládají své soubory do hlavního souborového systému.
Pro ostatní případy se to dává do `/opt`, do složek se jménem autora, firmy nebo softwaru. 
Je zde rozdíl mezi programy vytvořenými přímo pro spolupráci s tímto systémem a programy, které se sice zabalily tak, aby fungovaly na tom systému, ale nejsou jeho součástí. 
Pokud si nainstaluješ *Google Chrome*, který není open source a linuxové distribuce ho nechtějí distribuovat, najdeš ho zde.

## /proc
Obsahuje informace o aktuálně běžících procesech. 
Každý proces má tady svou složku, a v ní informace například o paměti, jakou ten proces používá. 
Existují programy, které se podívají dovnitř procesů a jsou schopné říct, volání jaké funkce daný proces zrovna dělá a kolik času stráví program v této funkci. 
`mem` = paměť využívaná procesem
`fd` - *file descriptors*, tady jsou všechny soubory otevřené daným procesem
`self` je symbolický odkaz na adresář s aktuálně běžícím procesem.
Můžeš díky němu zjistit číslo procesu právě běžícího programu:
```console
ls -l /proc/self
lrwxrwxrwx 1 root root 0 říj 17 09:48 /proc/self -> 6692
```
6692 je zde číslo procesu `ls`.

## /root
Domovský adresář pro superuživatele.

## /run
Obsahuje informace potřebné pro běh programu.
Je to novější standard, tady se dávají soubory, které se používají za běhu (např. informace o dockerových kontejnerech)

`/run/media/user` - zde se objeví obsah např. připojené flashky (toto je nejaktuálnější místo, kam se tyto věci dávají). 
`/run` je na to sémanticky lepší místo, protože obsah flashky mění za běhu tohoto počítače. Pak je tam adresář aktuálního uživatele, což je skvělé z hlediska přístupových práv, flashka se nezpřístupní všem, a jen tomu, kdo ji právě vložil.

## /sbin
Tato složa se dnes tak moc nepoužívá a měla by být odkaz na `/usr/sbin`. 
Tady jsou programy pro superuživatele, které by neměl spouštět běžný uživatel - nástroje na opravy disků, analýzu chyb, nastavení síťování a spousta jiných.

## /srv
Pokud na tvém počítači běží webový server, tady se tradičně dávají věci přístupné z toho webového serveru. Nejspíš tam teď nic nemáš.

## /sys
Informace o systému. Když je potřeba dát k dispozici nějaké informace o tom systému, nebo zařídit, aby se ten systém dal ovládat, je zde na to soubor. Dá se tím i nastavit např. rychlost větráčků atd.

## /tmp
Velice zajímavá složka - tady se dávají dočasné soubory. 
Na moderních systémech se tato složka automaticky smaže při startu systému. 
Historicky se tady ukládaly např. stažené soubory z prohlížeče, rozbalené archivy, a pak to člověk mohl zkopírovat někam jinam.
Pokud v automatických testech testuješ čtení a ukládání nějakých souborů, je dobrý nápad v nich vytvářet složku s testovacími soubory do `/tmp`, kde po doběhnutí testů se systém sám postará o úklid.
Některé programy si do `/tmp` dávají soubory potřebné pro jednorázovou akci. Nebo když v prohlížeči zmáčkneš u stahování přílohy *otevři* místo *stáhni*, tak se příloha stáhne do `/tmp`.

Krom toho existuje i `/var/tmp`, kam se dávají dočasné soubory, které nechceš smazat při restartu počítače.

## /usr
Složky pro uživatele, kopírují strukturu kořenového adresáře.

## /var
Tady jsou soubory, které se mění za běhu systému.
`/var/mail` je systémová pošta. E-mail je standard starší, než internet - daly se posílat zprávy mezi uživateli počítače.

`/var/cache/` - tady se dávají soubory, které zrychlují práci, ale dají se vygenerovat kdykoli znova. 
Když si nějaký program stáhne něco z internetu, dá si to sem a příště to nemusí stahovat znova. 
Pokud se to smaže, nic se neděje, stáhne se to prostě zas.

`/var/run/` - další adresář pro soubory, které se mění za běhu počítače.
`/var/run/` může být *symlink* na `/var` nebo `/run`, všechna tři místa mají totiž stejný účel.


# Volné místo na disku

Kolik ti zbývá místa na disku, zjistíš pomocí příkazu `df`. 

```console
$ df
Filesystem     1K-blocks      Used Available Use% Mounted on
udev             3889620         0   3889620   0% /dev
tmpfs             782916      2184    780732   1% /run
/dev/sda1      245084444 206939492  25625684  89% /
tmpfs            3914572    201872   3712700   6% /dev/shm
tmpfs               5120         4      5116   1% /run/lock
tmpfs            3914572         0   3914572   0% /sys/fs/cgroup
```

Ale protože z toho výpisu to není moc jasné, použij přepínač `-h`, který znamená *human*, pro lidi. Sloupeček *Avail* obsahuje informaci o volném místě.

```console
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3,8G     0  3,8G   0% /dev
tmpfs           765M  2,2M  763M   1% /run
/dev/sda1       234G  198G   25G  89% /
tmpfs           3,8G  119M  3,7G   4% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
tmpfs           3,8G     0  3,8G   0% /sys/fs/cgroup
```
