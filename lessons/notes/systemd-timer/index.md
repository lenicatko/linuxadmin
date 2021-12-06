> [warning]
> Tohle jsou jen velice strohé poznámky, ne materiály k samostudiu.

# Systemd jednotky

Opakování

* `systemctl status sshd`
* `systemctl stop sshd`
* `systemctl start sshd`
* `systemctl restart sshd`
* `systemctl cat sshd`


Nastavení systemd

* `/usr/lib/systemd/system/`
* `/usr/lib/systemd/system/sshd.service`
* `/etc/systemd/system/` – pro administrátora
* symlinky na `/dev/null`

Typy „units“:

`ls /usr/lib/systemd/system/ | cut -d. -f2`

* `service`
* `target`
* `socket`
* `timer`
* `mount`
* ...

Použití plného jména:

* `systemctl status sshd.service`
* `systemctl status graphical.target`
* `systemctl cat sshd.service`
* `systemctl cat sshd.socket`

(Odbočka k *socket*?)

* `systemctl cat graphical.target`

Závislosti:

* `systemctl list-dependencies sshd`
* `systemctl list-dependencies graphical.target`

Dokumentace:

* `man systemd.service`, ...
* `man systemd.service`, `man systemd.mount`


## Timer

* `man systemd.timer`
* `man systemd.time`
* `man systemd.unit`


## Naše služba

`/etc/systemd/system/ahoj.service`:

```
[Unit]
Description=echo

[Service]
Type=oneshot
ExecStart=/usr/bin/echo Ahoj!
```

* `sudo systemctl daemon-reload`
* `sudo systemctl start ahoj.service`
* `sudo systemctl status ahoj.service`

`/etc/systemd/system/ahoj.timer`:

```
[Unit]
Description=echo timer

[Timer]
OnCalendar=*-*-* *:*:00/10
AccuracySec=1s
Unit=ahoj.service
```

* `sudo systemctl daemon-reload`
* `sudo systemctl start ahoj.timer`
* `sudo systemctl status ahoj.service` (!)

## Uživatelská služba & časovač

```
mkdir -p ~/.config/systemd/user/
```

`~/.config/systemd/user/gong.service`:

```
[Unit]
Description=Gong

[Service]
Type=oneshot
ExecStart=/usr/bin/mplayer /usr/share/sounds/gnome/default/alerts/sonar.ogg
```


`~/.config/systemd/user/gong.timer`:

```
[Unit]
Description=Gong

[Timer]
OnCalendar=*-*-* *:*:00/10
AccuracySec=1s
Unit=gong.service
```
