# Poznámky – Mount

Na úvod: `realpath`


## Disk jako soubor

Minule jsme si řekli, disk obsahuje, podobně jako zazipovaný archiv,
nejen obsah souborů, ale i informace on nich (*inode*).
Vše na disku je uloženo jako sekvence bajtů.

Taky jsme si ukázali, že existují speciální soubory, které ovládají nebo
zpřístupňují různá zařízení.
Například existuje soubor, z nějž lze číst údaje o pohybu myši.

Stejně tak existuje speciální soubor, který reprezentuje *celý disk*.
Na ty samé informace se tedy můžeš “dívat” buď jako na jediný soubor,
který obsahuje informace o souborech a jejich obsahu (podobně jako
[zipovaný archiv](https://commons.wikimedia.org/wiki/File:ZIP-64_Internal_Layout.svg)),
nebo jako adresářovou strukturu s jednotlivými soubory, které můžeš číst zvlášť.

Kde je soubor který odpovídá celému disku, to ti řekne příkaz `findmnt`:

```console
$ findmnt /
TARGET SOURCE                                  FSTYPE OPTIONS
/      /dev/mapper/fedora_localhost--live-root ext4   rw,relatime,seclabel
```

Disk pro adresář `/` je přístupný v souboru `/dev/mapper/fedora_localhost--live-root`.
Používá formát `ext4` (ne `zip`) s následujícím nastavením:

* `rw` – umožňuje čtení i zápis
* `relatime` – čas posledního přístupu se neaktualizuje při každém přístupu
  (aby se disk neopotřebovával tak rychle)
* `seclabel` – soubory jsou označeny pro bezpečnostní systém SELinux

A co je ten soubor s celým diskem – v mém případě
`/dev/mapper/fedora_localhost--live-root`?

```console
$ ls -l /dev/mapper/fedora_localhost--live-root
lrwxrwxrwx. 1 root root 7 23. lis 12.58 /dev/mapper/fedora_localhost--live-root -> ../dm-0
$ realpath /dev/mapper/fedora_localhost--live-root
/dev/dm-0
$ ls -l /dev/dm-0
brw-rw----. 1 root disk 253, 0 23. lis 12.58 /dev/dm-0
```

Všimni si, že výpis z `ls -l` má tam, kde u nodmálních souborů najdeš `-`,
u adresářů `d` a odkazů `l`, uvedeno `b`.
Nejde tedy o normální soubor.
Víc informací poskytne `stat`:

```console
$ stat /dev/dm-0
  Soubor: /dev/dm-0
Velikost: 0         	Bloků: 0          I/O blok: 4096   blokové zařízení
Zařízení: 5h/5d	I-uzel: 20009       Odkazů: 1     Druh zařízení: fd,0
   Práva: (0660/brw-rw----)  UID: (    0/    root)   GID: (    6/    disk)
 Kontext: system_u:object_r:fixed_disk_device_t:s0
     Přístup: 2020-11-23 12:58:54.568866642 +0100
Změna obsahu: 2020-11-23 12:58:53.016000000 +0100
Změna i-uzlu: 2020-11-23 12:58:53.016000000 +0100
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
> $ stat -L /proc/self/fd/0 | head -n2
>   Soubor: /proc/self/fd/0
> Velikost: 0         	Bloků: 0          I/O blok: 1024   znakové zařízení
> ```
> Zapisovat větší soubory po jednotlivých znacích by bylo pomalé;
> na disky se proto zapisuje třeba 4MB dat najednou.
> A stejné je to pro čtení.

Obsah samotného disku není samozřejmě na disku uložen: v *inode* je jen řečeno
o který disk se jedná.
Místo velikosti souboru tak `ls -l` ukazuje dvě čísla, které disk identifikují.
(První, *major*, říká o jaký druh disku jde; druhé, *minor*, je číslo
disku daného druhu).

Stejně tak velikost disku není uložena v *inode*, a tak ji `stat` ani `ls -l`
neukážou.
Na velikost a množství volného a použitého místa existuje příkaz `df`:

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

Počet 1KB bloků není příliš přehledné; stejně tak tě často nebutou zajímat
informace o všech discích. Hodí se přepínač `-h` a možnost zadat konkrétní
disk:

```console
$ df -h /
Souborový systém                        Velikost Užito Volno Uži% Připojeno do
/dev/mapper/fedora_localhost--live-root      17G  7,1G  8,8G  45% /
```


## Kde je co připojeno

...

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

`FSTYPE` označuje souborový systém, tedy způsob, jak funguje – jak se
prohledávají addresáře, čtou soubory, zapisují informace, mění vlsatnictví,
atp.
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

Zkus si ve virtuálním počítači připojit/odpojit CD a podívej se,
co se stane s výstupem `findmnt`.
A taky kounkni na `/run/media/*`.
(Na opravdovém počítači bychom to dělali s flash diskem.)

Když má systém přistoupit k souboru jako `/run/media/petr/.../images/efiboot.img`,
podívá se do tabulky *přípojných bodů* (*mount points*) – to jsou cesty, které
ukazuje `findmnt` jako `TARGET`.
Podle toho pozná, na jakém je soubor zařízení a jakým způsobem
je má číst, zapisovat a kódovat (tj. jestli jde o `ext4`, `iso`, atd.).

`/run/media/petr/.../images/efiboot.img` tedy přísluší souboru `/images/efiboot.img`
na mém CD.


## Jaké mám disky

```
$ lsblk
NAME                     MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
sr0                       11:0    1  1,9G  0 rom  /run/media/petr/Fedora-WS-Live-32-1-6
vda                      252:0    0   20G  0 disk 
├─vda1                   252:1    0    1G  0 part /boot
└─vda2                   252:2    0   19G  0 part 
  ├─fedora_localhost--live-root
  │                      253:0    0   17G  0 lvm  /
  └─fedora_localhost--live-swap
                         253:1    0    2G  0 lvm  [SWAP]
```

- disk může rozdělen na části, které mohou mít různé filesystemy
- abstrakce nad sebou
- jakým způsobem číst ze souboru je možné udělat různými způsoby
    - vzdálený přístup k serveru - síťová komunikace (pro nás soubor)

- /boot - zavaděč operačního systému
    - bios se podívá na začátek pevnýho disku a načte instrukce
- swap - pokud nestačí paměť - zapisuje se do toho prostoru
    - odkládají se méně používané věci z paměti

```console
$ ls -l /dev/disk
drwxr-xr-x. 2 root root 400 Nov  7 09:41 by-id
drwxr-xr-x. 2 root root  80 Nov  7 09:34 by-partuuid
drwxr-xr-x. 2 root root 120 Nov  7 09:34 by-path
drwxr-xr-x. 2 root root 160 Nov  7 09:41 by-uuid

$ ls -l /dev/disk/by-id
...
```


- disk může rozdělen na části, které mohou mít různé filesystemy

## gparted

- umí modifikovat struktury na disku !
- práva na spuštění má pouze superuživatel - může dojít k nesprávnému použití a zrušení systému


## ISO a mount

Stáhni si obsah malého instalačního CD: „Virtual“ z [alpinelinux.org/downloads/](https://alpinelinux.org/downloads/):

```console
$ wget http://dl-cdn.alpinelinux.org/alpine/v3.12/releases/x86_64/alpine-virt-3.12.1-x86_64.iso
```

Jde o obsah disku, který je uložený v souboru:

```
$ file alpine-virt-3.12.1-x86_64.iso
alpine-virt-3.12.1-x86_64.iso: DOS/MBR boot sector; partition 2 : ID=0xef, start-CHS (0x3ff,254,63), end-CHS (0x3ff,254,63), startsector 284, 2880 sectors
```

Připoj ho. Pouze pro čtení (`-o ro`) a na `/mnt`:

```
$ sudo mount -o ro alpine-virt-3.12.1-x86_64.iso /mnt
```

V adresáři `/mnt` teď máš k dispozici soubory z tohoto virtuálního CD:

```
$ ls /mnt
apks  boot  efi
```

Pomocí `findmnt` můžeš zjistit další informace.

Při přístupu na soubor jako `/mnt/boot/vmlinuz-virt` tedy systém:

* Podle tabulky přípojných bodů zjistí, na kterém souborovém systému soubor je;
  zde `/mnt`.
* Zjistí jaký systém jde (`iso9660`) a o jaký jde disk (tady to není opravdový
  disk, ale soubor `~/alpine-virt-3.12.1-x86_64.iso`).
* Z `.iso` souboru přečte, kde je v něm uložen soubor `/boot/vmlinuz-virt`,
  a dá k dispozici jeho obsah.
  Aby mohl tenhle soubor přečíst, tak:
  * Podle tabulky přípojných bodů zjistí, na kterém souborovém systému soubor
    `/home/petr//alpine-virt-3.12.1-x86_64.iso` je; zde `/`.
  * Zjistí jaký systém jde (`ext4`) a o jaký jde disk (`/dev/dm-0`).
  * Z `/dev/dm-0` souboru přečte inode pro soubor
    `/home/petr/alpine-virt-3.12.1-x86_64.iso`,
    a dá k dispozici jeho obsah.
    Soubor `/dev/dm-0` odpovídá disku, takže čte přímo z příslušného zařízení.

> [note]
> Navíc máš virtuální počítač. To co tento počítač vnímá jako pevný disk
> je ve skutečnosti jen soubor někde na tvém opravdovém počítači.
> (Pro Gnome Boxes je v `~/.local/share/gnome-boxes/images`.)
> Ve skutečnosti tedy informace procestují ještě několik podobných úrovní,
> než doputují z/na opravdový disk.

XXX Šifrování/komprese?

Disk můžeš zase odpojit:

```console
$ sudo umount /mnt
$ ls /mnt
```


## Skrytí souboru

```console
$ sudo touch /mnt/zkouska
$ ls /mnt
zkouska
$ sudo mount -o ro alpine-virt-3.12.1-x86_64.iso /mnt
$ ls /mnt
apks  boot  efi
```

Soubor `zkouska` teď není přístupný – na `/mnt` byl připojen jiný disk.
Při přístupu na `/mnt/zkouska` se systém vůbec nedívá na souborový systém `/`.

```console
$ sudo umount /mnt
$ ls /mnt
zkouska
$ sudo rm /mnt/zkouska
```


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


## /etc/fstab

XXX

## Šifrované disky

XXX
