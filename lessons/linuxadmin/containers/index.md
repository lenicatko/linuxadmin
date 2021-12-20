# Kontejnery

Když vývojář naprogramuje nový program, na jeho počítači to zpravidla funguje dobře. 
Až ho má hotový a chce se o něj podělit s kolegy, dost často zjistí, že to u kolegů neběží,
protože jim zapomněl říct o důležité změně konfigurace v `/etc` nebo o potřebných balíčcích.
A nebo mají úplně jiný operační systém.
Tento problém se stává natolik často, že zasloužil si vlastní vtip: *works on my machine*.
Systémovým administrátorům podobné situace vadí a kontejnery mají za cíl to řešit.

Jak je to možné?
Kontejner neobsahuje jen program, ale celý souborový systém – `/` a všechno v něm, se všemi knihovnami a nastavením.
Při spuštění kontejneru se pak pro konkrétní porces nastaví jiný *kořenový adresář* (`/`) než pro zbytek systému.
Tento tedy uvidí úplně jiné soubory než zbytek systému.

Tenhle souborový systém pak můžeš vzít, zabalit do archivu a poslat někomu jnému,
u koho se pak bude spuštěný program chovat stejně.
V reálu je potřba nasdílet trochu víc informací než jen ten souborový systém,
ale i „to navíc“ se dá do archivu přidat.


## Sjednocení a izolace

Kontejnery poskytují **sjednocené** prostředí: okolní systém neovlivňuje program, který tak na všech systémech poběží stejně.
Zároveň řeší i **izolaci** prostředí: puštěný program neovlivňuje zbytek systému.

Pokud nějakému programu nevěříš stoprocentně a nechceš, aby ti napáchal problémy v počítači,
můžeš ho spustit v kontejneru, který zařídí izolaci.
Cokoli se takový proces snaží provést se provede jen v izolovaném prostředí kontejneru.

Izolace je trochu složitější než jen nastavení jiného kořenového adresáře.
Programu puštěnému v kontejneru může systém ukázat jiný seznam **uživatelů**:
`root` (uživatel číslo 0) v kontejneru může být pro zbytek systému obyčejný uživatel,
který má jen omezená oprávnění.
Můžeš změnit **síťování** - nastavit, jestli proces může komunikovat s internetem, nebo jen s konkrétními programy/pčítači.
Můžeš určit kolik **prostředků** smí procesy v kontejneru spotřebovat,
třeba max. 2kB paměti a 30s procesorového času.
Když bude chtít víc, tak ho systém ukončí.

Všechna omezení kontejneru se pak vztahují i na případné další procesy,
které ten první spustí.


## Kontejnery vs. virtuální stroje

Teoreticky by tedy kontejnery měly být bezpečné: mají svůj vlastní píseček a neměly by sahat na zbytek systému.
Je tu ale vždy možnost bezpečnostních chyb v samotném kontejnerovém systému.
Mnohem bezpečnější je v tomto ohledu virtuální počítač, který *simuluje* celý stroj – disk, obrazovku, procesor, atd.
Umožňuje ti spustit jiný operační systém, nebo dokonce systém určený pro jiný procesor.
(Znáš-li emulátory herních konzolí: to jsou taky virtuální počítače.)

Virtuální stroje ovšem mají podstatnou nevýhodu: jsou relativně velké (na disku i v paměti) a pomalejší než kontejnery.
Kontejnery totiž sdílejí jádro operačního systému s „hostitelem“.
A mezi sebou toho mohou sdílet víc.

> [note] Jiné operační systémy
> Kontejnery jdou spouštět i na Windows nebo macOS, kde není linuxové jádro.
> Na nich se použije virtuální stroj s Linuxem, ve kterém se pak spouští
> jednotlivé kontejnery.


## Sdílení vrstev

Díky kontejneru můžeš vzít program s celým prostředím a pustit ho na jiném systému.
Zde bývá problém, protože operační systémy bývají velké.
Například všechny knihovny – všechno v `/usr/lib*` – které jsou potřeba k běhu programu v kontejneru musí být součástí kontejneru.
Nemůžeš prostě použít kousky z hostitelského počítače – ty by nemusely být kompatibilní.
Všechno kromě jádra se musí do kontejneru nakopírovat.

Teď si představ, že máš na počítači 10 různých služeb, tři databáze,
dva webové servery a aplikaci, a pro každý druh této služby potřebuješ mít zvlášť celý souborový systém. 
Je toho celkem hodně.

A proto kontejnery používají *vrstvy*.
Ukazovali jsme si, jak fungují [zařízení a souborové systémy](../../linuxadmin/mount) a že existuje spousta jejich druhů.

Jeden ze zvláštních souborových systémů je `overlayfs`.
Je zvláštní v tom, že má všechna data uložená ve dvou *jiných* souborových systémech,
tzv. vrstvách:

* V první je základní souborový systém, který je pouze pro čtení.
* Do druhé se ukládají změny – rozdíl oproti první vrstvě.

Když je potřeba v *overlayfs* přečíst soubor,
hledá nejprve ve druhé vrstvě, a pokud se tam najde, přečte se z ní.
Pokud se nenajde, přečte se z první vrstvy.

Když je potřeba do souboru zapsat, zapisuje se pouze do druhé vrstvy.

> [note] Paralela s novelizací zákonů
> Zákony se většinou po vydání nemění.
> Pokud vznikne potřeba změnit určitý odstavec (nebo zákon zrušit),
> zveřejňuje se novela – aktualizace konkrétního paragrafu.
> Když chceš znát *aktuální znění* zákona, musíš se podívat do obou dokumentů.

K čemu to je dobré?
Představ si že máš systém, na němž má běžet 5 kontejnerů: 1 databáze a 4 webové servery,
můžeš mít jeden základ pro všechny – základní systém.
Potom vytvoříš dva `overlayfs`, které *oba* používají tuto základní vrstvu, ale v jednom doinstaluješ webový server a ve druhém databázi.
Teď si představ, že máš dvě webové aplikace:
dva z webových serverů potřebují knihovnu Flask a zbylé dva Django.
Vezmeš vrstvu se základním webovým serverem a nad ním vytvoříš další dva `overlayfs`: jeden pro Flask a druhý pro Django.
Společné části systému tak můžeš sdílet mezi všemi kontejnery, které je potřebují.

```plain
                 +------------------+         +---------------+
                 |      Flask       |         |     Django    |
                 +-+--------------+-+         +--+----------+-+
                   |              |              |          |
               +---v-+            |              |         +v----+
               |     |            |              |         |     |
               +-----+            |              |         +-----+
                                  |              |
    +-------------------+      +--v--------------v-+
    |     Databáze      |      |        Web        |
    +-+--------------+--+      +-----+----------+--+
      |              |               |          |
      |              |               |          |
      |              |               |          |
+-----v-+          +-v---------------v--+     +-v------+
|       |          |                    |     |        |
|       |          |  Základní systém   |     |        |
|       |          |                    |     |        |
+-------+          +--------------------+     +--------+
```

Univerzální základní vrstvy, které jsou užitečné více lidem,
se dají sdílet po internetu.


## Docker Hub

Aktuálně nejpopulárnější implementace kontejnetů je **Docker**.
Byl to první produkt který se v oblasti kontejnerizace masově prosadil.
Pro spoustu uživatelů je synonymem pro kontejnery,
i když existují i jiné technologie které dělají v podstatě to samé.

Firma Docker, Inc. provozuje portál [Docker Hub](https://hub.docker.com/),
kde jsou ke stažení obrazy kontejnerů – ony užitečné základní vrstvy.
Kdokoli si může nějakou vytvořit a na Docker Hubu ji sdílet.

Podívej se na stránku [kontejner s `httpd`](https://hub.docker.com/_/httpd):
uvidíš popis, odkazy na dokumentaci a různé verze, a podobně.


## Docker a Podman

Docker můžeš nainstalovat z balíčku `docker`.
Po instalaci je potřeba spustit službu stejného jména.

Jiná implementace kontejnerů se jmenuje *Podman*.
Sice není tak populární, ale na Fedoře funguje trošku líp.
Výhodou je, že nepotřebuje běžící službu (démona), který běží pod
superuživatelským účetm a k jehož ovládání
potřebuješ být ve speciální skupině.
Jinak se ale používá úplně stejně,
jen si typicky budeš muset vybrat jestli
chceš kontejnery stahovat z Docker Hubu nebo odjinud.
Nainstaluj si tedy ten:

```console
$ sudo dnf install podman
```

Všechny příkazy níže budou fungovat i s `docker` místo `podman`,
pokud máš na systému `docker` nastavený.

> [note] Instalace Dockeru
>
> Chceš-li použít Docker, instalace je následující:
>
> * Nainstaluj si balíček `docker`
> * Pusť příslužnou službu:
>    ```console
>    sudo systemctl start docker
>    ```
>
> * Přidej se do skupiny `docker`.
>    ```console
>    $ sudo usermod -a -G docker $(whoami)
>    ```
>
> * Znovu se přihlaš, aby se změna skupiny projevila.
>   Buď se odhlaš ze systému a znovu přihlaš,
>   nebo pro jeden terminál použij `su -`:
>   ```console
>   $ su - $(whoami)
>   ```
>
> Teď můžeš pokračovat dál, jen `podman` zamněň za `docker`, např.
> `docker pull httpd`.


## Stažení obrazu

Když máš nainstalováno, můžeš si stáhnout si základní obraz kontejneru
(základní vrstvu souborového systému), třeba pro webový server – `httpd`:

```console
$ podman pull httpd
```

V menu vyber kontejner z Docker Hubu – `docker.io` (`/library/httpd:latest`).

Příkaz `pull` stáhl spoust tzv. *blobů*.
To jsou jednotlivé vrstvy: základní systém, aktualizace k němu, webový server, atd.
Pokud už nějakou máš staženou z dřívějška (třeba pro jiný kontejner), rovnou se použije, nebude se stahovat podruhé.


## „Jednosměrná“ izolace

Stahování pomocí `pull` ale často není potřeba: když chceš spustit obraz
který ještě nemáš stáhne se automaticky.
Zkus si to s jiným kontejnerem: `ubuntu`,
tedy [základním kontejnerem](https://hub.docker.com/_/ubuntu)
pro tuto [distribuci Ubuntu](https://ubuntu.com/).

```console
$ podman run -it ubuntu
root@934745e0585a:/# 
```

Příkaz výše stáhne kontejner Ubuntu a v něm spustí Bash.

Přepínače `-it` připojí ke kontejneru aktuální terminál,
abys Bash mohl{{a}} ovládat.

Zkus to – spusť následující příkazy:
* Ověření, že vše funguje jak má:
  ```console
  root@934745e0585a:/# echo Ahoj!
  Ahoj!
  ```

* Kontrola že adresář `home` je prázdný – jsi v izolovaném prostředí:
  ```console
  root@934745e0585a:/# ls -a /home
  .  ..
  ```

* Kontrola že jsi `root`, uživatel číslo 0:
  ```console
  root@934745e0585a:/# id
  uid=0(root) gid=0(root) groups=0(root)
  ```

* Seznam procesů a PID aktuálního shellu:
  ```plain
  root@934745e0585a:/# ps -e
  USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
  root           1  0.0  0.1   4116  3560 pts/0    Ss   14:08   0:00 bash
  root           8  0.0  0.1   5904  2944 pts/0    R+   14:17   0:00 ps aux
  root@934745e0585a:/# echo $$
  1
  ```
  Na systému neběží nic než Bash a to co z něj spustíš.

  Bash má PID 1.
  Na normálním systému by měl PID 1 `systemd`, ale tady žádní démoni neběží.

A nakonec pusť `sleep`, na který se podíváme „zvenku“.
```console
root@934745e0585a:/# sleep 1000
```

Otevři si v jiném terminálu (svého systému) příkaz `htop` a pomocí <kbd>F5</kbd> si zobraz stromový výpis. 
Výpis vyfiltruj aby ukazoval jen program `sleep 1000` z kontejneru (<kbd>F4</kbd>, `sleep`, <kbd>Enter</kbd>).
Všimni si, že běží pod *tvým* uživatelským účtem, i když uvnitř kontejneru jsi `root`.

> [note]
> `htop` jsme instalovali minule; jestli ho nemáš, nainstalij ho:
> `sudo dnf install htop`.

V kontejneru běží normální procesy, které „vidíš zvenku“.
Izolace spočívá v tom, že mají nastavený jiný adresář `/`, přenastavené uživatelské účty, a podobně.

Pokud bys to samé udělal{{a}} s virtuálním počítačem, najdeš na hostitelském systému jen jeden proces,
který simuluje virtuální procesor, disk, jádro systému a *všechny* procesy v něm.
Procesy ve virtuálním stroji tedy na hostitelském systému nenajdeš – izolace je tam úplnější.

Neúplná („jednosměrná“) izolace má svoje výhody.
Můžeš třeba ukončit proces v kontejneru z hlavního systému:
přímo v `htop` zmáčkni <kbd>k</kbd><kbd>Enter</kbd>, což pošle signál `SIGTERM`.


## Instalace v Ubuntu

Zkus teď zjistit IP adresu kontejneru.

Když se pokusíš použit příkaz `ip a`, zjistíš, že tady není a musí se doinstalovat.
To se na Ubuntu dělá jinak než na Fedoře – dvěma příkazy:

```console
# apt-get update
# apt-get install iproute2
```

Na „normální“ verzi Ubuntu, nainstalované pro uživatele by příkaz `ip` byl k dispozici.
Systémy určené pro kontejnery jsou ale „ořezané“ – mnohem menší než "plné" distribuce.
Všechno co systémový administrátor pro konkrétní použití potřebuje je potřeba doinstalovat.
(Nebo použít vrstvu kde už je to doinstalované – jako `httpd`.)
Údržba takového systému je pak mnohem méně náročná než velkého „kombajnu“.
Čím méně nainstalovaných programů v kontejneru je, tím bezpečnější by to mělo být.

> [note]
> V „normální“ verzi Ubuntu potřebuje instalace `sudo`:
> ```console
> $ sudo apt-get update
> $ sudo apt-get install iproute2
> ```
> V kontejneru jsi `root`, takže není potřeba přidávat `sudo`.
> A navíc tu ani `sudo` není k dispozici, musí se taky instalovat zvlášť.

Pro vyskočení z ubuntí příkazové řádky stačí ukončit terminál,
například příkazem `exit` nebo zkratkou pro konec vstupu, <kbd>Ctrl</kbd>+<kbd>D</kbd>.


## Správa obrazů a kontejnerů

V téhle oblasti se setkáš se dvěma termíny, které se často pletou:

* **Obraz** (angl. *image*) obsahuje souborový systém a další
  informace, které potřebuješ ke spuštění kontejneru.
  (Například program který se má spustit.)
  Obraz můžeš stáhnout nebo s někým nasdílet.
  Jednou vytvořený obraz už se nedá měnit,
  ale dá se zdílet mezi několika kontejnery.
* **Kontejner** je něco co běží s daným obrazem.
  Může zapisovat nová data (do vlastní *overlayfs* vrstvy).
  Kontejner můžeš spustit nebo zastavit.

Pomocí příkazu `podman ps` si můžeš vypsat seznam všech stažených obrazů:

```console
$ podman images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
httpd               latest              2ae34abc2ed0        13 days ago         165MB
ubuntu              latest              775349758637        5 weeks ago         64.2MB
```

A příkaz `podman ps` vypíše všechny právě běžících kontejnerů.
Pokud ti právě v jiném terminálu běží Ubuntu, uvidíš něco jako:

```console
$ podman ps
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
c9f835211bbb        ubuntu              "/bin/bash"         39 seconds ago      Up 38 seconds                                   focused_khayyam
```

Když kontejner doběhne, zastaví se, ale zůstane na disku.
To se může občas hodit – když proces v kontejneru skončí s chybou,
budeš možná chtít zkopírovat data, která zapsal na disk.

Výpos všech kontejnerů – i těch které neběží – dostaneš pomocí přepínače `--all`:

```console
$ podman ps --all
CONTAINER ID        IMAGE               COMMAND             CREATED             STATUS                      PORTS               NAMES
4a523c41344d        ubuntu              "/bin/bash"         10 minutes ago      Exited (0) 2 minutes ago                        nostalgic_williams
```

Podle ID můžeš kontejner znovu „oživit“
(pustit v něm nový proces, pro Ubuntu Bash).
Slouží k tomu příkaz `start`.
Přepínač `--attach` připojí standardní vstup a výstup:

```console
$ podman start --attach 4a523c41344d
4a523c41344d:/# ls
bin   dev  home  lib32  libx32  mnt  proc  run   srv  tmp  var
boot  etc  lib   lib64  media   opt  root  sbin  sys  usr
4a523c41344d:/# exit
```

Zastavené kontejnery můžeš smazat pomocí: 
```console
$ podman rm <container-id>
```

Obdobně smažeš obrazy (ale funguje to jen pokud příslušný obraz nic nepoužívá):
```console
$ podman image rm <image-id>
```

Zastavené kontejnery zabírají místo na disku.
Když budeš chtít kontejner po doběhnutí rovnou smazat,
spusť ho (`run`) s přepínačem `--rm`.


## Webový server v kontejneru

Zkus místo `ubuntu` pustitkontejner s webovým serverem:

```console
$ podman run -it --rm httpd
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
AH00558: httpd: Could not reliably determine the server's fully qualified domain name, using 172.17.0.2. Set the 'ServerName' directive globally to suppress this message
[Sun Dec 13 15:35:46.045992 2020] [mpm_event:notice] [pid 1:tid 139871236002944] AH00489: Apache/2.4.41 (Unix) configured -- resuming normal operations
[Sun Dec 13 15:35:46.057586 2020] [core:notice] [pid 1:tid 139871236002944] AH00094: Command line: 'httpd -D FOREGROUND'
```

Image `httpd` je nastaven tak, že nepustí `bash`, ale spustí přímo server `httpd` hned na popředí.
Nejsou tu potřeba žádní démoni: jediný úkol toho kontejneru je pustit webový server, a přesně to dělá. 
Když `httpd` ukončíš (<kbd>Ctrl</kbd>+<kbd>C</kbd>), ukončí se i kontejner (a protože používáš přepínač `--rm`, všechno v něm se smaže).

Místo `-it` můžeš kontejner spustit taky s přepínačem `-d` (z angl. *detached*, odpojený),
což znamená, že se spustí na pozadí.
Můžeš tak spustit kontejnerů víc.

```console
$ podman run -d --rm httpd
$ podman run -d --rm httpd
$ podman run -d --rm httpd
$ podman run -d --rm httpd
```
Až je budeš chtít smazat, použij příkazy 
```console
$ podman stop <container-id>
$ podman rm <container-id> 
```

(ID kontejneru, `<container-id>`, ti vypsal `podman run`.
Můžeš ho zjistit i zpětně z `podman ps --all`.)

My se ale chceme podívat do webového serveru, takže spustíme v kontejneru `bash`.
Ne na pozadí (`-d`), ale interaktivně s aktuálním terminálem (`-it`):
```console
$ podman run -it --rm httpd bash
root@f0a4bf04ccc6:/usr/local/apache2# 
```

Podívej se do adresáře `htdocs`. Je v něm soubor `index.html`, který obsahuje větu *It works!*

```console
root@8933505e2865:/usr/local/apache2# ls
bin  build  cgi-bin  conf  error  htdocs  icons  include  logs	modules
root@8933505e2865:/usr/local/apache2# cd htdocs/
root@8933505e2865:/usr/local/apache2/htdocs# ls
index.html
root@8933505e2865:/usr/local/apache2/htdocs# cat index.html 
<html><body><h1>It works!</h1></body></html>
```

Kontejner teď zavři, napiš do příkazové řádky `exit`.
Díky `--rm` se kontejner po ukončení automaticky smaže – ale to nevadí,
nic podstatného jsme v něm nezměnili.
Stránka `index.html` je součástí obrazu `httpd`, takže bude vidět
z každého kontejneru spuštěného z tohoto obrazu.

Na tu stránku s *It works* se chceme podívat „zvenku“,
z virtuálního počítače na kterém který kontejner pouštíš. 
Proto spusť `httpd` s přepínačem `-p`, který zveřejňuje porty:

```console
$ podman run -p 8080:80 --rm httpd
```

Port `80` v kontejneru, se teď na tvém (virtuálním) počítači zpřístupní
jako port `8080`.
Když teď zadáš do prohlížeče adresu `localhost:8080` uvidíš stránku *It works!*.
Případně se můžeš podívat z terminálu:

```console
$ curl localhost:8080
<html><body><h1>It works!</h1></body></html>
```


## Připojení adresáře

Dále si zkusíme připojit nějaký vlastní adresář dovnitř souborového systému v kontejneru.
Zruš běžící `httpd` kontejner.

Nejdříve si vytvoř prázdný adresář a v něm soubor `index.html`.
```console
$ mkdir ~/container_htdocs
$ cd ~/container_htdocs
$ echo Ahoj! > index.html
```
A teď spusť kontejner s přepínačem `-v`:

```console
$ podman run -p 8080:80 -v ~/container_htdocs:/usr/local/apache2/htdocs/:Z --rm httpd
```


Když teď obnovíš stránku v prohlížeči, uvidíš `Ahoj!`:

```console
$ curl localhost:8080
Ahoj!
```

Jak to funguje?
Pusť si opět Bash, ale se zpřístupněným adresářem, a v něm `findmnt`:

```console
$ podman run -p 8080:80 -v ~/container_htdocs:/usr/local/apache2/htdocs/:Z -it --rm httpd bash
# findmnt
```

Zjistíš, že na cestě `/usr/local/apache2/htdocs` je *připojený* speciální
souborový systém, který dává kontejneru k dispozici adresář
`/home/petr/container_htdocs` z hostitelského systému.

Tenhle soubor můžeš dokonce změnit a změny se v hostitelském systému projeví.

```console
# echo ahoj ahoj>/usr/local/apache2/htdocs/index.html
```

> [warning]
> K nasdílenému adresáři má kontejner přístup i pro zápis.
> Na to si dávej pozor, pokud aplikaci v kontejneru nevěříš.

> [note]
> Kvůli bezpečnostnímu systému SELinux je potřeba použít za nasdílenou cestou
> `:Z`, což kontejneru zpřístupní soubory z domovského adresáře – standardně to
> totiž je z bezpečnostních důvodů zakázané.
> Nejde takto zpřístupnit *celý* domovský adresář – opět z bezpečnostních důvodů.
>
> Když místo `Z` na konci použiješ `O`, použije se dočasný *overlayfs*, takže
> kontejner sice může do daného adresáře zapisovat, ale všechny změny
> se po zastavení kontejneru zahodí.
> Takhle nasdílený adresář (základní vrstvu) ale neměň – to by mohlo mít
> nečekané následky.


## Dockerfile

Existují způsoby, jak udělat nový obraz kontejneru.
Asi nejpoužívanejší používá soubor Dockerfile (nebo neutrálnější `Containerfile`).

V *novém adresáři* si vytvoř si soubor `Dockerfile` (s velkým písmenem na začátku).
Do toho souboru zapíšeš jak nový obraz vyrobit.
Dockerfile má své vlastní příkazy – *začni s kontejnerem A*, *na tohle místo dej soubor B*, *doinstaluj balíček C*, atd.

Nový obraz bude založený na `httpd` a pro vytvoření se spustí jeden shellový příkaz:

```dockerfile
FROM httpd
RUN echo Ahoj > /usr/local/apache2/htdocs/index.html
```
Tím vznikne nový obraz disku, který bude až na jeden soubor stejný jako u `httpd`. 

Soubor ulož a vytvoř nový image příkazem `build` s několika argumenty:
 - výsledek se označí (`-t`, angl *tag*) jménem `mujhttpd`, 
 - tečka na konci příkazu znamená místo, ve kterém se nachází Dockerfile,
   který se má použít k vytvoření nového image.
   Vstup pro `podman build` totiž není jen `Dockerfile`, ale celý adresář,
   který může obsahovat další pomocné soubory.

Všechny příkazy z Dockerfile se postupně provedou
a výsledek se uloží jako nový obraz:

```console
$ podman build -t mujhttpd .
Sending build context to Docker daemon  3.072kB
Step 1/2 : FROM httpd
 ---> 2ae34abc2ed0
Step 2/2 : RUN echo Ahoj > /usr/local/apache2/htdocs/index.html
 ---> 47e1060cde54
Successfully built 47e1060cde54
Successfully tagged mujhttpd:latest
```

Když teď spustíš svůj kontejner stejně jako před chvíli oficiální `httpd`,
pomocí `curl` nebo prohlížeče si můžeš prohlédnout stránku s textem **Ahoj!**:

```console
$ podman run -p 8080:80 -d --rm mujhttpd
$ curl localhost:8080
Ahoj!
```

Takový hotový kontejner se dá pak nahrát třeba na Docker Hub.

### Build cache

`podman build` využívá *cache* – ukládá si mezivýsledky.
Po každém příkazu v `Dockerfile` se výsledný obraz uloží –
viz identifikační číslo za `--->` ve výstupu výše.
Když na konec Dokerfile přidáš nový příkaz nebo změníš něco uprostřed,
nezměněné příkazy ze začátku se nebudou provádět znovu.

Můžeš to zkusit tak, že spustíš `podman build` znovu, beze změn.
Ve výstupu se po přeskočených krocích objeví hláška `Using cache`:

```console
$ podman build -t mujhttpd .
STEP 1/2: FROM httpd
STEP 2/2: RUN echo Ahoj > /usr/local/apache2/htdocs/index.html
--> Using cache a72f2b49b848a35242d3b1258de163ea6bbb69a038090f13d76f59966890378c
COMMIT mujhttpd
--> a72f2b49b84
Successfully tagged localhost/mujhttpd:latest
a72f2b49b848a35242d3b1258de163ea6bbb69a038090f13d76f59966890378c
```

## Vyčištění systému

Kontejnery (a hlavně obrazy) dokážou zabrat docela dost místa na disku,
i když využívají sdílené vrstvy.
Když se budeš chtít všeho zbavit, napřed zjisti jaké kontejnery běží:

```console
$ podman ps
CONTAINER ID  IMAGE                           COMMAND           CREATED         STATUS             PORTS                 NAMES
0e753eec9815  docker.io/library/httpd:latest  httpd-foreground  32 minutes ago  Up 32 minutes ago                        happy_jennings
c33b1ebe3707  docker.io/library/httpd:latest  httpd-foreground  32 minutes ago  Up 32 minutes ago                        suspicious_cray
23649cdaef5c  docker.io/library/httpd:latest  httpd-foreground  32 minutes ago  Up 32 minutes ago                        romantic_allen
2649abd0dff0  docker.io/library/httpd:latest  httpd-foreground  28 minutes ago  Up 28 minutes ago  0.0.0.0:8080->80/tcp  ecstatic_mcclintock
```

Pak je všechny zastav:

```console
$ podman stop 0e753eec9815 c33b1ebe3707 23649cdaef5c 2649abd0dff0
2649abd0dff0
0e753eec9815
c33b1ebe3707
23649cdaef5c
```

A pak použij příkaz na smazání všech zastavených kontejnerů,
nepoužívaných obrazů a dalších dat:

```console
$ podman system prune --all
WARNING! This command removes:
	- all stopped containers
	- all networks not used by at least one container
	- all images without at least one container associated with them
	- all build cache

Are you sure you want to continue? [y/N] y
Deleted Containers
0e753eec9815e83d157f62e146a166e6e33f844c53cc22d924bc51ba3950d147
23649cdaef5cd1edd9159fad36274077312e38ad91969dc10f09f36e03bea722
337d48c6fdec05e3fc9e3e1a13da8c2ecc85447d88eb0e3f042c5e757560f07f
c33b1ebe370701b2c0d1df549442517dbd0e3aef857dda311450d68f5937a03c
Deleted Images
8362f2615893c70073d4b6576c884eade7f6ce81482780ac989d535b621750f9
ba6acccedd2923aee4c2acc6a23780b14ed4b8a5fa4e14e252a23b846df9b6c1
Total reclaimed space: 259.2MB
```

