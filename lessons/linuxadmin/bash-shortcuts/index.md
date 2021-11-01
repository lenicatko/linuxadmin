# Zkratky

Poznali jsme spoustu zkratek a kláves, které ti usnadní pohyb po příkazové
řádce.
Shrňme si je a přidejme pár dalších:

Některé zkratky umožňují hledat v historii:

* Šipky <kbd>↑</kbd> a <kbd>↓</kbd> listuje v historii zadaných příkazů.
* Klávesy <kbd>PgUp</kbd> a <kbd>PgDn</kbd> mačkej po tom, co napíšeš začátek příkazu.
  Listují v historii podobně jako šipky, ale omezují se na příkazy které začínají tím co
  už je na řádku napsáno.
  (Tohle ale není základní součást Bashe; na jiných systémech to nemusí být v základu nastaveno.)
* <kbd>Ctrl</kbd>+<kbd>R</kbd> spouští hledání v historii.
  Když stiskneš <kbd>Ctrl</kbd>+<kbd>R</kbd> a pak zadáš kousek textu a zobrazí
  se ti nejbližší příkaz, který text obsahuje.
  Když se chceš podívat dále do historie, opakovaně mačkáš <kbd>Ctrl</kbd>+<kbd>R</kbd>.
  Pro hledání opačným směrem („dopředu“) použij <kbd>Ctrl</kbd>+<kbd>S</kbd>
* <kbd>Levý Alt</kbd>+<kbd>.</kbd> doplní na místo kurzoru poslední „slovo“
  předchozího příkazu. Zadáváš-li tedy `ls nejaky-soubor.txt` a hned potom
  `head nejaky-soubor.txt`, můžeš ve druhém příkazu místo jména stisknout alt+tečku.

Mimochodem, příkaz `history` ti ukáže historii příkazů, tak jak si ji Bash pamatuje.
Každý příkaz tu má své číslo.
A když zadáš číslo za vykřičníkem, např. `!482`, Bash spustí příkaz s tím číslem

Další zkratky, které se můžou hodit:

*  <kbd>⭾TAB</kbd> je velmi užitečná klávesa, kterou už nejspíš taky znáš. Doplňuje příkazy a soubory, a když ji zmáčkneš 2×, zobrazí všechny soubory/příkazy, které začínají daným řetězcem.
* <kbd>Ctrl</kbd>+<kbd>C</kbd> zruší zadávání příkazu, nebo běh programu.
* <kbd>Ctrl</kbd>+<kbd>D</kbd> na začátku řádku ukončí Bash (často Bash k tomu ještě pro jistotu napíše `exit`) nebo ukončí vstup pro program (např. pro `cat`).
* <kbd>Ctrl</kbd>+<kbd>L</kbd> na začátku řádku vyčistí obrazovku.
* <kbd>Ctrl</kbd>+<kbd>W</kbd> smaže slovo před kurzorem.
* <kbd>Ctrl</kbd>+šipky <kbd>←</kbd><kbd>→</kbd> přesunují mezi slovy dlouhého příkazu.
* <kbd>Ctrl</kbd>+<kbd>S</kbd> pozastaví výstup (jakoby terminál *zamrznul*), ale příkazy se provádí.
* <kbd>Ctrl</kbd>+<kbd>Q</kbd> zobrazí vše, co bylo vytisknuto po <kbd>Ctrl</kbd>+<kbd>S</kbd> a „odmrazí“ terminál pro další příkazy.
* <kbd>Ctrl</kbd>+<kbd>Z</kbd> pozastaví právě prováděný příkaz a vrátí tě zpět do příkazové řádky.

Když ti tedy „zamrzne“ terminál, zkus ho odmrazit pomocí <kbd>Ctrl</kbd>+<kbd>Q</kbd>.


## Zkratky v grafickém prostředí

Nejen Bash umí spouštět příkazy.
Zkratky pro *celý systém*, nejen pro terminál, zpracovává na základní Fedoře
GNOME Shell.

Tyto zkratky se dají nastavit. V menu vpravo nahoře klikni na **Nastavení**,
vlevo vyber **Klávesnice** a vpravo **Přizpůsobit klávesové zkratky**.
Ukáže se několikaúrovňový seznam zkratek jako „Maximalozovat okno“ nebo
„Přepnout na pracovní plochu vpravo“ nebo „Zobrazit řádek ke spuštění příkazu“.
Spousta z nich používá klávesu <kbd>Super</kbd>: to je ta, která mívá na
klávesnicích logo operačního systému.

V seznamu se dá hledat – jak jméno akce tak i zkratku, např. `Alt+F2`.

Jde definovat i vlastní zkratku.
V podseznamu „Vlastní klávesové zkratky“ klikni na „Přidat klávesovou zkratku“.
Pak zadej jméno (to se ukáže v seznamu v nastavení), příkaz a
samotnou klávesovou zkratku.

Příkaz funguje podobně jako v Bashi, ale Bashové speciální znaky
jako `;`, `|` nebo `$` tu fungovat nebudou.
Když budeš chtít pouštět něco složitějšího,
vytvoř si vlastní příkaz a dej ho do `~/bin`!

Osobně doporučuju nastavit zkratku <kbd>Super</kbd>+<kbd>Enter</kbd>
na příkaz `gnome-terminal`, tedy otevření nového terminálu.
