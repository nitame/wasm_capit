## Utiliser les outils pour intéragir avec le moteur Javascript

L'interfaçage entre le code Rust et les fonctionnalités du navigateur peuvent-être pénibles à mettre en place si le
projet ne se base pas sur de l'outillage permettant de faire la glue entre Rust, Wasm et Javascript. C'est là que
wasm-pack vient à la rescousse.

### L'outillage

Grâce à l'outillage Rust+Wasm, il est possible de coder des projets en Rust et de les utiliser dans un projet web en
passant par Wasm. Voici quelques outils à connaître.

#### wasm-pack

wasm-pack se définit comme un Wasm workflow tool. Il peut
être [installé](https://rustwasm.github.io/wasm-pack/installer/) très facilement en ligne de commande. Il faut au
préalable avoir installé Rust. wasm-pack est très pratique pour créer, construire, tester et publier des modules wasm à
partir de code Rust. Et les intégrer à l'écosystème Javascript et dans le navigateur. Cela permet notamment de créer
des modules Wasm et de les importer comme s'il s'agissait de paquets JS directement dans des projets web sans se
préoccuper de l'API WebAssembly.

#### wasm-bindgen

wasm-bindgen est un utilitaire qui permet de s'interfacer avec l'API JS du navigateur. l'avantage d'utiliser
wasm-bindgen est que de nombreuses fonctionnalités natives du navigateur sont déjà implémentées et prêtes à l'emploi
dans le code Rust. Cela évite d'avoir à écrire des bindings bas niveau pour accéder au DOM par exemple.

### Un projet Rust avec wasm-pack et wasm-bindgen

Reprenons l'exemple de notre projet précédent, un simple greeting, qui prendrait en input un nom. Mais cette fois-ci, il
sera intéressant d'utiliser des fonctionnalités du navigateur comme `alert` et `prompt`.

Voici le code Rust :

```rust
use wasm_bindgen::prelude::*;

#[wasm_bindgen]
extern "C" {
    fn alert(s: &str);
    fn prompt(s: &str) -> String;
}

#[wasm_bindgen]
pub fn hi_mate() {
    let name = prompt("What's ye name?");
    alert(&format!("Ho hi, {}!", name));
}
```

Et les dépendances dans Cargo.toml :

```toml
[dependencies]
wasm-bindgen = "0.2.79"

[lib]
crate-type = ["cdylib"]
```

La macro `#[wasm_bindgen]` positionnée au-dessus de `extern "C"`sert à importer des fonctions JS dans le code Rust. Ce
qu'il faut comprendre c'est que l'on n'importe pas réellement les fonctions. Mais que des bindings ont été codé en Rust
pour pouvoir interagir avec l'API JS du navigateur. Et que lorsque le module wasm sera exécuté par le navigateur il fera
bien appel à la fonction `window.alert()`.

Pour voir à quoi ressemble ces bindings il faut aller voir l'API de la
lib [web-sys](https://docs.rs/web-sys/latest/web_sys/struct.Window.html#method.alert) (qui est une dependence de
wasm-bindgen)

Extrait du code source de web-sys :

```rust
#[wasm_bindgen(catch, method, structural, js_class = "Window", js_name = alert)]
#[doc = "The `alert()` method."]
#[doc = ""]
#[doc = "[MDN Documentation](https://developer.mozilla.org/en-US/docs/Web/API/Window/alert)"]
#[doc = ""]
#[doc = "*This API requires the following crate features to be activated: `Window`*"]
pub fn alert(this: &Window) -> Result<(), JsValue>;
```

La macro `#[wasm_bindgen]` placée au-dessus de `pub fn hi_mate()` sert à exporter la fonction Rust pour qu'elle puisse
être utilisée par le navigateur. D'ailleurs, c'est ce que nous allons voir maintenant. Pour utiliser la
fonction `hi_mate` il faut importer le module wasm dans un script JS.

wasm-pack permet de build un module wasm de différentes manières, celle utilisée ici est la target web.
`wasm-pack build --target web`
Cela produit un dossier `pkg` avec les fichiers suivants :

```text
pkg
├── package.json
├── simple_example_with_wasm_pack.d.ts
├── simple_example_with_wasm_pack.js
├── simple_example_with_wasm_pack_bg.wasm
└── simple_example_with_wasm_pack_bg.wasm.d.ts
```

Ce qui est intéressant, c'est le fichier .js, on peut voir que la fonction init est exportée et que toute la glue qui
permet de charger le binaire wasm avec l'API `WebAssembly` est déjà généré de façon optimisée.

```javascript
export function hi_mate() {
    wasm.hi_mate();
}

async function load(module, imports) {
    if (typeof Response === 'function' && module instanceof Response) {
        if (typeof WebAssembly.instantiateStreaming === 'function') {
            try {
                return await WebAssembly.instantiateStreaming(module, imports);

            } catch (e) {
                // ...
            }
        }

        const bytes = await module.arrayBuffer();
        return await WebAssembly.instantiate(bytes, imports);
        // ...
    }
}

async function init(input) {
    if (typeof input === 'undefined') {
        input = new URL('simple_example_with_wasm_pack_bg.wasm', import.meta.url);
    }
    // ...
    if (typeof input === 'string' || (typeof Request === 'function' && input instanceof Request) || (typeof URL === 'function' && input instanceof URL)) {
        input = fetch(input);
    }

    const {instance, module} = await load(await input, imports);

    wasm = instance.exports;
    init.__wbindgen_wasm_module = module;

    return wasm;
}

export default init;
```

Ce qui permet d'importer la function `hi_mate` dans notre script JS comme s'il s'agissait d'une fonction exportée depuis
un module NPM. La preuve par le code :

```html

<script type="module">
    import init, {hi_mate} from "./pkg/simple_example_with_wasm_pack.js";

    init().then(() => {
        hi_mate()
    });
</script>
```

Et lorsqu'un serveur est lancé pour servir le fichier HTML et que l'on ouvre le navigateur. Il y a bien une fenêtre
prompt qui demande d'insérer un nom puis une fenêtre alert qui affiche un greeting avec le nom.
