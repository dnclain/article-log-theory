# bofself-poc-log

Ce document est une recherche sur l'amélioration des logs pour les projets informatiques.

La maintenance des projets informatiques nécessite la mise en place de logs, des informations textuelles ou 'traces' qui seront disponibles dans des fichiers, des bases de données, de façon persistantes ou éphémères. Ces données textuelles sont archivées pour consultation, généralement compressées, et sont accessibles à la production, aux développeurs, mais aussi à la chefferie de projet, et à la maîtrise d'ouvrage. Nous verrons au cours de ce document comment les logs peuvent être utiles à ces derniers. 

Il existe une telle diversité de projets informatiques qu'il semble impossible de proposer des règles universelles de gestion des logs. Peut-être bien... Pourtant, une fois qu'on a compris à qui les logs sont destinés et qu'est-ce qu'ils en attendent, des principes assez simples sont facilement identifiés.

Dans ce document, nous définirons ce qu'est un 'log', les besoins du point de vue de différents acteurs, et enfin nous proposerons une ou plusieurs méthodes (parmis tant d'autres) de mise en place de log.

## Définition des logs
Lors d'un voyage, on consigne parfois des faits, on décrit les étapes, on expose ce qu'on a ressenti. On accompagne souvent nos impressions de photos, de cartes ou de vidéos. L'avènement des raisons sociaux a rendu populaire le partage de 'biographies' en ligne, parfois jusqu'à l'agacement de nos destinataires tellement certains de nos 'tweets' sont futiles. Dans d'autres cas, tellement de photos et de vidéos ont été prises que rares sont ceux qui trouve le temps d'organiser le récit de leur voyage. On apprécie alors des services comme Flickr(R) qui organisent automatiquement ces médias et simplifie la navigation.

D'une certaine façon, l'analogie du voyage illustre la difficulté de mettre en place des logs de qualité. Un log applicatif est en quelque sorte une autobiographie du déroulement d'une application. La lecture (ou l'analyse) de ce récit doit permettre à un acteur de comprendre ce qui se passe dans l'application.

Que doit-on mettre dans le récit d'un voyage ? Tout dépend de l'intention du voyageur *ET* du public visé. On en met pas les mêmes informations quand le récit est pour soi-même, notre famille, nos amis, ou un public d'inconnnu sur Internet. Il en est de même pour les logs. Le récit applicatif doit correspondre au besoin du public visé. Les 'purées' (pour ne pas dire diarrhées...) de logs sont généralement illisibles et inexploitables pour TOUS les acteurs. Ce contenu indigeste et puant est le drame des applicatifs produits par les sociétés de service. Les logs sont souvent générés automatiquement par les frameworks qui sont rarement des modèles de concision. Encore pire, le guide du bon petit logger n'est généralement par fourni au développeur prestataire.

La plupart des gestionnaires de log propose par défaut des niveaux de logs :


| Niveau   | Destination directe    | Pertinence |
| -------- | --------               | --------   |
| TRACE    | En développement       | Les traces sont utiles en développement uniquement. Il permet de suivre le code lors de séance de débuggage, et sont généralement TRES verbeux.       |
| DEBUG    | Développeur            | Les logs de niveau debug contiennent le suivi pas à pas d'un algorithme avec les données. Ces logs doivent être pertinents et écrits en pensant à la maintenance. Le niveau de verbosité est relatif.    |
| INFO     | Developpeur/Autorité   | Les logs de niveau info constitue note récit applicatif. La lecture du log info devrait suffire à savoir ce qui s'est passé dans l 'aplication       |
| WARN     | Exploitant             | Les warnings indiquent qu'un traitement a pu se faire, mais qu'il aurait pu être optimisé. **Les exploitants pensent qu'ils doivent intervenir dès qu'ils rencontrent un log de ce type**.        |
| ERROR    | Exploitant/Développeur | Les erreurs indiquent q'un traitement n'a pas pu se terminer correctement. Il s'agit toujours d'erreur technique (un fichier non trouvé, une base de données inaccessible, problème de droit, etc...). _Les erreurs fonctionnels sont de niveau INFO_. **L'exploitant intervient rapidement quand il voit passer des erreurs**. Les erreurs contiennent généralement la pile d'appel, ce qui est très utile pour les développeurs.    |
| FATAL    | Exploitant             | L'erreur fatale indique que l'application n'a pas pu démarrer ou a du s'arrêter brusquement, suite à une erreur irrécupérable. **L'exploitant intervient immédiatement quand il a une erreur FATAL.**      |

Par un mécanisme de fitrage, on fixe généralement les logs au niveau 'INFO' lorsque l'application est mise en production. On passe au niveau 'DEBUG' quand un comportement inhabituel est détecté et que les logs 'INFO' ne suffisent pas.

Des librairies de logs fournissent les outils pour créer ses propres niveaux de log, par exemple NOTICE ou COMMENT. C'est le cas de log4j2.

### Exemple de logs provenant d'un projet de gestion en java : 
Voici des logs tels qu'extraitent d'un projet. Tiron-en quelques leçons juste après :

#### Persistence 
```
YYYYMMDDHHmmss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] PersistenceUserImpl - lireUserById(id:2) 
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] SQLEXP - select * from user where id = 2 
YYYYMMDDHHmmss.SSS DEBUG [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] DATA - {"id":2, "name":"Bent", "firstname":"Joshua", "type":5 ...}
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] STAT - POOL:2ms PREP:3ms REQT:5ms NETW:10ms TOTL:20ms
...
```

#### User story
```
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] action=PROFIL L'utilisateur affiche son profil 
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0002] action=PROFIL [params:id=2,name=Bento,...] L'utilisateur modifie son profil 
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0003] action=ACCUEIL L'utilisateur affiche la page d'accueil
...
```

#### Background process
```
YYYYMMDDHHmmss.SSS INFO [CACHE|RG0000001] Démarrage de la mise à jour du cache {{{
YYYYMMDDHHmmss.SSS DEBUG [CACHE|RG0000001] Town [hash=908798678,id=1,label=Paris,cp=...]
YYYYMMDDHHmmss.SSS DEBUG [CACHE|RG0000001] Town [hash=908798678,id=2,label=Créteil,cp=...]
...
YYYYMMDDHHmmss.SSS INFO [CACHE|RG0000001] }}} Fin de la mise à jour du cache : 15565 ms
```

#### Accès Web
```
YYYYMMDDHHmmss.SSS INFO [username|IP10.0.0.1,192.168.0.217|RG0000001|SH000000000000|RS0001] action=PROFIL jsp>2ms metier>25ms total>28ms
...
```

#### J'ai des logs, et alors ?
Que penser des logs ci-dessus ? Il s'agit de ma première tentative d'organiser les logs. En effet, dans notre équipe, aucune règle de log existait. Il fallait bien commencer par quelque chose. 

Nous avons tenté de classer, d'organiser et de créer des logs du point de vue utilisateur. Le but recherché était de pouvoir lire les logs comme un roman. Pourtant il faut le reconnaître, ces logs sont illisibles... Beaucoup d'informations sont redondantes et il est difficile de retrouver ce qui est réellement nécessaire pour la maintenance. Par exemple, certaines phrases en langage naturel sont superflues.

Par ailleurs, les entêtes censées permettre d'associer les logs répartis sur plusieurs fichiers sont trop longues.

L'application en question étant très sollicitée, la quantité de logs présents rend la lecture encore plus compliquée. Il n'est pas possible de réduire le nombre de logs, mais il devrait être possible d'en faciliter la lecture.

Enfin, une ligne de log ne peut pas être facilement analysé. Par exemple, nous avons voulu extraire le temps de réponse et le nombre de connexion distinctes sur une période données. Pour cela, il a fallu reformater manuellement les logs d'Accès Web dans un format proche du CSV, puis l'importer dans Excel. 

Je travaille avec [gitea](https://gitea.io/) depuis quelques temps, et je dois avouer que les logs produits par cette application sont exemplaires. Il y a ni trop ni trop peu d'informations. Il facile d'associer les logs répartis sur 3 fichiers (exceptés pour les logs SQL), et aucune information n'est redondante. Cela faisait quelques temps que nous voulions fixer une norme de 'logging', et je crois bien avoir trouvé une excellente source d'inspiration.

Nous complétons les logs par [javamelody](https://github.com/javamelody/javamelody/wiki) comme outils de monitoring et d'analyse. Nous devrons donc veiller à ne pas reproduire dans les logs ce qui revient à javamelody.

## Besoins du _développeur_
IMO, le développeur a besoin de logs sur 3 périodes différentes de la vie du projet :

- En cours de développement
- Pour le déploiement
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
De même, les logs debug peuvent être assez nombreuses. L'utilisation de `log.isDebugEnabled()` ou d'une librairie qui teste si le niveau DEBUG est actif préservera les performances.

- Des logs INFO pour présenter le résultat d'un traitement. C'est eux le 'récit public' de notre voyage.

``` java
log.info("Nouvel utilisateur soumis : {}", newUser);
// traitement
log.info("Soumission utilisateur : {}", result);
```

La lecture des logs infos permet de savoir précisément les actions utilisateur et la réponse de l'application. Sa lecture doit permettre de suivre une utilisateur. Les erreurs fonctionnelles sont de niveau INFO. Il convient d'éviter les redondances, et d'être concis. Une information qui peut être déduite n'a pas besoin d'être détaillées textuellement. 

- Des logs WARN pour attirer la vigilance de la production. La présence de WARN interpelle forcément la production. Ces logs doivent donc être soignés et ciblés.
  
```java
log.warn("Encodage non spécifié, l'encodage par défaut sélectionné {}", config.getEncoding());
```
On a parfois tendance à créer des logs de niveau WARN lorsqu'un traitement rencontre une 'erreur' qu'il est capable de gérer. Ce n'est pas un warning. Une erreur prévisible fonctionnelle ou non rencontrée lors d'un traitement devrait être de niveau INFO . Comment bien choisir ce qui doit être de niveau WARN ? Examinons les cas suivants : 

1. Avertir de la non présence d'un fichier optionnel
2. Averissements fonctionnels
3. Données issues de la BD incomplètes
4. Indiquer des dégradations de perfomances

Parmis ces 4 situations, lequelles sont des logs de niveau WARN ? La 1, 3 et la 4. En effet, la 2 n'intéresse pas l'exploitation et doit être de niveau INFO (il s'agit du résultat d'un traitement). 

- Des logs ERROR pour avertir la production. En plus d'un message, ils doivent comporter des stacktraces pour faciliter la compréhension de l'erreur. C'est généralement la production qui repère les erreurs. Elle peut toutetefois donner un accès aux logs aux développeurs.

``` java
log.error("Lecture du fichier du résultat {} impossible, exception, viewFileName);
```
Les erreurs sont principalement de nature techniques, et concerne en majorité des exceptions de type Runtime. Une erreur est le résultat d'une impossibilité de terminer un traitement par suite d'un imprévu technique. Par exemple :

1. Erreur d'accès à la base de données
2. NullPointerException, ou plus généralement RuntimeException
3. Contrat d'appel non respecté d'une méthode ou d'un service
4. Données d'entrée incorrecte.

Nous avons choisit de faire produire les messages d'erreur les plus connus par le socle applicatif que nous avons développés. Les cas d'erreurs sont connus, et une solution est proposée. 

- Des logs FATAL pour indiquer des éléments manquants qui empêche le démarrage ou le bon fonctinnement de l'application.

``` java
log.fatal("Impossible de trouver le fichier de configuraiton {}", configFileName);
```

Un log FATAL précède généralement un `System.exit(errorCode);`

### Déploiement
Le développeur a besoin des logs de niveau FATAL, et des logs de DEBUG généraux qui sont envoyé sur la sortie standard.

### Anomalies de la production
En production, c'est généralement la stacktrace 

## Besoins de la _production_
### Déploiement

### Gestion des erreurs

## Besoins du _chef de projet_
### Anomalies de la production

### Performance

## Besoins de la _MOA_ et des _autorités_
### Performance

## Autres considérations
### Lisibilité
### Concision
### Export vers un outils d'analyse
### Niveau de logs supplémentaires
#### Début et fin de traitement
#### User story (Given, When, Then)

## De meilleurs logs en pratique
