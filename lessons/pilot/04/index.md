
# 4. Linuxová administrace - operační systém


## Linux

Dnes se podíváme na to, jak funguje operační systém.
Na Wikipedii pod heslem Linux najdeš hezký diagram, popisující systémové vrstvy.
https://en.wikipedia.org/wiki/Linux#Design

Úplný základ je hardware - procesor, paměť, pevný disk, zkrátka všechny součástky nacházející se uvnitř tvého počítače. 
Hardwarem se v tomto kurzu nezabýváme a ani nebudeme zabývat. 

Další vrstva je linuxové jádro. 
Jádro systému je program, který komunikuje s hardwarem, zprostředkovává služby hardwaru ostatním programům. 
Jádro se stará o to, aby programy mohly běžet vedle sebe, taky o správu uživatelů apod. 
V případě, že běží víc programů / procesů najednou, jádro zajišťuje, aby se mezi sebou střídaly (plánovač procesů). 
Jádro se stará o vykreslení pixelů na obrazovku, pokud o to požádá grafický program. 
Když chceš něco uložit na disk, jádro převezme informaci o tom, co uložit do jakého souboru, a provede to. 
Když nějaký program chce zahrát zvuk, pošle informaci jádru.

Komunikace s jádrem není úplně jednoduchá - je to totiž spousta čísel, které označují, co se má kam zapsat nebo přečíst. 
Proto existují knihovny, které práci s jádrem zjednodušují.
Zpřístupňují například funkci `open()`, která se stará o otevření souborů, tzn. komunikuje s jádrem, řekne mu, který soubor chce otevřít, dává tomu hezčí rozhraní, které pak používají programátoři. 
Na linuxových systémech se nejvíc používá knihovna, která se jmenuje `glibc`. 
Je to standardní knihovna programovacího jazyka C, obsahuje všechny funkce, které se dají volat v jádru a zprostředkovává jeho služby dalším programům.

Na této vrstvě jsou postaveny další programy. 
Patří k ním tzv. `daemony`, které běží pořád. 
Jeden z nich je `systemd`, který se stará o spouštění a správu ostatních `daemonů`. 
V praxi to znamená, že např. spouští zvukový systém, a když ten systém spadne, tak ho restartuje. 
Stará se o to, že při startu počítače naběhne přihlašovací obrazovka a o její vzhled. 
U serveru se stará o programy tohoto serveru, a akce v případě, že programy spadnou: zda se má poslat někomu e-mail, nebo restartovat program, nebo restartovat celý počítač.
Jiný druh programů jsou okénkové systémy, např. `Wayland`, který se stará o vykreslování grafických oken na obrazovku. 
Když ti běží víc grafických procesů najednou a každý má nějaké okno, v pozadí běží proces, `daemon`, který se jmenuje `wayland`, a ten bere ty všechny grafické instrukce od programů, spojuje je dohromady a výsledek vykreslí. 
Přes systém komunikuje s grafickou kartou. Všechny obrázky spojuje do jednoho.
Další druh programů jsou ostatní knihovny k použití, např. grafické GTK+, Qt vykreslují grafické ovládací prvky (jak vykreslit tlačítko, na které se dá kliknout a zavolání funkce pod tlačítkem atd.).

Na těchto knihovnách jsou postavené uživatelské programy jako prohližeče, editory nebo i Bash.

Reálný svět je o to složitější, že pro mnoho komponent existuje více variant (implementací). 
Když se ti nelíbí linuxové jádro, můžeš použit jádro BSD. 
Chová se trochu jinak, je více méně kompatibilní s ostatními vrstvami. 
Když se ti nelíbí `glibc`, existují alternativní knihovny. 
Android používá linuxové jádro, ale se zbytkem linuxového světa nemá moc společného, je postaven na jiné knihovně.
Místo `systemd` můžeš použít jinou variantu, stejně tak pro zvuky nebo grafiku (vedle `Wayland` existuje populární "prastará" `X11`).
Důležité je, že ve světě otevřených technologií si každý může - a mnozí toho využívají - napsat vlastní variantu.

Operační systém je tedy opravdu systém, který má spoustu různých komponent, které mezi sebou spolupracují. 

Z toho vychází problémy se spoluprací programů mezi sebou a spousta více či méně funkčních kombinací. 
Proto existují tzv. distribuce, kde berou určitou množinu těchhle důležitých komponent a snaží se, aby všechny spolupracovaly. 
Sestavit takovou distribuci je hodně práce, která zaměstnává stovky až tisíce lidí.
Některé ty komponenty jsou standardnější než ostatní. 
Proto třeba má cenu naučit se Bash - najdeš ho totiž na jakémkoli systému.

### Grafická prostředí 

Existuje i víc, různě úspěšných, grafických prostředí. Nejpopulárnější jsou:

* Gnome
* KDE
* xfce4 (šetrná varianta, která dobře funguje na starších počítačích)
* Mate

Všechno jsou to projekty s komunitami lidí nadšených pro svůj konkrétní projekt. 
Grafické prostředí přináší s sebou několik základních aplikací: terminál, textový editor, prohlížeč souborů.
To, že máš na svém systému nějaké grafické prostředí, ti nebrání v používání aplikací z jiných.
Příklad: v Gnome najdeš textový editor, který se jmenuje Gedit. 
Někomu se ale víc líbí textový editor, který vyvíjí KDE: Kate. 
Můžeš si do systému doinstalovat Kate a používat ho místo Geditu. Je to jen na tobě a tvém vkusu :).

Gnome je velmi běžné grafické prostředí. Povíme si o něm něco víc.

Podívejme se do `Nastavení` (Settings). Je vždy dobré věnovat čas projití nastavení svého systému. 
Gnome se snaží mít ve svém nastavení víc než jen chování sebe sama. Vše je přehledně na jednom místě.

Mrkneme teď společně na `Nastavení  - Zařízení - Klávesnice` (`Settings - Devices - Keyboard`), kde najdeš přehled klávesových zkratek na svém počítači. 
Zde si můžeš jednoduše přiřadit vlastní zkratky určitým činnostem a přidat nové.
Když sjedeš úplně dolů v nastavení a klikneš na <kbd>+</kbd>, zobrazí se menu přidání nové zkratky.
Přidáme si zkratku na pouštění textového editoru Gedit. Pole *Command* vyžaduje bashový příkaz, který spouští danou aplikaci.

*Název*: Pusť gedit
*Command*: gedit
*Shortcut*: zmáčkneme vlastní klávesovou zkratku, třeba <kbd>CTRL</kbd> + <kbd>G</kbd>.

TODO: screenshot

Když teď zmáčkneš <kbd>CTRL</kbd> + <kbd>G</kbd>, měl by nastartovat textový editor.

Pozor, jméno spuštěného programu v nadpisu je v Gnome něco jiného než opravdové jméno programu.
Když pustíš "gedit", ukáže se v nadpisu "Textový editor", puštěný "nautilus" -> "Soubory". 
Nejsnadnější je pak vyhledat na internetu, jak se doopravdy jmenuje daný program.

Minule jsme si ukázali, jak vytvářet krátké bashové skripty na určité úlohy. 
Nyní už víš, jak si zkrátit cestu k provedení těch úloh tím, že namíříš na ně klávesové zkratky.


## Zajímavé malé příkazy

Ukážeme si dalších několik příkazů, které je dobré znát.
`which` - příkaz, který existuje z historických důvodů a neměl by se už používat, ale jelikož si spousta lidí na něj zvyklá, jeho oblíbenost neklesá.

Nicméně existuje novější alternativa. 
Type ukáže, jaký příkaz přesně se pustí, když napíšeš "krátké" jméno programu. Například:
```console
$ type git
git is /usr/bin/git
$ type firefox 
firefox is /usr/bin/firefox
$ type python3
python3 is /usr/bin/python3
```
Když zapneš virtuální prostředí Pythonu a spustíš příkaz ještě jednou, zjistíš, že je cesta k Pythonu jiná, než před chvílí.
```console
(venv)$ type python3
python3 is /home/user/bash/venv/bin/python3
```

`type` je lepší, protože pokud se ho zeptáš na něco, co není program, jako `cd`, vrátí smysluplnou odpověď, zatímco `which` - ne.
U `cd` se dozvíme, že se jedná o zabudovanou funkci v shellu. 
Proč není to program, ale funkce? `cd` totiž nemůže být příkaz. 
Když pouštím nějaký proces, nemůže on ovládat proces, který ho pustil (nesmí jít jakoby o úroveň výše).

```console
$ type cd
cd is a shell builtin
```

Procesy se vytváří tak, že se proces rozdvojí (forkem) a pak začne dělat něco jiného. 
Normální programy fungují tak, že běží `bash`, kterému v jednom momentu řekneš, že chceš spustit `firefox`. 
`bash` se rozdvojí, pustí `firefox`, ale `bash` jede dál. Následně `firefox` žije vlastním životem. 
Když si `firefox` změní aktuální adresář, klidně to může udělat. Problém s `cd` je, že my chceme změnit aktuální adresář Bashe. 
A kdyby `cd` byl příkaz, který Bash spouští podobně jako `firefox`, `cd` by nemohlo změnit stav rodičovského procesu `bash`.
Proto je `cd` výjimka, zabudovaná funkce Bashe a umí měnit jeho interní stav. 
```

bash ----+--- bash ----------+-----------------
         |                   |
         +--- firefox        +--- bash -- X
```
`bash` se rozdvojí, jedna ta větev se nahradí programem `firefox`

Zkus provést

```console
$ type ls
ls is aliased to `ls --color=auto'
```

`ls` je alias, což znamená, že když napíšeš `ls`, provede se příkaz `ls --color=auto`. 
Proč to tak je? Autoři příkazu `ls` nepřidali do svého programu barvičky ve standardním provedení, ale jako přepínač. 
Spoustě uživatelů barvičky vyhovují a radši by je měli po ruce vždy, aniž by si museli ošoupat klávesnici psaním `--color=auto` ke každému příkazu. 
Proto autoři linuxových distribucí často změní standardní nastavení programu a dají uživatelům k dispozici už barevný výstup.

Pomocí aliasu si můžeme vytvořit vlastní zkratku pro oblíbený program.
```console
$ alias vykricnik='echo !'
$ vykricnik 
!
```

Tohle ale bude fungovat jen v aktuálním shellu.
Když pak zadáš `bash`, tak tam už `vykricnik` nebude fungovat (protože v něm není nadefinovaný).

### Jak Bash spouští Bashové skripty?

Napíšeme si `skript.sh`
```
#! /bin/bash

cd ..
alias kocka=cat
promenna=123
```

A pustíme si ho:
```console
$ chmod +x skript.sh
$ ./skript.sh
```

Nic? Skript se provedl, ale vše, co udělal, si nechal pro sebe.
Když se teď pokusíš o zjištění, co je *kocka* nebo *promenna*, dostaneš chybu:
```console
$ type kocka
$ bash: type: kocka: not found
$ echo $promenna

```

Tím, že spustíš skript, se vytvoří nový proces Bashe, které má své vlákno a vlastní stav.
Když skript skončí, stav zanikne. Proto alias ani proměnná v tom skriptu nikdy nezmění stav zdrojového shellu.
Když spustíš jakýkoli program: ať už je to skript, nebo program, ten proces nemůže změnit stav shellu, ze kterého ho pouštíš.

Co, když to ale přesně chceš - pozměnit stav aktuálního shellu?
Například: vytvořit nový adresář a přepnout se do něj.

Napíšeme skript `mcd` s obsahem:
```
#! /bin/bash

mkdir $1
cd $1
```

Toto nebude fungovat, protože skript vytvoří nový proces, pustí `bash`, nový `bash` vytvoří nový adresář - to fungovat bude, ale pak se do nového adresáře přepne a skončí. 
Aktuální adresář toho nového Bashe je zbytečně změněný, nepoznáš to z úrovně Bashe, který skript pustil:
```

         mcd test
bash ----+----------------------------------------
         |                                  ^
         |   bash `mcd test`      cd test   .
         +--- O ------------------- O-------X
              |                 ^
              |                 .
              +--- mkdir test-- X
```

Aby ses o tom přesvědčil{{a}} na vlastní oči, připiš do skriptu `mcd` další řádek:

```
#! /bin/bash

mkdir $1
cd $1
pwd
```
Když ho teď spustíš, uvidíš, že ti `bash` řekne "byl jsem tam", ale cesta v promptu se nezmění, jsi pořád na stejném místě v adresářové struktuře.

```console
$ chmod +x mcd
$ ./mcd priklad
/home/user/bash/04/priklad
$
```

#### export
Udělejme jiný příklad.
Proměnná se ve výchozím stavu nekopíruje do podprocesu. Abychom toho docílili, je třeba použít `export`.

Vytvoř soubor: `print-promenna`, který vypíše obsah proměnně `promenna`
```
#! /bin/bash

echo $promenna
```

Když ho spustíš, nevypíše nic, protože takovou proměnnou nezná, není nadefinovaná. 
```console
$ chmod +x print-promenna
$ ./print-promenna

```

Když ji nadefinuješ ve stávajícím Bashi a spustíš skript, opět neuvidíš obsah proměnné, Bash v podprocesu totiž nedědí stav nadřazeného Bashe.
```console
$ echo $promenna

$ promenna=123
$ echo $promenna
123
$ ./print-promenna

```

A pro předání obsahu proměnné do podprocesu se přesně hodí příkaz `export`. 

```console
$ echo $promenna
$ ./print-promenna

$ promenna=123
$ export promenna
$ ./print-promenna
123
```

A protože proměnná je teď nastavená, že se bude exportovat, když ji nastavíš na něco jiného, nová hodnota se předá dál.

```console
$ promenna=abc
$ ./print-promenna
abc
```

Co je to ten `export`? Stejně jako `cd` je to vestavěná funkce Bashe. A taky stejně jako `cd` mění stav stávajícího Bashe, takže nemůže být zvláštním programem.

```console
$ type export
export is a shell builtin
``` 

Další věc, kterou můžeš dostat jako výsledek příkazu `type`, vedle opravdového programu, aliasu a zabudované funkce shellu, je  i obyčejná, tebou nadefinovaná funkce.

Příkaz na `mcd` můžeš taky napsat jako funkci, která pak bude vypadat takto:

```console
mcd () 
{
    mkdir -p $1 && cd $1
}
```

Taková funkce se skládá z jména (`mcd`), kulatých závorek a složených závorek. 
Mezi složené závorky napiš příkaz/y, které má funkce provést.

Je možné to napsat i do jediného řádku, ale pak je třeba dávat pozor, za příkazy totiž patří středníky, například:
```console
mcd () { mkdir -p $1 && cd $; }
```

I v případě shellových funkcí nadefinovaných přímo v příkazové řádce, platí pravidlo, že patří pouze aktuálnímu shellu. 
Nemůžeš je použit mimo něj, v jiném terminálu nebo v podprocesu.

#### source

Příkaz, který umožňuje spouštět daný skript/program v aktuálním bashovém procesu, se jmenuje `source`. Mimochodem, je to další zabudovaná funkce shellu.

> [note]
>Pokud jsi nastavoval{{a}} Python na linuxovém počítači dle pokynů začátečnického kurzu, vzpomeneš si jistě na aktivaci virtuálního prostředí pomocí příkazu `source venv/bin/activate`. Příkaz source mění PATH tvého systému.

Pro srovnání grafický výstup procesu. Pokud skript jen spustíš, vypadá to tak:
```

bash ----+----------------------------------------
         |                                   ^
         |   bash cd      alias    ...
         +--- O ------------O------ O-------X
```

`source` spustí skript v aktuálním shellu:

```

           bash cd       alias    ...
bash ------- O ------------O-------O-------X
```

Namísto `source` lze použít i `.`, v `man bash` zjistíš , že má `.` i `source` stejný popis.


Zajímavost:
`echo` je také zabudovaná funkce shellu, ale když se podíváš do systému, zjistíš, že máš i program, který se jmenuje `echo`. 
Dělají teměř to stejné, vypíšou na standardní výstup to, co dostanou jako argument.
`/usr/bin/echo abc`
vs.
`echo abc`


Na spouštění podprocesů existuje i zkratka, a to závorky. 
Vše, co je v kulatých závorkách, se spustí v podprocesu.
```console
$( cd test; pwd ); pwd
/home/user/bash/04/test
/home/user/bash/04
```

Grafické znázornění vypadá takto:
```

         (
bash ----+--------------------------------------- pwd ---
         |                                   ^
         |   bash    cd test      pwd   )    .
         +---------- O ------------O---------X
```


Občas chceme použít výstup jednoho programu jako argument druhého programu.
Zde je rozdíl mezi standardním vstupem do programu a argumentem programu.

Příklad - výstup jednoho programu je **vstup** dalšího
```console
$ pwd | cat
/home/karel
```

Mohli bychom ale chtít použít pro tento cíl program `echo` s **argumentem** "aktuální adresář", jak v příkladu níže.
```console
$ echo /home/karel
/home/karel
```

Jak předat výstup programu jako argument? Pomocí dolaru a závorek:

```console
$ echo $( pwd )
/home/karel
```
Bash vyhodnotí příkaz `pwd` a doplní místo textu v `$( pwd )` výstup z něho.


*Cvičení*
Co udělají tyto příkazy?
```console
$ echo $( ls )
$ cat $( ls )
```
`echo` vypíše jména všech souborů v daném adresáři, `cat` přečte obsah všeho, co dostane jako argument.

Na konci výstupu `cat` možná přečteš podobný řádek: `cat: nazev-slozky: Is a directory`. Co je to? Není pravděpodobné, že obsah nějakého souboru zahrnuje takový text. 
Záhadu odhalíš, pokud přesměruješ standardní výstup příkazu jinam, například do virtuálního koše:
```console
$ cat $( ls ) > /dev/null 
cat: test: Is a directory
cat: venv: Is a directory
```
To, co zbylo, je náš starý známý - standardní chybový výstup. `cat` neumí přečíst obsah složek, a proto zahlásí chybu na `stderr`.

Ukážeme si při této příležitosti ještě jednu věc. 
Výstup programu níže u tebe může vypadat trochu jinak, ale příklad níže dobře poslouží za ukázku. 
Všimni si, že se výstup na dvou posledních řádcích trochu změnil oproti příkladu výše. A proč?

```console
$ cat $( ls ) | head
#! /bin/bash
cat:
mkdir -p $1
testcd $1
pwd
: Is a directory
cat: venv: Is a directory
```
Oba výstupy, i standardní, i chybový, směřují na stejné místo - tvůj terminál. 
`cat` přesměruje svůj standardní výstup k příkazu `head`, a chybový vypíše do příkazové řádky. 
`head` vypíše standardní výstup do příkazové řádky. 
Počítač přepíná mezi jednotlivými procesy a výstupy smíchá dohromady. 
Proto chybový výstup je částečně mezi standardním.


Pokud místo dolaru napíšeš zobáček, místo toho, aby se použil výstup programu jako argument, použije se název souboru, kam ukládá svůj výstup `ls`. 
Je to speciální jméno souboru, které si `echo` může přečíst.

```console
$ echo <( ls )
/dev/fd/63
$ echo /dev/fd/63
/dev/fd/63
```
`cat` v souboru najde jeho obsah. 
```console
$ cat <( ls )
mcd
$ cat /dev/fd/63
mcd
```


### Porovnání dvou souborů - program `diff`

Ulož si dvě básničky - `basnicka.txt`
*Haló haló,
co se stalo? 
Kolo se mi polámalo!*

`basnicka2.txt`
*Haló haló,
co se stalo???
Kolo se mi polámalo!*

Pomocí programu `diff -U3` dostaneš hezky výpis, co se v obou souborech liší (podobný výpis znáš z gitu).
```console
$ diff -U3 basnicka.txt basnicka2.txt
--- basnicka.txt	2020-04-26 14:59:04.770053968 +0200
+++ basnicka2.txt	2020-04-26 14:59:18.118062312 +0200
@@ -1,4 +1,4 @@
 Haló haló,
-co se stalo? 
+co se stalo???
 
 Kolo se mi polámalo!
```

Některé programy dávají své výsledky jen na standardní výstup. 
Standardní výstup je jen jeden, co ale, kdybychom chtěli porovnat výstup dvou příkazů? 
Pokud znáš verzovací nástroj `git`, může tě třeba zajímat, jak vypadal konkrétní soubor před týdnem a před měsícem. 
Git má na to příkazy, ale ony vypisují výsledky na standardní výstup. 
Jistě, můžeš si výstupy přesměrovat do souboru (to už znáš), a následně použit `diff`. 
Můžeš taky porovnat přímo ty zvláštní soubory, do kterých programy, když provádějí příkazy, zapisuji. 
A příkaz vypadá takto:

```console
$ diff -U3 <( cat basnicka.txt ) <( cat basnicka2.txt )
--- basnicka.txt	2020-04-26 14:59:04.770053968 +0200
+++ basnicka2.txt	2020-04-26 14:59:18.118062312 +0200
@@ -1,4 +1,4 @@
 Haló haló,
-co se stalo? 
+co se stalo???
 
 Kolo se mi polámalo!
```

A jak to funguje z hlediska procesů?

```
$ diff -U3 <( cat 1 ) <( cat 2 )


bash ----+---+---+--------------------------------
         |   |   | 
         |   |   +---- diff -U3 p1 p2
         |   |                  ^   ^
         |   |                  |   |
         |   |              /...|.../
         |   +--- cat 2 > p2    |
         |                      |
         |             /......../
         |             |
         +--- cat 1 > p1
                  ....
```

Jiný příklad. Existuje program `seq`, který vypíše sekvenci čísel, kde koncové číslo je mu dáno jako argument. 

```console
$ seq 2
1
2
```

`seq` vypisuje výsledek na standardní výstup. 

Když chceme porovnat dva výpisy `seq`, např. `seq 10` a `seq 11`, které nemáme v sebou vytvořených souborech, přijde nám s pomocí `diff` mezi speciálními soubory na disku.

```console
$ diff -U3 <( seq 10 ) <( seq 11 )
--- /dev/fd/63	2020-04-26 17:10:02.129532287 +0200
+++ /dev/fd/62	2020-04-26 17:10:02.129532287 +0200
@@ -8,3 +8,4 @@
 8
 9
 10
+11
```

## Vlastní modifikace Bashe

Už víš, že program `source` spustí příkaz v aktuálním shellu. 
Při spouštění Bash vždy provede příkaz `source ~/.bashrc` a provede příkazy, které jsou v něm nadefinované.
Pokud tedy chceš, aby tvůj alias nebo funkce fungovala vždy, když spustíš nový terminál, ulož si jej do souboru `.bashrc`.
Tvůj `.bashrc` bude nejspíš vypadat trochu jinak. 
Jeho obsah je různý v různých operačních systémech.

```console
$ cat .bashrc 
# .bashrc

# Source global definitions
if [ -f /etc/bashrc ]; then
	. /etc/bashrc
fi
```

Když změníš tento soubor, změny se projeví až v dalších nově spuštěných shellech, ten již běžící to neovlivní.
Pro načtení změn do již spuštěného shellu použij známý příkaz `source ~/.bashrc`


## File, whoami a groups

Dobrý nástroj na zjištění, jakého typu je určitý soubor, je `file`.

```console
$ file basnicka.txt
basnicka.txt: UTF-8 text

$ file print-promenna.sh 
print-promenna.sh: Bourne-Again shell script, ASCII text executable

$ file test/
test: directory

$ file /usr/bin/git
/usr/bin/git: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 3.2.0, BuildID[sha1]=2727a12383c5b9179acb3a7bcd8f6b79997b1844, stripped
```

`file` dokáže dobře napovědět, co je obsahem souboru. 
Zejména se hodí, když je obsah souboru binární a nedá se číst pomocí `less` nebo `cat`.


`whoami` ti řekne, jaké je tvé uživatelské jméno na tomto počítači.
`groups` je podobný příkaz, který vypíše všechny skupiny, ke kterým v systému patříš, včetně skupiny tvého uživatele (jmenuje se stejně jako uživatelské jméno).

Skupiny určují práva uživatelů, kteří k ním patří. Například uživatel ve skupině `docker` smí používat program Docker. 

Když je uživatel ve skupině `wheel`, může si zažádat o administrátorská práva (příkazem `sudo`) a dostane je. 

Unixové systémy jsou postaveny tak, že může k ním zároveň přistupovat několik uživatelů. 
Můžou mít každý svůj uživatelský profil a vzdáleně se k počítači připojovat. 
K přepínání mezi účty je příkaz `su`. 
Pokud se chci přepnout na účet root (správce systému), napíšu `su root`. 
Zeptá se to na heslo roota, a pokud ho správně zadáš, změníš identitu. 
Pravděpodobně ale neznáš heslo root uživatele, a to je dobře. 
Práva správce by se měla používat v opodstatněných případech. 
A pro tyto případy je příkaz `sudo`. 
Pokud potřebuješ použít administrátorská oprávnění, zadáš před příkaz `sudo` a následné **své** heslo. 
Pokud patříš ke správné skupině,  tento příkaz ti umožní provádět administrátorské příkazy.


O skupinách si podrobněji povíme jindy. 



===
Odbočkou se na konci naťukla tato složitost bashe, zde vynechávám z popisu. Ukázka spočívala v projití `man bash`. Mně to přijde celkem mimo i na druhý poslech... Zařaďte a popište dle vůle.
```console
b=$(basename $1)
${b%%.git}
```

Více info: `man bash`, hledat '${param'. Těch transformací tam je celý kopec...
