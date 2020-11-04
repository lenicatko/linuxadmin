# Roury a filtry

Teď, když znáš základní příkazy, si ukážeme nejmocnější vlastnost shellu:
možnost příkazy jednoduše spojovat dohromady.
Začni opět v adresáři `molecules`, kde je několik souborů s popisem molekul:

```console
$ cd ~/Dokumenty/data-shell/molecules
$ ls -F
cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
```

Pomocí `wc -l` si vypiš, kolik mají tyto soubory řádků:

```console
$ wc -l *.pdb
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 celkem
```

> [note]
> Velikost všech souborů by vypsal i `wc -l *`, ale psát `*.pdb` místo `*`
> se začne hodit hned za chvíli, když začneš tvořit nové soubory.

Který z těchto souborů je největší?
A který je nejmenší?
Tohle jsou docela jednoduché otázky, ale představ si, že bys těch souborů
měl{{a}} 6000.
To už by se hodilo zeptat počítače spíš než hledat očima.


## Přesměrování výstupu

První věc, kterou uděláme, je *přesměrování výstupu*.
Pomocí `>` pusť příkaz `wc` tak, aby místo na obrazovku zapisoval do souboru:

```console
$ wc -l *.pdb > delky.txt
```

Na obrazovku se nic nevypíše, ale vytvoří se soubor `delky.txt`:

```console
$ ls delky.txt
delky.txt
```

A v tom souboru je všechno, co by příkaz `wc` normálně vypsal do terminálu:

```console
$ cat delky.txt
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 celkem
```

Jak to funguje?

Každý příkaz v Bashi (a obecně každý proces — běžící program na počítači)
má k dispozici takzvaný *standardní výstup* (angl. *standard output*, *stdout*):
místo, kam může vypisovat výsledky.
Bash předtím, než spustí nějaký program, nastaví tenhle standardní výstup buď
do terminálu nebo do souboru.
Programy jako `cat`, `wc` nebo `tail` se pak nestarají o to, kde ty výsledky
skončí.
Prostě je vypíšou na předem určené místo.

{{ figure(
    img=static('wc-to-term.svg'),
    alt='Diagram příkazu `wc -l *.pdb`',
) }}

{{ figure(
    img=static('wc-to-file.svg'),
    alt='Diagram příkazu `wc -l *.pdb` přesměrovaného do souboru',
) }}

Pomocí `>` říkáš Bashi, aby výstup programu přesměroval do souboru.
Když soubor daného jména neexistuje, Bash ho vytvoří.
Pokud existuje, Bash jeho obsah ještě před spuštěním příkazu *smaže*
a nechá příkaz, ať do něj zapíše něco nového.
Takže postupuj opatrně: jak už víš že na příkazové řádce se nepoužívá
odpadkový koš nebo tlačítko Zpět.

> [warning] Přesměrováním přepíšeš existující soubor
> A ještě jednou v červeném rámečku: pokud soubor za `>` existuje, smaže se.
> Na kurzu to nevadí – virtuální počítač máš na hraní – ale „v reálném světě“
> si dělej zálohy.

## Hadí odbočka

Program python má přepínač -c, který umožňuje zadat pythonní příkaz přímo
jako argument příkazové řádky.
I výstup této operace můžeš přesměrovat do souboru a dál s ním pracovat.

```console
$ python -c 'print(1 + 1)' > ctyri.txt
$ cat ctyri.txt
2
```


## Řazení

Seznam se s příkazem `sort` (angl. *seřaď*), který seřadí řádky daného souboru.
Soubor `delky.txt` vypadá takhle:
```console
$ cat delky.txt
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 celkem
```

A příkaz `sort` ho seřadí následujícím způsobem:

```console
$ sort delky.txt
 107 celkem
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
   9 methane.pdb
```

To není úplně ono – `sort ` totiž řadí podle abecedy, nikoli číselně;
`102` je tedy před `12` podobně jako `BAC` je v abecedě před `BC`.

Ale jak už to bývá, tohle chování se dá změnit pomocí přepínače – v tomto
případě přepínače `-n` (`--numeric-sort`, číselné řazení):

```console
$ sort -n delky.txt
   9 methane.pdb
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
 107 celkem
```

## Nejmenší molekula

Která molekula tedy obsahuje nejmíň atomů?

Když umíš jména souborů seřazené podle počtu řádků vypsat,
stejně tak je můžeš uložit do souboru.
A ten soubor potom můžeš dál zpracovávat.
Nejkratší soubor bude v prvním řádku;
prvních <var>n</var> řádků souboru vypíšeš pomocí `head`:

```console
$ sort -n delky.txt > delky-serazene.txt
$ head -n1 delky-serazene.txt
   9 methane.pdb
```

Celý postup zjištění nejkratší molekuly tedy je:

```console
$ wc -l *.pdb > delky.txt
$ sort -n delky.txt > delky-serazene.txt
$ head -n1 delky-serazene.txt
   9 methane.pdb
```

Neboli česky:
* Počty řádků a jména souborů zapiš do `delky.txt`.
* Soubor `delky.txt` seřaď (číselně) a zapiš do souboru `delky-serazene.txt`.
* Ze souboru `delky-serazene.txt` vypiš první řádek.

Tahle sekvence příkazů bude fungovat i kdyby souborů s molekulami bylo 6000.
Můžeš si je zapsat do souboru a použít příště, až zase budeš hledat nejmenší
molekulu!

Jde to zjednodušit ještě dál, ale než si ukážeme jak na to,
podívejme se na přesměrování *vstupu*.


## Přesměrování vstupu

Tak jako jde přesměrovat *výstup* příkazu pomocí `>` – tedy to,
co by se vypsalo na obrazovku, zapsat do souboru,
jde přesměrovat i *vstup* – tedy to, co bys napsal{{a}} na klávesnici,
načíst ze souboru.

Už víš, že když příkaz jako `cat` pustíš bez jména souboru,
načítají informace „z klávesnice“:

```console
$ cat
haló haló,
haló haló,
co se stalo?
co se stalo?
^C
```

Podobně jako u standatního výstupu existuje *standardní vstup*
(angl. *standard input*, *stdin*) – místo, odkud program získává textové
informace.
Normálně to je „klávesnice“:

{{ figure(
    img=static('cat.svg'),
    alt='Diagram příkazu `cat`',
) }}

Standartní vstup můžeš přesměrovat tak, aby místo toho, co napíšeš klávesnici,
příkaz zpracoval daný soubor.

```console
$ cat < delky.txt
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 celkem
```

{{ figure(
    img=static('cat-from-file.svg'),
    alt='Diagram příkazu `cat`',
) }}

Spousta programů standardní vstup nepoužívá:
* `ls`, `cd` nebo `mv` dostávají všechny potřebné informace jako argumenty,
  nepotřebují vstup z klávesnice;
* Příkazy s argumentem jako `cat soubor.txt` nebo `wc soubor.txt` čtou
  daný soubor. Klávesnici ignorují, i když jim je k dispozici:

  {{ figure(
    img=static('cat-file-arg.svg'),
    alt='Diagram příkazu `cat soubor.txt`',
  ) }}

Spousta příkazů bez argumentu ale standardní vstup používá:

```console
$ sort -n < delky.txt
   9 methane.pdb
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
 107 celkem
```

```console
$ wc -l < delky.txt
7
```

```console
$ head -n1 < delky-serazene.txt
   9 methane.pdb
```

Všimni si, že příkaz `wc` nevypsal název souboru – chová se tak, jako kdybys
obsah `delky.txt` napsal{{a}} na klávesnici.

K čemu je to dobré?
Zatím, pravda, k ničemu.
Příkazy `cat`, `wc` i `sort` umí soubory z disku otevřít samy;
nepotřebují na to od Bashe pomoc.

Vstup a výstup ale můžeš přesměrovat i na jiné věci než jen soubory na disku.


## Datové potrubí

Najdi na klávesnisi znak `|`, kterému budu říkat *svislítko*.
Tenhle znak nejspíš nemá oficiální české jméno.
Formálně se mu dá říkat *svislá čára*, často ale uslyšíš pojmenování
*roura* nebo *pajpa* (z angl. *pipe*, roura), která vychází z funkce tohoto
znaku v Bashi. Tak moc je ta funkce důležitá.

Když svislítkem oddělíš dva příkazy, standardní výstup prvního z nich
se přesměruje na standardní vstup toho druhého.
Dva příkazy tak spojíš dohromady: první připraví mezivýsledek se kterým ten
druhý pak pracuje dál.

Zkus si to na příkladu:

```console
$ sort -n delky.txt | head -n 1
   9 methane.pdb
```

{{ figure(
    img=static('sort-head-pipeline.svg'),
    alt='Diagram příkazů `sort -n delky.txt | head -n 1`',
) }}

Toto je ekvivalent dvou příkazů, které jsi použil{{a}} dříve – jen s tím
rozdílem, že se pro mezivýsledek nepoužívá soubor, ale text se přesměruje
rovnou z jednoho příkazu do druhého.

```console
$ sort -n delky.txt > delky-serazene.txt
$ head -n1 < delky-serazene.txt
   9 methane.pdb
```

{{ figure(
    img=static('sort-head-nopipe.svg'),
    alt='Diagram dvojice příkazů výše',
) }}

Takovému přesměrování se říká *roura* (angl. *pipe*): můžeš si představit,
že text “teče” z jednoho příkazu do druhého.

Z rour můžeš sestavovat celá potrubí,
kterým budou data proudit mezi jednotlivými příkazy:

```console
$ wc -l *.pdb | sort -n | head -n 1
```

{{ figure(
    img=static('pipeline.svg'),
    alt='Diagram příkazů `wc -l *.pdb | sort -n | head -n 1`',
) }}

Toto spojování stojí za úspěchem Unixových shellů.
Místo obrovských programů (např. tabulkový procesor, textový editor,
webový prohlížeč) které mají nepřeberné množství různých funkcí se Unixoví
programátoři soustředí na jednoduché nástroje (např. `cat`, `wc`, `head`,
které bys uměl{{a}} napsat v Pythonu i ty).
Každý z těchto jednoduchých nástrojů dělá jen jednu věc, ale dělá ji dobře.
A pomocí rour jde nástroje spojovat k řešení mnohem komplexnějších situací,
než na jaké můžou myslet autoři složitých programů.

Každý způsob má samozřejmě svoje místo.
Nic tě nenutí používat *jen* příkazovou řádku, která je mnohem složitější
na ovládnutí než „klikátko“ s předpřipravenou nabídkou akcí.
Ale jakmile se Bash a malé nástroje naučíš efektivně používat,
{{gnd('sám', 'sama')}} poznáš jak jsou užitečné.

Modelu pospojovaných nástrojů se také říká *roury a filtry*
(angl. *pipes and filters*).
Roury už znáš; filtry jsou nástroje jako `wc` a `sort`, které nějak
transformují standardní vstup na standardní výstup.
Většina Unixových nástrojů funguje právě takhle: pokud jim nepředáš
argument, čtou ze std. vstupu a výsledky píší na std. výstup.


## Řádky a podvodníci

Protože výchozí std. vstup je klávesnice a výchozí std. výstup je obrazovka,
rourami většinou „teče“ text, který je více či méně srozumitelný pro lidi.
Často v něm každá položka bývá na zvláštním řádku.
Právě na taková data je připravena celá řada nástrojů: od filtrů jako `sort`,
který řadí řádky, po Pythoní `for radek in soubor:` po `git diff` a jeho
ekvivalent na webu GitHub.

Co ale takový `ls`, který skládá jména souborů za sebe – na jeden řádek
jich dá několik?

```console
$ ls -F ..
creatures/  diplomka_archiv/  north-pacific-gyre/  plan.txt   writing/
data/       molecules/        pizza.cfg            solar.pdf
```

Příkaz `ls` trochu podvádí: zjistí si, jestli má na std. výstupu terminál,
a pokud ano, přizpůsobí svůj výpis „lidem“.

{{ figure(
    img=static('ls-cheats.svg'),
    alt='Diagram příkazu `ls`, který trochu podvádí',
) }}

Když ale *nemá* na std. výstupu terminál, dává každé jméno souboru na
zvláštní řádek, aby se výsledek dal dobře zpracovat dalšími filtry:

```console
$ ls -F .. | cat
creatures/
data/
diplomka_archiv/
molecules/
north-pacific-gyre/
pizza.cfg
plan.txt
solar.pdf
writing/
```

Když si to zkusíš, zjistíš že z výstupu zmizely i barvy.
I to je vlastnost, která se špatně zpracovává filtrem, a tak ji `ls` přidává
jen pokud vypisuje do terminálu.

> [note]
> Budeš-li někdy psát program který něco vypisuje barevně, nezapomeň
> dát uživatelům možnost barvy vypnout (viz `--color` v `man ls`).
> A ideálně nastav výchozí chování podle toho, jestli je std. výstup terminálem.










