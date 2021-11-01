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
které používá jak samotný Bash, tak i programy které spouští.

Jedna z takových proměnných je `LANG` – aktuální jazyk.
U mě je nastaven na češtinu, takže programy vypisují hlášky v češtině
(pokud jsou překlady k dispozici):

```console
$ echo $LANG
cs_CZ.UTF-8
$ cat neexistujuci
cat: neexistujuci: Adresář nebo soubor neexistuje
```

Můžu ho ale nastavit i na angličtinu.
Jak už víš, proměnnou můžeš nastavit pomocí příkazu
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

Všechny proměnné prostředí ti vypíše příkaz `env` (bez argumentů):

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

Tyto proměnné má k dispozici jakýkoli program spuštěný z Bashe.
Třeba z Pythonu je přístupné jako slovník `os.environ`:

```python
import os

for key, value in os.environ.items():
    print(f'{key}={value}')
```


## Exportování

Proměnné Bashe se ale do prostředí nedostávají automaticky.

Následující příkaz nastaví proměnnou `jmeno`.
Pak pomocí `env` zjistí hodnotu proměnné *prostředí*,
která se posílá programům (jako `env` a `python`) – tam `jmeno` není.
Příkaz `echo $jmeno` ale obsah proměnné vypíše.

```console
$ jmeno=minotaur.dat
$ env | grep jmeno
$ echo $jmeno
minotaur.dat
```

Ačkoli je v Bashi `jmeno` nastavené a doplňuje se při použití dolaru,
programu `env` se neposlalo.
Když ale to samé vyzkoušíš s `LANG`, zjistíš že tahle proměnná se progrramu
`env` poslala.

Proč?

Proměnné v Bashi mají určité *příznaky* (angl. *attributes*/*properties*).
Nejdůležitější z nich je příznak *export*, který říká, jestli je proměnná
součástí prostředí, nebo jestli jde jen o „lokální“ proměnnoiu Bashe.
Nastavíš ho příkazem `export`.

```console
$ export jmeno
$ env | grep jmeno
jmeno=minotaur.dat
$ echo $jmeno
minotaur.dat
```

Teď Bash předá proměnnou `jmeno` příkazům, které z něj spustíš.

Příkaz `export` nekopíruje hodnotu proměnné do prostředí, ale říká Bashi,
že má tuto proměnnou předat každému novému příkazu.
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

Jedna z proměnných které máš (na Fedoře) v Bashi automaticky exportované
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

Potom nastav proměnnou `LS_COLORS` na středník a zkus `ls` znovu.
(Středník je pro Bash speciální, je ho třeba dát do uvozovek.)
Příkaz `ls` si teď vybere jiný způsob obarvení:

```console
$ LS_COLORS=';'
$ ls
```

Když znáš formát zadávání barev (do čehož teď nebudeme zabíhat),
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

A některé proměnné Bash nejen používá, ale i nastavuje:

* `HOSTNAME`: jméno počítače
* `LC_*něco*`: nastavení formátování různých řetězců
* `PWD`: aktuální adresář
* `HOME`: domovský adresář

Některé z nich nastavuje speciálním způsobem, takže 


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
Pak můžeš jméno uzavřít do „kudrnatých“ závorek,
<code>${<var>jméno</var>}</code>:

```console
$ majitel=Petr
$ echo $majitelovo jablko   # proměnná jménem "majitelovo" neexistuje
jablko
$ echo ${majitel}ovo jablko
Petrovo jablko
```

Fuguje to vždy; <code>$<var>jméno</var></code> bez závorek je jen zkratka.

```console
$ echo ${LANG}
cs_CZ.UTF-8
```


## Proměnná jména programů

Proměnné Bash nahrazuje i ve jménech programů, takže můžeš napsat:

```console
$ zpracuj=head
$ $zpracuj sunspot.txt
* Sunspot data collected by Robin McQuinn from *)
* http://sidc.oma.be/html/sunspot.html         *)

* Month: 1749 01 *) 58
* Month: 1749 02 *) 63
* Month: 1749 03 *) 70
* Month: 1749 04 *) 56
* Month: 1749 05 *) 85
* Month: 1749 06 *) 84
* Month: 1749 07 *) 95
$ zpracuj='wc -l'
$ $zpracuj sunspot.txt 
3080 sunspot.txt
$ zpracuj=less
$ $zpracuj sunspot.txt 
```

Tohle se reálně používá.

V proměnné `EDITOR` spousta programů hledá editor, který spustí když
chtějí aby uživatel změnil nějaký soubor – například když se Git ptá na
popis změn.
Ve Fedoře je nastaven na `/usr/bin/nano`, ale v `~/.bashrc` si můžeš nastavit
jiný.
(Pozor ale na to, že nastavení jako `git config` nebo proměnná `GIT_EDITOR`
tuhle hodnotu „přebíjí“.)

V proměnné `PAGER` zase můžeš nastavit „skrolovátko“.
Opět – spousta programů se tím pak bude řídit.
Tahle proměnná není „od výroby“ nastavená; když je prázdná tak programy
typicky použijí `less`.


## Výzva

Některé proměnné prostředí využívá samotný Bash.
Jedna z nejzákladnějších je `PS1`, která ovládá výzvu
(angl. *prompt*; `PS` je zkratka *prompt string*).
Bývá nastavena na docela složitou hodnotu:

```console
[\u@\h \W]\$
```

Bash v ní nahrazuje sekvence se zpětnými lomítky.
(Co které znamená, to můžeš případně dohledat v dokumentaci.)

Když chceš výzvu zkrátit, nastav `PS1` na něco kratšího.
Doporučuju výzvu ukončit dolarem (aby bylo jasné že to je výzva) a
mezerou (která musí být v uvozovkách, aby ji Bash bral jako součást hodnoty).

```console
[user@fedora data]$ PS1='$ '
$ 
```


Když napíšeš do příkazové řádky víceřádkový příkaz (např. v uvozovkách),
výzva se změní. Poslední znak většinou bude `>` místo `$`.
Například:

```plain
$ echo "jeden řádek
> druhý řádek
> třetí řádek"
jeden řádek
druhý řádek
třetí řádek
```

Tuto variantu výzvy nastavíš pomocí proměnné `PS2`.

