
# 9. Linuxová administrace - cron, Ansible, barvičky

Dnes si vyzkoušíme přehrávat zvuky z příkazové řádky!
Jaké zvukové soubory máš k dispozici na počítači?

## Příkaz `locate`

Příkaz `locate` projde databázi všech souborů, které máš na disku a vypíše seznam všech souborů, které mají v názvu hledanou sekvenci písmen:

```console
$ locate .ogg
```

`locate` je `daemon`, který jednou za čas projde celý disk a zaindexuje všechny soubory.
Pokud sis tento příkaz právě nainstaloval{{a}}, tak ti ještě nic neřekne, protože je jeho databáze zatím prázdná. 
Tu si můžeš nechat vytvořit/zaindexovat pomocí `updatedb` (nutno spouštět jako `root`, či se `sudo`).

Pokud nemáš žádné nahrané zvuky, můžeš sáhnout po těch systémových.
Ty najdeš v `/usr/share/sounds`, kde jsou různé soubory ve formátu `.ogg`.


## Přehrávání zvuků z příkazové řádky

Proč se hodí přehrávat zvuk v příkazové řádce?

Protože pak to můžeš **automatizovat**. 
Třeba spouštět stále dokola.

My si to zkusíme s přehrávačem `mplayer`. 
Mrkni, jestli je v tvém systému. 
Pokud ne a jsi na Fedoře, nainstaluj si repozitář `free` z  [rpmfusion](https://rpmfusion.org/Configuration) 

```console
$ while true ; do mplayer <cesta/k/tvemu/zvuku>; done
```

Až to budeš chtít zrušit, zmáčkni `Ctrl+C`.


## Spouštění zvuku podle času

Jde to přehrát? Tak je čas na vyzkoušení, jak spustit zvuk v zadaný čas.

Řekneme počítači, aby v určitém intervalu něco opakoval. 
Příklad se zvukem ti možná nepřijde úplně prakticky (i když... pokud někdy budeš potřebovat budík a nabíječka k mobilu bude ta tam, touhle cestou si hravě poradíš!). 
Je to ale dobrý příklad, jak si ukázat princip **pravidelně plánovaných úloh**.

V praxi systémového administrátora se může stát, že budeš chtít každý den udělat zálohu databáze. 
Do speciálního konfiguračního souboru zadáš ve speciálním formátu informaci, že se má pustit každý den v určitou hodinu určitý skript.
Jiný příklad: můžeš si třeba chtít každý den na konci dne uložit své poznámky do gitu.

O spouštění programů podle času se stará démon `cron`. 
Jeho konfigurace se nastavuje přes `crontab`.

Má pěknou manuálovou stránku, kde si najdeš veškeré informace, co tento program dělá.
```console
$ man crontab
```

Na této stránce najdeš informaci, že přepínač `-e` spustí editor, do kterého si můžeš napsat, co a jak často se má spouštět. 

Co potřebuješ napsat do takového konfiguračního souboru, není napsané na této manuálové stránce.
`man` má totiž více *kapitol*, kde každá kapitola má spoustu stránek.
Základním příkazem `man` se dostaneš do kapitoly `1`, kde je napsané, jak se zadaný příkaz spouští a jaké má parametry.
Na konci první manuálové stránky je odkaz na `SEE ALSO crontab(5), cron(8)`.
Informace o tom, jak nakonfigurovat soubor, najdeš v kapitole 5. 

Tuto kapitolu otevřeš pomocí příkazu níže:

```console
$ man 5 crontab
```

A teď už je čas na nakonfigurování naší úlohy.

```console
$ crontab -e
```

Otevře se ti textový editor. 
Na některých operačních systémech jsou v něm už komentáře s vysvětlivkami, na jiných je tento soubor úplně prázdný.

Do otevřeného souboru napiš 5 hvězdiček a příkaz, který se má spouštět. 
Zaměň příkladovou cestu k zvukovému souboru za zvuk na tvém počítači.

```console
* * * * * mplayer /usr/share/sounds/gnome/default/alerts/bark.ogg
```

Každá z hvězdiček vyjadřuje jinou část času (minuta, hodina, ...). 
Aby sis to ale nemusel{{a}} pokaždé číst znova, doporučujeme si na první řádek dát komentář, co která pozice říká.
Výsledek:
```
# min hod den mesic den_tydne prikaz
  *   *   *   *     *         mplayer /usr/share/sounds/gnome/default/alerts/bark.ogg
```

Když to uložíš a zavřeš, `crontab` se nainstaluje.

5 hvězdiček znamená, že teď ti to každou minutu přehraje zvuk. Jupí! 

Toto nastavení vydrží i po vypnutí počítače. 
Zkrátka dokud si to zas nevypneš.

Počkej pár minut a ověř, jestli se to opravdu každou minutu přehrává.
Moc dlouho tě to asi bavit nebude, tak zkus snížit frekvenci, s jakou se zvuk bude přehrávat. 
Nastav přehrávání každou 50. minutu.

```console
$ crontab -e
50 * * * * mplayer /cesta/ke/zvuku
```
`*` znamená každou/každý. 

Zkus se zamyslet, co znamená toto nastavení:
```console
$ crontab -e
* 4 * * * mplayer /cesta/ke/zvuku
```
Odpověď: bude to přehrávat zvuk každou minutu od 4. do 5. hodin.

Proto je lepší nemít hvězdičky vlevo od hodnoty, kterou chceme nastavit.

Každý druhý den se napíše:
```console
$ crontab -e
0 0 /2 * * mplayer /cesta/ke/zvuku
```

A 2. den v měsíci takto:
```console
$ crontab -e
0 0 2 * * mplayer /cesta/ke/zvuku
```

`crontab` je mocný nástroj, ale i krypticky. 
V manuálu najdeš všechno, co potřebuješ k jeho nastavení.

[note]
> Cron může posílat maily, pokud jde z programu nějaký chybový výstup. 
> E-mail je věc starší, než je internet. 
> Původně se emaily posílaly v rámci jednoho počítače.

> Příkazem `mail` si je můžeš zobrazit. 
> Po čase je dobré se podívat, jestli ti je `cron` neposílá, což by nám po čase mohlo naplnit takovou schránku.


## Příkaz `watch`

Jiný zajímavý příkaz, který si dnes ukážeme, hodně pomáhá administrátorům, když chtějí v reálném čase koukat, jak se mění nějaká hodnota.

Znáš už příkaz `top`, který se automaticky aktualizuje, ale ne všechny příkazy toto dělají. 
Zkus například `date`, který ti řekne aktuální datum a čas.
Nebo `free`, který ukáže, jak kolik paměti je použito.
Jiný takový příkaz je `df`, který ukáže volné místo na disku. 

```console
$ df
Filesystem     1K-blocks      Used Available Use% Mounted on
udev             3889396         0   3889396   0% /dev
tmpfs             782916      2276    780640   1% /run
/dev/sda1      245084444 217345020  15220156  94% /
tmpfs            3914572    284800   3629772   8% /dev/shm
tmpfs               5120         4      5116   1% /run/lock
tmpfs            3914572         0   3914572   0% /sys/fs/cgroup
```

Můžeš použít přepínač `-h`, výpis pak bude čitelný pro lidi (v kB, MB, GB...).

```console
$ df -h
Filesystem      Size  Used Avail Use% Mounted on
udev            3,8G     0  3,8G   0% /dev
tmpfs           765M  2,3M  763M   1% /run
/dev/sda1       234G  208G   15G  94% /
tmpfs           3,8G  349M  3,4G  10% /dev/shm
tmpfs           5,0M  4,0K  5,0M   1% /run/lock
tmpfs           3,8G     0  3,8G   0% /sys/fs/cgroup

```
Problém s příkazem je, že vypíše aktuální stav a pak se ukončí.
Pokud něco zapisuje určité množství dat na tvůj disk, například nahráváš video a chceš koukat, jak moc se zaplňuje místo, musíš se podívat po lepším řešení.

Existuje program, které bude pouštět zadaný program v cyklu a zobrazí vždy poslední hodnotu.
Příkaz `watch` nám spouští zadaný příkaz každé 2s.
Zkus s programy výše:

```console
$ watch df -h
$ watch date
$ watch -d free
```
Toto zvýrazní rozdílné znaky od minulého zobrazení.

Pokud bys to chtěl{{a}} spouštět častěji, tak to zařídíme pomocí `-n` (čas v sekundách), takže třeba:

```console
$ watch -n 0.7 free
```

Ukončíš to přes <kbd>CTRL</kbd>+<kbd>C</kbd>.

Co umí `date`? Zkus některé přepínače.

```console
$ date -Is
2020-12-02T20:47:38+01:00
$ date -Id
2020-12-02
$ date -Im
2020-12-02T20:47+01:00
```

To je dostatečně hezký formát, aby se to dalo použít jako název souboru na Linuxu. 
Zde totiž nějaké dvojtečky a plusy v názvu souboru nevadí, ale pozor, na Windows by to mohlo zlobit.


## Ansible

Příklad: Představ si, že jsi systémový administrátor na serverové farmě. 
Staráš se o 500 počítačů. Každý z nich má nastavený `cron` na zálohy. 
Třetina má nastavený webový server, další třetina je velký cluster databází a poslední třetina dělá maily. 
Tvůj úkol je zajistit, aby se na všech těch serverech záloha dělala dvakrát častěji.

Co uděláš?
Připojíš se `ssh` k prvnímu serveru, pozměníš `crontab`. 
Hurá! Jen 499 a můžeš jít domů. 

Jiná varianta je, že si napíšeš v Bashi cyklus, který se připojí ke každému počítači a změní interval crontabu.
Budeš mít z toho skvělý pocit a práce bude odvedená během chvilky.
...Jenže pak zjistíš, že u serveru číslo 54 se to jaksi nepovedlo. 
Což znamená, že bys měl{{a}} zjistit, jestli u ostatních serverů to dopadlo dobře, nebo taky ne.

Změny konfigurace na vícero strojích se dnes už nedělají tak, že jdeš k počítači a fyzicky na něm upravíš konfigurační soubor, a následně restartuješ příslušnou službu.

Představ si, že na tvé farmě právě shořel jeden z tvých serverů. 
Nakoupíš nový počítač a chceš, aby se nakonfiguroval úplně stejně jako ten předchozí. 
Potřebuješ určitou verzi databáze s určitou konfigurací, a taky tam potřebuješ nastavit `crontab`, a ještě dát tam změnu z úterka, s tou zvýšenou frekvencí záloh.

Používají se na to systémy, kde si do souboru napíšeš, jak má tvůj systém vypadat a pustíš program, který se připojí na těch 500 serverů a každý nastaví podle toho seznamu.
Pak shoří server, vyměníš ho za jiný a pustíš ten program znova. 
Důležité je, že nastavení toho počítače máš v gitu.
Je to verzovaný soubor, a tak se můžou sledovat změny, které kdo v tom nastavení dělal.

Ansible dělá přesně takové věci. 
Popíšeš si systém, pustíš Ansible, řekneš mu, na jaké počítače se má připojit, on se přípojí a počítač nastaví.

Nevýhoda je, že je to další systém, který se musíš naučit konfigurovat. 
Dnes je to ovšem čím dál rozšířenější a systémoví administrátoři s Ansiblem nebo podobnými systémy (jako *Puppet*) běžně pracují.


### Nainstalujeme si nějaký *užitečný program*

Nejlepší způsob, jak se naučit pracovat s Ansiblem, je říct si, jaký balíček chceš mít vždy na svém systému.
A ten si nakonfigurujeme.

Nainstaluj Ansible:

```console
$ sudo dnf install ansible
```

Udělej si nový adresář pro tuto lekci.

Pro Ansible se píšou tzv. *playbooky*, což jsou konfigurační soubory ve formátu `yml`/`yaml`. 
[YAML](https://en.wikipedia.org/wiki/YAML) vyžaduje určité odsazení v souboru a bude si stěžovat, pokud ho nedodržíš. 

Napíšeme si konfigurační soubor s názvem `setup.yml`.
Protože chceš konfigurovat jen vlastní počítač, `hosts` nastav na `localhost` a `connection` - `local`.
```
# setup.yml

- hosts: localhost
  connection: local
```

Pokud bys to používal{{a}} i s dalšími počítači, správně by se nadefinovaly v samostatném souboru.

Abys mohl{{a}} něco nainstalovat, je třeba býti `root`em.
```
   become_user: root
```

Dále se píše stav, do jakého chceš dostat svůj systém.
Nejde o příkazy, ale o cíl, jak to má na konci vypadat.
Až potom ten soubor pustíš, nebude provádět příkazy, ale podívá se, jestli je počítač ve správném stavu. 
Když není, tak ho tam dostane. 
Když pustíš Ansible dvakrát za sebou, poprvé udělá nějakou změnu, a podruhé si všimne, že už není co dělat, a tak žádnou změnu neudělá. 
Vytvoř si tedy `tasks`. Úlohy, tedy `tasks`, jsou seznam. V `yml` se seznamy definují pomlčkou. 
Každá úloha musí mít jméno.
Nainstalujeme si `htop`, což je vylepšený `top`. 
Pro instalaci musíš být `root`, a proto se to nastavení potvrdí v úloze (`become: yes`).

```
   tasks:
   - name: Install htop
     become: yes
```

Přichází čas na instrukci, jak balíček nainstalovat.
Na Fedoře se instaluje pomocí `dnf`. 
Pak řekneš, že balíček `htop` chceš nainstalovat v poslední dostupné verzi.

```
     dnf:
        state: latest
        name:
        - htop
```


Celý blok nakonec vypadá takto. Pozor na odsazení:
```
- hosts: localhost
  connection: local
  become_user: root
  tasks:
   - name: Install htop
     become: yes
     dnf:
        state: latest
        name:
        - htop
```

Teď to zkus spustit:

```console
$ ansible-playbook -K setup.yml
BECOME password: 
[WARNING]: provided hosts list is empty, only localhost is available. Note that
the implicit localhost does not match 'all'

PLAY [localhost] ***************************************************************

TASK [Gathering Facts] *********************************************************
ok: [localhost]

TASK [Install htop] ************************************************************
changed: [localhost]

PLAY RECAP *********************************************************************
localhost                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0  
```

Díky parametru `-K` se tě Ansible nejdřív zeptá na `sudo` heslo a s tím pak spouští celý *playbook*.

Pokud máš nainstalovaný `cowsay` a dostáváš tak výpis příkazu do obrázků kraviček, můžeš si to zkusit odinstalovat. 
Pod `tasks` dopiš další odrážku:

```
   - name: Uninstall cowsay
     become: yes
     dnf:
        state: absent   # chceš, aby nebyl nainstalován
        name:
        - cowsay
```

Když ten *playbook* spouštíš dvakrát po sobě, tak to sice chvilku trvá, ale už ti to řekne `ok=3 changed=0`, protože druhé spuštění už nic nezměnilo.

Co všechno se dá dělat s Ansiblem?
- instalovat balíčky (to jsme si právě zkusili)
- kopírovat soubory - např. konfigurační soubor pro `cron`.
- přidávat uživatele, skupinu uživatelů
- a mnohem víc...

Existují pro to tzv. *pluginy*, které dělají přesně tyto věci. 
Pokud ti nebude stačit nabídka pluginů, můžeš si v Pythonu napsat další.

Tento příklad se připojuje pouze k aktuálnímu počítači. 
Pokud bys to chtěl{{a}} pouštět i na jiných počítačích, je třeba je zapsat do takzvaného *inventáře* (*inventory* file - `/etc/ansible/hosts`). Návod a příklady najdeš v dokumentaci programu.


### Barvičky v terminálu

Když napíšeš `ls`, vypíše se barevný výstup. Jak to dělá? A hlavně, jak si to uděláš ty?

Krátká odpověď je, že použiješ knihovnu, například Pythonní [Click](https://click.palletsprojects.com/en/7.x/), což je knihovna na uživatelská textová rozhraní.
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
Má to ale i svá úskalí, například změna velikosti terminálu nemusí dopadnout dobře. 
Pokud budeš mít za úkol napsát progress bar, určitě si na to nainstaluj knihovnu, která to správně vyřeší za tebe.

Pozor, na Windows to fungovat nebude, tam se to řeší přes speciální systémovou funkci "nastav barvu terminálu". 


*To by chtělo trošku líp rozepsat, pokud bude v materiálech, vypadá to sice cool, ale i na druhý poslech je to celkem magic.*

Pro zajímavost, terminálové editory fungují právě na tomto principu *escape sekvencí*.
Když přesvědčíš `vi`, že to, na co je napojený, je terminál, můžeš udělat něco jako
```
$ vi > /tmp/session
```
A to si zobrazíš třeba přes:
```
less /tmp/session
```
