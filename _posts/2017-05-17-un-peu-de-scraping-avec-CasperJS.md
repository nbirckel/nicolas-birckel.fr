---
layout: post
title:  "Un peu de scraping avec CasperJS"
date:   2017-05-17 9:02:14 +0200
categories: javascript
cover : scraping-casperjs.png
excerpt_separator: <!--more-->
---

Cela fait quelques semaines que je me penche sur javascript pour automatiser certaines choses avec PhantomJS et CasperJS. Après avoir fait un script de  captures d'écran de site sous différents viewports, je me suis demandé comment scraper des données avec.

C'est ce qu'on va voir dans cet article.
<!--more-->

## Tour d'horizon

Premièrement, je tiens à préciser que le scraping, c'est mal ! **Privilégiez toujours les moyens officiels d'obtenir les données que vous convoitez en passant par des flux RSS, des API, des accès payants ou autres.** 

Dans l'exemple que je vais traiter, on va récupérer les films à l'affiche et les horaires de séances du Kinepolis du coin, qui sont disponibles à cette adresse : [https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui](https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui).

On va ensuite parcourir les données scrapées pour les afficher dans le terminal.


## Un coup d'oeil dans le DOM de la page Aujourd'hui du Kinépolis Nancy :

Regardons dans l'inspecteur comment est structurée la page [https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui](https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui).

![la-div-de-chaque-fiche](/assets/img/kine-laboite.png){:class="lozad" data-src="/assets/img/kine-laboite.png"}

Chaque "fiche" de film est contenue dans une div avec la classe `movie-row-wrapper`. 

On va donc pouvoir selectionner toutes les occurences de cette classe dans la page puis parcourir chacune d'entre elle pour récuperer les informations qui nous interessent, à savoir le titre du film et les horaires.

Continuons d'inspecter le DOM pour trouver comment cibler le titre du film et les horaires.

![le-titre-du-film](/assets/img/kine-letitre.png){:class="lozad" data-src="/assets/img/kine-letitre.png"}

Les titres des films n'en sont pas réellement : chaque titre est un lien enfant d'un paragraphe avec la classe `movie-overview-title`.
On va donc pouvoir s'appuyer sur le selecteur `.movie-overview-title a` pour récuperer le contenu de celui-ci.

Et pour les horaires ?

![les-horaires-du-film](/assets/img/kine-lhoraire.png){:class="lozad" data-src="/assets/img/kine-lhoraire.png)"}

Toujours en parcourant le DOM, on repère rapidement que les horaires des différentes séances sur trouvent dans des balises span ayant pour classe `t-seg`.
Chaque horaire est contenu dans une balise a pointant vers le système de réservation de Kinépolis.

On garde donc bien au chaud le selecteur `.t-seg a` pour la suite des opérations.

### Structurons nos données

Concrètement, je veux que mon script casperJS, après avoir parcouru la page et récuperé les données qui m'interessent, me retourne une liste de films associés à leurs horaires de séances respectifs.

Du coup, chaque film va être un objet ayant deux propriétés : un titre et des horaires. 

Le titre va être récuperé en scrapant le contenu de la balise ciblé par `.movie-overview-title a`. Quant aux horaires, il peut y en avoir plusieurs, en fonction du moment de la journée où j'exécute le script. Ceux-ci vont donc être stockés dans un Array.

Je veux retourner dans mon terminal le contenu de chaque objet film à la suite l'un de l'autre. Je vais donc stocker tous ces objets film dans un Array pour ensuite le parcourir avec une boucle.

## Rentrons dans le code.

Commençons à écrire notre script. Pour rappel, il s'appuie sur [CasperJs](http://casperjs.org/) qui lui même s'appuie sur [PhantomJS](http://phantomjs.org/). Sans eux, tout cela ne marche pas. Installez les avant si vous voulez tester :)

### On déclare quelques variables :

```javascript
var casper = require('casper').create();
var films;
```

### Puis une fonction pour parcourir le DOM et récupérer les éléments qui nous interessent :

```javascript
function getFilms() {
    var films = document.querySelectorAll('.movie-row-wrapper');
    var listFilms = [];
    for (var i=0; i <films.length;i++){
    	var film = {};
    	var f= films[i].querySelector('.movie-overview-title a');
    	var horaires =[];
    	var h = films[i].querySelectorAll('.t-seg a');
    		for (var j=0; j<h.length;j++) {
    			var horaire = h[j].innerHTML;
    			horaires.push(horaire);
    		};
    	film['title'] = f.innerHTML;
    	film['horaire'] = horaires;
    	listFilms.push(film);
    };
   return listFilms;
};
```

Concrétement, cette fonction va sélectionner et stocker dans un Array films toutes les "fiches" de films de la page. 

On déclare ensuite un nouvel Array ListFilms vide que l'on va remplir plus tard.

On parcourt ensuite notre Array films. A chaque itération de la boucle :

 * On crée notre objet film, 
 * On récupère le lien ciblé par `.movie-overview-title a` dans l'occurence en cours de `.movie-row-wrapper`,
 * On crée un Array pour stocker les horaires,
 * On sélectionne toutes les occurences de `.t-seg a` existantes dans le `.movie-row-wrapper` qu'on parcourt et on les stock dans un Array,
 * On parcourt toutes les instances de `.t-seg a` trouvées pour extraire sont contenu que l'on insère dans l'Array stockant les horaires,
 * On enregistre les propriétés titre et horaire de notre objet film,
 * Et pour finir, n insère l'objet film dans notre Array Listfilms.


Une fois toutes les occurences de fiche de films parcourues, la fonction retourne notre array Listfilms rempli.


### Ouvrons l'url du site avec CasperJS et faisons lui évaluer notre fonction :

```javascript
casper.start('https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui');

casper.then(function () {
    films = this.evaluate(getFilms);
});
```

### Parcourons ensuite l'array films pour afficher dans notre terminal le listing des films et leurs horaires

```javascript
casper.run(function () {
	casper.echo("Les prochaines séances :", 'ERROR');
    for(var i in films) {
        casper.echo(films[i].title , 'PARAMETER');
        var horaires = films[i].horaire.join(' - ');
        casper.echo(horaires);
    }
    casper.done();
});
```
J'avais initialement utilisé `console.log()` pour écrire dans mon terminal. Je suis passé à `capser.echo()` car cela permet de passer un paramètre supplémentaire pour mettre un peu de couleur dans notre liste de films.

## Le script complet 

```javascript
var casper = require('casper').create();
var films;

function getFilms() {
    var films = document.querySelectorAll('.movie-row-wrapper');
    var listFilms = [];
    for (var i=0; i <films.length;i++){
    	var film = {};
    	var f= films[i].querySelector('.movie-overview-title a');
    	var horaires =[];
    	var h = films[i].querySelectorAll('.t-seg a');
    		for (var j=0; j<h.length;j++) {
    			var horaire = h[j].innerHTML;
    			horaires.push(horaire);
    		};
    	film['title'] = f.innerHTML;
    	film['horaire'] = horaires;
    	listFilms.push(film);
    };
   return listFilms;
}

casper.start('https://kinepolis.fr/cinemas/kinepolis-nancy/aujourdhui');

casper.then(function () {
    films = this.evaluate(getFilms);
});

casper.run(function () {
	casper.echo("Les prochaines séances :", 'ERROR');
    for(var i in films) {
        casper.echo(films[i].title , 'PARAMETER');
        var horaires = films[i].horaire.join(' - ');
        casper.echo(horaires);
    }
    casper.done();
});

```
Pour le lancer, il suffit de taper dans le terminal : `casperjs le-nom-du-script.js` 

![resultat-script](/assets/img/resultat-scraping.png){:class="lozad" data-src="/assets/img/resultat-scraping.png"}

## Pour conclure

Ce script est loin d'être parfait, mais il marche et fait ce que j'en attendais. Il pourrait également être amélioré, en indiquant par exemple si le film est en 3D ou non, ou pour n'afficher que les horaires d'un film précis passé en paramètre lors du lancement du script.

Le script est dispo sur github au besoin : [https://github.com/nbirckel/prochaines-seances](https://github.com/nbirckel/prochaines-seances).
