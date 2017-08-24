# Article-log-theory

La maintenance des projets informatiques nécessite la mise en place de logs, des informations textuelles ou 'traces' qui seront disponibles dans des fichiers ou des bases de données, de façon persistantes ou éphémères. Ces données textuelles sont archivées pour consultation, généralement compressées, et sont accessibles à la production, aux développeurs, mais aussi à la chefferie de projet, et à la maîtrise d'ouvrage.

Tous les projets informatiques ont besoin de produire des logs, ne serait-ce que pour assurer le débogage. Mais voilà, il en existe une telle diversité, qu'il semble impossible de proposer des règles universelles de production de logs. Peut-être bien... Qui n'a jamais produit de logs qui au final ne sont jamais lus, parce que trop verbeux ou tout simplement illisibles...  

Nous verrons dans ce billet que la mise en place de logs 'utiles' se résous relativement facilement à l'aide de 4 angles d'attaques :

1) Donner un sens à chaque niveau de log (TRACE, DEBUG, INFO, ...)
2) Définir qui est le destinataire des logs
3) Découper les logs par couches applicatives
4) Utiliser une API dédiée 

Au cours de notre étude, nous définirons ce qu'est un 'log', nous identifierons les besoins du point de vue des acteurs concernés, et enfin nous proposerons au moins une méthode (parmis tant d'autres) de mise en place de log.

## Qu'est-ce qu'un log ?
Certains amateurs de voyage ont pris l'habitude de consigner leurs aventures dans un carnet. Ce carnet comporte des faits, les impressions de l'auteur, des photos, et si le carnet est électronique, des vidéos. L'avènement des réseaux sociaux a rendu populaire le partage de 'biographies' en ligne, parfois jusqu'à l'agacement des followers tellement certains 'tweets' sont futiles. Dans d'autres cas, tellement de photos et de vidéos ont été prises que rares sont ceux qui trouvent le temps d'organiser le récit de leur voyage. On apprécie alors des services comme Flickr(R) qui organisent automatiquement ces médias et simplifie la navigation.

D'une certaine façon, l'analogie du voyage illustre la difficulté de mettre en place des logs de qualité. Un log applicatif est en quelque sorte une autobiographie du déroulement d'une application. La lecture (ou l'analyse) de ce récit doit permettre à un acteur de comprendre ce qui se passe dans l'application. Pas assez de logs, et la maintenance est quasi impossible. Trop de logs, et la maintenance devient complexe et décourageante.

Que doit-on mettre dans le récit d'un voyage ? Tout dépend du public visé *ET* de l'intention du voyageur. On ne met pas les mêmes informations quand le récit est pour soi-même, notre famille, nos amis, ou un public d'inconnus sur Internet. Il en est de même pour les logs. Le récit applicatif doit correspondre au besoin du public visé. Les 'purées' (pour ne pas dire diarrhées...) de logs sont généralement illisibles et inexploitables pour TOUS les acteurs. Ce contenu indigeste et puant est le drame des applicatifs produits par certaines sociétés de service. Les logs sont souvent générés automatiquement par les frameworks qui sont rarement des modèles de concision. Enfin, le guide du bon petit logger n'est généralement par fourni au développeur prestataire.

Des outils comme elasticsearch et logstash simplifient bien sûr l'analyse des logs, mais il n'est pas toujours possible d'avoir une telle architecture, et l'analyse des logs est plus complexe quand ceux-ci ne sont pas standardisés. Il est toujours bon de rendre les logs lisibles et cohérents à la source.

Nous ne parlerons ici que des logs qui sont produits et conservés dans des fichiers, mais les principes devraient être les mêmes pour des logs conservés dans une base de données ou un autre outils de persistence.

## Les niveaux de logs
La plupart des gestionnaires de log propose par défaut les niveaux de logs suivants : TRACE, DEBUG, INFO, WARN, ERROR, FATAL.

Toutefois, aucun n'en fournit le sens. Le tableau ci-dessous fournit un exemple de sens. Cela reste naturellement discutable. Libre au lecteur de modifier ce tableau pour son compte. 


| Niveau   | Destination directe    | Pertinence |
| -------- | --------               | --------   |
| TRACE    | Développeur       		| Les traces sont utiles en développement et en maintenance. Il permet de suivre le déroulement des algorithmes lors des séances de débuggage, et sont généralement TRES verbeux. Ces logs doivent être pertinents, soignés et écrits en pensant à la maintenance, ce qui veut dire que ce ne sont pas des logs FOURRE-TOUT.      |
| DEBUG    | Développeur            | Les logs de niveau debug contiennent généralement les *données* manipulées, et les étapes algorithmiques clé. Ces logs doivent être pertinents, soignés et écrits en pensant à la maintenance. L'activation des niveaux DEBUG et/ou TRACE a souvent pour effet de ralentir l'applicatif.   |
| INFO     | Developpeur/Autorité   | Les logs de niveau info constitue note récit applicatif 'public'. La lecture du log info devrait suffire à savoir ce qui s'est passé dans l 'aplication du point de vue métier.      |
| WARN     | Exploitant             | Les warnings indiquent qu'un traitement a pu se faire, mais qu'il aurait pu être optimisé. **Les exploitants pensent qu'ils doivent intervenir dès qu'ils rencontrent un log de ce type**.        |
| ERROR    | Exploitant/Développeur | Les erreurs indiquent qu'un traitement n'a pas pu se terminer correctement. Il s'agit toujours d'erreur technique (un fichier non trouvé, une base de données inaccessible, problème de droit, etc...). _Les erreurs fonctionnels (càd le résultat d'un traitement) pour leur part sont de niveau INFO_. **L'exploitant intervient rapidement quand il voit passer des erreurs**. Les erreurs contiennent généralement la pile d'appel, ce qui est très utile pour les développeurs.    |
| FATAL    | Exploitant             | L'erreur fatale indique que l'application n'a pas pu démarrer ou a du s'arrêter brusquement, suite à une erreur irrécupérable. **L'exploitant intervient immédiatement quand il a une erreur FATAL.**      |

Par un mécanisme de fitrage, on fixe généralement les logs au niveau 'INFO' lorsque l'application est mise en production. On passe au niveau 'DEBUG' pour consulter les données manipulées et les étapes algorithmiques clé, et au niveau 'TRACE' quand un comportement inhabituel est détecté et que les logs 'DEBUG' ou 'INFO' ne suffisent pas.

	Plus un niveau de log est courant, plus le message qu'il porte doit être synthétique.

Certaines librairies de logs fournissent les outils pour créer ses propres niveaux de log, par exemple DATA, NOTICE ou COMMENT. C'est le cas de log4j2.

### Exemple de logs provenant d'un projet de gestion en java : 
Voici de (mauvais) logs tels qu'extraites d'un projet. Juste après, nous en tirerons quelques leçons :

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
YYYY-MM-DD_HH:mm:ss.SSS DEBUG [CACHE|RG0000001] Town [hash=680745609,id=2,label=Créteil,cp=...]
...
YYYY-MM-DD_HH:mm:ss.SSS INFO [CACHE|RG0000001] }}} Fin de la mise à jour du cache : 15565 ms
```

#### Accès Web
```text
YYYY-MM-DD_HH:mm:ss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] action=PROFIL jsp>2ms metier>25ms total>28ms
...
```

#### J'ai des logs... c'est bon non ?
Et non, ce n'est pas bon... Que penser des logs ci-dessus ? Il s'agit de notre première tentative d'organiser nos logs.

Nous avons tenté de classer, d'organiser et de créer des logs du point de vue utilisateur. Le but recherché était de pouvoir lire les logs comme un roman. Pourtant il faut le reconnaître, ces logs sont complètement illisibles..., ou plus exactement : rien ne saute aux yeux en première lecture. L'information importantes, nécessaire à la maintenance, est noyée avec des informations redondantes. Par exemple, certaines phrases en langage naturel sont superflues.

Par ailleurs, les entêtes censées permettre d'associer les logs répartis sur plusieurs fichiers sont trop longues.

L'application en question étant très sollicitée, la quantité de logs présents rend la lecture malaisée. Il n'est pas possible de réduire le nombre de logs, mais il devrait être possible d'en faciliter la lecture.

Enfin, une ligne de log ne peut pas être facilement analysé. Par exemple, nous avons voulu extraire le temps de réponse et le nombre de connexion distinctes sur une période données. Pour cela, il a fallu reformater manuellement les logs d'Accès Web dans un format proche du CSV, puis l'importer dans Excel. (ok... on aurait pu le faire avec logstash, mais nous n'avons pas encore ces outils en production.. snif)

Quelques leçons : Les logs devraient être lisibles facilement avec un simple éditeur de texte. Il ne devrait y avoir ni trop ni trop peu d'informations. Il devrait être facile d'associer les logs répartis sur plusieurs fichiers. Il est indispensable d'éviter de redonder l'information. Enfin, il convient de privilégier des logs synthétiques plutôt que des phrases en langage naturel.

Nous avons pris l'habitude d'installer un outils de monitoring sur nos projets web en JAVA : [javamelody](https://github.com/javamelody/javamelody/wiki). Nous devrons donc veiller à ne pas reproduire dans les logs ce qui revient à javamelody.

## Besoins du _développeur_
IMO, le développeur a besoin de logs sur 3 périodes différentes de la vie du projet :

- En cours de développement
- Pour le déploiement (recette ou production)
- Pour le traitement des anomalies en production

### Développement
Le développeur est responsable de l'écriture des logs. Comme indiqué plus haut, il écrit :
- Des logs de niveau TRACE (ou traces). Ces logs sont l'équivalent des brouillons que l'on fait avant de rédiger un document. Toutefois, les logs de niveau TRACE *DEVRAIENT* être soignés et synthétiques.

``` java
log.trace("Etape 1 : Suppression de {} d'id {}", entry, id);
```
Pour ce niveau de log, il est préférable de choisir une librairie qui ne traite les logs que si le niveau TRACE est activé, sous peine de plomber les performances. Pour les librairies anciennes comme log4j 1.2, il vaut mieux faire :

``` java
if (log.isTraceEnabled()) {
	log.trace("Etape 1 : Suppression de " + entry + " d'id " + id);
}
```
En effet, les traces sont généralement très nombreuses, ils peuvent sérieusement consommer du temps CPU et amoindrir les performances de l'application.

- Des logs DEBUG pour analyser ses algorithmes et les données manipulées. Ces logs *DOIVENT* être soignés. Ils sont l'équivalent des notes contenues dans un carnet de voyage.

``` java
log.debug("Lecture fichier {} terminée. Taille:{}", fileName, size);
```
De même, les logs debug peuvent être assez nombreuses. L'utilisation de `log.isDebugEnabled()` ou d'une librairie qui teste au préalable si le niveau DEBUG est actif préservera les performances.

- Des logs INFO pour présenter les étapes et le résultat d'un traitement métier. C'est eux le 'récit public' de notre voyage, celui qu'on diffuse sur un blog, ou sur facebook par exemple.

``` java
log.info("Nouvel utilisateur : {}", newUser);
// traitement
log.info("Résultat : {}", result);
```
La lecture des logs infos permet de savoir précisément les actions utilisateur et la réponse de l'application. Sa lecture doit permettre de suivre un utilisateur. Les erreurs fonctionnelles sont de niveau INFO. Il convient d'éviter les redondances, et d'être concis. Une information qui peut être déduite n'a pas besoin d'être détaillée textuellement. 

- Des logs WARN pour attirer la vigilance de la production. La présence de WARN interpelle forcément la production. Ces logs doivent donc être soignés et ciblés.
  
```java
log.warn("Encodage non spécifié, l'encodage par défaut sélectionné {}", config.getEncoding());
```
On a parfois tendance à créer des logs de niveau WARN lorsqu'un traitement rencontre une 'erreur' qu'il est capable de gérer. Ce n'est pas un warning (selon la définition plus haut qu'on lui a attribué dans le cadre de ce document). Une erreur fonctionnelle prévisible devrait être donc de niveau INFO . Comment bien choisir ce qui doit être de niveau WARN ? Examinons les cas suivants : 

1. Avertir de la non présence d'un fichier optionnel
2. Avertissements fonctionnels (ou métier)
3. Données issues de la BD incomplètes
4. Indiquer des dégradations de perfomances

Parmis ces 4 situations, lequelles sont des logs de niveau WARN ? La 1, 3 et la 4. En effet, la 2 n'intéresse pas l'exploitation et doit être de niveau INFO (il s'agit du résultat d'un traitement). 

- Des logs ERROR pour avertir la production. En plus d'un message, ils doivent comporter des stacktraces pour faciliter l'identification de l'erreur. C'est généralement la production qui repère les erreurs. Elle peut toutefois donner aux développeurs un accès aux logs.

``` java
log.error("La lecture du fichier {} a échoué, exception, viewFileName);
```
Les erreurs sont principalement de nature technique, et concerne en majorité des exceptions de type Runtime. Une erreur est une impossibilité de terminer un traitement par suite d'un imprévu technique. Par exemple :

1. Erreur d'accès à la base de données
2. NullPointerException, ou plus généralement RuntimeException
3. Contrat d'appel non respecté d'une méthode ou d'un service
4. Données incorrectes.

Nous avons choisi de faire produire les messages des erreurs les plus connus par le socle applicatif. Les cas d'erreurs étant connus, une proposition de solution précuite est proposée dans le message de l'erreur. 

- Des logs FATAL pour indiquer des éléments manquants qui empêchent le démarrage ou le bon fonctionnement de l'application.

``` java
log.fatal("Impossible de trouver le fichier de configuraiton {}", configFileName);
```

Un log FATAL précède généralement un `System.exit(errorCode);`. Comme  vous l'avez sûrement noté, les logs 'FATAL' et 'ERROR' sont écrits avec des phrases complètes. 

### Déploiement
Le développeur a besoin des logs de niveau FATAL, et des logs de DEBUG généraux qui sont envoyés sur la sortie standard.

### Anomalies de la production
En production, c'est généralement les stacktraces des logs de niveaux WARN et ERROR qui nous intéressent, ainsi que les logs de niveau INFO. Les logs 'DEBUG' et parfois 'TRACE' sont activés seulement si le développeur ne s'y retrouve pas.

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
 
Le temps de réponse instantanée, la consommation mémoire, ..., c'est-à-dire l'état de santé générale de l'application peuvent généralement être fournis par des outils comme JavaMelody. Toutefois certaines données auraient intérêt à se retrouver dans les logs, comme le temps de réponse, la consommation mémoire, etc, données qui pourront ête interrogées plus tard.

Afin de faciliter l'extraction de ces données, celles-ci ont intérêts à avoir un format facile à traiter. Par exemple le format UNL.
Reprenons les logs 'Accès Web' que nous avons vus plus haut, essayons par exemple :

```java
YYYY-MM-DD HH:mm:ss.SSS | INFO | IP | 10.0.0.1,192.168.0.217 | RG | 0000001 | SH | username | 000000000000 | RS | 0001| PROFIL | VW| 2 | ms | BS | 25 | ms | TTL | 28 | ms
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

Ainsi, les logs de niveau INFO doivent contenir ces informations. Les données insérées, mises à jour ou supprimées doivent être loggées de façon à pouvoir défaire des opérations jugées malveillantes si nécessaire.

## Autres considérations
### Catégories de logs
Les applications n-tiers structurés à l'aide de pattern ont généralement (plus ou moins) les couches suivantes :

1. L'utilisateur (navigateur ou desktop)
2. La vue
3. Le contrôleur (ou le client)
4. La commande
5. Une couche de transport (optionnelle)
6. Le service
7. La persistence
8. Les DAO
9. La connexion à la base de données
10. D'eventuels périphériques (imprimantes, périphériques connectés, etc...), ou traitements asynchrones.

Chaque couche jourera un rôle différent dans la production de logs. 

En fonction des besoins détaillés plus haut dans le document, il en ressort les catégories suivantes :
- Protocole HTTP ou autre (clic sur la Vue)
- Commande utilisateur (les opérations demandées, les données envoyées ou reçus et le résultat du traitement).
- La couche transport si présente (la couche transport sert à implémenter les protocoles d'appels RPC)
- Les services appelés
- La persistence (SQL, données modifiées).
- Un fichier par périphérique, ou par type de traitement asynchrone
- Les erreurs

Une ou plusieurs catégories peut être envoyé dans un même fichier, ou un fichier distinct :
- general.log
- http ou view.log
- userstory.log
- transport.log
- business.log
- persistence.log (catégories : sqlstat, sqldata, sqlexp)
- error.log
- cache.log, device-xxx, resultat, ...

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
* Renommer le fichier déj filtré en '.csv'
* Ajouter une ligne au début du fichier:
 	
	sep=|
* Ouvrir le fichier avec Excel
* Créer un tableau croisé dynamique pour faire des graphiques et des calculs sur ces ensembles de données.

En parlant, le format de date supporté par Excel est `YYYY-MM-DD HH:mm:ss`. Il faudra donc penser à supprimmer '.SSS'  avant d'importer en CSV. 

### Niveau de logs supplémentaires
Il est parfois pratique d'ajouter des niveaux de log supplémentaires en plus des 5 suivants : TRACE, DEBUG, INFO, ERROR, FATAL.
Certaines applications ajoutent des niveaux comme NOTICE, COMMENT, DATA etc.

Il est même possible de modifier les logs par défaut, en modifiant leur nom :

|Nom de départ|Nom d'arrivée|
|-------------|-------------|
|TRACE|ALGO|
|DEBUG|DATA|
|-------------|-------------|

Afin de garder une cohérence avec d'autres projets, nous utilisons les catégories (un nom spécial que l'on donne au logger) de logs, plutôt que des niveaux de logs supplémentaires. 

Pour encore plus de souplesse, nous avons développé une API dédiée pour des cas d'utilisation courants : 

#### Début et fin de traitement
```java
FuncId func = log.start("Ajout de PJ de type {}, de nom {} et de taille {} Mo", fileType, fileName, fileSizeMB);
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
Ce fichier contiendra les accès http lorsqu'il s'agit d'un application WEB. Il doit contenir la date, le numéro de requête, la méthode HTTP utilisée, la ressource demandée et des statistiques de temps de réponse. Lorsque le socle est maîtrisé, il est possible d'ajouter les informations de session et le numéro de requête session. Nous avons choisi de procéder en 2 temps :

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| HTTP | IP |10.0.0.1,192.168.0.217| SN |username|000000000000| RS |0001| GET /servlet?action=PROFIL
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| STAT | BS |25|ms| VW |2|ms| TL |28|ms
...
```

#### userstory.log
Ce fichier contiendra le comportement utilisateur. Il doit contenir les actions demandées (dans le cadre d'un MVC), la date, le numéro de requête, les paramètres d'entrée et le résultat du traitement.

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| ACTION | PROFIL |{"id"=3, "group"="GSP"}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | OK | CD |25|ms| xxx.jsp
YYYY-MM-DD HH:mm:ss.SSS| D |0000001| DATA   | {...} <-- données sérialisées à afficher en DEBUG.
YYYY-MM-DD HH:mm:ss.SSS| I |0000002| RESULT | KO | CD |4|ms| Action incorrecte 
...
```

#### transport.log ou http.log
Ce fichier contiendra l'appel de la couche de service, quand un protocole RPC est utilisé. D'un point de vue strict, le fichier transport.log est l'équivalent d'http.log mais pour les services exposés.

#### service.log
Ce fichier contiendra les appels de service. Il doit contenir les références de services, la date, le numéro de requête, les paramètres d'entrée et le résultat du traitement.

```text
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| CALL   | lireProfil |{"id"=3, "group"="GSP"}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | lireProfil | OK | SCE |20|ms
# ou
YYYY-MM-DD HH:mm:ss.SSS| I |0000001| RESULT | lireProfil | KO | SCE |2|ms
YYYY-MM-DD HH:mm:ss.SSS| D |0000001| RETURN | lireProfil | {...} <-- données sérialisées de retour à afficher en DEBUG. 
...
```

Les marqueurs temporels ne sont pas toujours obligatoires. Même si cela prend un peu plus de place, pour ma part, je préfère les conserver pour faciliter le classement entre les fichiers. 

#### persistence.log
Ce fichier contiendra les appels de la couche persistence. Les seules données importantes sont les requêtes SQL, les données insérées ou modifiées et les statistiques d'appels. 

```text
YYYY-MM-DD HH:mm:ss.SSS| D |0000001|UPI | lireProfil(id:2) 
YYYY-MM-DD HH:mm:ss.SSS| I |0000001|EXPR| select * from profil where id = 2 
YYYY-MM-DD HH:mm:ss.SSS| D |0000001|DATA| {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYY-MM-DD HH:mm:ss.SSS| I |0000001|STAT| OK | POOL |2|ms| PREP |3|ms| REQT |5|ms| NETW |10|ms| TOTL |20|ms
...
```

Les marqueur temporels ne sont pas toujours obligatoires. Même si cela prend un peu plus de place, je préfère les conserver pour faciliter le classement. Les données lues et les méthodes appelées sont en DEBUG. Par contre, les données insérée et modifiées sont loggés au niveau INFO.

# Conclusion
Nous avons vu les quelques principes qui permettent aux logs d'être bien plus efficientes :

1) Donner un sens à chaque niveau de log (TRACE, DEBUG, INFO, ...)
> Avoir une définition commune des logs permet de garder la même cohérence d'un bout à l'autre de l'application, et entre les applications d'un équipe.

2) Définir qui sont les destinataires des logs
> On est ainsi capable de savoir quel niveau et quelle quantité d'information il faut produire. 
> Les logs doivent être lisibles à la source, sans avoir besoin d'un outils RELK. Un éditeur de texte devrait suffire. 
> Plus les logs se produisent souvent, plus ils doivent être concis et synthétiques.
> Il ne devrait pas y avoir de log poubelle. Chaque log doit être soigné.

3) Découper les logs par couches applicatives
> La plupart des applications n-tiers comportent plus ou moins les mêmes couches applicatives. L'idée n'est pas de créer un fichier de log par couche, mais d'attribuer à des logs une catégorie, un nom, et de lui donner un sens applicatif, généralement calqué sur la couche application sur laquelle le log se produit.

4) Utiliser une API dédiée adaptée aux règles de votre équipe
> Il s'agit généralement d'un wrap autour de la librairie de logs utilisés pour rendre prévisible et standardisé la production de logs. Nous en avons vu un petit aperçu. Un autre billet détaillera un exemple d'API possible autour de log4j2 par exemple.  

N'hésitez pas à réagir. Votre expérience sera utile à tous les lecteurs.