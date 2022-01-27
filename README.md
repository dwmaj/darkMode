# DarkMode

Depuis 2019, on voit de plus en plus des applications, des sites ou encore des OS avec un mode sombre «darkMode». Si vous demandez autour de vous pourquoi utiliser un thème sombre, vous allez le plus souvent avoir les réponses suivante:

- ça permet de réduire la fatigue des yeux;
- on consomme moins d'énergie, donc plus de batterie pour nos téléphones (et c'est même mieux pour la planète);

Personnellement, je ne suis pas convaincu pour la fatigue de mes yeux, et même si ça permet aux écrans OLED (uniquement) de consommer moins d'énergie, j'ai toujours autant de problème de batterie sur mon téléphone… Je pense juste que certaines personnent préfèrent un mode sombre, ça fait «plus cool».

Si le sujet vous intéresse, voici quelques lectures:
- [A Brief History of “Dark Mode”—From the Matrix-like Displays of the Early ’80s to Today](https://eyeondesign.aiga.org/a-brief-history-of-dark-mode-from-the-matrix-like-displays-of-the-early-80s-to-today/)
- [Dark Mode — the fad that wasn’t](https://uxdesign.cc/dark-mode-the-fad-that-wasnt-97a12502e5fe)


## CSS

Pour gérer un darkMode, on va surtout travailler en CSS, et grâce à la caractéristique média `prefers-color-scheme` on va pouvoir détecter la préférence de l'utilisateur quant au thème à utiliser (sombre ou clair).

Sur MacOS on peut choisir dans les préférences système un thème clair, sombre ou automatique.
<img width="893" alt="image" src="https://user-images.githubusercontent.com/2959650/150374263-1fdcf6b2-cda1-4156-9282-a73a87fac6dd.png">


Grâce à `prefers-color-scheme` on va pouvoir identifier le thème préférentiel de l'utilisateur, et adapter le style CSS en conséquence. [Les explications se trouvent sur MD](https://developer.mozilla.org/fr/docs/Web/CSS/@media/prefers-color-scheme).

```CSS
body{
    background-color: #FAFAFA;
}

@media(prefers-color-scheme: dark){
    body{
        background-color: #070707;
    }
}
```
Voici un exemple en CSS.

## Variables CSS

Pour changer les couleurs d'un thème light à un thème dark (ou inversement), il va nous falloir modifier presque l'ensemble de nos couleurs. Pour éviter un labeur inutile, on va se servir de variables.

### Les propriétés personnalisées CSS

On les appelle souvent variables CSS, mais en réalité, on parle plutôt de *custom properties*. Il faut d'abord déclarer notre variable et ensuite l'utiliser.

Voici un exemple tout simple utilisant une couleur:

```CSS
element{
    --main-color: #BADA55;
}

element{
    color: var(--main-color);
}
```

Dans notre exemple, nous cherchons à utiliser des variables CSS pour modifier les couleurs de nos éléments. Nous allons avoir besoin de ces variables dans l'ensemble de notre CSS. On parle ici d'une variable globale. Pour définir une variable globale en CSS on va l'ajouter au `:root`, qui représente la racine de l'arbre représentant le document.

```CSS
:root{
    --main-color: #BADA55;
}

body{
    color: var(--main-color);
}
```

### On combine avec le media

Voici comment nous allons combiner l'utilisation des variables CSS avec le `prefers-color-scheme`:

```CSS
:root {
    --color-bg: #fff;
    --color-bg-offset: #121212;
    --color-text: #121212;
    --color-text-offset: #fff;
    --color-primary: #121212;
    --color-secondary: #fff;
}

@media(prefers-color-scheme: dark){
    :root {
        --color-bg: #121212;
        --color-bg-offset: #FFF;
        --color-text: #fff;
        --color-text-offset: #121212;
        --color-primary: #fff;
        --color-secondary: #121212;
    }
}
```

Nous pourrons utiliser ces variables pour construire notre thème et bénéficier automatiquement d'un changement de thème en fonction des préférences de l'utilisateur.

## Comment contrôler le thème avec un bouton?

Si vous souhaitez controler l'apparence de votre thème avec un bouton à la place du `prefers-color-scheme` ou en plus, il va nous falloir un petit peu de javascript.

### les attributs de données

Nous avons choisi de travailler avec les attribut de données pour modifier l'apparence de notre thème. On va donc ajouter un `data-theme`sur notre `body` directement dans l'HTML:

```HTML
<body data-theme="light">
```

Il nous faut ensuite un bouton pour l'intéraction:

```HTML
<button class="btn btn--theme">Day/Night</button>
```

Il nous faut ensuite du javascript pour changer la valeur de notre `data-theme=""`au click de la souris:

```JAVASCRIPT
const darkTheme = document.querySelector(".btn--theme");

darkTheme.addEventListener("click", function(){
    if(document.body.getAttribute("data-theme") === "dark"){
        
        document.body.setAttribute("data-theme", "light");

    } else {

        document.body.setAttribute("data-theme", "dark");

    } 
});
```

`getAttribute` nous permet de vérifier la valeur qui est présente sur notre body et `setAttribute` nous permet de définir la valeur light si le thème est en dark et vice-versa.

### Et en CSS?

On utilise le sélecteur d'attribut pour changer nos couleurs:

```CSS
[data-theme='light']{
    --color-bg: #fff;
    --color-bg-offset: #121212;
    --color-text: #121212;
    --color-text-offset: #fff;
    --color-primary: #121212;
    --color-secondary: #fff; 
}
[data-theme='dark']{
    --color-bg: #121212;
    --color-bg-offset: #FFF;
    --color-text: #fff;
    --color-text-offset: #121212;
    --color-primary: #fff;
    --color-secondary: #121212; 
}
```

### Un choix de thème persistant

Bien que notre exemple fonctionne parfaitement, chaque fois que nous allons changer de page, le choix de thème que nous avions fait grâce à notre bouton ne sera pas sauvegardé. Il faut donc trouver un moyen de stocker le choix de thème que fait l'utilisateur pour avoir un thème persistant entre les changements de pages (ou si l'utilisateur revient sur notre site après avoir fermé la page).

Nous allons utiliser le **localStorage** pour stocker la valeur du thème. Il faudra ensuite, à l'ouverture d'une page, vérifier si l'utilisateur à déjà stocker une valeur et si oui, afficher le bon thème.

Reprenons notre bout de code, et ajoutons dans notre **localStorage** la valeur du thème:

```JAVASCRIPT
const darkTheme = document.querySelector(".btn--theme");

darkTheme.addEventListener("click", function(){
    if(document.body.getAttribute("data-theme") === "dark"){
        
        document.body.setAttribute("data-theme", "light");
        localStorage.setItem("theme", "light");

    } else {

        document.body.setAttribute("data-theme", "dark");
        localStorage.setItem("theme", "dark");

    } 
});
```

`setItem` nous permet d'ajouter l'élement `theme` dans notre **localStorage** avec la valeur `light`.

<img width="1036" alt="image" src="https://user-images.githubusercontent.com/2959650/150374084-17ad726c-5abf-4507-86e0-ce08be9babf3.png">

Pour consulter votre **localStorage** dans Chrome, ouvrez l'onglet _Application > Storage > LocalStorage_, on peut voir dans notre exemple la valeur `dark`.

### Afficher le bon thème à l'ouverture d'une page

La première chose à faire est de vérifier s'il y a déjà une valeur dans le **localStorage**, s'il y a une valeur, c'est que l'utilisateur à déjà fait un choix:

```JAVASCRIPT
let theme = localStorage.getItem("theme");

if(theme === "dark"){

    document.body.setAttribute("data-theme", "dark");

} else if(theme === "light"){

    document.body.setAttribute("data-theme", "light");

}
```

#### Optimisation

Pour éviter de répéter nos `setAttribute` plusieurs fois, nous allons ajouter des fonction:

```JAVASCRIPT
function dark(){
    document.body.setAttribute("data-theme", "dark");
}

function light(){
    document.body.setAttribute("data-theme", "light");
}
```

Ce qui nous donne en reprenant l'exemple ci-dessus:

```JAVASCRIPT
let theme = localStorage.getItem("theme");

if(theme === "dark"){

    dark();

} else if(theme === "light"){

    light();

}
```


### Qu'est ce qu'on oublie?

Il est toujours très important de tester nos projets. 

Prenons un exemple:
- j'ai choisi un thème clair sur mon ordinateur;
- j'arrive sur un site;
- le js du site check mon **localStorage**, il est vide, c'est la première fois que je vais sur le site;
- le CSS check mon média `prefers-color-scheme`, il n'est pas en dark, les variables du `:root` sont utilisées;
- je vois un bouton pour un changement de thème, je click:
  -  mon javascript ajoute un `data-theme="dark"` sur le body;
  -  mon javascript `setItem("theme", "light")`dans mon **localStorage**;
  -  mon CSS change la valeur de mes variables;
  -  j'ai mon thème Dark, c'est la classe :blush:


Prenons un autre exemple:
- j'ai choisi un thème **DARK** sur mon ordinateur;
- j'arrive sur un site;
- le js du site check mon **localStorage**, il est vide, c'est la première fois que je vais sur le site;
- le CSS check mon média `prefers-color-scheme`, il est en dark, les variables dark sont utilisées;
- je vois un bouton pour un changement de thème sombre, je click:
  -  rien ne se passe? :sweat_smile:
  -  mon javascript ajoute un `data-theme="dark"` sur le body alors que je suis déjà en dark… :sweat:;
  -  mon javascript `setItem("theme", "dark")`dans mon **localStorage** alors que je suis déjà en dark… :disappointed_relieved:;
  -  mon CSS change la valeur de mes variables, mais ça ne sert à rien :weary:;
  -  C'est un fail… :sob:


Il faut ajouter une condition en plus, quand l'utilisateur clique sur le bouton, est-il déjà en darkMode? A-il un thème sombre sur sa machine? On va utiliser `matchMedia` en javascript, qui nous permet de vérifier la valeur css du `prefers-color-scheme`.

```JAVASCRIPT
const userDark = window.matchMedia("(prefers-color-scheme: dark)").matches;
```

Ensuite on vérifie sur le **localStorage** si il est déjà défini sur dark ou si le **localStorage** est vide mais que le userDark est en dark.

```JAVASCRIPT
let theme = localStorage.getItem("theme");
if((!theme && userDark) || (theme === "dark")){
    dark();
} else if(theme === "light"){
    light();
}
```

## Au final

Voici le code que nous utiliserons pour avoir un darkTheme persitant

```JAVASCRIPT
const darkTheme = document.querySelector(".btn--theme");

//Gérer le data-theme du body
darkTheme.addEventListener("click", function(){
    if(document.body.getAttribute("data-theme") === "dark"){
        light();
        localStorage.setItem("theme", "light");
    } else {
        dark();
        localStorage.setItem("theme", "dark");
    } 
});

//Est ce que l"utilisateur veut un theme dark?
const userDark = window.matchMedia("(prefers-color-scheme: dark)").matches;

//Est ce que l'utilisateur a déjà indiqué une préférence sur notre site?
let theme = localStorage.getItem("theme");
if((!theme && userDark) || (theme === "dark")){
    dark();
} else if(theme === "light"){
    light();
}

//function pour le dark
function dark(){
    document.body.setAttribute("data-theme", "dark");
}
//function pour le light
function light(){
    document.body.setAttribute("data-theme", "light");
}

```

# One more things…

Pourquoi juste un thème sombre? Pourquoi ne pas profiter de l'occasion pour être créatif :stuck_out_tongue_winking_eye:, [voici un petit exemple](https://mxb.dev/blog/color-theme-switcher/), et surtout, **regardez l'icône de peinture en haut à droite**!
