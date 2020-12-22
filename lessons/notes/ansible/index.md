# Automatizace administrace – Ansible

Představ si, že jsi systémový administrátor na serverové farmě. 
Staráš se o 500 počítačů. Každý z nich má nastavený `cron` nebo timer na zálohy.
Třetina má nastavený webový server, další třetina je velký cluster databází a poslední třetina dělá maily.
Tvůj úkol je zajistit, aby se na všech těch serverech záloha dělala dvakrát častěji.

Co uděláš?
Připojíš se přes `ssh` k prvnímu serveru a pozměníš `crontab` nebo timer.
Hurá! Jen 499 a můžeš jít domů!

Jiná varianta je, že si napíšeš v Bashi cyklus, který se připojí ke každému počítači a změní interval crontabu.
Budeš mít z toho skvělý pocit a práce bude odvedená během chvilky.
...Jenže pak zjistíš, že u serveru číslo 54 se to jaksi nepovedlo.
Což znamená, že bys měl{{a}} zjistit, jestli u ostatních serverů to dopadlo dobře, nebo taky ne.

Teď si představ, že na tvé farmě právě jeden z tvých serverů shořel.
Nakoupíš nový počítač a chceš, aby fungoval úplně stejně jako ten předchozí.
Potřebuješ určitou verzi databáze s určitou konfigurací, a taky tam potřebuješ
nastavit `crontab` včetně poslední změny, zvýšení frekvence záloh.

Změny konfigurace na vícero strojích se dnes už nedělají tak,
že jdeš k počítači, přímo na něm upravíš konfigurační soubor
a následně restartuješ příslušnou službu.
Používají se na to systémy, kde si do souboru napíšeš,
jak má tvůj systém vypadat a pustíš program,
který se připojí na těch 500 serverů a každý nastaví podle toho seznamu.
Kdž pak server shoří, vyměníš ho za jiný a pustíš ten program znova.
A všechno nastavení máš v Gitu, takže na něm můžeš spolupracovat v týmu.

Na takové věci se používají systémy na správu konfigurace, jako jsou
Ansible, Chef, Puppet, SaltStack, nebo Terraform.
Popíšeš si jaké počítače se mají nastavit a jak, pustíš jeden z těchto
programů a serverová farma je nastavená.

Drobná nevýhoda je, že je to další systém, který se musíš naučit konfigurovat.
Dnes je to ovšem čím dál rozšířenější a systémoví administrátoři s Ansiblem
nebo podobnými systémy běžně pracují.


## Deklarativní zápis

Jeden z konceptuálních rozdílů mezi klasickou administrací a Ansible je ten,
že Ansible je *deklarativní*.
Místo toho, že bys spouštěl{{a}} příkazy které něco udělají,
popíšeš jaký má být výsledný stav nastavovaného systému.
Úkol Ansible je potom zkontrolovat, že je systém ve správném stavu – a pokud
není, 

V praxi se tyto přístupy tolik neliší (`sudo dnf install httpd` taky nedělá
nic, pokud je `httpd` už nainstalovaný), ale jde o užitečný úhel pohledu.


## Idempotence

A když už jsme u cizích slov: úkoly pro Ansible by měly být *idempotentní*,
když Ansible pustíš dvakrát za sebou, ve druhém běhu se nic nestane.
Například „zajisti, že je nainstalovaný `httpd`“ je idempotentní – pokud to
provedeš několikrát za sebou, výsledek je stejný jako kdybys to provedl{{a}}
jen jednou.

Oproti tomu „vytvoř nový šifrovací klíč pro `ssh`“ idempotentní není – pokaždé
se vytvoří jiný klíč.
Idempotentní úkol by byl „zajisti, že je vytvořený šifrovací klíč pro `ssh`“.

> [note]
> Ansible nezaručuje, že všechny úkoly jsou opravdu idempotentní.
> Lze tak například úkol „Napiš zprávu, že je vše OK“, který zapíše novou
> zprávu při každém spuštění.
> Dobře napsané úkoly ale idempotentní jsou.
> (Aspoň v těch důležitých aspektech.)


## Instalujeme program

Nejlepší způsob jak se naučit pracovat s Ansiblem je praxe.
Zkusme si nainstalovat balíček.
Ten si nakonfigurujeme.

Nainstaluj Ansible.
(To je potřeba udělat jen na jednom počítači v celé serverové farmě  – na tom,
který bude řídit ty ostatní.)

```console
$ sudo dnf install ansible
```

Pak bude potřeba napsat konfigurační soubory.
Bude jich víc, tak si pro tuto lekci udělej si nový adresář.

Pro Ansible se píšou tzv. *playbooky*, což jsou soubory ve formátu
[YAML](https://en.wikipedia.org/wiki/YAML).
Podobně jako Python tenhle formát vyžaduje odsazování a bude si stěžovat,
pokud ho nedodržíš.
Pozor tedy na mezery a pomlčky na začátcích řádků.

Napiš si playbook s názvem `setup.yml`.

První věc, kterou nastavíme, jsou informace o počítačích které chceš
konfigurovat: které to jsou a jak se k nim připojit.
Protože máš teď k dispozici jen jeden virtuální počítač,
nastav `hosts` na `localhost` a `connection` na `local`.

```
# setup.yml

- hosts: localhost
  connection: local
```

V praxi se tahle sekce většinou píše do samostatného souboru.

Abys mohl{{a}} něco nainstalovat, je třeba býti `root`em.
To je potřeba zapsat; později pak řekneme ať k tomu Ansible použije `sudo`
a heslo.

```
   become_user: root
```

Dále se píše stav, do jakého chceš dostat svůj systém.
Vytvoř si tedy seznam `tasks`.
V YAML se seznamy definují pomlčkou u každého prvku.

Každá úloha musí mít jméno.
Nainstalujeme si třeba `htop`, což je vylepšený `top`. 
Pro instalaci musíš být `root`, což musíš zadat i v úloze:

```
   tasks:
   - name: Install htop
     become: yes
```

Přichází čas na instrukci, jak balíček nainstalovat.
Na Fedoře se instaluje pomocí pluginu `dnf`, kterému řekneš,
že balíček `htop` chceš nainstalovat v poslední dostupné verzi:

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

Díky parametru `-K` se tě Ansible nejdřív zeptá na heslo pro `sudo`
a s tím pak spouští celý *playbook*.

Když ten samý příkaz pustíš znova, Ansible ohlásí že je vše v pořádku;
nic se nezměnilo.


## A co dál?

Co všechno se dá dělat s Ansiblem?
- instalovat balíčky (to jsme si právě zkusili)
- kopírovat soubory - např. konfigurační soubor pro `cron`.
- přidávat uživatele, skupinu uživatelů
- a mnohem víc...

Existují pro to tzv. *pluginy*, které dělají přesně tyto věci. 
Pokud ti nebude stačit nabídka pluginů, můžeš si v Pythonu napsat další.

Tedy je malá ukázka toho, co je možné; další možnosti najdeš v dokumentaci.
(Nebo na dalším kurzu?)

```
- hosts: localhost
  connection: local
  become_user: root
  tasks:
   - name: Install packages
     become: yes
     dnf:
        state: latest
        name:
        - htop
        - httpd
   - name: Uninstall packages
     become: yes
     dnf:
        state: removed
        name:
        - cowsay
   - name: Add a user
     become: yes
     user:
        name: testuser1
   - name: Add a file to serve
     become: yes
     copy:
        dest: /var/www/html/index.html
        # Znak `|` v YAML uvozuje odsazený blok s víceřádkovým textem
        content: |
            Hello world!
            Welcome to my server!
   - name: Start & enable httpd
     become: yes
     systemd:
        name: httpd
        state: started
        enabled: true
```


Tento příklad se připojuje pouze k aktuálnímu počítači. 
Pokud bys to chtěl{{a}} pouštět i na jiných počítačích,
je třeba je zapsat do takzvaného *inventáře* (*inventory* file - `/etc/ansible/hosts`).
Návod a příklady opět najdeš v dokumentaci Ansible.
