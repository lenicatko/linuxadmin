# Programy, funkce a aliasy

Když pracuju s počítačem, docela často potřebuju vytvořit adresář
a rovnou se do něj přepnout:

```console
$ mkdir novy-projekt
$ cd novy-projekt
```

Štve mě ale, že musím psát dva příkazy a dvakrát vypisovat jméno adresáře.
Co takhle si to zautomatizovat – vytvořit příkaz
<code>mcd <var>novy-projekt</vat></code>, který udělá oba kroky najednou?

> [note]
> Dvakát vypisovat jméno adresáře bych nemusel: zkratka
> <kbd>Alt</kbd>+<kbd>.</kbd> doplní poslední slovo minulého příkazu.
> (Jestli ti to nefunguje, zkus ten druhý <kbd>Alt</kbd>.)
> Ale stejně na tohle chci mít krátkou zkratku!

Vytvoř si následující skript a ulož si ho jako `mcd` (bez přípony):

```bash
#! /bin/bash

echo Vytvářím $1 >&2
mkdir -p $1
cd $1
```

Pak ho označ za spustitelný, spusť, a zkontroluj že se nový adresář vytvořil:

```console
$ chmod +x mcd
$ ./mcd novy-adresar
$ ls
```

Přepnutí (`cd`) ještě úplně nefunguje.
Spravíme to později. Zatím se podívejme, co tento skript dělá:

* `#! /bin/bash` říká systému, že tento skript je napsaný v Bashi; pro jeho
  vykonání se tedy bude spouštět Bash.
* Za `$1` Bash doplní první argument skriptu – v našem případě jméno adresáře,
  který pomocí příkazu `mcd` chceš vytvořit.
* Přesměrování `>&2` říká, že std. výstup se má přesměrovat na std. chybový výstup;
  informační hláška z `echo` tedy půjde na chybový výstup.
  Není to součást výstupu programu.
* `chmod +x mcd` nastavuje právo spouštět soubor `mcd` jako příkaz
* Příkaz je potřeba spustit pomocí `./mcd` (a ne jen `mcd`): pro příkazy bez
  daného adresáře Bash hledá program v `$PATH`, kde tento příkaz ještě není.

Skript se spustí, `echo` vypíše, `mkdir` vytvoří adresář – ale `cd` nezabralo!
Čím to?

Vzpomeň si, jak Bash pouští příkazy.


  {{ figure(
    img=static('forking-mcd.svg'),
    alt='Diagram spouštění jednotlivých podprocesů',
  ) }}

K tomu je potřeba dodat, že proces *nemůže měnit*
proměnné prostředí, aktuální adresář nebo otevřené soubory *jiných procesů*.

Když Bash proces *spouští*, může mu prostředí nastavit.
(Dělá to řed `exec`, takže mění *svoje* prostředí.)
Ale jakmile proces běží, nemůže měnit svůj „mateřský“ proces.

Příkaz `cd` ve skriptu tudíž změní aktuální adresář – ale jen v Bashi, který
obsluhuje náš skript.
Aktuální adresář původního Bash zůstane nezměněný.

No jo, ale jak může fungovat příkaz `cd`?
Jaktože změní adresář Bashe, který ho pouští?

Inu, `cd` je zvláštní.
Není to program, který běží ve vlastním procesu.

Pojďme se podívat, jaké příkazy vlastně Bash umí spouštět.


## Normální příkazy

To, co se spustí, když zadáš určitý příkaz, ti řekne příkaz `type`.
Většina základních příkazů jsou programy uložené na disku:

```console
$ type cat
cat je /usr/bin/cat
$ type man
man je /usr/bin/man
$ type mkdir
mkdir je /usr/bin/mkdir
```

> [note]
> Někdy se můžeš setkat s příkazem `which`, který dělá víceméně to samé,
> ale není k dispozici na tolika systémech. Je lepší se naučit používat `type`.

Někdy můžeš dostat informaci o tom, že je soubor „zahashován“ – to pro nás
znamená to samé co příklady výše.
(Znamená to že Bash tenhle příkaz už jednou našel v `$PATH` a nemusí ho
hledat znovu, takže ho může spouštět trošku rychleji.)

```console
$ echo|cat

$ type cat
cat je zahashován (/usr/bin/cat)
```

Tyhle soubory opravdu existují, ač nejsou psány pro lidské oči:

```console
$ ls /usr/bin/cat
/usr/bin/cat
$ less /usr/bin/cat
"/usr/bin/cat" may be a binary file.  See it anyway? y
```

Existují ale i výjimky, které pro lidské oči jsou:

```console
$ type pip
pip je /usr/bin/pip
$ less /usr/bin/pip
```

## Zabudované příkazy

Kromě toho existují i zabudované příkazy shellu.
To nejsou programy, pro které Bash vytváří nový proces.
Proto můžou měnit stav Bashe – proměnné prostředí, aktuální adresář nebo
(teoreticky) otevřené soubory, Nebo ho třeba ukončit:

```console
$ type export
export je součást shellu
$ type cd
cd je součást shellu
$ type exit
exit je součást shellu
```

Některé příkazy jako `echo` a `pwd` jsou zabudované v Bashi jen proto,
aby se rychleji spouštěly; mají i „opravdový“ program v `/usr/bin`,
který se dá zavolat:

```console
$ type echo
echo je součást shellu
$ ls /usr/bin/echo
/usr/bin/echo
$ /usr/bin/echo Ahoj ahoj!
Ahoj ahoj!
```

S přepínačem `-a` ti  `type` vypíše všechna místa, kde se daný program dá
najít; používá se ale jen to první:

```console
$ type -a echo
echo je součást shellu
echo je /usr/bin/echo
```

> [note]
> Někdy se zabudovaný příkaz od programu v `/usr/bin` liší – porovnej
> `echo --help` s `/usr/bin/echo --help`.
> Tohle není moc skvělá vlastnost.

Takže teď už znáš trik, kterým `cd` dokáže měnit adresář Bashe,
který ho spustil.
Má protekci, potvůrka!

Jak to ale zařídíme pro příkaz `mcd`?


### Source

Další zabudovaný příkaz je `source`:

```console
$ type source
source je součást shellu
```

Source otevře daný soubor a příkazy v něm spustí *v aktuálním Bashovém procesu*,
tak jako bys je napsal{{a}} na klávesnici.
Neboli – přesně to co hledáme pro `mcd`!

```console
$ source ./mcd test
Vytvářím test
$ pwd
.../test
$ cd ..
```

Jestli používáš Linux pro Python, pravděpodobně tenhle příkaz znáš.
Skript pro aktivaci virtuálního prostředí, `venv/bin/activate`,
potřebuje měnit proměnné `PS1` (a tudíž text výzvy) a `PATH`
(a tudíž význam příkazů `python` a `pip`) – a to v aktuálním shellu.
Má tedy stejný problém jako `mcd`.

A protože je `source` dlouhé slovo, má i zkratku: tečku.
Ano, tečka je taky Bashový příkaz.

```console
$ . ./mcd dalsi-test
Vytvářím dalsi-test
$ pwd
.../dalsi-test
$ cd ..
```

> [note]
> Ten příkaz je: tečka, mezera, tečka, lomítko, `m`, `c`, `d`.
>
> V materiálech pro Python používám delší `source` protože … je vidět.

Problém `source` i tečky je ale ten, že je vždycky musíš napsat.
I kdyby byl skript `mcd` ve správném adresáři, musel by se pouštět
pomocí <code>. mcd <var>novy-adresar</vat></code>.
A to nechci, to je pořád moc dlouhé. Pojďme hledat dál.



## Klíčová slova

Kromě programů v souborech a zabudovaných příkazů existují ještě další tři
druhy příkazů.
Jeden jsou klíčová slova, která mají jinou syntax než příkazy: potřebují
strukurovanější zápis, ne jen argumenty a přepínače:

```console
$ type for
for je klíčové slovo shellu
$ type while
while je klíčové slovo shellu
$ type do
do je klíčové slovo shellu
$ type if
if je klíčové slovo shellu
```


## Aliasy

Zajímavější jsou *aliasy*, zkratky.
Jeden alias který má Fedora ve výchozím nastavení, je `ll`:

```console
$ type ll
ls je alias na „ls -l --color=auto“
```

Kdykoli zavoláš příkaz <code>ll <var>argumenty</vat></code>, spustí
se reálně <code>ls -l --color=auto <var>argumenty</vat></code>.
Pro zopakování: přepínač `-l` zapíná „dlouhý“ výstup:

```console
$ ll ~
celkem 40
drwxrwxr-x. 2 petr petr 4096 26. říj 14.56 bin
drwxr-xr-x. 2 petr petr 4096 26. říj 16.36 dev
drwxr-xr-x. 5 petr petr 4096 26. říj 14.30 Dokumenty
drwxr-xr-x. 2 petr petr 4096  6. říj 16.24 Hudba
drwxr-xr-x. 2 petr petr 4096 17. říj 15.06 Obrázky
drwxr-xr-x. 2 petr petr 4096  6. říj 16.24 Plocha
drwxr-xr-x. 2 petr petr 4096  6. říj 18.13 Stažené
drwxr-xr-x. 2 petr petr 4096  6. říj 16.24 Šablony
drwxr-xr-x. 2 petr petr 4096  6. říj 16.24 Veřejné
drwxr-xr-x. 2 petr petr 4096  6. říj 16.24 Videa
```

Příkaz `ll` Bash nahradil za `ls -l --color=auto` a zbytek argumentů (`~`)
ponechal.
Reálně se tedy pustil příkaz `ls -l --color=auto ~`.
Zkontroluj, že s ním dostaneš stejný výsledek.

Co je ale zajímavější?
Samotný `ls` je alias:

```console
$ type ls
ls je alias na „ls --color=auto“
```

Kdykoli zavoláš příkaz <code>ls <var>argumenty</vat></code>, spustí
se reálně <code>ls --color=auto <var>argumenty</vat></code>.
Jednou použitý alias se ale už nenahrazuje, místo toho Bash hledá `ls` dál.
Kde? To ti poví `type -a`!

```console
$ type -a ls
ls je alias na „ls --color=auto“
ls je /usr/bin/ls
```

A skutečně, když pustíš `/usr/bin/ls ~`, žádné barvičky neuvidíš.
Ale s `/usr/bin/ls --color=auto ~` ano.

Nové aliasy si můžeš definovat pomocí příkazu `alias`.
Co třeba vytvořit příkaz `pozor`, který funguje jeko `echo` ale na
začátek řádku dá varovný štítek `POZOR:`?

```console
$ alias pozor='echo POZOR:'
$ pozor vysoké napětí
POZOR: vysoké napětí
$ pozor zlý pes
POZOR: zlý pes
```

Alias se dá zrušit pomocí `unalias`:

```console
$ unalias pozor
$ pozor pohov
bash: pozor: Příkaz nebyl nenalezen...
```

Spousta lidí aliasy používá na jednoduché zkratky, např.
`alias gc='git commit'`, `alias gst='git status'`, `alias ga='gitk --all'`.

Nevýhoda aliasů spočívá v tom, že fungují jen na začátku příkazu.
Můžou tedy zkrátit `echo POZOR: pes` na `pozor pes`, ale neumí už zkrátit
`mkdir novy-projekt; cd novy-projekt` na `mcd novy-projekt`.
Budeme se tedy muset podívat na poslední druh příkazů.


## Funkce shellu

Poslední druh příkazů je funkce.
Jedna z mála předpřipravených funkcí je `quote`:

```console
$ type quote
quote je funkce
quote ()
{
    local quoted=${1//\'/\'\\\'\'};
    printf "'%s'" "$quoted"
}
```

Podobně jako u aliasů příkaz `type` u funkcí uvádí i jejich definici.
Přesně takhle se funkce definuje: jméno, prázdné závorky, a tělo funkce –
příkazy – v kudrnatých závorkách:

```console
$ ahoj() {
>    echo ahoj
>    echo světe
> }
$ ahoj
ahoj
světe
```

Na rozdíl od funkcí v Pythonu jsou závorky `()` v definici vždycky prázdné.
Každá funkce bere neomezený počet argumentů, které Bash uloží do proměnných
`$1`, `$2`, `$3`, atd.
Jejich počet je v proměnné `$#`; všechny dohromady jsou v `$@`.

Jak tedy nadefinovat příkaz `mcd`?

```console
$ mcd () {
>     mkdir -p $1
>     cd $1
> }
```

Tenkhle příkaz funguje – zkus si ho!

Má ale jednu mouchu: když zapomeneš zadat argument, `mkdir -p` selže, ale
`cd` bez argumentů tě přepne do domovského adresáře.
Proto je dobré příkaz `cd` provést jenom tehdy, když se povede `mkdir`.

```console
$ mcd () {
>     mkdir -p $1 && cd $1
> }
```

A to je on – vytoužený příkaz na vytvoření adresáře a přepnutí do něj.
Ufff. Stálo to za to?

Zvlášť když příkaz zmizí hned když vypneš Bash?


## bashrc

Je to tak, téhle ságy ještě není konec.

Programy v souborech jsou uložené celkem bezpečně, ale aliasy i shellové funkce
platí jenom pro shell ve kterém jsou nadefinovány.
Po vypnutí shellu zmizí.

Dá se to samozřejmě obejít tak, že funkci vždy nadefinuješ znovu.
Můžeš si dokonce vytvořit skript, do kterého definice dáš a vždy ho na začátku
práce s Bashem pustíš (pomocí `source`, aby se funkce nadefinovaly ve tvém
procesu).

Ale nejlepší je dát do souboru `~/.bashrc` (tedy skrytého souboru `.bashrc`
ve tvém domovském adresáři).
Ten totiž Bash načte na začátku sám!

Dej si tedy definici `mcd` na konec souboru `~/.bashrc`
a zapni nový Bash (buď novém termnálu, nebo třeba příkazem `exec bash`.

Teď bude `mcd` k dispozici vždy!


## Jednodušší příkazy

Ne všechny zkratky jsou takhle složité.
Většinou si vystačíš s aliasem nebo spustitelným skriptem v `$PATH`,
tedy často v adresáři `~/bin`.
Skripty můžou být psané v Bashi (`#! /usr/bin/bash`), v Pythonu
(`#! /usr/bin/python`) nebo jakémkoli jiném jazyce.

Někteří lidé své zkratky sdílejí; autor tohoto textu je má k nahlédnutí
[na GitHubu](https://github.com/encukou/bin).
Samozřejmě bez záruky.


## A další?

A co další náležitosti shellových příkazů?
Manuálová stránka?
Přepínače – aspoň `--help`?

To už je nad rámec tohoto kurzu.
Přeci jen se teď chceme naučit Linuxové systémy ovládat, ne vytvářet :)

Mimochodem, příkaz `mcd` už existoval – a dělal něco úplně jiného!
Příkaz `type -a mcd` ti ukáže i `/usr/bin/mcd`, ale co hůř: `man mcd`
ukáže nápovědu úplně jiného příkazu.
Pokud chceš vytvářet funkční systémy, je dobré tohle zkontrolovat ještě dřív
než začneš vytvářet vlastní příkaz.
