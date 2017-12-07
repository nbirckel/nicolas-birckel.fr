---
layout: post
title:  "Alimenter Highcharts depuis Google Spreadsheet"
date:   2016-06-07 9:22:14 +0200
categories: tools
cover :
---

J'ai dernièrement eu l'occasion de me frotter à la librairie [Highcharts][highcharts-url] pour la réalisation d'un graphique dynamique présentant un budget, dont le résultat est consultable [ici][budget-url].

Hightcharts, via son module `data.js`, peux lire le jeu de données du graphique depuis différents types de source : csv, fichiers json, mais également depuis Google Spreadsheet, ce qui peux s'averer utile pour la création d'un dashboard personnalisé, alimenté par différentes personnes par exemple.

## Créons un mini dashboard

On va créer pas à pas un mini dashboard de ce type :

![dashboard SF5][dashboard-img]{:class="lozad" dat-src="/assets/img/Dashboard.png"}


### Dans le Google Spreadsheet

il faut une feuille spécifique par graphique.  on va donc créer une feuille *league points* et une feuille *W/L*.

Pour les League points, on va faire un graphique de type ligne et aire. Dans la feuille *league points*, les données seront donc structurées comme ceci :

|League Points|
|:---:|
|1028|
|963|
|838|
|913|


les victoires/defaites seront affichées en camembert. Il va falloir structurer les données dans la feuille *W/L* de cette façon :

||data|
|---|:---:|
|win|65|
|lose|35|


Pour que votre script Hightcharts puisse lire vos données, il vous faudra publier votre Google Spreadsheet et récuperer la clé publique de celui-ci.

### Coté HTML

Dans le body de votre fichier, on crée les div qui vont servir à Hightcharts :

```html
<h1>Street Fighter 5 - Personal Ranked Dashboard</h1>
<div id="containers">
  <div id="container"></div>
  <div id="container1"></div>
</div>
```

### Coté JS 

Deux paramètres de data.js sont à renseigner pour la création des graphique par Highcharts : googleSpreadsheetKey et googleSpreadsheetWorksheet.

Le premier sert à accéder à votre Spreadsheet via sa clé, le second à indiquer quelle feuille est utilisée pour les données du graphique.

Pour notre exemple, on va avoir ce code pour le graphique en aire et ligne pour les League Points :

```
$('#container').highcharts({
	chart: {
    defaultSeriesType: 'areaspline', // on précise le type de graph
    backgroundColor: '#fff'
  },
    data: {
        googleSpreadsheetKey: '1nRDIGe1ihyO3ZZ0MdNnHaQkkf-N0I3iCs0o1Fm7Pr6E', // à remplacer par la clé de votre spreadsheet //
        googleSpreadsheetWorksheet: 1 // on indique qu'on utilise les données de la feuille 1 de votre spreadsheet //
    },
    title: {
      text: 'League Points evolution'

    },
    yAxis: {
      min: 1.5,
      title: {
        text: ''
      }
    },

      xAxis: {
      title: {
        text: ''
      },
      labels: {
        enabled: false
      }},
    credits: {
enabled: false

},
plotOptions: {
            series: {
                animation: {
                    duration: 2000
                }
            }
        }
});
```
Et celui-ci pour le camembert win/lose :

```
('#container1').highcharts({
   
  chart: {
    defaultSeriesType: 'pie',
    backgroundColor: '#fff'

  },
    data: {
        googleSpreadsheetKey: '1nRDIGe1ihyO3ZZ0MdNnHaQkkf-N0I3iCs0o1Fm7Pr6E',
        googleSpreadsheetWorksheet: 2 
    },
    },
    title: {
      text: 'Win / Lose'
    },
    yAxis: {
      min: 1.5,
      title: {
        text: ''
      }},
      xAxis: {
      title: {
        text: ''
      },
      labels: {
        enabled: false
      }},
    credits: {
enabled: false

},
plotOptions: {
            series: {
                animation: {
                    duration: 2000
                }
            }
        }

});
```

### Pour conclure 

Dans le cas de mon dashboard SF5, je rentrais mes données à la main, ne pouvant les récuperer de façon automatique depuis la PS4 ( Au passage, merci Capcom de ne pas fournir ce type de stats sur le web directement au joueurs :/).

Mais dans beaucoup de cas d'utilisation, vous allez pouvoir peupler de façon automatique votre Google Spreadsheet, et vous faire votre petit dashboard perso, surtout que Highcharts est gratuit pour les projets non commerciaux.

[budget-url]: http://www.culturecommunication.gouv.fr/Budget-2016
[highcharts-url]: http://www.highcharts.com/products/highcharts
[dashboard-img]: /assets/img/Dashboard.png
