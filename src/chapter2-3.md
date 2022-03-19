## Wasm sans le navigateur

Après avoir vu comment écrire et utiliser des modules wasm dans le navigateur, avec et sans outillage, il est temps de
voir toutes les potentialités de wasm. Parce que wasm est une stack-based virtual machine et peut donc s'interfacer avec
n'importe quel matériel et n'importe quel OS pour peu que ce dernier embarque un wasm runtime.

### Wasm runtime

Après avoir compilé un module wasm vers un binaire wasm, il faut que la machine hôte puisse exécuter ce binaire et donc
comprendre les instructions qui lui sont passées. Un des points fort de Wasm est la portabilité. Cela signifie que
l'exécution d'un binaire wasm aura un résultat prédictible peu importe l'OS sur lequel il est exécuté.

Mais pour arriver à ce résultat, il faut bien un outil (runtime) qui est capable de comprendre les instructions
contenues dans le binaire et de faire le lien avec les fonctions natives de l'OS. Par exemple, lire un stream, charger
un buffer en mémoire, donner accès au filesystem. La liste est longue.

Voici une liste non exhaustive des runtimes pour WebAssembly qui existent :

* wasmer
* wasmtime
* lucet
* WAMR - WebAssembly Micro Runtime

Pour l'exemple de code, nous allons utiliser wasmer.

### WASI

Avant d'installer wasmer et d'exécuter notre module wasm, quelques précisions sur WASI.

Dans le précédent paragraphe, il était question du runtime et de sa capacité à faire le lien entre le binaire wasm et
les fonctions natives de l'OS. Pour que cela fonctionne sur la multitude d'OS et de hardware qui existe aujourd'hui, il
faut un system interface, c'est-à-dire une API, qui expose la façon dont l'OS accède, par exemple, à la hiérarchie du
filesystem.

WASI est une initiative pour standardiser la façon dont les runtimes s'interfacent avec les OS, pour garantir une
portabilité générique et assurer de la stabilité pour les outils qui s'appuient sur les runtimes wasm.

### Exemple avec du code

Enfin ! Nous allons pouvoir lancer un programme écrit en rust, compilé en binaire wasm et exécuter par wasmer (wasm
runtime) sur une machine sans passer par le navigateur.

#### 🛠️ Prérequis 🛠️

- Avoir installé [wasmer](https://github.com/wasmerio/wasmer#install)
- Avoir ajouté la target wasm32-wasi aves rustup, si ce n'est pas déjà fait, lancer la
  commande `rustup target add wasm32-wasi` dans un terminal

#### 🚧 Coder le projet 🚧

Lancer la commande `cargo new simple-example-with-wasmer`. Cela créer le projet, cette fois il ne s'agit pas d'une lib
rust mais d'un exécutable. Le point d'entré est le fichier `main` et la fonction `main` va être exportée par défaut dans
la target wasm.

Dans le fichier `scr/main.rs` ajouter le code suivant :

```rust
fn main() {
    println!("What's ye name?");
    let mut buffer = String::from("");
    let stdin = std::io::stdin();
    stdin.read_line(&mut buffer);
    println!("Ho hi, {}!", buffer.trim());
}
```

My, my, my! C'est du code Rust, pas de macro pour interagir avec le runtime wasm. On se sent comme à la maison, en
charentaises et en peignoir au coin du feu.

Pas de spécificité dans le `Cargo.toml` non plus :

```toml
[package]
name = "simple-example-with-wasmer"
version = "0.1.0"
edition = "2021"

[dependencies]
```

Il ne reste plus qu'à compiler le code rust vers la cible wasm-wasi :
`cargo build --target wasm32-wasi --release`
On retrouve le binaire produit dans le dossier `target`

```text
.
├── Cargo.lock
├── Cargo.toml
├── README.md
├── src
│  └── main.rs
└── target
   ├── CACHEDIR.TAG
   ├── release
   │  ├── build
   │  ├── deps
   │  ├── examples
   │  └── incremental
   └── wasm32-wasi
      ├── CACHEDIR.TAG
      └── release
         ├── build
         ├── deps
         │  ├── simple_example_with_wasmer-11e993127ad92e3a.d
         │  └── simple_example_with_wasmer-11e993127ad92e3a.wasm
         ├── examples
         ├── incremental
         ├── simple-example-with-wasmer.d
         └── simple-example-with-wasmer.wasm <-- le binaire wasm est ici !
```

#### 🙌 Show time 🙌

Lorsque la commande `wasmer run target/wasm32-wasi/release/simple-example-with-wasmer.wasm` le runtime wasm exécute le
programme. Et c'est fait 🎉. On a notre premier programme compilé en wasm qui peut s'exécuter sur n'importe quelle
machine qui embarque un runtime wasm.
