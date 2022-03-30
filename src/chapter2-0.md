## Wasm 101

### WASM, WAT, WASI, ... ???

![étapes de compilation](./images/figure-2-0a.png)
L'illustration ci-dessus montre les étapes qui permettent de passer du code au binaire à execution par une runtime.

### WebAssembly

Quand on parle de Wasm, cela peut faire référence à la spec du format d'instruction binaire, à la VM, ou encore aux
fichiers binaires (.wasm) qui vont être exécutés par le moteur. Par la suite, nous allons nous intéressé surtout aux
fichiers .wasm et aux manières de produire ce binaire.

### WebAssembly text format

WebAssembly Text Format abbrégé par WAT, est une representation textuelle du binaire Wasm compréhensible et éditable par
des humains. WAT est basé sur les S-expressions.

### WebAssembly system interface

WebAssembly System Interface abrégé par WASI, est une interface (comme son nom l'indique) qui permet de "communiquer"
avec un OS conceptuel (à différencier d'un OS spécifique). WASI a pour but de proposer d'utiliser des binaires wasm pour
interagir avec n'importe quel OS de façon standardisée.

### WebAssembly runtime

Pour pouvoir exécuter un binaire wasm sur une machine (device, server, ...) il manque une dernière brique. Un runtime
qui permet de lancer les instructions contenues dans le binaire et interagir avec les différents process de la machine
cible. Il existe plusieurs wasm runtimes : wasmtime, wasmer, etc.

> enough babbling! time to show the code.