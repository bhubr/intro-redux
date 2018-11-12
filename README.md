# Introduction à Redux

## Qu'est-ce que Redux ?

D'après la [page d'accueil](https://redux.js.org/) :

> Redux is a predictable state container for JavaScript apps.

Un conteneur d'état prédictible pour les applications JavaScript, donc. Mais encore ?

Voyons ce que cette définition recouvre, et surtout le point de départ. Les applications JavaScript, notamment front-end, doivent gérer un certain nombre de données "dynamiques" &mdash; dans le sens où elles peuvent évoluer dans le temps, et en fonction des interactions de l'utilisateur :

* L'utilisateur est-il connecté ou non ?
* A-t-il des messages (notamment non-lus) dans sa boîte de réception, et si oui, combien ?
* A-t-il ouvert une ou des fenêtres de chat avec ses amis connectés ?
* etc.

Les différentes librairies et frameworks front-end ont chacun leur approche de la gestion de ces données.

Dans les applications React, cela se fait via le state. Quand une certaine donnée  doit être partagée entre deux composants, il est courant de la placer dans le state du parent commun de ces deux composants. Pour le reste, certains composants (exemple: formulaires) peuvent gérer leur propre state de façon assez indépendante.

La gestion du state se complexifie au fur et à mesure que l'application grossit :

* Le fait de gérer plus de données différentes (utilisateurs, posts, messages de chat...) fait ajouter des clés / valeurs au state.
* On se retrouve alors avec beaucoup de props à passer, "d'étage en étage", jusqu'aux composants qui les nécessitent.
* Si la hiérarchie de composants s'étoffe, et qu'un composant situé "en bas" doit indirectement provoquer une modification du state du composant `App`, il doit le faire via une prop passée de composant en composant, depuis `App`.
* Il n'est parfois pas "évident" de savoir quel composant doit contenir, dans son state, quelle(s) donnée(s).

Redux propose une alternative efficace au passage de nombreuses props de composant en composant, et permet de simplifier la gestion de l'état.

Il faut garder à l'esprit que Redux est une *librairie*, indépendante de React. Bien que fonctionnant très bien avec React, elle peut être utilisée avec d'autres librairies ou frameworks, voire de façon autonome, sans framework.

Si on reprend la définition du début, Redux offre :

* un _conteneur_ appelé le "store" (magasin en français), qui va contenir _tout_ le state (si on travaille avec React). D'où le terme de "state container".
* Ce conteneur est _prédictible_ : des contraintes sont imposées sur la façon de modifier le contenu du store (le state). Si ces contraintes sont respectées, on peut prédire très précisément les évolutions du state.

## Comment ça marche ?

### Les actions

Avec Redux, l'état des données dans le store est mis à jour en envoyant à ce dernier des *actions*. Une action est un objet contenant :

* au moins une propriété, `type`, qui indique le but de cette action, autrement dit, quel type de modification elle doit apporter au state.
* éventuellement un ou des paramètres, permettant de fournir des informations au store.

On va donner deux exemples d'utilisation des actions. Ces exemples sont "incomplets", dans le sens où ils ne montrent pas encore comment le nouveau state est "calculé" à partir des propriétés de l'action.

#### Exemple 1 - Compteur

Dans cet exemple, notre state est un objet contenant une seule paire clé-valeur. La clé est `count`, et la valeur associée est initialisée à zéro. Le state initial est donc cet objet :

```javascript
{
  count: 0
}
```

Pour modifier le compteur, on va envoyer deux types d'actions au store :

* une action pour *ajouter* (type: `ADD`) un nombre à la valeur du compteur,
* une action pour *soustraire* (type: `SUBSTRACT`) un nombre à la valeur du compteur.

Voici quatre exemples, deux pour l'action *ajouter*, deux pour l'action *soustraire*, avec à chaque fois un paramètre indiquant ce qu'on doit ajouter ou soustraire à la valeur du compteur :

```javascript
// Ajouter 1 au compteur
const add1 = {
  type: 'ADD',
  quantity: 1
};

// Ajouter 10 au compteur
const add10 = {
  type: 'ADD',
  quantity: 10
};

// Soustraire 1 au compteur
const sub1 = {
  type: 'SUBSTRACT',
  quantity: 1
};

// Soustraire 10 au compteur
const sub10 = {
  type: 'SUBSTRACT',
  quantity: 10
};
```

Sans se préoccuper trop encore du lien entre React et Redux, on peut imaginer que ces quatre actions pourraient correspondre à 4 boutons dans une application.

Pour modifier le compteur, on va appliquer une série d'actions parmi les 4 ci-dessus (une action pouvant être appliquée plusieurs fois). Voici les valeurs successives du state après une série d'actions, en partant de l'état initial `{ count: 0 }` :

<div class="stateChanges">
* Appliquer `add1`<br><span class="arrow">&rarr;</span> `{ count: 1 }`
* Appliquer `add10` **deux fois**<br><span class="arrow">&rarr;</span> `{ count: 21 }`
* Appliquer `sub10`<br><span class="arrow">&rarr;</span> `{ count: 11 }`
* Appliquer `sub1` **quatre fois**<br><span class="arrow">&rarr;</span> `{ count: 7 }`
* Appliquer `add1` **deux fois**<br><span class="arrow">&rarr;</span> `{ count: 9 }`
</div>
Les modifications sont "cumulatives" : pour calculer le nouvel état après une action, on ne repart pas de l'état initial, mais toujours de l'état qui précédait l'action.

#### Exemple 2 - Données de formulaire

Ici, notre state est un objet stockant les données saisies dans un formulaire. Chaque paire clé-valeur permet de gérer l'état d'un champ spécifiquement (on peut imaginer que la clé correspond à l'attribut `name` d'un input, et la valeur à son attribut `value`).

Le state initial est un objet, correspondant aux données qu'on pourrait entrer dans un formulaire d'inscription basique :

```javascript
{
  name: '',
  email: '',
  password: ''
}
```

Pour modifier le formulaire, on dispose de deux actions :

* une action pour indiquer une valeur (`SET`) : elle aura besoin, en plus de `type`, de deux propriétés `key` (la clé) et `value`.
* une action pour remettre le formulaire à zéro (`RESET`), ne nécessitant pas de propriété autre que `type`.

Voici quelques exemples avec ces actions :

```javascript
const setName = {
  type: 'SET',
  key: 'name',
  value: 'John Doe'
};

const setEmail = {
  type: 'SET',
  key: 'email',
  value: 'johndoe@example.com'
};

const setPassword = {
  type: 'SET',
  key: 'password',
  value: 'secure1234'
};

const reset = {
  type: 'RESET'
};
```

Voici les valeurs successives du state en appliquant ces actions, en partant de l'état initial :

<div class="stateChanges">
* Appliquer `setName`<br><span class="arrow">&rarr;</span> `{ name: 'John Doe', email: '', password: '' }`
* Appliquer `setEmail`<br><span class="arrow">&rarr;</span> `{ name: 'John Doe', email: 'johndoe@example.com', password: '' }`
* Appliquer `setPassword`<br><span class="arrow">&rarr;</span> `{ name: 'John Doe', email: 'johndoe@example.com', password: 'secure1234' }`
* Appliquer `reset`<br><span class="arrow">&rarr;</span> `{ name: '', email: '', password: '' }`
</div>

### Les reducers

Les *reducers* décrivent *comment* le state doit être modifié, en réponse aux actions envoyées au store.

Un reducer est une fonction, qui reçoit comme arguments :

* le state actuel,
* une action,

et retourne le nouveau state (`newState`) :

```javascript
const reducer = (state, action) => {

  // ... Calcul de la nouvelle valeur du state

  // Renvoi de cette nouvelle valeur
  return newState
};
```

On va reprendre les deux exemples vus pour les actions.

#### Exemple 1 - Compteur

Pour rappel, dans cet exemple :
* on dispose de deux actions `ADD` et `SUBSTRACT`, chacune recevant un paramètre `quantity` avec la "quantité" à ajouter ou soustraire au state,
* le state initial, stocké ici dans `initialState`, est `{ count: 0 }`

Voici le reducer pour le compteur, qui va effectuer les modifications au state, en fonction des actions :

```javascript
const initialState = { count: 0 };

const counterReducer = (state = initialState, action) => {
  // On récupère le paramètre "quantity" de l'action
  const { quantity } = action;
  // On récupère la valeur du compteur dans le state
  const { count } = state;
  switch(action.type) {
    case 'ADD':
      return { count: count + quantity };
    case 'SUBSTRACT':
      return { count: count - quantity };
    // Si on reçoit une action d'un type non pris en charge
    // par le reducer, on renvoie le state tel quel
    default:
      return state;
  }
};
```
On indique `initialState` après le `=`, pour donner une valeur à `state` lors du premier appel au reducer.

Dans le code du reducer, l'utilisation de `switch` est une alternative à l'utilisation de `if/else`, et lui est préférable, car elle permet d'éviter une écriture "lourde" telle que :

```javascript
if(action.type === 'TYPE1') {
  // ...
}
else if(action.type === 'TYPE2') {
  // ...
}
else if(action.type === 'TYPE3') {
  // ...
}
else {
  return newState;
}
```

Un tel code deviendrait moins lisible au fur et à mesure que le nombre d'actions à gérer par le reducer augmenterait.

Reprenons les actions de l'exemple 1 de la section précédente :

```javascript
// Ajouter 1 au compteur
const add1 = {
  type: 'ADD',
  quantity: 1
};

// Ajouter 10 au compteur
const add10 = {
  type: 'ADD',
  quantity: 10
};

// Soustraire 1 au compteur
const sub1 = {
  type: 'SUBSTRACT',
  quantity: 1
};

// Soustraire 10 au compteur
const sub10 = {
  type: 'SUBSTRACT',
  quantity: 10
};
```

Si on appelle le reducer avec le state `{ count: 0 }` et l'action `add1`, on va entrer dans le `case 'ADD'` du `switch`. On renvoie un nouveau state, en attribuant, pour la clé `count`, l'ancienne valeur de `count` augmentée de 1. Le state retourné par le reducer est donc `{ count: 1 }`.

#### Exemple 2 - Données de formulaire

Pour rappel :
* on dispose de deux actions, `SET` (recevant une clé `key` et une valeur `value`), et `RESET` (sans arguments).
* on part du state suivant : `{ name: '', email: '', password: '' }`.

Voici le reducer permettant de traiter ces actions :

```javascript
const initialState = { name: '', email: '', password: '' };

const formReducer = (state = initialState, action) => {
  switch(action.type) {
    case 'SET':
      // On récupère la clé et la valeur dans l'action
      const { key, value } = action;
      // On renvoie une copie de l'ancien state (...state),
      // dans laquelle on "écrase" l'ancienne valeur associée
      // à la clé key
      return { ...state, [key]: value };
    case 'RESET':
      // On réinitialise tous les champs, avec une chaîne vide
      return initialState;
    default:
      return state;
  }
};
```

Le fait qu'on n'ait pas indiqué "en dur" `{ name: '', email: '', password: '' }` dans le `case 'RESET'`, mais qu'on ait utilisé `initialState`, déclaré hors de la fonction, permettrait de réutiliser ce reducer dans une autre application, pour un formulaire gérant d'autres données que des paramètres d'inscription.

Détaillons un peu ce qui se passe, si on appelle ce reducer, en passant comme state `{ name: '', email: '', password: '' }`, et comme action `{ type: 'SET', key: 'name', value: 'John Doe' }`.

`action.type` vaut `SET`, on entre donc dans le `case 'SET'`. On récupère les valeurs de `key` et `value` (`name` et `John Doe`).

Le return est suivi d'accolades ouvrantes, permettant de créer un nouvel objet. `...state` permet d'insérer dans cet objet toutes les paires clés-valeurs de `state`. Ensuite, `[key]: value` va ici valoir `name: 'John Doe'`. Du fait qu'on l'ait indiqué *après* `...state`, si la clé `name` existait déjà dans `state`, la valeur associée va être écrasée par `name: 'John Doe'`.

Avec cette action, on obtient `{ name: 'John Doe', email: '', password: '' }` en sortie du reducer.

### Le store

Ainsi que le précise la [section "Store"](https://redux.js.org/basics/store) de la documentation Redux, le store a les responsabilités suivantes :

* Contenir l'état de l'application
* Permettre l'accès à cet état, via une méthode `getState()` : le state n'est en effet pas accessible en tant que variable globale, pour éviter sa manipulation directe.
* Permettre la manipulation du state via une méthode `dispatch(action)`. C'est cette méthode qui permet d'appliquer une action. `dispatch` peut être traduit par envoyer, expédier, déployer.
* Permettre de mettre en place des *listeners* : un listener est une fonction, dont l'utilisation est semblable à un gestionnaire d'évènements, et est appelé à chaque mise à jour du state, après l'exécution du reducer. On "enregistre" un listener auprès du store via `subscribe(listener)`.
* Permettre de "désenregistrer" un listener. À cette fin, `subscribe(listener)` renvoie une fonction, qu'on peut stocker, puis appeler quand on souhaite désenregistrer le listener.

Tout le state est dans le store, et toute l'application n'utilise qu'*un seul* store.

Pour créer le store, on utilise la méthode `createStore` fournie par Redux, en lui fournissant deux arguments, le second étant optionnel :

* `reducer` : le reducer permettant de gérer les mises à jour du state
* `initialState` : la valeur initiale à donner au state contenu dans le store

Dans les sections suivantes, on va assembler toutes les notions vues jusqu'ici : *actions*, *reducers* et *store*, avec les deux exemples vus précédemment, et un troisième.

On va encore rester en dehors de React. Mais afin de rendre ces exemples plus parlants, chaque exemple sera une application interactive. Il nous faut donc aborder quelques notions de JavaScript natif, ce qu'on va faire dans la section qui suit.

### Préalable à la mise en oeuvre

#### Le DOM

Voici d'abord les notions minimales à connaître sur les fonctionnalités natives de JavaScript dans le navigateur, pour la manipulation du DOM (autrement dit, la page web chargée dans le navigateur).

Le DOM est une "arborescence" de noeuds, très hiérarchisée, chaque noeud correspondant à une balise d'un document HTML. Tout le DOM est contenu dans l'objet global `document`, accessible n'importe où dans une application JavaScript (uniquement dans un navigateur).

`document` a de très nombreuses propriétés, dont une propriété `children` : une collection, semblable à un tableau, contenant un seul élément : le noeud "racine" de l'arborescence, correspondant à la balise `html`.

Ce noeud a également une propriété `children`, contenant cette fois deux éléments : les noeuds correspondant aux balises `head` et `body`.

Chacun de ces noeuds a une propriété `children`... Ainsi de suite. On peut parcourir toute l'arborescence en parcourant les `children` de chaque noeud.

#### Récupérer un élément par son `id`

Pour récupérer un noeud particulier, l'objet `document` dispose d'une méthode `getElementById`, permettant de récupérer le noeud correspondant à une certaine balise, en fournissant à la méthode l'attribut `id` de la balise.

Partons d'un document HTML minimaliste :
```html
<!DOCTYPE html>
<html>
  <style>
    body {
      max-width: 300px;
      margin: 0 auto;
    }
  </style>
  <body>
    <h1>Exemple DOM</h1>
    <div id="content">
      <p id="par">Un paragraphe.</p>
      <button id="btn">Un bouton</button>
    </div>
    <script src="exemple-dom.js"></script>
  </body>
</html>
```

Pour accéder à la div identifiée par l'id `content`, on peut saisir ceci :

```javascript
const contentDiv = document.getElementById('content');
```

#### Accéder aux propriétés d'un noeud

Une fois qu'on a récupéré un noeud, par exemple via `getElementById`, on peut effectuer toutes sortes d'opérations, pour lire des propriétés de ce noeud, ou pour les modifier. Un exemple, avec la div récupérée précédemment :

```javascript
// Affiche dans la console tout le contenu HTML de la div
console.log(contentDiv.innerHTML);
// Modifie la couleur, le padding et la bordure,
// en manipulant le style du noeud (lui-même un objet)
contentDiv.style.color = '#378';
contentDiv.style.padding = '20px';
contentDiv.style.border = '1px solid #aef';
contentDiv.style.borderRadius = '5px';
```

Dans cet exemple, on a lu une propriété (`innerHTML)`, et on en a modifié une autre (`style`).

#### Réagir à des évènements sur un noeud

On va cette fois récupérer le noeud correspondant au bouton, et faire en sorte qu'une fonction soit exécutée lorsqu'on clique dessus :

```javascript
// Récupération du noeud correspondant au bouton #btn
const button = document.getElementById('btn');
// Récupération du noeud correspondant au paragraphe #par
const paragraph = document.getElementById('par');

// Variable "compteur", pour retenir le nombre de clics
let clickCount = 0;

// Fonction qu'on veut appeler quand le bouton est cliqué
const handleClick = event => {
  // Enregistre le clic en incrémentant le compteur
  clickCount += 1;
  // On récupère la propriété target de l'objet event,
  // qui ici est le bouton
  const target = event.target;
  paragraph.innerHTML = `${clickCount} clics sur #${target.id}`;
}

// On "attache" le gestionnaire d'évènement
// (l'équivalent du onClick dans une app React)
button.addEventListener('click', handleClick);
```
La méthode `addEventListener` est disponible sur chaque noeud, et prend obligatoirement ces deux arguments :

* une chaîne contenant le type d'évènement. On ne peut pas écrire tout et n'importe quoi, ces évènements sont standardisés, et il en existe de nombreux : `click`, `change`, `input`, `submit`, etc.
* La fonction à appeler lorsque l'évènement du type spécifié se produit sur *ce noeud*.

La fonction `handleClick` va être appelée à chaque clic sur le bouton. Elle va modifier la variable `clickCount`, et remplacer le contenu du paragraphe, en manipulant sa propriété `innerHTML`.

### Mise en oeuvre

Le point commun à ces trois exemples est de partir d'un fichier HTML, qui charge, en plus de notre application JS, la librairie Redux.

#### Exemple 1 - Compteur

Voici le *body* du HTML de l'application compteur (sans les scripts, insérés  à la toute fin).

```html
<h1>Compteur Redux</h1>
<div>
  <button id="sub10">-10</button>
  <button id="sub1">-1</button>
  <span id="count">0</span>
  <button id="add1">+1</button>
  <button id="add10">+10</button>
</div>
```

Décomposons ensuite l'application `redux-counter.js`.

On commence par créer les actions, en reprenant celles de l'exemple 1 de la section *Actions*&nbsp;:

```javascript
// Ajouter 1 au compteur
const add1 = {
  type: 'ADD',
  quantity: 1
};

// Ajouter 10 au compteur
const add10 = {
  type: 'ADD',
  quantity: 10
};

// Soustraire 1 au compteur
const sub1 = {
  type: 'SUBSTRACT',
  quantity: 1
};

// Soustraire 10 au compteur
const sub10 = {
  type: 'SUBSTRACT',
  quantity: 10
};
```

On reprend ensuite le reducer de l'exemple 1 de la section *Reducers* :

```javascript
const initialState = { count: 0 };

const counterReducer = (state = initialState, action) => {
  // On récupère le paramètre "quantity" de l'action
  const { quantity } = action;
  // On récupère la valeur du compteur dans le state
  const { count } = state;
  switch(action.type) {
    case 'ADD':
      return { count: count + quantity };
    case 'SUBSTRACT':
      return { count: count - quantity };
    // Si on reçoit une action d'un type non pris en charge
    // par le reducer, on renvoie le state tel quel
    default:
      return state;
  }
};
```

On initialise le store. Du fait d'avoir chargé la librairie Redux via une balise `<script>`, l'objet `Redux` est accessible globalement. C'est la façon originelle d'utiliser des librairies externes en JS, avant l'apparition de l'`import` avec ES6.

```javascript
// Destructure Redux pour récupérer createStore
const { createStore } = Redux;
// Initialise le store, en lui passant le reducer
// et un deuxième paramètre pour utiliser les Redux Tools
const store = createStore(
  counterReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__
  && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

Une remarque : ici, on ne passe pas le state initial comme argument à `createStore`. Un state initial sera néanmoins mis en place : lors du premier appel à `counterReducer`, `state` prendra la valeur stockée dans `initialState`.

Un deuxième argument est bien passé : il faut faire abstraction de sa complexité apparente, et juste savoir que cela permet d'utiliser l'extension Redux Dev Tools avec notre application. La fonction `createStore` est capable de distringuer ce 2ème argument d'une valeur initiale pour le state, absente ici.

Enfin, on termine en mettant en place le "câblage" de toute cette mécanique.

D'abord, on définit une fonction `displayCounter`, qui met à jour le contenu du noeud correspondant à la balise `<span id="count">`, en manipulant sa propriété `innerHTML`.

```javascript
const countSpan = document.getElementById('count');

// Affiche / rafraîchit l'affichage du compte
const displayCounter = () => {
  // Destructure le state récupéré via getState()
  const { count } = store.getState();
  countSpan.innerHTML = count;
}
```

Ensuite, on va placer des *event listeners* sur chacun des boutons.

Chacun des event listeners appelle `store.dispatch(action)`, en passant une action spécifique, en fonction du bouton qui a été cliqué. Rappelons que `dispatch` est la méthode fournie par le store pour appliquer une action.

```javascript
const sub10Btn = document.getElementById('sub10');
const sub1Btn = document.getElementById('sub1');
const add1Btn = document.getElementById('add1');
const add10Btn = document.getElementById('add10');

// Mise en place des gestionnaires d'évènements
sub10Btn.addEventListener(
  'click', e => store.dispatch(sub10)
);
sub1Btn.addEventListener(
  'click', e => store.dispatch(sub1)
);
add1Btn.addEventListener(
  'click', e => store.dispatch(add1)
);
add10Btn.addEventListener(
  'click', e => store.dispatch(add10)
);
```

Enfin, on va enregistrer un event listener auprès du store : c'est tout simplement la fonction `displayCounter`, qui sera donc appelée à chaque fois que le state sera mis à jour.

```javascript
store.subscribe(displayCounter);
```

#### Exemple 2 - Formulaire

Voici le *body* du HTML de l'application formulaire (sans les scripts, insérés  à la toute fin).

```html
<h1>Formulaire Redux</h1>
<form id="signup">
  <input type="text" placeholder="Entrez votre nom" required />
  <input type="email" placeholder="Entrez votre email" required />
  <input type="password" placeholder="Entrez votre mot de passe" required />
  <button type="submit" disabled>S'inscrire</button>
</div>
```

Décomposons ensuite l'application `redux-form.js`.

On commence par créer les actions. Tout en s'inspirant des actions de l'exemple 2, on va devoir rendre ces actions un peu plus génériques. Idéalement, on voudrait obtenir une action `{ type: 'SET', key: someKey, value: someValue }`, où `someKey` et `someValue` seraient variables.

Pour cela, rien de mieux qu'une fonction, qui va prendre deux paramètres : `key` et `value`, et nous renvoyer une action contenant les bonnes propriétés.

Cette fonction est un *créateur d'action*, ainsi que décrit dans la section [Action creators](https://redux.js.org/basics/actions#action-creators) de la documentation Redux.

```javascript
const createSetAction = (key, value) => {
  return {
    type: 'SET',
    key,    // syntaxe raccourcie, équivalente
    value   // à key: key et value: value
  };
};
```

On garde par contre l'action `reset` telle que vue précédemment :

```javascript
const resetAction = {
  type: 'RESET'
};
```

On reprend ensuite le reducer de l'exemple 2 de la section *Reducers* :

```javascript
const initialState = { name: '', email: '', password: '' };

const formReducer = (state = initialState, action) => {
  switch(action.type) {
    case 'SET':
      // On récupère la clé et la valeur dans l'action
      const { key, value } = action;
      // On renvoie une copie de l'ancien state (...state),
      // dans laquelle on "écrase" l'ancienne valeur associée
      // à la clé key
      return { ...state, [key]: value };
    case 'RESET':
      // On réinitialise tous les champs, avec une chaîne vide
      return initialState;
    default:
      return state;
  }
};
```

On initialise le store, cette fois avec `formReducer` comme argument, et toujours un 2ème argument permettant l'utilisation des Redux Dev Tools.

```javascript
// Destructure Redux pour récupérer createStore
const { createStore } = Redux;
// Initialise le store, en lui passant le reducer
const store = createStore(
  formReducer,
  window.__REDUX_DEVTOOLS_EXTENSION__
  && window.__REDUX_DEVTOOLS_EXTENSION__()
);
```

Ensuite, on va placer des *event listeners* sur chacun des inputs.

Chacun des event listeners appelle `store.dispatch(action)`, en passant une action spécifique, en fonction de l'input dans lequel on a saisi du texte.

Ici, on va se servir d'une méthode différente pour récupérer les inputs: `getElementsByTagName(tag)`, permettant de récupérer tous les noeuds correspondants à une certaine balise HTML (ici `input`).

Ce qu'on récupère (`inputs`) ressemble à un tableau, mais n'en est pas réellement un. En revanche, on peut le parcourir avec une boucle `for`, comme un tableau.

```javascript
const inputs = document.getElementsByTagName('input');

const inputListener = event => {
  // Destructure les attributs name et value
  // de l'élément ciblé par l'évènement
  const { name, value } = event.target;
  // Crée l'action de type: 'SET', en lui passant
  // les valeurs pour key et value
  const setAction = createSetAction(name, value);
  store.dispatch(setAction);
}

// On ajoute le même listener sur chaque input
for (let i = 0 ; i < inputs.length ; i++) {
  // Récupère un noeud dans la liste
  const input = inputs[i];
  input.addEventListener(
    'input', inputListener
  );
}
```

Pour terminer, on définit une fonction `checkForm`, qui utilise `store.getState()` pour vérifier que chaque champ est bien rempli. Si c'est le cas, la fonction enlève l'attribut `disabled` du bouton de soumission.

```javascript
const submitBtn = document.getElementById('submit');

// Rafraîchit l'affichage du formulaire
const checkForm = () => {
  // Destructure le state
  // pour récupérer les 3 chaînes
  const { name, email, password } = store.getState();
  // Initialise un booléen, true si
  // les 3 champs sont remplis
  const readyToSubmit = name && email && password;
  // En fonction du booléen, enlève ou remet disabled
  submitBtn.disabled = readyToSubmit ? '' : 'disabled';
}
```

Enfin, on va enregistrer `checkForm` comme event listener auprès du store :

```javascript
store.subscribe(checkForm);
```

#### Exemple 3 - Todo list

Un peu de contexte : on cherche à concevoir une application "todo-list" &mdash; un grand classique &mdash; destinée à être utilisée sur un mobile, sans avoir besoin de stocker les données sur un serveur (on peut se servir du local storage).

Une tâche de la todo-list est un objet contenant 3 propriétés :

* `id`, unique, permettant d'identifier une tâche
* `title`, l'intitulé de la tâche à réaliser
* `done`, l'état de la tâche : réalisée (`true`) ou non (`false`)

Pour créer une nouvelle tâche dans l'application, on a un simple formulaire, consistant en un unique champ input, permettant de renseigner l'attribut `title`.

Si l'on disposait d'un backend, on enverrait à ce dernier le `title` saisi. Il enregistrerait une nouvelle tâche dans la base de données : cette dernière fournirait automatiquement un `id`, incrémenté à chaque insertion d'une nouvelle tâche, et on pourrait faire en sorte que `done` soit positionné à `false` par défaut dans la base de données. Le backend répondrait alors avec l'objet complet, doté des 3 attributs.

Ici, puisque tout est géré côté client, il nous faut un moyen de donner un `id` unique à chaque tâche.


Notre state est maintenant un objet contenant une clé `list`, dont la valeur associée est un tableau, initialement vide. Le state initial est donc cet objet :

```javascript
{
  list: []
}
```

Par la suite, on va ajouter des objets à cette liste, représentant des tâches à accomplir.

Chaque tâche sera un objet ayant cette "forme":

```javascript
{
  id: 1,
  title: 'Do something',
  done: false
}
```

Pour modifier la liste, on va envoyer 3 types d'actions au store :

* une action pour *ajouter* (type: `ADD`) une tâche à la liste
* une action pour *enlever* (type: `REMOVE`) une tâche à la liste
* une action pour *inverser* (type: `TOGGLE`) l'état `done` (terminé ou non) de la tâche

Pour simplifier cet exemple, on ne se laisse pas la possibilité de modifier le titre d'une tâche une fois qu'elle a été créée.

Voici à quoi peuvent ressembler ces actions :

```javascript
const addTask1 = {
  type: 'ADD',
  task: {
    id: 1,
    title: 'Learn Redux'
  }
};

const addTask2 = {
  type: 'ADD',
  task: {
    id: 2,
    title: 'Learn Node.js'
  }
};

const addTask3 = {
  type: 'ADD',
  task: {
    id: 3,
    title: 'Learn MySQL'
  }
};

const toggleTask1 = {
  type: 'TOGGLE',
  taskId: 1
}

const removeTask3 = {
  type: 'REMOVE',
  taskId: 3
};
```

Voici les valeurs successives du state en appliquant ces actions, en partant de l'état initial :

<div class="stateChanges">
* Appliquer `addTask1`<br><span class="arrow">&rarr;</span> `{ { id: 1, title: 'Learn Redux', done: false } }`
* Appliquer `setEmail`<br><span class="arrow">&rarr;</span> `{ name: 'John Doe', email: 'johndoe@example.com', password: '' }`
* Appliquer `setPassword`<br><span class="arrow">&rarr;</span> `{ name: 'John Doe', email: 'johndoe@example.com', password: 'secure1234' }`
* Appliquer `reset`<br><span class="arrow">&rarr;</span> `{ name: '', email: '', password: '' }`
</div>
