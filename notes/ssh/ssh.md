# SSH

## Historique 

Le protocole SSH (*Secure Shell*) a été créé en 1995 par Tatu Ylönen pour remplacer des protocoles non sécurisés comme Telnet et rlogin, qui transmettaient les données en clair. La première version, SSH-1, a rapidement gagné en popularité, mais elle présentait des vulnérabilités cryptographiques qui ont conduit à son abandon. Pour y remédier, SSH-2 a été introduit en 2006 comme une norme ouverte (RFC 4251). Cette version a apporté des améliorations majeures : un chiffrement plus robuste, une meilleure intégrité des données et un mécanisme d’authentification plus flexible.

Aujourd’hui, SSH-2 (souvent simplement appelé SSH) est devenu un outil incontournable dans le monde de l’administration système, du développement logiciel et de l’infonuagique (*cloud computing*). Il est utilisé quotidiennement par les administrateurs pour gérer des serveurs à distance, automatiser des tâches via des scripts, ou encore sécuriser des communications entre services. Grâce à sa fiabilité et à sa sécurité, SSH est intégré dans la majorité des systèmes Linux et est également disponible sur Windows et macOS. Son rôle est central dans les pratiques modernes de DevOps, de gestion d’infrastructure et de cybersécurité. 

## Fonctionnement général 

Le protocole SSH fonctionne selon un modèle client-serveur. Lorsqu’un utilisateur souhaite se connecter à une machine distante, le client SSH initie une requête vers le serveur SSH. Ce dernier répond en envoyant sa clé publique, permettant au client de vérifier l’identité du serveur. Cette étape est cruciale pour éviter les attaques de type homme dans le milieu (*man-in-the-middle*). Une fois l’identité du serveur confirmée, les deux parties négocient un algorithme de chiffrement et établissent une session sécurisée à l’aide de clés de session temporaires. Ces clés sont utilisées pour chiffrer toutes les données échangées pendant la session. 

Après l’établissement du canal sécurisé, le serveur SSH procède à l’authentification de l’utilisateur. Celle-ci peut se faire de plusieurs façons : par mot de passe, par clé publique, par certificats, ou via des méthodes plus avancées comme l’intégration avec Kerberos (Active Directory) ou avec PAM et l’authentification à deux facteurs. L’authentification par clé publique ou par certificat est particulièrement sécurisée : l’utilisateur possède une paire de clés (publique et privée), et le serveur conserve une copie de la clé publique. Lors de la connexion, le serveur envoie un défi que seul le détenteur de la clé privée peut résoudre, prouvant ainsi son identité sans jamais transmettre la clé privée sur le réseau. 

![Connexion SSH](../assets/images/ssh-high-level.png)
*Diagramme traduit du site ssh.com*

## Possibilités offertes 

Une fois la connexion établie et l’utilisateur authentifié, SSH permet une variété d’opérations sécurisées. La plus courante est l’accès à une interface en ligne de commande sur la machine distante, permettant l’administration système, l’exécution de scripts ou la surveillance de services. SSH permet aussi le transfert de fichiers via les protocoles SCP (*Secure Copy*) et SFTP (*SSH File Transfer Protocol*), ainsi que la création de tunnels sécurisés pour rediriger du trafic réseau (*port forwarding*). Ces fonctionnalités font de SSH un outil polyvalent et indispensable dans les environnements informatiques modernes, notamment en DevOps, en cybersécurité et dans le *cloud*. 