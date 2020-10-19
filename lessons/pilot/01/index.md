
# 1. Linuxová administrace - Bash (1/2)
Pod pojmem **linuxová administrace** si lidé často představují spoustu terminálové magie, a především _skriptování v Bashi_. 
My si dnes vysvětlíme základní pojmy, a hned si povíme, že obsahem kurzu není zdaleka jen _Bash_. 
Budeme mu věnovat první dva srazy a potom se podíváme do dalších zákoutí linuxového světa.
Dozvíš se mimo jiné:
- jak zefektivníš používání linuxového počítače, 
- co jsou a jak fungují souborové systémy,
- jak vypadá správa disku a místa na disku,
- jak vypadá správa serverů,
- a taky něco málo o kernelu.


Jako úvod do Bashe použijeme otevřené materiály, které připravili tvůrci ze Software Carpentry (anglicky). 
Materiály najdeš pod odkazem:
http://swcarpentry.github.io/shell-novice/
(Do projektu vytváření materiálů v češtině se můžeš zapojit i ty - můžeš přeložit výše uvedené materiály nebo udělat tahák s příkazy, které ti přijdou nejdůležitější).


## Shell
Co je to _Bash_? 
Je to druh tak zvaného _shellu_. 
*Shell* je program, kterému zadáváš příkazy a on podle nich pouští další programy. 
Česky často pro shell používáme pojem _příkazová řádka_.
Je to ten program, který ti vypisuje nějakou informaci, která končí znakem `$`. Po znaku dolaru vidíš blikající kurzor.

Příklad:
```console
user@yourcomputer:~$
```
Když napíšeš za něj jméno nějakého jiného programu, tento program se spustí. 

Příklad:
```console
user@yourcomputer:~$ python
Python 2.7.17 (default, Nov  7 2019, 10:07:09) 
[GCC 9.2.1 20191008] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
user@yourcomputer:~$
```

To je funkce shellu - zadávám příkaz, on spouští další programy. 
Jaký je rozdíl mezi shellem a Bashem? Bash je jedna konkrétní varianta shellu. 
Vztah Bashe k shellu si můžeš představit analogicky ke grafickým aplikacím: 
Existuje program na spouštění webových stránek, kterému říkáme *prohlížeč*. 
Pak existují různé konkrétní prohlížeče: Firefox, Chrome a další. 
Stejně tak jeden druh programů pro příkazovou řádku je shell, a ten může mít rozličná konkrétní provedení: _Bash_, _zshell_, _fish_ a spoustu dalších. 
Bash je z nich nejpouživanější.
A tak, když spustíš příkazovou řádku, měl by se ti spustit bash, pokud sis nehrál{{a}} se svým systémem a nenastavil{{a}} ho jinak.

Pro jistotu spusť příkazovou řádku a zadej příkaz `echo $0`. 
Měl by ti říct, že shell je bash: 
```console
$ echo $0
/bin/bash
```

Proč používat příkazovou řádku:
* Příkazová řádka je textová! Možná ti to teď nepřipadá jako moc velká výhra, ale možnost zkopírovat a vložit text je velice užitečná. Pokud máš problém s konkrétním úkonem, můžeš si označit text, poslat ho například na diskuzní fórum, kde někdo se na to podívá a pomůže ti najít řešení. Posílání screenshotů a grafických postupů (*„a pak zmáčkni modré tlačítko vpravo nahoře“*) je mnohem složitější. 
* Když dokážeš napsat příkaz, znamená to, že ho můžeš napsat do souboru a říct počítači třeba: _tohle udělej 1000x a trošku to vždycky změň_. Tomu se říká *skriptování.* Budeš si tím moci leccos zautomatizovat.

## Prompt

Když spustíš příkazovou řádku, před blikajícím kurzorem vidíš ještě nějaké informace. Ty se dají nastavit podle tebe a tvých potřeb, my si ale nejprve ukážeme základní nastavení: 

```console
jmeno@nazevpocitace:~$
```

Co tady vidíš? První je __uživatelské jméno__, za zavináčem se nachází __jméno počítače__, a po dvojtečce __adresář__, ve kterém právě jsi.
Tato část, zvaná `prompt`, standardně končí dolarem. Za znak dolaru pak můžeš psát příkazy. 

> [note]
> V digitálním světě je zvykem, že se příkazy shellu uvádějí ve formátu `$ prikaz` nebo `# prikaz`. 
> Ani dolar, ani křížek se neopisují - nejsou součástí příkazu. 
> Dolar znamená, že se příkaz pouštíš jako normální uživatel, křížek znamená že musíš být přihlášen{{a}} jako administrátor{{gnd('', 'ka')}}. 
> O uživatelích si podrobně povíme jindy, ale už odteď budeme u příkladů v kurzu dodržovat tuto konvenci. 


### Úkol podle SW Carpentry

Udělej si pro tento kurz nový, prázdný adresář (pokud ho tedy jestli ještě nemáš).

Stáhni z http://swcarpentry.github.io/shell-novice/setup.html soubor `data-shell.zip` a v adresáři pro kurz ho rozbal.
Budeme chvilku předstírat, že vedeme výzkum mořských živočichů a znečištění vody na základě dat z plavby po Pacifickém Oceánu. 
Máme několik souborů s nějakými kousky dat a na každém z nich musíme provést několik operací a výsledky sepsat do nějaké zprávy. 
Problém je, že naklikat to ručně bude trvat 20 vteřin pro každý řádek a záznamů je opravdu hodně. 
Pokud bychom do toho přece jen šli, nestihneme termín. Místo toho se naučíme, jak zpracovávat věci automaticky.
Bude chvíli trvat, než se to naučíš, ale pak bude práce rychlejší.


## Příkazy

Velice užitečný příkaz je `ls`, který vypíše obsah aktuálního adresáře. Zkus to u sebe.

A co se stane, pokud zkusíš zadat neexistující příkaz, třeba `ks`?
Standardně ti bash řekne `command not found`. 
Některé operační systémy k tomu přidají nápovědu, jestli jsi tím náhodou nemyslel nějaký jiný program, případně nabídnou ti jeho instalaci. 

Jak se to liší od okénkového prostředí?
V okénkovém prostředí bys měl{{a}} k dispozici někde v menu pod tlačítky všechny funkce programu a mohl{{a}} by ses k nim postupně doklikat. 
Všechno, co s takovým programem můžeš udělat, najdeš přímo v něm.
V textovém rozhraní si každý příkaz musíš pamatovat nebo najít na taháku, v nápovědě nebo na internetu. 
Na druhou stranu je příkazů, které tam můžeš zadat, mnohem víc.


## Pohyb po souborovém systému

Příkaz `pwd` ti řekne, kde jsi v adresářovém systému (je to zkratka od _**p**rint **w**orking **d**irectory_).
Jméno adresáře, ve kterém se nacházíš, nejspíš obsahuje tvoje uživatelské jméno, takže u tebe bude výstup vypadat trochu jinak než u mě:

```
$ pwd
/home/user
```
Co to znamená?
Adresářový systém je stromová struktura, ve které se nachází adresáře, v nich další adresáře nebo soubory.


`/`
tzv. root (kořenový) adresář - začátek souborového systému
- všechny další adresáře a soubory jsou v tomto kořenovém adresáři
- obsahuje mimo jiné složku `/home`, která obsahuje tzv. domovské adresáře. 
- všechno ostatní v kořenovém adresáři je součást systému, v další části kurzu se podíváme, co to všechno znamená.
- strom adresářů je oddělován jednoduchými lomítky 

`/home/user/`
- první lomítko (viz výše) je také název adresáře
- domovský adresář (v tomto případě uživatele _user_, u tebe se jmenuje jinak)
- zde si uživatel ukládá vlastní soubory a složky - zde si můžeš dělat, co chceš, tzn. vytvořit vlastní strukturu dat.
- `~` je zkratka pro domovský adresář


### ls a přepínače

Napsáním `ls` se spustí program, který se jmenuje **ls**.
Tomu programu můžeš dat různé přepínače nebo argumenty, které modifikují chování programu:
```console
$ ls -F
```

(Pozor, mezi `s` a `-` je mezera, ale mezi `-` a `F` ne!)

Tento příkaz opět vypíše obsah aktuální složky, ale přepínač `-F` trochu pozmění výstup:
- adresáře budou mít na konci lomítka
- odkazy budou označené zavináčem
- spustitelné soubory budou mít hvězdičku

**Přepínače** (angl. *options*) začínají pomlčkou. Nějak modifikují základní chování programu.
**Argumenty** jsou typicky jména souborů, se kterými má pracovat. Taky se oddělují mezerou.

Zkus se podívat, co se nachází v adresáři `/`:
```console
$ ls -F /
```

Dobrá zpráva: nemusíš se učit nazpaměť, co dělá který přepínač každého programu. 
Naprostá většina příkazů má totiž vestavěný přepínač `--help` s offline nápovědou, který ti poskytne informaci, jakým způsobem můžeš modifikovat základní chování programu.

Přepínače existují ve dvou variantách: krátké a dlouhé.
Krátká začíná `-`, za kterým je jedno písmenko. Dlouhá začíná `--`, za kterým je delší název: `-h` vs `--help`. 
Často jsou tyto varianty paralelní (můžeš použít kterou chceš a dosáhneš stejného výsledku).
Když pracuješ přímo v příkazové řádce, typicky použiješ krátkou variantu. 
Při psaní skriptů je ale vhodnější používat dlouhou variantu, protože výstižněji vyjadřuje tu konkrétní modifikaci a lépe se to pak bude číst. 

Všechno, co vidíš po zadání do příkazové řádky příkazu 
```console
$ ls --help
```
je součást programu `ls`.

Existuje i jiná varianta, jak se podívat, co který program dělá:
`$ man ls`

To jsou takzvané manuálové stránky, zpravidla mnohem delší a podrobnější nápověda - zabírá hodně místa, a proto ji lehčí operační systémy občas nemají :). 
Jak ovládat manuál? Klávesy <kbd>PgUp</kbd>/<kbd>PgDown</kbd> použij pro pohyb, klávesou `q` výpis vypni a vráť se do Bashe.

Co udělá `ls`, když přepínač nezná? 
Zkus:
```console
$ ls -j
```

Odkáže tě to na `--help`.

*Cvičení*:
Zkus zjistit, co dělají s programem `ls` přepínače `-h` a `-l`.

*Odpověď*:
Když zadáš `ls --help`, dozvíš se, že `-h` je „human readable“ a `-l` je „long listing format“.

S přepínačem `-l` bude formát výpisu bude mnohem robustnější. Dozvíš se mj. kdo vlastní daný soubor, jaká práva má daný soubor, kolik zabírá místa na disku, kdy byl naposled modifikován.

Když zkombinuješ oba přepínače zároveň pomocí příkazu:
```
$ ls -h -l
```

Dostaneš velikosti souborů převedené do trošku nepřesné, ale líp čitelné podoby. Takto se dozvíš, že například adresáře mají téměř vždy 4kB. (Samotné `ls -l` by oznámilo 4096 bajtů).

Krátké přepínače můžeš sdružovat: místo `ls -l -h -F` můžeš psát `ls -lhF` jen s jednou pomlčkou na začátku.
Dlouhé bohužel sdružovat nejde.


### cd

Možná už znáš příkaz `cd` (z angl. ***c**hange **d**irectory*).
Když ho zadáš bez argumentu, přesune tě do domovského adresáře (např. `/home/user`). Když ho zadáš s argumentem, přesune tě do adresáře, který uvedeš jako argument.

Příklad:

```console
user@yourcomputer:~$ cd /etc
user@yourcomputer:/etc$ 
```

Všimni si změny promptu po této operaci. Místo `~`, která znamená domovský adresář, je tam nyní `/etc`. Kdyby ses chtěl{{a}} dostat zpět do domovského adresáře, máš několik možnosti:

1. `$ cd`
2. `$ cd ~`
3. `$ cd /home/user`
4. (Bonus: `$ cd -` tě přesune do naposledy navštíveného adresáře.)


*Úkol:* pomocí `cd` se dostaň do adresáře, ve kterém jsi rozbalil{{a}} archiv `data-shell.zip`. Pro kontrolu, zda jsi na správném místě, použij příkaz `ls`.

Příkaz `ls` umí vypsat obsah adresáře, který uvedeš jako jeho argument. Zkus například:

```
$ ls data-shell
creatures    molecules    data    solar.pdf
...
$ ls data-shell/data
animal-counts    elements    pdb    salmon.txt
...
```

Zde pro tebe máme tip. Nemusíš si pamatovat všechny adresáře ve svém počítači a přesné jméno každého souboru. 
Bash v sobě kouzla na zrychlení práce, které ti napoví nebo ti umožní zadat jen začátek jména. 
Stačí, že napíšeš jen první písmenko a zmáčkneš tabulátor (klávesu <kbd>⭾ TAB</kbd>, kterou najdeš na klávesnici nad <kbd>Caps Lock</kbd>). 
Chování Bashe pak záleží na tom, co se nachází v daném adresáři. 
Když zmáčkneš <kbd>⭾ TAB</kbd> jednou, doplní se buď název souboru/adresáře, nebo nejdelší společná část, pokud stejným písmenkem začíná víc souborů v daném adresáři. Pak Bash čeká na další písmenko.
Když zmáčkneš <kbd>⭾ TAB</kbd> podruhé, vypíšou se možné varianty - všechny soubory začínající stejným písmenkem.
Když víš co chceš napsat, klávesou <kbd>⭾ TAB</kbd> si hodně urychlíš práci. 
Když nevíš, co chceš napsat, zmáčkneš <kbd>⭾ TAB</kbd> dvakrát a podíváš se, co máš k dispozici.

Do adresáře `data-shell` se dostaneš pomocí: 
```console
$ cd data-shell
```
Kdybys chtěl{{a}} "skočit" o adresář nahoru, zpět do složky, v níž jsi před chvíli byl{{a}}, je na to příkaz:
```console
$ cd ..
```

Jména adresářů je možno spojovat pomocí lomítka a tvořit tak *cestu*, která říká jak se v adresářovém systému pohnout. Například místo téhle sekvence příkazů:
```
$ cd data-shell
$ cd data
$ cd ..
$ cd data
$ cd ..
$ cd ..
```
můžeš napsat jen:
```console
$ cd data-shell/data/../data/../..
```

Název aktuálního adresáře je součástí promptu.


**Tipy:**
* `cd -` tě přesune do posledního navštíveného adresáře. (Zkus si `cd -` zadat několikrát po sobě.)
* **vkládání prostředním tlačítkem myši** - vybereš myší text (samo se to zkopíruje) a prostředním tlačítkem se to vloží na určené místo. Funguje to na Linuxu všude, i mezi terminálem a grafickými programy. Klasická schránka <kbd>Ctrl</kbd>+<kbd>C</kbd>/<kbd>Ctrl</kbd>+<kbd>V</kbd> je samostatná, je možné mít v každé schránce něco jiného.
* **Copy & paste** - klasické <kbd>Ctrl</kbd>+<kbd>C</kbd>/<kbd>Ctrl</kbd>+<kbd>V</kbd> v terminálu nefunguje. Tyto operace se dělají pomocí <kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>C</kbd>/<kbd>Ctrl</kbd>+<kbd>Shift</kbd>+<kbd>V</kbd>.
Důvod? Ještě než vznikly grafické programy, v dávných dobách počítačového vývoje, mělo <kbd>Ctrl</kbd>+<kbd>C</kbd> v příkazové řádce jiný význam: přerušení aktuálního programu.


## Operace kopie, přesun, přejmenování, mazání

*Úkol:* Přesuň se do složky `data-shell/creatures`

Je zde mimo jiné soubor `basilisk.dat`. `.dat` je tak zvaná *přípona*, která je pro počítač součástí názvu souboru. 
To, že je tam přípona, nemá význam pro samotný systém. Je to ale vodítko pro lidi, aby věděli, jaký druh dat mají v daném souboru očekávat. 
Přípona je důležitá pro Windows, který podle ní zjistí, jakým programem soubor otevřít. 
Linux se místo toho typicky podívá do samotného souboru a program vybere podle obsahu. 

Prohlížet si soubory složka po složce není moc zajímavé. Proto existuje příkaz `ls -R`, který rekurzivně zobrazí všechny adresáře a jejich obsah. 
_Rekurzivně_ znamená, že se podívá do každého adresáře a když tam najde další adresář, tak se podívá i do toho adresáře a tak dál, a vypíše obsah všeho, co v těch adresářích je.
Ve výpisu uvidíš mimo jiné takové řádky:
`./data`
`./creatures`
Tečka na začátku znamená aktuální adresář. Dvě tečky, `..`, už znáš: znamenají nadřazený adresář.


### tree
`tree` je příkaz, který hezkým způsobem zobrazí strukturu adresářů a jejich obsah. Obsahově je ekvivalentní k `ls -R`.
Ne všechny systémy ho mají přímo v sobě, občas se musí doinstalovat.

**Pár slov o konvencích**

 - Soubory se většinou pojmenovávají malými písmenky. (Zjednodušuje to např. spolupráci s Windows, které neberou ohled na velikost písmen.)
 - Jako oddělovač slov slouží `-` nebo `_`. (Mezeru raději nepoužívej - má svůj význam jako oddělovač argumentů.)
 - Soubory často mají příponu oddělenou tečkou, která naznačuje jaký druh dat v sobě mají.
 - Když potřebuješ dát do názvu datum, tak ve formátu YYYY-MM-DD (rok, měsíc a den – např. 2020-03-02). Když pak seřadíš data podle abecedy, seřadí se automaticky správně.


### Skryté soubory

Když půjdeš do svého domovského adresáře a zavoláš příkaz `ls`, uvidíš spoustu různých souborů. Nebudou tam ale všechny. Některé soubory jsou skryté. Když chceš vidět všechno, použij přepínač `--all` (`-a`):

```console
$ ls --all
.bashrc    .cache    .config    Documents
...
```
nebo
```console
$ ls -a
.bashrc    .cache    .config    Documents
...
```

Skryté soubory začínají tečkou a v domovském adresáři je jich opravdu hodně. 
Spousta programů totiž právě tam ukládá své konfigurační soubory.
Není to nejlepší. Dnes už existuje za tímto účelem vytvořený skryty adresář `.config`, ale ne všichni ho začali používat. 

Když vytvoříš vlastní soubor a pojmenuješ ho s tečkou na začátku, taky se po zavolání standardního `ls` taky neukáže.

Speciální případy schovaných souborů jsou `..`, nadřazená složka, a samozřejmě `.`, aktuální adresář.

V grafickém programu na prohlížení souborů si můžeš většinou nastavit, aby ukazoval nebo neukazoval skryté soubory.


### Nový adresář
V adresáři `data-shell` udělej nový adresář. Nový adresář se vytváří pomocí příkazu mkdir (_**m**a**k**e **dir**ectory_).

```console
$ mkdir thesis
```

Po zmáčknutí <kbd>Enter</kbd> se nic viditelného nestane. Když program uspěje, je zticha. Složka se tedy na pozadí vytvořila, jen to zatím nevidíš. Pro kontrolu můžeš zadat příkaz `ls`.

Kdyby se během provádění programu vyskytla chyba, dozvěděl{{a}} by ses o tom hned. Pokus se třeba znovu vytvořit adresář se stejným jménem: `mkdir` si postěžuje, že takový soubor už existuje.

Nově vytvořený adresář neobsahuje žádné soubory, kromě speciálních adresářů `.` a `..` .

### Vytvoření prázdného souboru
Zkus si:
```console
$ cd thesis/
$ nano draft.txt
```
Ten poslední řádek otevře pro editaci nový soubor.
Pokud by už v tom adresáři soubor `draft.txt` existoval, otevře ho, aniž by vytvářel novou kopii.
Do tohoto souboru si něco napiš, například básničku. 
Dole v `nano` se nachází nápověda, co všechno s tím souborem můžeme udělat - my bychom ho chtěli uložit. 
Stisk klávesy`Ctrl` je zkrácený na znak `^`. A tak uložení souboru proveď stisknutím <kbd>Ctrl</kbd>+<kbd>O</kbd>. `nano` pak ukonči a vyjdi zpátky do příkazové řádky pomocí kombinace <kbd>Ctrl</kbd>+<kbd>X</kbd>. (Příkaz potvrď Enterem.)

Jiný způsob, jak vytvořit soubor, je `touch`.
Ten neotvírá žádný editor, jen vytvoří prázdný soubor.
```console
$ touch soubor.dat
$ ls
draft.txt    soubor.dat
```

> [note] Proč zrovna `touch`?
> Příkaz `touch` původně slouží pro nastavení data změny už existujících souborů; vytvoření nového souboru je jen vedlejší efekt. Časem se ale vytváření souboru stalo nejběžnějším případem, kdy se `touch` používá.


### Přesouvání, přejmenování
Soubor `draft.txt` by bylo fajn přejmenovat. Neobsahuje totiž osnovu závěrečné práce, ale básničku. Pro větší efekt si to ale zkus z jiné složky.

```console
$ cd ..
```

Přejmenování souboru je vlastně přesun - přemístění na jiné místo.
Příkaz `mv` bere jako první argument zdroj (co chceš přesunout) a poslední cíl (kam chceš přesunout). V našem případě to bude:
```console
$ mv thesis/draft.txt thesis/basnicka.txt
$ ls thesis/
basnicka.txt    soubor.dat
```

Pro vypsání obsahu souboru existuje příkaz `cat`:
```console
$ cat thesis/basnicka.txt
Haló haló,
co se stalo
```

Pochází z *concatenate*, původně sloužícího pro spojování více souborů:
```console
$ cat thesis/basnicka.txt thesis/basnicka.txt thesis/basnicka.txt
Haló haló,
co se stalo

Haló haló,
co se stalo

Haló haló,
co se stalo

```
Jako v případě `touch` se nejběžnější použití časem změnilo a momentálně se `cat` nejvíc používá pro výpis právě jednoho souboru.

`cat` vypíše obsah souboru, `ls` obsah adresáře (tedy jména souborů).

Kromě přesunutí můžeš soubor i kopírovat.
Příkaz `cp` funguje podobně jako `mv`, zkopíruje zdrojový soubor na nové místo.
```console
$ cp thesis/basnicka.txt thesis/basen.txt
$ cat thesis/basnicka.txt thesis/basen.txt
Haló haló,
co se stalo

Haló haló,
co se stalo

```
Co se stane, když dáš příkazu `ls` víc argumentů? Použije (vypíše) je všechny.
```console
$ ls thesis/basnicka.txt thesis/basen.txt
thesis/basnicka.txt thesis/basen.txt
```

Po všech těchto operacích máš ve složce `data-shell/thesis` spoustu věcí, které k ní nepatří. 
Básničky by bylo dobré přenést do složky s poznámkami. V `data-shell` si tedy udělej adresář `notes`. 
Abys nemusel{{a}} soubory přesouvat po jednom, můžeš dát příkazu `mv` víc argumentů, z nichž poslední znamená cíl – název adresáře, kam se mají všechny předchozí přesunout. 
```console
$ mv thesis/draft.txt thesis/basnicka.txt notes/
$ ls notes/
basen.txt    basnicka.txt
```


### Kopírování
Kdybys chtěl{{a}} zkopírovat adresář, nepodaří se to. Zkus příkaz:
```console
$ cp notes poznamky
```
Kopírování tímto způsobem nejde. Přesunout složku ale ano. V čem je rozdíl? Kopírovaná složka může být moc velká, takže ti to preventivně není umožněno.

Proto je potřeba říci explicitně, že chceš zkopírovat adresář a všechno co je v něm – rekurzivně (tedy včetně podadresářů, pod-podadresářů, pod-pod-podadresářů atd.).
Na to slouží přepínač `-r` (`--recursive`):
```console
$ cp -r notes poznamky
$ ls notes
basen.txt    basnicka.txt
$ ls poznamky
basen.txt    basnicka.txt
```


### Mazání

Smazat soubor můžeš pomocí `rm`. 
```console
$ rm notes/basnicka.txt
```
Tahle operace se ti povede. 
Smazání celého adresáře se ovšem nepodaří:
```console
$ rm notes
rm: cannot remove 'notes': Is a directory
```
Vyřeš to už známou fintou - pomocí přepínače `-r`, tedy *rekurzivní* smazání všeho, co v adresáři je.
```console
$ rm -r notes
```
**Varování: Mazání je napořád, není tu žádný koš!**
Operace se tiše povede, ale podobně jako v případě jiných příkazů se nedozvíš, co se přesně stalo. 
Proto může být přínosnější použít místo toho:
```console
$ rm -vr notes
removed 'notes/basen.txt'
removed 'notes/basnicka.txt'
removed directory 'notes'
```
které ti navíc řekne, které soubory se mažou. Přepínač `-v` (`--verbose`) zapne tzv. „ukecaný“ mód.


> [note] Pozor na kopírování z internetu
> Návody na internetu bývají většinou pomocné, ale občas se najdou lidi, kteří chtějí jiným znepříjemnit život. 
> Pomocí relativně krátkých příkazů si můžeš udělat velký problém se svým počítačem. 
> Proto buď obzvlášť opatrný při zadávání příkazů nalezených na internetových fórech: pokud nevíš co nějaký příkaz dělá, radši se poraď s někým komu věříš.
> Příklad takového velice zlého kanadského žertíku, který určitě NEZADÁVEJ, je `rm -r /`: příkaz na rekurzivní **smazání obsahu celého disku**.

## Masky názvů souborů

Přesuň se do složky `data-shell/data`.
Jak vypsat obsah všech textových souborů obsažených v této složce?

Už znáš příkaz `cat`. A tomu můžeš zadat spoustu souborů jako argumenty:
```console
$ cat amino-acids.txt animals.txt morse.txt planets.txt sunspot.txt
```

Takhle opisovat jména je ale zdlouhavé a nepříjemné. Jde to lépe? Ano.
Existují pro to znaky se speciálním významem.

* **hvězdička** ve jménu souboru zastupuje jakýkoli text. Je to takzvaná *divoká karta* (angl. *wild card*), která říká: místo hvězdičky může být „cokoli“.
  Následující příkazy jsou ekvivalentní – program `ls` dostává ty samé argumenty:
  ```console
  $ ls *.txt
  amino-acids.txt animals.txt morse.txt planets.txt sunspot.txt
  $ ls amino-acids.txt animals.txt morse.txt planets.txt sunspot.txt
  amino-acids.txt animals.txt morse.txt planets.txt sunspot.txt
  ```
  Příkaz:
  ```console
  $ ls */*.pdb
  ```
  ukáže všechny soubory v podadresářích, které mají příponu .pdb.
 
  Hvězdička nahrazuje jen jednoduché jméno souboru, nikoli lomítko. Například:
  ```console
  $ ls *.txt 
  ```
  neukáže soubor `animal-counts/animals.txt`.

* **otazník** - nahradí se jedním jakýmkoli písmenkem.

  Příkaz:
  ```console
  $ ls ???????.txt
  animals.txt    planets.txt
  ```

  Zobrazí jen soubory, které mají přesně 7 písmen v názvu.

Zkus o tom zjistit víc ve složce `data-shell/molecules`.
Kdy si ji vypíšeš, dostaneš několik anglických názvů:
```console
$ ls
cubane.pdb    ethane pdb    methane.pdb    octane.pdb    pentane.pdb    propane.pdb
```

Úkol: zjisti, co se stane, když použiješ tyto masky souboru ve složce `molecules`:
```console
$ ls *t*ane.pdb
$ ls *t?ane.*
$ ls *t??ane.*
```
Odpověď: 
1. `ethane.pdb`    `methane.pdb`    `octane.pdb`    `pentane.pdb`
2. `ethane.pdb`    `methane.pdb`
3. chybová hláška: žádné soubory, které odpovídají této masce, neexistují.

> [note]
> Hvězdičky předělává na konkrétní názvy souborů přímo Bash, takže pokud program `ls` dostane už seznam těchto názvů souborů, nikoli hvězdičky a otazníky. (Výjimkou je případ, kdy masce žádný soubor neodpovídá: pak Bash masku nechá beze změny.) Později v kurzu zjistíš, proč je to důležité.


## Superschopnosti Bashe

Když máš tolik malých prográmků, z nichž každý dobře dělá jednu věc, můžeš je dle chuti a potřeby spojovat dohromady.

Abychom si ukázali tu moc v plné kráse, ukážeme si ještě několik příkazů. 
* `wc` - _**w**ord **c**ount_ je příkaz, kterému můžeš dát jméno souboru a dostaneš vypíše několik čísel:

```console
$ wc data-shell/molecules/cubane.pdb
     20    156    1158   data-shell/molecules/cubane.pdb
```
Čísla znamenají počet řádků, slov a znaků v souboru. (Znaky se počítají všechny, včetně mezer.)

Výpis programu `wc` můžeš modifikovat pomocí přepínačů – podívej se do nápovědy pro další možnosti.

### Přesměrování s `>`

Když programu `wc` dáš několik souborů jako vstup, zpracuje postupně je všechny a přidá řádek *total* se součtem všech vstupů.
```console
$ wc molecules/*.pdb
  20  156 1158 molecules/cubane.pdb
  12   84  622 molecules/ethane.pdb
   9   57  422 molecules/methane.pdb
  30  246 1828 molecules/octane.pdb
  21  165 1226 molecules/pentane.pdb
  15  111  825 molecules/propane.pdb
 107  819 6081 total

```
Co se stane, když napíšeš jen `wc` bez dalších argumentů?
Program začne a čeká. O řádek níže bliká kurzor, ale bez promptu („otázky“) Bashe.
V tomto případě `wc` čeká na to, co mu napíšeš do příkazové řádky.
Vyzkoušej si to - zkus něco napsat. 
Až něco napíšeš, vstup ukonči pomocí klávesové zkratky <kbd>Ctrl</kbd>+<kbd>D</kbd>.
Dostaneš informaci o tom, kolik řádků, slov a znaků jsi právě napsal{{a}}.

```console
$ wc
Haló haló
co se stalo

      3       5      25
```

Programy, které dokážou zpracovat soubor uložený na disku, typicky dokážou také zpracovat vstup který napíšeš přímo do příkazové řádky. 
Mezi souborem na disku a tím co píšeš na klávesnici není tak velký rozdíl: pro program jsou to jen sekvence znaků, které se dají přečíst a zpracovat.

Ukážeme si to na příkladě `cat`.
Zadej `cat` bez argumentů a napiš `ahoj`<kbd>Enter</kbd>`čau`<kbd>Enter</kbd>. Program `cat` ti vstup zopakuje:
```console
$ cat
ahoj
ahoj
čau
čau
```

Program `cat` vypisuje, tak jako dříve, obsah souboru který dostal jako argument. V tomto případě to ale není soubor z disku, ale řádky které píšeš.

Zpět se vrať opět pomocí <kbd>Ctrl</kbd>+<kbd>D</kbd>. Tahle zkratka pošle programu `cat` signál, že vstup skončil.

Stejně tak program, který umí něco vypsat na obrazovku, umí svůj výstup přesměrovat jinam – uložit do nějakého souboru. 
Ale musíš si o to přímo říct.

Vypiš si počet řádků všech souborů `*.pdb` ve složce `molecules`:
```console
$ wc -l *.pdb
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 total
```

Vypadá to hezky, ale náš cíl je automatizovat si práci, ne obdivovat text na obrazovce :)
Chceme, aby počítač za nás něco počítal, ale nemusíme na to koukat.

Potřebné operaci se říká *přesměrování* (angl. *redirection*) a zapisuje se pomocí `>`. 
Před šípkou řekneš jaká operace se má provést; za šipkou pak kam se má uložit její výstup.

```console
$ wc -l *.pdb > lengths.txt
$ ls
cubane.pdb  lengths.txt  octane.pdb   propane.pdb
ethane.pdb  methane.pdb  pentane.pdb
$ cat lengths.txt 
  20 cubane.pdb
  12 ethane.pdb
   9 methane.pdb
  30 octane.pdb
  21 pentane.pdb
  15 propane.pdb
 107 total
```

Teď se můžeš podívat, kolik záznamů (řádků) je v souboru `lengths.txt`

```console
$ wc --lines lengths.txt 
7 lengths.txt
```
Je jich sedm: 6 pro jednotlivé soubory a navíc řádek *total*.

Existuje příkaz `sort`, kterému můžeš dát jméno souboru a on ho seřadí.

```console
$ sort lengths.txt 
 107 total
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
   9 methane.pdb
```

Co se stane, když spustíš `sort` bez jména souboru?
Bude čekat na vstup z klávesnice. Známou zkratkou <kbd>Ctrl</kbd>+<kbd>D</kbd> vstup ukončíš.

<pre><code>
$ sort
12
10
2
22
24
<kbd>Ctrl</kbd>+<kbd>D</kbd>
10
12
2
22
24
</code></pre>

Všimni si, že `sort` seřadí čísla, ale v abecedním pořadí.
Abys obdržel výstup seřazený numericky, použij přepínač `-n`.

<pre><code>
$ sort -n 
10
12
2
24
<kbd>Ctrl</kbd>+<kbd>D</kbd>
2
10
12
24
</code></pre>

A teď můžeš seřadit soubor `lengths.txt` podle počtu řádků v jednotlivých zanalyzovaných souborech:

```console
$ sort -n lengths.txt 
   9 methane.pdb
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
 107 total
```

Unixové systémy mají spoustu malých programů jako `wc`, `sort` a `cat` a nechávají na tobě, aby sis je sám pospojoval{{a}}.


### Přesměrování s `|`

Ukázali jsme si, jak můžeš pomoci pár příkazů udělat hned několik operací na souborech ve složce `molecules`.

Spočítal{{a}} jsi řádky:
```console
$ wc -l *.pdb > lengths.txt
```
Přečetl{{a}} jsi soubor:
```console
$ cat lengths.txt 
```
A seřadil{{a}} výstup:
```console
$ sort -n lengths.txt 
```

Občas bychom se ale rádi obešli bez vytváření pomocného souboru (jako `lengths.txt`) a radši pracovali přímo s výstupy jednotlivých prográmků. 
I to je v Bashi možné. 
Můžeš přesměrovat výstup a použít ho rovnou jako vstup pro další příkaz.
V našem případě chceme vidět soubory už rovnou seřazené podle počtu řádků. 

K tomuto účelu se výborně hodí *roura* (angl. *pipe*), zapisovaná svislítkem (`|`).
Následující příkaz spustí `wc -l *.pdb`, kterýu zanalyzuje počet řádků v příslušných souborech, a výstup (to, co by se vypsalo na obrazovku) pošle do programu `sort -n` (tak, jako bys to příkazu `sort` napsal{{a}} na klávesnici).

```console
$ wc -l *.pdb | sort -n
   9 methane.pdb
  12 ethane.pdb
  15 propane.pdb
  20 cubane.pdb
  21 pentane.pdb
  30 octane.pdb
 107 total
```

Nejen soubory na disku nebo vstup z klávesnice, ale i výstup z jiného programu je pro program jako `sort` jen sekvence znaků, které umí nějak zpracovat.

Přesměrování pomocí `>` a `|` je velmi mocná operace, kterou můžeš provést s jakýmkoli programem. Čím dřív si na tyto operace zvykneš, tím rychlejší a výkonnější během své práce s Bashem budeš.


#### Hadí odbočka
Program `python` má přepínač `-c`, který umožňuje provést pythonní příkaz přímo v příkazové řádce. 
I výstup této operace můžeš přesměrovat do souboru a dál s ním pracovat.

```console
$ python3 -c 'print(1 + 3)'
4
$ python3 -c 'print(1 + 3)' > four.txt
$ cat four.txt 
4
```

<!-- (Ukázat až po `echo`?)
Nebo můžeš do Pythonu přesměrovat výstup z jiného programu, který Python provede jako kdybys ho napsal{{a}} na klávesnici:

```console
$ echo 'print(3 + 4)' | python3
7
```

A všechno to jde řetězit a kombinovat dohromady:

```console
$ echo 'print(3 + 4)' | python3 > seven.txt
7
```
-->


## Čtení dlouhatánských souborů

Když se podíváš do souboru `octane.pdb`, zjistíš, že je obrovský: 

```console
$ cat octane.pdb 
COMPND      OCTANE
AUTHOR      DAVE WOODCOCK  96 01 05
ATOM      1  C           1      -4.397   0.370  -0.255  1.00  0.00
ATOM      2  C           1      -3.113  -0.447  -0.421  1.00  0.00
ATOM      3  C           1      -1.896   0.386  -0.007  1.00  0.00
…
ATOM     24  H           1       4.493  -0.641  -0.854  1.00  0.00
ATOM     25  H           1       4.368  -1.282   0.801  1.00  0.00
ATOM     26  H           1       5.254   0.230   0.498  1.00  0.00
TER      27              1
END
```

Moderní terminály nemají problém se scrollováním pomocí myši, ale pohodlné to není. 
Navíc, pokud bys byl{{a}} připojen{{a}} vzdáleně k serveru na ostrůvku v Tichém Oceánu, nebudeš mít žádnou myš k dipozici. 
Kdybys i přesto potřeboval{{a}} takový obrovský soubor přečíst, existují na to nástroje.

Program `less` ti pomocí šipek nahoru a dolů umožní scrollovat ve velkém souboru. 
Po celých stránkách můžeš navigovat pomocí <kbd>PgUp</kbd>/<kbd>PgDown</kbd>. A zmáčknutím <kbd>Q</kbd> `less` ukončíš. 

Existuje víc programů, které se ukončí pomocí klávesy <kbd>Q</kbd>. Dnes jsme si ukázali `man`; možná taky znáš příkazy `git log` a `git diff`. 
Víš, jak to dělají? Svůj výstup totiž přesměrují do programu `less`. Získají tak skrolování „bez práce“. 

V programu `less` se dá vyhledávat. Stiskem <kbd>/</kbd> přejdi do režimu hledání textu, za lomítko rovnou napiš text, který hledáš, a potvrď <kbd>Enter</kbd>.

> [note]
> Hledaný vzor je *regulární výraz*, kde má spousta znaků různé speciální funkce. Jestli si s regulárními výrazy nerozumíš, před každý speciální znak (`.`, `[` a podobně – cokoli kromě písmen, čísel, mezer), dej zpětné lomítko: třeba `abc.upper()` najdeš pomocí `\'abc\'\.upper\(\)`.

Další způsob, jak číst velký soubor, je příkaz `head`, který z něj ukáže prvních 10 řádků.

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

Můžeš taky přečíst 10 posledních řádků pomocí programu `tail`:

```console
$ tail octane.pdb 
ATOM     19  H           1       0.645   1.314  -0.386  1.00  0.00
ATOM     20  H           1       1.979  -0.666  -1.054  1.00  0.00
ATOM     21  H           1       1.855  -1.309   0.604  1.00  0.00
ATOM     22  H           1       3.030   0.696   1.467  1.00  0.00
ATOM     23  H           1       3.155   1.337  -0.188  1.00  0.00
ATOM     24  H           1       4.493  -0.641  -0.854  1.00  0.00
ATOM     25  H           1       4.368  -1.282   0.801  1.00  0.00
ATOM     26  H           1       5.254   0.230   0.498  1.00  0.00
TER      27              1
END
```

Výstupy obou těchto programů můžeš modifikovat pomocí přepínačů, které najdeš v dokumentaci.

Další pomocný nástroj pro práci s bashem je `echo`.
`echo` vypíše všechny argumenty, které dostane.

```console
$ echo haló haló
haló haló
```
K čemu je to dobré? Můžeš si takto otestovat, jestli operace, které se chystáš provést na souborech, dělají přesně to, co chceme, než je opravdu provedeme. Bash může jednotlivé argumenty trochu měnit, takže konkrétní program dostává trochu jiný seznam argumentů než co napíšeš.

Následující program říká Bashi: zavolej program `echo` a dej mu tyto tři argumenty. Všimni si, že mezery zde nehrají roli: oddělují jednotlivé argumenty; `echo` dostane tři argumenty bez mezer.

```console
$ echo ahoj      halo halooo
ahoj halo haloo
```

V následujícím příkazu `echo` dostane celkem 7 argumentů: `ahoj` a 6 jmen souborů, které Bash vytvoří z `*.pdb`. 

```console
$ echo ahoj *.pdb
ahoj cubane.pdb ethane.pdb methane.pdb octane.pdb pentane.pdb propane.pdb
```

Pokud potřebuješ zachovat řetězec i včetně mezer a hvězdiček, můžeš použít uvozovky. Bash bude všechno, co je uvnitř jednoduchých uvozovek (`'`), bere jako jeden argument.

```console
$ echo 'ahoj *.pdb'
ahoj *.pdb
```

Doporučuji používat uvozovky vždy, když si nejseš jist{{gnd('ý', 'á')}}, jestli něco Bash nebude považovat za speciální znak:

```console
$ echo '<o.o> je robot #1! :)'
<o.o> je robot #1! :)
```

> [note]
> Na rozdíl od Pythonu je v Bashi rozdíl mezi jednoduchými a dvojitými uvozovkami.
> Uvnitř dvojitých bude sice Bash zachovávat mezery, `*` a `?`, ale jiné znaky si speciální význam zachovají (příklad je `$`, které uvidíš níže).
> Uvnitř jednoduchých je speciální znak jen jeden: ukončující `'`.

Echo je velice jednoduchý program, který ti umožní mimo jiné zjistit, co všechno Bash provede se speciálními znaky, než je předá na zpracování dalším programům.

Připomeň si výstup:
```console
$ echo $0
/bin/bash
```

Místo `$0` je echu předáno `/bin/bash`. Samotné `echo` nic nepřevádí, pouze vypisuje přesně to, co dostává jako argument. To Bash provádí operaci, ve které `$0` nahradí konkrétním řetězcem.

`echo` je další způsob, jak vytvářet soubory. 
Znáš už `touch`, `nano` (případně jakýkoli jiný textový editor); teď k nim přidej přesměrování z `echo`:

```console
$ echo haló haló > basnicka.txt
$ cat basnicka.txt
haló haló
```
Co se stane, když budeš chtít dopsat do básničky další řádek?

```console
$ echo 'co se stalo?' > basnicka.txt
$ cat basnicka.txt
co se stalo
```
Pozor na to – přesměrováním se původní soubor přepíše. 

Pro přidání textu do souboru můžeš použit další druh přesměrování: `>>`.
S ním původní obsah v souboru zůstane a nový text bude přidán na konec souboru.

```console
$ echo haló haló > basnicka.txt
$ cat basnicka.txt
haló haló
$ echo 'co se stalo?' >> basnicka.txt
$ cat basnicka.txt
haló haló
co se stalo?
```

Jiný způsob, jak napsat něco do souboru, je přesměrovat vstup z `cat`. Text pak napíšeš přímo do příkazové řádky:

```console
$ cat > basnicka.txt
ahoj ahoj
jak se máš?
$ cat basnicka.txt
ahoj ahoj
jak se máš?
```

Ještě jiný způsob, jak uložit text do souboru, je použít `echo` a jednoduché uvozovky. 
Dokonce ti to umožní psát obsah souboru na víc řádků. 
Na konci je potřeba řetězec uzavřít a přesměrovat do souboru.

```console
$ echo 'halo halo
> co se stalo
> ...' > basnicka.txt
$ cat basnicka.txt 
halo halo
co se stalo
...
```

> [note] Rozdíl mezi `cat` a `echo`
> `cat` vypisuje obsah souboru, zatímco `echo` vypisuje to, co dostává jako *argumenty*.


Přečti si teď soubor `data-shell/data/animals.txt`.
Jedná se o zápis pozorování zvířátek v přírodě.
Tvým úkolem bude zjistit, jaká zvířátka byla aspoň jednou spatřená.

```console
$ cd ..
$ cat data/animals.txt
2012-11-05,deer
2012-11-05,rabbit
2012-11-05,raccoon
2012-11-06,rabbit
2012-11-06,deer
2012-11-06,fox
2012-11-07,rabbit
2012-11-07,bear
```

Vidíš, že data mají určitou strukturu: každý řádek obsahuje datum a zvířátko, oddělené čárkou. Pokud máš k dispozici takto pevně daný oddělovač, program `cut` ti pomůže rozdělit data na řádku (podle čárky jako *oddělovače*, angl. `--delimiter`, `-d`) a ukázat pouze druhý záznam (`--field`, `-f`) z rozdělené struktury. 

```console
$ cat data/animals.txt | cut -d ',' -f 2
deer
rabbit
raccoon
rabbit
deer
fox
rabbit
bear
```

Záznamy se ale pořád opakují. 
Ve zrušení duplikátů ti pomůže program `uniq`, který, pokud dostane několik opakujících se řádků, vypíše jen jeden. 
Má drobnou nevýhodu: porovnává vždy s posledním řádkem, který viděl, takže abys dosáhl{{a}} správné deduplikace, potřebuješ mu dát jako vstup vždy seřazený seznam.

Zkus napsat takový příkaz, který vypíše pouze unikátní záznamy zvířátek v souboru `data/animals.txt`.

{% filter solution %}

```console
$ cat data/animals.txt | cut -d ',' -f 2 | sort | uniq
bear
deer
fox
rabbit
raccoon
```
{% endfilter %}

Na tomto příkladu vyniká filozofie Unixu: mít malinkaté prográmky, které dobře umí dělat jednu věc a jsou jednoduché na implementaci, ale dají se dle potřeby spojovat. 

## Závěr - užitečné bashové příkazy
A je to! 
Co ses všechno naučil{{a}}?
    
```console
    ls
    ls -lhF
    ls --help
    pwd
    man <prikaz>
    cd
    mkdir nazev-slozky
    
    touch nazev-souboru
    nano nazev-souboru
    echo 'nějaký řetězec' > nazev-souboru
    cat nazev-souboru
    
    mv zdrojovy-soubor cilovy-soubor
    cp zdrojovy-soubor cilovy-soubor
    cp -r zdrojova-slozka cilova-slozka
    
    rm soubor
    rm -r slozka
    
    head
    head (-n 5)
    tail
    tail (-n 5)
    
    wc nazev-souboru
    sort
    sort (-n)
    uniq
    cut -d ',' -f 2
    
    >
    >>
    |
```

