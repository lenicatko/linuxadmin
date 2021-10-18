{# 
  # XXX Nelle’s Pipeline: Processing Files
  # - https://swcarpentry.github.io/shell-novice/05-loop/index.html#nelles-pipeline-processing-files
#}

# Nela a medúzy: Zpracování souborů

Nela teď ví o svých souborech dost na to, aby je zpracovala pomocí programu
`goostats` – skriptu, který napsal její vedoucí.
Tenhle program vypočítá jakousi statistiku ohledně proteinů.
Potřebuje dva argumenty:

1. vstupní soubor (odkut přečte naměřená data),
2. výstupní soubor (kam zapíše výsledky).

Protože se Nela shell zatím učí, vybuduje si příkazy postupně.
První krok je vybrat správné vstupní soubory – ty co končí na `A` nebo `B`,
ale ne `Z`.
Nela píše:

```console
$ cd ~/Dokumenty/data-shell/north-pacific-gyre/2012-07-03
> for soubor in NENE*A.txt NENE*B.txt
> do
>     echo $soubor
> done
NENE01729A.txt
NENE01736A.txt
NENE01751A.txt
...
NENE02040B.txt
NENE02043B.txt
```

Dále bude potřeba vymyslet jména souborů do kterých se zapíšou výsledky.
Řekněme že je pojmenuje tak, že na začátek každého vstupního jména
přidá `vysledky-`.
Napíše si tedy cyklus, který ukáže vstupní i výstupní jméno:

```console
$ for soubor in NENE*[AB].txt
> do
>     echo $soubor vysledky-$soubor
> done
NENE01729A.txt vysledky-NENE01729A.txt
NENE01729B.txt vysledky-NENE01729B.txt
NENE01736A.txt vysledky-NENE01736A.txt
...
NENE02043A.txt vysledky-NENE02043A.txt
NENE02043B.txt vysledky-NENE02043B.txt
```

Nové soubory nezačínají na `NENE`, takže i když je potom vytvoří,
cyklus `for soubor in NENE*[AB].txt` je nebude brát v potaz.

Zatím tedy sice nespustila program `goostats`, ale jména souborů už zná.

Psát skoro ten samý příkaz pořád dokola je ale nudné.
A navíc tak Nela může snadno udělat chybu.
Proto zmáčkne <kbd>↑</kbd> a Bash ukáže cyklus na jednom řádku:

```console
$ for soubor in NENE*[AB].txt; do     echo $soubor vysledky-$soubor; done
```

Pomocí <kbd>←</kbd> se dostane za `echo` a připíše příkaz, kterým se spouští
skript od vedoucího: `bash goostats`.

```console
$ for soubor in NENE*[AB].txt; do     echo bash goostats $soubor vysledky-$soubor; done
bash goostats NENE01729A.txt vysledky-NENE01729A.txt
bash goostats NENE01729B.txt vysledky-NENE01729B.txt
bash goostats NENE01736A.txt vysledky-NENE01736A.txt
...
bash goostats NENE02043A.txt vysledky-NENE02043A.txt
bash goostats NENE02043B.txt vysledky-NENE02043B.txt
```

To jsou přesně příkazy, které by musela spouštět ručně!
Znovu tedy vyvolá předchozí řádek a smaže slovo `echo`.
Skript se teď opravdu pustí.

```console
$ for soubor in NENE*[AB].txt; do     bash goostats $soubor vysledky-$soubor; done
```

Program `goostats` se možná spustil, ale nic se nevypisuje.
To Nele trochu vadí.
Rozhodne se cyklus ukončit (<kbd>Ctrl</kbd>+<kbd>C</kbd>) a přidat do cyklu
další příkaz: `echo $soubor`.
Vždycky před spuštěním analýzy se tak vypíše jméno zpracovávaného souboru.

```console
$ for soubor in NENE*[AB].txt; do     echo $soubor; bash goostats $soubor vysledky-$soubor; done
NENE01729A.txt
```

> [note]
> Neboli na více řádků:
>
> ```console
> $ for soubor in NENE*[AB].txt
> > do
> >     echo $soubor
> >     bash goostats $soubor vysledky-$soubor
> > done
> ````

Tenhle příkaz už něco vypisuje! Každá analýza trvá asi 5 vteřin.
Nela má reálně těch souborů 1500 (naše demo je menší), a tak si spočítá
že to celé bude trvat asi dvě hodiny.
Než si půjde uvařit kafe a přečíst nový oceánografický článek,
otevře si ještě novou konzoli a zkontroluje, že v adresáři vznikají
soubory s výsledky a obsah prvních z nich vypadá rozumně.
