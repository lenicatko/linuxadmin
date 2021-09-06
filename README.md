# Linuxová administrace

Tady budou vznikat otevřené materiály pro kurz linuxové administrace.


## Instalace a spuštění

Chceš-li server spustit na svém počítači, např. proto, že se chceš zapojit do
vývoje, je potřeba ho nejdřív nainstalovat:

### Instalace `poetry`

Nemáš-li `poetry`, nainstaluj si jej. Na to je několik způsobů:

* podle [návodu](https://python-poetry.org/docs/)
* v aktivovaném [virtuálním prostředí](https://naucse.python.cz/lessons/beginners/install/)
  pomocí `python -m pip install poetry`
* na Fedoře pomocí balíčkovacího systému: `sudo dnf install poetry`

### Instalace závislostí

Přepni se do adresáře s projektem a spusť:

```console
$ poetry install --dev
```

### Lokální server

Chceš-li si kurz prohlédnout, přepni se do adresáře s projektem a spusť:

```console
$ poetry run python -m naucse serve
```

* Program vypíše adresu (např. `http://0.0.0.0:8003/`).
  * Buď adresu navštiv v prohlížeči a doklikej se na kurz, nebo
  * na konec adresy přidej `/course/local/` a navštiv kurz přímo.

### Sestavení pro naucse.python.cz

Sestavení pro naucse.python.cz se dělá automaticky, když pošleš větev `main`
na GitHub.
Chceš-li to ale udělat ručně, zadej:

```console
$ poetry run python -m naucse_render compile _compiled
```

V adresáři `_compiled` pak najdeš veškeré informace o kurzu v strojově čitelné
podobě.

### Přidání kurzu

(TODO)


## Licence

Každá lekce má vlastní licenci, která je uvedena v metadatech.
Používáme pouze [licence pro otevřený obsah][free content licenses].
Všechen obsah musí mít uvedenou licenci.

---

Content has its own license specified in the appropriate matadata.
Only [free content licenses] are used. By contributing to an already licensed
document, you agree to have it licensed under the same license.
(And feel free to add yourself to the authors list in its metadata.)
When contributing new document(s) a license must be specified in the metadata.

[free content licenses]: https://en.wikipedia.org/wiki/List_of_free_content_licenses
