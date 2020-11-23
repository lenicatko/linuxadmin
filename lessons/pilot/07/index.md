
# 7. Linuxová administrace - síť

Dnešní sraz předpokládá, že máš aspoň dva linuxové počítače k dispozici. Může to být tvůj operační systém a virtuální stroj, nebo dva virtuální stroje, pokud tvůj operační systém je Windows.

*Příklady níže jsou uvedeny pro kombinaci Ubuntu 18.04 (OS) - Fedora 32 (VM).*


## Sesíťování 
Pro připojení na jiný počítač potřebuješ, aby na tom vzdáleném počítači běžel **server**, na který se dá připojit.
Je to program, který poslouchá na internetu na určitém portu, a v případě, že se s ním někdo spojí, dá k dispozici vzdálenou příkazovou řádku. 
Na svém počítači potřebuješ mít **klienta**, čili program, který to spojení naváže a zprostředkuje.

Podíváme se na program, který nám umožní propojení dvou počítačů, a to [OpenSSH](https://www.openssh.com/). 
Měl{{a}} bys ho mít nainstalovaný na obou počítačích, které chceš propojit.

Do terminálů napiš příkaz `ssh --help`. Pokud na obou počítačích uvidíš výstup jako níže, máš vše připraveno a můžeš jít na další krok.

```console
$ ssh --help
unknown option -- -
usage: ssh [-46AaCfGgKkMNnqsTtVvXxYy] [-b bind_address] [-c cipher_spec]
           [-D [bind_address:]port] [-E log_file] [-e escape_char]
           [-F configfile] [-I pkcs11] [-i identity_file]
           [-J [user@]host[:port]] [-L address] [-l login_name] [-m mac_spec]
           [-O ctl_cmd] [-o option] [-p port] [-Q query_option] [-R address]
           [-S ctl_path] [-W host:port] [-w local_tun[:remote_tun]]
           [user@]hostname [command]
```

### Instalace OpenSSH - pokud ssh nebylo nalezeno

Některé linuxové distribuce jsou velmi minimalistické. 
Má to spoustu výhod: jsou to maličké systémy, a tak nepotřebují moc místa. 
Jsou konfigurovatelné, a tedy je můžeš nastavit pro jedno konkrétní použití, aniž bys "táhl{{a}}" s sebou plnou palbu grafických klikátek, programovacích jazyků, služeb atd. 
Tyto systémy, jako [Alpine Linux](https://www.alpinelinux.org/), ti zprostředkují terminál, a další potřebné služby si k nim musíš nainstalovat {{gnd('sám', 'sama') }}.

Pokud jsi tedy obdržel{{a}} výstup jako níže, další krok záleží na operačním systému, který máš.

```console
$ ssh --help
ssh not found
```

Potřebuješ nainstalovat balíček `openssh`.
Instalace probíhá např. takto:
Na Fedoře: `sudo dnf install <nazevprogramu>`
Na Debianu/Ubuntu: `sudo apt-get install <nazevprogramu>`
Na Alpine: `apk add <nazevprogramu>`

Pokud používáš jinou linuxovou distribuci, zjisti na internetu, jak se do ní instalují balíčky.

Program `apk`/ `dnf` / `apt-get` :
- podívá se do repozitáře balíčků
- zjistí, že je hledaný balíček k dispozici 
- stáhne archiv s balíčkem a rozbalí ho na disk
- podívá se, na čem všem tento balíček závisí, ty stáhne a nainstaluje taky


Podívej se, jestli balíček už máš k dispozici:
`$ ssh --help` 

Pokud vidíš nápovědu, je vše v pořádku.


### Služby, které běží pořád

A co dál, když to je už nainstalované?
1. Musíš nastartovat server
2. Říct, že se má spouštět při každém spuštění počítače

`sshd` je tzv. démon (anglicky *daemon*), což znamená, že je to speciální program, který běží na pozadí a běží pořád.

Je více způsobů, jak se o takové aplikace starat. 
Většinou například chceš, aby se takový program nahodil znova, když se z nějakého důvodu vypne. 
Existují programy, které přesně to pro tebe dělají. Nastartují, ukončují a jinak spravují běžící daemony.
Dnes na většině systémů je standardem program `systemd`, který je  správce procesů na tom počítači. 
Má své příkazy, které je dobré se naučit.
Můžeš mu říct, "tento webový server by měl vždy běžet, tak ho spusť".
Jedna z jeho nevýhod je, že je velký a dělá hromadu dalších věcí.
 
Na Fedoře spustíš daemon příkazy:
```
$ systemctl enable sshd   # umožni nastartování sshd pokaždé, když je spuštěn počítač
$ systemctl start sshd    # nastartuje daemon
$ systemctl status sshd   # ověří status služby
```

Tohle se spouští jen na serveru - počítači, ke kterému se chceš připojit. Tvůj počítač slouží jako klient, nepotřebuje tedy mít spuštěný daemon.

Protože `systemd` je velký, na Alpine Linuxu není. 
Příkazy pro Alpine vypadají takto:

```console
$ rc-update add sshd
$ /etc/init.d/sshd start
$ rc-status 
```

### Kde bydlí server?

K serveru se budeme chtít připojit a k tomu budeme potřebovat znát jeho adresu. 
Každý počítač má svou adresu na síti, což je obdoba poštovní adresy.
Zjisti adresu počítače, na který se budeš chtít připojit (ten s rozjetým `sshd`).

```console
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: wlp4s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether 34:02:86:19:c7:c9 brd ff:ff:ff:ff:ff:ff
    inet 192.168.88.51/24 brd 192.168.88.255 scope global dynamic noprefixroute wlp4s0
       valid_lft 391sec preferred_lft 391sec
    inet6 fe80::13cb:7e81:84f5:e263/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```

V tomto šíleném výpisu pod `wlp4s0`, což je Wi-Fi spojení (u tebe se může jmenovat jinak), se nachází kouzelné číslo `inet 192.168.88.51`. Pokud jsi připojen{{a}} k internetu kabelem, hledej položku začínající písmenem `e`, např. `enp0s3` nebo `eth0`.

Už znáš cílovou adresu, tak ji zkus "zavolat" ze svého počítače (klienta). 
`ping` je standardní nástroj pro vyzkoušení, jestli síťové spojení mezi dvěma počítači funguje. Je to jako ping-pong. Posílá se malý balík dat a čeká, jestli se vrátí z druhé strany.

```console
$ ping 192.168.88.51
```
(<kbd>CTRL</kbd>+<kbd>C</kbd>, pokud chceš zastavit)
Můžeš volat i jménem:

```console
$ ping github.com
```

#### Když nemáš IP adresu... (Alpine Linux)
V Alpine Linuxu ale síť není standardně nastavená.
Je to jako když si dáš na dům cedulku s adresou, to ještě neznamená, že ti tam bude i chodit pošta.
K tomu, aby se vědělo, že tuto adresu máš zrovna ty, slouží služba/server DHCP. Ten ti řekne "ty budeš 10.11.12.13".
Takže budeš potřebovat nastavit síť. 
Jak se to dělá? 
Založ si nový soubor `/etc/network/interfaces`.

Naučíš se používat editor `vi`, protože to je textový editor, který je na Alpine nainstalovaný.
Editor spustíš příkazem:
`$ vi /etc/network/interfaces`
* editor je přes celou obrazovku, nejsou tam žádná tlačítka, menu a další "rušivé" věci.
* zmáčknutím `i` se dostaneme do vkládacího režimu.

Do souboru napiš:
```
auto eth0
iface eth0 inet dhcp
```
* vi ukončíš zmáčknutím klávesy <kbd>ESC</kbd> (pro ukončení vkládacího režimu), následně napiš `:wq` a zmáčkni <kbd>ENTER</kbd>. 

Aby se změny projevily, je třeba znovu nastartovat síť, což provedeš pomocí příkazu
```console
$ /etc/init.d/networking restart
```

Teď si to můžeš vyzkoušet:
```console
$ ping github.com
```

## Linuxová administrace

A teď se konečně dostáváme k předmětu kurzu :).

Linuxová administrace je typicky o tom:
* upravit konfiguraci v textovém souboru
* restartovat službu 

...a máš vystaráno.

Na hlavním PC zadej `ssh <ip-adresa-serveru>`

Zeptá se tě to na heslo a snaží se připojit k tomu počítači jako ten samý uživatel, jako na tvém hlavním počítači.
To asi nebude ono, ten vzdálený počítač má jiné uživatele.

Potřebuješ uživatele, který bude pro tebe dostupný na tvém vzdáleném počítači (serveru). Pojďme ho vytvořit.
Na Fedoře:
```console
$ sudo useradd hanka
$ sudo passwd hanka
Changing password for user hanka.
New password:
Retype new password:
passwd: all authentication tokens updated successfully.
```

Teď zkus se ještě jednou připojit k serveru, tentokrát se jménem uživatele jako v příkladu níže:
```console
$ ssh hanka@<ip-adresa-serveru>
hanka@<ip-adresa-serveru> password: 
[hanka@localhost ~]$ 
```
Všimni si, že se změnil prompt.
Právě jsi připojen{{a}} k vzdálenému počítači a cokoliv do toho terminálu napíšeš, provede se vzdáleně.

```console
[hanka@localhost ~]$ echo ~
/home/hanka
[hanka@localhost ~]$ echo ~root
/root
```

Je dost důležité vědět, na kterém počítači pracuješ. 
Je dobré mít jiný prompt, protože pak je hezky vidět, na čem zrovna pracuješ.

***Celá tato část není dopracovaná. 
Mám zde problém s právy, protože uživatel vytvořen `useradd` nepatří do sudoers a neznám heslo svého roota. Chce to učesat pro jednotný postup.***


Prompt vzdáleného počítače můžeš změnit jako superuživatel.
```
# vi /etc/hostname
```
V souboru pak můžeš přepsat `localhost` na něco jiného, např. `test`.

Soubor ulož a zavři (`<Esc>:wq`). 
Jak vypadá prompt?
Pořád je tam původní `localhost`. Jak to? 
Spousta programů typicky funguje tak, že si při spuštění načtou konfiguraci a s tou pak pracují.
Musíme jim tedy explicitně říct: "teď si načti novou konfiguraci".

`# hostname -F /etc/hostname`

Můžeš taky nastavit hostname na jakýkoli řetězec, který se ti líbí:
`# hostname ahoj`

Pokud se teď odhlásíš a příhlásiš do vzdáleného počítače pomocí ssh, uvidíš tam nový prompt.

```console
[hanka@test ~]$ 
[hanka@ahoj ~]$ 
```

Nakonec ultimátní test moci nad vzdáleným počítačem: `poweroff`.
