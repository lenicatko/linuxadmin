# SSH - vzdálená administrace

Dnes se podíváme na to, jak spravovat servery přes síť.

Po troše nastavování to bude podobné jako většina toho, co jsme zatím dělali.
Jen bez grafických okýnek – připojíme se čistě přes příkazovou řádku.

> [note]
> Samozřejmě je možné sdílet obrazovku; nástroje na to existují i na Linuxu.
> Na serverech ale často nebývá grafické prostředí vůbec nainstalováno,
> protože vykreslování celkem zbytečně zabírá výkon.
> A každý software navíc taky zvyšuje šanci, že se na systému vyskytne
> bezpečnostní chyba.

Technicky to není složité: terminál je jen soubor, do kterého se zapisuje to,
co napíšeš na klávesnici a ze kterého lze číst výstup shellu a spouštěných
programů.
Dokonce i vychytávky jako barvičky nebo hrátky s kurzorem (které používá
editor `nano` nebo skrolovátko `less`) se dají posílat jako text.

Nejsložitější na celé věci je komunikaci *zabezpečit*.
Když se posílá všechno co napíšeš na klávesnici,
znamená to že „po drátech“ putuje i tvoje přihlašovací heslo.
Kdysi, když se administrátorům počítačových sítí dalo věřit (protože stejně
měli přístup ke všem počítačům ve firemní či univerzitní síti), se informace
posílaly „jen tak“.
Dnes, když informace putují internetem přes spoustu zařízení, z nichž každé
si může posílané informace přečíst, se ale musí šifrovat.
Díky efektivním nástrojům se šifruje všechno, co se pošle, nikoli jen hesla.

> [note]
> Nezabezpečené připojení se stále používají ke konfiguraci
> zařízení, které jsou k ovládacímu počítači připojeny přímo,
> ne přes síť/internet.
> Máš-li zkušenosti s MicroPythonem, vzpomeň jak probíhala komunikace
> s „destičkou“.

K zabezpečenému připojení použijeme protokol SSH (*secure shell*).
Podobně jako HTTP existuje server – služba `sshd`.
Ten je potřeba nastavit a spustit.
K počítači na kterém běží `sshd` je pak možné se vzdáleně připojit.

Na druhé straně budeme potřebovat *klienta*, program, který se k SSH serveru
umí připojit.
Na Linuxu a macOS existuje program `ssh`.
Pro Windows existuje grafický `puTTY`, který si můžeš nainstalovat
ze [stránek PuTTY](https://www.chiark.greenend.org.uk/~sgtatham/putty/latest.html).

> [note]
> Grafický klient PuTTY je k dispozici i pro ostatní systémy.
> Na Fedoře je v repozitářích, můžeš si ho nainstalovat přes
> `sudo dnf install putty`.


## Instalace OpenSSH

Na rozdíl od `httpd` se balíček s SSH serverem ve Fedoře nejmenuje `sshd`.
Historicky totiž byl pro HTTP k dispozici jen Apache,
ale pro SSH bylo na výběr z více několik možností.
Dnes se většinou používá OpenSSH, který si nainstaluj následovně:

```console
$ sudo dnf install openssh-server
```

Samotná služba už se ovšem jmenuje `sshd`:

```console
$ sudo systemctl start sshd
```

Možná budeš potřebovat i otevřít firewall.
Příkazu `firewall-cmd` stačí zadat jméno `ssh`:

```console
$ sudo firewall-cmd --add-service=ssh
```

Kdybys někdy pracoval{{a}} s jiným firewallem,
hodí se vědět že protokolu `ssh` přísluší port 22.


## Připojení

Teď se můžeš přihlásit!
Většinou se připojuje z jiného stroje, ale ze začátku se připojíme
z tvého virtuálního počítače zase na tvůj virtuální počítač.

Nebude to tedy zatím správa vzdáleného stroje, ale bude to jednodušší
– a bude to vypadat podobně jako s opravdovým vzdáleným strojem.

Příkaz `ssh` potřebuje dostat jméno poc'itače, ke kterému se připojuješ.
Můžeš zadat svou IP adresu (podobně jako u webového serveru), nebo
jméno `localhost`, které vždy označuje „tento počítač“:

```console
$ ssh localhost
The authenticity of host 'localhost (::1)' can't be established.
ECDSA key fingerprint is SHA256:dzya/6SYp058Apl4++46lt8IMPNTuIAznjUGlrI1ymU.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'localhost' (ECDSA) to the list of known hosts.
petr@localhost's password: 
Last login: Mon Dec  7 15:41:10 2020 from 192.168.122.1
[petr@localhost ~]$ 
```

Program `ssh` tě po připojení upozorní, že k tomuto počítači se ještě nikdy
nepřipojoval a zeptá se, jestli je to opravdu počítač ke kterému se chceš
připojit.
Kdyby ses totiž připojoval{{a}} přes internet, někdo „mezi“ tvým a cílovým
počítačem by se mohl docela jednoduše vydávat za server,
na který se chceš připojit.
Proto až „o něco půjde“, je dobré si ověřit že daný „otisk“ – v ukázce výše
`SHA256:dzya/6SYp058Apl4++46lt8IMPNTuIAznjUGlrI1ymU` – odpovídá počítači,
ke kterému se připojuješ.
Otisk by ti měl dát admistrátor počítače, ke kterému se připojuješ;
dá se zjistit pomocí:

```console
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
256 SHA256:0LNQwY8++rCDzynAAQx2MNP9OMupnpsps0loay8P3NM root@localhost.localdomain (ECDSA)
```

Jestli sedí, napiš `yes`.
Dále zadej heslo a jsi připojen{{a}}!

Klient `ssh` se standardně připojuje pod tvým uživatelským jménem.
Kdybys na vzdáleném počítači měla účet pojmenovaný jinak, nebo se chtěl{{a}}
přihlásit jako někdo jiný, odděl uživatelské jméno od jména serveru znakem `@`.
Máš-li stále účet `test1`, zkus to – ale ideálně v jiném terminálu,
ať můžeme pokračovat s opravdovým účtem:

```console
$ ssh test1@localhost
[test1@localhost ~]$ exit
$
```

Tentokrát se `ssh` na bezpečnostní ověření neptá.
Jakmile ses totiž k tomuto počtači připojil{{a}}, `ssh` si adresu `localhost`
spojil s daným klíčem.
Zaznamenal si to do souboru `~/.ssh/known_hosts`:

```console
$ cat ~/.ssh/known_hosts
localhost ecdsa-sha2-nistp256 AAAAE2VjZHNhLXNoYTItbmlzdHAyNTYAAAAIbmlzdHAyNTYAAABBBC9n7rIp76I0uxvicyZyRzUeh2PGElLwLV2Vw/AtrxOQkXWK/yfY/n5WlFiRblrOHf01tiH52C/zy1CiVLgl+bc=
```

> [note]
> Klíč v souboru je delší a bezpečnější verze „otisku“, na který se tě `ssh`
> ptal.
> Nebylo by ale praktické nutit člověka aby takhle dlouhý klíč porovnával.

Kdyby se klíč na serveru změnil, může to znamenat dvě věci:

* Administrátor heslo změnil, třeba kvůli bezpečnosti.
* Nepřipojuješ se k počítači, ke kterému ses připojoval{{a}} minule – a
  kdyby ses připojil{{a}}, pravděpodobně bys novému počítači jako první věc
  poslal{{a}} přihlašovací heslo.

První variantu si můžeš vyzkoušet příkazem.
Nebudu ho vysvětlovat do detailů; stačí vědět že vytvoří nový šifrovací klíč
pro SSHD:

```console
$ sudo ssh-keygen -f /etc/ssh/ssh_host_ecdsa_key -N '' -t ecdsa
```

Zkus se pak připojit:

```console
$ ssh localhost
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
@    WARNING: REMOTE HOST IDENTIFICATION HAS CHANGED!     @
@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@@
IT IS POSSIBLE THAT SOMEONE IS DOING SOMETHING NASTY!
...
Host key verification failed.
```

Když takovou zprávu uvidíš, ověř si že jde o první variantu výše – pokud od
administrátora např. nepřišel důvěryhodný mail, tak mu radši třeba zavolej.

Teď jsi ale administrátor ty a víš že je všechno v pořádku.
Smaž  tedy ze souboru `~/.ssh/known_hosts` starý neplatný záznam
a připoj se znovu.


## Vzdálené připojení – ssh

> [warning]
> Pokud se ti minule nepovedlo připojit k HTTP serveru,
> nejspíš nebude fungovat ani SSH.

A teď se připoj vzdáleně!
Máš-li na svém *opravdovém* počítači Linux nebo macOS, zjisti IP adresu
virtuálního počítače (příkazem `ip a`), na opravdovém počítači
si otevři příkazovou řádku a zadej <code>ssh <var>jmeno</var>@<var>adresa</var></code>:

```console
$ ssh petr@192.168.122.133
The authenticity of host '192.168.122.133 (192.168.122.133)' can't be established.
ECDSA key fingerprint is SHA256:0LNQwY8++rCDzynAAQx2MNP9OMupnpsps0loay8P3NM.
Are you sure you want to continue connecting (yes/no/[fingerprint])? 
```

Opět si ověř otisk klíče podle:

```console
$ ssh-keygen -l -f /etc/ssh/ssh_host_ecdsa_key.pub
256 SHA256:0LNQwY8++rCDzynAAQx2MNP9OMupnpsps0loay8P3NM root@localhost.localdomain (ECDSA)
```

A zkontroluj, že otisk sedí.
Pro plnou bezpečnost je potřeba zkontrolovat každé písmenko,
ne jen začátek nebo konec.

Je li vše OK, zadej `yes`, napiš svoje heslo – a máš vzdálený přístup
k virtuálnímu počítači!


## Vzdálené připojení – PuTTY

> [warning]
> Pokud se ti minule nepovedlo připojit k HTTP serveru,
> nejspíš nebude fungovat ani SSH.

Další SSH klient je grafický PuTTY.
Máš-li ho nainstalovaný, spusť ho a do políčka *Host Name (or IP address)*
napiš IP adresu virtuálního počítače.
*Port* nech na 22 a *Connection Type* na SSH.

{{ figure(
    img=static('putty-config.png'),
    alt='Diagram příkazu `cat`',
) }}

Pak klikni na *Open*, zadej své jméno a heslo – a máš vzdálený přístup
k virtuálnímu počítači!


## Jméno serveru

Podobně se připojit k jakémukoli jinému serveru, kam máš přístup.

Aby se servery nepletly, je dobré společně se službou SSHD nastavit i jméno
počítače.
To se dělá pomocí příkazu `hostname`, který bez argumentu jméno zobrazí
a s argumentem ho nastaví:

```console
$ hostname
localhost
$ sudo hostname virtualka
[sudo] heslo pro petr: 
$ hostname
virtualka
```

Programy většinou jméno počítače čtou jen při spuštění.
Když se teď *znovu přihlásíš* (přes SSH, nebo otevřeš nový terminál),
ve výzvě najdeš nově nastavené jméno.

Ale jako spousta jiného nastavení se tahle změna zruší když počítač vypneš
a zase zapneš.
Trvalé nastavení – tedy to, podle kterého se jméno nastavuje při startu
systému – je v souboru `/etc/hostname`.

```console
$ cat /etc/hostname
localhost.localdomain
[petr@localhost ~]$ echo virtualka | sudo tee /etc/hostname
virtualka
```


## SSH klíč

Zatím ses k virtuálnímu počítači přihlašoval{{a}} heslem.
To ovšem není nejbezpečnější a nejkomfortnější varianta.

Lepší je si vytvořit soubor s *klíčem*.
Vlastně to jsou soubory dva: *veřejný klíč*, který můžeš s kýmkoli sdílet,
a *soukromý klíč*, který je naopak jen tvůj – funguje jako heslo.

Svůj *veřejný klíč* dáš na server, ke kterému se budeš připojovat,
a nastavíš že mu server bude „věřit“.
Server pak bude věřit komukoli, kdo má příslušný soukromý klíč.

Klíče se tvoří příkazem `ssh-keygen`.
Na otázku, kam je chceš uložit, odpověz <kbd>Enter</kbd> – tím vybereš
výchozí nastavení: `/home/petr/.ssh/id_rsa` (soukromý)
a `/home/petr/.ssh/id_rsa.pub` (veřejný).
Na těchto místech pak klíče očekává `ssh`.

Pokud soubor už existuje, znamená to že klíč už máš.
Nevytvářej si nový; na otázku `Overwrite (y/n)?` odpověz `n`.

Další otázka je na heslo, kterým můžeš svůj soukromý klíč chránit.
Je tu pro situaci, kdy si nezamkneš počítač a někdo ti soubor s klíčem
ukradne.
Pokud máš nastavené heslo, samotný soubor s klíčem není k ničemu.
(Dokud ho zloděj neuhodne, což ale může automatizovat: podle složitosti hesla
takové uhodnutí hesla může trvat od sekund po desetiletí.)

```console
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/petr/.ssh/id_rsa): 
Created directory '/home/petr/.ssh'.
Enter passphrase (empty for no passphrase): 
Enter same passphrase again: 
Your identification has been saved in /home/petr/.ssh/id_rsa
Your public key has been saved in /home/petr/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:RihRspvWIrPti3X1x/HAYZ0N5CcqOTp1aaolCw7qZP4 petr@virtualka
```

Výsledný klíč pak musíš nakopírovat na server, kam se budeš přihlašovat.
Patří do adresáře `~/.ssh/authorized_keys`.
Můžeš ho tam vložit ručně, ale lepší je použít příkaz `ssh-copy-id`,
což je specializovaný SSH klient, který se přihlásí (pomocí hesla) na vzdálený
server a klíč tam přidá (pokud už tam není)
a správně tomu souboru nastaví práva:

```console
$ ssh-copy-id localhost
/usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
/usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
petr@localhost's password: 

Number of key(s) added: 1

Now try logging into the machine, with:   "ssh 'localhost'"
and check to make sure that only the key(s) you wanted were added.
```

Když teď zadáš `ssh localhost`, místo hesla k účtu zadat heslo ke klíči.
Jakou to má výhodu?
Když jednou klíč „odemkneš“, zůstane až do odhlášení odemčený
a nemusíš heslo zadávat znovu.
(O to se stará jiná služba, *keyring* – v našem případě GNOME keyring.)


## SSH klíč a GitHub

Protokol SSH se dá použít i na jiná šifrovaná spojení.
Pokud používáš službu GitHub,
můžeš si tam svůj privátní klíč přidat v nastavení
a pak ho používat na *push* a *pull*:

{{ figure(
    img=static('github-ssh-key.png'),
    alt='Nastavení SSH klíče na github.com',
) }}

Klikni na *New SSH key* a do políčka pro klíč zkopíruj obsah souboru
`~/.ssh/id_rsa.pub`.
Od teď můžeš klonovat repozitáře pomocí SSH místo HTTPS:

{{ figure(
    img=static('github-ssh-clone.png'),
    alt='Ukázání SSH adresy na github.com',
) }}

```console
git clone git@github.com:naucse/prezencka.git
```

Otisky klíčů serveru GitHub zveřejňuje [na svých stránkách](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints).

A u existujících repozitářů můžeš změnit remote („zkratku“) z HTTPS na SSH:

```console
git remote set-url origin git@github.com:naucse/prezencka.git
```


## Jméno serveru

Až se budeš připojovat na vzdálené počítače, je dobré vždycky vědět
kde právě „jsi“ – kterému počítači zadáváš příkazy.
K tomu se používá jméno počítaě, které můžeš jako superuživatel změnit:

```console
$ sudo hostname virtualka1
```

Tohle nastavení ale „nepřežije“ restart počítače.
Trvalé jméno je uloženo v souboru `/etc/hostname`, odkud se načte vždycky při
zapnutí počítače.
Načítá se příkazem, který můžeš použít i ty:

```console
$ sudo hostname -F /etc/hostname
```


## Nastavení SSHD

Nastavení `sshd` najdeš v souboru `/etc/ssh/sshd_config`
(a `/etc/ssh/sshd_config.d/`).

Pozor, na rozdíl od většiny konfiguračních souborů ho může číst jen
superuživatel. Použij tedy `sudo less`.

Některé servery pouští SSH na jiném portu než 22, aby se vyhnuly robotům
které se na tomle portu zkouší připojit pomocí běžných hesel.
Z bezpečnostního hlediska je to sporné (není velký problém vyzkoušet ostatní
porty), ale můžeš si zkusit tuhle změnu udělat. Port změň třeba na 2222.

Pár poznámek k tomu, jak na to:

* Bude potřeba nastavit SELinux – čti komnentáře v konfiguračním souboru.
* Po změně konfigurace bude potřeba službu restartovat (`systemctl reload`).
* Nový port bude potřeba zadat při spouštění klienta: `ssh localhost -p 2222`.
