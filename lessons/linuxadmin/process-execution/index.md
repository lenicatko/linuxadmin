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
Ve výstupu `ps` uvidíš všechny spuštěné procesy:

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

Všimni si, že v posledním příkladu chybí `echo`.
Proč? I k tomu se dnes dostaneme.


## Nastavení, spuštění a čekání

Když Bash spouští program, udělá tři věci:
* Nastaví prostředí – přesměrovaný vstup/výstup, proměnné prostředí apod.;
* program spustí;
* čeká, až se program ukončí.

Pokud Bash spouští několik programů najednou, udělá obecně každý z těchto
kroků pro všechny spouštěné programy najednou:
napřed všem programům nastaví prostředí, potom teprve všechny spustí,
a pak na všechny počká.

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
  (Zbylé – std. chyby, vstup pro `ps` i výstup z `cat` – nechá jako u Bashe.)
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

Z pohledu spouštění je `echo | ps` úplně stejné jako `ps | cat`.
Příkaz `echo` se ale ukončí hned poté co vypíše svůj řádek, což je většinou
velice rychlé – zvládne to ještě dřív než se `ps` stihne zeptat systému na
běžící programy.
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


## Rolovátko

Víš, proč se třeba `man` nebo `git diff` chovají a ovládají stejně jako `less`?
Proto, že samy spustí `less` a pošlou do něho rourou text,
který chtějí zobrazit.

  {{ figure(
    img=static('man-less.svg'),
    alt='Diagram spouštění `less` z `man`',
  ) }}

Podobné programy většinou respektují proměnnou `PAGER`,
kterou můžeš vybrat program, který se použije místo `less`.
Zkus si `more`, starší verzi `less` která umí <kbd>Enter</kbd>,
<kbd>Mezerník</kbd> a <kbd>q</kbd>, ale neumí se vracet (<kbd>↑</kbd>):

```console
$ PAGER=more man bash
```

Nebo rolování vypni úplně pomocí `cat`:

```console
$ PAGER=cat man bash
```
Nebo jakýkoli jiný filtr – nikdo ti nezakazuje zadávat nesmysly:

```console
$ PAGER=wc   man bash
$ PAGER=tac  man bash
$ PAGER=rev  man bash
```
