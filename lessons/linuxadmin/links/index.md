# *inode* a odkazy

Vytvoř si soubor `basnicka.txt`.
Napiš do něj nějakou básničku.
To bute *obsah* souboru – většinou ta nejdůležitější část toho,
co soubor je.

## inode

Dnes se ale podíváme na to ostatní, co si počítač o souborech „pamatuje“:
jméno souboru, délka, přístupová práva a podobně.
Tenhle druh informací je uložen jako *inode* (česky *I-uzel*)
a přečíst je umí program `stat`:

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

Tenhle výstup je hezky naformátovaný.
Reálně je ale v *inode* uloženo jen několik čísel.
Čísla program `stat` ukáže s přepínačem `-t`:

```console
$ stat -t basnicka.txt
basnicka.txt 94 8 81b4 1000 1000 fd00 694527 1 0 0 1604926537 1604926537 1604926537 0 4096 unconfined_u:object_r:user_home_t:s0
```

> [note]
> Kontext `unconfined_u...` je součást bezpečnostního systému SELinux,
> který by vydal na zvláštní kurz. Zatím ho budeme ignorovat.

Stejná čísla můžeš získat i z Pythonu:

```console
$ python
>>> import os
>>> os.stat('basnicka.txt')
os.stat_result(st_mode=33204, st_ino=694527, st_dev=64768, st_nlink=1, st_uid=1000, st_gid=1000, st_size=94, st_atime=1604926537, st_mtime=1604926537, st_ctime=1604926537)
>>> os.stat('basnicka.txt').st_ino
694527
>>> os.stat('basnicka.txt').st_mtime
1604926537
>>> os.stat('basnicka.txt').st_mtime_ns
1604926537038584844
>>> exit()  # (nebo Ctrl+D)
```

> [note]
> Čtení těchto informací nám trochu ztěžuje fakt, že jedno číslo
> lze zapsat několika způsoby: *tisíc* je v desítkové soustavě `1000`,
> v šestnáctkové `3e8`, osmičkové `1750`, dvojkové `1111101000` –
> a nebo se dá vypsat jako příslušné jméno uživatele.

Podle výpisů výše můj soubor:
* Obsahuje 94 bajtů (`st_size`). Jeto krátká básnička.
* Je „normální“ datový soubor a má práva `rw-rw-r--` (`st_mode`).
  (Jako číslo to je `33204` v desítkové soustavě, `0664` v osmičkové.)
* Je *inode* číslo 694527 (`st_ino`) na disku 64768 (`st_dev`).
* Je jednou odkázaný (`st_nlink`). O tom za chvilku.
* Patří uživateli číslo 1000 (`st_uid`) a skupině 1000 (`st_gid`);
  `stat` si k číslům vyhledal jméno uživatele/skupiny.
* Má uložené časy posledního přístupu (`st_atime`, *access time*)*,
  poslední změny obsahu (`st_mtime`, *modification time*) a poslední
  změny *inode* (`st_ctime`, původně z *creation time*).
  (Číselně jsou časy uloženy jako počet (mili)sekund od 1. 1. 1970.)

  > [note] atime
  > Čas posledního přístupu se (ve výchozím nastavení na Fedoře) aktualizuje
  > jen „občas“, aby se zamezilo opotřebování disku příliš častým zapisováním.
  > Je přesný jen na dny, ne sekundy/milisekundy.

Obsah souboru není součást *inode*.
Je na disku uložen zvlášť – v osmi „blocích“ (`st_blocks`).
Které přesně bloky to jsou, to je v *inode* zaznamenáno taky,
ale `stat` tuhle informaci neukazuje.

> [note]
> Bloky jsou něco jako stranky v knížce: systém podle nich na disku
> hledá informace. Soubor („kapitola“) vždy začíná na začátku bloku.
> Soubory typicky zabírají minimálně osm bloků – a tady už paralela se
> stránkami v knížce selhává.

*Inode* mají všechny soubory – včetně adresářů,
rour a jiných speciálních souborů.
Vyzkoušej si `stat` pro aktuální adresář:

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

## Jména souborů

Jedna informace kterou *inode* neobsahuje je jméno souboru.
Program `stat` sice jméno vypíše, ale to je jen „hlavička“,
která říká k cemu se vztahují informace pod ní.

Zkus si pustit `stat` pro `.`, `./.`, což jsou to dvě jména pro ten samý
adresář.
Js-li v domovském adresáři, přidej i `~`:

```console
$ stat  .  ./.  ~
  Soubor: .
Velikost: 4096      	Bloků: 8          I/O blok: 4096   adresář
Zařízení: fd00h/64768d	I-uzel: 694505      Odkazů: 2
[...]
  Soubor: ./.
Velikost: 4096      	Bloků: 8          I/O blok: 4096   adresář
Zařízení: fd00h/64768d	I-uzel: 694505      Odkazů: 2
[...]
  Soubor: /home/petr
Velikost: 4096      	Bloků: 8          I/O blok: 4096   adresář
Zařízení: fd00h/64768d	I-uzel: 694505      Odkazů: 2
[...]
```

Dostaneš úplně stejné výpisy: `.` a `./.` jsou jen dvě jména pro ten stejný
*inode*.

Když je *inode* jen sada čísel, kde je tedy uloženo jméno souboru?

V adresáři, který soubor obsahuje.

Obsah adresáře si můžeš představit jako slovník v Pythonu:
přiřazuje jména konkrétním položkám.
Například:

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

Toto je skutečný obsah *adresáře*.
`ls -l` sice vypisuje informací víc, ale ty čte z *inode* jednotlivých souborů,
nikoli ze samotného adresáře.

Jméno souboru tedy není součástí samotného souboru – je to způsob,
jak se z adresáře k souboru dostat.

Proto může mít jeden soubor dvě různá jména.
To už jsme viděli u adresářů (`.` a `./.`), ale jde to i u normálních souborů.


## Odkazy

Pojďme udělat adresář se dvěma jmény pro stejný soubor:

* `basnicka.txt` → inode 694527
* `takybasnicka.txt` → inode 694527 (stejné jako výše!)
* `.` → inode 694505
* `..` → inode 694438

Použijeme k tomu příkaz `ln`.
Ten funguje podobně jako `cp`, ale netvoří kopii.
Jen přidá odkaz na stejný soubor:

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

> [note]
> Pro zápis použij `echo` nebo `nano`; složitější editory můžou při
> uložení vytvořit nový soubor.

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

Obě jména označují stejný soubor.
Tomu se říká *hardlink* (*tvrdý odkaz*). A k čemu je to užitečné?
Tvrdé odkazy se často používají na šetření místem na disku:
když najdeš dva stejné soubory (které se nebudou měnit),
můžeš zařídit aby je systém uložil jen jednou.
Popravdě se dnes *hardlinky* příliš nepoužívají.
Ale je dobré vědět, že existuji.

Souborový systém eviduje, kolik jmen daný soubor má – to je položka `st_nlink`
s *inode*.
Ukazuje ji i `ls -l`, hned ve druhém sloupci:

```console
$ ls -al
celkem 16
drwxrwxr-x. 2 petr petr 4096  9. lis 14.50 .
drwxrwxr-x. 3 petr petr 4096  9. lis 14.43 ..
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 basnicka.txt
-rw-rw-r--. 2 petr petr    6  9. lis 14.53 takybasnicka.txt
$  #        ^
$  #        |
```

Když smažeš jeden ze souborů, číslo se sníží o 1. 
Když smažeš druhý, číslo spadne na nulu a systém bude vědět,
že daný soubor už není potřeba a smaže ho.


## Symbolický odkaz

Užitečnější a frekventovanější je *symbolický odkaz*
(angl. *symbolic link*, *symlink*).

Ten funguje jako „odkaz“: je v něm napsáno, že tenhle soubor se nachází jinde.

Symbolický odkaz vytvoříš pomocí příkazu `ln` s přepínačem `-s`:

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

`ls -l` zobrazí symbolický odkaz pomocí šipky: `BASEN -> basnicka.txt`.
Čísla *inode* obou souborů jsou jiná – nejsou to dvě jména stejného souboru.
Text básničky je uložený v `basnicka.txt`; v `BASEN` je jen informace,
že obsah se nachází v souboru `basnicka.txt`.

S odkazem `BASEN` můžeš pracovat jako s jinými soubory.
Když ho otevřeš pro čtení, systém sáhne po místě,
na které odkazuje, tedy `basnicka.txt`, a otevře ten.

```console
$ cat BASEN
haló
```

Podobně fngují ostatní operace jako zápis do souboru nebo procházení adresáře.

Taky si všimni, že symbolický odkaz má povolena všechna práva.
Systém vždy použije práva odkazovaného souboru.


### Odkaz na nejnovější verzi

Symbolické odkazy jsou velmi užitečné.

Když potřebuješ mít soubor na určitém místě na disku,
ale nechceš ho kopírovat, udělej symbolický odkaz.

Často uvidíš soubor *nejnovější verze* jako symbolický odkaz na
nejnovější verzi nějakého souboru.
Třeba na Fedoře je příkaz `python` odkazem na `python3`,
a to je zase odkazem na `python3.8`:

```console
$ ls -l /usr/bin/python*
lrwxrwxrwx. 1 root root     9 25. zář 15.18 /usr/bin/python -> ./python3
...
lrwxrwxrwx. 1 root root     9 25. zář 14.55 /usr/bin/python3 -> python3.9
-rwxr-xr-x. 1 root root 15536 25. zář 14.55 /usr/bin/python3.9
```

Když tedy chci konkrétní verzi Pythonu, spustím `python3.9` nebo
(po instalaci) `python3.10`.
Když chci Python 3, spustím příkaz `python3`, což reprezentuje verzi,
která je na mém systému nejvhodnější – a po aktualizaci systému se může změnit.
A když chci jakýkoli Python, spustím `python`, což je odkaz na `python3`,
a to je odkaz na konkrétní verzi.

### Přesměrování

*Symlink* může odkazovat i na adresář.

V některých (často historických) verzích UNIXu byly některé příkazy v `/bin/`
(např. `/bin/bash`) a jiné v `/usr/bin` (např. `/usr/bin/python`).
Autoři Fedory si ale myslí, že tohle rozdělení je zbytečné a všechny programy
by měly být na stejném místě.

Respektive na obou místech, aby staré programy fungoivaly dál, ať už
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

Když takový soubor budeš chtít otevřit, systém řekne, že takový soubor neexistuje:

```console
$ cat NIC
cat: NIC: No such file or directory
```

Co s takovými symbolickými odkazy „nikam“?
Pokud máš nějaké velké soubory (třeba videa z dovolené) na externím disku
a chceš s nimi pracovat na svém počítači, kde ti dochází místo,
můžeš si vytvoři *symlinky* na určitá místa na externím disku.
Ty můžeš třeba zorganizovat do adresářů podle data.
Když disk odpojíš, soubory nebudou k dispozici, ale *symlinky*
na ně zůstanou.
Když disk zase připojíš pod stejným jménem, odkazy začnou fungovat.


### V klikátku

V grafickém prohlížeči souborů můžeš odkazy dělat pomocí
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+přetažení myší.
A podle ikonky se šipečkou poznáš, že se jedná o symbolický odkaz.
