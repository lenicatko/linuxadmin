
# 3. Linuxová administrace - Procesy a soubory

## Grafické aplikace

Grafické programy se dají spustit nejen z hlavní nabídky aplikací, ale taky z příkazové řádky. 
Pojďme si to vyzkoušet.

Otevři si příkazovou řádku a napiš do ní některý z následujících příkazů:

```console
$ firefox
```
nebo
```console
$ gedit
```
nebo taky
```console
$ gnome-terminal
``` 

Takto jsi spustil{{a}} program. Můžeš mu předat určité parametry navíc, jak to již znáš z negrafických programů. Například:
```console
$ firefox https://github.com
```

Co všechno se dá takhle nastavit? To ti řekne nápověda:
```console
$ firefox --help
Usage: /usr/lib64/firefox/firefox [ options ... ] [URL]
       where options include:

X11 options
  --display=DISPLAY  X display to use
  --sync             Make X calls synchronous
  --g-fatal-warnings Make all warnings fatal

Firefox options
  -h or --help       Print this message.
  -v or --version    Print Firefox version.
  -P <profile>       Start with <profile>.
  --profile <path>   Start with profile at <path>.
  --migration        Start with migration wizard.
  --ProfileManager   Start with ProfileManager.
  --no-remote        Do not accept or send remote commands; implies
                     --new-instance.
  --new-instance     Open new instance, not a new window in running instance.
  --UILocale <locale> Start with <locale> resources as UI Locale.
  --safe-mode        Disables extensions and themes for this session.
  -MOZ_LOG=<modules> Treated as MOZ_LOG=<modules> environment variable, overrides it.
  -MOZ_LOG_FILE=<file> Treated as MOZ_LOG_FILE=<file> environment variable, overrides it.
                     If MOZ_LOG_FILE is not specified as an argument or as an environment variable,
                     logging will be written to stdout.
  --headless         Run without a GUI.
(...)
```

Proč bys měl(a) spouštět aplikaci jménem z příkazové řádky, když na ni můžeš kliknout v okénkovém prostředí? 
Hodí se to pro účely automatizace.
Pokud znáš jméno aplikace a možnosti jejího rozhraní, můžeš si napsat Bashový skript, který udělá kus únavné rutinní práce místo tebe.
Ovšem zjistit skutečné jméno programu pro danou aplikaci (Textový Editor vs. `gedit`) v praxi většinou znamená hledání na internetu nebo v `/usr/share/applications`.


## Procesy

**Proces** je synonymum pro *běžící program*. Jeho vlastnosti jsou:
* provádí akce
* pracuje na datech
* může ukazovat výsledky své činnosti v okénkách nebo příkazové řádce

Každý běžící program má svůj proces.
Když si otevřeš několik terminálů, vytvoří se několik procesů pro Bash. 
Každý má svůj vlastní stav, svou cestu, pamatuje si poslední spuštěný příkaz apod. 
Jeden proces = jedna instance programu.


### Seznamy procesů

Jaké procesy aktuálně běží na tvém systému?
To ti řekne příkaz `ps`:

```console
$ ps
  PID TTY          TIME CMD
 5403 pts/8    00:00:00 bash
 5449 pts/8    00:00:00 ps
```

Výstup programu je až překvapivě krátký, že? 
Program `ps` v základu totiž ukazuje pouze procesy běžící v aktuálním terminálu.
S parametrem `-a` dostaneš všechny aktuálně běžící procesy v systému. Většinou je jich hodně:
```console
$ ps -a
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
Výpis je uveden v textovém formátu, takže s ním můžeš dál pracovat pomocí nástrojů, které už znáš: `cut`, `grep` a dalších.

Hezčí výstup dostaneš, pokud použiješ program `top`:
```console
$ top
```

Výpis bude obsahovat tabulku podobnou příkladu níže, která se pravidelně aktualizuje:
```console
  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND     
 3267 hanka     20   0 2957412 321036 146352 S   2,3  4,1  30:39.88 Web Content 
 1392 hanka     20   0 3726628 295760 145672 S   2,0  3,8  17:12.48 gnome-shell 
 1108 hanka     20   0  714448 196972 164264 S   1,0  2,5  22:04.77 Xorg        
 3053 hanka     20   0 3489236 429100 167936 S   1,0  5,5  15:53.94 Web Content 
 (...)
```

Co tady vidíš?
* výpis procesů, které se vejdou na obrazovku
* procesy jsou seřazeny podle toho, kolik zabírají prostředků (vytížení procesoru)
* na prvních řádcích bývají typicky grafické aplikace (Chrome, Web Content)
* asi tam budeš mít nějaký *shell* - `bash`, `zsh`, `fish` etc.
* `gnome-shell` je taky druh *shellu*, ale grafický - je to plocha, ze které můžeš spouštět programy.
* `Xorg` - grafické prostředí, stará se o vykreslování oken

Pro posun ve výpisu použij <kbd>PgUp</kbd>/<kbd>PgDown</kbd>.


Co znamenají jednotlivé sloupce ve výpisu `top`?
1. sloupec *PID* je **číslo procesu** (angl. *Process ID*) – unikátní číslo. Když ho znáš, můžeš proces libovolně ovládat. Jak? Dozvíš se za chvíli.

2. sloupec *USER* - **vlastník**
Každý proces patří určitému uživateli, jehož jménem běží.
Každý uživatel na počítači má svůj vlastní uživatelský účet, třeba `hanka`. 
Kromě toho na všech linuxových počítačích existuje i hlavní systémový administrátor, `root`, jehož procesy mají plný přístup k celému systému.

3. **PR** - priorita: některé procesy jsou důležitější, než ostatní
  `51` má nižší prioritu
  `20` má vyšší prioritu
Pokud procesy s vyšší prioritou mají zrovna co dělat, provedou své činnosti první v pořadí, před procesy s nižší prioritou. 
4. **NI** - nice - podobně jako priorita, ale tady to je ve smyslu, _jak hezky se tento proces chová k ostatním_. Když má něco vysoký *NICE*, tak upřednostňuje ostatní procesy a běží jen, když není co jiného dělat. *Priorita* i hodnota *nice* jsou samostatné vlastnosti procesu.
5. **VIRT**,  **RES**, **SHR** - tyto tři atributy popisují, kolik proces zabírá paměti
6. **S** je stav procesu. Ten může být _běžící_  (`R`) , _spící_ (`S`) a další
7. **%CPU** - procentuální vytížení procesoru
8. **%MEM** - procentuální využití paměti
9. **TIME** - jak dlouho tento proces už běží. Počítá se ale jen procesorový čas, což znamená, že když proces spí, tak se čas nenavyšuje. Pro příklad: Bash, který většinu času jen čeká na příkaz, nebude mít tento čas moc velký. Pro prohlížeč, který typicky běží dlouho a neustále něco překresluje a stahuje, bude čas větší.
10. **COMMAND** - konečně ten samotný příkaz

`top` ukončíš zmáčknutím klávesy <kbd>Q</kbd>.


### Ovládání procesů
PID, číslo procesu, je velice důležitý údaj. Již jsme si řekli, že je unikátní.

**Úkol:** Pokud jsi ukončil{{a}} běh programu `top`, opět ho spusť. Zkus najít číslo procesu `top`, který běží ve tvé příkazové řádce. Budeš na to potřebovat dva terminály.
**Řešení**
* v jednom běží `top`
* ve druhém spusť:
```console
$ ps -a | grep top
 7590 pts/0    00:00:02 top
```

Jiná možnost je použít: 
```console
$ ps aux | grep -w top
```

Jedna z možností ovládání procesu je ukončit ho. Pamatuješ si z minula, jaký příkaz ukončuje proces?
Zkus to v terminálu (tam, kde jsi zjišťoval{{a}} číslo procesu `top`):
U nás číslo procesu je `7590`, u tebe bude jiné!
```console
$ kill 7590
```
Zkontroluj na druhém terminálu, že se `top` opravdu ukončil a vidíš opět prompt. 

> [note]
> Pokud špatně opíšeš číslo procesu, ukončíš příkazem `kill` nějaký jiný proces. 
> Může to mít nečekané následky.
>

Příkaz `kill <číslo procesu>` “hezky poprosí” proces, aby se ukončil.
Funguje to podobně jako <kbd>CTRL</kbd>+<kbd>C</kbd>.
Některé programy na prosbu nereagují; pomocí pouhého `kill` je neukončíš.

> [note]
> Některé grafické programy jako např. Firefox spouští víc procesů. 
> Pokud se pokusíš ukončit příkazem `kill` jeden z nich, nejspíš neuvidíš žádný rozdíl. Proces naběhne znovu. Tyto programy radši vypínej tlačítkem :).

# Soubory

## Disk, terminál a roura

Co je to soubor?
Můžeme si sestavit lehce naivní definici: *je to něco, z čeho můžeme číst a/nebo do čeho můžeme zapisovat*.
Shrňme pro přehled:
- jsou soubory, na kterých můžeš provést oboje (*číst i zapisovat*) – třeba soubor s programem, který píšeš 
- jsou soubory, které můžeš jen *číst* – třeba soubor se seznamem uživatelů počítače 
- jsou soubory, do kterých můžeš jen *zapisovat*,    
- a taky ty, u kterých nemáš právo na ani jedno – třeba soubor, který obsahuje hesla uživatelů.

Se soubory se dají dělat i další operace, než jen čtení a zápis, ale v tomto kurzu se jimi nebudeme zabývat. 

Soubory, se kterými máš pravděpodobně největší zkušenosti, jsou uložené na disku. 
Soubor se nachází na disku pod nějakým jménem a proces si ho může otevřít, přečíst a do něj zapsat.
Existují i soubory, které nejsou na disku, ale dá se do nich zapisovat a z nich číst. 
Podívejme se na program `ps`. 
Výsledky programu můžeš zapsat do souboru, například:
```console
$ ps aux > vystup.txt
```
Tento příkaz se dá přeložit na větu: *Bashi, spusť `ps aux` a řekni mu, aby psal do souboru `vystup.txt`*.
`ps` píše na tzv. *standardní výstup* (angl. *standard output*, *stdout*), a v příkladu výše je výstup nastaven právě na soubor `vystup.txt`. 
Kdybys výstup nepřesměroval{{a}}, uviděl{{a}} bys ho v terminálu.
Pro program `ps` se nic nemění, pořád píše do souboru. Terminál je něco, do čeho můžeš zapisovat a z něj číst. 
Když proces zapisuje do terminálu, můžeš to přečíst na své obrazovce. 
Proces `bash` čte přímo ze souboru, v tomto případě z terminálu, znaky které píšeš na klávesnici.
Kromě toho je taky do terminálu rovnou vypíše, abys věděl{{a}}, co jsi napsal{{a}}.
Terminál je tedy taky soubor, i když není na disku.

```console
$ ps aux | grep top
```

Příkazem výše Bash vytvoří tzv. rouru, do které jeden program (v tomto případě `ps`) zapisuje, a jiný (u nás `grep`) čte to, co ten první zapsal.
Roura, která programy spojuje, je taky soubor, i když není na disku. Dá se do ní zapisovat a dá se z ní číst.

Teď už známe 3 druhy souborů:
   * uložené na disku
   * terminál
   * roura


## Otevřené soubory procesu
Pamatuješ si, co znamená speciální proměnná `$$`?
```console
$ echo $$
8962
```
Je to číslo procesu aktuálního Bashe, každý spuštěný Bash ho má jiné. 

Existuje program `lsof`, který ukáže všechny otevřené soubory daného procesu, a ten si právě ukážeme.
Každý proces si může otevřít další soubory - to už znáš například z Pythonu - a podíváme se, jak zjistit seznam takto používaných souborů daného procesu. Použití programu je `lsof -p <číslo procesu>`
Vypiš si všechny soubory stávajícího Bashe.  
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
* **COMMAND** je jméno programu, který otvírá daný soubor (`lsof` umí soubory vypisovat i pro víc programů najednou).
* **PID** - číslo procesu (náš starý známý)
* **USER** - uživatel, jehož jménem proces běží
* **FD** je zkratka pro *file descriptor*, což je číslo otevřeného souboru. Toto číslo nás bude dnes nejvíc zajímat
   * **cwd** - speciální hodnota pro aktuální adresář (*current working directory*) – tahle hodnota se dá změnit pomocí `cd`
   * **rtd** - kořenový adresář (*root directory*) - mělo by to být `/`
   * **txt** - kód samotného programu (každý program musí být uložený někde na disku. Když ho pustíš jako proces, systém soubor přečte a provede)
   * **mem** - soubory v paměti
   * `0u` - 0 a dál - otevřené soubory včetně písmenka pro mód, např.:
     * `1u` - otevřeno pro R i W
     * `1r` - otevřeno pro čtení (R)
     * `1w` - otevřeno pro zápis (W)
* **TYPE**
    * **DIR** - adresář
    * **REG** - normální soubor
* **DEVICE** - číslo zařízení (disku), na kterém soubor je
* **SIZE/OFF** - velikost
* **NODE** - číslo souboru (unikátní v rámci *DEVICE*)
* **NAME** - jméno souboru


## Terminál jako soubor

Soubor, který má Bash otevřený jako `0u`, `1u` a `2u`, je (za normálních okolností) terminál, ve kterém Bash běží.
Zjisti z výstupu výše, který to je. V našem příkladu je to `/dev/pts/0`, u tebe může být jméno jiné.

Jak už víš, terminál je soubor, do kterého můžeš zapisovat.
Otevři si další okno terminálu a zadej následující příkaz. (Za `/dev/pts/0` doplň „svoje“ jméno terminálu):
```console
$ echo abc > /dev/pts/0
```
Všimni si, že se `abc` objeví ve druhém terminálovém okénku!

Když znáš jméno souboru a víš, jak se do něj dostat (znáš cestu), můžeš do něj zapisovat. 
Soubor pro terminál není zas tolik zvláštní: jakmile znáš jméno, můžeš do něj zapisovat jako do jakéhokoli jiného souboru.


### Procvičování v Pythonu
Procvičme si to trochu v Pythonu. 
Budeš potřebovat textový editor a dva terminály (budeme na ně dále odkazovat pomocí písmen A a B). 

> [note]
> Budeme používat Python, který je zabudovaný přímo v systému. Nevytvářej/neaktivuj si virtuální prostředí – pracuješ na virtuálním stroji, to úplně stačí.

V obou terminálech se přepni do adresáře, do kterého ukládáš soubory pro dnešní lekci. (Případně si ho předem vytvoř.)

Otevři si textový editor.
Vytvoř a ulož soubor `soubory.py`, do něhož napiš tento kód:

```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os   
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# Pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

time.sleep(600)
```

V terminálu A spusť soubory.py.
```console
(venv)$ python soubory.py
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
Vidíš výpis programu `python`? Skvěle, pojďme o kousek dál.

Do `soubory.py` dopiš tento kus kódu, který otevře soubor.
```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os   
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

with open("soubory.py") as soubor:
        time.sleep(600)
```
Po uložení souboru ukonči program v terminálu A (<kbd>CTRL</kbd>+<kbd>C</kbd>) a spusť ho znovu.
Do terminálu B zadej aktualizovaný příkaz `lsof -p <číslo procesu>`.
Podívej se na poslední řádek výpisu. Vidíš tam něco nového?
```console
$ lsof -p 6604
(...)
python  6604 user    3r   REG    8,1      333 1314091 /home/user/bash/03/soubory.py
```
Měl by přibýt záznam o `soubory.py` s číslem `3r`. 

Řekl{{a}} jsi Pythonu, aby otevřel pro čtení soubor s vlastním programem.
Protože příkaz `lsof` vypíše otevřené soubory procesu, vidíš ve výsledcích nově výsledek této operace.  

Zatím všechno v pohodě? Tak otevři jeden soubor pro čtení a nějaký jiný pro zápis. 
Změn svůj skript na příklad níže.
```python
# soubory.py
# modul, kde jsou zpřístupněné služby operačního systému
import os   
import time

# číslo právě běžícího procesu (ne Bashe, ale Pythonu)
# pokaždé, když spustíš soubory.py v příkazové řádce, se toto číslo změní
print(os.getpid())

with open("soubory.py") as soubor:
    with open("jiny.txt", "w") as fd2:
        time.sleep(600)
```

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
Ve sloupci FD vidíš malé `r`, `w`, `u` - pro čtení, zápis a oboje.


Python nám v mnohém pomáhá, ale taky vzdaluje od systémové vrstvy. Pythonní objekty nejsou přesně totéž jako způsob, jakým soubory zpracovává operační systém. 
My se chceme podívat, jak funguje vevnitř operační systém, nikoliv Python.
Proto si otevři tyto soubory ještě jednou pomocí modulu `os`, který je pro to přímo dělaný.

Zaměň celý blok s `with` za tento kus kódu:
```python
soubor1 = os.open("soubory.py", os.O_RDONLY)
soubor2 = os.open("jiny.txt", os.O_WRONLY)
print(soubor1, soubor2)
time.sleep(600)
```

Takto upravený program spusť v terminálu A. 
Kromě nového čísla procesu bys měl(a) vidět na dalším řádku dvě čísla. U nás jsou to `3, 4`.
```console
(venv)$ python soubory.py
19257
3, 4
```

V terminálu B spusť opět příkaz `lsof` s novým číslem procesu a podívej se na dva poslední řádky. 
`FD` otevřených souborů odpovídá číslům z terminálu A.
Funkce `os.open()` vrací číslo souboru. 
Každý otevřený soubor je očíslovaný, a podle tohoto čísla můžeš s ním pak pracovat.

> [note]
> Poznámka pro zvídavé: pod složitým `os.O_RDONLY` a `os.O_WRONLY` se
> skrývají jenom číselné konstanty `0` a `1`. Je tomu tak proto, že v UNIXových operačních systémech se systémové operace (např. `open`) modifikují pomocí číselných konstant, kterým byla pro lepší čitelnost přiřazena jednoduchá, krátká jména.

> [note]
> Pokud otevřeš pomocí pythonní funkce
> `open`  neexistující soubor pro zápis, funkce soubor vytvoří a otevře ho.
> Oproti tomu systémová funkce `os.open()` ho nevytvoří, ale zahlásí chybu. Pokud se ti to stane, můžeš společně s `os.O_WRONLY` použít `os.O_CREAT`. Kombinují se pomocí operátoru `|`:
> ```console
> os.open('jiny.txt', os.O_WRONLY | os.O_CREAT)
> ```
> Anebo můžeš soubor vytvořit v terminálu pomocí`touch jiny.txt`.


Přidej do pythonního souboru pod řádky, kde soubory otevíráš, tento řádek:
```python
print(os.read(soubor1, 10))    
```
Tento řádek načte prvních 10 bajtů ze souboru `soubory.py` a vypíše je do terminálu. 
V terminálu A spusť `soubory.py`.
V našem případě výstup vypadá takto:
```console
(venv)$ python soubory.py
20019
b'# soubory.'
3, 4
```

Do souboru můžeš i něco napsat. Přidej za předchozí řádek tento kód, který zapíše 4 písmenka do `jiny.txt`.
```python
os.write(soubor2, b'abcd\n') 
```

Nezapomeň na konec dát `\n`, nový řádek, aby se pak text hezky vypisoval.
Když teď spustíš program v terminálu A a v terminálu B vypíšeš obsah souboru `jiny.txt` pomocí programu `cat`, měl by se ti zobrazit text "abcd".
```console
$ cat jiny.txt 
abcd
```

Jak víš z kurzu Pythonu, otevřené soubory je dobré vždy na konci manipulace zavřít.
Tak to pojď udělat a dokončit ukázkový skript.
Na konec, před `time.sleep`, dopiš:
```python
os.close(soubor1)
os.close(soubor2)
```

Celou dobu pracuješ se soubory otevřenými pythonním programem, tedy `3` a `4`. 
Určitě tě zajímá, co jsou vlastně `0`, `1`, `2` ze sloupce `FD` ve výpisu `lsof`.

V textovém editoru smaž `time.sleep` a místo toho dopiš řádky
```python
os.write(1, b'Tohle jde do souboru 1\n')
os.write(2, b'Tohle jde do souboru 2\n')
```
Jak myslíš, kam se to vypíše?
Všechno se to vypíše do terminálu, ve kterém spouštíš skript.
```console
(venv)$ python soubory.py 
7102
3 4
b'# soubory.'
Tohle jde do souboru 1
Tohle jde do souboru 2
```

Funguje to?
Tak zkus přesměrovat výstup `soubory.py` do `jiny.txt`:
```console
(venv)$ python soubory.py > jiny.txt
Tohle jde do souboru 2
(venv)$ cat jiny.txt 
Tohle jde do souboru 1
7167
3 4
b'## soubory'
```
Když se podíváš do souboru `jiny.txt`, najdeš v něm text *Tohle jde do souboru 1*.

Pokud to tam máš, připiš do skriptu ještě poslední řádek.

```python
print(os.read(0, 10))
```

Spusť ho v terminálu A bez přesměrování.

```console
(venv)$ python soubory.py
```

Teď, když něco napíšeš do terminálu, terminál vypíše prvních 10 bajtů tvého textu.

### Standardní vstup, standardní výstup a standardní chybový výstup

Tyto tři soubory, `0`, `1` a `2`, má každý proces otevřené.

`0` je standardní vstup (angl. *standard input stream*, *stdin*), čili to, co napíšeš do programu. 
Normálně mají programy jako standardní vstup terminál, ze kterého jsou spuštěny.
Ale může to být i jiný soubor.
Podívej se ještě jednou na příkaz ze začátku dnešní lekce.
```console
$ ps -a | grep top
```
Příkaz `grep` má jako standardní vstup výstup programu `ps -a` – konkrétně rouru, do které `ps` píše.
Standardní vstup může být i normální soubor:

```console
grep print < soubory.py
```

`1` je standardní výstup (angl. *standard output stream*, *stdout*), čili místo, kam program píše informace, které chce předat světu.
Když použiješ `ps -a`, na standardní výstup dostaneš tabulku s daty všech procesů.
`echo abc` vypíše na standardní výstup text *abc*.
`echo abc | wc` - `echo` vypíše na svůj standardní výstup 4 písmenka, `wc` na svém standardním vstupu ty 4 znaky dostane.

`2` je standardní chybový výstup (angl. *standard error stream*, *stderr*), což je místo, kam programy píší chybové hlášky.
Často to bývá stejný terminál jako *stdout*, ale přesměrovává se samostatně.
Příklad:
```console
$ cp a b       
cp: cannot stat a no such file or dir
```

Soubor `a` neexistuje, a proto dostaneš chybovou hlášku. 

Když přesměruješ standardní výstup do souboru, pořád se chybová hláška objeví v terminálu. Zkus:

```console
$ cp a b > jiny.txt
cp: cannot stat a no such file or dir
```

Proč je to tak? Zápisem `>` přesměruješ jen *stdout* (1), nikoli *stderr* (2).
Zobáček `>` přesměrovává pouze standardní výstup, zatímco chybový výstup se běžně nepřesměrovává.
A k čemu je to užitečné? 

Na standardní výstup programy většinou píšou výsledky své práce v nějakém formátu, který se pak dá automaticky zpracovat. 
Když nastane chyba, program ji ohlásí, ale ohlásí ji na jiném místě, než vypíše výstup, aby nekazil další zpracování. 
Třeba `grep` z příkladu `ps -a | grep top` výše pak nevidí chybové hlášky, jelikož čte jen *stdout*, nikoli *stderr*.

Vraťme se na chvíli k Pythonu.

V terminálu A spusť skript. 
```console
(venv)$ python soubory.py
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
Pythonní traceback (popis toho, kde se stala chyba) ale jde do chybového výstupu, tedy stále do terminálu.

Co kdybys ale přece jen chtěl{{a}} přesměrovat ten druhý, chybový, výstup?
Existuje na to speciální operátor `2>` - přesměrování souboru číslo 2.
```console
(venv)$ python soubor.py 2> jiny.txt
7388
3 4
b'## soubory'
Tohle jde do souboru 1
^C
(venv)$ cat jiny.txt 
Tohle jde do souboru 2
Traceback (most recent call last):
  File "soubory.py", line 21, in <module>
    print(os.read(0, 10))
KeyboardInterrupt
```

Můžeš přesměrovat i oboje:
```console
(venv)$ python soubory.py > vystup.txt 2> chyby.txt
^C(venv)$ cat vystup.txt 
Tohle jde do souboru 1
7423
3 4
b'## soubory'
(venv)$ cat chyby.txt 
Tohle jde do souboru 2
Traceback (most recent call last):
  File "soubory.py", line 21, in <module>
    print(os.read(0, 10))
KeyboardInterrupt
```

Záleží na pořadí `>` a `2>`? Ne. Bash obecně provádí příkazy v pořadí, v jakém je napíšeš.

Pro úplnost si ukážeme, že můžeš přesměrovat i standardní vstup pomocí opačné šipky.

```console
(venv)$ python soubory.py < jiny.txt
7481
3 4
b'## soubory'
Tohle jde do souboru 1
Tohle jde do souboru 2
b'abcd\n jde '
```
Poslední řádek je vstup načtený ze souboru `jiny.txt`.
Podívej se na soubor 0 ve výpisu `lsof`.
Standardní vstup je nastaven na „opravdový“ soubor na disku.
```console
$ lsof -p 7481
...
python  7481 user 0r REG 253,0 150 299463294 /home/user/bash/03/jiny.txt
...
```

Výsledek vypsaný do terminálu je stejný jako při přetečení dat rourou, ale soubor otevřený pro standardní vstup bude vypadat trochu jinak.
```console
(venv)$ cat jiny.txt | python soubory.py
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

Programy občas kontrolují, zda jejich výstup je mířen na terminál nebo soubor, a podle toho přizpůsobují i vzhled výstupů. 
Porovnej u sebe tyto dva příkazy

```console
$ grep abc jiny.txt
$ grep abc jiny.txt | cat
```

V prvním výstupu bys měl{{a}} vidět písmenka, které splňují podmínku, červenou barvou. 
Zatímco když výstup dáš na zpracování dál např. programu `cat`, `grep` nepřidá žádné barvičky navíc.

> [note]
> Odbočka pro zvídavé - vstup se může předat i s číslem, o němž chceme,
> aby ten soubor dostal: 
> ```
> console $ python soubory.py 8< jiny.txt 
> ```
> 
> Soubor `jiny.txt` se takto předá Pythonu na zpracování jako `8r` (když
> se podíváš do `lsof`).  Jen čísla 0, 1, 2 mají určený význam, ostatní
> čísla můžeš používat dle libosti.

Zvláštní případ použití je přesměrování některého výstupu do speciálního souboru, který si nepamatuje nic, co se do něj píše. Je to `/dev/null`.

```console
(venv)$ python soubory.py 2> /dev/null
```

Celý chybový výstup se "vyhodí do vzduchu", nebude o něm záznam v systému.

Jiný příklad - jsou programy, které píšou do terminálu spoustu výstupu, např. `find`. 
Pokud chceš vidět jen problémy, můžeš přesměrovat standardní výstup do `/dev/null` a v terminálu uvidíš jen chybové hlášení:

```console
$ find /var/cache > /dev/null
$ find: … Permision denied
```

V některých adresářích můžeš mít tolik souborů, na něž nemáš přístup, že se ti ani nebude chtít o tom číst. 
Pak si chybový výstup přesměruješ do `/dev/null` a nemusíš se tím zabývat.
`$ find /var/cache 2> /dev/null`


#### Přesměrování obojího

Zatím každá věc šla někam jinam, ale zkusíme si oboje přesměrovat do toho samého.

```console
(venv)$python soubory.py > jiny.txt 2>&1
                                    ^--- chybový směruje tam, kam první
```

Kdyby v příkazu nebyl `&`, výstup se přesměruje do souboru s názvem `1`.
`&1` znamená *odkaz* na soubor s číslem 1, tedy standardní výstup.

Tady už záleží v jakém pořadí se skládají příkazy, protože tohle fungovat nebude:
```
                         ,-------- 1. přesměruje stderr na stdout, tedy na terminál
                         |    ,--- 2. přesměruje stdout do souboru
                         v    v
(venv)$python soubory.py 2>&1 > jiny
```

Proto *stderr* půjde na terminál a *stdout* do souboru.

### Jak se tvoří procesy? (teoreticky)

Vytvoř si nový pythonní soubor v textovém editoru: `procesy.py`.

Znáš funkce, které vrací nějakou hodnotu.
```python
print(len(‘abc’))
```
Funkce `len` a `print` se vrátí po zavolání.
Existují i funkce, které se nevrací?
Ano, například `exit()`

Když máme funkce, které se nevrátí, a taky funkce, které se vrátí jednou, můžou existovat funkce, které se vrátí víckrát?
Zkus:

```python
import os
os.fork()
print("Tohle se stane.")
```

Věta *Tohle se stane.* se vypíše dvakrát.

Jak to, že se `fork` vrací dvakrát? Naklonuje aktuální proces a pokračuje dál už se dvěma samostatnými procesy.

Tyto procesy se liší jen v jedné jediné věci - tím, co vrátí `os.fork()`.

```python
import os
os.fork()
hodnota = os.fork()
print("Tohle se stane.", hodnota)
```

```console
(venv)$ python procesy.py
Tohle se stane. 1827
Tohle se stane. 0
```

V jednom procesu je `0`, druhé číslo je číslo procesu.
Jeden proces, „rodič“, dostal číslo druhého procesu, „potomka", aby ho mohl ovládat.
Druhý proces dostal 0, aby poznal že je „potomek".

```python
import os
os.fork()
hodnota = os.fork()
if hodnota == 0:
    print("já jsem dítě")
else:
    print("já jsem rodič")
```
A teď už ty programy mohou dělat úplně jiné věci.

Spousta programů před forkem často vytvoří rouru, což je soubor, který má dva konce:
   * jeden pro čtení
   * druhý pro zápis

Existuje na to funkce `os.pipe()`, která rouru vytvoří a vrátí dvojicí hodnot: jeden soubor pro čtení, druhý pro zápis.
```python
import os

r, w = os.pipe()

if hodnota == 0:
    print("já jsem dítě")
    os.close(r)
    os.write(w, b’Ahoj’)
else:
    print("já jsem rodič")
    os.close(w)
    pozdrav = os.read(r, 10)
    print(pozdrav)
```

Co dělá tento kód? Proces-dítě vypíše, že je dítě a zavře konec roury pro čtení. 
Pak napíše do konce pro zápis "Ahoj".
Proces-rodič vypíše, že je rodič, zavře rouru pro zápis a přečte prvních 10 bajtů z konce pro čtení. 
Takto si můžou procesy povídat.

Jaký typ mají proměnné `r` a `w`? 
Čísla, jako v příkladech výše. 
Můžeš to ověřit tím, že si je vypíšeš po vytvoření roury příkazem `print(r, w)`.

> Toto přesně dělá Bash, když mu nastavíš přesměrování jako
> ```console
> (venv)$ python soubory.py 2> /dev/null
> ```
> 
> _toto je třeba líp popsat, z videa mi nebylo jasné co který proces kdy přesměruje ;)_
> Ten hlavní proces je `python`, na šipečce se "odforkuje" a dětský proces přesměruje chybový výstup na uvedený soubor.
