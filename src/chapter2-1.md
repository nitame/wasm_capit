### Compiler un langage haut niveau vers un binaire wasm

Pour la suite de cette démonstration, je vais utiliser Rust comme langage haut niveau. De nombreux langages possèdent
déjà les outils pour permettre d'obtenir un binaire wasm (c/c++, javascript, Typescript, ...) pour les autres il s'agit
certainement d'une question de temps.

#### Pourquoi Rust ?

Je vous dirais bien que Rust possède des killer features comme le borrow-checker, une gestion mémoire au poil, du
pattern matching et des macros hygiéniques. Rust a tout ça, mais si j'ai choisi Rust, c'est parce que je ne suis pas un
bon développeur et que je ne sais pas écrire du c/c++ sans tomber sur une dangling pointer exception ou un crash sur un
programme multi-thread que je suis incapable de debugger. Sans compter que les outils Rust+Wasm existent déjà et qu'ils
fonctionnent bien.

#### Salut Wasm, depuis Rust

🛠️ **Prérequis** 🛠️

- Avoir installer rustup, rustc, cargo. Si ce n'est pas déjà fait suivre les
  instructions [ici](https://www.rust-lang.org/tools/install).
- Avoir ajouté la targert wasm32-unknown-unknown aves rustup, si ce n'est pas déjà fait, lancer la
  commande `rustup target add wasm32-unknown-unknown` dans un terminal

🚧 **Commencer le projet** 🚧
Lancer la commande `cargo new --lib salut-wasm`. Cela créer le projet

