# Úkoly - Odkazy

## Bezedný adresář

Vytvoř si nový, prázdný adresář, přejdi do něj a zadej:

```console
$ mkdir bezedny
$ ln -s . bezedny/bezedny
$ ls -l bezedny/
```

Zkus si:

<code>$ ls -l bezedny/<kbd>tab</kbd><kbd>tab</kbd><kbd>tab</kbd></code>

```console
$ tree bezedny
$ tree -l bezedny
```

Co se právě stalo?


> [note]
> Na tohle je dobré dávat pozor, až budeš psát program,
> který prochází adresářové struktury.
> Vždy si ověř, že neprocházíš donekonečna jeden adresář.


## Viz zde

A co se stane tady?

```console
$ ln -s tu tu
$ ls -l tu
$ cat tu
```
