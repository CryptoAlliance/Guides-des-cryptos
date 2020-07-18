##### Table des matières

- [Ajouter une ipv6 (optionnel)](#Ajouter-une-ipv6-optionnel)
	- [OVH](#OVH)
- [Ajouter un utilisateur](#Ajouter-un-utilisateur)
	- [Ajouter le groupe sudo à votre utilisateur](#Ajouter-le-groupe-sudo-à-votre-utilisateur)
- [Interdire l'accès ssh à root](#Interdire-l'-accès-ssh-à-root)
	- [Edition de la configuration du service sshd](#Edition-de-la-configuration-du-service-sshd)
	- [Redémarrer le service sshd](#Redémarrer-le-service-sshd)
- [Activation du firewall](#Activation-du-firewall)
- [Mise à jour du système](#Mise-à-jour-du-système)
- [Ajouter un espace swap (optionnel)](#Ajouter-un-espace-swap-optionnel)
- [Redémarrer le système](#Redémarrer-le-système)

Ce guide à pour but de vous aider à réaliser quelques configurations de base sur le VPS livré par votre fournisseur avec Ubuntu server 16.04. La documentation officielle se trouve à cette [adresse](https://help.ubuntu.com/16.04/serverguide/index.html "adresse") (certaines parties sont en français et d'autres en anglais).

Tout d'abord, [connectez vous en ssh à votre serveur](https://kinsta.com/fr/blog/se-connecter-via-ssh/ "connectez vous en ssh à votre serveur") avec l'utilisateur root et le mot de passe fourni par votre fournisseur.

# Ajouter une ipv6 (optionnel)
## OVH
OVH fourni une ipv6 mais ne la configure pas par défaut. Commencons par désactiver la configuration réseau d'ovh.

Nous allons créer un nouveau fichier.

`nano /etc/cloud/cloud.cfg.d/99-disable-network-config.cfg`

L'éditeur texte s'ouvre. A l'intérieur encodez ceci :

> network: {config: disabled}

Enregistrez le fichier avec la combinaison de touche [ctrl] + [o] puis validez avec [enter]. Vous pouvez quitter l'éditeur avec la combinaison [ctrl] + [x].

Supprimez le fichier /etc/network/interfaces.d/50-cloud-init.cfg. Celui-ci est automatiquement généré par le système d'ovh.

`rm -f /etc/network/interfaces.d/50-cloud-init.cfg`

Créons une nouveau fichier /etc/network/interfaces.d/ens3.cfg qui va contenir notre configuration réseau. ens3 correspond au nom de notre interface réseau.

`nano /etc/network/interfaces.d/ens3.cfg`

A l'intérieur, encodez ce qui suit, en remplaçant les valeurs des paramètres address et gateway par les données fournies par OVH (adresse ipv6 et adresse du routeur ipv6).

> auto ens3:0  
> iface ens3:0 inet dhcp  
> auto ens3:1  
> iface ens3:1 inet6 static  
> address 2001:41d0:0305:120d:0450:0000:0000:315b  
> netmask 64  
> gateway 2001:41d0:0305:120d:0450:0000:0000:0001  

Enregistrez le fichier ([ctrl] + [o] puis [enter]) et quittez l'éditeur ([ctrl] + [x]).

Rechargez la configuration réseau.

`systemctl restart networking`

Si vous tapez ce qui suit, vous devriez voir vos adresses ipv4 et ipv6.

`ifconfig`

Vous pouvez également utiliser un [outil en ligne](https://www.google.com/search?client=firefox-b-ab&q=ipv6+ping "outil en ligne") pour réaliser un ping en ipv6 et vérifier que votre VPS est bien joignable via votre adresse.

# Ajouter un utilisateur

L'objectif est de ne plus utiliser l'utilisateur root (c'est l'équivalent de l'administrateur sous Windows, il a tout les pouvoirs).

Pour ajouter un utilisateur, taper la commande ci-dessous.

`adduser le_nom_de_votre_utilisateur`

Il va vous être demandé d'introduire un mot de passe et ensuite de le répéter. Vous n'être pas obligé de répondre aux questions qui suivent, vous pouvez appuyer sur la touche [enter] sans rien encoder. Répondez “y” à la dernière question (ou “o” si votre système est en français).

> root@vps6248783:~# adduser bob  
> Adding user `bob' …  
> Adding new group `bob' (1002) …  
> Adding new user `bob' (1002) with group `bob' …  
> The home directory `/home/bob' already exists. Not copying from `/etc/skel'.  
> Enter new UNIX password:  
> Retype new UNIX password:  
> passwd: password updated successfully  
> Changing the user information for bob  
> Enter the new value, or press ENTER for the default  
> Full Name []:  
> Room Number []:  
> Work Phone []:  
> Home Phone []:  
> Other []:  
> Is the information correct? [Y/n] y  

## Ajouter le groupe sudo à votre utilisateur

Maintenant nous allons donner à notre nouvel utilisateur le droit d'exécuter n'importe quelle commande comme s'il était l'utilisateur root. Par facilité, on va appeler cet utilisateur “bob”.

Nous devons éditer le fichier /etc/group contenant tous les groupes du systèmes ainsi que leurs utilisateurs membres. Pour se faire, nous utilisons la commande "nano".

`nano /etc/group`

Une fois le fichier ouvert, utilisez les flèches de votre clavier pour vous rendre au à la ligne contenant ceci :
> sudo:x:27:ubuntu

Il se peut que le chiffre ne soit pas la même, ceci n'a pas d'importance, il s'agit juste du numéro de groupe, ignorez le. Sur la même ligne ajoutez, “,bob”. Vous devez avoir ceci.
> sudo:x:27:ubuntu,bob

Enregistrez votre modification en appuyant sur les touches [ctrl] + [o] puis validez avec [enter]. Quittez l'éditeur avec la combinaison [ctrl] + [x].

Vous pouvez maintenant clôturer votre session ssh avec la commande suivante.
`exit`

# Interdire l'accès ssh à root

Connectez vous en ssh, mais avec bob cette fois-ci.

Si vous avez accès à votre serveur à distance, c'est grace à un service qui tourne sur celui-ci appelé "sshd" (dans notre cas il s'agit du serveur openssh). Nous allons modifier la configuration de ce service pour interdire à l'utilisateur root de se connecter en ssh.

## Edition de la configuration du service sshd

Editez le fichier /etc/ssh/sshd_config.
`sudo nano /etc/ssh/sshd_config`

Deux choses ont du vous interpeller. Premièrement, la commande “sudo” devant la commande “nano” et deuxièmement le fait qu'il vous est demandé d'introduire un mot de passe. Le fichier que nous voulons éditer peut normalement être modifié uniquement par l'utilisateur root. La commande “sudo” est en fait l'équivalent sous Windows de “exécuter en tant qu'administrateur”. Vous êtes membre du groupe du même nom que cette commande (sudo) ce qui vous donne le droit de l'utiliser. Encodez maintenant le mot de passe de l'utilisateur bob.

Recherchez la ligne contenant le paramètre “PermitRootLogin” (vous pouvez utiliser la combinaison [ctrl] + [w] pour réaliser une recherche automatique dans le texte) :
> \#PermitRootLogin prohibit-password

La valeur du paramètre peut être différente de “prohibit-password”, ce n'est pas grave car vous devez remplacer la valeur par “no”. Enlevez également le caractère dièse qui commence la ligne, celui-ci sert à dire au logiciel qu'il ne doit pas tenir compte de cette ligne (or ce n'est pas ce que nous voulons). Vous devez obtenir ceci :
> PermitRootLogin no

Enregistrez la modification ([ctrl] + [o] puis [enter]) et quittez l'éditeur ([ctrl] + [x]).

## Redémarrer le service sshd

`sudo systemctl restart sshd`

Le service est redémarré avec la nouvelle configuration.

# Activation du firewall

Ubuntu 16.04 est livré avec un firewall qui s'appelle ufw (Uncomplicated Firewall), cependant celui-ci n'est pas activé.

Par défaut, ufw bloque tout ce qui vient de l'extérieur. Avant de l'activer nous allons ajouter une règle pour autoriser les connexions au service sshd.

Exécutez la commande suivante.

`sudo ufw limit ssh/tcp`

Nous pouvons maintenant activer le firewall.

`sudo ufw enable`

Le système vous prévient que les connexions ssh existantes peuvent être affectées. Ne vous tracassez pas et acceptez.

> bob@vps6238453:~$ sudo ufw enable  
> Command may disrupt existing ssh connections. Proceed with operation (y|n)? y  
> Firewall is active and enabled on system startup  

Vous pouvez vérifier que tout est bon avec la commande ci-dessous.

`sudo ufw status`

> bob@vps6238453:~$ sudo ufw status  
> Status: active  
>   
> To Action From  
> – —— —-  
> 22/tcp LIMIT Anywhere  
> 22/tcp (v6) LIMIT Anywhere (v6)  

# Mise à jour du système

Les logiciels installés sur le système livré par votre fournisseur ne sont pas forcément à jour. Nous allons commencer par mettre à jour la liste des logiciels disponibles (il s'agit en fait de paquets mais je simplifie volontairement l'explication).

`sudo apt-get update`

Et maintenant mettons à jour les logiciels déjà installés sur le système.

`sudo apt-get upgrade`

Il devrait trouver des logiciels à mettre à jour et vous les proposer. Répondez "y".

# Ajouter un espace swap (optionnel)

La swap est un espace sur votre disque dur qui est utilisé par le système dans le cas où la mémoire RAM est pleine. Il s'agit d'une solution de secours (la mémoire RAM étant bien évidemment beaucoup plus efficace). Certains fournisseurs comme OVH ne livre pas les VPS avec un espace swap déjà configuré.

Vous pouvez vérifier son existence ou non avec cette commande :

`swapon --show`

Si elle ne renvoi rien, c'est qu'aucune swap n'est montée, vous pouvez donc continuer ce chapitre.

Commençons par créer le fichier d'échange swap (l'espace en question qui sera utilisé par le système). Le “2G” signifie que la swap se réservera 2G d'espace sur le disque dur. Vous pouvez modifier cette taille à votre guise.

`sudo fallocate -l 2G /swapfile`

Modifions les permissions sur ce fichier pour que seul l'utilisateur root puisse y accéder.

`sudo chmod 600 /swapfile`

Transformons le fichier créé en espace swap.

`sudo mkswap /swapfile`

Activez la swap avec la commande suivante.

`sudo swapon /swapfile`

Vérifiez la présence de la swap sur le système comme réalisé précédemment.

`sudo swapon --show`

Vous devriez avoir quelque chose comme ceci :

> NAME TYPE SIZE USED PRIO  
> /swapfile file 2G 29.3M -1  

Cette configuration sera perdue en cas de redémarrage. Pour la rendre permanente, nous allons ajouter une ligne dans le fichier /etc/fstab. Il s'agit du fichier listant toutes les partitions que le système doit monter au démarrage.

`sudo echo "/swapfile none swap sw 0 0" >> /etc/fstab`

# Redémarrer le système

Il s'agit de la dernière étape de ce guide. Il n'est pas toujours nécessaire de redémarrer le système mais par précaution je vous conseille de le faire avant de vous lancer dans l'installation d'un masternode ou autres logiciels. La connexion ssh va se couper.

`sudo shutdown -r 0`

> bob@vps6238453:~$ sudo shutdown -r 0  
> Shutdown scheduled for Sat 2018-12-15 01:31:52 CET, use 'shutdown -c' to cancel.  

------------

[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png "Creative Commons License")](http://creativecommons.org/licenses/by-sa/4.0/ "Creative Commons License")
Cette œuvre est mise à disposition selon les termes de la Licence [Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0 International](https://creativecommons.org/licenses/by-sa/4.0/deed.fr "Licence Creative Commons Attribution - Partage dans les Mêmes Conditions 4.0 International").
