## Il était une fois Wasm

**1989 : WWW**

Tim Berner Lee invente le world wide web

**1995 : javascript**

Javascript a été créé en 10 jours par Brendan Eich. Le langage n'était pas sans défaut, surtout côté perf, mais pour le
rendu de formulaire HTML ça fonctionnait plutôt bien.

**2008 : chrome, moteur v8**

Avec des sites web qui nécéssitaient toujours plus de fonctionnalités d'affichage, les perfs en JS devenaient un sujet.
Google avec son browser chrome et le moteur v8, qui permet de faire de la compilation JIT de JS, apporte des niveaux de
performances assez incroyables et ouvre la voie à ce que va devenir JS et l'écosystème web que l'on connaît aujourd'hui.

**2015 : Wasm**

Web Assembly permet d'exécuter du code écrit dans d'autres langages que JS, dans le browser comme s'il s'agissait de
modules natifs. Dit autrement, cela permet d'écrire du code en *c, c++, rust, etc.* qui va être exécuté par le browser
et d'avoir le même niveau de performance que s'il était exécuté nativement sur une machine.
