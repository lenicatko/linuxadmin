
# Hledání

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

(Jsou to básničky ze [soutěže časopisu Salon](http://wiki.c2.com/?ComputerErrorHaiku).)

## grep
Náš úkol je podívat se na řádky, kde se nachází slovo *not*.
Existuje program `grep`, kterému předáš hledané slovo a soubor(y) k prohledání
– a on řádky s daným slovem vypíše.

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
- `-w`: Hledá jen celá slova.
- `-F`: Hledá přesně zadaný řetězec (viz níže).

Takhle můžeš najít všechny řádky se slovem *the* nebo *The*:

```console
$ grep -i the haiku.txt 
The Tao that is seen
Is not the true Tao, until
and the presence of absence:
"My Thesis" not found.
```

Jejda, tam je ale něco navíc!
Jak příkazem odstraníš z výpisu řádek se slovem *Thesis*?

```console
$ grep -iw the haiku.txt 
The Tao that is seen
Is not the true Tao, until
and the presence of absence:
```

`grep` je velice užitečný program. Pokud pracuješ s Bashem, brzy se z něj stane tvůj nejlepší kamarád. 
Pozor ale na to, co `grep` bere jako hledaný řetězec: je to totiž regulární výraz (angl. *regular expression*, *regex*).
Pokud hledáš jen písmenka, nenarazíš na problém, ale u znaků jako tečky, hvězdičky, otazníky apod. může být výstup jiný než očekáváš.

Zkus několik příkazů:
```console
$ grep -i the.i haiku.txt 
$ grep -i '.*' haiku.txt
$ grep -iw '.....' haiku.txt
```

Regulární výrazy jsou velice užitečné, ale taky jsou nad rámec tohoto kurzu
(nebo aspoň této lekce).
Zatím si tedy zapamatuj, že speciální funkce znaků můžeš vypnout pomocí
přepínače `-F` (aby je nezpracoval `grep`) a uzavřením do jednoduchých
uvozovek (aby je nezpracoval Bash).


## find

Příkaz `find` je další užitečný hledací příkaz.

```console
$ find . -name '*.txt'
./haiku.txt
./data/LittleWomen.txt
./data/two.txt
./data/one.txt
```

Příklad výše hledá v aktuálním adresáři (`.`) a všech podadresářích soubory,
které odpovídají masce `*.txt`.

> [note]
> Proč je `'*.txt'` v uvozovkách?
> Vzpomeň si, že znak `*` zpracovává samotný Bash: kdybys `*.txt` zadal{{a}}
> bez uvozovek, `find` by dostal jako argumenty všechny soubory `*.txt`
> z aktuálního adresáře.
> Uvozovky říkají Bashi, aby příkazu `find` předal opravdu `*.txt`.
> `find` tak může odpovídající soubory hledat sám – i v podadresářích.

Příkaz `find` má kromě `-name` spoustu jiných přepínačů.
Až je budeš potřebovat, najdeš je v manuálové stránce.

Podobného výsledku jako výše (ale bez jména adresáře) dosáhneš když zadáš:

```console
$ ls --recursive | grep txt
haiku.txt
LittleWomen.txt
one.txt
two.txt
```

I bez jména adresáře se takový výstup občas hodí – třeba když chceš spočítat,
kolik těch souborů je:

```console
$ ls --recursive | grep txt | wc -l
4
```
