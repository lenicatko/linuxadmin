# Další kombinace příkazů

Už víš, že pomocí „roury“ (`|`) se dá poslat výstup z jednoho příkazu
do druhého.
Existuje ale několik dalších příkazů, kterými se v Bashi dají příkazy
kombinovat.

Nejjednodušší je středník (`;`): Bash napřed provede příkaz na levé straně,
počká až se dokončí, a potom spustí ten na pravé.

```console
$ echo 'Vysledek je'; python -c 'print(1 + 1)'
Vysledek je
2
```

Docela užitečný operátor je `&&` (logické A; angl. *AND*): napřed se provede
příkaz na levé straně a počká se na výsledek.
Příkaz na pravé straně se provede pouze pokud první příkaz uspěl.
To že příkaz „uspěl“ znamená, že vrátil návratovou hodnotu 0.

```console
$ python -c 'print(1 + )' && echo 'je výsledek'
  File "<string>", line 1
    print(1 + )
              ^
SyntaxError: invalid syntax
```

Tenhle operátor se hodí např. v případech, kdy píšeš program který připraví
nějaký soubor. Příkaz `python priprav-soubor.py && head pripraveny-soubor.dat`
vypíše ukázku výsledku, ale jen když se všechno povedlo.
Když Python skončí s chybou, může existovat stará verze `pripraveny-soubor.dat`
z minulého spuštění.
Ale nevypíše se a na obrazovce zůstane chybová hláška od Pythonu.

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


## Spouštění na pozadí

A nakonec, když dáš za příkaz `&`, tak Bash nebude čekat na konec programu,
ale pokračuje hned dál.
Když program skončí, dá Bash před dalším promptem vědět.

Hezky se to zkouší na příkazu `sleep`, který čeká daný počet vteřin:

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
takže pokud bude něco psát na std. výstupů, bude se s Bashem „hádat“.

```console
$ curl https://httpbin.org/get &
[1] 25537
$ { ... sem curl vypíše jakási stažená JSON data ...  }
```

Programy běžící na pozadí můžeš vypsat pomocí `jobs`.
U každého vypíše malé číslo, pomocí kterého se taková „úloha“ (angl. *job*)
dá ovládat:

```console
$ sleep 200 &
[1] 34696
$ sleep 300 &
[2] 34701
$ sleep 400 &
[3] 34706
$ jobs
[1]   Běží                 sleep 20 &
[2]-  Běží                 sleep 30 &
[3]+  Běží                 sleep 40 &
```

Úloha nemusí být vždy jen jednotlivý proces:

```
$ sleep 500 && sleep 600 &
[4] 34846
$ jobs
[1]   Běží                 sleep 20 &
[2]   Běží                 sleep 30 &
[3]-  Běží                 sleep 40 &
[4]+  Běží                 sleep 4 && sleep 5 &
```

Úlohu který běží na pozadí lze popředí pomocí příkazu `fg`
(z angl. *foreground*, popředí);
kterému předáš číslo úlohy za procentem (`%`):

```console
$ fg %1
sleep 200
```

Úlohu běžící na popředí – tu, na kterou Bash právě čeká – můžeš
zastavit pomocí <kbd>Ctrl</kbd>+<kbd>Z</kbd>:

```console
^Z
[1]+  Pozastavena             sleep 200
$ jobs
[1]+  Pozastavena             sleep 200
[2]   Běží                 sleep 300 &
[3]   Běží                 sleep 400 &
[4]-  Běží                 sleep 500 && sleep 600 &
```

Pozastavená úloha neběží.
Můžeš ji ale odpauzovat buď příkazem `fg`, který ji dá na popředí (tj. Bash
bude čekat než se ukončí).
Nebo ji můžeš spustit na pozadí příkazem `bg` (z angl. *background*, pozadí),
Po tom bude úloha ve stejném stavu jako kdybys ji spustil{{a}} s `&`.

```console
$ bg %1
[1]+ sleep 200 &
$ jobs
[1]+  Běží                 sleep 200
[2]   Běží                 sleep 300 &
[3]   Běží                 sleep 400 &
[4]-  Běží                 sleep 500 && sleep 600 &
```

Argumentům s procentem rozumí i příkaz `kill`, který danou úlohu ukončí:

```console
$ kill %4
[4]+  Ukončen (SIGTERM)      sleep 500 && sleep 600
```

Když zadáš příkazu `fg` nebo `bg` bez argumentu, vybere „poslední“ úlohu.
Ta je ve výstupu `jobs` označena plusem.

Existují i další zkratky, např.:
* `%+` a `%%` vyberou „poslední“ úlohu – tu označenou plusem.
* `%-` vybere a „předposlední“ úlohu – tu označenou minusem.
* `%abcd` vybere úlohu spuštěnou příkazem začínajícím na `abcd` (pokud
  je spuštěna  právě jedna taková úloha).


