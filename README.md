# bofself-poc-log

Ce document est une recherche sur l'amélioration des logs pour les projets informatiques.

La maintenance des projets informatiques nécessite la mise en place de logs, des informations textuelles ou 'traces' qui seront disponibles dans des fichiers, des bases de données, de façon persistantes ou éphémères. Ces données textuelles sont archivées pour consultation, généralement compressées, et sont accessibles à la production, aux développeurs, mais aussi à la chefferie de projet, et à la maîtrise d'ouvrage. Nous verrons au cours de ce document comment les logs peuvent être utiles à ces derniers. 

Il existe une telle diversité de projets informatiques qu'il semble impossible de proposer des règles universelles de gestion des logs. Peut-être bien... Pourtant, une fois qu'on a compris à qui les logs sont destinés et qu'est-ce qu'ils en attendent, des principes assez simples sont facilement identifiables.

Dans ce document, nous définirons ce qu'est un 'log', les besoins du point de vue de différents acteurs, et enfin nous proposerons une ou plusieurs méthodes (parmis tant d'autres) de mise en place de log.

## Définition des logs
Lors d'un voyage, on consigne parfois des faits, on décrit les étapes, on expose ce qu'on a ressenti. On accompagne souvent nos impressions de photos, de cartes ou de vidéos. L'avènement des raisons sociaux a rendu populaire le partage de 'biographies' en ligne, parfois jusqu'à l'agacement de nos destinataires tellement certains de nos 'tweets' sont futiles. Dans d'autres cas, tellement de photos et de vidéos ont été prises que rares sont ceux qui trouve le temps d'organiser le récit de leur voyage. On apprécie alors des services comme Flickr(R) qui organisent automatiquement ces médias et simplifie la navigation.

D'une certaine façon, l'analogie du voyage illustre la difficulté de mettre en place des logs de qualité. Un log applicatif est en quelque sorte une autobiographie du déroulement d'une application. La lecture (ou l'analyse) de ce récit doit permettre à un acteur de comprendre ce qui se passe dans l'application.

Que doit-on mettre dans le récit d'un voyage ? Tout dépend de l'intention du voyageur *ET* du public visé. On en met pas les mêmes informations quand le récit est pour soi-même, notre famille, nos amis, ou un public d'inconnnu sur Internet. Il en est de même pour les logs. Le récit applicatif doit correspondre au besoin du public visé. Les 'purées' (pour ne pas dire diarrhées...) de logs sont généralement illisibles et inexploitables pour TOUS les acteurs. Ce contenu indigeste et puant est le drame des applicatifs produits par les sociétés de service. Les logs sont souvent générés automatiquement par les frameworks qui sont rarement des modèles de concision. Encore pire, le guide du bon petit logger n'est généralement par fourni au développeur prestataire.

La plupart des gestionnaires de log propose par défaut les niveaux de logs suivants :


| Niveau   | Destination directe    | Pertinence |
| -------- | --------               | --------   |
| TRACE    | En développement       | Les traces sont utiles en développement uniquement. Il permet de suivre le code lors de séance de débuggage, et sont généralement TRES verbeux.       |
| DEBUG    | Développeur            | Les logs de niveau debug contiennent le suivi pas à pas d'un algorithme avec les données. Ces logs doivent être pertinents et écrits en pensant à la maintenance. Le niveau de verbosité est relatif.    |
| INFO     | Developpeur/Autorité   | Les logs de niveau info constitue note récit applicatif. La lecture du log info devrait suffire à savoir ce qui s'est passé dans l 'aplication       |
| WARN     | Exploitant             | Les warnings indiquent qu'un traitement a pu se faire, mais qu'il aurait pu être optimisé. **Les exploitants pensent qu'ils doivent intervenir dès qu'ils rencontrent un log de ce type**.        |
| ERROR    | Exploitant/Développeur | Les erreurs indiquent qu'un traitement n'a pas pu se terminer correctement. Il s'agit toujours d'erreur technique (un fichier non trouvé, une base de données inaccessible, problème de droit, etc...). _Les erreurs fonctionnels sont de niveau INFO_. **L'exploitant intervient rapidement quand il voit passer des erreurs**. Les erreurs contiennent généralement la pile d'appel, ce qui est très utile pour les développeurs.    |
| FATAL    | Exploitant             | L'erreur fatale indique que l'application n'a pas pu démarrer ou a du s'arrêter brusquement, suite à une erreur irrécupérable. **L'exploitant intervient immédiatement quand il a une erreur FATAL.**      |

Par un mécanisme de fitrage, on fixe généralement les logs au niveau 'INFO' lorsque l'application est mise en production. On passe au niveau 'DEBUG' quand un comportement inhabituel est détecté et que les logs 'INFO' ne suffisent pas.

Des librairies de logs fournissent les outils pour créer ses propres niveaux de log, par exemple NOTICE ou COMMENT. C'est le cas de log4j2.

### Exemple de logs provenant d'un projet de gestion en java : 
Voici des logs tels qu'extraites d'un projet. Juste après, nous en tirerons quelques leçons :

#### Persistence 
```text
YYYY-MM-DD_HH:mm:ss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] PersistenceUserImpl - lireUserById(id:2) 
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] SQLEXP - select * from user where id = 2 
YYYY-MM-DD_HH:mm:ss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] DATA - {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] STAT - POOL:2ms PREP:3ms REQT:5ms NETW:10ms TOTL:20ms
...
```

#### User story
```text
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] action=PROFIL L'utilisateur affiche son profil 
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0002] action=PROFIL [params:id=2,name=Bento,...] L'utilisateur modifie son profil 
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0003] action=ACCUEIL L'utilisateur affiche la page d'accueil
...
```

#### Background process
```text
YYYY-MM-DD_HH:mm:ss.SSS INFO [CACHE|RG0000001] Démarrage de la mise à jour du cache {{{
YYYY-MM-DD_HH:mm:ss.SSS DEBUG [CACHE|RG0000001] Town [hash=908798678,id=1,label=Paris,cp=...]
YYYY-MM-DD_HH:mm:ss.SSS DEBUG [CACHE|RG0000001] Town [hash=908798678,id=2,label=Créteil,cp=...]
...
YYYY-MM-DD_HH:mm:ss.SSS INFO [CACHE|RG0000001] }}} Fin de la mise à jour du cache : 15565 ms
```

#### Accès Web
```text
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] action=PROFIL jsp>2ms metier>25ms total>28ms
...
```

#### J'ai des logs, et alors ?
Que penser des logs ci-dessus ? Il s'agit de ma première tentative d'organiser les logs. En effet, dans notre équipe, aucune règle de log n'existait. Il fallait bien commencer par quelque chose. 

Nous avons tenté de classer, d'organiser et de créer des logs du point de vue utilisateur. Le but recherché était de pouvoir lire les logs comme un roman. Pourtant il faut le reconnaître, ces logs sont illisibles... Beaucoup d'informations sont redondantes et il est difficile de retrouver ce qui est réellement nécessaire pour la maintenance. Par exemple, certaines phrases en langage naturel sont superflues.

Par ailleurs, les entêtes censées permettre d'associer les logs répartis sur plusieurs fichiers sont trop longues.

L'application en question étant très sollicitée, la quantité de logs présents rend la lecture encore plus compliquée. Il n'est pas possible de réduire le nombre de logs, mais il devrait être possible d'en faciliter la lecture.

Enfin, une ligne de log ne peut pas être facilement analysé. Par exemple, nous avons voulu extraire le temps de réponse et le nombre de connexion distinctes sur une période données. Pour cela, il a fallu reformater manuellement les logs d'Accès Web dans un format proche du CSV, puis l'importer dans Excel. 

Je travaille avec [gitea](https://gitea.io/) depuis quelques temps, et je dois avouer que les logs produits par cette application sont exemplaires. Il y a ni trop ni trop peu d'informations. Il est facile d'associer les logs répartis sur 3 fichiers (exceptés pour les logs SQL), et aucune information n'est redondante. Cela faisait quelques temps que nous voulions fixer une norme de 'logging', et je crois bien avoir trouvé une excellente source d'inspiration.

Nous complétons les logs par [javamelody](https://github.com/javamelody/javamelody/wiki) comme outils de monitoring et d'analyse. Nous devrons donc veiller à ne pas reproduire dans les logs ce qui revient à javamelody.

## Besoins du _développeur_
IMO, le développeur a besoin de logs sur 3 périodes différentes de la vie du projet :

- En cours de développement
- Pour le déploiement (recette ou production)
- Pour le traitement des anomalies en production

### Développement
Le développeur est responsable de l'écriture des logs. Comme indiqué plus haut, il écrit :
- Des logs de niveau TRACE (ou traces) pendant l'écriture de son code. Ces logs sont l'équivalent des brouillons que l'on fait avant de rédiger un document.

``` java
log.trace("Etape 1 : Suppression de l'entrée {} d'id {}", entry, id);
```
Pour ce niveau de log, il est préférable de choisir une librairie qui ne traite les logs que si le niveau TRACE est activé, sous peine de plomber les performances. Pour les librairies anciennes comme log4j 1.2, il vaut mieux faire :

``` java
if (log.isTraceEnabled()) {
	log.trace("Etape 1 : Suppression de l'entrée " + entry + " d'id " + id);
}
```
En effet, les traces sont généralement très nombreuses, ils peuvent sérieusement consommer du temps CPU et amoindrir les performances de l'application.

- Des logs DEBUG pour analyser ses algorithmes. Ces logs *DOIVENT* être soignés. Ils sont l'équivalent des notes contenues dans un carnet de voyage.

``` java
log.debug("Lecture du fichier {} terminée. Taille:{}", fileName, size);
```
De même, les logs debug peuvent être assez nombreuses. L'utilisation de `log.isDebugEnabled()` ou d'une librairie qui teste au préalable si le niveau DEBUG est actif préservera les performances.

- Des logs INFO pour présenter le résultat d'un traitement. C'est eux le 'récit public' de notre voyage.

``` java
log.info("Nouvel utilisateur soumis : {}", newUser);
// traitement
log.info("Soumission utilisateur : {}", result);
```

La lecture des logs infos permet de savoir précisément les actions utilisateur et la réponse de l'application. Sa lecture doit permettre de suivre un utilisateur. Les erreurs fonctionnelles sont de niveau INFO. Il convient d'éviter les redondances, et d'être concis. Une information qui peut être déduite n'a pas besoin d'être détaillée textuellement. 

- Des logs WARN pour attirer la vigilance de la production. La présence de WARN interpelle forcément la production. Ces logs doivent donc être soignés et ciblés.
  
```java
log.warn("Encodage non spécifié, l'encodage par défaut sélectionné {}", config.getEncoding());
```
On a parfois tendance à créer des logs de niveau WARN lorsqu'un traitement rencontre une 'erreur' qu'il est capable de gérer. Ce n'est pas un warning. Une erreur fonctionnelle prévisible devrait être de niveau INFO . Comment bien choisir ce qui doit être de niveau WARN ? Examinons les cas suivants : 

1. Avertir de la non présence d'un fichier optionnel
2. Averissements fonctionnels
3. Données issues de la BD incomplètes
4. Indiquer des dégradations de perfomances

Parmis ces 4 situations, lequelles sont des logs de niveau WARN ? La 1, 3 et la 4. En effet, la 2 n'intéresse pas l'exploitation et doit être de niveau INFO (il s'agit du résultat d'un traitement). 

- Des logs ERROR pour avertir la production. En plus d'un message, ils doivent comporter des stacktraces pour faciliter la compréhension de l'erreur. C'est généralement la production qui repère les erreurs. Elle peut toutetefois donner un accès aux logs aux développeurs.

``` java
log.error("Lecture du fichier du résultat {} impossible, exception, viewFileName);
```
Les erreurs sont principalement de nature technique, et concerne en majorité des exceptions de type Runtime. Une erreur est le résultat d'une impossibilité de terminer un traitement par suite d'un imprévu technique. Par exemple :

1. Erreur d'accès à la base de données
2. NullPointerException, ou plus généralement RuntimeException
3. Contrat d'appel non respecté d'une méthode ou d'un service
4. Données incorrectes.

Nous avons choisi de faire produire les messages des erreurs les plus connus par le socle applicatif. Les cas d'erreurs étant connus, une solution précuite peut être proposée. 

- Des logs FATAL pour indiquer des éléments manquants qui empêche le démarrage ou le bon fonctionnement de l'application.

``` java
log.fatal("Impossible de trouver le fichier de configuraiton {}", configFileName);
```

Un log FATAL précède généralement un `System.exit(errorCode);`

### Déploiement
Le développeur a besoin des logs de niveau FATAL, et des logs de DEBUG généraux qui sont envoyés sur la sortie standard.

### Anomalies de la production
En production, c'est généralement les stacktraces des logs de niveaux WARN et ERROR qui nous intéressent, ainsi que les logs de niveau INFO. Les logs DEBUG sont activés seulement si le développeur ne s'y retrouve pas.

## Besoins de la _production_
En production, les seuls logs utiles sont de niveau WARN, ERROR et FATAL.

### Déploiement
Lors du déploiement sur la recette et/ou la production, l'application devrait émettre de logs de niveau FATAL qui indiquent que des règles de déploiement n'ont pas été respectées. Ces logs devraient contenir également une proposition de solution.

### Gestion des erreurs
La production utilise toujours un outils de monitoring, comme Racvision dans l'éducation nationale, pour s'assurer de l'état de santé d'une application. Elle a également des outils qui analysent les logs et mettent en évidence ceux au-dessus du niveau WARN. (le quatuor RELK (=Redis, Elasticsearch, Logstash, Kibana) par exemple).

L'application devrait :
- Indiquer dans Racvision (ou tout outil similaire) tous les points d'écoute susceptibles de générer des erreurs (base de données, consommation mémoire, etc..)
- Indiquer ces mêmes messages du niveau approprié dans les logs pour établir un historique.

## Besoins du _chef de projet_
IMO, le chef de projet a besoin de savoir si l'application répond bien aux actions utilisateurs. Il a besoin d'un état de santé, et de statistiques sur les performances de l'application. Il a besoin de savoir la fréquence des erreurs rencontrées. Ces données pourront être transmises à sa hiérarchie.
 
Le temps de réponse instannée, la consommation mémoire, soit l'état de santé générale de l'application peuvent généralement être fournis par des outils comme JavaMelody. Toutefois certaines données auraient intérêt à se retrouver dans les logs, comme le temps de réponse, la consommation mémoire, etc, données qui pourront ête interrogées plus tard pour établir un historique.

Afin de faciliter l'extraction de ces données, celles-ci ont intérêts à avoir un format facile à traiter. Par exemple le format UNL.
Reprenons les logs 'Accès Web' que nous avons vus plus haut :

```java
YYYY-MM-DD HH:mm:ss.SSS|INFO|IP|10.0.0.1,192.168.0.217|RG|0000001|SH|username|000000000000|RS|0001|PROFIL|VW|2|ms|BS|25|ms|TTL|28|ms
```

Soit dit en passant, l'utilisation d'outil comme Elasticsearch impose d'avoir une forme de log 'interrogeable'. Ce qui précède est donc une bonne pratique et simplifiera grandement les règles d'extraction de données.

## Besoins de la _MOA_ et des _autorités_
Les applications de gestion ont parfois une valeur légale. Il nous faut rendre des comptes à des syndicats, des policiers et aux politiques. Les besoins sont généralement statistiques, mais peuvent concerner des déroulements complets de comportement utilisateurs, notamment lors de litiges ou de plaintes.
 
### Performance
La MOA est intéressée par des statistiques métiers, comme cela :

* Nombre de connexion distinctes sur une période donnée
* Nombre de connexion globale 
* Temps de réponse
* Nombre de dossiers traités sur une période
* ...

En loggant correctement le comportement métier, il est possible de fournir très simplement ce genre de statistique.

### Comportement utilisateur
Les autorités ont parfois besoin de savoir :

* L'adresse IP réelle (derrière un proxy par exemple)
* Les opérations effectuées par un utilisateur donné
* Les données manipulées

Ainsi, les logs de niveau INFO doivent contenir ces informations, de façon à pouvoir être lu comme un roman. Les données insérées ou mises à jour doivent être loggées de façon à pouvoir défaire des opérations jugées malveillantes si nécessaire.

## Autres considérations
### Catégories de logs
Les applications n-tiers structurés à l'aide de pattern ont généralement (plus ou moins) les couches suivantes :

0. L'utilisateur (navigateur ou desktop)
1. La vue
2. La commande
3. Le contrôleur ou le client
4. Une couche de transport (optionnelle)
4. Le service
5. La persistence
6. Les DAO
7. La connexion à la base de données
8. D'eventuels périphériques (imprimantes, périphériques connectés, etc...), ou traitements asynchrones.

Chaque couche jourera un rôle différent dans la production de logs. 

En fonction des besoins détaillés plus haut dans le document, il en ressort les catégories suivantes :
- Protocole HTTP ou autre (clic sur la Vue)
- Commande utilisateur (les opérations demandées, et les données envoyées ou reçus).
- La couche transport si présente (la couche transport sert à implémenter les protocoles d'appels RPC)
- Les services appelés
- La persistence (SQL, données modifiées).
- Un fichier par périphérique, ou par type de traitement asynchrone
- Les erreurs

Une ou plusieurs catégories peut être envoyé dans un même fichier, ou un fichier distinct :
- http ou view
- userstory
- transport
- business
- persistence (catégories : sqlstat, sqldata, sqlexp)
- error
- cache, device-xxx, resultat, ...

Les données n'ont pas besoin d'être répliquées d'un fichier à l'autre. La combinaison des données entre les fichiers permet de déduire et de retrouver les informations. 

### Lisibilité
Les logs deviennent ilisibles quand aucun effort n'est fait pour structurer l'information, ou quand des données superflues sont ajoutées. Prenons par exemple le cas de l'exemple ci-dessous, le fichier persistence déjà décrit plus haut. :

```text
YYYY-MM-DD HH:mm:ss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] PersistenceUserImpl - lireUserById(id:2) 
YYYY-MM-DD HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] SQLEXP - select * from user where id = 2 
YYYY-MM-DD HH:mm:ss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] DATA - {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYY-MM-DD HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] STAT - POOL:2ms PREP:3ms REQT:5ms NETW:10ms TOTL:20ms
...
```

Plusieurs informations sont inutiles et empêchent la lisibilité.
1. Les notions de temps, de niveau de log, d'ip, de session ne sont pas obligatoire dans un fichier de persistence.
2. L'ordre d'apparition suffit. 
3. Pour faciliter les relations entre les fichiers, il est possible de conserver le numéro de requête global. Eventuellement le niveau de log.

Cela pourrait donner :

```text
YYYY-MM-DD HH:mm:ss.SSS|DEBUG|0000001|PersistenceUserImpl|lireUserById(id:2) 
YYYY-MM-DD HH:mm:ss.SSS|INFO|0000001|SQLEXP| select * from user where id = 2 
YYYY-MM-DD HH:mm:ss.SSS|DEBUG|0000001|DATA| {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYY-MM-DD HH:mm:ss.SSS|INFO|0000001|STAT|POOL|2|ms|PREP|3|ms|REQT|5|ms|NETW|10|ms|TOTL|20|ms
...
```

C'est quand même un peu plus clair non ?

### Concision
La concision permet d'économiser encore de l'espace pour faciliter la lecture. Si je reprends l'exemple ci-dessus :

```text
YYYYMMDDHHmmss.SSS| D |0000001| PUI    | lireUserById(id:2) 
YYYYMMDDHHmmss.SSS| I |0000001| SQLEXP | select * from user where id = 2 
YYYYMMDDHHmmss.SSS| D |0000001| SQLDAT | {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYYMMDDHHmmss.SSS| I |0000001| SQLSTA | OK | POOL |2|ms| PREP |3|ms| REQT |5|ms| NETW |10|ms| TOTL |20|ms
...
```
Cela améliore encore sensiblement la lisibilité. Si la notion de temps n'est pas indispensable, cela donne :

```text
D |0000001| PUI    | lireUserById(id:2) 
I |0000001| SQLEXP | select * from user where id = 2 
D |0000001| SQLDAT | {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
I |0000001| SQLSTA | OK | POOL |2|ms| PREP |3|ms| REQT |5|ms| NETW |10|ms| TOTL |20|ms
...
```

### Import par un outils d'analyse

On utilise généralement le ***quatuor RELK*** ou le ***trio ELK***, sinon cela se fait en 2 temps :
* Filtrage par catégorie de log
* Import dans un outil, par exemple un tableur

A condition que le format soit de type CSV (ou UNL), il est possible d'importer ces données dans _Excel_ comme suit :
* Renommer le fichier en .csv
* Ajouter une ligne au début du fichier:
 	
	sep=|
* Ouvrir le fichier avec Excel

Enfin, créer un tableau croisé dynamique pour faire des graphiques et des calculs sur ces ensembles de données.
En parlant, le format de date supporté par Excel est `YYYY-MM-DD HH:mm:ss`. Il faudra donc penser à supprimmer '.SSS'  avant d'importer en CSV. 

### Niveau de logs supplémentaires
Il est parfois pratique d'ajouter des niveaux de log supplémentaires en plus des 5 suivants : TRACE, DEBUG, INFO, ERROR, FATAL.
Certaines applications ajoutent des niveaux comme NOTICE, COMMENT, etc.

Il est même possible de modifier les logs par défaut, en modifiant leur nom :

|Nom de départ|Nom d'arrivée|
|-------------|-------------|
|TRACE|CODE|
|DEBUG|ALGO|
|-------------|-------------|

Afin de garder une cohérence avec d'autres projets, nous préférons utiliser les catégories de logs, plutôt que des niveaux de logs supplémentaires. Par contre, nous avons voulu en faciliter l'utilisation en proposant une API dédiée pour les cas suivants : 

#### Début et fin de traitement
```java
FuncId func = log.startFunc("Ajout de PJ de type {}, de nom {} et de taille {} Mo", fileType, fileName, fileSizeMB);
// traitement
log.done(fundId, "Ajout {} OK.", fileName)
// ou 
log.success(funcId)
// ou
log.fail(funcId, exception)
```

#### User story (Given, When, Then)
Les userstory sont une façon de représenter un test ou un traitement. Ils peuvent s'appliquer à la navigation, mais aussi à tout type de traitement qui nécessite les 4 étapes :

1. Setup
2. Exercise
3. Verify
4. Teardown

```java
// Setup
log.given("Session : {}", sessionHash);
log.given("Paramètres : {}", io.getParamValues());
// Exercise
log.when("Action " + io.getParam("action"));
// traitement
// Verify
log.then("Success");
// Teardown
```

Ce qui pourrait donner :

```text
@GIVEN Session : -978897675
@GIVEN Paramètres : [2,CONSULTERPJ]
>WHEN Action CONSULTERPJ
-THEN Success
```

ou en faisant un effort de concision :

```text
@ Session : -978897675
@ Paramètres : [2,CONSULTERPJ]
> Action CONSULTERPJ
- Success
```

## De meilleurs logs en pratique
Au vu des éléments ci-dessus, il se dégage les quelques principes suivants :

### Classer les catégorie de logs par fichier
#### http.log
Ce fichier contiendra les accès http lorsqu'il s'agit d'un application WEB. Il doit contenir la date, le numéro de requête, la méthode utilisée, la ressource demandée et des statistiques de temps de réponse. Lorsque le socle est maîtrisé, il est possible d'ajouter les informations de session et le numéro de requête session. Nous avons choisi de procéder en 2 temps :

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| HTTP | IP |10.0.0.1,192.168.0.217| SN |username|000000000000| RS |0001| GET /servlet?action=PROFIL
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| STAT | BS |25|ms| VW |2|ms| TL |28|ms
...
```
à partir de ce fichier, on récupère tous les numéros de requêtes d'une session, 

#### userstory.log
Ce fichier contiendra le comportement utilisateur. Il doit contenir les actions demandées, la date, le numéro de requête, les paramètres d'entrée et le résultat du traitement.

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| ACTION | PROFIL |{"id"=3, "group"="GSP"}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | OK | CD |25|ms| xxx.jsp
YYYY-MM-DD HH:mm:ss.SSS| D |0000001| DATA   | {...} <-- données sérialisées à afficher en DEBUG.
YYYY-MM-DD HH:mm:ss.SSS| I |0000002| RESULT | KO | CD |4|ms| Action incorrecte 
...
```

#### transport.log ou http.log
Ce fichier contiendra l'appel de la couche de service, quand un protole RPC est utilisé. D'un point de vue strict, le fichier transport.log est l'équivalent d'http.log mais pour les services exposés.

> A FAIRE

#### service.log
Ce fichier contiendra les appels de service. Il doit contenir les références de services, la date, le numéro de requête, les paramètres d'entrée et le résultat du traitement.

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| CALL   | lireProfil |{"id"=3, "group"="GSP"}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | lireProfil | OK | SCE |20|m
# ou
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | lireProfil | KO | SCE |2|ms
YYYY-MM-DD HH:mm:ss.SSS| D |0000001| RETURN | lireProfil | {...} <-- données sérialisées de retour à afficher en DEBUG. 
...
```

Le marqueur temporel n'est pas obligatoire.

#### persistence.log
Ce fichier contiendra les appels de la couche persistence. Les seules données importantes sont les requêtes SQL, les données insérées ou modifiées, les statistiques d'appels. 

```text
YYYY-MM-DD HH:mm:ss.SSS| D |0000001|UPI | lireProfil(id:2) 
YYYY-MM-DD HH:mm:ss.SSS| I |0000001|EXPR| select * from profil where id = 2 
YYYY-MM-DD HH:mm:ss.SSS| D |0000001|DATA| {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001|STAT| OK | POOL |2|ms| PREP |3|ms| REQT |5|ms| NETW |10|ms| TOTL |20|ms
...
```

Les marqueur temporels n'est pas obligatoires. Les données lues et les méthodes appelées sont en DEBUG. Par contre, les données insérée et modifiées sont loggés au niveau INFO.
