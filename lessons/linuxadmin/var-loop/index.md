# Proměnné a cyklus

Přesuň se do složky `data-shell/creatures`. 
Jsou v ní tři soubory:
 
```console
$ ls
basilisk.dat  minotaur.dat  unicorn.dat
```

Podívej se, jaké informace tyto soubory obsahují. 
Na začátku každého z nich je nějaká informace o zvířeti které popisují,
a následuje část DNA sekvence.
Tvůj úkol je zjistit, jaké informace se nachází na prvních třech řádcích
každého souboru.
Pro tento úkol se může hodit například příkaz `head`.
(Všimni si, co `head` dělá když mu dáš ke zpracování více souborů.)

```console
$ head -n3 *.dat
==> basilisk.dat <==
COMMON NAME: basilisk
CLASSIFICATION: basiliscus vulgaris
UPDATED: 1745-05-02

==> minotaur.dat <==
COMMON NAME: minotaur
CLASSIFICATION: bos hominus
UPDATED: 1765-02-17

==> unicorn.dat <==
COMMON NAME: unicorn
CLASSIFICATION: equus monoceros
UPDATED: 1738-11-24
```

Představ si, že pro další zpracování potřebuješ jen *druhý* řádek každého
z těchto souborů.
Pro jeden souboru to půjde jen s příkazy, které už znáš.
Zkus to například s baziliškem: napiš příkaz, který vypíše druhý řádek
souboru `basilisk.dat`.

{% filter solution %}

Můžeš na to jít postupně: nejprve vybereš dva první řádky,
pak z těch dvou jeden poslední.

```console
$ head -n2 basilisk.dat | tail -n1
CLASSIFICATION: basiliscus vulgaris
```

{% endfilter %}

Teď stačí pokaždé přepsat jméno souboru a máš to! To je ale trochu otravné.
Jde to zjednodušit? Samozřejmě.

## Proměnné

V Pythonu bychom pro takový opakovaný kus kódu
použili proměnnou a napsali cyklus.
To se teď naučíme dělat v Bashi. Začneme proměnnými.

Přiřaď do proměnné `jmeno` slovo `minotaur.dat`:
```console
$ jmeno=minotaur.dat
```
Pozor na to, že kolem rovnítka nesmí být mezery!

Když pak chceš použit obsah této proměnné a napíšeš `echo jmeno`, co to vypíše?

```console
$ echo jmeno
jmeno
```

Jak vidíš, není to správná cesta.
Abys Bashi dal{{a}} najevo že chceš skutečně vypsat obsah proměnné `jmeno`,
je potřeba něco navíc.
To něco je speciální operátor: dolar před jménem proměnné.

```console
$ echo $jmeno
minotaur.dat
``` 

Na rozdíl od Pythonu se v Bashi proměnné nedosazují automaticky.
Je na to vždy potřeba použít `$`.

> [note]
> Doporučuju si k `$` poznamenat, že znamená “dosadit hodnotu” proměnné,
> ne “použít proměnnou”.
>
> Při *nastavení* proměnné se totiž dolar nepíše: v příkazu
> `jmeno=minotaur.dat` dosazovat hodnotu nechceš.
> 
> (Dolar je taky poslední znak výzvy, *promptu*; to je zcela jiný význam
> stejného znaku.)

Proměnnou můžeš použít jako název souboru.
Následujícím příkazem získáš druhý řádek ze souboru, který je uložený v proměnné `jmeno`:

```console
$ head -n2 $jmeno | tail -n1
CLASSIFICATION: bos hominus
```

Podobně jako v Pythonu můžeš novým přiřazením přepsat obsah proměnné a zpřístupnit tak pro další příkazy jiná data pod stejným jménem. 

```console
$ jmeno=unicorn.dat
$ head -n2 $jmeno | tail -n1
CLASSIFICATION: equus monocer

$ jmeno=basilisk.dat
$ head -n2 $jmeno | tail -n1
CLASSIFICATION: basiliscus vulgaris
```

Do proměnných se hodí dávat i příkazy, které pak podle potřeby můžeš přepsat něčím jiným.
Představ si, že normálně používáš program `less` pro čtení souborů.
Jednoho dne ale budeš psát instrukce, které by měly fungovat i na počítači starém 30 let,
kde program `less` není. Je tam ale archaický program `more`.

V takových případech můžeš jméno programu brát z proměnné:

Příklad
```console
$ moje_ctecka=less
$ $moje_ctecka unicorn.dat
```

```console
$ moje_ctecka=more
$ $moje_ctecka unicorn.dat
```

Všimni si, jak se výstup pokaždé mění.


## Cyklus for

Pojďme teď proměnné využít v cyklech.

Cyklus `for` se v Bashi zapisuje podobně jako v Pythonu, jen potřebuje trochu
víc psaní a odřádkovávání.

```console
$ for jmeno in a b c
> do
>     echo $jmeno
> done
a
b
c
```

V prvním řádku nechceš dosadit hodnotu proměnné, ale jen do ní přiřazuješ.
Proto zde nepoužiješ znak `$`. `a b c` jsou slova, která se do této proměnné budou postupně přiřazovat.
Po zmáčknutí <kbd>Enter</kbd> příkaz pokračuje (poznáš to podle změny výzvy na `>`: Bash zatím nezačal nic vykonávat).

Následuje klíčové slovo `do`, kterým začíná blok příkazů které se pro každou hodnotu proměnné `jmeno` provedou.
My chceme vypsat hodnotu proměnné, čili `echo $jmeno`.
Na dalším řádku následuje klíčové slovo `done`, které ukončuje `do` a tedy celý cyklus.

Na rozdíl od Pythonu není nutné tělo cyklu odsadit: blok kódu se uvozuje pomocí
slov `do` a `done`; mezery na začátku řádku nemají význam.
Hezké odsazení ale zvětšuje přehlednost kódu.
V materiálech je proto píšu.

Když za `done` zmáčkneš <kbd>Enter</kbd>, vypíše se postupně `a`, `b` a `c`.

Když pak dáš šipku nahoru, místo víceřádkového příkazu ti ho Bash vypíše v jednořádkové formě.
Jednotlivé příkazy budou odděleny středníky. 
Středník a nový řádek (<kbd>Enter</kbd>) v Bashi většinou znamenají totéž.
(Ale nejsou *přesně* totéž – všimni si že za `do` středník není.)

```console
$ for jmeno in a b c; do echo $jmeno; done
a
b
c
```

A teď už víš dost na to, abys vypsal{{a}} druhý řádek každého souboru `*.dat`!
Zkus to udělat.

{% filter solution %}
Jak na to? Dobré je si úkol rozdělit na dva kroky: cyklus přes soubory a samotný výpis.

Nejprve si ve `for` cyklu vypiš obsah proměnné pomocí `echo`, aby sis byl{{a}} jist{{ gnd('ý', 'á') }}, že pracuješ se správnou hodnotou. 
Když si zvykneš na výpomoc programu `echo`, ušetříš si v budoucnu spoustu starostí :) 

```console
$ for jmeno in *.dat
> do
>     echo $jmeno
> done
basilisk.dat
minotaur.dat
unicorn.dat
```
Teď už víš, že rámec cyklu `for` cyklu je napsán správně.
A tak můžeš místo `echo` provést už známý příkaz s `head` a `tail`:

```console
$ for jmeno in *.dat
> do
>     head -n 2 $jmeno | tail -n 1
> done
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

{% endfilter %}

### Kontrola příkazu…

Kdybys chtěl{{a}} udělat něco drastičtějšího než jen vypsat části souborů,
doporučuju přidat ještě prostřední krok: použij `echo` pro vypsání
samotného příkazu a zkontroluj, že se vypíšou opravdu příkazy
chceš provést.

Příklad:
```console
$ for jmeno in *.dat
> do
>     echo rm $jmeno 
> done
rm basilisk.dat
rm minotaur.dat
rm unicorn.dat
```
Po kontrole můžeš smazat `echo` a příkazy provést.

Při přidávání `echo` si ale dej pozor na přesměrování a jiné “potrubní”
speciální znaky.
Následující příkaz nedělá jenom kontrolu:

```console
$ for jmeno in *.dat
> do
>     echo cat $jmeno >> vsechno.txt
> done
```

Místo něj by bylo lepší napsat:

```console
$ for jmeno in *.dat
> do
>     echo "cat $jmeno >> vsechno.txt"
> done
```

### …vnořené cykly…

Cykly se dají i vnořovat, podobně jako v Pythonu.
Následující příkaz kontroluje připravení adresářové struktury pro
měření vlastností několika molekul za několika různých teplot:

```console
$ for molekula in cubane ethane methane
> do
>     for teplota in 25 30 37 40
>     do
>         echo mkdir $molekula-$teplota
>     done
> done
```


### …a něco o chybách

Tady je příklad časté chyby, kdy počítač sice udělá to co chceš,
ale zároveň vypíše chybovou hlášku `head: cannot open 'ls' for reading: No such file or directory`.

```console
$ for jmeno in ls *.dat
> do
>     head -n 2 $jmeno | tail -n 1
> done
head: cannot open 'ls' for reading: No such file or directory
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```
Proč příkaz `head` říká, že neumí otevřít soubor `ls`? Co se tady stalo?
Odpověď je jednoduchá: Bash interpretuje část `ls *.dat` jako *seznam slov*, nikoli výstup příkazu `ls`.
Do proměnné `jmeno` tedy přiřadí postupně slova `ls`, pak `basilisk.dat`, pak `minotaur.dat` atd.
Příkaz `head -n 2 $jmeno` si pak u `ls` stěžuje, že takový soubor neexistuje.

Můžeš taky narazit na jinou chybu. Už víš, že `for` cyklus prochází slova, ne příkazy. V příkladu níže jsme zapomněli v těle cyklu uvést `echo`. Co se stane, když ho spustíš?

```console
$ for jmeno in cat dog mouse
> do
>     $jmeno
> done

```
To, co je pro první řádek cyklu slovo, se v těle používá jako Bashový příkaz. 
Zavolal{{a}} jsi tedy příkaz `cat` bez argumentů, který čeká na vstup z klávesnice. 
Ukončíš-li ho klávesovou zkratkou <kbd>Ctrl</kbd>+<kbd>D</kbd>,
objeví se následně chybový výpis o tom, že příkazy `dog` a `mouse` nejsou Bashi známé.


## While?

V Bashi existuje taky cyklus `while`. Seznámíme se s ním ale až později.
