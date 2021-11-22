
# Poznámky – Subshell


> [warning]
> Toto jsou zatím jen rozdrobené poznámky, ne materiály pro (samo)studium.

## Na začátek trocha opakování

Už znáš příkaz `type`, který ukáže jaký příkaz přesně se pustí když
napíšeš „krátké“ jméno programu. Například:

```console
$ type git
git is /usr/bin/git
$ type firefox 
firefox is /usr/bin/firefox
$ type python3
python3 is /usr/bin/python3
```

Když zapneš virtuální prostředí Pythonu a spustíš příkaz ještě jednou, zjistíš
že je cesta k Pythonu jiná než před chvílí:

```console
(venv)$ type python3
python3 is /home/user/bash/venv/bin/python3
```

U `cd` ti `type` řekne, že se jedná o zabudovanou funkci v shellu.
Proč to není program, ale funkce? `cd` totiž nemůže být příkaz.
Když pouštím nějaký proces, nemůže on ovládat proces, který ho pustil (nesmí jít jakoby o úroveň výše).

```console
$ type cd
cd is a shell builtin
```

Procesy se vytváří tak, že se proces rozdvojí (forkem) a pak začne dělat něco jiného. 
Normální programy fungují tak, že běží `bash`, kterému v jednom momentu řekneš, že chceš spustit `firefox`. 
`bash` se rozdvojí, pustí `firefox`, ale `bash` jede dál. Následně `firefox` žije vlastním životem. 


```
bash ----+--- bash ----------
         |
         +--- firefox
```

Když si `firefox` změní aktuální adresář, klidně to může udělat. Problém s `cd` je, že my chceme změnit aktuální adresář Bashe. 
A kdyby `cd` byl příkaz, který Bash spouští podobně jako `firefox`, `cd` by nemohlo změnit stav rodičovského procesu `bash`.
Proto je `cd` výjimka, zabudovaná funkce Bashe a umí měnit jeho interní stav.



#### export

Další opakování.
Proměnná se ve výchozím stavu nekopíruje do podprocesu. Abychom toho docílili, je třeba použít `export`.

Vytvoř soubor: `echo-promenna`, který vypíše obsah proměnně `promenna`
```
#! /bin/bash

echo $promenna
```

Když ho spustíš, nevypíše nic, protože takovou proměnnou nezná, není nadefinovaná. 
```console
$ chmod +x echo-promenna
$ ./echo-promenna

```

Když ji nadefinuješ ve stávajícím Bashi a spustíš skript, opět neuvidíš obsah proměnné, Bash v podprocesu totiž nedědí stav nadřazeného Bashe.
```console
$ echo $promenna

$ promenna=123
$ echo $promenna
123
$ ./echo-promenna

```

A pro předání obsahu proměnné do podprocesu se přesně hodí příkaz `export`. 

```console
$ echo $promenna
$ ./echo-promenna

$ promenna=123
$ export promenna
$ ./echo-promenna
123
```

A protože proměnná je teď nastavená, že se bude exportovat, když ji nastavíš na něco jiného, nová hodnota se předá dál.

```console
$ promenna=abc
$ ./echo-promenna
abc
```

Co je to ten `export`? Stejně jako `cd` je to vestavěná funkce Bashe. A taky stejně jako `cd` mění stav stávajícího Bashe, takže nemůže být zvláštním programem.

```console
$ type export
export is a shell builtin
``` 

Další věc, kterou můžeš dostat jako výsledek příkazu `type`, vedle opravdového programu, aliasu a zabudované funkce shellu, je  i obyčejná, tebou nadefinovaná funkce.

Příkaz na `mcd` můžeš napsat jako funkci, která pak bude vypadat takto:

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

Tohle je celkem zbytečné, protože aktuální adresář je k dispozici např. i
v proměnné `PWD`:

```console
$ echo $PWD
/home/karel
```

Ale u výstupu programu `locate` to tak není.
Vem si třeba následující příkazy:

```console
$ locate bark.ogg
/usr/share/sounds/gnome/default/alerts/bark.ogg
$ mplayer /usr/share/sounds/gnome/default/alerts/bark.ogg
```

Místo nich můžeš psát:

```console
$ mplayer $(locate bark.ogg)
```

*Cvičení*
Co udělají tyto příkazy?

```console
$ echo $( ls )
$ cat $( ls )
```
`echo` vypíše jména všech souborů v daném adresáři.
`cat` přečte obsah všeho, co dostane jako argument – a to včetně obsahu případných netextových („binárních“) souborů.

Tenhle `cat` taky může narazit na chyby, které odhalíš,
pokud přesměruješ standardní výstup do virtuálního koše:

```console
$ cat $( ls ) > /dev/null 
cat: test: Is a directory
cat: venv: Is a directory
```
To, co zbylo, je náš starý známý - standardní chybový výstup. `cat` neumí přečíst obsah adresářů, a proto zahlásí chybu na `stderr`.

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
Oba výstupy, i standardní i chybový, směřují na stejné místo - tvůj terminál. 
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
Git má na to příkazy, ale ty vypisují výsledky na standardní výstup. 
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

V závorkách mohou být libovolné Bashové příkazy:

```console
$ diff -U3 <( seq 10 ) <( seq 5; seq 7 10 )
--- /dev/fd/63	2021-11-22 16:37:33.842564914 +0100
+++ /dev/fd/62	2021-11-22 16:37:33.843564919 +0100
@@ -3,7 +3,6 @@
 3
 4
 5
-6
 7
 8
 9
```


## Transformace proměnných

Odbočkou se na konci naťukla tato složitost bashe, transformace proměnných.
Viz `man bash`, záznamy jako `${parameter:-word}`.
