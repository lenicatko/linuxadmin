# Úkoly – Bash II &c

## 1. Grepování yamlů

Procvič si příkazy jako `ls`, `wc`, `grep` a jak je spojit dohromady.

Stáhni a rozbal si tyto archivy s informacemi o komunitních akcích: kurzech/srazech PyLadies a srazech Pyvo.

    $ wget -O pyladies-cz.zip https://github.com/PyLadiesCZ/pyladies.cz/archive/master.zip
    $ unzip pyladies-cz.zip
    $ wget -O pyvo-data.zip https://github.com/pyvec/pyvo-data/archive/master.zip
    $ unzip pyvo-data.zip

Data si prohlédni a zjisti, co se v nich skrývá za informace.
Zvlášť doporučuju prohlédnout soubor `pyvo-data-master/series/brno-pyvo/events/2018-10-25-casove.yaml`
který je použit na stránce [pyvo.cz/brno-pyvo/2018-10/](https://pyvo.cz/brno-pyvo/2018-10/),
a soubor `pyladies.cz-master/teams/brno.yml`
který je použit na stránce [pyladies.cz/brno/#team](https://pyladies.cz/brno/#team).

Použij základní shellové příkazy (ne Python) na zodpovězení otázek níže.
Hledáš jen orientační hodnoty; nemusí to být na 100% přesné.

> [note]
> YAML soubory by se správně měly číst knihovnou na YAML, aby byla zachována struktura. Ty je ale ber jako "čistý text", kde hledané informace jsou na řádcích ve tvaru `klíč: hodnota` (případně s nějakýma mezerama a/nebo pomlčkama navíc).
> Proto odpovědi nemusí být na 100% přesné.
> 
> "Zakomentované" informace (začínající `#`) můžeš pro jednoduchost počítat
> taky. (I když jich je po COVIDu často víc než těch nezakomentovaných.)


1. Kolik bylo Pyv v Brně?
   * *Pro každý sraz existuje soubor.*
2. Kolik bylo Pyv celkem?
3. Z kolika přednášek na Pyvech jsou videa?  *(Předpokládej že každá přednáška může mít max. 1 video)*
   * *Videa jsou označena `video:`*
4. Kolik bylo kurzů/srazů PyLadies?
   * *Srazy jsou v adresáři `meetups/` a každý má jméno, `name:`*
5. (Bonusová výzva) Z kolika Pyv jsou videa?

### Nápověda

Šablonami jako `adresar/*/podadresar/*` můžeš vybrat soubory z více adresářů.

Příkaz `grep` má zajímavé přepínače `-r`, `-l`/`-L`, `-h`/`-H` a `-e`.

## 2. Uniq

Příkaz `uniq` odstraní *po sobě jdoucí* duplikované řádky:

```text
$ echo '
> jedna
> dva
> dva
> tři
> tři
> tři
> jedna
> ' | uniq

jedna
dva
tři
jedna

```

Často se používá `sort | uniq`, aby se stejné řádky z celého souboru dostaly k sobě.

Použij `uniq` k zodpovězení těchto otázek:

6. Vypiš všechna místa konání Pyv (stačí mít v rámci každého řádku identifikátor jako `artbar`).
7. Přidej informaci o tom, kolikrát na kterém místě Pyvo bylo. 

Příkaz `uniq` má zajímavý přepínač `-c`.

### Bonusová výzva

Existuje zajímavý příkaz `cut`, který má zajímavé přepínače `-d` a `-f`.

8. Jaké jsou 3 nejčastější křestní jména organizátorů/koučů/atd. PyLadies?
