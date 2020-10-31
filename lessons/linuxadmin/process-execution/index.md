# Spouštění procesů

Pojďme se podívat na to, co přesně Bash dělá, když spustí program.

Příkaz `ps` bez argumentů vypíše jen ty procesy, které „patří“ k aktuálnímu
terminálu:

```console
$ ps
    PID TTY          TIME CMD
   2328 pts/0    00:00:00 bash
  14509 pts/0    00:00:00 ps
```

Vidíš, že běží dva programy: jak `bash` tak `ps`.

Co když přesměruješ data z jednoho příkazu do druhého?

```console
$ ps | cat
    PID TTY          TIME CMD
   2328 pts/0    00:00:00 bash
  14661 pts/0    00:00:00 ps
  14662 pts/0    00:00:00 cat
```

```console
$ ps | cat | sort
    PID TTY          TIME CMD
  14679 pts/0    00:00:00 ps
  14680 pts/0    00:00:00 cat
  14681 pts/0    00:00:00 sort
   2328 pts/0    00:00:00 bash
```

```console
$ echo | ps
    PID TTY          TIME CMD
   2328 pts/0    00:00:00 bash
  14689 pts/0    00:00:00 ps
```


## Nastavení, spuštění a čekání

Když Bash spouští program, udělá tři věci:
* Nastaví prostředí – přesměrovaný vstup/výstup, proměnné prostředí apod.;
* program spustí;
* čeká, až se program ukončí.

Pokud Bash spouští několik programů najednou, udělá obecně každý z těchto
kroků pro všechny spouštěné programy najednou.

Příklady:

### `ps`

* Prostředí je nastavené – std. vstup i výstup jdou na stejné místo jako
  u Bashe.
* Spustí se program `ps`.
* Čeká se, než se `ps` ukončí.
  (Program `ps` mezitím něco vypíše na std. výstup.)

Po ukončení Bash vypíše výzvu a čeká na další příkaz.

  {{ figure(
    img=static('ps.svg'),
    alt='Diagram spouštění `ps`',
  ) }}

### `ps | cat`

* Bash vytvoří novou rouru. Std. výstup z `ps` a std. vstup pro `cat`
  přesměruje na tuto rouru.
  (Zbylé – std. chyby, vstup pr `ps` i výstup z `cat` – nechá jako u Bashe.)
* Spustí programy `ps` i `cat`.
* Čeká, než se `ps` i `cat` ukončí.
  (Programy mezitím spolu komunikují nebo píší na std. výstup.)

  {{ figure(
    img=static('ps-cat.svg'),
    alt='Diagram spouštění `ps | cat`',
  ) }}

### `ps | cat | sort`

Příkazů propojených rourami může být více;
Bash skládá za sebe stejným spůsobem.

  {{ figure(
    img=static('ps-cat-sort.svg'),
    alt='Diagram spouštění `ps | cat | sort`',
  ) }}

### `echo | ps`

Z pohledu Bashe je `echo | ps` úplně stejné jako `ps | cat`.
Příkaz `echo` se ale ukončí hned poté co vypíše svůj řádek, což je většinou
velice rychlé – stihne to dřív než se `ps` zeptá systému na běžící programy.
Proto `echo` většinou není ve výstupu vidět.:

```console
$ echo | ps
    PID TTY          TIME CMD
   2328 pts/0    00:00:00 bash
  14689 pts/0    00:00:00 ps
```

  {{ figure(
    img=static('echo-ps.svg'),
    alt='Diagram spouštění `echo | ps`',
  ) }}


## Další kombinace příkazů

Kromě `|` se příkazy dají kombinovat několika dalšími operátory.

Nejjednodušší je středník (`;`): napřed se provede příkaz na levé straně,
potom ten na pravé.

```console
$ echo 'Vysledek je'; python -c 'print(1 + 1)'
Vysledek je
2
```

Docela užitečné je `&&` (logické A; angl. *AND*): napřed se provede příkaz
na levé straně, a pouze pokud uspěl (`$?` je 0), se provede ten na pravé.

```console
$ python -c 'print(1 + )' && echo 'je výsledek'
  File "<string>", line 1
    print(1 + )
              ^
SyntaxError: invalid syntax
```

Tenhle operátor se hodí např. v případech, kdy píšeš program, který připraví
nějaký soubor. Příkaz `python priprav-soubor.py && head pripraveny-soubor.dat`
vypíše ukázku výsledku, ale jen když se všechno povedlo.
Když Python skončí s chybou, `pripraveny-soubor.dat` (existující od minula)
se nevypíše – na obrazovce zůstane chybová hláška od Pythonu.

Opak je `||` (logické NEBO; angl. *OR*): příkaz na pravé straně se provede,
pokud příkaz nalevo *neuspěl* (`$?` není 0).

```console
$ python -c 'print(1 + )' || echo 'Jejda! Chyba!'
  File "<string>", line 1
    print(1 + )
              ^
SyntaxError: invalid syntax
Jejda! Chyba!
```

A nakonec, když dáš za příkaz `&`, tak Bash nebude čekat na konec programu,
ale pokračuje hned dál.
Když program skončí, dá Bash před dalším promptem vědět:

```console
$ sleep 1
$ sleep 1 &
[1] 25507
$
$ # vteřinu počkej...
[1]+  Dokončena              sleep 1
$
```

Pozor na to, že program „v pozadí“ sdílí s Bashem terminál,
takže pokud bude něco psát (nebo hůř, číst) se std. vstupu/výstupů,
bude se s Bashem „hádat“.

```console
$ curl https://httpbin.org/get &
[1] 25537
$ { ... sem curl vypíše jakási stažená JSON data ...  }
```

