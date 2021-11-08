# Poznámky – správa uživatelů

Dnes se podíváme na uživatelské účty.
Na Linuxovém systému může mít účet více uživatelů.

Dokonce může systém používat několik uživatelů zároveň.
U počítačů, které mají jednu obrazovku a jednu klávesnici se to moc nevidí,
ale i existují systémy s více obrazovkami – a častěji servery bez obrazovky,
kam se uživatelé připojují vzdáleně.
Nebo se živí uživatelé nepřipojují a procesy běží jejich jménem automaticky.

Každý uživatel má své uživatelské jméno.
To svoje jsi zadával{{a}} při instalaci systému.
Když chceš zjistit jaké je, můžeš použít příkaz `whoami`:

```console
$ whoami
hanka
```


## Uživatelé

Už víš, že u procesů a souborů je vždy uvedeno jméno uživatele.
Většinou je to tvoje uživatelské jméno nebo `root`, ale občas zahlédneš i jiné.

```console
$ ps -Af | tail -n6
hanka       2835    2474  0 11:02 pts/1    00:00:00 bash
root        3179       2  0 11:02 ?        00:00:00 [kworker/0:3-cgroup_destroy]
root        3239     972  0 11:04 ?        00:00:00 systemd-userwork
root        3242     972  0 11:05 ?        00:00:00 systemd-userwork
root        3243     972  0 11:05 ?        00:00:00 systemd-userwork
hanka       3288    2835  0 11:06 pts/1    00:00:00 ps -Af
$ ls -l | tail -n6
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Obrázky
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Plocha
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Stažené
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Šablony
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Veřejné
drwxr-xr-x. 1 hanka hanka   0  4. říj 16.35 Videa
$ ls -l /usr/bin | tail -n6
-rwxr-xr-x. 1 root root       70416 29. led  2021 zipnote
-rwxr-xr-x. 1 root root       66256 29. led  2021 zipsplit
-rwxr-xr-x. 1 root root        2210 26. led  2021 zless
-rwxr-xr-x. 1 root root        1846 26. led  2021 zmore
-rwxr-xr-x. 1 root root        4557 26. led  2021 znew
lrwxrwxrwx. 1 root root           6 17. bře  2021 zsoelim -> soelim
```

U souborů, ve výstupu `ls -l`, je jméno zopakováno.
První jméno označuje vlastníka souboru; druhé je jméno *skupiny*
která soubor vlastní.
Každý uživatel má svoji vlastní skupinu, která se používá když
něco není potřeba sdílet skupinově.

Uživatel může být členem více skupin.
Podívej se, v jakých skupinách jsi ty:

```console
$ groups
hanka wheel
```

Skupina `wheel` je pro uživatele, kteří mají přístup k admistrátorským
oprávněním. Můžou tak např. přidávat další uživatele.

Příkaz `groups` bere i argumenty – uživatele, jejichž skupiny vypíše.
Koukni se třeba, že uživate `root` není ve skupině `wheel`:

```console
$ groups hanka root
hanka : hanka wheel
root : root
```

Ještě víc informacíti dá příkaz `id`, který u každého jména
(uživatele nebo skupiny) ukáže i číslo.
Systém používá tyhle čísla.
Jména jsou „pro lidi“ – používají se při výpisech a nastavování.

```
$ id
uid=1000(hanka) gid=1000(hanka) skupiny=1000(hanka),10(wheel) kontext=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
$ id hanka root
uid=1000(hanka) gid=1000(hanka) skupiny=1000(hanka),10(wheel)
uid=0(root) gid=0(root) skupiny=0(root)
```

> [note]
> Informace o „kontextu“ má co dělat s podsystémem SELinux, používaným pro
> lepší zabezpečení systému.
> Pro teď ji ignoruj.


## Kam nemůžeš

Už víš, že si můžeš dávat vlastní příkazy do `~/bin`.
Tenhle adresář je ve tvém domovském adresáři, do kterého ostatní uživatelé
počítače nemají přístup.

Kdybys chtěl{{a}} dát svůj program k dispozici všem uživatelům,
do adresáře `/usr/bin/` (nebo jiného sdíleného adresáře v `$PATH`),
narazíš.
Na podobných sdílených místech nemázádný tvůj proces právo soubory vytvářet,
měnit ani mazat:

```console
$ cp ~/bin/mujprogram /usr/bin/mujprogram
cp: nelze vytvořit obyčejný soubor '/usr/bin/mcd': Operace zamítnuta

$ echo "..." > /usr/bin/mujprogram
bash: /usr/bin/mujprogram: Operace zamítnuta

$ nano /usr/bin/mujprogram   # obdobná chyba nastane při uložení
```

Když budeš chtít podobně měnit systém, musíš poprosit administrátora systému,
aby změny udělal.
A když jsi administrátor{{gnd('em', 'kou')}} ty, budeš na tyhle změny
muset proces pustit pod administrátorským uživatelským účtem.
Ten je oddělený od toho obyčejného uživatelského, což má dobrý důvod:
je menší šance že systém změníš (nebo zničíš) nechtěně.

Jak to udělat, to si řekneme za chvíli; teď ještě něco k oprávněním.


## Práva souborů

„Práva“ u souborů zobrazuje `ls -l` v prvním sloupečku:

```console
$ cd ~/Dokumenty/data-shell/
$ ls -l pizza.cfg
-rw-r--r--. 1 hanka hanka    32 Aug  8  2019 pizza.cfg
```

> [note]
> Pro `ls -l` bývá nastavený alias `ll`, který klidně použij.
> Nemusí ale být na všech systémech (a na všech účtech),
> proto budu `ls -l` rozepisovat.

Práva (angl. *permissions*) jsou součástí *módu* (angl. *mode*),
který je zobrazen jako ta zkratkovitá sada symbolů: `-rw-r--r--.`
Pojďme rozluštit, co to znamená.

První znak udává typ souboru:

* `-` znamená normální soubor;
* `d` uvidíš u adresářů;
* další písmenka značí speciální soubory: `p` je roura (*pipe*), `c` nebo
  `d` označují „zařízení“ značí jako `/dev/null` nebo terminály (`/dev/tty*`),
  o odkazech (`l`) si řekneme později.

Po prvním písmenku jsou samotná práva: tři trojice písmen `rwx`,
kde každé může být nahrazeno pomlčkou: `rwx`, `rw-`, `r--`, `-w-` apod.
První trojice se vztahuje k vlastníkovi, druhá ke skupině a třetí k ostatním
uživatelům.
Podívejme se napřed na první trojici.

### Čtení – `r`

Když je první písmenko trojice, `r`, udává právo ke čtení.
Pomlčka znamená, že nemá právo soubor přečíst.

Můžeš si to vyzkoušet příkazem `chmod` (z angl. *change mode*, změnit mód):

* <code>chmod -r <var>jméno_souboru</var></code> právo odebere,
* <code>chmod +r <var>jméno_souboru</var></code> právo naopak přidá.

```console
$ ls -l pizza.cfg
-rw-r--r--. 1 hanka hanka    32 Aug  8  2019 pizza.cfg
$ chmod -r pizza.cfg
$ cat pizza.cfg
cat: pizza.cfg: Operace zamítnuta
$ ls -l pizza.cfg
--w-------. 1 hanka hanka 32 Aug  8  2019 pizza.cfg
$ chmod +r pizza.cfg
$ ls -l pizza.cfg 
-rw-r--r--. 1 hanka hanka 32  8. srp  2019 pizza.cfg
```

Všimni si že právo ke čtení se vztahuje na *obsah* souboru.
Jméno, mód, vlastníka a ostatní vlastnosti souboru můžeš zjistit i bez práva
ke čtení obsahu.


### Zápis – `w`

Druhé písmenko udává právo ke čtení.
Když je zde místo `w` pomlčka, vlastník do souboru nesmí zapisovat.
Toto oprávnění se dá změnit pomocí `chmod -w` a `chmod +w`:

```console
$ chmod -w pizza.cfg
$ ls -l pizza.cfg
-r--r--r--. 1 hanka hanka 32  8. srp  2019 pizza.cfg
$ echo Nový řádek >> pizza.cfg
bash: pizza.cfg: Operace zamítnuta
$ chmod +w pizza.cfg
$ ls -l pizza.cfg
-rw-rw-r--. 1 hanka hanka 32  8. srp  2019 pizza.cfg
$ echo Nový řádek >> pizza.cfg
$ cat pizza.cfg
alpha: 0.5
beta: 0.2
gamma: off
Nový řádek
```

### Spuštění – `x`

Poslední z trojice je právo spuštění, `x`, které už znáš:
pomocí `chmod +x` označuješ spustitelné soubory.

```console
$ ./pizza.cfg 
bash: ./pizza.cfg: Operace zamítnuta
$ chmod +x pizza.cfg 
$ ls -l pizza.cfg
-r-xr-xr-x. 1 hanka hanka 32  8. srp  2019 pizza.cfg
$ ./pizza.cfg  # Soubor bez "shebangu" se Bash pokusí spustit jako skript
./pizza.cfg: řádek 1: alpha:: příkaz nenalezen
./pizza.cfg: řádek 2: beta:: příkaz nenalezen
./pizza.cfg: řádek 3: gamma:: příkaz nenalezen
$ chmod -x pizza.cfg 
$ ls -l pizza.cfg
-r--r--r--. 1 hanka hanka 32  8. srp  2019 pizza.cfg
```

Všechny soubory s příkazy jsou spustitelné:

```console
$ type cat gedit
cat je zahashován (/usr/bin/cat)
gedit je /usr/bin/gedit
$ ls -l /usr/bin/cat
-rwxr-xr-x. 1 root root 37032 26. bře  2021 /usr/bin/cat
$ ls -l /usr/bin/gedit
-rwxr-xr-x. 1 root root 16072  2. dub  2021 /usr/bin/gedit
```

### `x` u adresářů

Zvláštní význam má právo `x` u adresářů, u kterých je téměř vždy nastaveno.

Když se budeš dívat na adresáře, bude se hodit přepínač `-d`,
se kterým `ls` místo *obsahu* adresáře popíše *adresář* samotný.
Vyzkoušej si:

```console
$ ls -l .
$ ls -l -d .
drwxr-xr-x. 8 hanka hanka 4096  6. říj 23.40 .
```

A co znamená to `x`?
To je právo adresář *procházet*, tedy přistupovat k tomu, co je v něm.
Když `x` pro nějaký adresář odebereš, přijdeš o právo jakkoli pracovat se
soubory (a podadresáři) v něm:

```console
$ chmod -x .
$ ls .
ls: nelze přistoupit k '.': Operace zamítnuta
$ cat pizza.cfg
cat: pizza.cfg: Operace zamítnuta
$ cat ~/Dokumenty/data-shell/pizza.cfg
cat: /home/hanka/Dokumenty/data-shell/pizza.cfg: Operace zamítnuta
```

To se vztahuje i na adresář `.` (neboli `./.`), takže nejde jen tak
aktuálnímu adresáři právo `x` zpět přidat:

```console
$ chmod +x .
chmod: nelze přistoupit k '.': Operace zamítnuta
```

V tomhle případě můžeš do *nadřazeného* adresáře – to ještě smíš.
Z něj sice nemůžeš “zpět” do `data-shell`:

```console
$ cd ..
$ cd data-shell/
bash: cd: data-shell/: Operace zamítnuta
```

Ale můžeš pro `data-shell` přidat právo `x` a situaci vrátit „do normálu“:

```console
$ chmod +x data-shell
$ cd data-shell/
$ ls
creatures  diplomka_archiv  north-pacific-gyre  plan.txt   writing
data       molecules        pizza.cfg           solar.pdf
$ ls -ld . pizza.cfg
drwxr-xr-x. 8 hanka hanka 4096  6. říj 23.40 .
-r--r--r--. 1 hanka hanka   32  8. srp  2019 pizza.cfg
```

### Vlastník, skupina a ostatní

'rwx' se opakuje 3× – pro *vlastníka*, *skupinu* a *ostatní*.
Vlastník i skupina, která soubor vlastní, jsou ve výstupu `ls -l` vidět:

```console
$ ls -l pizza.cfg
-rw-rw-r--. 1 hanka hanka 36  8. lis 14.59 pizza.cfg
```

Práva pro konkrétní kategoriii z těchto tří můžeš nastavit tak, že příkazu
`chmod` dáš před `+` nebo `-` jedno z písmen
`u` (*user* – uživatel, vlastník), `g` (*group* – skupina)
a `o` (*others*, ostatní) nebo `a` (*all* – všechny tři). Například:

```console
$ chmod u+x .
$ chmod g+x .
$ chmod o+x .
$ chmod a-x .
```

Kromě `+` a `-` rozumí `chmod` symbolu `=`, které práva nastaví na danou sadu
(tj. zadaná přidá a nezadaná odebere).
A několik specifikací se dá spojit pomocí čárky, takže můžeš konkrétní práva nastavit třeba takto:

```console
$ chmod u=rw,g=r,o= pizza.cfg
```


Když so podíváš na svůj domovský adresář, zjistíš že to do něj nikdo jiný
nevleze: `rwx` je tu jen pro tebe.

```console
$ ls -ld ~
drwx------. 25 hanka hanka 4096  2. lis 14.31 /home/hanka
```

Když tahle práva odebereš i sobě (`chmod u-rwx ~`),
bude tvůj systém podobně nepoužitelný jako kdyby chtěl tvůj uživatelský učet
používat někdo jiný.
Běžící procesy se neukončí, ale můžeš mít problém například
otevřít nový terminál nebo prohlížeč.
Příkaz `chmod u+rwx ~` ale vše vrátí do pořádku.


### Změna skupiny

Skupinu můžeš u souboru který vlastníš změnit příkazem
`chgrp` (z angl. *change group*, změň skupinu):

```console
$ chgrp wheel pizza.cfg
$ ls -l pizza.cfg
--w-rw----. 1 hanka wheel 36  8. lis 14.59 pizza.cfg
```

To ovšem pro tebe nic nezmění.
Vždy totiž platí jen první trojice `rwx`, která se na soubor vztahuje, takže:

* první `rwx` platí pro vlastníka,
* druhé `rwx` platí pro všechny ze skupiny *kromě* vlastníka,
* třetí `rwx` platí pro všechny ostatní – ty, kdo nejsou ani vlastníkem ani
  nepatří do skupiny.

A vlastníka souboru změnit nemůžeš.
Tuhle operaci bys totiž nemohl{{a}} vzít zpět
(to by musel provést nový vlastník), takže tohle lze udělat jen
z administrátorského účtu.
Tam k tomu slouží příkaz `chown` (z angl. *change owner*, změň vlastníka)

Pojďme se podívat jak se přihlásit pod jiným účtem;
za domácí úkol si pak s `chgrp` a `chown` pohraj.


## Změna uživatele

Kdo jsi – tedy pod jakým účtem jsi přihlášen{{a}} – ti řekne příkaz `id`:

```console
$ id
```

Chceš-li se přihlásit pod jiným účtem, můžeš použít příkaz `su`.
Kdyby na systému existoval uživatel `pepa`, mohl{{a}} bys napsat:

```console
$ su pepa
```

(`su` je z angl. *set user*, ale lépe
se to pamatuje jako moravské „od včil *su Pepa*“).

Pojď si to vyzkoušet s účtem `root`, který jsi už viděl{{a}} ve výstupu
`ps` a `ls`:

```console
$ su root
Heslo: 
```

Zjistíš že k účtu `root` neznáš heslo.
Tak to ukonči pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd>, zkusíme to jinak.


## Nový uživatel

V nastavení GNOME Shell (tom „klikacím“, z menu vpravo nahoře)
vyber sekci **Uživatelé**.
Tady můžeš přidávat nové uživatelské účty.
Aktuálně tam bude jen jeden – systémové účty jako `root` jsou skryté.

Při instalaci systému jsi nastavil{{a}}, že jsi správce.
To znamená že si tvoje procesy můžou *zažádat* o administrátorská práva;
typicky je ale dostanou až po zadání hesla.

Klikni tedy na tlačítko **Odemknout** a zadej svoje heslo.

> [note]
> Kdykoli se tě systém zeptá na heslo, znamená to že se buď přihlašuješ,
> nebo dělat něco, co ovlivní celý systém (tj. víc než jen tvůj účet) – a je
> to tedy potenciálně nebezpečné.
> Přidání uživatele zas tak nebezpečné není, ale rozhodně to ovlivní víc než
> jen tvůj účet.

Přes tlačítko **Přidat uživatele** teď přidej nového uživatele
s uživatelským jménem `test1`.
Heslo k novému účtu musí být složitější než `1234`; zapiš si ho na papír.

Když teď Nastavení zavřeš a otevřeš znovu, možnost měnit ostatní účty budeš
muset znovu „odemknout“ a zadat svoje heslo.

Gnome GUI na uživatele.
Jsem administrátor.
Vytvořit nového uživatele `test1` – chce to heslo, i když jsem "administrátor".
Co je to administrátor?

Zavřít nastavení, otevřít znovu – chce to heslo znovu.
Administrátor – systém mi přidělí oprávnění, když zadám *své* heslo.
„Být“ administrátor chci co nejmíň.

Přepni se teď do příkazové řádky a přepni se na nový účet `test`.
Když budeš vyplňovat heslo, na obrazovku se nic nevypíše (ani hvězdičky/tečky
jako když zadáváš heslo grafických aplikací).
To je u zadávání hesel do příkazové řádky normální:

```console
$ su test1
Heslo:
$ whoami
test1
```

Pak se porozhlédni.
Zjistíš, že se nezměnil aktuální adresář;
dokud v něm budeš, můžeš se dívat na soubory – když už někde „jsi“
systém nekontroluje, jestli tam máš přístup:

```console
$ pwd
/home/hanka/Dokumenty/data-shell
$ ls
creatures  molecules           notes.txt  solar.pdf
data       north-pacific-gyre  pizza.cfg  writing
$ ps -al
F S   UID     PID    PPID  C PRI  NI ADDR SZ WCHAN  TTY          TIME CMD
0 S  1000    1624    1614  0  80   0 - 118849 -     tty2     00:00:00 gnome-sess
4 S     0   11788    2835  0  80   0 - 63928 -      pts/1    00:00:00 su
4 S  1001   11794   11788  0  80   0 - 58240 -      pts/1    00:00:00 bash
4 S  1001   12107   12034  0  80   0 - 56507 -      pts/1    00:00:00 ps
```

Když se ale přepneš do domovského adresáře uživatele `test1`, zjistíš,
že se nejde přepnout zpátky:

```console
$ cd
$ pwd
/home/test1
$ cd -
bash: cd: /home/hanka/Dokumenty/data-shell: Operace zamítnuta
```

Navíc tu chybí adresáře jako `Obrázky` a `Video`.
Ty totiž vytváří grafické rozhraní – GNOME – které uživatel `test`
ještě nespustil:

```console
$ ls
$ ls -a
.   .bash_logout   .bashrc   .xauthf9T9mu
..  .bash_profile  .mozilla  .xauthIdeOEt
```

Přepni tedy uživatele „graficky“: v menu vpravo nahoře vyber
**Přepnout uživatele** (NE „odhlásit“).
Přihláění bude trvat déle – například o vytvoření výše zmíněných adresářů.

Když si teď pustíš **Nastavení** a budeš chtít odemknout nastavení uživatelů,
bude to po tobě chtít heslo *správce*, nikoli heslo uživatele `test1`.
Test totiž nemá právo přidávat účty; na to musí poprosit administrátora.

Přepni se teď zpátky na svůj hlavní účet (přes **Přepnout uživatele**,
ne „odhlásit“).

Zpátky ke konzoli.
Jsi-li v ní stále přihlášen{{a}} jako `test1`, napiš `exit`.
(Příkaz `su` pouští nový shell; když ho ukončíš, vrátíš se do shellu
ve kterém jsi pustil{{a}} `su`.)

```console
$ exit
$ whoami
hanks
```

Superuživatel `root` nemá heslo, takže `su root` nebude fungovat:

```console
$ su root
```

> [warning]
> Zbytek tohoto textu jsou zatím jen zkratkovité poznámky.
> Dopíšu je pro další běh kurzu.

## Kde ti uživatelé jsou

```console
$ less /etc/passwd
$ less /etc/shadow
```

```console
$ ls -l /etc/passwd /etc/shadow
```

Za zmínku stojí *superuživatel* `root`, který má zvláštní postavení:
může na systému dělat *cokoli*.
Oprávnění se na něj nevztahují.

## Přihlášení

Změnit uživatele může *jen* administrátor.
Jak tedy funguje `su`?

```console
$ su test1
$ ps -au
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
hanka        1504  0.0  0.1 371584  2812 tty2     Ssl+ 13:37   0:00 /usr/libe...
hanka        1507  1.1  3.3 1723936 67372 tty2    Sl+  13:37   1:03 /usr/libe...
hanka        1570  0.0  0.1 467944  3192 tty2     Sl+  13:37   0:00 /usr/libe...
hanka       15640  0.0  0.3 227968  6084 pts/0    Ss   14:31   0:00 bash
root       18130  0.0  0.4 246780  8444 pts/0    S    15:10   0:00 su test1
test1      18140  0.1  0.2 227404  5396 pts/0    S    15:10   0:00 -bash
test1      18171  0.0  0.1 228432  3980 pts/0    R+   15:10   0:00 ps -a -u
```

```console
$ type su
su je /usr/bin/su
$ ls -l /usr/bin/su
-rwsr-xr-x. 1 root root 58384 12. úno  2021 /usr/bin/su
```

Všimni si zvláštních písmenek v módu: `rws` – to `s` místo `x` znamená
*setuid*, program se spouští s právy vlastníka.
Je tedy jedno kdo `su` spustí; proces bude mít vždycky práva superuživatele
`root`, tj. může dělat *cokoli*.
Program `su` ale umí dělat jen jednu věc: zeptá na heslo a spustí shell pod
jiným uživatelským účtem.

Programy s módem *setuid* jsou napsány zvlášť opatrně,
aby umožnily opravdu jen to k čemu slouží.

```console
$ exit
$ whoami
hanka
```


## Sudo

```console
$ ls -l /usr/bin/sudo
```

```console
$ whoami
hanka
$ sudo whoami
root
$ sudo ps -a -u
$ sudo ll
$ sudo ls -l
```

```console
$ sudo less /etc/shadow
```

Jsme ve virtuálním počítači, pojďme ho trochu rozbít:

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
$ sudo su hanka
```

Uživatel pro kterého funguje `sudo` se ovšem může přihlásit jako superuživatel!

```console
$ sudo su root
# pwd
# exit
```

Všimni si výzvy (promptu) `#` místo `$` – občas ho najdeš v tutoriálech;
znamená, že musíš *být root*.
Dnes se častěji používá `sudo` před každým příkazem – být administrátorem
je nebezpečné.
Navíc `sudo` se dá nastavovat – pro každého uživatele říct, jaké příkazy
je oprávněn přes `sudo` spouštět.

Potřebuješ-li pustit příkaz jako `root`,
všimni si, že se `sudo` neptá na heslo pokaždé.
Pro konkrétní terminál si ho vždy nějakou dobu pamatuje, a to i mezi spuštěními.

Potřebuješ-li opravdu pustit nový shell jako `root`
(např. zachraňuješ počítač),
lepší než `sudo su root` je `sudo -i` (interaktivní):

```console
$ sudo -i
# pwd
# exit
```


## Změna hesla

Další věc, kterou může dělat jen administrátor, je změna hesla.
Existuje ale příkaz `passwd` který heslo mění – a má příznak *setuid*.

```console
$ type passwd
passwd je /usr/bin/passwd
[test1@fedora data-shell]$ ls -l /usr/bin/passwd
-rwsr-xr-x. 1 root root 32712 30. led  2021 /usr/bin/passwd
```
Zkus si změnit heslo na něco jednoduchého:

```console
$ passwd
Změna hesla uživatele hanka.
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
$ sudo usermod hanka -aG editors
$ groups hanka
$ groups
```

Uživatel a skupiny pro daný proces jsou vždy nastavené *při přihlášení*.
Aby se změny členství ve skupinách projevily, musíš se znovu přihlásit:

```
$ groups
$ su hanka
$ groups
$ ps a
```

{#

## Poznámka 1 – Další oprávnění




## Poznámka 2 – `-x` jen zabraňuje vstupu

Do domovského adresáře ti nikdo jiný nevleze:

```console
$ ls -ld ~
drwx------. 25 hanka hanka 4096  2. lis 14.31 /home/hanka
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

#}
