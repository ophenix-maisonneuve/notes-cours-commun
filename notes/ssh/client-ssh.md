# Client SSH 

## Accès à distance 

L'obtention d’un accès à un terminal sur un serveur distant est le principal cas d’utilisation de SSH. Cela permet à un utilisateur d’interagir avec un serveur comme s’il y était connecté physiquement, et ainsi d’avoir accès à toutes les commandes qui lui sont normalement disponibles sur ce serveur. Il suffit d’utiliser la commande suivante pour se connecter: 

```bash
ssh <utilisateur>@<adresse IP ou nom de domaine> 
ssh tux@serveur.maisonneuve.lan
``` 

## Transfert de fichiers 

SSH offre deux façons distinctes d’échanger des fichiers entre la machine locale et le serveur distant: SCP (*Secure Copy Protocol*), qui ressemble à la commande `cp`, et SFTP (*SSH File Transfer Protocol*) qui s’apparente beaucoup à FTP. 

### SCP 

SCP permet de copier des fichiers ou des répertoires de la machine locale vers le serveur distant, ou du serveur distant vers la machine locale. 

Pour copier de la machine locale vers le serveur distant: 

```bash
scp <fichier ou répertoire> <utilisateur>@<serveur>:</chemin/destination/> 
scp fichier.txt tux@serveur.maisonneuve.lan:/home/tux/ 
```
 

Pour copier du serveur distant vers la machine locale: 

```bash
scp <utilisateur>@<serveur>:</chemin/vers/fichier> </destination_locale> 
scp tux@serveur.maisonneuve.lan:/home/tux/fichier.txt ./tmp 
```

### SFTP 

SFTP s’utilise de la même façon que FTP, mais est complètement sécurisé à l’aide de SSH. La plupart des clients FTP, par exemple FileZilla, supportent SFTP et fournissent donc une interface graphique conviviale pour transférer des fichiers entre la machine locale et le serveur distant. Il existe également un outil en ligne de commande: 

```bash
sftp <utilisateur>@<serveur> 
sftp tux@serveur.maisonneuve.lan 
```

Une fois connecté, il est possible d’utiliser les commandes suivantes pour naviguer dans l’arborescence et transférer des fichiers: 

`cd`: changer de répertoire sur le serveur distant 

`lcd`: changer de répertoire sur la machine locale 

`ls`: lister le contenu du répertoire courant sur le serveur distant 

`lls`: lister le contenu du répertoire courant sur la machine locale 

`put`: téléverser un fichier de la machine locale verse le serveur distant 

`get`: télécharger un fichier du serveur distant vers la machine locale 

## Tunnels SSH 

SSH offre également la possibilité de rediriger le trafic réseau entre le serveur distant et la machine locale dans un tunnel sécurisé. Cette fonctionnalité est souvent appelée tunnels SSH, ou redirection de ports SSH (SSH port forwarding). Il existe deux principaux types de tunnels SSH: 

### Tunnel local (*local forwarding*) 

Ce type de tunnel permet de lier un port sur la machine locale à un service s’exécutant sur un serveur distant. Il est particulièrement utile dans le cas où la machine locale n’a pas accès directement au service distant, mais a un accès SSH à un serveur qui, lui, y a accès.  


![Redirection de port local](../assets/images/ssh-local-port-forward.png)
*Tunnel local (diagramme de techtarget.com)* 

 
```bash
ssh -L <port_local>:<serveur distant>:<port distant> <utilisateur>@<serveur> 
```
 

Un exemple assez courant serait, par exemple, une machine ayant accès à un serveur web, mais pas directement au serveur de base de données qui supporte le serveur web. Comme le serveur web a, lui, accès à la BD, il serait possible d’y accéder à partir de la machine locale en créant un tunnel local comme ceci: 

 
```bash
ssh -L 3306:serveur_bd:3306 tux@serveur-web 
```
 

La BD serait ensuite disponible au localhost:3306. 



### Tunnel distant (*remote forwarding*) 

Ce type de tunnel permet d’exposer un service s’exécutant sur la machine locale sur un hôte distant, de manière à permettre à tous les clients ayant accès à ce serveur distant d’accéder au service.  

 
![Redirection de port local](../assets/images/ssh-remote-port-forward.png)
 *Tunnel distant (diagramme de techtarget.com)* 

 

 ```bash
ssh -R <port distant>:<hôte dest.>:<port dest.> <utilisateur>@<serveur> 
```
 

Cette technique est fréquemment utilisée pour exposer temporairement un service qui n’est pas encore déployé en production (par exemple, une application web en cours de développement). En créant un tunnel distant, il est possible d’exposer le service sur un serveur disponible publiquement afin qu’une personne puisse s’y connecter. Une fois le tunnel détruit, le service local n’est plus accessible. La commande suivante est utilisée pour créer ce type de tunnel: 

```bash
ssh -R 8888:localhost:8080 tux@serveur-web 
```

Le serveur web écoutant sur le port 8080 local serait alors disponible sur le serveur serveur-web sur le port 8888. Autrement dit, toute connexion sur serveur-web:8888 serait redirigée vers le port 8080 de la machine locale. 

{: .highlight}
> Pour que le service soit accessible pour tous, et non seulement pour la machine locale, on doit activer l’option GatewayPorts dans la configuration SSH: 

```bash 
>> sudo nano /etc/ssh/sshd_config 
```
```bash
GatewayPorts yes 
```