# Spustitelné skripty

Pamatuješ si ještě, jak jsme v cyklu klasifikovali různá stvoření?

```console
$ cd Dokumenty/data-shell/creatures/
$ for x in *.dat
> do
>     head -n 2 $x | tail -n 1
> done
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

Psát takhle dlouhé příkazy v Bashi není moc pohodlné.
Pojďme si tento „program“ uložit do souboru.

> [note] Skript vs. program
> Programům v Bashi se často říká „skripty“ (angl. *script*: předpis, scénář).
> Není to o jazyku – skript můžeš napsat i v Pythonu.
> Skripty jsou jednoduché, jednoúčelové programy, které tolik neřeší
> např. ošetřování chyb nebo komunikaci s uživatelem.
> Často volají existující nástroje a „spojují“ je dohromady.
>
> Není ale žádná pevná hranice mezi „skriptem“ a „programem“.


## Spustitelné skripty

Zkus si založit v editoru soubor `klasifikace`:

```console
$ nano klasifikace
```

> [note] Bez přípony
> Programy v Linuxu typicky nemají příponu: nemáme `cat.exe` ale jen `cat`.
>
> Bashové skripty se dají nastavit i tak, aby byly spustitelné přímo jako
> příkazy, a proto ani `klasifikace` příponu nemá.
>
> (Když je přípona pro shellový skript potřeba, používá se většinou `.sh`.)

Do souboru zapiš cyklus, tak jako bys ho psal{{a}} v Bashi:

```console
for x in *.dat
do
    head -n 2 $x | tail -n 1
done
```

Následně můžeš skript spustit. Musíš být ve složce, která obsahuje tento soubor. 
Stejně jako u Pythonu spustíš skript pomocí názvu programu, který ho má přečíst (tedy `bash`), a názvu souboru, který se má spustit.
```console
$ bash klasifikace 
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```


## Ze skriptu příkaz

Teď uděláme ze skriptu příkaz.
To znamená že zařídíme, aby nebylo potřeba program volat z tohoto adresáře
a s `bash` na začátku.
Zkus si, že to teď nefunguje:

```console
$ klasifikace
bash: klasifikace: command not found
```

So s tím?
Na začátek příkazu patří takzvaný *shebang*, který říká jakým programem
se soubor bude spouštět.
V našem případě chceme skript spouštět pomocí Bashe.
Na první řádek souboru klasifikace proto připiš dva speciální znaky `#!` a jméno souboru, ve kterém je uložený program Bash:

```bash
#! /bin/bash
```

> [note]
> Shebang je instrukce pro systém, že `klasifikace` se bude spouštět Bashem.
> Pro Bash samotný je to komentář – začíná `#`.

Další věc, kterou musíš udělat, je nastavení *práv* souboru (angl. *permissions*).
To jsou příznaky, které zobrazuje `ls -l` v levém sloupci:

```console
$ ls -l klasifikace
-rw-r--r-- 1 user users 58 Oct 15 19:12 klasifikace
```

Existují tři základní druhy oprávnění, pro které se zobrazí buď písmenko (pokud je oprávnění povoleno) nebo pomlčka (pokud je odepřeno):

- r - právo pro čtení souboru (**r**ead)
- w - právo na zápis souboru (**w**rite)
- x - právo na spuštění souboru (e**x**ecute)

Tyto práva se opakují 3x za sebou, jednou pro vlastníka souboru (zde `-rw`, vlastník `user` tedy může soubor `klasifikace` číst a psát), podruhé pro skupinu (*group*, zde `-r-`, skupina `users` tedy může jen číst), a potřetí pro všechny ostatní uživatele (zde opět můžou jen číst).

Příkaz musíš přidat právo na spuštění, což zajistí příkaz `chmod +x`. (Přesné chování `chmod` si vysvětlíme později.)

```console
$ chmod +x klasifikace
$ ls -l klasifikace 
-rwxr-xr-x 1 user user 58 Oct 15 19:12 klasifikace
```
Po přidání *shebangu* a nastavení práv stačí zadát jméno souboru, a on se spustí v Bashi!
Jen je ho zatím potřeba zadat s plnou cestou, tedy s `./` na začátku:

```console
$ ./klasifikace 
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

### Argumenty

Náš skript `klasifikace` zpracovává vždy všechny soubory `.dat` v daném adresáři. 
Co kdybys chtěl{{a}} použít tento skript trochu jinak: aby ukázal data jen z daného souboru? 
Vytvořme program `klasifikuj` tak, aby fungovalo třeba:

```console
$ ./klasifikuj minotaur.dat
```
Vytvoř soubor `klasifikuj`, dej do něj tento obsah a přidej právo na spuštění:
```bash
#! /bin/bash
head -n2 "$1" | tail -n1
```

`"$1"` je speciální proměnná označující první argument příkazové řádky, tedy u `./klasifikuj minotaur.dat` ono `minotaur.dat`.

Proč je v uvozovkách? 
Protože kdybys zadal{{a}} jako první argument text s mezerami, uložily by se jako víc argumentů. 
Když je sekvence takto v uvozovkách, celý první argument se použije správně.
Platí obecné pravidlo: pokud používáš v Bashi argument ve kterém může být mezera,
napiš ho pro jistotu do uvozovek.

Kdybys chtěl{{a}} použít další argumenty, použij `"$2"`, `"$3"` atd.
A `$0`, nultý argument, je jméno skriptu který se spouští.

Existuje ještě `$@`, což znamená všechny argumenty. 
Uprav původní skript `klasifikace` tak, aby necyklil po `*.dat`, ale `"$@"`.
Teď ho můžeš spouštět jak pro jeden soubor tak i pro seznam souborů.

```console
$ cat klasifikace 
for jmeno in "$@"
do
    head -n2 $jmeno | tail -n1
done
$ ./klasifikace unicorn.dat
CLASSIFICATION: equus monoceros
$ ./klasifikace *.dat
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

> [note]
> Zajímavé na "$@" je to, že i když je v uvozovkách, více „slov“.


### Spustitelné skripty - nejen pro Bash
Shebang můžeš nastavovat nejen pro Bashové skripty. 
Ukážeme si to na příkladě Pythonu. 
Vytvoř soubor `nahoda`, který bude obsahovat tento kód:
 
```python
#! /usr/bin/python3

import random
print(random.randrange(100))
```
Souboru přidej právo na spuštění a můžeš ho spouštět stejně jako *klasfikaci* z příkladu výše. 
*Shebang* zajistí, že se program spustí Pythonem – tak jako kdybys napsal{{a}}
<code>python3 <var>jméno_souboru</var></code>.

```console
$ chmod +x nahoda
$ ./nahoda
34
$ ./nahoda
73
```

Ukázka výše funguje pro všechny programy, jen musíš vědět, kde přesně na disku
jsou (často v `/usr/bin/` nebo `/bin/`; do detailů půjdeme později).

Ukažme si to ještě s `cat`.
Do souboru `README` napiš:

```plain
#! /usr/bin/cat
 
Tady jsou informace.
```
Přidej práva na spouštění a spusť ho:

```text
$ chmod +x README
$ ./README
#! /usr/bin/cat

Tady jsou informace.
```
Je to stejné, jako kdybys spustil{{a}} `cat ./README`!


### Generický shebang

Občas budeš psát skripty, které nejen na tvém počítači, ale i jinde.
Třeba na systémech na kterých Bash chybí – třeba z bezpečnostních důvodů, nebo
se šetří místem na disku.
Bude tam ale nainstalovaný jiný shell.

Standard POSIX, kterým se víceméně řídí všechny „Unixové“ systémy
(Linux, BSD, Unix, Solaris, macOS, atd.), specifikuje shell jménem `sh`.
Do shebangu tedy napíšeš `#! /bin/sh`.

Z tohoto shellu Bash vychází; to co funguje v `sh` funguje v naprosté většině
případů i v Bashi.
Naopak to ale neplatí: Bash má „vychytávky“ navíc.
Skript pro `sh` je občas potřeba zjednodušit.

Často také uvidíš skripty, ve kterých za `#!` není mezera ale rovnou lomítko.
To na chování nic nemění, jen se ušetří jeden znak.
Nejčastější shebang pro shellový skript tedy vypadá takto:

```text
#!/bin/sh
```


### PATH

Možná si teď kladeš otázku, jak se zbavit `./` na začátku příkazu
U příkazů jako `cat` stačí zadat jen jméno, ale když zadáš pouze:

```console
$ klasifikace *.dat
klasifikace: command not found
```

Bash si postěžuje, že takový příkaz nezná.
Když totiž Bash nedostane přesné umístění souboru s příkazem (což `./klasifikace` je),
tak se totiž dívá jen do určitých adresářů.
Které to jsou to určuje proměnná `PATH`.
Vypiš si její obsah:

```console
$ echo $PATH
/home/user/.local/bin:/home/user/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/var/lib/snapd/snap/bin
```

Uvidíš spoustu adresářů oddělených dvojtečkou.
Měl{{a}} bys v mimo jiné vidět `/usr/bin`, kde jsou systémové příkazy jako `cat` a `echo`.
A taky je tam <code>/home/<var>tvoje_jmeno</var>/bin</code>, kam můžeš dávat vlastní příkazy.

Jak už víš, domovský adresář se v Bashi zkracuje jako `~`, takže místo <code>/home/<var>tvoje_jmeno</var>/bin</code>
můžeš v příkazech psát `~/bin`.

Jak tedy „nainstalovat“ vlastní příkaz?
Pokud adresář `~/bin/` neexistuje, vytvoř ho.
A pak do něj zkopíruj soubor `klasifikace`:

```console
$ mkdir ~/bin
$ cp klasifikace ~/bin/
```

Pak můžeš skript spouštět jako příkaz, bez `./` na začátku:

```console
$ klasifikace *.dat
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

Mimochodem, v domovském adresáři je skrytý skript `~/.bashrc`, který se spustí vždy, když pustíš Bash. 
Je to soubor, ve kterém je možné upravit si prostředí, např. přidat cesty do `PATH`, upravit `PS1`, nadefinovat si vlastní funkce a spoustu dalších.
