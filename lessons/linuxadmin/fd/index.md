# Soubory a deskriptory

Co je to soubor?

Lehce naivní ale přesto užitečná definice je: soubor je něco,
z čeho můžeme číst a/nebo do čeho můžeme zapisovat.

Se soubory se dají dělat i další operace než jen čtení a zápis, ale v tomto
kurzu se jimi většinou nebudeme zabývat.

Pravděpodobně máš největší zkušenosti se soubory, které jsou uložené na disku
pod nějakým jménem.
Třeba následující příkaz zapisuje do souboru `vystup.txt`:

```console
$ ps -Af > vystup.txt
```

Tento příkaz znamená: Bashi, spusť `ps -Af` a řekni mu aby psal do souboru
`vystup.txt`.
`ps` píše na svůj standardní výstup, což je soubor – v příkladu výše je to
soubor `vystup.txt`.

Kdybys výstup nepřesměroval{{a}}, uviděl{{a}} bys ho v terminálu.
Pro program `ps` se nic nemění: pořád píše na svůj standardní výstup,
což je *soubor*.

Je to soubor, který reprezentuje terminál – něco, do čeho může proces zapisovat
(a co se pak objeví na obrazovce) a z čeho může číst (když uživatel něco zadá).
Obsah tohoto souboru není uložen na disku, ale přesto jde o soubor.

Podívej se na tenhle příkaz:

```
$ ps -Af | grep -w ps
```

Příkaz `ps -Af` stále píše na svůj standardní výstup,
což je *soubor*.

V *tomhle* případě je to soubor, který reprezentuje „rouru“ – něco, do čeho
může jeden proces zapisovat a druhý to pak může číst.
Ani obsah roury není uložen na disku – „proudí“ přímo z jednoho procesu do
druhého – ale přesto jde o soubor.

Známe tedy už tři druhy souborů:

* normální souboru uložené na disku,
* terminál,
* rouru.


## Otevřené soubory procesu

Každý proces si může otevírat další soubory - to už znáš z funkce `open`
v Pythonu.
Podíváme se, jak zjistit seznam takto používaných souborů.
Existuje na to program `lsof` (z angl. *list open files*).
Většinou se spouští s přepínačem `-p <číslo procesu>`, aby ukázal jen
otevřené soubory jednoho procesu.

Použijme PID Bashe, které se jednoduše zjišťuje – je v proměnné `$$`:

```console
$ lsof -p $$
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
bash    5236 user  cwd    DIR    8,1     4096 1310722 /home/user
bash    5236 user  rtd    DIR    8,1     4096       2 /
bash    5236 user  txt    REG    8,1  1113504  524393 /bin/bash
bash    5236 user  mem    REG    8,1    47568 7344943 /lib/x86_64-linux-gnu/libnss_files-2.27.so
bash    5236 user  mem    REG    8,1    26376 6032184 /usr/lib/x86_64-linux-gnu/gconv/gconv-modules.cache
(...)
bash    5236 user    0u   CHR  136,0      0t0       3 /dev/pts/0
bash    5236 user    1u   CHR  136,0      0t0       3 /dev/pts/0
bash    5236 user    2u   CHR  136,0      0t0       3 /dev/pts/0
bash    5236 user  255u   CHR  136,0      0t0       3 /dev/pts/0
```

Jak vidíš, je to poměrně velká tabulka.
* `COMMAND` je jméno programu, který otvírá daný soubor
  (bez `-p` vypisuje `lsof` soubory pro víc programů najednou).
* `PID` je číslo procesu, náš starý známý.
* `USER` je uživatel, jehož jménem proces běží.
* `FD` je zkratka pro **deskriptor souboru** (angl. *file descriptor*),
  což je číslo otevřeného souboru. Toto číslo nás bude dnes nejvíc zajímat.

   * `cwd` je speciální hodnota pro aktuální adresář
     (*current working directory*) – právě tahle hodnota se dá v Bashi změnit
     pomocí `cd`
   * `rtd` je kořenový adresář (*root directory*) - mělo by to být `/`
   * `txt` je kód samotného programu. (Každý program musí být uložený někde
     na disku. Když ho pustíš jako proces, systém soubor načte a začne provádět
     příkazy v něm uložené.)
   * `mem` jsou soubory načtené v paměti, většinou další součásti programu.
   * `0` a dál jsou konečně normální otevřené soubory.
     Písmenka označují mód, např.:
     * `1r` - otevřeno pro čtení (angl. *read*)
     * `1w` - otevřeno pro zápis (angl. *write*)
     * `1u` - otevřeno „univerzálně“, pro čtení i zápis
* `TYPE` může být např.
    * `DIR` - adresář
    * `REG` - normální soubor
    * `CHR` - speciální soubor, např. terminál
* `DEVICE` - číslo zařízení (disku), na kterém soubor je
* `SIZE/OFF` - velikost / pozice v souboru
* `NODE` - číslo souboru (unikátní v rámci `DEVICE`)
* `NAME` - jméno souboru


## Terminál jako soubor

Soubor, který má Bash otevřený jako `0u`, `1u` i `2u`, je za normálních
okolností terminál, ve kterém Bash běží.
Zjisti z výstupu výše, který to je.
V našem příkladu je to `/dev/pts/0`, u tebe může být jméno jiné.

Jak už víš, terminál je soubor, do kterého můžeš zapisovat.
Otevři si další okno terminálu a zadej následující příkaz. (Za `/dev/pts/0` doplň „svoje“ jméno terminálu):

```console
$ echo abc > /dev/pts/0
```

Všimni si, že se `abc` objeví ve druhém terminálovém okénku!

Když znáš jméno souboru a víš, jak se do něj dostat (znáš cestu), můžeš do něj zapisovat. 
Soubor pro terminál není zas tolik zvláštní: jakmile znáš jméno, můžeš do něj zapisovat jako do jakéhokoli jiného souboru.

> [node] K čemu to je dobré?
> Svého času takhle administrátoři posílali zprávy dalším lidem,
> co zrovna pracovali na tom samém stroji.
> Ale dnes se to už tolik nepoužívá.
> Jen to ukazuje jak věci fungují – „všechno je soubor“.


### Procvičování v Pythonu

Procvičme si to trochu v Pythonu.
Budeš potřebovat textový editor a dva terminály.
V tomto textu jim budu říkat A a B.

> [note]
> Budeme používat Python, který je zabudovaný přímo v systému.
> Nevytvářej/neaktivuj si virtuální prostředí – pracuješ na virtuálním stroji,
> to úplně stačí.

V obou terminálech se přepni do adresáře,
do kterého ukládáš soubory pro dnešní lekci.
Jestli takový nemáš, vytvoř si ho.

Otevři si textový editor a následující kód si ulož do souboru
`soubory.py`:

```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# Pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

# Nechme program 600 vteřin (10 minut) čekat.
# To bude dost času na to, abychom mohli např. analyzovat otevřené soubory.
time.sleep(600)
```

A v terminálu A program spusť.

```console
$ python soubory.py
17342
```

Mělo by se ti vypsat číslo procesu (jiné, než v našem příkladu). 

V terminálu B pusť příkaz:
```console
$ lsof -p <číslo procesu z terminálu A>
COMMAND  PID USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
python  6503 user  cwd    DIR    8,1     4096 1310722 /home/user
python  6503 user  rtd    DIR    8,1     4096       2 /
python  6503 user  txt    REG    8,1  4873376 5773466 /usr/bin/python3.7
python  6503 user  mem    REG    8,1 11531024 5773943 /usr/lib/locale/locale-archive
python  6503 user  mem    REG    8,1  1700792 7344916 /lib/x86_64-linux-gnu/libm-2.27.so
python  6503 user  mem    REG    8,1   116960 7345025 /lib/x86_64-linux-gnu/libz.so.1.2.11
(...)
python  6503 user    0u   CHR  136,0      0t0       3 /dev/pts/0
python  6503 user    1u   CHR  136,0      0t0       3 /dev/pts/0
python  6503 user    2u   CHR  136,0      0t0       3 /dev/pts/0

```
Zkontroluj, jestli pod `txt` vidíš nějako verzi Pythonu.
Vidíš? Skvěle, pojďme o kousek dál.

V `soubory.py` změň poslední sekci – dopiš kus kódu, který otevře soubor:

```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os   
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

# Tentokrát čekáme s otevřeným souborem:
with open('soubory.py', encoding='utf-8') as soubor:
    time.sleep(600)
```
Po uložení souboru ukonči program v terminálu A (<kbd>CTRL</kbd>+<kbd>C</kbd>)
a spusť ho znovu. Vypíše se jiné číslo procesu.
Do terminálu B zadej *aktualizovaný* příkaz `lsof -p <číslo procesu>`.
Podívej se na poslední řádek výpisu. Vidíš tam něco nového?

```console
$ lsof -p 6604
(...)
python  6604 user    3r   REG    8,1      333 1314091 /home/user/bash/03/soubory.py
```

Měl by přibýt záznam o nově otevřeném souboru (`soubory.py`) s číslem `3r`.
Tři je další zatím nevyužité číslo (po 0, 1, a 2);
`r` indikuje soubor otevřený pro čtení.

Řekl{{a}} jsi Pythonu, aby otevřel pro čtení soubor s vlastním programem.
Protože příkaz `lsof` vypíše otevřené soubory procesu,
vidíš ve výsledcích nově výsledek této operace.

Zatím všechno v pohodě? Tak otevři jeden soubor pro čtení a nějaký jiný pro zápis. 
Změn svůj program na příklad níže.

```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os   
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

with open('soubory.py', encoding='utf-8') as soubor:
    with open('jiny.txt', mode='w', encoding='utf-8') as soubor2:
        print(soubor.fileno(), soubor2.fileno())
        time.sleep(600)
```

Kromě PID se teď vypíší výsledky metody `fileno`.
Jsou to deskriptory souborů – čísla která pro otevřené soubory používá systém,
a která budou vidět i ve výstupu `lsof`.

Ukonči běh předchozího programu v terminálu A (<kbd>CTRL</kbd>+<kbd>C</kbd>) a spusť soubor ještě jednou.
V terminálu B spusť aktualizovaný příkaz `lsof`. 
```console
$ lsof -p 6904
python  6904 user    0u   CHR  136,0      0t0       3 /dev/pts/0
python  6904 user    1u   CHR  136,0      0t0       3 /dev/pts/0
python  6904 user    2u   CHR  136,0      0t0       3 /dev/pts/0
python  6904 user    3r   REG    8,1      373 1314108 /home/user/bash/03/soubory.py
python  6904 user    4w   REG    8,1        0 1314053 /home/user/bash/03/jiny.txt
```
Výsledek doufám nepřekvapí.

### Deskriptory

Python nám v mnohém pomáhá, ale taky vzdaluje od systémové vrstvy.
Pythonní objekty které vrací funkce `open` nejsou přesně totéž jako způsob,
jakým soubory zpracovává operační systém. Dělají spoustu věcí navíc.

My se chceme podívat, jak funguje vevnitř operační systém, nikoliv Pythonní zlepšováky.
Proto si otevři tyto soubory ještě jednou pomocí modulu `os`, který umožňuje dělat věci trošku víc „přímo“.

Zaměň celý blok s `with` za tento kus kódu:
```python
fd1 = os.open("soubory.py", os.O_RDONLY)
fd2 = os.open("jiny.txt", os.O_WRONLY)
print(fd1, fd2)
time.sleep(600)
```

Takto upravený program spusť v terminálu A. 
Kromě nového čísla procesu bys měl(a) vidět na dalším řádku dvě čísla. U nás jsou to `3, 4`.

```console
$ python soubory.py
19257
3, 4
```

V terminálu B spusť opět příkaz `lsof` s novým číslem procesu a podívej se na dva poslední řádky.
Deskriptory otevřených souborů by měly opět odpovídat číslům z terminálu A.

> [note]
> Poznámka pro zvídavé: pod složitým `os.O_RDONLY` a `os.O_WRONLY` se
> skrývají jenom číselné konstanty `0` a `1`.
> Systémové operace (např. `open`) používají číselné konstant, kterým byla pro
> lepší čitelnost přiřazena krátká jména.

> [note]
> Pokud otevřeš pomocí pythonní funkce
> `open`  neexistující soubor pro zápis, funkce soubor vytvoří a otevře ho.
> Oproti tomu systémová funkce `os.open()` ho nevytvoří, ale zahlásí chybu.
> Pokud se ti to stane, můžeš společně s `os.O_WRONLY` použít `os.O_CREAT`.
> Kombinují se pomocí operátoru `|`:
>
> ```console
> os.open('jiny.txt', os.O_WRONLY | os.O_CREAT)
> ```
> Anebo můžeš soubor vytvořit v terminálu pomocí`touch jiny.txt`.


Přidej do pythonního souboru pod řádky, kde soubory otevíráš, tento řádek:
```python
print(os.read(fd1, 10))
```
Tento řádek načte prvních 10 bajtů ze souboru `soubory.py` a vypíše je do terminálu. 
V terminálu A spusť `soubory.py`.
V našem případě výstup vypadá takto:
```console
$ python soubory.py
20019
b'# soubory.'
3, 4
```

Do souboru můžeš i něco napsat.
Pozor, to co zapisuješ (a čteš) není Pythonní řetězec (text), ale bajty.
Když se ale omezíš na anglickou abecedu, hlavní rozdíl mezi nimi je zápis
s `b` na začátku.

Přidej za předchozí řádek tento kód, který do `jiny.txt` zapíše 4 písmenka.

```python
os.write(fd2, b'abcd\n') 
```

Nezapomeň na konec dát `\n`, nový řádek, aby se pak text hezky vypisoval.
Když teď spustíš program v terminálu A a v terminálu B vypíšeš obsah souboru `jiny.txt` pomocí programu `cat`, měl by se ti zobrazit text "abcd".
```console
$ cat jiny.txt 
abcd
```

Jak víš z kurzu Pythonu, otevřené soubory je dobré vždy na konci manipulace zavřít.
Tak to pojď udělat: na konec, před `time.sleep`, dopiš:

```python
os.close(fd1)
os.close(fd2)
```


### Standardní deskriptory

Celou dobu pracuješ se soubory otevřenými pythonním programem, tedy `3` a `4`. 
Určitě tě zajímá, co jsou vlastně soubory označení `0`, `1`, `2`.

V textovém editoru smaž `time.sleep` a místo toho dopiš řádky:

```python
os.write(1, b'Tohle jde do souboru 1\n')
os.write(2, b'Tohle jde do souboru 2\n')
```

Kam se to vypíše?

{% filter solution %}
Všechno se to vypíše do terminálu, ve kterém program spouštíš.

```console
$ python soubory.py 
7102
3 4
b'# soubory.'
Tohle jde do souboru 1
Tohle jde do souboru 2
```
{% endfilter %}

Funguje to?
Tak zkus přesměrovat výstup `soubory.py` do `jiny.txt`:
```console
$ python soubory.py > jiny.txt
Tohle jde do souboru 2
$ cat jiny.txt 
Tohle jde do souboru 1
7167
3 4
b'## soubory'
```
Když se podíváš do souboru `jiny.txt`, najdeš v něm text *Tohle jde do souboru 1*.

Pokud to tam máš, připiš do programu ještě poslední řádek.

```python
print(os.read(0, 10))
```

Spusť ho v terminálu A bez přesměrování.

```console
$ python soubory.py
```

Tdyž teď něco napíšeš do terminálu, Python vypíše prvních 10 bajtů tvého textu.

Co se tady děje?

Tyto tři soubory, `0`, `1` a `2`, má každý proces otevřené.

`0` je náš starý známý *standardní vstup*.
Když nepřesměrováváš, je to terminál: ťe se to co napíšeš na klávesnici.
Ale může to být i jiný soubor – třeba následující `grep` má pod
číslem 0 otevřený soubor `soubory.py`:

```console
grep print < soubory.py
```

`1` je standardní výstup – místo, kam program píše informace,
které chce předat světu.

### Chybový výstup

`2` je ale nové: je to standardní *chybový* výstup
(angl. *standard error stream*, *stderr*), místo,
kam programy píší chybové hlášky.
Často to bývá stejný terminál jako *stdout*, ale přesměrovává se samostatně.

Zkus si třeba zkopírovat neexistující soubor pomocí `cp`.
Dostaneš chybovou hlášku:

```console
$ cp a b
cp: cannot stat a no such file or dir
```

Když přesměruješ standardní výstup do souboru, chybová hláška se přesto objeví
v terminálu. Zkus:

```console
$ cp a b > jiny.txt
cp: cannot stat a no such file or dir
```

Zobáček `>` přesměrovává pouze standardní výstup, zatímco chybový výstup
nechá nepřesměrovaný.

A k čemu je to užitečné? 

Na standardní výstup programy většinou píšou výsledky své práce v nějakém
formátu, který se pak dá automaticky zpracovat. 
Když nastane chyba, program ji ohlásí, ale ohlásí ji na jiném místě, než kam
vypíše výstup, aby nekazil další zpracování.

Když zadáš `ps -A | grep ps`, tak `grep` zpracovává výstup z příkazu `ps`:

```console
$  ps -A | grep ps
    776 ?        00:00:00 psimon
    840 ?        00:00:00 cupsd
   5431 pts/1    00:00:00 ps
```

Když ale uděláš chybu a vynecháš pomlčku před `A`,
tak se chyba vypíše na terminál a `grep` zpracuje jen prázdný soubor:

```console
$ ps -A | grep top
[petr@fedora ~]$ ps A | grep top
error: unsupported option (BSD syntax)
(...)
```


### Přesměrovat všechno

Vraťme se na chvíli k Pythonu.

V terminálu A spusť aktuální program.

```console
$ python soubory.py
7305
3 4
b'## soubory'
Tohle jde do souboru 1
Tohle jde do souboru 2
```

Měly by se ti vypsat všechny tyto informace na terminál. 

A co se stane, když přesměruješ výstup programu a pak zmáčkneš <kbd>Ctrl</kbd>+<kbd>C</kbd>?
```console
$ python soubory.py > jiny.txt
Tohle jde do souboru 2
Traceback (most recent call last):
  File "soubory.py", line 21, in <module>
    print(os.read(0, 10))
KeyboardInterrupt
```

Výstup – všechno kromě chybového výstupu – je v souboru `jiny.txt`.
Chybová hláška ale jde do chybového výstupu, tedy stále do terminálu.

Co kdybys ale přece jen chtěl{{a}} přesměrovat ten druhý, chybový, výstup?
Existuje na to speciální operátor `2>` - tedy přesměrování souboru číslo 2.
```console
$ python soubor.py 2> jiny.txt
7388
3 4
b'## soubory'
Tohle jde do souboru 1
^C
$ cat jiny.txt 
Tohle jde do souboru 2
Traceback (most recent call last):
  File "soubory.py", line 21, in <module>
    print(os.read(0, 10))
KeyboardInterrupt
```

#### Přesměrování obojího

Můžeš přesměrovat i oba výstupy:

```console
$ python soubory.py > vystup.txt 2> chyby.txt
^C$ cat vystup.txt 
Tohle jde do souboru 1
7423
3 4
b'## soubory'
$ cat chyby.txt 
Tohle jde do souboru 2
Traceback (most recent call last):
  File "soubory.py", line 21, in <module>
    print(os.read(0, 10))
KeyboardInterrupt
```

Když přesměrováváš do různých souborů, tak nezáleží na pořadí `>` a `2>`.

Když ale použiješ dvakrát stejný soubor (např. `> jiny.txt 2> jiny.txt`),
narazíš na problém: v souboru se většinou objeví jen jeden z výsledků.
Když je jeden soubor otevřený pro čtení dvakrát, a zapisuje se do obou
deskriptorů zároveň, obvykle „vyhraje“ jen jeden.

Bash má na řešení této situace speciální operátor:

```console
$python soubory.py > jiny.txt 2>&1
#                               ^--- chybový směruje tam, kam první
```

Kdyby v příkazu nebyl `&`, výstup se přesměruje do souboru s názvem `1`.
`&1` ale „odkazuje“ na deskriptor 1, tedy `jiny.txt`.

Tady už záleží v jakém pořadí se skládají příkazy, protože tohle fungovat nebude:

```
#                  ,-------- 1. přesměruje stderr na stdout, tedy na terminál
#                  |    ,--- 2. přesměruje stdout do souboru
#                  ↓    ↓
$python soubory.py 2>&1 > jiny
```

Proto *stderr* půjde na terminál a *stdout* do souboru.


### Přesměrování vstupu

Pro úplnost si ukážeme i přesměrování standardnho vstupu.

```console
$ python soubory.py < jiny.txt
7481
3 4
b'## soubory'
Tohle jde do souboru 1
Tohle jde do souboru 2
b'abcd\n jde '
```

Poslední řádek je vstup načtený ze souboru `jiny.txt`.
Podívej se na soubor 0 ve výpisu `lsof` – standardní vstup je nastaven na
„opravdový“ soubor na disku.

```console
$ lsof -p 7481
...
python  7481 user 0r REG 253,0 150 299463294 /home/user/bash/03/jiny.txt
...
```

Když data „přitečou“ rourou, bude standardní vstup vypadat ještě trochu jinak.
```console
$ cat jiny.txt | python soubory.py
7485
3 4
b'## soubory'
Tohle jde do souboru 1
Tohle jde do souboru 2
b'abcd\n jde '
```

Podívej se na výpis `lsof`, na soubor číslo 0:

```console
$ lsof -p 7485
...
python  7485 user 0r FIFO 0,12 0t0 1352745 pipe
...
```

Ono `pipe` znamená *roura*.
Stejně jako terminál (např. `/dev/pts/0`) není obsah roury uložený na disku
(data „proudí“ přímo z jednoho procesu do druhého).
Na rozdíl od terminálu ale roura ani nemá jméno: nemůžeš udělat
`echo Ahoj > /dev/pts/0` jako u terminálu.
S rourou může pracovat jen proces, který už ji má k dispozici.
V tomto případě rouru vytvořil Bash, jeden její konec dal procesu `cat`
a druhý procesu `python`.

> [note] Odbočka pro zvídavé
> Přesměrování funguje i s jinými čísly než 2, a to i se vstupem.
> Například:
> ```
> console $ python soubory.py 8< jiny.txt 
> ```
> 
> Soubor `jiny.txt` se takto předá Pythonu na zpracování jako `8r` (když
> se podíváš do `lsof`).
> Dá se pak přečíst např. pomocí `os.read(8, 10)`.
> Jen čísla 0, 1, 2 mají určený význam, ostatní můžeš používat dle libosti.
> Jen pozor že `open` nebo `os.open` si nějaké číslo „zamluví“ pro sebe.


## Zahození výstupu

Zvláštní případ použití je přesměrování některého výstupu do speciálního
souboru, který si nic z toho co se do něj píše, neukládá.
Jmenuje se `/dev/null`.

```console
$ python soubory.py 2> /dev/null
```

Celý chybový výstup se tak „vyhodí“.

> [note]
> `/dev/null` není ani normální soubor, ani terminál, ani roura.
> Je to další druh speciálního souboru.

Jiný příklad - jsou programy, které píšou do terminálu spoustu výstupu.
Třeba `find`.
Pokud chceš vidět jen problémy, ale ne nalezené soubory, můžeš přesměrovat
standardní výstup do `/dev/null` a v terminálu uvidíš jen chybová hlášení
– tady o tom, že k některým souborům nemáš přístup:

```console
$ find /var/cache > /dev/null
find: … Permision denied
```

Nebo naopak můžeš mít tolik souborů k nimž nemáš přístup,
že se ti ani nebude chtít o tom číst; chceš jen dostat ty ke kterým přístup máš.
Pak si do `/dev/null` přesměruješ chybový výstup:

```console
$ find /var/cache 2> /dev/null
```
