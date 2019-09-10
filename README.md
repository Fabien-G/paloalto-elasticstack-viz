*** Projet en cours au 10/09/2019 ***  !!! Ne pas se fier à ce projet pour le moment !!!


Mise en place d'une visualisation des syslog Palo Alto dans la suite ELK (ElasticSearch / Logstash / Kibana).
J'en profite au passage pour traduire en francais les explications fournies par sm-biz (merci à lui).

Récupération des données Palo Alto v8.1

***

## Tableaux de bord

Le projet comprend neuf tableaux de bord, qui ont été pré-construits à partir des visualisations incluses.

Vue d'ensemble | Tableau des menaces | Tableau du traffic
------------ | ------------- | -------------
[![Dashboard - Overview](https://i.imgur.com/xxl0XCfm.png)](https://i.imgur.com/xxl0XCf.png) | [![Dashboard - Threats](https://i.imgur.com/obE4dIbm.png)](https://i.imgur.com/obE4dIb.png) | [![Dashboard - Traffic](https://i.imgur.com/xuxsmnom.png)](https://i.imgur.com/xuxsmno.png)

En plus de ce qui précède, il existe des tableaux de bord pour;
* Applications
* Threat Highlights (menaces marquantes)
* URL Filtering (filtrage d'URL)
* Blocked URLs (blocage d'URL)
* System Logs & Events
* Config Overview (vue d'ensemble de la configuration)

Par défaut, les tableaux de bord sont configurés pour le thème sombre.
Une fois installés, vous pouvez les changer pour le thème clair, ajouter/supprimer/réorganiser des visualisations individuelles ou créer vos propres tableaux de bord.
Les tableaux de bord peuvent également être configurés pour fonctionner en plein écran et rafraîchissement automatique, parfait pour les écrans de supervision.


## Contexte

Ce projet vise à fournir un moyen simple d'extraire et de visualiser les données syslog des pare-feu de Palo Alto.
Il utilise Elastic Stack (gratuit) de [www.elastic.co](https://www.elastic.co/elk-stack) comme plate-forme de données et de visualisation. Il fournit une configuration et des modèles d'index, pour les journaux suivants:

* Traffic
* Threat/URL
* Config
* System

Une suite complète de visualisations et de tableaux de bord est incluse.


**Elastic Stack**

[(from the site)](https://www.elastic.co/elk-stack): Qu'est-ce que ELK ? "ELK" est l'acronyme de trois projets open source : Elasticsearch, Logstash et Kibana. Elasticsearch est un moteur de recherche et d'analyse. Logstash est un pipeline de traitement de données côté serveur qui ingère des données provenant de plusieurs sources simultanément, les transforme, puis les envoie à une "stash" comme Elasticsearch. Kibana permet aux utilisateurs de visualiser les données à l'aide de tableaux et de graphiques dans Elasticsearch.

En bref, Elastic Stack (ELK) fournit une plate-forme simple, évolutive et robuste ingérant les entrées syslog d'un pare-feu Palo Alto (PANW) et affichant leurs sorties. La configuration requise pour LogStash & ElasticSearch est fournie ici, ainsi qu'un certain nombre de visualisations prédéfinies pour Kibana. L'interface Kibana vous permet de créer vos propres visualisations supplémentaires en toute simplicité. Toutes les visualisations de base de ce projet ont été construites en une seule journée.

Elastic Stack inclut également un serveur syslog intégré, ce qui simplifie grandement le déploiement de la solution comme un tout.

En utilisant uniquement le fichier de configuration d'Elastic Stack, nous avons tout ce qu'il faut pour une solution tout-en-un.

**Credit**

Une grande partie de ce projet a été créé sur la base des pages suivantes:
* [Shadow-Box's ELK template for Traffic & Threat Logs](https://github.com/shadow-box/Palo-Alto-Networks-ELK-Stack)
* [ELK + PALO ALTO NETWORKS](https://anderikistan.com/2016/03/26/elk-palo-alto-networks/) by [Ian Anderson](https://twitter.com/anderikistan)
* [PANW Firewall Visualisations using Elastic Stack](https://github.com/sm-biz/paloalto-elasticstack-viz)

=> Un grand merci pour leurs travail


## Tutoriel

Ce projet a été monté sur un Ubuntu 16.04 LTS, en utilisant la version d'Elastic Stack 6.1 (avec serveur syslog intégré) et un pare-feu PA-220.
nginx a été utilisé pour sécuriser l'authentification vers Kibana via reverse-proxy.

Pour ceux qui ne sont pas familiers avec une partie de la pile ELK, un tutoriel complet sur l'installation et la configuration d'Elastic Stack existe, incluant la sécurisation de la plate-forme et l'installation des visualisations.
Ce tutoriel créé par sm-biz (encore merci) est[disponible ici](https://github.com/sm-biz/paloalto-elasticstack-viz/wiki)


## Existing Install

Sinon, si vous êtes à l'aise avec la suite ELK, alors tout ce que vous avez à faire est:

- Télécharger les fichiers de ce repo
  - PAN-OS.conf
  - traffic_template_mapping-v1.json
  - threat_template_mapping-v1.json
  - searches-base.json
  - visualisations-base.json
  - dashboards-base.json

- Installer Elastic Stack 6.1
  - ElasticSearch
  - Kibana
  - LogStash
  
- Edit 'PAN-OS.conf'
  - **Définir votre timezone** *(Very important)*
  - Copier le fichier dans votre dossier **conf**. Pour Ubuntu/Debian il se trouve dans "/etc/logstash/conf.d/", les répertoires des autres  [available here](https://www.elastic.co/guide/en/logstash/current/dir-layout.html)

- Upload the two pre-built index templates with additional GeoIP fields
```
curl -XPUT http://<your-elasticsearch-server>:9200/_template/panos-traffic?pretty -H 'Content-Type: application/json' -d @traffic_template_mapping-v1.json
curl -XPUT http://<your-elasticsearch-server>:9200/_template/panos-threat?pretty -H 'Content-Type: application/json' -d @threat_template_mapping-v1.json
```    
- Restart Elastic Search & LogStash
- Configure your PANW Firewall(s) to send syslog messages to your Elastic Stack server
  - UDP 5514
  - Format BSD
  - Facility LOG_USER
  
- Ensure that your firewall generates at least one traffic, threat, system & config syslog entry each
  - You may have to trigger a threat log entry. Follow [this guide](https://live.paloaltonetworks.com/t5/Management-Articles/How-to-Test-Threat-Prevention-Using-a-Web-Browser/ta-p/62073) from Palo Alto for instructions
  - After committing to set your syslog server, you will need to do another committ (any change) to actually send a config log message
  
- Once the data is rolling, login to Kibana and create the 4 new index patterns, all with a Time Filter field of '@timestamp'
  - panos-traffic
  - panos-threat
  - panos-system
  - panos-config

- And lastly, import the saved object files (in this orders)
  - searches-base.json
  - visualisations-base.json
  - dashboards-base.json
  
And that's it! Once you have some logs in the system, you should see the dashboards start to fill up
  
 
## References

* [Elastic Stack Documentation:](https://www.elastic.co/guide/en/elasticsearch/reference/6.1/index.html) For more information on the Elastic Stack, including the official install guides
* [Basics of Visualisation:](https://www.elastic.co/guide/en/kibana/6.1/tutorial-visualizing.html) To get started making your own visualisations and dashboards
