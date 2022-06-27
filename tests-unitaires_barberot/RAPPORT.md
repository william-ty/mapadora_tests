# LG3 Ingénierie des tests - tests unitaires

# Introduction 

Il nous est apparu d'emblée que ce projet côté API se prêtait peu à des tests unitaires. En effet, il s'agit côté serveur d'une API node assez simple avec un ORM (Sequelize) pour gérer les transactions avec la BDD (Postgresql/PostGis), des classes (model) correspondantes, des contrôleurs/services permettant d'effectuer des opérations CRUD, et enfin des routes définissant des endpoints accessibles depuis les clients web et mobile. La plupart des tests effectués sur l'API ont été mis en place avec Postman (Newman), et réalisent en chaîne différents scénarios CRUD sur les objets via les routes définies. 
Nous avons cependant pu réaliser quelques tests unitaires côté client Web afin de tester l'affichage de composants ou encore des opérations (calcul de durée) faites à partir des données reçues de la base. 
L'outil Cypress nous a également permis de tester l'UI via quelques scénarios utilisateurs et fonctionnalités (création de compte, invitation de participants). Ce rapport propose de présenter quelques tests unitaires que nous avons développés dans le cadre de ce projet. 

# Tests réalisés 

## Factories

Nous avons pu nous rendre compte en mettant en place des seeders pour remplir notre base de données pour réaliser nos tests que c'était une tâche chronophage, et notamment lorsque les attributs des objets étaient eux-mêmes des objets. Nous avons effectué ce travail à la fin du projet, ce qui nous a obligé à définir tous les objets et les sous objets. Or, lorsque des tests unitaires sont réalisés au fur et à mesure, ces jeux de données sont construits petit à petit, ce qui permet de les réutiliser de manière transversale tant pour les tests unitaires que pour des tests de vue. Cela permet également de développer sans avoir de base de données dans un premier temps. 
C'est d'ailleurs ce que nous avons réalisé avec la "TravelFactory" côté web qui permettait de gérer l'affichage d'une liste de Cards (pour la liste des voyages) lorsque le web n'était pas encore connecté à l'API. Cette factory en a impliqué trois autres : stepFactory, qui comprend un objet Element (elementFactory) ainsi que des points (GeoJson). 
Côté client, une UserFactory a également été créée pour pouvoir tester les fonctionnalités de signin/signup dans le cadre des tests avec Cypress. 

## Difficultés rencontrées

Nous nous sommes rendus compte qu'il était difficile d'extraire de la logique à tester dans l'API. En effet, toutes les méthodes font appel à l'ORM. Les tests étaient donc dépendants de la connexion à la base et au lancement de l'API. Nous sommes donc arrivés à la conclusion qu'il fallait refactorer le code de manière à séparer la logique de l'interaction avec la base des opérations sur les objets. Cela nous a questionné du fait que la plupart des requêtes côté Web sont effectuées avec React Query qui permet une gestion efficace du cache et des requêtes. Ces requêtes sont localisées dans les composants qui en ont besoin, et ces composants font des requêtes en base. En voulant isoler le composant et le rendre agnostique, il a fallu relocaliser les requêtes dans le composant de niveau supérieur par exemple.  
Dans le cadre de ce rendu, nous proposont deux tests : l'un sur le calcul de la durée du voyage, et l'autre sur l'affichage conditionnel d'un composant React.

## Test du calcul de la durée du voyage (WEB)

Nous avons testé la méthode de calcul de la durée d'un voyage côté client, qui est en fait la somme des durées des étapes de ce voyage. L'API renvoit toutes les étapes d'un voyage, et le client se charge d'en faire la somme. Cette durée est ensuite affichée dans les Cards des voyages. Dans un premier temps, cette fonction était effectuée dans une pipe d'instructions suivant le fetch des étapes du voyage. Nous avons extrait ce morceau de code dans une fonction indépendante prenant en entrée une liste d'étapes. Ce refactoring nous a permis d'isoler la logique pour pouvoir la tester.  

## Test de l'affichage du composant "TravelCards" conditionnel

Nous avons choisi de tester ce composant qui est en fait une liste de Cards affichant les informations résumées du voyage. Son affichage est conditionnel en fonction de l'espace dans lequel il est généré : 
* la "Card publique" qui est celle affichée sur la page d'accueil
* la "Card privée" qui est celle affichée dans le dashboard utilisateur
* la "Card d'invitation" qui s'affiche dans la rubrique "notifications" du dashboard utilisateur. 

La mise en place de ce test a été assez complexe du fait de la nécessité d'un refactoring important. En effet, ce composant est composé de sous-composants. Chaque sous-composant effectuait ses requêtes en base pour pouvoir récupérer des données. L'architecture du composant est visible sur le schéma dans ce même dossier. 

### Refactoring 

Dans l'architecture initiale, les appels à l'API étaient effectués par les composants et les sous-composants, excepté la liste des voyages qui était envoyée en entrée du composant TravelCard puisque cette liste pouvait varier en fonction des besoins (voyages publics ou privés par ex.). 
La mise en place de tests sur l'affichage du composant ont été faits sur le composant UnitTravelCard, ce qui a permis de localiser les appels API au sein du composant TravelCards. Le ce composant fait ensuite redescendre les résultats des requêtes aux composants de niveaux inférieurs. Ainsi, le composant UnitTravelCard peut être testé avec des factories sans avoir à mocker les appels à la base. 

### Nature du test
Le test permet de vérifier le bon affichage du composant en fonction de l'enum en entrée (Public, Private ou DealWithInvitation). La librairie react-testing-library a été ajoutée à Jest, et nous avons ajouté un attribut data-testId aux bouttons pour pouvoir comparer le contenu. 
Trois tests unitaires correspondant aux trois affichages on été rédigés dans le dossier /ui. 

## Autres tests

D'autres tests effectués avec l'outil Cypress sont visibles dans le dossier des outils de tests. Ils concernent essentiellement des tests d'IHM. 

## Bilan

En résumé, nous retenons quelques enseignements des tests unitaires : 
* l'utilisation de la librairie de test React de Jest est intéressante du fait qu'elle ne nécessite pas de lancer l'API ou le client web pour pouvoir générer le composant et le tester. 
* ils ne sont intéressant que s'ils sont réalisés au fur et à mesure, voire qu'ils constituent le point de départ du développement d'une nouvelle fonctionnalité. 
* ils permettent de refactorer le code pour limiter le couplage afin de pouvoir réaliser des tests sur la logique métier de manière agnostique de la base ou de l'API.
* ils nécessitent des jeux de données qu'il est intéressant de construire au fur et à mesure afin de pouvoir les réuitiliser dans d'autres jeux de tests mais aussi dans des tests d'affichage de composants. 


   
    
