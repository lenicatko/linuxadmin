# Odkazy

Vytvoř si soubor `basnicka.txt`.

Jaké informace si systém o tomto souboru ukládá (kromě samotného obsahu)?

Informace o souboru jsou na disku zapsané v takzvaném *inode* (česky *I-uzel*).
Příkaz `stat` je umí přečíst:

```console
$ stat basnicka.txt
  Soubor: basnicka.txt
Velikost: 94        	Bloků: 8          I/O blok: 4096   běžný soubor
Zařízení: fd00h/64768d	I-uzel: 694527      Odkazů: 1
   Práva: (0664/-rw-rw-r--)  UID: ( 1000/    petr)   GID: ( 1000/    petr)
 Kontext: unconfined_u:object_r:user_home_t:s0
     Přístup: 2020-11-09 13:55:37.038584844 +0100
Změna obsahu: 2020-11-09 13:55:37.038584844 +0100
Změna i-uzlu: 2020-11-09 13:55:37.038584844 +0100
       Vznik: -
```

Tenhle výstup je hezky naformátovaný; reálně je tu ale uložených několik čísel:

```console
$ stat -t basnicka.txt
basnicka.txt 94 8 81b4 1000 1000 fd00 694527 1 0 0 1604926537 1604926537 1604926537 0 4096 unconfined_u:object_r:user_home_t:s0
```

```console
$ python
>>> import os
>>> os.stat('basnicka.txt')
os.stat_result(st_mode=33204, st_ino=694527, st_dev=64768, st_nlink=1, st_uid=1000, st_gid=1000, st_size=94, st_atime=1604926537, st_mtime=1604926537, st_ctime=1604926537)
>>> os.stat('basnicka.txt').st_mode
33204
>>> os.stat('basnicka.txt').st_mtime
33204
>>> os.stat('basnicka.txt').st_mtime_ns
1604926537038584844
>>> os.stat('basnicka.txt').st_mtime
33204
>>> exit()  # (nebo Ctrl+D)
```

Můj soubor:
* Obsahuje 94 bajtů (`st_size`).
* Je „normální“ datový soubor a má práva `-rw-rw-r--` (`st_mode`).
  (Jako číslo: 33204 desítkově, 0664 osmičkově.)
* Je uložený jako záznam číslo 694527 (`st_ino`) na disku 64768 (`st_dev`). (`fd00h` je jen jiný zápis čísla 64768.)
* Je jednou odkázaný (`st_nlink`). O tom za chvilku.
* Patří uživateli 1000 (`st_uid`) a skupině 1000 (`st_gid`).
* Má uložené časy posledního přístupu (`st_atime`, *access time*)*,
  poslední změny obsahu (`st_mtime`, *modification time*) a poslední
  změny *inode* (`st_ctime`, původně z *creation time*) jsou uloženy taky.
  (Číselně jako počet (mili)sekund od 1. 1. 1970.)

Obsah souboru je na disku uložen v osmi „blocích“ (`st_blocks`), což je
minimum.
Které přesně bloky to jsou, to je samozřejmě zaznamenáno taky,
ale `stat` tuhle informaci neukazuje.

> [note] atime
> Čas posledního přístupu se (ve výchozím nastavení) aktualizuje jen „občas“,
> aby se zamezilo opotřebování disku příliš častým zapisováním.
> Je přesný jen na dny, ne sekundy/milisekundy.

Tyto informace neobsahují jméno souboru! kdepak je jméno souboru?

V adresáři.

```console
$ stat .
  Soubor: .
Velikost: 4096      	Bloků: 8          I/O blok: 4096   adresář
Zařízení: fd00h/64768d	I-uzel: 694505      Odkazů: 2
   Práva: (0775/drwxrwxr-x)  UID: ( 1000/    petr)   GID: ( 1000/    petr)
 Kontext: unconfined_u:object_r:user_home_t:s0
     Přístup: 2020-11-09 14:43:39.820144055 +0100
Změna obsahu: 2020-11-09 14:43:37.653146851 +0100
Změna i-uzlu: 2020-11-09 14:43:37.653146851 +0100
       Vznik: -
```

Obsah tohoto adresáře je 4096 bytů – docela hodně!
Ale co je vlastně obsah adresáře?
Něco jako Pythonní slovník: přiřazuje jména konkrétním položkám.

* `basnicka.txt` → inode 694527
* `.` → inode 694505
* `..` → inode 694438

Neboli:

```console
$ ls -ai1
694505 .
694438 ..
694527 basnicka.txt
```

Toto je obsah *adresáře* – všechny `ls -l` informace *čte* z jednotlivých
souborů.

Jméno souboru je uvedeno právě tady – není součástí samotného souboru.

Co to znamená?

Jediný soubor může mít dvě jména!
Pojďme udělat adresář se dvěma jmény pro stejný soubor:

* `basnicka.txt` → inode 694527
* `takybasnicka.txt` → inode 694527 (stejné jako výše!)
* `.` → inode 694505
* `..` → inode 694438

Příkaz `ln` funguje podobně jako `cp`, ale netvoří *kopii* – jen přidá
odkaz na stejný soubor:

```console
$ ln basnicka.txt takybasnicka.txt
$ ls -ai1
694505 .
694438 ..
694527 basnicka.txt
694527 takybasnicka.txt
```

Pomocí `ls -l` nebo `cat` se můžeš přesvědčit, že obě básničky mají stejnou
velikost i obsah:

```console
drwxrwxr-x. 2 petr petr 4096  9. lis 14.50 .
drwxrwxr-x. 3 petr petr 4096  9. lis 14.43 ..
-rw-rw-r--. 2 petr petr   94  9. lis 14.43 basnicka.txt
-rw-rw-r--. 2 petr petr   94  9. lis 14.43 takybasnicka.txt
```

A když do souboru něco zapíšeš, projeví se to tam i tam – `basnicka.txt`
a `takybasnicka.txt` jsou jen dvě jména stejného souboru:

```console
$ echo haló > basnicka.txt
$ cat takybasnicka.txt
haló
$ ls -al
celkem 16
drwxrwxr-x. 2 petr petr 4096  9. lis 14.50 .
drwxrwxr-x. 3 petr petr 4096  9. lis 14.43 ..
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 basnicka.txt
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 takybasnicka.txt
```

Oba soubory jsou zcela stejné. Když změníš jeden, změní se i druhý.
Tomu se říká **hardlink** (*tvrdý odkaz*). A k čemu je to užitečné?
Tvrdé odkazy se často používají na šetření místem na disku:
když najdeš dva stejné soubory (které se nebudou měnit),
můžeš zařídit, aby je systém uložil jen jednou.
Popravdě se ale dnes *hardlinky* příliš nepoužívají.
Ale je dobré vědět, že existuji.

Souborový systém dokonce kontroluje, kolikrát je daný soubor v tomto do systému,
a to pomocí čísla `st_nlink`, na které se můžeš podívat ve druhém sloupci `ls -l`.

```console
$ ls -al
celkem 16
drwxrwxr-x. 2 petr petr 4096  9. lis 14.50 .
drwxrwxr-x. 3 petr petr 4096  9. lis 14.43 ..
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 basnicka.txt
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 takybasnicka.txt
                           ^
                           |
                           +-- toto počítadlo existence souboru na disku
                               (= kolikrát se na disku nachází)   
```

Když smažeš jeden ze souborů, číslo se sníží o 1. 
Když smažeš druhý, číslo spadne na nulu a systém bude vědět,
že daný soubor už není potřeba a smaže ho.


## Symbolický odkaz

Užitečnější a frekventovanější je **symlink** (symbolický odkaz).
Můžeš si představit, že tento druh odkazu říká:
"obsah tohoto souboru zde není, podívej se jinam".

Symbolický odkaz vytvoříš pomocí `ln` s přepínačem `-s`:

```console
$ ln -s basnicka.txt BASEN
$ ls -ail
celkem 16
694505 drwxrwxr-x. 2 petr petr 4096  9. lis 15.05 .
694438 drwxrwxr-x. 3 petr petr 4096  9. lis 14.43 ..
694508 lrwxrwxrwx. 1 petr petr   12  9. lis 15.05 BASEN -> basnicka.txt
694527 -rw-rw-r--. 2 petr petr    6  9. lis 14.53 basnicka.txt
694527 -rw-rw-r--. 2 petr petr    6  9. lis 14.53 takybasnicka.txt
```

Podívej se na výstup v terminálu. `BASEN -> basnicka.txt` je zobrazení symbolického odkazu.
Čísla *inode* obou souborů jsou jiná.
Soubor `BASEN` v sobě nemá uložený obsah naší básničky 
Je v něm uložená jen informace, že obsah se nachází v souboru `basnicka.txt`.

S odkazem `BASEN` můžeš pracovat jako s jinými soubory.
Když ho otevřeš pro čtení, systém sáhne po místě,
na které odkazuje, tedy `basnicka.txt`, a otevře ten.

```console
$ cat BASEN
haló
```

Taky si všimni, že symbolický odkaz má nastavena všechna práva.
Systém vždy použije práva odkazovaného souboru.


### Odkaz na nejnovější verzi

Symbolické odkazy jsou velmi užitečné.

Když potřebuješ mít soubor na určitém místě na disku,
ale nechceš ho kopírovat, udělej symbolický odkaz.

Často uvidíš soubor *nejnovější verze* jako symbolický odkaz na
nejnovější verzi nějakého souboru.
Třeba na mém systému příkaz `python` odkazem na `python3`,
a to je zase odkazem na `python3.8`:

```console
$ ls -l /usr/bin/python*
lrwxrwxrwx. 1 root root     9 25. zář 15.18 /usr/bin/python -> ./python3
...
lrwxrwxrwx. 1 root root     9 25. zář 14.55 /usr/bin/python3 -> python3.8
-rwxr-xr-x. 1 root root 15536 25. zář 14.55 /usr/bin/python3.8
```

Když tedy chci konkrétní verzi Pythonu, spustím `python3.8` nebo
(po instalaci) `python3.9`.
Když chci Python 3, spustím příkaz `python3`, což reprezentuje verzi,
která je na mém systému nejvhodnější – a po aktualizaci systému se může změnit.
A když chci jakýkoli Python, spustím `python`, což je odkaz na `python3`,
což je odkaz na konkrétní verzi.


### V klikátku

V grafickém prohlížeči souborů můžeš odkazy dělat pomocí
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+přetažení myší.
A podle ikonky se šipečkou poznáš, že se jedná o symbolický odkaz.


### Adresáře

*Symlink* může odkazovat i na adresář

Můžeš si například nastavit *symlink* na svůj domovský adresář:

```console
ln -s ~ tady-je-doma
ls -l
lrwxrwxrwx 1 petr petr         10 zář 28 12:46 tady-je-doma -> /home/petr
```

Na to je dobré dávat pozor, až budeš psát skript,
který něco čte a ukládá v adresářové struktuře - vždy si ověř, že odkazuješ na správné místo na disku.


### Přesměrování

V některých (často historických) verzích UNIXu byly některé příkazy v `/bin/`
(např. `/bin/bash`) a jiné v `/usr/bin` (např. `/usr/bin/python`).
Autoři Fedory si ale myslí, že tohle rozdělení je zbytečné a všechny programy
by měly být na stejném místě.

Respektive na obou místech, aby staré programy fungovaly dál, ať už
předpokládají `/bin/` nebo `/usr/bin`.

Jak tenhle problém vyřešit?

```console
$ ls -dl /bin
lrwxrwxrwx. 1 root root 7 28. led  2020 /bin -> usr/bin
```

Adresář `/bin` je symbolický odkaz. Prakticky se chová stejně jako `/usr/bin`.


### Odkaz nikam

Dá se udělat i odkaz někam, kde nic není.
```console
$ ln -s nic.txt NIC
$ ls -l NIC
lrwxrwxrwx. 1 petr petr  7  9. lis 15.22 NIC -> nic.txt
```

Když takový soubor budeš chtít otevřít, systém řekne, že takový soubor neexistuje:

```console
$ cat NIC
cat: NIC: No such file or directory
```

Co s takovými symbolickými odkazy „nikam“?
Pokud máš nějaké velké soubory (třeba videa z dovolené) na externím disku a chceš s nimi pracovat na svém počítači,
vytvoř si *symlinky* na určitá místa na externím disku.
Ty můžeš třeba zorganizovat do adresářů podle data.
Když disk odpojíš, soubory nebudou k dispozici, ale *symlinky*
na ně zůstanou.
Když disk zase připojíš, odkazy začnou fungovat.


