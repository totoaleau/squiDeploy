# :octopus: SquiDeploy-Master :octopus:

## :dart: Objectif 

Déploiement automatique d'infrastructure d'Honeypots avec un serveur ELK permettant la collecte et traitement des logs dans un SIEM. 
- ***SIEM (Security Information and Event Management)*** : *solution qui permet de surveiller, détecter et d’alerter concernant des événements ou incidents de sécurité dans un environnement IT. Il fournit une vue globale et centralisée en matière de posture de sécurité d’une infrastructure IT. Il donne aux professionnels de la cybersécurité un aperçu des activités au sein de leur environnement IT.*
- ***Honeypot ("pot de miel")*** : *concept mis au point par des experts en sécurité informatique dont le but de piéger les pirates lors d'attaques et d'étudier leur comportement pour mieux s’en protéger, en plus de leur restreindre l’accès à tout le système en production.*


## :floppy_disk: Composants 

- **VirtualBox** : hyperviseur d’Oracle permettant de créer, importer, exporter et gérer des VMs. 
- **Vagrant** : solution de HaschiCorp pour le déploiement industrialisé d’environnements virtualisés. Vagrant permet, entre autres, le déploiement de machines virtuelles standardisées. La préparation, le packaging des VM se fait dans des fichiers de configuration appelés « Box ». Ces derniers sont ensuite transmis au « Provider » qui s’occupera de l’exécution de la (ou des) VM (dans notre cas VirtualBox).
- **ElasticSearch** : moteur de recherche et d’analytique basé sur la bibliothèque Apache Lucene développé par Elastic NV. ElasticSearch récupère et stock les données sous une forme similaire au No-SQL (Schema-less JSON). Cette forme de stockage permet un traitement et une analyse plus souple et performante. 
- **Kibana** : solution de visualisation des données sous forme de dashboard. Kibana a été développé par Elastic NV pour travailler en tandem avec ElasticSearch.
- **FileBeat** : agent développé par Elastic NV qui s’installe directement sur les endpoints (serveurs, PC, etc). L’agent collecte les logs et les informations demandés et les transfère à LogStash ou, dans notre cas, à ElasticSearch.
- **T-Pot** : plateforme de honeypot mettant à disposition plusieurs services à simuler. Les services sont sous forme de conteneurs.

## :factory: Installation 

### Étape 1 : Installer VirtualBox
Télécharger l’Hyperviseur (ou Provider) [VirtualBox](https://www.virtualbox.org/wiki/Downloads). Il suffit de cliquer sur l’OS qui hébergera VirtualBox (dans notre cas Windows).
Une fois téléchargé, lancez l’exécutable et suivez l’installation. 
*Gardez à l’esprit que T-Pot a besoin d’un accès internet, donc installez et autorisez les fonctionnalités de VirtualBox permettant de gérer le réseau virtuel.*

### Étape 2 : Installer Vagrant
Télécharger l’exécutable permettant l’installation de [Vagrant](https://www.vagrantup.com/downloads). Suivez les étapes de l’installation normalement.

### Étape 3 : Récupérer notre répo GitHub
Cloner notre repo [GitHub](https://github.com/totoaleau/squiDeploy) pour obtenir les dossiers et fichiers nécessaires à la réalisation de l’environnement : <code> git clone https://github.com/totoaleau/squiDeploy </code>

### Étape 4 : Configurer le serveur ELK
*Pour commencer, nous allons configurer le serveur ELK qui permet de centraliser la collection de logs, leur traitement et l’affichage du résultat sur des dashboards dans Kibana.*
- Il faut se rendre dans le dossier <code>/ELK</code> du répo précédemment téléchargé puis ouvrir le fichier <code>« Vagrantfile »</code>. Commençons par indiquer l’IP que prendra cette machine en modifiant la **ligne 42**.   
 ![Explications](/images/7.png)
 
- Ensuite on indique le nombre de vCPU et la quantité de RAM à allouer au futur serveur ELK. La modification s’effectue aux **lignes 61 et 62**.   
*Nos ressources étant limitées, nous avons alloué 4 vCPU et 8192 Mo de RAM.*  
 ![Explications](/images/8.png)

- On sauvegarde nos modifications, on quitte le fichier et on ouvre le fichier <code>elasticsearch.yml</code> dans le dossier <code>/ELK</code>. A la **ligne 56**, on attribue de nouveau l’IP du serveur ELK alloué dans le fichier Vagrantfile puis on le ferme en sauvegardant.   
 ![Explications](/images/9.png)
 
- Toujours dans le dossier <code>/ELK</code>, ouvrons le fichier <code>jvm.options</code> pour indiquer la RAM max à attribuer par JVM (**lignes 31 et 32**). Les JVM sont des composantes de ElasticSearch gérant, notamment, la capacité de traitement. Nous conseillons de mettre la moitié de la RAM qui avait été définie dans le Vagrantfile.   
*Par exemple, si le serveur ELK se voit attribué 8Go de RAM, alors on maximisera la RAM des JVM à 4Go.*  
En termes de syntaxe, pour 4Go attribués, on l’écrira : <code>-Xms4g et -Xmx4g</code>. Le changement effectué, on sauvegarde et ferme le fichier.   
 ![Explications](/images/10.png)
 
- On ouvre alors le fichier <code>kibana.yml</code>.     
A la **ligne 7**, on indique à Kibana l’IP qu’il se verra attribuer.   
 ![Explications](/images/11.png)    
A la **ligne 32**, on indique à Kibana l’IP du serveur ElasticSearch qui fournira les informations à afficher. Le changement fait, on peut sauvegarder et quitter.   
 ![Explications](/images/12.png)
 
- Enfin, on ouvre le fichier <code>filebeat.yml</code> pour indiquer les IP du serveur Kibana (**ligne 104**) et Elasticsearch (**ligne 131**).   
 ![Explications](/images/13.png)   
 ![Explications](/images/14.png)    

### Étape 5 : Lancer le serveur ELK
Avec Vagrant installé, il suffit de se placer dans le dossier <code>/ELK</code> et de lancer la commande <code>« vagrant up »</code>. Vagrant utilisera tous les fichiers du dossier ELK pour créer la VM (en utilisant notamment une box Debian déjà présente dans Vagrant) et configurer les services.   
*Nota Bene : la présence de box permet de remplacer l’ISO pour la création de VM.*

### Étape 6 : Configuration du honeypot
Nous avons donc un serveur ELK virtualisé capable de récupérer et traiter des données (avec ElasticSearch) et d’afficher les informations résultantes du traitement (avec Kibana).   
Il est donc temps de configurer une VM T-Pot qui émettra des données pour collection et traitement.   
- Pour cela, on entre dans le dossier <code>/T-Pot</code> et modifions le fichier <code>Vagrantfile</code> à la **ligne 42** en allouant une adresse IP au futur honeypot.   
- Avec la même logique que pour le serveur ELK, on indique la RAM et le nombre de vCPU à attribuer à la machine aux **lignes 477 et 478** (un minimum de 8192 Mo de RAM et 4 CPUs est recommandé).   
 ![Explications](/images/15.png)   
 ![Explications](/images/16.png)   
- Les modifications effectuées, on sauvegarde et quitte pour ouvrir le fichier <code>filebeat.yml</code>.   
On modifie les **lignes 125 et 152** pour indiquer les IPs et les ports de Kibana et ElasticSearch.   
 ![Explications](/images/17.png)  
 ![Explications](/images/18.png)   

### Étape 7 : Lancer le honeypot
Même logique que pour ELK, on se dirige dans le dossier T-Pot et on lance un <code>« vagrant up »</code>.   
« And voilà », nous avons déployé sur un réseau un honeypot ainsi qu’un serveur ELK.   
Happy monitoring !   


## :rocket: Pour aller plus loin :
Dans cette démonstration, nous n’avons déployé qu’un seul T-Pot.   
- Si on désire en déployer plus, il suffit de copier le dossier T-Pot et de faire des changements au niveau de l’IP de la machine.       
Pour un déploiement industrialisé et automatique, préparez les dossiers à l’avance puis exécutez un script qui effectue des « vagrant up » dans chaque dossier de VM.   
- A la base, le projet devait être développé en utilisant Terraform pour la partie IaaC, et OpenStack pour la virtualisation. Cependant, OpenStack est très difficile à utiliser correctement dans sa version DevStack. Il a donc fallu trouver une alternative moins consommatrice.   
Terraform et OpenStack serait très pratique pour déployer cette infrastructure dans le Cloud, ce qui serait une très bonne suite au projet.   
- Dans l'état du projet, les logs des honeypots sont récupérés telquels par le serveur ELK, Kibana n'effectue aucun traitement et aucun tableau de bord n'est fourni par défaut. Améliorer cette présentation de la donnée en proposant des tableaux de bords déjà fonctionnel serait intéressant pour simplifier la surveillance de l'activité des honeypots.
- Pré-configurer des alertes pur prévenir de la détection d'activité par les honeypots serait aussi un plus.  


## :bust_in_silhouette: Team members 
- COURTONNE Emilie 
- DESSEAUX Thomas 
- TOURRET Romain
