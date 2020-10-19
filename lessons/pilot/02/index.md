
# 2. Linuxová administrace - Bash (2/2)

Dnes se opět ponoříme do Bashe. 
Už znáš jeho několik základních příkazů a operací. 
Nově se podíváme, jak vypadají Bashové proměnné, cykly, a jak se píšou skripty.


## Na severním Pacifiku

### Standardní výstup
V materiálech často narazíš na pojem _standardní výstup_, (angl. *standard output stream*, zkráceně *stdout*). Jedná se o místo, kam program vypisuje své výsledky.
Většina programů vypisuje normálně výsledky svých operací přímo do příkazové řádky. Standardní výstup je teminál.
Můžeš ale výstup přesměrovat, např. pomocí `>` do souboru nebo pomocí `|` do jiného programu. 

> [note]
> Některé příkazy přesměrování detekují a chovají se trochu jinak, když je použiješ samostatně, a jinak, když jejich výstup přesměruješ. Srovnej:
> 
> ```console
> $ ls *.txt
> NENE01729A.txt  NENE01751B.txt  NENE01971Z.txt  NENE02040A.txt  NENE02043B.txt
> NENE01729B.txt  NENE01812A.txt  NENE01978A.txt  NENE02040B.txt
> 
> $ ls *.txt | cat
> NENE01729A.txt
> NENE01729B.txt
> NENE01736A.txt
> NENE01751A.txt
> NENE01751B.txt
> NENE01812A.txt
> NENE01843A.txt
> NENE01843B.txt
> NENE01971Z.txt
> ```
> 
> Výstup do terminálu bývá přehlednější pro lidi (a často je i obarvený).
> Přesměrovaný výstup bývá jednodušší na strojové zpracování.


### Operace na souborech
Zapni terminál a přepni se do adresáře `data-shell/north-pacific-gyre/2012-07-03`. 

Pomocí `ls` se podívej, co v adresáři je:

```console
$ ls
goodiff         NENE01736A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040Z.txt
goostats        NENE01751A.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01729A.txt  NENE01751B.txt  NENE01971Z.txt  NENE02040A.txt  NENE02043B.txt
NENE01729B.txt  NENE01812A.txt  NENE01978A.txt  NENE02040B.txt
```

Úkol: Kolik mají řádků datové soubory s `.txt` na konci?

{% filter solution %}
```console
$ wc -l *.txt
```
{% endfilter %}

Všechny by měly mít 300 řádků. Když se ale podíváš na výstup, zjistíš, že tomu tak není. 
Představ si, že máš těchto souborů k ověření několik tisíc a procházení *očima* nebude možné. 
Jak zjistíš, jestli všechny tyto soubory mají opravdu 300 řádků?

{% filter solution %}

`wc -l *.txt | sort | head` - uvidíš, jestli soubory na začátku jsou OK
`wc -l *.txt | sort | tail` - uvidíš, jestli soubory na konci jsou OK

Podívej se na výstup prvního z těch příkazů:

```
  240 NENE02018B.txt
  300 NENE01729A.txt
  300 NENE01729B.txt
  300 NENE01736A.txt
  300 NENE01751A.txt
  300 NENE01751B.txt
  300 NENE01812A.txt
  300 NENE01843A.txt
  300 NENE01843B.txt
  300 NENE01971Z.txt
```
  
{% endfilter %}

Soubory končí většinou na písmenko `A` nebo `B,` ale je zde jeden končící `Z`. Spočítej, kolik je souborů končících `Z`.

Odpověď
```console
$ ls *Z.txt | wc -l
2
```

Už podle názvů tyhle soubory nevypadají na to, že by je někdo vytvořil ručně. 
Můžeš bezpečně předpokládat, že jsou výstupem nějakého programu na zpracování vzorků.

Podívej se do několika z nich (např. pomocí příkazu `less`), abys zjistil{{a}} jak zhruba vypadají informace v nich obsažené. 

Obsahem je relativní výskyt 300 bílkovin v různých hloubkách oceánu.
`A` a `B` jsou dvě hloubky, na nichž bylo prováděno měření; `Z` znamená, že se hloubku z nějakého důvodu nepodařilo zjistit. 
Kratší soubor je způsobený restarem počítače během měření.
Nevíme, zda takový soubor bude použitelný pro další zpracování. 


### Kouzla Bashe

Už znáš hvězdičku a otazník, díky nimž můžeš vytvořit masky souborů.
Existují i další „finty“, jak jména souborů filtrovat. Jednu z nich si teď ukážeme.
Když dáš do hranatých závorek výčet znaků, které chceš hledat, např. `[AB]`, celý výraz v závorkách se nahradí za jedno písmenko – buďto `A` nebo `B`.
Můžeš tak odfiltrovat soubory, které končí na `Z`:

```console
$ ls *[AB].txt
NENE01729A.txt  NENE01751A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040B.txt
NENE01729B.txt  NENE01751B.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01736A.txt  NENE01812A.txt  NENE01978A.txt  NENE02040A.txt  NENE02043B.txt
```
V tomto případě dostaneš stejný výstup, když napíšeš příkaz:

```console
$ ls *A.txt *B.txt
```

Pozor ale na to, kdyby neexistoval žádný soubor, kterému taková maska odpovídá. 
Představ si, že chceš vypsat všechny soubory, které mají na konci `A`, `B` nebo `C`:

```console
$ ls *[ABC].txt
NENE01729A.txt  NENE01751A.txt  NENE01843A.txt  NENE01978B.txt  NENE02040B.txt
NENE01729B.txt  NENE01751B.txt  NENE01843B.txt  NENE02018B.txt  NENE02043A.txt
NENE01736A.txt  NENE01812A.txt  NENE01978A.txt  NENE02040A.txt  NENE02043B.txt

$ ls *A.txt *B.txt *C.txt
ls: cannot access '*C.txt': No such file or directory
 NENE01729A.txt   NENE01751B.txt   NENE01978A.txt   NENE02040B.txt
 NENE01729B.txt   NENE01812A.txt   NENE01978B.txt   NENE02043A.txt
 NENE01736A.txt   NENE01843A.txt   NENE02018B.txt   NENE02043B.txt
 NENE01751A.txt   NENE01843B.txt   NENE02040A.txt
```
Program `ls` si postěžuje, že takové soubory nezná. 

Zkus si, co v těchto případech dostává příkaz `ls` od Bashe.
To můžeš zjistit tak, že místo `ls` zadáš `echo`:

```console
$ echo *[ABC].txt
$ echo *A.txt *B.txt *C.txt
```

## Proměnné
Přesuň se do složky `data-shell/creatures`. 
Jsou v ní tři soubory:
 
```console
$ ls
basilisk.dat  minotaur.dat  unicorn.dat
```

Podívej se, jaké informace tyto soubory obsahují. 
Na začátku každého z nich je nějaká informace o zvířeti, které popisují, a následuje část DNA sekvence.
Tvůj úkol je zjistit, jaké informace se nachází u všech těchto souborech na prvních třech řádcích.
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

Představ si, že pro další zpracování potřebuješ jen druhý řádek každého z těch souboru.
U jednoho souboru to půjde jen s příkazy, které už znáš.
Zkus to například s baziliškem.

{% filter solution %}

Můžeš na to jít postupně: nejprve přečteš dva první řádky, pak z těch dvou jeden poslední.

```console
$ head -n2 basilisk.dat | tail -n1
CLASSIFICATION: basiliscus vulgaris
```

{% endfilter %}

Teď stačí pokaždé přepsat jméno souboru a máš to! To je ale trochu otravné.
Jde to zjednodušit? Samozřejmě.

V Pythonu bychom pro takový kus kódu, který se opakuje, použili proměnnou a napsali cyklus. To se teď naučíme dělat v Bashi.

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

Jak vidíš, není to správná cesta. Je potřeba něco navíc, abys Bashi dal{{a}} najevo, že chceš skutečně vypsat obsah proměnné `jmeno`.
Tuto službu nám zajistí speciální operátor: dolar před jménem proměnné.

```console
$ echo $jmeno
minotaur.dat
``` 

Na rozdíl od Pythonu se v Bashi nepoužívá proměnná automaticky.
Když chceš dosadit hodnotu proměnné, musíš použít `$`.

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
> [note]
> Pozor, při přiřazování nepiš `$`!
> Operátor `$` v Bashi *dosazuje hodnotu proměnné*.
> Když s proměnnou pracuješ jinak (třeba do ní přiřazuješ), nemá `$` u jména co dělat.
> 
> (Dolar je taky poslední znak *promptu*; to je zcela jiný význam stejného znaku.)

Do proměnných se hodí dávat i příkazy, které pak podle potřeby můžeš přepsat něčím jiným. 
Představ si, že máš skript, který používá program `less` pro čtení souborů. 
Jednoho dne ale budeš muset tento skript spustit na počítači starém 30 let, který žádný `less` nemá. Má ale archaický program `more`. 
Takový skript můžeš dopředu nachystat, aby jméno programu bral z proměnné.

Příklad
```console
$ moje_skrolovatko=less
$ $moje_skrolovatko unicorn.dat
```

```console
$ moje_skrolovatko=more
$ $moje_skrolovatko unicorn.dat
```
Všimni si, jak se výstup pokaždé mění.

> [note]
> Program `less` se vyvinul ze staršího programu `more`.
> Umí toho mnohem víc než jeho předchůdce, ale potřebuje „moderní“ terminál.
> Starší `more` si bez problémů vystačí s tiskárnou místo obrazovky.
> (Jméno `less` vychází z minimalistického hesla *less is more*.)


### Prompt a proměnné prostředí

Když napíšeš do příkazové řádky víceřádkový příkaz (zahájíš ho otvírací uvozovkou a ukončíš zadávání zavírací uvozovkou), všimni si změny výzvy.
Bude nejspíš vypadat zcela jinak. Poslední znak se většinou změní z `$` na `>`; to zohledníme tady v materiálech:

```console
$ echo "jeden řádek
> druhý řádek
> třetí řádek"
jeden řádek
druhý řádek
třetí řádek
```

#### Proměnné prostředí

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


## Cykly
### For

Cyklus `for` se v Bashi zapisuje podobně jako v Pythonu.

```console
$ for jmeno in a b c
> do
>     echo $jmeno
> done
a
b
c
```
V prvním řádku nechceš zjistit hodnotu proměnné, ale jen do ní přiřazuješ. Proto zde nepoužiješ znak `$`. `a b c` jsou slova, která se budou postupně přiřazovat do této proměnné. 
Po zmáčknutí <kbd>Enter</kbd> příkaz pokračuje (poznáš to podle změny výzvy na `>`). 
Následuje klíčové slovo `do`, které označuje zahájení příkazu pro každý z prvků `for` cyklu. 
Chceme, aby se pro každý průchod cyklem vypsal obsah proměnné `jmeno`. Napiš tedy na dalším řádku `echo $jmeno`.
Na dalším řádku následuje klíčové slovo `done`, označující konec příkazu a konec `for` cyklu.
Na rozdíl od Pythonu není nutné tělo cyklu odsadit: blok kódu se uvozuje pomocí slov `do` a `done`. Hezké odsazení ale zvětšuje přehlednost kódu. Doporučujeme, aby sis na něj zvykl{{a}}. 

Příkaz spusť; vypíše se postupně `a`, `b` a `c`.

Když pak dáš šipku nahoru, místo víceřádkového příkazu ti ho Bash vypíše v jednořádkové formě. Jednotlivé příkazy budou odděleny středníky. 
Středník a nový řádek v Bashi většinou znamenají totéž.

```console
$ for jmeno in a b c; do echo $jmeno; done
a
b
c
```

A teď už víš vše pro to, abys vypsal{{a}} druhý řádek každého souboru!
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

### Kontrola příkazu...
Kdybys chtěl{{a}} udělat něco drastičtějšího než jen vypsat části souborů, doporučujeme přidat ještě prostřední krok: použít `echo` pro vypsání samotného příkazu a kontrolu, že by se stalo opravdu to, co po počítači chceš.

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
Po kontrole můžeš smazat `echo` a příkaz se provede.

### ...a něco o chybách
Častá chyba je konstrukce níže, která sice udělá to co chceš, ale zároveň vypíše chybovou hlášku `head: cannot open 'ls' for reading: No such file or directory`.

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
Odpověď je jednoduchá: Bash interpretuje část `ls *.dat` jako seznam slov, nikoli výstup příkazu `ls`.
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


### While

V Bashi existuje taky cyklus `while`. Jeho syntaxe je podobná cyklu `for`.
```
$ while true; do head /dev/urandom|sha256sum; done
```
`true` je příkaz, který nedělá nic, a jeho návratová hodnota je 0 (tedy „pravda“, „OK“).

Mimochodem: `false` je další příkaz, který nedělá nic. Jeho návratová hodnota je ale 1 (tedy „nepravda“, „chyba“).


## Historie příkazů

Několik pomocných zkratek a kláves, které ti usnadní pohyb po příkazové řádce:
* Šipky <kbd>↑</kbd> a <kbd>↓</kbd> listuje v historii zadaných příkazů. To už znáš.
*  Příkaz `history` zobrazí historii zadaných příkazů.
*  <kbd>⭾TAB</kbd> je velmi užitečná klávesa, kterou už nejspíš taky znáš. Doplňuje příkazy a soubory, a když ji zmáčkneš 2×, zobrazí všechny soubory/příkazy, které začínají daným řetězcem.
* Některé operační systémy umožňují listovat v historii pomocí kláves <kbd>PgUp</kbd>/<kbd>PgDown</kbd>. Např. zadáš `cat ` a pak procházíš všemi zadanými příkazy, které začínají tímto začátkem.  (Není to ale základní součást Bashe; na jiných systémech to nemusí fungovat.)
* <kbd>Ctrl</kbd>+<kbd>R</kbd>: Hledání v historii: stiskneš <kbd>Ctrl</kbd>+<kbd>R</kbd>, pak zadáš část textu a zobrazí se ti nejbližší příkaz, který vyhovuje podmínce. Pokud chceš procházet dále do historie, opakovaně mačkáš <kbd>Ctrl</kbd>+<kbd>R</kbd>. Pro vyhledávání v historii dopředu použij <kbd>Ctrl</kbd>+<kbd>S</kbd>
* <kbd>Ctrl</kbd>+<kbd>C</kbd> zruší zadávání příkazu, nebo běh programu.
* <kbd>Ctrl</kbd>+<kbd>D</kbd> na začátku řádku ukončí Bash (často Bash k tomu ještě pro jistotu napíše `exit`) nebo ukončí vstup pro program (např. pro `cat`).
* <kbd>Ctrl</kbd>+<kbd>L</kbd> na začátku řádku vyčistí obrazovku.
* <kbd>Ctrl</kbd>+<kbd>W</kbd> smaže slovo před kurzorem.
* <kbd>Ctrl</kbd>+šipky <kbd>←</kbd><kbd>→</kbd> přesunují mezi slovy dlouhého příkazu.
* <kbd>Ctrl</kbd>+<kbd>S</kbd> pozastaví výstup (jakoby terminál *zamrznul*), ale příkazy se provádí.
* <kbd>Ctrl</kbd>+<kbd>Q</kbd> zobrazí vše, co bylo vytisknuto po <kbd>Ctrl</kbd>+<kbd>S</kbd> a „odmrazí“ terminál pro další příkazy.
* <kbd>Ctrl</kbd>+<kbd>Z</kbd> pozastaví právě prováděný příkaz a vrátí tě zpět do příkazové řádky.

K poslednímu triku, pozastavení příkazu, si povězme něco víc.

Otevři interaktivní režim Pythonu a zmáčkni <kbd>Ctrl</kbd>+<kbd>Z</kbd>
```console
$ python
Python 3.7.5 (default, Nov 20 2019, 09:21:52) 
[GCC 9.2.1 20191008] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> 
[1]+  Stopped                 python
$
```

Po pozastavení uvidíš znovu prompt Bashe.

Příkazem `jobs` zobrazíš pozastavené příkazy:

```console
$ jobs
[1]+  Stopped                 python
```

Příkaz `fg` (_**f**ore**g**round_) tě vrátí k poslednímu zastavenému příkazu.
```console
$ fg
python3
```

Python naposledy vypsal `>>>` a myslí si, že teď zadáváš příkaz. Zmáčkni <kbd>Enter</kbd> a `>>>` se objeví znovu.
Pak znovu stiskni <kbd>Ctrl</kbd>+<kbd>Z</kbd>; Python se znovu pozastaví a můžeš zadávat Bashové příkazy.

Příkaz `bg` (_**b**ack**g**round_) spustí zastavený program na pozadí.
```console
$ bg
[1]+ python3 &
```

Pomocí `kill %<číslo zastaveného programu>` ukončíš konkrétní proces (v našem případě zastavený příkaz číslo 1).
```console
$ kill %1
[1]+  Terminated              python3
```
Pokud spustíš příkaz s `&` na konci, spustí se proces přímo na pozadí a umožní ti pracovat dál ve stejné příkazové řádce.
```console
$ gitk --all &
```


## Bash skripty

Podobně jako u Pythonu můzeš Bashové příkazy psát do souboru, a pak Bashi říct ať je provede.
Zkus si to se souborem `klasifikace`:

```console
$ nano klasifikace
```

do souboru zapiš:

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
Bashové skripty se dají nastavit i tak, aby byly spustitelné přímo jako příkazy. Proto soubory s Bashovými skripty často nemají příponu. (Když je přípona vhodná, používá se `.sh`.)

Udělat ze skriptu příkaz ale není úplně přímočaré.
Zkus použit naši *klasifikaci* jako příkaz:

```console
$ klasifikace 
klasifikace: command not found
```
So s tím?
Na začátek skriptu patří takzvaný *shebang*, který říká jakým programem se soubor bude spouštět. 
V našem případě chceme spouštět Bash. 
Na první řádek souboru klasifikace proto připiš dva speciální znaky `#!` a jméno souboru, ve kterém je uložený program Bash:
```bash
#! /bin/bash
```

> [note]
> Shebang je instrukce pro systém, že `klasifikace` se bude spouštět Bashem.
> Pro Bash samotný je to komentář – začíná `#`.

(XXX: sem dát mini povídání k `$PATH` a nutnost pouštění s `./`)

Další věc, kterou musíš udělat, je nastavení speciálních práv souboru.

`ls -l` zobrazí spoustu informací o souboru `klasifikace`, ale nás momentálně zajímají především práva, která jsou zobrazena hned vlevo:

```console
$ ls -l klasifikace 
-rw-r--r-- 1 user user 58 Oct 15 19:12 klasifikace
```

Existují tři základní druhy oprávnění, pro které se zobrazí buď písmenko (pokud je oprávnění povoleno) nebo pomlčka (pokud je odepřeno):

- r - právo pro čtení souboru (**r**ead)
- w - právo na zápis souboru (**w**rite)
- x - právo na spuštění souboru (e**x**ecute)

Tyto práva se opakují 3x za sebou, jednou pro vlastníka souboru (zde `-rw`, vlastník tedy může číst a psát), podruhé pro skupinu (*group*, zde `-r-`, skupina tedy může jen číst), a potřetí pro všechny ostatní uživatele (zde opět můžou jen číst).

Spustitelnému souboru musíš přidat právo na spuštění, což zajistí příkaz `chmod +x`. (Přesné chování `chmod` si vysvětlíme později.)
```console
$ chmod +x klasifikace
$ ls -l klasifikace 
-rwxr-xr-x 1 user user 58 Oct 15 19:12 klasifikace
```
Po těchto operacích stačí zadát jméno souboru, a on se spustí v Bashi! Jen je ho potřeba zadat s plnou cestou, tedy s `./` na začátku:

```console
$ ./klasifikace 
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

Náš skript `klasifikace` zpracovává vždy všechny soubory `.dat` v daném adresáři. 
Co kdybys chtěl{{a}} použít tento skript trochu jinak: aby ukázal data jen z daného souboru? 
Vytvořme programu `klasifikuj` tak, aby fungovalo třeba:

```console
$ ./klasifikuj minotaur.dat
```
Vytvoř soubor `klasifikuj`, dej do něj tento obsah, a přidej právo na spuštění:
```bash
#! /bin/bash
head -n2 "$1" | tail -n1
```

`"$1"` je speciální proměnná označující první argument příkazové řádky, tedy u `./klasifikuj minotaur.dat` ono `minotaur.dat`.

Proč je v uvozovkách? 
Protože kdybys zadal{{a}} jako první argument text s mezerami, uložily by se jako víc argumentů. 
Když je sekvence takto v uvozovkách, celý první argument se použije správně.
Platí obecné pravidlo: pokud bereš odněkud argument, napiš ho pro jistotu do uvozovek.

Kdybys chtěl{{a}} použít další argumenty, použij `"$2"`, `"$3"` atd.
A `$0` je nultý argument, tedy jméno programu, který se spouští.

```console
$ ./klasifikuj minotaur.dat
CLASSIFICATION: bos hominus
```

Existuje ještě `$@`, což znamená všechny argumenty. 
Uprav původní skript `klasifikace` tak, aby necyklil po `*.dat`, ale `"$@"`. Teď ho můžeš spouštět jak pro jeden soubor, tak pro seznam souborů.

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
*Shebang* zajistí, že se program spustí Pythonem. 
```console
$ chmod +x nahoda
$ ./nahoda
34
$ ./nahoda
73
```

Ukázka výše funguje pro všechny programy, jen musíš vědět, kde přesně na disku jsou (často v `/usr/bin/`; do detailů půjdeme později).
 
Ukažme si to ještě s `cat`.
Do souboru `README` napiš:
```plain
#! /usr/bin/cat
 
Tady jsou informace.
```
Přidej práva na spouštění a spusť ho:

```console
$ chmod +x README
$ ./README
#! /usr/bin/cat

Tady jsou informace.
```
Je to stejné, jako kdybys spustil{{a}} `cat README`!


### PATH
Možná si teď kladeš otázku, k čemu je v příkladu `./`, když ostatní příkazy zadáváš prostě jménem.
Když zadáš pouze:

```console
$ klasifikace *.dat
klasifikace: command not found
```

Bash si postěžuje, že nezná takový příkaz. 
Bash se totiž dívá jen do určitých adresářů, pokud mu nebylo dáno přesné umístění souboru (což `./klasifikace` je). 
Tyto adresáře, které prohledává Bash, jsou obsažené v proměnné `PATH`.
Vypiš si její obsah na příkazovou řádku:

```console
$ echo $PATH
/home/user/.local/bin:/home/user/bin:/usr/share/Modules/bin:/usr/local/bin:/usr/local/sbin:/usr/bin:/usr/sbin:/var/lib/snapd/snap/bin
```

Je v ní spousta adresářů.
Měl{{a}} bys v mimo jiné vidět `/usr/bin`, kde jsou systémové příkazy, a `/home/user/bin`, kde můžeš definovat vlastní příkazy. 
Jak to funguje? Zkopíruj soubor `klasifikace` do adresáře `bin` ve své domovské složce (v materiálech je to `/home/user/bin/`, ale u tebe se jmenuje jinak). 
Pak můžeš skript spouštět jako příkaz, bez `./` na začátku:

```console
$ klasifikace *.dat
CLASSIFICATION: basiliscus vulgaris
CLASSIFICATION: bos hominus
CLASSIFICATION: equus monoceros
```

V domovském adresáři je skrytý skript `~/.bashrc`, který se spustí vždy, když pustíš Bash. 
Je to soubor, ve kterém je možné upravit si prostředí, např. přidat cesty do `PATH`, upravit `PS1`, nadefinovat si vlastní funkce a spoustu dalších.


## Hledání

Přesuň se do adresáře `data-shell/writing`, kde se nachází soubor `haiku.txt`

```console
$ cat haiku.txt 
The Tao that is seen
Is not the true Tao, until
You bring fresh toner.

With searching comes loss
and the presence of absence:
"My Thesis" not found.

Yesterday it worked
Today it is not working
Software is like that.
```

### grep
Náš úkol je podívat se na řádky, kde se nachází slovo *not*.
Existuje program `grep`, kterému když předáš hledané slovo a soubor(y) k prohledání, řádky s daným slovem vypíše.

Následující příkaz tedy hledá v souboru `haiku.txt` řetězec `not` a vypíše každý řádek s tímto řetězcem:

```console
$ grep not haiku.txt 
Is not the true Tao, until
"My Thesis" not found.
Today it is not working
```

Příkaz `grep` má spoustu zajímavých přepínačů, například:
- `-i`: Nezáleží na velikosti písmen (_case **i**nsensitive_).
- `-n`: Vypíše čísla řádků.
- `-w`: Hledá jen slova.
- `-F`: Hledá přesně zadaný řetězec (viz níže).

Takhle můžeš najít všechny řádky se slovem *the* nebo *The*:

```console
$ grep -i the haiku.txt 
The Tao that is seen
Is not the true Tao, until
and the presence of absence:
"My Thesis" not found.
```

Ups, je tam ale něco navíc! Jakým příkazem odstraníš z výpisu řádek se slovem *Thesis*?

```console
$ grep -iw the haiku.txt 
The Tao that is seen
Is not the true Tao, until
and the presence of absence:
```

`grep` je velice užitečný program. Pokud pracuješ s Bashem, brzy se z něj stane tvůj nejlepší kamarád. 
Pozor ale na to, co `grep` bere jako hledaný řetězec, je to totiž regulární výraz (angl. *regular expression*, často taky *regex*). Pokud hledáš jen písmenka, nenarazíš na problém, ale u znaků jako tečky, hvězdičky, otazníky může být výstup jiný, než očekáváš.

Zkus několik příkazů:
```console
$ grep -i the.i haiku.txt 
$ grep -i '.*' haiku.txt
$ grep -iw '.....' haiku.txt
```

Speciální funkce znaků můžeš vypnout pomocí přepínače `-F`.
S ním bude `grep` hledat přesně zadaný řetězec.


### find

Příkaz `find` je další užitečný hledací příkaz.

```console
$ find . -name '*.txt'
./haiku.txt
./data/LittleWomen.txt
./data/two.txt
./data/one.txt
```
Příklad výše hledá v tomto adresáři (`.`) a všech podadresářích soubory, které odpovídají masce `*.txt`.

> [note]
> Proč je `'*.txt'` v uvozovkách?
> Vzpomeň si, že znak `*` zpracovává samotný Bash: kdybys `*.txt` zadal{{a}} bez uvozovek, `find` by dostal jako argumenty všechny `*.txt` soubory z aktuálního adresáře.
> Uvozovky říkají Bashi, aby příkazu `find` předal opravdu `*.txt`, a `find` tak může odpovídající soubory hledat sám.

Podobného výsledku dosáhneš (ale bez jména adresáře), když zadáš:

```console
$ ls --recursive | grep txt
haiku.txt
LittleWomen.txt
one.txt
two.txt
```

I bez jména adresáře se ale tento výstup občas hodí – třeba když chceš spočítat, kolik těch souborů je:

```console
$ ls --recursive | grep txt | wc -l
4
```

## Poznámka k neznámým příkazům

Kdykoli ti někdo řekne nebo napíše o zajímavém programu, který si určitě musíš vyzkoušet, buď opatrn{{gnd('ý', 'á')}}. 
I v případě, že si ten program nainstaluješ, použij nápovědu pro zjištění, zda opravdu dělá to, co ti bylo řečeno / napsáno.
A právě zde je rozdíl mezi `man <jméno neznámého programu>` a `<jméno neznámého programu> --help`.

`man <jméno neznámého programu>` spouští program `man` a jako argument mu dává *neznámý program*, aniž by ho spustil.
V případě`<jméno neznámého programu> --help` říkáš: *Hej, neznámý programe, řekni mi něco o sobě.*, čili spouštíš ho, aby ti ukázal vestavěnou nápovědu. Pokud nevěříš *neznámému programu*, nemůžeš věřit ani tomuto příkazu.


## Úkol

Inspirace Bashe: https://github.com/encukou/bin


Naklonuj si tyto repozitáře (přímo v adresáři pro tento kurz; `git clone` udělá nový podadresář):

    $ git clone https://github.com/pyladiescz/pyladies.cz
    $ git clone https://github.com/pyvec/pyvo-data

Data si prohlédni a zjisti, co se v nich skrývá za informace. Zvlášť doporučuju třeba soubor `pyvo-data/series/brno-pyvo/events/2018-10-25-casove.yaml`.

Použij základní shellové příkazy (ne Python) na zodpovězení těchto otázek:

* Kolik bylo kurzů/srazů PyLadies?
* Kolik bylo Pyv v Brně?
* Kolik bylo Pyv celkem?
* Z kolika přednášek na Pyvech jsou videa?  *(Předpokládej že každá přednáška může mít max. 1 video)*
* Z kolika Pyv jsou videa?

* Vypiš všechna místa konání Pyv (stačí identifikátor jako `artbar`) a kolikrát tam Pyvo bylo.
* Jaké jsou 3 nejčastější křestní jména organizátorů/koučů/atd. PyLadies?

YAML soubory by se správně měl číst knihovnou na YAML, aby byla zachována struktura. Ty je ale ber jako "čistý text", kde hledané informace jsou na řádcích ve tvaru `klíč: hodnota` (příp. s nějakýma mezerama a/nebo pomlčkama navíc). Odpovědi tak nemusí být 100% přesné.


---

V Pythonu napiš funkci, která bere řetězec a vrátí "obrácený" řetězec: znaky jsou v něm pozpátku a podle následujícího slovníku. Znaky, které ve slovníku nejsou, program vypíše nezměněné.

(Nápověda k Pythonu je níže.)

    {'a': 'ɐ', 'b': 'q', 'c': 'ɔ', 'd': 'p', 'e': 'ǝ', 'f': 'ɟ', 'g': 'ƃ',
    'h': 'ɥ', 'i': 'ᴉ', 'j': 'ɾ', 'k': 'ʞ', 'l': 'l', 'm': 'ɯ', 'n': 'u',
    'o': 'o', 'p': 'd', 'q': 'b', 'r': 'ɹ', 's': 's', 't': 'ʇ', 'u': 'n',
    'v': 'ʌ', 'w': 'ʍ', 'x': 'x', 'y': 'ʎ', 'z': 'z', 'A': '∀', 'B': 'B',
    'C': 'Ɔ', 'D': 'D', 'E': 'Ǝ', 'F': 'Ⅎ', 'G': 'פ', 'H': 'H', 'I': 'I',
    'J': 'ſ', 'K': 'ʞ', 'L': '˥', 'M': 'W', 'N': 'N', 'O': 'O', 'P': 'Ԁ',
    'Q': 'Q', 'R': 'R', 'S': 'S', 'T': '┴', 'U': '∩', 'V': 'Λ', 'W': 'M',
    'X': 'X', 'Y': '⅄', 'Z': 'Z', '0': '0', '1': 'Ɩ', '2': 'ᄅ', '3': 'Ɛ',
    '4': 'ㄣ', '5': 'ϛ', '6': '9', '7': 'ㄥ', '8': '8', '9': '6', ',': "'",
    '.': '˙', '?': '¿', '!': '¡', '"': '„', "'": ',', '`': ',', '(': ')',
    ')': '(', '[': ']', ']': '[', '{': '}', '}': '{', '<': '>', '>': '<',
    '&': '⅋', '_': '‾'}

Např.:

    >>> obrat("Ahoj, brněnské PyLadies!")
    '¡sǝᴉpɐ˥ʎԀ éʞsuěuɹq 'ɾoɥ∀'

Udělej z toho program pro příkazovou řádku, který bere soubory k obrácení. Když nedostane žádný argument, použije standardní vstup. Argument `-` taky znamená standardní vstup.

    $ echo Ahoj | obrat
    ɾoɥ∀
    $ echo Ahoj | obrat -
    ɾoɥ∀


    $ echo 'Ahoj,
    > PyLadies!' > pozdrav.txt
    $ obrat pozdrav.txt
    'ɾoɥ∀
    ¡sǝᴉpɐ˥ʎԀ
    $ echo haha | echo obrat pozdrav.txt - pozdrav.txt
    'ɾoɥ∀
    ¡sǝᴉpɐ˥ʎԀ
    ɐɥɐɥ
    'ɾoɥ∀
    ¡sǝᴉpɐ˥ʎԀ

Zařiď, aby s přepínačem `--help` program vypsal krátkou nápovědu (a ignoroval ostatní argumenty).
Je-li použit jiný přepínač (začínající `-`), program by měl uživateli vynadat (na chybovém výstupu), vrátit chybovou návratovou hodnotu a ignorovat ostatní argumenty.

Nakonec program změň tak, aby vracel chybovou návratovou hodnotu když některý znak chybí ve slovníku.

    $ echo Ahoj | obrat
    ɾoɥ∀
    $ echo $?
    0
    $ echo Čau | obrat
    nɐČ
    $ echo $?
    1

Naimportuješ-li `sys` a `os`, pak:

* `sys.argv` je seznam argumentů (včetně jména programu)
* `sys.stdin` je *už otevřený* soubor se std. vstupem (netřeba `with` či `close`)
* Podobně `sys.stdout` je soubor se standardním výstupem (tam píše `print`) a `sys.stderr` je soubor chybovým výstupem.
* `os.environ` je slovník* s proměnnýma prostředí
* `exit(1)` ukončí program s danou hodnotou

(* přesněji řečeno, objekt který se chová jako slovník)
