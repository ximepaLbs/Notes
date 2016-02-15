# Choix du Framework de monitoring
*Auteur : ximepa <ximepa@dataam.me>*

*Date : 2015-05-18*


## Les "pure API" (type I)


Avantages : 

Facile à déployer, 100% personalisables, coût nul ou minimal, très "devops"

Inconvénients : toute une infra doit être déployée, car ce sont juste des "frameworks de remontée d'évènements".
Le stockage, la représentation (Graphique/temporelle/KPI), et les alertes/traitements de ces évènements doivent être configurés par d'autres outils.

Le leader est **Sensu**

-[Sensu](www.sensuapp.org)
Il connecte des  "check scripts" et des "handler scripts" . 
On est sur un système de collecte d'information prédéterminée
les politiques possibles de réaction à base de "handler" sont : alerte ou collecte our processing

- *le +*

	- API Rest
	- très bonne appréciation de la communauté devops
	- modèle 'queue-suscriber'
	- **scalable**
	- ruby (scripts) et JSON (configuration).
	- intégration des plugins NAGIOS existant
	- un "support entreprise" pas cher (https://sensuapp.org/support/) Dans les 1$/noeud/mois

- *le -*
	
	- simplement une "gestion d'évènements" : on doit construire un "hybride" pour la partie "représentation" des donées, stockage, et rajouter un gestionnaire d'alerte à faire en plus. En général, il est combiné avec un pagerduty (alerte/réaction) et graphite (pour la représentation)
	- le dashboard est moyen
	- besoin d'un agent...


- Monitoring (ex-watchdog)

## Les Systematiques (type II): 

*le +* : 

- One size-fit-all : tout est d'un bloc : récupération d'évènements, stockage, représentation. 
- Récupération d´ évènements multi sources : syslog, applicatif, réseau, O.S, DB etc...
- Possibilité de faire de la vérification de configuration (gros + en termes de sécurité)

*le -* :

- lourd voir très lourd à configurer
- pas facilement scalable : conçus pour des architectures fixes

Les recommandés : 

### [Nagios](https://www.nagios.com/products/nagiosxi)

- *le +*

	- le choix "safe" par défaut car "tout le monde connait"
	- une architecture par plugins avex 1M plugins
	- simple et robuste
- *le -*
	- cher :4000 USD
	- non scalable : pour infrastructure statique. Redémarrer à chaque ajoute de host...
	- interface...
	- API
	- Vieux : fondamentalement pas adapté au devops/cloud
	- le "master" fait toute les vérifications (BW..)

### [Zenoss](http://www.zenoss.com/)
 *bon aussi mais considéré comme un peu après*
- *le +*  : simple à installer
- *le -* 
	- besoin de SNMP
	- cher
	- pas de configuration CLI
	- Comme Nagios, une version "CORE", en CLI, avec des features limitées, et une version "Entreprise", plus performantes, plus facile à administrer, avec une interface graphique.

### [Zabbix](http://www.zabbix.com/monitor_everything.php)

- *le -*
	- lourd à configurer
	- massif à prendre en mains
	- architecture client/serveur classique
- *le +*
	- interface customisable
	- monitoring graphique, par agent, réseau ou snmp. La possibilité d'être "agentless" est un gros +
	- scalable : auto-discovery de services
	- léger en terme de charge machine
	- python
	- Documentation
	- Gartner 2013 Best Monitoring software #2 après Nagios
	- remontée d'évènements multi-couches : 'hw, network, O.S, middleware'


### [Icinga2](https://www.icinga.org/)

- *le +* 
	- support du format de données "graphite"
	- input de collectd ou monit
	- conçu pour du cloud clusterisable, oug balancing, réplication....
	- un peu plus moderne et plus adapté au cloud

- *le -* 
	- plus récent,moins connu
	- lourd à configurer
	- moins bonne réputation que Zabbix ou Zenoss


## les complémentaires (type 3) : représentation de données

- Munin : systeme client/serveur de représentation graphiques de "santé" d'un parc de manière graphique
[disque, net]

	- *le -*
		- graphes only
		- text/cli/code (pas de supervision via gui)
		- architecture fixe

	- *le +*
		- simple a deployer
		- data stockée au format rrd

- Cacti
  	- *le +*
		- souvent intégré avec Nagios

- Graphite 

## api vs systématique

L'avantage d'un outil de type I:c'est *léger*, et on peut facilement faire du *custom*. Très adapté à une architecture qui évolue. 
Pour moi, notre architecture étant amenée à évoluer, tant physiquement qu'applicativement, ce choix est celui de la scalabilité et des API.
Les Type I sont des outils qui permettent d'avoir une "vision" sur des incidents pré-determinés (je "check" la taille du disque disponible, ou le temps de lancement de tel ou tel binaire, etc.)

C'est beaucoup mieux que des scripts bash, car plus facilement maintenable et réutilisables. 
Mais il faut développer à chaque "node" les routines décrivant les choses à superviser et les détails de remontée d'incident. Donc il y a une montée en charge, avec la création des évènements qu'on veut remonter, et des politiques de réaction associées. 
En gros, c'est une logique de dev : un évènement en plus de monitoring = un bout de code en plus

Le gros avantage, c'est que c'est une logique linux : ils font une chose et le font bien. On peut donc facilement l'intégrer, soit à Manager, soit les coupler à d'autres stacks de monitoring de type ELK (Elastic Search, Logstash, Kibana). Dont on avait parlé.
Un autre avantage est que l'on peut construire des graphes de performances / monitoringévolués, que ce soit avec Kibana, Graphite, ou la combinaison des deux (graphana)

Pour moi, c´ est là "way to go" long terme. Mais cela va prendre du temps pour construire l'infra (essentiellement applicative). Pas de monitoring de ce type au moins un mois. Par contre, cela peut-être construit de façon incrémentale.


Les outils de type II sont en général considérés comme top pour des entreprises dont l'IT n'est pas le coeur de métier, et où les infrastructures n'évoluent pas trop. Je recommande Zabbix parmi tout ceux testés. Il y a une grande learning curve, mais ensuite, ça fait tout. C'est top pour superviser une infrastructure, car on a un outil, qui n'évolue pas trop. Une fois qu'on sait s'en servir, ca fait le boulot, malgré ses limites.
C'est une approche traditionnelle plutôt que devops. Plus de configuration mais pas de dev. A première vue, la learning curve est du même ordre.

Si j'avais à choisir, je prendrais l'approche devops, car c'est dans la lignée du cloud et de notre approcher "Code" des services. C'est intégrable et scalable. Par contre, on est sur du long terme avec des résultats pas immédiats. A prendre en compte dans la décision

Je mettrais à jour ce fichier avec des schémas d'architecture.



----

References : 

[Comparaison Nagios vs Zenoss](http://systems-management.softwareinsider.com/compare/17-24/Nagios-XI-vs-Zenoss-Service-Dynamics)

[OpenSource monitoring systems](http://www.slideshare.net/forthscale/open-source-monitirng-systems-1)

[Alternatives Nagios 2014](http://www.slideshare.net/superdupersheep/stop-using-nagios-so-it-can-die-peacefully)

[Nagios vs Sensu](http://www.slideshare.net/davetibbs/sensu-at-brightpearl-36142201?next_slideshow=1)

[Sensu monitoring](http://www.slideshare.net/miquelruizm/monitoring-with-sensu)

[OSS Monitoring](http://www.slideshare.net/vo_mike/the-opensource-monitoring-landscape?qid=39f04ff7-2956-4c17-b339-948a6ffdc532&v=default&b=&from_search=19)