# Vypisování souborů

Teď, když se umíš pohybovat po adresářovém systému a vytvářet soubory,
se podíváme na to, co v těch souborech je.

Začni v adresáři `data-shell/molecules`:

```console
$ cd ~/Dokumenty/data-shell/molecules
$ ls -F
cubane.pdb  ethane.pdb  methane.pdb  octane.pdb  pentane.pdb  propane.pdb
```


## Vypisovací kočka

Podle přípony jsou tu soubory ve formátu *Protein Data Bank*,
což je textový formát, kterým se dají popsat druhy a pozice atomů v molekule.
Jak takový `.pdb` soubor vypadá?
Zkus si nějaký vypsat pomocí příkazu `cat`:

```console
$ cat methane.pdb
COMPND      METHANE
AUTHOR      DAVE WOODCOCK  95 12 18
ATOM      1  C           1       0.257  -0.363   0.000  1.00  0.00
ATOM      2  H           1       0.257   0.727   0.000  1.00  0.00
ATOM      3  H           1       0.771  -0.727   0.890  1.00  0.00
ATOM      4  H           1       0.771  -0.727  -0.890  1.00  0.00
ATOM      5  H           1      -0.771  -0.727   0.000  1.00  0.00
TER       6              1
END
```

Do terminálu se ti vypíše obsah souboru – to, co by se ti ukázalo v textovém
editoru, kdybys soubor otevřel{{a}}.

### Proč `cat`?

Jméno `cat` je zkratka z angl. *catenate*/*concatenate* – zřetězit,
spojit do sekvence. Je to termín pro spojování řetězců, jak to znáš
z Pythonu: `'Ahoj' + ' ' + 'světe'`.
Když dáš příkazu `cat` několik argumentů, tak postupně vypíše obsah každého
daného souboru. Spojí je dohromady, jeden za druhý:

```console
$ cat methane.pdb ethane.pdb
COMPND      METHANE
AUTHOR      DAVE WOODCOCK  95 12 18
ATOM      1  C           1       0.257  -0.363   0.000  1.00  0.00
ATOM      2  H           1       0.257   0.727   0.000  1.00  0.00
ATOM      3  H           1       0.771  -0.727   0.890  1.00  0.00
ATOM      4  H           1       0.771  -0.727  -0.890  1.00  0.00
ATOM      5  H           1      -0.771  -0.727   0.000  1.00  0.00
TER       6              1
END
COMPND      ETHANE
AUTHOR      DAVE WOODCOCK  95 12 18
ATOM      1  C           1      -0.752   0.001  -0.141  1.00  0.00
ATOM      2  C           1       0.752  -0.001   0.141  1.00  0.00
ATOM      3  H           1      -1.158   0.991   0.070  1.00  0.00
ATOM      4  H           1      -1.240  -0.737   0.496  1.00  0.00
ATOM      5  H           1      -0.924  -0.249  -1.188  1.00  0.00
ATOM      6  H           1       1.158  -0.991  -0.070  1.00  0.00
ATOM      7  H           1       0.924   0.249   1.188  1.00  0.00
ATOM      8  H           1       1.240   0.737  -0.496  1.00  0.00
TER       9              1
END
```

V praxi se `cat` většinou používá pro vypsání jediného souboru,
ale jméno stále odkazuje na tohle spojování.
S kočkama samozřejmě nemá vůbec, ale vůbec nic společného 😺


### Opakovací kočka

Když zadáš jen příkaz `cat` bez jména souboru, nestane se naoko nic.
To proto, že `cat` v tomhle případě vypisuje vstup z klávesnice – a to
po řádcích, takže dokud něco nenapíšeš a nestiskneš <kbd>Enter</kbd>,
může to vypadat že „Bash zamrzl“.

A jakmile napíšeš řádek, `cat` ho zopakuje.
To zatím není moc užitečné.
Jako skoro každý program se `cat` dá ukončit
pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd>.

```console
$ cat
ahoj
ahoj
blablabla
blablabla
^C
```

> [note]
> První opakování řádku má „na svědomí“ terminál, který automaticky ukazuje
> co napíšeš na klávesnici a nechá tě např. opravovat chyby pomocí
> <kbd>Backspace</kbd>.
> Příkazu předá až kompletní řádek, když stiskneš <kbd>Enter</kbd>.
> (Tohle chování se dá vypnout; programy jako `nano` se tak můžou starat
> o veškerý výstup samy.)
> Druhé opakování přichází od příkazu `cat`, který vypisuje co od terminálu
> obdrží, když zmáčkneš <kbd>Enter</kbd>.


## Hlavičky a ocásky

Zkus si vypdat obsah souboru `octane.pdb`.
Ten je trochu delší; máš-li menší terminál, možná se tam ani nevejde.
V dnešních terminálech můžeš sice rolovat posuvníkem,
ale hodí se umět vypsat jenom začátek – hlavičku – souboru.
To se dělá příkazem `head`:

```console
$ head octane.pdb
COMPND      OCTANE
AUTHOR      DAVE WOODCOCK  96 01 05
ATOM      1  C           1      -4.397   0.370  -0.255  1.00  0.00
ATOM      2  C           1      -3.113  -0.447  -0.421  1.00  0.00
ATOM      3  C           1      -1.896   0.386  -0.007  1.00  0.00
ATOM      4  C           1      -0.611  -0.426  -0.198  1.00  0.00
ATOM      5  C           1       0.608   0.405   0.216  1.00  0.00
ATOM      6  C           1       1.892  -0.400   0.001  1.00  0.00
ATOM      7  C           1       3.113   0.429   0.414  1.00  0.00
ATOM      8  C           1       4.397  -0.374   0.199  1.00  0.00
```

Normálně `head` vypíše prvních 10 řádků, ale pomocí přepínače `-n`
můžeš zadat i jiný počet:

```console
$ head -n4 octane.pdb
COMPND      OCTANE
AUTHOR      DAVE WOODCOCK  96 01 05
ATOM      1  C           1      -4.397   0.370  -0.255  1.00  0.00
ATOM      2  C           1      -3.113  -0.447  -0.421  1.00  0.00
```

Záporné hodnoty fungují jako v Pythonu: s `-n -10` vypíše `head` všechny řádky
kromě deseti posledních.

Podobný příkaz je `tail`, který naopak vypíše poslední řádky ze souboru:

```console
$ tail -n 4 octane.pdb
ATOM     25  H           1       4.368  -1.282   0.801  1.00  0.00
ATOM     26  H           1       5.254   0.230   0.498  1.00  0.00
TER      27              1
END
```


## Méně je více

Příkazy `head` a `tail` jsou užitečné, ale někdy se potřebuješ v dlouhém
dokumentu trochu „porozhlédnout“.
K tomu slouží příkaz `less`, který ukáže obsah souboru podobným způsobem
jako `man`.
Klávesami <kbd>↑</kbd>/<kbd>↓</kbd>/<kbd>PgUp</kbd>/<kbd>PgDb</kbd>
se v souboru „pohybuj“ a pomocí <kbd>Q</kbd> se dostaneš ven.

```console
$ less octane.pdb
```

> [note] Proč „less“?
> Příkaz `less` vychází ze staršího `more` (více), nástroje pro terminály
> které neuměly „rolovat“ v historii.
> Po vypsání kousku textu zobrazil něco jako *Press Space for more*,
> „stiskni mezerník pro více [textu]“ a čekal na stisk klávesy.
>
> Jméno *Less* (méně) si jen hraje se rčením *méně je více*
> (less is more) – `less` je vylepšenou verzí `more`, která se např. umí
> v souboru vracet zpátky.


## Počítání řádků

Občas nepotřebuješ přímo obsah souboru, ale chceš vědět kolik obsahu v něm je.
Na to, abys to zjistil{{a}}, slouží příkaz `wc`.
Název je zkratka z anglického *word count*, počet slov; podobnost s evropskou
zkratkou WC (*water closet*) je čistě náhodná.

```console
$ wc octane.pdb
  30  246 1828 octane.pdb
```

Příkaz `wc` vypíše čtyři hodnoty: počet řádků, počet slov, počet bytů
(tedy velikost souboru) a jméno souboru.
Z nápovědy se dozvíš jak vybrat jen jednu z těchto hodnot (nebo víc jiných).
Asi nejužitečnější je počet řádků, který vybereš přepínačem `-l`:

```console
$ wc -l octane.pdb
30 octane.pdb
```

Když dáš příkazu `wc` víc souborů, vypíše informace o každém z nich:

```console
$ wc *.pdb
  20  156 1158 cubane.pdb
  12   84  622 ethane.pdb
   9   57  422 methane.pdb
  30  246 1828 octane.pdb
  21  165 1226 pentane.pdb
  15  111  825 propane.pdb
 107  819 6081 celkem
```

V posledním řádku pak vidíš součet všech položek.

Když naopak příkazu `wc` nezadáš žádná jména souborů, funguje podobně jako
`cat`: zpracuje vstup z klávesnice.
To už se může hodit: když si zkopíruješ nějaký text z editoru do schránky,
můžeš ji vložit „do“ spuštěného příkazu `wc`.

> [none]
> Dnešní editory mají funkci počítání slov zabudovanou.
> Je dokonce i v `nano` (pod <kbd>Alt</kbd>+<kbd>D</kbd>).
> Ale textová políčka třeba v prohlížeči bývají „hloupější“.

Tentokrát text neukončuj pomocí <kbd>Ctrl</kbd>+<kbd>C</kbd> – tím bys
příkaz `wc` ukončil{{a}}, takže by nedostal šanci vypsat výsledek.
Místo toho použij <kbd>Ctrl</kbd>+<kbd>D</kbd>, což znamená „konec vstupu“.

```console
$ wc
haló haló,
co se stalo?
      2       5      23
```
