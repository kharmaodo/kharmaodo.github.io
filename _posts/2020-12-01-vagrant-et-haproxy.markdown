---
layout: post
title: Vagrant et HAProxy - Mise en place d'un Load Balancer avec la virtualisation
date: 2020-12-01 00:00:00 +0000
categories: [DevOps, Industrialisation, Virtualisation]
tags: [Vagrant, HAProxy, VirtualBox, Load Balancing]
author: Maodo DIOP
---

Si les questions suivantes vous sont familières :

* Comment simplifier les tâches de déploiements d'applications dans plusieurs serveurs avec le même stack d'outils ?
* Ou ne plus répéter les tâches d'installations des outils pour l'infrastructure de développement ou de la production ?
* Ou bien ne plus entendre la fameuse phrase « Pourtant cela marche sur mon environnement/machine » de la part des collaborateurs.
* Comment fournir un stack préfabriqué de l'entreprise pour les nouveaux collaborateurs pour leur simplifier les tâches d'installations et/ou de configuration ?
* Comment configurer un arsenal de serveurs avec plus ou moins les mêmes caractéristiques ?

Si ces genres de questions vous concernent, alors la virtualisation et ses outils référents sont le remède de votre souci. Un des outils permettant de résoudre votre problème est **Vagrant**.

Vagrant s'adresse aux développeurs, aux devops, aux geeks etc…

Wikipédia cite : « Vagrant est un logiciel libre et open-source pour la création et la configuration des environnements de développement virtuel. Il peut être considéré comme un wrapper autour de logiciels de virtualisation comme VirtualBox. »

L'on peut citer d'autres outils de virtualisation en l'occurrence VMware, Hyper-V, Xen, KVM, Oracle VM, Proxmox VE, oVirt etc.

Parce qu'elle utilise le principe de virtualisation, il est donc compatible avec la majorité des systèmes d'exploitation (OS). C'est-à-dire qu'un utilisateur travaillant sous Mac aura accès au même environnement qu'un développeur sous Windows ou Linux tout en conservant les mêmes stacks.

Dans la suite, l'on utilisera **VirtualBox** comme outils de virtualisation avec Vagrant car ce dernier est en réalité une surcouche des solutions de virtualisation. Il faudra noter aussi que Vagrant n'est pas un outil de *provisioning* qui gère les dépendances logicielles de la machine virtuelle (VM).

Les solutions de *provisionning* par ordre de priorité sont pour les débutants l'usage des scripts shell puis enfin l'utilisation des solutions prouvées comme : Ansible, Puppet, Solo, Chef.

## Vagrant repose sur la trilogie suivante :

### 1. Un fichier d'entrée Vagrantfile

* Écrit en Ruby, ce fichier de configuration principale va servir à définir le dossier racine du projet avec quelques options de configurations. Les options de base de configurations seront :
    * Le type de machine souhaité (Ubunty, Centos, Debian etc…) et dans le vocabulaire de Vagrant c'est la **box**.
    * La configuration réseau (nat, dhcp, ip fixe etc…)
    * Le *provider* qui est soit un script shell et/ou les outils tels Ansible, Puppet etc.

### 2. Les boxes

* Ceux sont la plupart du temps des OS (Ubuntu entre autres…) ou des applications préinstallées prêtes à l'emploi pour accélérer la construction de l'environnement virtuel.

### 3. Un ou plusieurs providers

* Vagrant ne virtualisant pas directement les environnements fait appel à un *provider* pour accomplir cette tâche. VirtualBox étant la plus utilisée pour la plupart du temps mais VmWare gagne aussi du terrain.

## Un cas d'exemple universel : « Hello World » à la sauce Vagrant

Créer un répertoire que l'on nommera `lab-vagrant` et démarrons un petit projet vagrant en exécutant la commande :

```bash
λ vagrant init
````

```
A `Vagrantfile` has been placed in this directory. You are now
ready to `vagrant up` your first virtual environment! Please read
the comments in the Vagrantfile as well as documentation on
`vagrantup.com` for more information on using Vagrant.
```

Lister le contenu de ce répertoire :

```bash
λ ls
Vagrantfile
```

Cette commande crée un fichier de configuration de nom `Vagrantfile` dont le contenu est commenté. Nous allons modifier ce fichier en supprimer les commentaires et le personnaliser.

Générerons notre VM en exécutant la commande ci-dessous :

```bash
λ vagrant up
```

**Explication du code en rapport avec ce qui est plus haut.**

Nous avons configuré notre environnement avec le système d'exploitation `hashicorp/precision32` avec sa version `1.0.0`. Celui-ci est une *box* dans le jargon de Vagrant, nous pourrions pu utiliser un Ubuntu ou un Debian ou CentOS pourquoi pas ?

```ruby
config.vm.box = “hashicorp/precise32”
config.vm.box_version = “1.0.0”
```

Notre logiciel de virtualisation étant VirtualBox, nous définissons une mémoire octroyée et l'absence d'interface ergonomique.

```ruby
config.vm.provider “virtualbox” do |vb|
  vb.gui = false
  vb.memory = “1024”
end
```

Puis en dernier, l'affichage d'un message une fois que le *provisioning* est terminé par les citations des auteurs (Shakespeare et La Fontaine).

Pour les besoins de simplicité, nous appelons un script-line avec l'aide des possibilités du Shell en lieu et place des outils de *provisionning* comme Puppet, Salt, Chef, Solo, Ansible etc…

## Commandes Vagrant fréquemment utilisées

| Action | Commande |
| :--- | :--- |
| Initialiser une box | `Vagrant init <nom-vm>` |
| Ajouter une box | `vagrant box add distrib/version` |
| Lister les box | `vagrant box list` |
| Générer un(e) VM | `vagrant up` |
| Connexion en SSH | `vagrant ssh <nom-vm>` |
| Éteindre la VM | `vagrant halt` |
| Détruire la VM | `vagrant destroy` |
| Arrêter la VM | `Vagrant suspend` |

Avec un `vagrant ssh`, l'on peut faire ce que l'on veut sur sa VM en terme d'installations de paquets, de personnalisation (clavier, heure…)

```bash
vagrant@vm1hello:~$ uname -a
Linux vm1hello 3.13.0–32-generic #57~precise1-Ubuntu SMP Tue Jul 15 03:50:54 UTC 2014 i686 i686 i386 GNU/Linux
vagrant@vm1hello:~$ whoami
vagrant
vagrant@vm1hello:~$ sudo apt-get update
# ... Logs de l'update ...
vagrant@vm1hello:~$ sudo apt-get install -y git
# ... Logs de l'installation de git ...
```

-----

## Vagrant et HAProxy comme Load Balancing

Le *load balancing* est une technique qui consiste à répartir les charges sur différents appareils d'un même réseau. Elle permet notamment aux serveurs de sites Internet à forte audience de ne pas se retrouver surchargés. Dans le *load balancing*, les multiples requêtes sont distribuées sur plusieurs serveurs.

Techniquement, chaque élément qui compose le système a un rôle :

1.  L'internet à travers lequel l'utilisateur accède.
2.  Le serveur DNS, chargé de faire la jonction entre le nom de domaine et l'adresse IP du serveur qui l'héberge.
3.  Le *loadbalancer* a pour but de transmettre les charges sur différentes ressources.

Pour ce billet, nous allons utiliser Vagrant pour faire du *load balancing* avec **HAProxy**.

Nous allons utiliser trois (3) instances avec les caractéristiques suivantes :

| Instance | Hostname | OS / Rôle | Private IP |
| :--- | :--- | :--- | :--- |
| **Instance 1** | `haproxy` | Ubuntu / Load Balancer | `192.168.205.30` |
| **Instance 2** | `webserver1` | Ubuntu + Apache / Web Server 1 | `192.168.205.10` |
| **Instance 3** | `webserver2` | Ubuntu + Apache / Web Server 2 | `192.168.205.20` |

Sur les deux derniers serveurs (`webserver1`, `webserver2`), Apache y est installé servant de tests pour le *load balancing* servi par HAProxy.

### Configuration des Instances via Vagrant et Shell Scripts

Pour configurer nos instances, nous définissons pour chaque instance l'appel à un script shell pour l'installation et la configuration :

#### `webserver.sh`

Ce script installe le paquet Apache 2, ajoute le nom du serveur dans le fichier de configuration d'Apache et redémarre Apache.

Le fichier `index.html` est modifié pour qu'il affiche selon l'algorithme utilisé par le *loadbalancer* pour l'élection des serveurs :

  * Soit `Webserver1` / `Je suis votre serviteur !`
  * Ou `Webserver2` / `Je suis votre serviteur !`

#### `haproxy.sh`

Pour la configuration HAProxy, nous avons les étapes suivantes :

1.  Mise à jour du gestionnaire des paquets : `apt-get update -y -qq`
2.  Installer HAProxy : `apt-get install -y haproxy`
3.  Activer HAProxy comme *daemon* au démarrage :
    ```bash
    cat > /etc/default/haproxy <<EOF
    ENABLED=1
    EOF
    ```
4.  Configuration des serveurs pour le *load balancing* avec l'algorithme **roundrobin** :
    ```conf
    server webserver1 192.168.50.10:80 check
    server webserver2 192.168.50.20:80 check
    ```
5.  Accéder à l'interface web des statistiques de HAProxy :
      * `stats enable`
      * `stats auth admin:admin`
      * `stats uri /haproxy?stats`
6.  Valider la configuration : `haproxy -f /etc/haproxy/haproxy.cfg -c`
7.  Démarrer l'utilitaire HAProxy : `service haproxy start`

### Test de l'Infrastructure

Pour tester notre infrastructure, lancer les commandes suivantes :

```bash
vagrant up
```

Nous pouvons nous connecter à chaque instance en SSH :

```bash
vagrant ssh webserver1
vagrant ssh webserver2
vagrant ssh haproxy
```

En appelant le serveur (par l'IP du load balancer), l'on est servi suivant la disponibilité de `webserver1` et/ou `webserver2`.

Nous pouvons tester en mode web et avoir les réponses suivant la disponibilité des serveurs.

HAProxy offre l'interface d'administration des serveurs. Il suffit de se pointer sur `http://192.168.50.30/haproxy?stats`.

Ici nous remarquons que tous nos serveurs sont au vert. Essayer d'éteindre le premier serveur à savoir `webserver1`.

```bash
λ vagrant halt webserver1
==> webserver1: Attempting graceful shutdown of VM…
```

Et nous aurons l'interface HAProxy qui prend en compte l'état actuel des serveurs.

-----

## Liens Utiles

  * Boxes Vagrant : [http://www.vagrantbox.es/](https://www.google.com/search?q=http://www.vagrantbox.es/)
  * HAProxy Ubuntu : [https://upcloud.com/community/tutorials/haproxy-load-balancer-ubuntu/](https://www.google.com/search?q=https://upcloud.com/community/tutorials/haproxy-load-balancer-ubuntu/)
  * HAProxy Wikipedia : [https://en.wikipedia.org/wiki/HAProxy](https://en.wikipedia.org/wiki/HAProxy)

<!-- end list -->

```

---
