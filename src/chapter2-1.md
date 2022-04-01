## Compiler un langage haut niveau vers un binaire wasm

Pour la suite de cette démonstration, je vais utiliser Rust comme langage haut niveau. De nombreux langages possèdent
déjà les outils pour permettre d'obtenir un binaire wasm (c/c++, javascript, Typescript, ...) pour les autres il s'agit
certainement d'une question de temps.

### Pourquoi Rust ?

Je vous dirais bien que Rust possède des killer features comme le borrow-checker, une gestion mémoire au poil, du
pattern matching et des macros hygiéniques. Rust a tout ça, mais si j'ai choisi Rust, c'est parce que je ne suis pas un
bon développeur et que je ne sais pas écrire du c/c++ sans tomber sur une dangling pointer exception ou un crash sur un
programme multi-thread que je suis incapable de debugger. Sans compter que les outils Rust+Wasm existent déjà et qu'ils
fonctionnent bien.

### Salut Wasm, depuis Rust

#### 🛠️ Prérequis 🛠️

- Avoir installé rustup, rustc, cargo. Si ce n'est pas déjà fait suivre les
  instructions [ici](https://www.rust-lang.org/tools/install).
- Avoir ajouté la target wasm32-unknown-unknown aves rustup, si ce n'est pas déjà fait, lancer la
  commande `rustup target add wasm32-unknown-unknown` dans un terminal

#### 🚧 Commencer le projet 🚧

Lancer la commande `cargo new --lib simple-example-without-tools`. Cela créer le projet

Dans le fichier `scr/lib.rs` ajouter le code suivant :

```rust
#[no_mangle]
fn answer() -> u32 {
    42
}
```

Ajouter la configuration suivante dans le fichier Cargo.toml :
```toml
[lib]
crate-type = ["cdylib"]
```

Puis construire l'artéfact wasm avec la commande suivante :
`cargo build --target wasm32-unknown-unknown --release`

Cela produit le binaire wasm dans le dossier target

```text
.
├── Cargo.lock
├── Cargo.toml
├── index.html
├── README.md
├── src
│  └── lib.rs
└── target
   ├── CACHEDIR.TAG
   ├── release
   │  ├── build
   │  ├── deps
   │  ├── examples
   │  └── incremental
   └── wasm32-unknown-unknown
      ├── CACHEDIR.TAG
      └── release
         ├── build
         ├── deps
         │  ├── simple_example_without_tools.d
         │  └── simple_example_without_tools.wasm
         ├── examples
         ├── incremental
         ├── simple_example_without_tools.d
         └── simple_example_without_tools.wasm <-- voici le binaire Wasm
```

Dans un fichier HTML à la racine du projet, importer le binaire wasm avec une balise script

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title - simple rust wasm</title>
</head>
<body>
<script>
    WebAssembly.instantiateStreaming(fetch('target/wasm32-unknown-unknown/release/simple_example_without_tools.wasm'))
            .then(obj => {
                console.log('The answer is: ', obj.instance.exports.answer());
            });
</script>
</body>
</html>
```

Dans le navigateur, en ouvrant le fichier HTML. La console du navigateur affiche bien la réponse.

```text
The answer is: 42
```

#### 🙌 It's alive!!! 🙌

Pour résumer, voici les 2 étapes pour exécuter un binaire wasm dans le navigateur :

* compiler du code avec pour cible Wasm, il faut que les outils de compilation du langage supportent le standard wasm
  pour pouvoir exposer les fonctions avec les types de données gérés par Wasm.
* importer le code dans le navigateur, il faut que le navigateur intègre l'API WebAssembly qui permet d'interagir avec
  le binaire Wasm

L'exemple de code ne montre pas les limitations de Wasm (Seulement 4 types de données int32, int64, float32, float64)
pour les voir il faut jouer avec des types de données plus complexes. Par exemple les chaînes de caractères.

Dans ce cas, il faut avoir un code rust qui ressemble à ça :

```rust
use std::ffi::CString;
use std::os::raw::c_char;

#[no_mangle]
fn hi_mate() -> *const c_char {
    let c_to_print = CString::new("Ho hi, mate!").expect("CString::new failed");
    c_to_print.into_raw()
}

#[no_mangle]
fn hi_mate_len() -> usize {
    "Ho hi, mate!".len()
}
```

Et un script dans le fichier HTML beaucoup plus verbeux :

```html

<script>
    WebAssembly.instantiateStreaming(fetch('target/wasm32-unknown-unknown/release/simple_example_without_tools.wasm'))
            .then(obj => {
                const linearMemory = obj.instance.exports.memory;
                const offset = obj.instance.exports.hi_mate();
                const len = obj.instance.exports.hi_mate_len();
                const stringBuffer = new Uint8Array(linearMemory.buffer, offset, len);
                let str = '';
                for (let i = 0; i < stringBuffer.length; i++) {
                    str += String.fromCharCode(stringBuffer[i]);
                }
                console.log(str);
            });
</script>
```

Cela montre qu'il est tout à fait possible de passer outre les "limitations" dues aux 4 types de données utilisables dans
le standard Wasm, mais que cela a un coût. Celui d'écrire de la glue assez bas niveau ne serait-ce que pour manipuler
des chaînes de caractères.

Mais pas de panique, il existe déjà des outils qui permettent d'écrire du code haut niveau sans se préoccuper des
bindings entre Rust, Wasm et le moteur Javascript du navigateur. Ce sera l'exemple du prochain chapitre.