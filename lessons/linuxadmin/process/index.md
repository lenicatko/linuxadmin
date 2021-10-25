# Procesy

**Proces** je termín pro **běžící program**.
Proces na počítači:

* provádí akce
* pracuje na datch
* může ukazovat výsledky své činnosti v okýnkách nebo na příkazové řádce.

Každý běžící program má svůj proces.
Když si otevřeš několik terminálů, vytvoří se několik procesů pro Bash.
Každý má svůj vlastní stav, svůj aktuální adresář apod.


## Seznam procesů: ps

Jaké aktuálně běží ti řekne příkaz `ps`:

```console
$ ps
  PID TTY          TIME CMD
 5403 pts/8    00:00:00 bash
 5449 pts/8    00:00:00 ps
```

Tenhle výstup je až překvapivě krátký.
Program `ps` totiž v základu ukazuje pouze procesy běžící v aktuálním terminálu.
S parametrem `-a` se jich vypíše víc, ale až s parametrem `-A` dostaneš
všechny aktuálně běžící procesy v systému.
Většinou je jich hodně (zvlášť v našem grafickém prostředí):

```console
$ ps -A
  PID TTY          TIME CMD
 1211 tty1     00:05:53 Xorg
 1413 tty1     00:02:16 gnome-shell
 1510 tty1     00:00:00 ibus-x11
 1740 tty1     00:00:00 gsd-disk-utilit
 1747 tty1     00:00:00 python3
 2394 tty1     00:19:03 firefox
 2451 tty1     00:01:52 Web Content
 (...)
```

Výpis je uveden v textovém formátu, takže s ním můžeš dál pracovat pomocí
nástrojů které už znáš: `cut`, `grep` atd.

Občas se ti bude hodit více informací, které `ps` vypíše s přepínač `-f`.
Mimojiné se takhle ve výstupu objeví argumenty každého procesu, např.
`-Af` u samotné ho `ps -Af`:

```console
$ ps -Af
 (...)
petr        4175    3767  0 14:18 pts/1    00:00:00 ps -Af
petr        4176    3767  0 14:18 pts/1    00:00:00 tail
```

> [note]
> Příkaz `ps` přidal přepínače s pomlčkou (`-a`, `-A`) relativně nedávno.
> Přepínače bez pomlček jsou ale dál k dispozici, takže občas v návodech
> a příkladech najdeš `ps a` nebo `ps aux`. Pozor, jde o jiné přepínače:
> `a` neznamená nutně totéž co `-a`.
> V manuálové stránce jsou popsány oba.

Hezčí výstup dostaneš, když použiješ program `top`:

```console
$ top
```

Ten ti ukáže tabulku podobnou příkladu níže, která se pravidelně aktualizuje:

```
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
 3267 hanka     20   0 2957412 321036 146352 S   2,3  4,1  30:39.88 Web Content
 1392 hanka     20   0 3726628 295760 145672 S   2,0  3,8  17:12.48 gnome-shell
 1108 hanka     20   0  714448 196972 164264 S   1,0  2,5  22:04.77 Xorg
 3053 hanka     20   0 3489236 429100 167936 S   1,0  5,5  15:53.94 Web Content
 (...)
```

Je to výpis procesů, které se vejdou na obrazovku.
Jsou seřazeny podle podle vytížení procesoru (t.j. kolik zabírají prostředků),
takže neaktivní procesy jsou níže (nebo nejsou vůbec vidět) a na prvních
řádcích bývají typicky grafické aplikace (zvlášť prohlížeče).

Asi tam budeš mít mimojiné grafický shell – `gnome-shell`, terminál a
samotný `top`.
Bash ale u začátku nejspíš nebude – ten teď nedělá nic, jen čeká až skončí
`top`. Můžeš ho zkusit najít pomocí kláves <kbd>PgUp</kbd>/<kbd>PgDn</kbd>.


Co znamenají jednotlivé sloupce ve výpisu `top`?
1. `PID` je **číslo procesu** (angl. *Process ID*) – unikátní číslo.
   Když ho znáš, můžeš pomocí něj proces ovládat – o tom za chvíli.
2. `USER` je **vlastník**
   Každý proces patří určitému uživateli, jehož jménem běží.
   To se používá např. pro zjištění jestli má proces právo otevřít soubor,
   který patří danému uživateli.
   Každý uživatel na počítači má svůj vlastní uživatelský účet.
   Na osobních bývá jen jeden; na univerzitním serveru jich budou klidně tisíce.
   Kromě toho na všech linuxových počítačích existuje i hlavní systémový
   administrátor (`root`, jehož procesy mají plný přístup k celému systému)
   a celá řada účtů pro systémové služby.
3. `PR` a `NI` ukazují *prioritu* a jak je proces “hodný” (angl. *nice*)
   k ostatním.
   Když je proces k ostatním “hodný” (má tu vyšší číslo), tak systém
   upřednostňuje ostatní procesy.
   “Hodný” program pak běží, když není co jiného dělat.

   Hodnotu *nice* lze nastavovat; prioritu nastavuje jádro systému samo
   podle *nice* (a jiných proměnných).

5. `VIRT`,  `RES`, `SHR` popisují kolik proces zabírá paměti.
   (Počítání paměti je relativně složité, ale vyšší čísla znamenají “větší”
   proces.)
6. `S` je stav procesu. Ten může být běžící (`R`, *running*),
   spící (`S`, *sleeping*, např. čeká na vstup z klávesnice),
   čekající na data z disku (`D`) a další.
7. `%CPU` je procentuální vytížení procesoru.
   (U vícejádrových počítačů může být součet vyšší než 100%.)
8. `%MEM` je procentuální využití paměti.
   (Počítání paměti je relativně složité; tohle číslo je třeba brát s rezervou.)
9. `TIME` říká jak dlouho tento proces už běží.
   Počítá se ale jen *procesorový čas*: když proces spí, tak se čas nenavyšuje.
   Pro příklad: Bash, který většinou času jen čeká na zadání příkazu nebo
   dokončení spuštěného procesu, nebude mít tento čas moc velký.
   U prohlížeče, který typicky běží dlouho a neustále něco překresluje
   a stahuje, bude čas větší.
10. `COMMAND` je jméno procesu – příkaz kterým byl proces spuštěn.

`top` ukončíš zmáčknutím klávesy <kbd>Q</kbd>.

Stejné informace umí ukázat i grafický program `gnome-system-monitor`
(*Sledování systému*):

```
$ gnome-system-monitor
```


## Ovládání procesů

PID, číslo procesu, je velice důležitý údaj.
Je unikátní a proces se s jeho pomocí dá ovládat.

Pokud jsi už `top` ukončil{{a}}, opět ho spusť.
Zkus najít jeho číslo procesu (PID).

Budeš na to nejspíš potřebovat druhý terminál – v jednom ti běží `top`,
ve druhém budeš hledat PID.
Nové okno otevřít z menu *Terminál* nebo pomocí
<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>N</kbd>.
(Případně můžeš otevřít novou kartu, podobně jako v prohlížečích,
přes grafické tlačítko nebo <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>T</kbd>.)

Vzpomeň si na příkaz `ps -A`.
Jak z něj najdeš číslo procesu (PID) programu `top`?

{% filter solution %}
Ve druhém terminálu spusť:
```
$ ps -A | grep top
   3186 pts/0    00:00:06 top
```

První sloupec je PID.

Když použiješ rozšířený výpis, objeví se ti i `grep top`;
správný proces můžeš vybrat očima. PID je teď ve druhém sloupci:

```
$ ps -Af | grep top
hanka       3186    2494  0 13:31 pts/0    00:00:06 top
hanka       4291    3767  0 14:23 pts/1    00:00:00 grep --color=auto top
```

{% endfilter %}

Podle PID se dá proces ovládat.
Například ho ukončit, když předáš číslo jako argument příkazu `kill`:

```
$ kill <číslo procesu>
```

Zkontroluj na druhém terminálu, že se `top` opravdu ukončil a vidíš opět prompt.

> [note]
> Pokud špatně opíšeš číslo procesu, ukončíš příkazem `kill` nějaký jiný proces.
> Může to mít nečekané následky.


Příkaz `kill` není tak “drsný” jak se podle jména zdá:
pošle procesu `top` signál, který “hezky poprosí” aby se ukončil.
Funguje to podobně jako <kbd>CTRL</kbd>+<kbd>C</kbd>.
`top` tak má možnost uvést terminál do původního stavu a např. zavřít případné
otevřené soubory.
Některé procesy ale můžou tuhle “prosbu” úplně ignorovat;
jak ukončit takové procesy (a co to znamená “signál” se dozvíme později.

> [note]
> Některé grafické programy (např. Firefox) spouští víc procesů. 
> Pokud se pokusíš ukončit příkazem `kill` jeden z nich, nejspíš neuvidíš
> žádný rozdíl. Proces naběhne znovu. Tyto programy radši vypínej tlačítkem :)


## PID Bashe

PID aktuálního shellu je uloženo v proměnné `$$` (podobně jako `$?` obsahuje
návratovou hodnotu posledního příkazu.)

Ověř si to: najdi PID pro `bash` stejně jako jsi to udělal{{a}} pro `top`
a porovnej ho s hodnotou `$$`:

```console
$ ps -A | grep bash
   3767 pts/1    00:00:00 bash
$ echo $$
3767
```

Co se ale stane, když se Bash pokusíš ukončit?

```console
$ kill $$
$
```

Příkaz `kill` není tak “drsný” jak se podle jména zdá:
pošle procesu `top` signál, který “hezky poprosí” aby se ukončil.
Funguje to podobně jako <kbd>CTRL</kbd>+<kbd>C</kbd>.
`top` tak má možnost uvést terminál do původního stavu a např. zavřít případné
otevřené soubory.
Některé procesy ale můžou tuhle “prosbu” úplně ignorovat – a Bash je právě
jeden z nich.
Ostatně ani <kbd>CTRL</kbd>+<kbd>C</kbd> Bash neukončí,
zruší jen aktuálně zadávaný příkaz.

I takovéhle procesy ale jdou ukončit. Když použiješ `kill` se speciálním
přepínačem, tak se na nic neptá a proces “natvrdo” ukončí.

> [warning]
> Toto je trochu nebezpečná operace: program nedostane možnost po sobě
> “uklidit” a může po sobě něco nechat v nekonzistentním stavu.
> Proto používej `kill -9` jen když jiné způsoby nefungují.
> (A nebo teď, ve virtuálním stroji, na zkoušku.)
