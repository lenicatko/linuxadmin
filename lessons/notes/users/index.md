# Poznámky – správa uživatelů

Úvod

```console
$ locate battery.png
$ eog /usr/share/icons/gnome/256x256/devices/battery.png
```

Hlasitost – systémové zvuky

```console
$ locate bark.ogg
$ ogg123 /usr/share/sounds/gnome/default/alerts/bark.ogg
bash: ogg123: Příkaz nebyl nenalezen...
Nainstalovat balíček „vorbis-tools“, který poskytuje příkaz „ogg123“? [N/y] y
```


## Uživatelé

```console
$ ps aux
```

```console
$ ls -l
```

```console
$ ls -l /usr/bin
```

```console
$ groups
$ whoami
$ id
```

Skupina `wheel` je pro administrátory.


## Kam nemůžeš

```console
$ echo "..." > /usr/bin/mujprogram
$ console: /usr/bin/mujprogram: Operace zamítnuta
```

```console
$ ll /usr/lib64/python*/site-packages
```

```console
$ ll /usr/share/icons/gnome/256x256/devices/
```

```console
$ nano /usr/bin/mujprogram
```

uložení → `Error writing /usr/bin/mujprogram: Permission denied`

```console
$ echo > /usr/share/sounds/gnome/default/alerts/bark.ogg
```

Pod svým uživatelským účtem nemůžeš zasahovat do sdílených částí systému.

Existují výjimky – instalace `ogg123` – byla pro všechny uživatele!



## Práva souborů

`ls -l` a `chmod`

Čtení – `r`

```console
$ cd ~/Dokumenty/data-shell/
$ cat pizza.cfg
alpha: 0.5
beta: 0.2
gamma: off
$ ls -l pizza.cfg
-rw-r--r--. 1 petr petr    32 Aug  8  2019 pizza.cfg
$ chmod -r pizza.cfg
$ ls -l pizza.cfg
--w-------. 1 petr petr 32 Aug  8  2019 pizza.cfg
cat: pizza.cfg: Operace zamítnuta
$ chmod +r pizza.cfg
$ ls -l pizza.cfg 
-rw-r--r--. 1 petr petr 32  8. srp  2019 pizza.cfg
```

Zápis – `w`

```console
$ chmod -w pizza.cfg
$ echo > pizza.cfg 
bash: pizza.cfg: Operace zamítnuta
$ ls -l pizza.cfg
-r--r--r--. 1 petr petr 32  8. srp  2019 pizza.cfg
$ chmod +w pizza.cfg
$ ls -l pizza.cfg
-rw-rw-r--. 1 petr petr 32  8. srp  2019 pizza.cfg
```

Spuštění – `x`

```console
$ ./pizza.cfg 
bash: ./pizza.cfg: Operace zamítnuta
$ chmod +x pizza.cfg 
$ ./pizza.cfg
./pizza.cfg: řádek 1: alpha:: příkaz nenalezen
./pizza.cfg: řádek 2: beta:: příkaz nenalezen
./pizza.cfg: řádek 3: gamma:: příkaz nenalezen
$ ls -l pizza.cfg
-r-xr-xr-x. 1 petr petr 32  8. srp  2019 pizza.cfg
$ chmod -x pizza.cfg 
$ ls -l pizza.cfg
-r--r--r--. 1 petr petr 32  8. srp  2019 pizza.cfg
```

`x` u adresářů

```console
$ ls -l .
$ ls -l -d .
drwxr-xr-x. 8 petr petr 4096  6. říj 23.40 .
```

To `-d` zobrazí info pro samotný adresář.

```console
$ chmod -x .
$ ls .
ls: nelze přistoupit k '.': Operace zamítnuta
$ cat pizza.cfg
cat: pizza.cfg: Operace zamítnuta
$ cat ~/Dokumenty/data-shell/pizza.cfg
cat: /home/petr/Dokumenty/data-shell/pizza.cfg: Operace zamítnuta
$ chmod +x .
chmod: nelze přistoupit k '.': Operace zamítnuta
```

oops...

```console
$ cd ..
$ chmod +x data-shell
$ cd data-shell/
$ ls
creatures  diplomka_archiv  north-pacific-gyre  plan.txt   writing
data       molecules        pizza.cfg           solar.pdf
$ ls -ld . pizza.cfg
drwxr-xr-x. 8 petr petr 4096  6. říj 23.40 .
-r--r--r--. 1 petr petr   32  8. srp  2019 pizza.cfg
```

'rwx' se opakuje 3× – pro *uživatele*, *skupinu* a *všechny*.

```console
$ chmod u+x data-shell
$ chmod g+x data-shell
$ chmod o+x data-shell
$ chmod a-x data-shell
```

Do domovského adresáře ti nikdo jiný nevleze – `rwx` je tu jen pro tebe:

```console
$ ls -ld ~
drwx------. 25 petr petr 4096  2. lis 14.31 /home/petr
```


## Změna uživatele

```console
$ id
```

```console
$ su moje_uzivatelske_jmeno
```

```console
$ su root
```

... neznáš heslo.


## Nový uživatel

Gnome GUI na uživatele.
Jsem administrátor.
Vytvořit nového uživatele `test1` – chce to heslo, i když jsem "administrátor".
Co je to administrátor?

Zavřít nastavení, otevřít znovu – chce to heslo znovu.
Administrátor – systém mi přidělí oprávnění, když zadám *své* heslo.
„Být“ administrátor chci co nejmíň.

```console
$ su test1
$ pwd
$ ls
$ ps
```

```console
$ ls -al
```

Přepnout uživatele v GUI (NE odhlásit!) → vytvoří se adresáře.
Nastavení uživatelů – chce to heslo administrátora...

Přepnout zpátky – neodhlašovat.

Zpátky ke konzoli.
K přepnutí uživatele potřebuješ heslo:

```console
$ su moje_uzivatelske_jmeno
$ exit
```

Superuživatel nemá heslo!

```console
$ su root
```

```console
$ exit
```


## Kde ti uživatelé jsou

```console
$ less /etc/passwd
$ less /etc/shadow
```

```console
$ ls -l /etc/passwd /etc/shadow
```


## Přihlášení

Změnit uživatele může *jen* administrátor.
Jak tedy funguje `su`?

```console
$ su test1
$ ps -a -u
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
petr        1504  0.0  0.1 371584  2812 tty2     Ssl+ 13:37   0:00 /usr/libe...
petr        1507  1.1  3.3 1723936 67372 tty2    Sl+  13:37   1:03 /usr/libe...
petr        1570  0.0  0.1 467944  3192 tty2     Sl+  13:37   0:00 /usr/libe...
petr       15640  0.0  0.3 227968  6084 pts/0    Ss   14:31   0:00 bash
root       18130  0.0  0.4 246780  8444 pts/0    S    15:10   0:00 su test1
test1      18140  0.1  0.2 227404  5396 pts/0    S    15:10   0:00 -bash
test1      18171  0.0  0.1 228432  3980 pts/0    R+   15:10   0:00 ps -a -u
```

```console
$ ls -l /usr/bin/su
```

```console
$ exit
```


## Sudo

```console
$ ls -l /usr/bin/sudo
```

```console
$ whoami
$ sudo whoami
$ sudo ps -a -u
$ sudo ll
$ sudo ls -l
```

```console
$ sudo less /etc/shadow
```

Jsme ve virtuálním počítači, pojďme he trochu rozbít:

```console
$ nano /usr/bin/zkouska
```

```console
$ sudo nano /usr/bin/zkouska
```

Když jsem (přes `sudo`) superuživatelem, `su` se mě neptá na heslo:

```console
$ sudo su test1
```

Ale `sudo` funguje to jen pro *administrátory* (`wheel`):

```console
$ sudo su petr
```

Můžu se ovšem přihlásit jako superuživatel!

```console
$ sudo su root
# pwd
# exit
```

Všimni si promptu `#` – občas ho najdeš v tutoriálech;
znamená že musí *být root*.
Dnes se častěji používá `sudo` před příkazem – být administrátorem
je nebezpečné.
Navíc `sudo` se dá nastavovat – pro každého uživatele říct, jaké příkazy
je oprávněn přes `sudo` spouštět.

Potřabuješ-li pustit příkazů jako `root`,
všimni si, že se `sudo` neptá na heslo pokaždé.
Pro konkrétní terminál si ho vždy nějakou dobu pamatuje, a to i mezi spuštěními.
(Není v tom žádná velká magie – viz `sudo ls /var/run/sudo/ts`.)

Potřabuješ-li opravdu pustit celý shell jako `root`
(např. zachraňuješ počítač),
lepší než `sudo su root` je `sudo -i` (interaktivní):

```console
$ sudo -i
# pwd
# exit
```


## Změna hesla

Další věc kterou může dělat jen administrátor je změna hesla.
Existuje ale *setuid* `passwd`, který heslo mění!

Zkus si změnit heslo na něco jednoduchého:

```console
$ passwd
Změna hesla uživatele petr.
Current password: 
Nové heslo: 
ŠPATNÉ HESLO: Heslo je kratší než 8 znaků
passwd: Chyba manipulace s autentizačním tokenem
```

Nejde to!
Stejně tak nesmíš měnit heslo jiného uživatele:

```console
$ passwd test1
passwd: Pouze root smí zadat uživatelské jméno.
```

Ale `root` může vše!

```console
$ sudo passwd test1
Změna hesla uživatele test1.
Nové heslo: 
ŠPATNÉ HESLO: Heslo je kratší než 8 znaků
Opakujte nové heslo: 
passwd: všechny autentizační tokeny byly úspěšně změněny.
```

(Hláška `ŠPATNÉ HESLO` se sice objeví, ale je jen informační – heslo může
být klidně jednopísmenné.)


## Instalace balíčků

```console
$ sudo dnf remove vorbis-tools
$ ogg123 /usr/share/sounds/gnome/default/alerts/bark.ogg
bash: /usr/bin/ogg123: Adresář nebo soubor neexistuje
```

Instalace by spustila pomocný *setuid* program, který umí *pouze* instalovat
nový software, ale ne aktualizovat/měnit/odstraňovat existující.
Neovlivňuje tak ostatní uživatele, tak jako to dělá `dnf remove`.

Příkaz `dnf` ale umí všechno, co se týká správy instalovaného software.
Třeba instalovat nové věci:

```console
$ sl
bash: sl: Příkaz nebyl nenalezen...
Podobný příkaz je: 'ls'
$ sudo dnf install sl
$ sl
```

Pomocí `dnf` můžeš instalovat jak grafické aplikace (na které funguje
i grafický nástroj `gnome-software`) tak i programy pro příkazovou řádku
nebo systémové knihovny.
Taky se můžeš např. podívat jaký  software máš nainstalovaný
(což jde i bez `sudo`):

```
$ sudo dnf list installed
```

Taky můžeš systém aktualizovat:

```console
$ sudo dnf update
```


Programy pro instalaci software jsou jedna z věcí, které se velice často liší
mezi různými distribucemi Linuxu.
Fedora má `dnf`, ale Debian a Ubuntu mají `apt-get`, Arch `pacman`
(*pac*kage *man*ager) atd. Každý se používá trošku jinak.


## Přidání uživatele v konzoli

```console
$ useradd test2
```

```console
$ sudo useradd test2
$ sudo ls -l /home/test2
$ sudo ls -al /home/test2
$ sudo ls -al ~test2
```

```console
$ cat /etc/passwd
```

```console
$ userdel test2
$ sudo ls -al ~test2
$ sudo ls -al /home/test2
```

```console
$ sudo useradd test2
```


## Skupiny

```console
$ groups
$ groups test1
```

```console
$ sudo groupadd editors
$ sudo usermod test1 -aG editors
$ groups test1
```

```console
$ sudo usermod petr -aG editors
$ groups petr
$ groups
```

Uživatel a skupiny pro daný proces jsou vždy nastavené *při přihlášení*.
Aby se změny členství skupinách projevily, musíš se znovu přihlásit:

```
$ groups
$ su petr
$ groups
$ ps a
```


## Poznámka 1 – Další oprávnění




## Poznámka 2 – `-x` jen zabraňuje vstupu

Do domovského adresáře ti nikdo jiný nevleze:

```console
$ ls -ld ~
drwx------. 25 petr petr 4096  2. lis 14.31 /home/petr
```

Ale když už tam (jako jiný uživatel) jsem, můžu se procházet:

```console
$ cd ~/Dokumenty/data-shell/
$ su test1
Heslo: 
$ ls
creatures  diplomka_archiv  north-pacific-gyre  plan.txt   writing
data       molecules        pizza.cfg           solar.pdf
```

(`su` bez `-` nedělá „login“)


## Poznámka 3 – proč ne `useradd`

Je `groupadd`, `groupmod` a `groupdel`; `usedmod` a `userdel`, proč `adduser`
a ne `useradd`?

Příkaz `useradd` historicky sice vytvořil uživatele, ale nenastavil mu skupinu
a shell, nevytvořil domovský adresář a tak dále.


## Poznámka poslední

```console
$ sudo poweroff
```


