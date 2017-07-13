# bofself-poc-log

Ce projet est une recherche sur la théorie des logs pour les projets informatiques.

La maintenance des projets informatiques nécessite la mise en place de logs, des informations textuelles ou 'traces' qui seront disponibles dans des fichiers, des bases de données, de façon persistantes ou éphémères. Ces données textuelles sont archivées pour consultation, généralement compressées, et sont accessibles à la production, aux développeurs, mais aussi à la chefferie de projet, et à la maîtrise d'ouvrage. Nous verrons au cours de ce document comment les logs peuvent être utiles à ces derniers. 

Il existe une telle diversité de projets informatiques qu'il semble impossible de proposer des règles universelles de gestion des logs. Peut-être bien... Pourtant, une fois qu'on a identifié à qui les logs sont destinés et qu'est-ce qu'ils en attendent, des principes assez simples sont facilement identifiées.

Dans ce document, nous définirons ce qu'est un 'log', les besoins du point de vue de différents acteurs, et enfin nous proposerons une ou plusieurs méthodes (parmis tant d'autres) de mise en place de log.

## Définition des logs
Lors d'un voyage, on consigne parfois des faits, on décrit les étapes, on expose ce qu'on a ressenti. On accompagne souvent nos impressions de photos, de cartes ou de vidéos. L'avènement des raisons sociaux a rendu populaire le partage de 'biographies' en ligne, parfois jusqu'à l'agacement de nos destinataires tellement certains de nos 'tweets' sont futiles. Dans d'autres cas, tellement photos et de vidéos ont été prises qu'on trouve rarement le temps d'organiser le récit de son voyage. On apprécie alors des services comme Flickr(R) qui organisent automatiquement ces données et simplifie la navigation.

D'une certaine façon, l'utilisation de ces outils mondernes illustre la difficulté de mettre en place des logs de qualité. Un log applicatif est en quelque sorte une autobiographie du déroulement d'une application. La lecture (ou l'analyse) de ce récit doit permettre à un acteur de comprendre ce qui se passe dans l'application.

Que doit-on mettre dans le récit d'un voyage ? Tout dépend de l'intention du voyageur *ET* du public visé. On en met pas les mêmes informations quand le récit est pour soi, notre famille, nos amis, ou un public d'inconnnu sur Internet. Il en est de même pour les logs. Le récit applicatif doit correspondre au besoin du public visé. Les 'purées' (pour ne pas dire diarrhées...) de logs sont généralement illisibles et inexploitables pour TOUS les acteurs. Ce contenu indigeste et puant est le drame des applicatifs produits par les sociétés de service. Les logs sont souvent générés automatiquement par les frameworks qui sont rarement des modèles de concision. Par ailleurs, le guide du bon petit logger n'est généralement par fournit par le prestataire.

Heureusement, la plupart des gestionnaire de log propose par défaut des niveaux qu'il est possible de filtrer par configuration :


| Niveau   | Destination directe    | Pertinence |
| -------- | --------               | --------   |
| TRACE    | En développement       | Les traces sont utiles en développement uniquement. Il permet de suivre le code lors de séance de débuggage, et sont généralement TRES verbeux.       |
| DEBUG    | Développeur            | Les logs de niveau debug contiennent le suivi pas à pas d'un algorithme avec les données. Ces logs doivent être pertinents et écrits en pensant à la maintenance. Le niveau de verbosité est relatif.    |
| INFO     | Developpeur/Autorité   | Les logs de niveau info constitue note récit applicatif. La lecture du log info devrait suffire à savoir ce qui s'est passé dans l 'aplication       |
| WARN     | Exploitant             | Les warnings indiquent qu'un traitement a pu se faire, mais qu'il aurait pu être optimisé. Les exploitants pensent qu'ils doivent intervenir dès qu'ils rencontrent un log de ce type.        |
| ERROR    | Exploitant/Développeur | Les erreurs indiquent q'un traitement n'a pas pu se terminer correctement. Il s'agit toujours d'erreur technique (un fichier non trouvé, une base de données inaccessible, problème de droit, etc...). Les erreurs fonctionnels sont de niveau INFO. L'exploitant pense qu'il doit intervenir quand il voit passer un tel message de log. Les erreurs contiennent généralement la pile d'appel, ce qui est très utile pour les développeurs.    |
| FATAL    | Exploitant             | L'erreur fatale indique que l'application n'a pas pu démarrer ou a du s'arrête brusquement, suite à une erreur irrécupérable.       |




## Les logs du point de vue _'développeur'_

## Les logs du point de vue _'responsable de production'_

## Les logs du point de vue _'chef de projet'_

## Les logs du point de vue _'MOA'_

## Autres considérations

## La théorie des logs en pratique
