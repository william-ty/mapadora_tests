# Tests IHM avec Cypress

## Introduction

Suite aux différents cours portant sur les outils de tests, nous avons mis en place quelques tests réalisés avec l'outil Cypess. Ce dernier permet de tester l'affichage d'éléments via des selecteurs particuliers, mais permet aussi de faire des tests unitaires ou des tests d'API via des requêtes (comme Postman). Nous nous sommes limités à des tests d'IHM, bien que nous ayons écrit quelques requêtes comme par exemple celle qui permet de se logguer pour ne pas passer systématiquement par l'interface. 

## Tests mis en place

Nous avons testé trois fonctionnalités, bien que l'application présente de nombreux éléments à tester : la création de compte, le login d'un utilisateur et l'ajout d'un participant à un voyage. 
La progression des cours nous a amenés à affiner les patterns des tests, ce qui nous a également permis de faire évoluer l'oganisation des fichiers. Le refactoring des tests ayant pris un certain temps, nous nous sommes concentrés sur ces trois fonctionnalités en tentant d'appliquer les bonnes pratiques mentionnées notamment dans la documentation de Cypress. 

### Prérequis 

Nous avons créé plusieurs factories ainsi que des seeders pour remplir la base de données avec des éléments. En effet, les différents tests nécessitaient d'avoir des données en base ou des factories d'objet (notamment dans le cadre du page-object pattern): 
* pour le test de la création de compte : une factory qui créé un utilisateurs (voir dans le dossier Model pour les objets)
* pour le test du login : un objet Credentials (login/password) ainsi que l'utilisateur existant en base. 
* pour le test de l'ajout d'un participant ne possédant pas de compte sur l'application : deux utilisateurs et les Credentials associés, l'un ayant déjà un compte et l'aute non. Le premier utilisateur a un déjà créé un voyage qui apparaît dans sa liste de voyages. 

### Pattern
Les trois fonctionnalités testées font appel à plusieurs écrans. Chaque écran correspond à une classe page-object. On trouve dans le dossier page-objects du projet : 
* HomePage : c'est la racine du site
* ParticipantPage : c'est la page à l'intérieur du dashboard utilisateur qui permet de consulter les participants à un voyage et d'en inviter des nouveaux. 
* ProfilEditionPage : c'est la page qui permet d'afficher les informations de compte de l'utilisateur 
* SigninPage : la page de connexion 
* SignupPage : la page de création de compte
Chaque page comprend les méthode et un test qui permet de confirmer l'affichage attendu. En parallèle de cela, des selecteurs particuliers _[data-cy = ""]_ ont été définis dans les balises des éléments à tester. Cela facilite la sélection de l'objet pour pouvoir ensuite écrire les assertions. 

Le dossier _api_ contient les appels à l'API avec la fonction Cypress cy.request qui permet de faire une requête sans passer par l'interface. Ces appels sont utilisés lors du test d'ajout de participant. 

Le dossier _integration_ comprend les tests des fonctionnalités signin, signup et addParticipant faisant appel aux PageObject. 

### Scénarios de test
.Signup
L'utilisateur entre sur la page d'accueil du site et clique sur "Créer un compte". Il entre ensuite ses informations. La redirection sur la page Signin indique que le compte a bien été créé. 

.Signin
Un utilisateur déjà enregistré en base tente de se connecter. Il clique sur "Me Connecter", entre ses informations. Une redirection sur le dashboard utilisateur confirme le test. 

.addParticipant
Un utilisateur possédant déjà un compte et ayant déjà créé un voyage est connecté à son compte (requête POST via cy.request). Il accède à la page des participants depuis le dashboard. 
Dans un premier temps, il invite un participant sans adresse mail, ce qui ajoute directement le nom de la personne dans la liste des participants. Cet ajout n'a aucune autre implication. L'apparition du nom du participant dans la liste "Participants ajoutés" confirme cet ajout. Le participant est ensuite supprimé de la liste. Sa disparition de la liste confirme le test. 
L'administrateur du voyage ajoute ensuite un participant avec une adresse mail. Cette personne n'a pas encore de compte sur l'application. De la même manière que précédemment, il rentre les informations nom + email dans le formualaire et clique sur le boutton ajouter. L'apparition du nom et du mail de cette personne dans "Participants en attente" confirme qu'une invitation a été envoyée. Ce participant se déconnecte de l'application. 
La personne ajoutée créé un compte (via une requête POST comme précédemment) avec le même email que celui renseigné par l'administrateur du voyage. Le nouvel utilisateur arrive ensuite sur son dashboard, et doit constater que l'onglet notification a un badge avec les nombre de notifications arrivées (un petit 1 ici). Cet élément confirme une première partie de test. L'utiliateur clique sur cet onglet notification et doit constater qu'il est invité à rejoindre le voyage de l'administrateur. Il clique sur accepter l'invitation et le voyage s'ajoute à sa liste de voyages. On rebascule ensuite sur le compte de l'administrateur du système pour constater dans la page participants que l'utilsiateur invité est passé dans la catégorie des participants au voyage. 

## Difficultés et retours d'expérience

Nous avons pu nous rendre compte que les tests d'IHM sont assez chronophages et sont très dépendants de la base de données, le lancement de l'API et du client web, ainsi que des performances du poste (temps de chargement des pages etc.). Cet outil permet cependant de faire des tests de scénarios utilisateurs complet et de rendre compte de la manière dont s'affichent les composants. L'un des avantages de Cypress est de pouvoir proposer en un seul outil plusieurs catégories de tests : interface, api, unitaires, etc. et de les intégrer à la pipeline de déploiement afin de les automatiser. 
