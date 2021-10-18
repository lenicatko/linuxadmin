XXX: from notes/02:

        ## Proměnné prostředí

        Pomocí příkazu `env` vypíšeš všechny proměnné prostředí, které existují v Bashi. Je jich tam opravdu hodně a určují, jak se chová nejen Bash, ale i programy které spouští.
        Ukážeme si několik z nich.

        - `SHELL` - říká, jak máš zrovna shell. Bude to nejspíš `/bin/bash`
        - `HOSTNAME` - jméno počítače
        - `LC_*něco*` - nastavení formátování různých řetězců
        - `PWD` - aktuální adresář
        - `HOME` - domovský adresář
        - `VISUAL` - editor, který preferuješ
        - `PS1` - do této proměnné se dá zapsat, čím nám bude začínat každý řádek v Bashi. Můžeš přepsat např. jen na `PS1='$ '` nebo použít zvláštní sekvence jako `\w`, což znamená aktuální adresář.
        - `PS2` je „pokračovací“ prompt (pro zadávání víceřádkového příkazu)

        Obsah každé proměnné můžeš vypísat pomocí příkazu `echo $<proměnná>`: 

        ```console
        $ echo $PS1
        ```

        Existují také speciální proměnné, které jsou pojmenované různými znaky. Třeba:
        ```console
        $ echo $?
        ```
        vypíše návratovou hodnotu posledního příkazu.

        Příkazy v Unixu vrací hodnoty, podle nichž většinou poznáš, jestli příkaz skončil v pořádku, nebo chybou.
        - Pokud je v `$0` hodnota `0`, pak proběhl příkaz v pořádku.
        - Hodnota jiná než `0` značí, příkaz skončil chybou

        Příklad:
        ```console
        $ ls unicorn.dat 
        unicorn.dat
        $ echo $?
        0

        $ ls jednorozec.dat
        ls: cannot access 'jednorozec.dat': No such file or directory
        $ echo $?
        2
        ```
        Na rozdíl od Pythonu Bash není tak striktní, pokud jde o chyby.
        Nepovedený příkaz ho nezastaví.
        Je proto dobré si občas kontrolovat, že příkaz proběhl korektně.

        Proměnná `$$` obsahuje číslo aktuálního shellu. Když otevřeš nový terminál s novým Bashem, uvidíš jiné číslo.

        Podobně jako v Pythonu se jména proměnných v Bashi můžou skládat z písmen, čísel a podtržítek. 
        Bash umí jednoduše skládat slova dohromady a přidat tak řetězec před nebo za obsah proměnné:

        ```console
        $ jmeno=minotaur
        $ echo $jmeno.abc
        minotaur.abc
        $ echo my$jmeno
        myminotaur
        ``` 

        Kdybys ale chtěl{{a}} dát proměnnou doprostřed do nějakého většího slova, tímto způsobem to nepůjde.
        Můžeš ale použít složené závorky:

        ```console
        $ echo my${jmeno}.abc
        myminotaur.abc
        ```

        Obecně je `$jmeno` zkratka pro `${jmeno}`; varianta se závorkami bude fungovat ve více případech.


        ## PS2

        Když napíšeš do příkazové řádky víceřádkový příkaz (např. v uvozovkách),
        výzva se změní.
        Bude nejspíš vypadat zcela jinak; poslední znak většinou bude `>` místo `$`.
        Například:

        ```console
        $ echo "jeden řádek
        > druhý řádek
        > třetí řádek"
        jeden řádek
        druhý řádek
        třetí řádek
        ```

        Tuto variantu výzvy nastavíš proměnnou `PS2`.


# Proměnné Bashe a proměnné prostředí

Už víš, že Bash používá *proměnné*.
Asi nejčastěji se používají vrámci cyklu `for`:

```console
$ cd ~/Dokumenty/data-shell/creatures/
$ for jmeno in *.dat;
> do
>     head -n2 $jmeno | tail -n1
> done
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

Spousta proměnných v Bashi je součást takzvaného *prostředí*,
které používá jak samotný Bash, tak i programy, které spouští.

Jedna z takových proměnných je `LANG` – aktuální jazyk.
U mě je nastaven na češtinu, takže Bash i programy v něm spuštěné
vypisují hlášky v češtině:

```console
$ echo $LANG
cs_CZ.UTF-8
$ exbho
bash: exbho: Příkaz nebyl nenalezen...
$ cat neexistujuci
cat: neexistujuci: Adresář nebo soubor neexistuje
```

Můžu ho ale nastavit i na angličtinu.
Proměnnou můžeš nastavit pomocí příkazu
<code><var>jméno</var>=<var>hodnota</var></code>,
kde před a za rovnítkem *nesmí být mezera*:

```console
$ LANG=en_US.UTF-8
$ exbho
bash: exbho: command not found...
$ cat neexistujuci
cat: neexistujuci: No such file or directory
```

> [note]
> Pokud máš anglický systém, je možné, že pouhé nastavení `LANG=cs_CZ.UTF-8`
> ti jazyk nepřepne – příslušné překlady musí být na systému nainstalovány,
> aby se daly použít.

Aktuální prostředí můžeš spravovat z Pythonu, kde je přístupné jako slovník `os.environ`:

```python
import os

for key, value in os.environ.items():
    print(f'{key}={value}')
```

To samé dělá příkaz `env`:

```console
$ env | head
SHELL=/bin/bash
SESSION_MANAGER=local/unix:@/tmp/.ICE-unix/1742,unix/unix:/tmp/.ICE-unix/1742
COLORTERM=truecolor
HISTCONTROL=ignoredups
XDG_MENU_PREFIX=gnome-
HISTSIZE=1000
HOSTNAME=localhost.localdomain
SSH_AUTH_SOCK=/run/user/1000/keyring/ssh
XMODIFIERS=@im=ibus
DESKTOP_SESSION=gnome-xorg
```

```console
$ env | grep ^LANG=
LANG=cs_CZ.UTF-8
```

> [note]
> Příkaz `env` toho dělá víc; Pythonní program simuluje
> jen `env` bez argumentů a přepínačů.


## Exportování

Proměnné Bashe se ale do prostředí nedostávají automaticky.

Tento příkaz nastaví proměnnou `jmeno`.
Pak můžeš pomocí `env` zjistit hodnotu proměnné *prostředí*,
která se posílá programům jako `env`;
`echo $jmeno` pak vypíše proměnnou Bashe,
který ji doplňuje v argumentech:

```console
$ jmeno=minotaur.dat
$ env | grep jmeno
$ echo $jmeno
minotaur.dat
```

Ačkoli je v Bashi jméno nastavené, programu `env` se neposlalo.

Proměnné v Bashi mají určité *příznaky* (angl. *attributes*/*properties*).
Nejdůležitější z nich je příznak *export*, který nastavíš příkazem `export`.

```console
$ export jmeno
$ env | grep jmeno
jmeno=minotaur.dat
$ echo $jmeno
minotaur.dat
```

Teď Bash předá proměnnou `jmeno` příkazům, které z něj spustíš.

Příkaz `export` nekopíruje hodnotu proměnné do prostředí, ale říká Bashi,
že má tuto proměnnou předat vždy, když se spouští nový příkaz.
Když tedy proměnnou nastavíš na něco jiného,
změna se projeví v dalším příkazu i bez dalšího `export`:

```console
$ jmeno=unicorn.dat
$ env | grep jmeno
jmeno=unicorn.dat
$ echo $jmeno
unicorn.dat
```

Příznak můžeš vypnout pomocí `export -n`:

```console
$ export -n jmeno
$ env | grep jmeno
$ echo $jmeno
unicorn.dat
```

A taky můžeš nastavit příznak i hodnotu najednou:

```console
$ export jmeno=basilisk.dat
$ env | grep jmeno
jmeno=basilisk.dat
$ echo $jmeno
basilisk.dat
```


> [note] Další příznaky
> Proměnné můžou mít i další příznaky, i když většinou nejsou zas tak užitečné.
> Například pomocí `declare -u velke` můžeš Bashi říct, že v proměnné `velke`
> můžou být jen velká písmenka:
> ```console
> $ declare -u velke
> $ velke=Ahoj
> $ echo $velke
> AHOJ
> ```
> Více příznaků najdeš v nápovědě příkazu `export`.


## Barvičky pro `ls`

Jedna z proměnných, které máš (na Fedoře) v Bashi automaticky exportované,
se jmenuje `LC_COLORS`.
Využívá ji příkaz `ls` k nastavení barevnosti.

Zaběhni do nějakého adresáře, kde je mix adresářů a normálních souborů:

```console
$ cd ~/Dokumenty/data-shell/data
```

A zkus si co dělá `ls` ve výchozím nastavení:

```console
$ ls
ahoj.txt         animal-counts  archiv    morse.txt  planets.txt  sunspot.txt
amino-acids.txt  animals.txt    elements  pdb        salmon.txt
```

Potom nastav proměnnou `LS_COLORS` na prázdný řetězec a zkus `ls` znovu.
Příkaz `ls` si teď vybere jiný způsob obarvení:

```console
$ LS_COLORS=
$ ls
```

Když znáš formát (do čehož teď nebudeme zabíhat),
můžeš si barvičky nastavit podle sebe:

```console
$ LS_COLORS='di=38;1;31'
$ ls
```

> [note]
> Kolem hodnoty jsou uvozovky, aby Bash znak `;` nebral jako oddělovač příkazů.

Proměnnou prostředí můžeš nastavit i pro jeden jediný příkaz,
a to tak, že nastavení (<code><var>jméno</var>=<var>hodnota</var></code>)
připíšeš před jméno příkazu:

```console
$ LS_COLORS='di=38;1;33' ls
$ ls
```

Tím se nastaví pouze proměnná prostředí – nikoli proměnná Bashe.

```console
$ pozdrav=ahoj env | grep pozdrav
pozdrav=ahoj
$ echo $pozdrav

$ pozdrav=ahoj echo $pozdrav

```

Stejně jako `LS_COLORS` se dají použít všechny ostatní proměnné, které
ovlivňují chování programů, např.
* už zmíněná `LANG`,
* `FLASK_APP` (pro ty, kdo znají framework Flask),
* `PYTHONVERBOSE` – při nastavení na `1` vypisuje Python např. detaily všech
  naimportovaných modulů, nebo
* `PAGER` a `EDITOR` – viz níže.


## Jméno v závorkách

Bash nahrazuje všechny výskyty <code>$<var>jméno</var></code> obsahem dané
proměnné.
Dělá to nejen v samostatných argumentech, ale i v jejich částech:

```console
$ komparativ=lepší
$ echo nej$komparativ
nejlepší
```

Občas, když tohle budeš tohle chtít použít, budeš potřebovat Bashi říct,
co je ještě jméno proměnné.
Pak můžeš jméno uzavřít do kudrnatých závorek,
<code>${<var>jméno</var>}</code>:

```console
$ majitel=Petr
$ echo $majitelovo   # proměnná jménem "majitelovo" neexistuje
$ echo ${majitel}ovo
Petrovo
```

Fuguje to vždy; <code>$<var>jméno</var></code> bez závorek je jen zkratka.

```console
$ echo ${LANG}
cs_CZ.UTF-8
```

Proměnné Bash nahrazuje i ve jménech programů, takže můžeš napsat:

```console
$ zpracuj=head
$ $zpracuj unicorn.dat
COMMON NAME: unicorn
CLASSIFICATION: equus monoceros
UPDATED: 1738-11-24
AGCCGGGTCG
CTTTACCTTA
AAGCCGAGGG
GGGTGGTACG
CCGAACATAA
ACGCTTTAAC
GTCCCTCCAG
$ zpracuj='wc -l'
$ $zpracuj unicorn.dat
163 unicorn.dat
$ zpracuj=less
$ $zpracuj unicorn.dat
```


## Rolovátko

Víš, proč se třeba `man` nebo `git diff` chovají a ovládají stejně jako `less`?
Proto, že samy spustí `less` a pošlou do něho rourou text,
který chtějí zobrazit.

  {{ figure(
    img=static('man-less.svg'),
    alt='Diagram spouštění `less` z `man`',
  ) }}

Podobné programy většinou respektují proměnnou `PAGER`,
kterou můžeš vybrat program, který se použije místo `less`.
Zkus si `more`, starší verzi `less` která umí <kbd>Enter</kbd>,
<kbd>Mezerník</kbd> a <kbd>q</kbd>, ale neumí se vracet (<kbd>↑</kbd>):

```console
$ PAGER=more man bash
```

Nebo rolování vypni úplně pomocí `cat`:

```console
$ PAGER=cat man bash
```
Nebo jakýkoli jiný filtr – nikdo ti nezakazuje zadávat nesmysly:

```console
$ PAGER=wc man bash
$ PAGER=tac man bash
```
