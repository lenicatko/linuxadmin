# Mount


## Disk jako soubor

Minule jsme si řekli, disk obsahuje, podobně jako zazipovaný archiv,
nejen obsah souborů ale i informace on nich (*inode*).

Obsah souboru je dlouhá sekvence nul a jedniček (kterou často lze dekódovat
jako text, obrázek nebo třeba písničku).
Obsahy všech souborů a informace o nich jsou pak na disku uložené jako
jako ještě delší sekvence nul a jedniček.

Taky víš, že existují speciální soubory které ovládají nebo
zpřístupňují různá zařízení.
Například jeden ze souborů `/dev/input/mouse*` zpřístupňuje myš:
superuživatel z tohoto souboru může číst údaje o pohybu myši.

Stejně tak existuje speciální soubor, který reprezentuje *celý disk*.
Na ty samé informace se tedy můžeš “dívat” buď jako na
adresářovou strukturu s jednotlivými soubory, které můžeš číst zvlášť,
nebo jako na jediný dlouhý soubor, který obsahuje informace o souborech
a jejich obsahu (podobně jako zazipovaný archiv).

Kde je soubor který odpovídá celému disku, to ti řekne příkaz `findmnt`:

```console
$ findmnt /
TARGET SOURCE           FSTYPE OPTIONS
/      /dev/vda2[/root] btrfs  rw,relatime,seclabel,compress=zstd:1,space_cache,
```

Disk pro adresář `/` je přístupný v souboru `/dev/vda2`
(resp. jeho podčásti `root`).

Další sloupce popisují přesný způsob,jak ty ty jedničky a nuly,
které jsou uloženy na disku, interpretovat jako soubory a adresáře.

Ze sloupce *FS Type* (*filesystem type*, druh souborového systému)
je vidět, že se používá formát `btrfs` (a ne třeba `zip`).
Ve sloupci *options* je pak detailní nastavení:

* `rw` – umožňuje čtení i zápis
* `relatime` – čas posledního přístupu se neaktualizuje při každém přístupu
  (aby se disk neopotřebovával tak rychle)
* `seclabel` – soubory jsou označeny pro bezpečnostní systém SELinux
* `compress=zstd:1` – soubory jsou zkomprimované, aby zabíraly míň místa
* `space_cache` – volné místo na disku je označeno, aby se rychleji hledalo

Tohle je příklad – u tvého disku tohle nastavení možná vypadá jinak.

A co je ten soubor s celým diskem – v mém případě
`/dev/vda2`?

```console
$ ls -l /dev/vda2
brw-rw----. 1 root disk 252, 0 15. lis 14.06 /dev/vda2
```

Všimni si, že výpis z `ls -l` má tam, kde u nodmálních souborů najdeš `-`,
u adresářů `d` a odkazů `l`, uvedeno `b`.
Nejde tedy o normální soubor.
Víc informací poskytne `stat`:

```console
$ stat /dev/vda2
  Soubor: /dev/vda2
Velikost: 0         	Bloků: 0          I/O blok: 4096   blokové zařízení
Zařízení: 5h/5d	I-uzel: 256         Odkazů: 1     Druh zařízení: fc,2
   Práva: (0660/brw-rw----)  UID: (    0/    root)   GID: (    6/    disk)
 Kontext: system_u:object_r:fixed_disk_device_t:s0
     Přístup: 2021-11-15 14:06:35.687000000 +0100
Změna obsahu: 2021-11-15 14:06:35.687000000 +0100
Změna i-uzlu: 2021-11-15 14:06:35.687000000 +0100
       Vznik: -
```

Jde o *blokové zařízení* (angl. *block device*): místo, kam se dají ukládat
informace ve větších blocích.

> [note]
> Kromě blokových existují i znaková zařízení (angl. *character device*),
> kam se zapisuje po jednotlivých znacích.
> Jsou to třeba terminály:
>
> ```console
> $ stat /dev/tty0
>   Soubor: /dev/tty0
> Velikost: 0         	Bloků: 0          I/O blok: 4096   znakové zařízení
> Zařízení: 5h/5d	I-uzel: 13          Odkazů: 1     Druh zařízení: 4,0
> [...]
> ```
> Zapisovat větší soubory po jednotlivých znacích by bylo pomalé;
> na disky se proto zapisuje třeba 4MB dat najednou.
> A stejné je to pro čtení.

V *inode* pro zařízení není řečeno, kde začíná obsah souboru a jak je dlouhý.
Místo velikosti souboru tak `ls -l` ukazuje dvě čísla, které disk identifikují.
(První, *major*, říká o jaký druh disku jde; druhé, *minor*, je pořadové číslo
disku daného druhu).

Protože velikost disku není uložena v *inode*, `stat` ani `ls -l` ji neukážou.
Velikost disků a množství volného a použitého místa ti ukáže příkaz `df`:

```console
$ df
Souborový systém                        1K bloků   Užito   Volné Uži% Připojeno do
devtmpfs                                  993372       0  993372   0% /dev
tmpfs                                    1012980       0 1012980   0% /dev/shm
tmpfs                                    1012980    1444 1011536   1% /run
/dev/mapper/fedora_localhost--live-root 17410832 7365588 9137776  45% /
tmpfs                                    1012980      60 1012920   1% /tmp
/dev/vda1                                 999320  277428  653080  30% /boot
tmpfs                                     202596      76  202520   1% /run/user/1000
```

Počet 1KB bloků není příliš přehledný; stejně tak tě často nebutou zajímat
informace o všech discích.
Hodí se proto možnost zadat konkrétní disk a přepínač `-h`,
který velikosti zaokrouhlí a přidá jednotky:

```console
$ df -h /
Souborový systém                        Velikost Užito Volno Uži% Připojeno do
/dev/mapper/fedora_localhost--live-root      17G  7,1G  8,8G  45% /
```


## Kde je co připojeno

Jak ukazuje výstup příkazu `df`, na počítači můžeš mít více „disků“ –
nebo obecněji, víc *souborových systémů*.
Ne všechny souborové systémy jsou totiž uložené na disku – některé můžou
existovat jen v paměti počítače, jiné třeba obsahují soubory
z univerzitního/firemního serveru, které se podle potřeby načítají přes síť.

Všechny ti je ukáže příkaz `findmnt`:

```console
$ findmnt
TARGET                       SOURCE     FSTYPE   OPTIONS
/                            /dev/mapper/fedora_localhost--live-root
│                                       ext4     rw,relatime,seclabel
├─/sys                       sysfs      sysfs    rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/security     securityfs security rw,nosuid,nodev,noexec,relatime
│ ├─/sys/fs/cgroup           cgroup2    cgroup2  rw,nosuid,nodev,noexec,relatime,seclabel,nsdelega
│ ├─/sys/fs/pstore           pstore     pstore   rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/bpf              none       bpf      rw,nosuid,nodev,noexec,relatime,mode=700
│ ├─/sys/fs/selinux          selinuxfs  selinuxf rw,relatime
│ ├─/sys/kernel/debug        debugfs    debugfs  rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/kernel/tracing      tracefs    tracefs  rw,nosuid,nodev,noexec,relatime,seclabel
│ ├─/sys/fs/fuse/connections fusectl    fusectl  rw,nosuid,nodev,noexec,relatime
│ └─/sys/kernel/config       configfs   configfs rw,nosuid,nodev,noexec,relatime
├─/proc                      proc       proc     rw,nosuid,nodev,noexec,relatime
│ └─/proc/sys/fs/binfmt_misc systemd-1  autofs   rw,relatime,fd=30,pgrp=1,timeout=0,minproto=5,max
├─/dev                       devtmpfs   devtmpfs rw,nosuid,seclabel,size=993372k,nr_inodes=248343,
│ ├─/dev/shm                 tmpfs      tmpfs    rw,nosuid,nodev,seclabel
│ ├─/dev/pts                 devpts     devpts   rw,nosuid,noexec,relatime,seclabel,gid=5,mode=620
│ ├─/dev/hugepages           hugetlbfs  hugetlbf rw,relatime,seclabel,pagesize=2M
│ └─/dev/mqueue              mqueue     mqueue   rw,nosuid,nodev,noexec,relatime,seclabel
├─/run                       tmpfs      tmpfs    rw,nosuid,nodev,seclabel,mode=755
│ └─/run/user/1000           tmpfs      tmpfs    rw,nosuid,nodev,relatime,seclabel,size=202596k,mo
│   └─/run/user/1000/gvfs    gvfsd-fuse fuse.gvf rw,nosuid,nodev,relatime,user_id=1000,group_id=10
├─/tmp                       tmpfs      tmpfs    rw,nosuid,nodev,seclabel
├─/boot                      /dev/vda1  ext4     rw,relatime,seclabel
└─/var/lib/nfs/rpc_pipefs    sunrpc     rpc_pipe rw,relatime
```

Jako u `findmnt /` označuje `FSTYPE` druh souborového systému.
To je způsob, jak funguje – jak se prohledávají addresáře, čtou soubory,
zapisují informace, mění vlsatnictví, atp.
Časté systémy jsou:

* `ext4`, `ext3`, `btrfs` – souborové systémy používané v Linuxu
* `ntfs`, `apfs` – souborové systémy pro Windows, resp. macOS
* `fat32`, `fat64` – souborové systémy pro DOS, používané na Flash discích
* `iso9660` – souborový systém používaný pro CD a DVD: navržený pouze pro čtení,
  v čemž je dost podobný ZIP archivům
* `tmpfs` – disk v paměti (po vypnutí počítače se smaže)
* `sysfs` – speciální souborový systém, který zprostředkovává informace o systému
* `devtmpfs` – speciální souborový systém, který zprostředkovává informace o zařízeních

> [note]
> „Flashky“ často používají souborové systémy jako FAT, který lze bez problémů
> přečíst na všech systémech.
> Takové systémy nepodporují Linuxové vychytávky: znaky jako `*`, `\`, `"`
> ve jménech souborů, symbolické a tvrdé odkazy, rozlišování velikosti písmen
> a podobně.

Zkus si ve virtuálním počítači připojit/odpojit instalační DVD a podívej se,
co se stane s výstupem `findmnt`.
(Na opravdovém počítači bychom to dělali s flash diskem.)
Přibude tam něco jako:

```
├─/run                       tmpfs      tmpfs    rw,nosuid,nodev,seclabel,size=402224k,nr_inodes=
│ ├─/run/media/petr/Fedora-WS-Live-34-1-2
│ │                          /dev/sr0   iso9660  ro,nosuid,nodev,relatime,nojoliet,check=s,map=n,
```

To znamená, že soubory z adresáře `/run/media/petr/Fedora-WS-Live-34-1-2`
se čtou z instalačního DVD – konkrétně z blokového zařízení `/dev/sr0`,
které reprezentuje virtuální DVD mechaniku.

Když má systém přistoupit k souboru jako `/run/media/petr/Fedora-WS-Live-34-1-2/images/efiboot.img`,
podívá se do tabulky *přípojných bodů* (*mount points*) – do tabulky,
kterou `findmnt` ukazuje.
Podle toho systém pozná, na jakém je soubor zařízení a jakým způsobem
je má číst, zapisovat a kódovat (tj. jestli jde o `ext4`, `iso`, atd.).

`/run/media/petr/Fedora-WS-Live-34-1-2/images/efiboot.img` tedy přísluší souboru
`/images/efiboot.img` na instalačním DVD.


## Jaké mám disky

Všechny disky (bloková zařízení) na systému ukazuje program `lsblk`:

```console
$ lsblk
NAME   MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0     11:0    1  1,9G  0 rom  /run/media/petr/Fedora-WS-Live-34-1-2
zram0  251:0    0  1,9G  0 disk [SWAP]
vda    252:0    0   20G  0 disk 
├─vda1 252:1    0    1G  0 part /boot
└─vda2 252:2    0   19G  0 part /home
```

V prvním sloupci je jméno disku, tak jak ho najdeš v adresáři `/dev/`.
Následuje číslo disku (major:minor), označení zda lze disk vyměnit
(jako CD/DVD/Flashku), velikost, čtení, druh, a nakonec adresář kam
je disk připojen (resp. jeden z takových adresářů).

Zde je `sr0` instalační DVD a `zram0` *swap* – odkládací místo,
kam systém zapíše méně používaný obsah paměťi,
když dochází „opravdová” paměť počítače .

Nejzajímavější je ale `vda`.
To je zkratka z „virtuální disk A“; na opravdovém počítači uvidíš třeba `sda`
nebo `nvme0n1` (podle technologie jakou je disk připojen – SCSI nebo NVME).

Disk (např. `vda`) může rozdělen na oddíly (např. `vda1`, `vda2`).
Každý oddíl může obsahovat jiný souborový systém.
Můžeš tak např. na disku mít oddíl pro Linux (s `btrfs`),
jiný oddíl pro Windows (s `ntfs`),
a datový oddíl kterému rozumí oba systémy (třeba s `fat64`).

Takové věci nastavují při instalaci systému, ale dají se měnit i potom,
třeba programem `gparted`.
Pozor na to, že manipulací s oddíly disku se dá celkem
jednoduše zničit celý systém.


## Připojování

Stáhni si obsah malého instalačního CD: „Virtual“ z [alpinelinux.org/downloads/](https://alpinelinux.org/downloads/).
Můžeš k tomu použít program `wget`:

```console
$ wget https://dl-cdn.alpinelinux.org/alpine/v3.14/releases/x86_64/alpine-virt-3.14.3-x86_64.iso -O alpine.iso
```

> [note]
> Kopíruješ-li na srazu z projektoru, použij tuhle zkratku:
>
> ```console
> $ wget https://tinyurl.com/alpine-3-14 -O alpine.iso
> ```

Jde o obsah disku, který je uložený v souboru – podobně jako instalační DVD,
které jsi stahoval{{a}} pro teto kurz.
Obsahuje Linuxovou distribuci Alpine, která je menší než je Fedora – neumí toho
tolik, zato se rychleji stahuje a nezabere tolik místa.

{# XXX
```
$ file alpine.iso
alpine.iso: DOS/MBR boot sector; partition 2 : ID=0xef, start-CHS (0x3ff,254,63), end-CHS (0x3ff,254,63), startsector 284, 2880 sectors
```
#}

Pomocí programu `mount` pak tento disk *připoj*.
Přepínačem `-o` vyber připojení pouze pro čtení (`ro`);
dvěma argumenty řekni který soubor připojit (ten stažený)
a kam ho připojit (`/mnt`):

```
$ sudo mount -o ro alpine.iso /mnt
```

V adresáři `/mnt` teď budeš mít k dispozici soubory ze staženého CD:

```
$ ls /mnt
apks  boot  efi
```

Pomocí `findmnt` můžeš zjistit další informace.

Při přístupu na soubor jako `/mnt/boot/vmlinuz-virt` teď systém:

* Podle tabulky přípojných bodů zjistí, na kterém souborovém systému soubor je;
  zde `/mnt`.
* Zjistí jaký systém jde (`iso9660`) a o jaký jde disk (tady to není opravdový
  disk, ale soubor `~/alpine.iso`).
* Struktura souboru `alpine.iso` je podobná jako `zip`.
  Uvnitř je nějaký ISO9660 adresář,
  ze kterého systém přečte kde v tomhle archivu začíná
  soubor `/boot/vmlinuz-virt` a jak je dlouhý, a dá k dispozici jeho obsah.
  Aby ale mohl tenhle soubor přečíst, tak:
  * Podle tabulky přípojných bodů zjistí, na kterém souborovém systému soubor
    `~/alpine.iso` je; zde `/`.
  * Zjistí jaký systém jde (`btrfs`) a o jaký jde disk (`/dev/vda0`).
  * Z `/dev/vda0` souboru přečte inode pro soubor
    `~/alpine.iso`, a dá k dispozici jeho obsah.
    Soubor `/dev/dm-0` odpovídá disku, takže čte přímo z příslušného zařízení.

Navíc máš virtuální počítač. To co tento počítač vnímá jako pevný disk
je ve skutečnosti jen soubor někde na tvém opravdovém počítači.
(Pro Gnome Boxes je v `~/.local/share/gnome-boxes/images`.)
Ve skutečnosti tedy informace procestují ještě několik podobných úrovní,
než doputují z/na opravdový disk.


## Odpojování

Připojený disk můžeš odpojit pomocí příkazu `umount`:

```console
$ sudo umount /mnt
$ ls /mnt
```

Když zkusíš odpojit používaný souborový systém (např. `/` nebo `/home`),
zjistíš, že tohle ti systém nedovolí.
Tohle bývá občas problém při odpojování „flashky“, když je z ní v nějakém
programu otevřený soubor.

Příkaz `umount` může trvat delší dobu.
Systém totiž data nezapisuje rovnou na disk,
ale nějakou dobu si je drží v paměti.
Může tak zapisovat větší bloky najednou, což je efektivnější;
nebo když je disk pomalý, může zapisující aplikaci nahlásit že je vše zapsáno,
ale ve skutečnosti na disk zapisovat ještě pár minut.
To je důvod, proč se doporučuje „flasku“ odpojit v systému (ať už `umount`
nebo v grafickém programu) a až potom ji vysunout fyzicky.

Když náhle odpojíš počítač/disk/flashku bez „softwarového“ odpojení/ukončení,
je možné že se souborový systém poškodí.
Starších souborové systémy, jako `fat32` na flashkách,
jsou na tohle zvlášť náchylné.
Je možné souborový systém zkontrolovat a zkusit opravit pomocí `fsck`;
detaily najdeš v manuálové stránce toho programu.

Záznam o tom, co systém dělá, ti vypíše příkaz `dmesg`.
Často tu najdeš např. jméno právě připojeného zařízení.


## Subvolume

Někdy se na jednom souborovém systému nachází víc kořenových adresářů.
V našich příkladech je `/home` na disku `/dev/vda2[/home]`
a `/` na disku `/dev/vda2[/root]`.
Téhle vychytávce se říká *subvolumes* (viz např. `man btrfs-subvolume`).
Kdybys *subvolume* chtěl{{a}} připojit, musíš příkazu `mount` předat
volbu jako `-o subvolume=/home`.


## Skrytí souboru

Disk musíš vždy připojit na existující adresář – v příkladu výše to byl `/mnt`,
který je na připojování disků vyhrazený.

Když tenhle adresář něco obsahuje, jeho obsah se skryje. Zkus si to:

```console
$ sudo touch /mnt/zkouska
$ ls /mnt
zkouska
$ sudo mount -o ro alpine.iso /mnt
$ ls /mnt
apks  boot  efi
```

Soubor `zkouska` teď není přístupný.
Při přístupu na `/mnt/zkouska` se systém vůbec nedívá na souborový systém `/`.

Když ale disk odpojíš, bude `zkouska` zase vidět:

```console
$ sudo umount /mnt
$ ls /mnt
zkouska
$ sudo rm /mnt/zkouska
```


### Další speciální zařízení

Blokové zařízení umí číst nebo zapisovat kusy informací – většinou obsah
souborů nebo informace o nich (*inode*).

Ne všecha bloková zařízení ale reprezentují *disk*.
Pojďme se podívat na několik příkladů jiných druhů.


#### Šifrování disku

Šifrování disku se řeší podobně. Například:

* Zařízení `/dev/mapper/luks-...` je připojeno se souborový systémem `btrfs`.
  Systém ho tak dává k dispozici jako hierarchii souborů a adresářů.
  Data zapsaná na tohle zařízení ale nejdou přímo na disk:
  systém je zašifruje a výsledek zapíše do zařízení `/dev/sda1`.
  Podobně funguje čtení: blok z `/dev/mapper/luks-...` se čte tak,
  že se přečte z `/dev/sda1` a dešifruje.
* Zařízení `/dev/sda1` pak je oddíl na opravdovém disku. 
  Klidně to ale může být disk virtuální nebo normální soubor.


#### LLVM a RAID

Můžeš mít několik fyzických disků spojených dohromady jako jeden
souborový systém.
Tímto se jednoduše zvětšuje nebo zmenšuje kapacita disku.
Aby fungovaly jako jeden souborový systém, je nad nimi zařízení,
která se tváří jako jediný disk.
Podle toho na které místo zapisuješ nebo čteš pak systém ví
na který konkrétní disk se obrátit.

Případně můžeš veškerý obsah disku duplikovat: když systém zapisuje data,
zapíše je na dva nebo víc opravdových disků.
Když se pak jeden z disků poškodí, jednoduše ho vyměníš.
Na nový disk se pak nakopírují data z těch ostatních.

K takové organizaci disků a duplikování dat se nejčastěji používají techologie
[LVM](https://en.wikipedia.org/wiki/Logical_Volume_Manager_%28Linux%29), resp.
[RAID](https://en.wikipedia.org/wiki/RAID).
Jestli se chystáš pracovat jako systémový administrátor,
měl{{a}} by ses s nimi rozhodně časem seznámit.

### Souborové systémy bez disku

Souborový systém je způsob, jak data na disku (nebo jiném zařízení)
interpretovat jako soubory a adresáře – jak číst, zapisovat, procházet adresář,
měnit práva a podobně.
Souborový systém ale nemusí nutně pracovat s diskem.

Když se systému zeptáš na obsah adresáře `/proc`, nedívá se na disk,
ale odpoví podle toho, jaké procesy zrovna na počítači běží.
Podobně funguje adresář `/proc/self/fd`, který obsahuje otevřené soubory
aktuálního procesu:

```console
$ ls -l /proc/self/fd < /dev/null
celkem 0
lr-x------. 1 petr petr 64 15. lis 16.02 0 -> /dev/null
lrwx------. 1 petr petr 64 15. lis 16.02 1 -> /dev/pts/0
lrwx------. 1 petr petr 64 15. lis 16.02 2 -> /dev/pts/0
lr-x------. 1 petr petr 64 15. lis 16.02 3 -> /var/lib/sss/mc/passwd
lr-x------. 1 petr petr 64 15. lis 16.02 4 -> /var/lib/sss/mc/group
lr-x------. 1 petr petr 64 15. lis 16.02 5 -> /proc/6500/fd
```

Příkaz `findmnt /proc` ti prozradí, že souborový systém není `btrfs` nebo
`iso9660`, ale `proc`.
Všechny operace (čtení psaní, procházení adresářů) jsou specifické pro tenhle
informační adresář.

Podobných souborových systémů, které zprostředkovávají informace o běžícím
systému, je celkem hodně – `procfs` na `/proc`, `sysfs` na `/sys`,
`devfs` na `/dev`, a tak dál.

Trochu jinak funguje souborový systém je `tmpfs`, který bývá připojený na
`/tmp` – místo pro dočasné soubory.
Ten obsahuje opravdové soubory a adresáře, ale místo na disk je ukládá
do paměti počítače.
Po vypnutí stroje tak obsah zmizí.

Existují i souborové systémy sdílené mezi několika počítači: všechny operace
(čtení, psaní, změny práv, atd.) se posílají po síti na server a odpovědi
systém reprezentuje jako soubory a adresáře.


## findmnt a mount

Z historických důvodů dává příkaz `mount` dává ty samé informace jako `findmnt`:

```console
$ mount
sysfs on /sys type sysfs (rw,nosuid,nodev,noexec,relatime,seclabel)
proc on /proc type proc (rw,nosuid,nodev,noexec,relatime)
devtmpfs on /dev type devtmpfs (rw,nosuid,seclabel,size=993372k,nr_inodes=248343,mode=755)
securityfs on /sys/kernel/security type securityfs (rw,nosuid,nodev,noexec,relatime)
...
```

Toto použití najdeš ve spoustě existujících návodů,
podle manuálových stránek `mount` je ale `findmnt` lepší varianta.

Pro skripty a jiné programové zpracování se hodí přepínače `-l` a `-n`:

```console
$ findmnt -ln
TARGET                   SOURCE     FSTYPE   OPTIONS
/sys                     sysfs      sysfs    rw,nosuid,nodev,noexec,relatime,seclabel
/proc                    proc       proc     rw,nosuid,nodev,noexec,relatime
/dev                     devtmpfs   devtmpfs rw,nosuid,seclabel,size=993372k,nr_inodes=248343,mode
/sys/kernel/security     securityfs security rw,nosuid,nodev,noexec,relatime
```

Nebo `-J` pro JSON.


## `/etc/fstab`

Seznam zařízení která se připojí při startu systému najdeš
v souboru `/etc/fstab`.
Ten se vytváří se při instalaci, ale můžeš ho měnit i ručně.
Když třeba nastavovat počítačovou laboratoř na univerzitě a budeš chtít
připojit domovské adresáře studentů z univerzitního serveru,
popis připojení přidáš právě do tohoto souboru.

V prvním sloupci tu pravděpodobně najdeš disky označené podle `UUID`
(identifikátoru), ale můžeš použít i cestu k blokovému zařízení.
Ostatní sloupce jsi už většinou viděl{{a}} jinde; co přesně znamenají
ti řekne `man fstab`.

Zkus si připojit stažený obraz CD.
Soubor `/etc/fstab` otevři v editoruu (jako `root`) a na konec připiš:

```console
/home/<tvoje_jmeno>/alpine.iso /mnt iso9660 ro
```

Příkaz `mount -a` teď tenhle disk připojí.
A to stejné se stane i po restartu systému.


{#

## Vychytávky

Pokud tě zajímá, jak doopravdy funguje souborový systém, existuje knihovna
`python-fuse` (file system in user space), s jejiž pomoci si můžeš napsat
Pythonní třídu, ve které nadefinuješ souborové operace,
a vzniklý souborový systém si připojit do systému.

#}
