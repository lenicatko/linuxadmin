# Instalace balíčků

```console
$ sudo dnf remove vorbis-tools
$ ogg123 /usr/share/sounds/gnome/default/alerts/bark.ogg
bash: /usr/bin/ogg123: Adresář nebo soubor neexistuje
```

Instalace by spustila pomocný *setuid* program, který umí *pouze* instalovat
nový software, ale ne aktualizovat/měnit/odstraňovat existující.
Neovlivňuje tak ostatní uživatele, tak jako to dělá `dnf remove`.

Příkaz `dnf` ale umí všechno, co se týká správy instalovaného software.
Třeba instalovat nové věci:

```console
$ sl
bash: sl: Příkaz nebyl nenalezen...
Podobný příkaz je: 'ls'
$ sudo dnf install sl
$ sl
```

Pomocí `dnf` můžeš instalovat jak grafické aplikace (na které funguje
i grafický nástroj `gnome-software`) tak i programy pro příkazovou řádku
nebo systémové knihovny.
Taky se můžeš např. podívat jaký  software máš nainstalovaný
(což jde i bez `sudo`):

```
$ sudo dnf list installed
```

Taky můžeš systém aktualizovat:

```console
$ sudo dnf update
```


Programy pro instalaci software jsou jedna z věcí, které se velice často liší
mezi různými distribucemi Linuxu.
Fedora má `dnf`, ale Debian a Ubuntu mají `apt-get`, Arch `pacman`
(*pac*kage *man*ager) atd. Každý se používá trošku jinak.
