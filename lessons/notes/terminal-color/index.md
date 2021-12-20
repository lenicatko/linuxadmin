
> [warning]
> Tohle jsou jen poznámky, ne materiály k samostudiu.

# Barvičky v terminálu

Když napíšeš `ls`, vypíše se barevný výstup. Jak to dělá? A hlavně, jak si to uděláš ty?

Krátká odpověď je, že použiješ knihovnu, například Pythonní [Click](https://click.palletsprojects.com/en/8.0.x/api/#click.style), což je knihovna na uživatelská textová rozhraní.
Taková knihovna se postará o veškerou správu: bude fungovat na různých terminálech, poradí si i se změnou velikostí okna terminálu a vyřeší spoustu dalších problémů.
Pokud budeš chtít obarvovat textový výstup, je to nejlepší cesta, jak dosáhnout cíle.

Je ale dobré vědět, na čem ty knihovny staví a jak to funguje na pozadí.
Začneme nahlédnutím do prehistorie :)

Původní terminály byly připojené k tiskárnám. 
Po zadání příkazu do terminálu se požadovaný výstup vytiskl na papír.
Neexistovala klávesa <kbd>Backspace</kbd> ani <kbd>ESC</kbd>, protože když jsi udělal{{a}} překlep, už byl dávno vytisknutý na papíře. 
Mohl{{a}} jsi příkaz maximálně zrušit a napsat nový.
Postupem času terminál získal nové funkce a začal ukazovat vepsaný text na obrazovce. 
U těch nových, grafických terminálů už by šlo opravit překlep. 
Jenže s touhle variantou se původně nepočítalo, a tak se řešilo, jak napsat na klávesnici <kbd>ESC</kbd>. 
To řešení bylo, že když budeš chtít smazat písmeno, pošle se speciální řídící znak, který znamená mazání.
Tak jako když zmáčkneš <kbd>Enter</kbd>, je to speciální znak `\n`, který přehodí kurzor na nový řádek, vznikl speciální znak, který posune kurzor o jedno místo zpátky a případně smaže poslední znak.

Když se podíváš do tabulky ASCII (`man ascii`), zjistíš, že ta má na začátku spoustu řídících znaků. 
Prvních 31 znaků na začátku jsou speciální znaky a mají různé významy jako začátek textu, začátek hlavičky, protože ASCII tabulka vychází z telegrafů a jiných forem komunikace ještě před internetem.

Dnes se jich používá jen pár: nový řádek, tabulátor a pár dalších.

Podívejme se ale na ten jeden konkrétní znak, 27 - ESC.
To znamená útěk z toho módu, když se mají vykreslit písmenka na obrazovku, a informaci, že se bude dít něco navíc. 
ESC uvozuje sekvenci znaku, která říká, co se má stát, např. *nastav barvu na červenou*.
Znak s číslem 27 nejspíš na své klávesnici nenajdeš, a proto je čas na ukázku v Pythonu. 
Hexadecimálně 27 je 1B. 

Vytvoř si soubor `barvicky.py` s obsahem níže:
```python
# barvicky.py
print('\x1B[1;31m slovo')
```
Když ho spustíš, výpis v terminálu bude červený.
```
       +--- tzv. kontrolní sekvence, která nastavuje barvu
       |
    ------
\x1B[1;31m
----
  |
  +--- je jediný znak, a to ASCII ESC
```

Kdy to přestane psát červeně? 
Až zavřeš terminál, nebo až napíšeš reset sekvenci.

Reset vypadá takto:
```python
print('\x1B[1;31m slovo   \x1B[0m')
```
1 znamená, že nastavuješ barvu, 31 je číslo konkrétní barvy.
Můžeš si udělat duhu v terminálu - vyzkoušej si skript:

```python
for i in range(30, 40):
    print(f'\x1B[1;{i}m slovo   \x1B[0m')
    
for i in range(40, 50):
    print(f'\x1B[1;{i}m slovo   \x1B[0m')
    
for i in range(256):
    print(f'\x1B[38;5;{i}mM\x1B[0m', end='')
    
for i in range(256):
    print(f'\x1B[48;5;{i}mM\x1B[0m', end='')    
```

S těmito sekvencemi bývá problém, protože existuje různé terminály a jejich dodavatelé si barvičky implementovali po svém. 
To se sice standardizovalo do tak zvaných ANSI sekvencí, kterých ukázku vidíš ve skriptu výše, ale i tak nejlepší varianta je použít nějakou knihovnu jako již zmíněný Click.

Existují další řídící sekvence, které umí pohybovat kurzorem.
Jedna základní je `\r`, která posune kurzor na začátek řádku.
Vykoušej si cyklus v příkladu níže.

```python
for i in range(100):
    print(i, "=" * i, end='\r')
    time.sleep(1)
```

Takto se například dají vykreslovat progress bary.
Má to ale i svá úskalí – například změna velikosti terminálu nemusí dopadnout dobře.
Pokud budeš mít za úkol napsát progress bar, určitě si na to nainstaluj knihovnu, která to správně vyřeší za tebe.

Další sekvence umí smazat obrazovku, pohybovat kurzorem a podobně.

Právě na tomto principu fungují terminálové editory.
Kdybys přesvědčil{{a}} `nano`, že to na co je napojený je terminál (což bohužel není snadné),
mohl{{a}} bys teoreticky udělat něco jako
```
$ nano > /tmp/nano-vystup
```
A výsledek si zobrazit třeba přes:
```
less /tmp/nano-vystup
```

Zkus si to s příkazem `sl`, který je potřeba doinstalovat:

```console
$ sudo dnf install sl
$ sl
$ sl > /tmp/sl
$ cat /tmp/sl
$ cat -t /tmp/sl   # `cat -t` vypíše znak "escape" jako `^[`
```
