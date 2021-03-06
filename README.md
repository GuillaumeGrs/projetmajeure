
# PROJET DE MAJEURE - Virtualisation & Cloud

#### équipe 1 - VIGNAUD - MEUNIER - GRESLE

***

*Introduction* : L'objectif de ce projet est d'utiliser différentes fonctionnaliés de VMware vSphere afin de déployer un cloud privé. Le data center de CPE est utilisé comme infrastucture sur laquelle le cluster de virtualisation sera utilisé. L'architecture globale, ainsi que notre plan d'adressage sont représentés sur le schéma suivant:

![plan-intro](https://github.com/GuillaumeGrs/projetmajeure/blob/master/PLAN_INTRO.PNG)

***

1. *Installation et configuration ESXi :*

Dans un premier temps, nous devons créer deux hyperviseurs ESXi (type 1) qui auront pour rôle d'orchestrer l'exécution des différentes machines virtuelles et de leur allouer les ressources nécessaires en fonction de leurs besoin.

* Nous avons tout d'abord récupéré les images nécessaires (ESXi 7 et Windows Server 2012) qui étaient présentes dans le stockage *lunlibrary* du datacenter.

* Nous avons ensuite configuré les hyperviseurs ESXi, premièrement,  nous avons configuré les disques durs en mode *Thin* afin de pouvoir mutualiser les ressources entre tous les groupes

* Ensuite nous avons mis 4 adaptateurs réseau sur les ESXi afin de se connecter aux quatre réseaux de l'infrastructure. Dans un premier temps, un standard switch par port sera mis en place. Les ports attribués à notre groupe ont les IDs 101,102,103.

* Le nom de domaine pour notre groupe est : *4eti-01.tpv.cpe.localdomain* 

* Le plan d'adressage utilisé pour le premier ESXi est le suivant, les adresses Ip statiques ont été configurées de la manière suivante:

![plan](https://github.com/GuillaumeGrs/projetmajeure/blob/master/Capture.JPG)




2. *Configuration du stockage NAS*

* Un serveur NAS a été mis en place avec l'outil TrueNAS son adresse sur le réseau de management est 192.168.17.62/24. L'interface réseau NAS qui nous a été attribuée sur le réseau de stockage est 172.16.1.1/24 (em2). Il s'agit d'un réseau de stockage partagé qui nous servira pour la suite, cette interface em2 est connectée sur notre réseau de stockage. On attribue le stockage à nos deux ESXi via un adaptateur ISCSI qui "écoute" le NAS sur l'interface 172.16.1.254 pour ESXi1 et 172.16.1.253 pour ESXi2.


3. *Mise en place du contrôleur de domaine*

* Sur ESXi1, nous avons créé une première machine virtuelle sous Windows Serveur 2012, sur laquelle nous avons installé les rôles et les fonctionnalités suivantes pour installer un controleur de domaine:
                                     - DNS (la recherche inversé a été mise en place + pointeur PTR) 
                                     - AD DS
                                     - DHCP (pour attribuer les addresses IP aux machines sur le réseau Prod)

* A présent l'active directory est mis en place, des utilisateurs "classiques" sont créés. Avec la mise en place de l'ensemble AD DS et DNS, il est possible de se connecter à n'importe quel utilisateur ou administrateur sur n'importe quelle machine cliente (qui tournent également sous Windows Serveur). Pour cela, les machines clientes ont été ajoutées dans notre domaine 4eti-01.tpv.cpe.localdomain.
    
    
4. *Déploiement de vCenter*

* Dans cette partie, nous avons déployé vCenter afin de pouvoir gérer nos machines virtuelles et nos machines hôtes directement depuis celui-ci. Un cluster a été créé et les ESXi ont été configurés. Nous avons quelques difficultés sur cette partie (l'installation de vCenter) qui nous a fait perdre du temps: nous avions connecté la VM cliente sur laquelle vCenter est déployé sur le réseau de management, donc la mauvaise adresse de DNS était jointe (celle de l'école et pas celle de notre Windows Serveur).
* Ensuite, une fois vCenter déployé, nous avons migré les VMs sur le premier ESXi afin de mettre le second en maintenance pour sa configuration, et vis-versa. Le cluster est ainsi créé. Le DRS permet de maîtriser la répartion des charges sur les ESXi du cluster: les VMs sont migrées automatiquement de l'un à l'autre selon la consommation des ressources. Pour accéder 
* L'objectif suivant était de créer un unique Distributed Switch qui vient remplacer les standard vSwitch créés précédement. Pour cela 3 nouveaux groupes de ports sont créés: un pour le réseau stockage, pour le réseau Prod et un dernier pour Vmotion: ils sont ajoutés au Distributed Switch. Les ports des VMs de ces réseaux sont aussi migrés.
* Une nouvelle machine Linux a été déployée, iso linux sur esxi et pas sur vcenter image.

5. *vSphere 7.0 Update 2 ESXi Upgrade*

* A ce stade du projet, à partir d'une image iso, on met à jour la version VMware vSphere vers une version plus récente, la 7.0.2. Cela est réalisé dans vCenter dans lequel l'image est importée, puis une "baseline upgrade" est créée: cela nous permet alors d'appliquer la mise à jour sur notre cluster et tout est géré automatiquement pendant celle-ci (maintenance, migration des VMs...)

6. *Déploiement du NAT réseau Prod*

* Les machines virtuelles clientes sont connectées à un seul réseau: seulement celui de Prod. Or ces machines doivent aussi avoir accès à internet: pour cela une translation d'adresse IP doit avoir lieu. Dans les paramètres d'accès à distance, le réseau de Prod est déclaré en tant qu'interne et celui de management en externe sur notre Windows Serveur. Malgré les paramètrages, cette partie n'est pas encore fonctionnelle.


*Conclusion* : Ce projet nous a permis d'apprendre énormément sur les outils réseaux, l'administration système et l'environnement VMware. Même si nous avons manqué de temps pour mettre en place toute les fonctionnalités prévues, notre cloud privé est fonctionnel.



