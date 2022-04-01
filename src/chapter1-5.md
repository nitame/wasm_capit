## Wasm : c'est le turfu ou encore un truc de hipster ?

<details>
    <summary>Le(s) point(s) faible(s) de Wasm</summary>

### Wasm ne supporte que 4 types de donnée

* les entiers 32 bit
* les entiers 64 bit
* les flottants 32 bit
* les flottants 64 bit

Donc pour bosser sur des chaînes de caractères en UTF-8 par exemple, on est obligé de passer par de l'encodage avec des
entiers et des vecteurs. Mais rassurons-nous des gens très bien ont commencé à travailler là-dessus pour nous proposer
des runtimes ([wasmitme](https://wasmtime.dev/), [wasmer](https://wasmer.io/)) qui embarque de plus en plus de
fonctionnalités.

### L'outillage ? Pas encore assez mature

### Les bindings. Il faut parfois réécrire du code bas niveau pour utiliser des librairies existantes

</details>