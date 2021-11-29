# Firewall

> [warning]
> Toto jsou jen poznámky, ne materiály k samostudiu


Ještě než půjdeme na náš dnešní úkol, musíme pozměnit nastavení firewallu na naší Fedoře.
Firewall je síťové zařízení, které povoluje/blokuje připojení podle zadaných parametrů.
Dnes spustíme webový server.
Webové servery poslouchají na portu 80, který musíš povolit. 

Port je číslo, které říká k jaké službě na daném počítači se chceš připojit.
Většina webových služeb má výchozí číslo portu, který se použije když nezadáš nějaký jiný.
- Webserver má port 80 (HTTP), 443 (HTTPS).
- E-mailový server má port 993 (IMAPS) a 995 (POP3S).
- Vzdálený shell má port 22 (SSH).

Pokud ti běží server (teď ještě ne, ale už za chvilku) a chceš na něho vidět zvenku,
z jiného počítače, je třeba otevřít port 80.
(Nebo nějaký jiný, ale uživatelé by pak museli do svých prohlížečů to jiné číslo portu zadávat.)

Když nainstaluješ nový systém, nainstaluje se ti s ním firewall,
který defaultně porty z beszpečnostních důvodů zavíře – znepřístupní.
Na konfiguraci firewallu existuje nástroj.
Webový server na virtuálním počítači povolíš příkazem:
```console
# firewall-cmd --add-service=http
success
``` 
A zapiš, že má být povolen port pro `http` po dalším startu:
```console
# firewall-cmd --permanent --add-service=http 
success
```

## Zjištění adresy

```
$ ip a
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
2: enp1s0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel state UP group default qlen 1000
    link/ether 52:54:00:1f:2c:5b brd ff:ff:ff:ff:ff:ff
    inet 192.168.122.64/24 brd 192.168.122.255 scope global dynamic noprefixroute enp1s0
       valid_lft 3107sec preferred_lft 3107sec
    inet6 fe80::4aa5:69dc:eba6:6e54/64 scope link noprefixroute 
       valid_lft forever preferred_lft forever
```
