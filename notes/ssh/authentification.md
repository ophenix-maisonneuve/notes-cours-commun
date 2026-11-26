# Authentification 

## Authentification du client 

### Mot de passe 

Avec **SSH**, l’authentification par mot de passe est l’un des mécanismes fondamentaux permettant à un utilisateur de prouver son identité lorsqu’il tente d’établir une connexion sécurisée à un serveur distant. Lorsqu’un utilisateur souhaite se connecter à un serveur via SSH, le client SSH (souvent un terminal ou une application dédiée) initie une requête de connexion vers le serveur. Le serveur, après avoir reçu cette requête, demande une authentification. Si l’authentification par mot de passe est activée sur le serveur, il invite l’utilisateur à entrer son mot de passe. Ce mot de passe est ensuite transmis sur le canal de communication chiffré grâce aux algorithmes négociés dans la phase de négociation cryptographique (voir section précédente). 

Contrairement aux connexions non sécurisées (comme Telnet), SSH chiffre toutes les données échangées, y compris le mot de passe. Cela signifie que même si un attaquant intercepte le trafic réseau, il ne pourra pas lire le mot de passe sans casser le chiffrement, ce qui est extrêmement improbable avec les algorithmes modernes comme AES. Malgré tout, l’authentification par mot de passe est généralement considérée comme moins sécuritaire que d’autres options, comme les clés publiques et les certificats. En effet, la sécurité repose aussi sur la robustesse du mot de passe choisi. Un mot de passe faible ou facile à deviner pourrait être compromis par des attaques par force brute ou par dictionnaire, rendant toute la sécurité du protocole SSH inopérante. 

Sur le serveur SSH (souvent géré par **sshd** sous Linux), l’authentification par mot de passe peut être activée ou désactivée via le fichier de configuration `/etc/ssh/sshd_config` à l’aide de la directive `PasswordAuthentication`. Pour renforcer la sécurité, les administrateurs peuvent désactiver cette option et privilégier des méthodes plus robustes. De plus, des outils comme **fail2ban** peuvent être utilisés pour bloquer les adresses IP après plusieurs tentatives de connexion échouées, réduisant ainsi les risques d’attaques automatisées. 

L’authentification par mot de passe est simple à mettre en oeuvre et ne nécessite pas de configuration complexe côté client. Elle est donc idéale pour des environnements temporaires ou des utilisateurs peu expérimentés. Cependant, elle présente des limites importantes en termes de sécurité et ne permet pas aux outils d’automatisation (scripts, Ansible, Puppet, Chef, etc) d’établir une connexion sans l’intervention d’un administrateur. 

Pour sécuriser l’authentification par mot de passe, il est recommandé d’utiliser des mots de passe longs, complexes, et uniques. L’ajout d’une couche d’authentification à deux facteurs (2FA) peut également renforcer la sécurité. Toutefois, dans les environnements professionnels ou critiques, il est préférable d’utiliser l’authentification par clé publique ou par certificats, qui reposent sur la cryptographie à clé publique et éliminent le besoin de transmettre un mot de passe. Ces méthodes sont non seulement plus sécurisées, mais aussi plus adaptées aux besoins grandissants d’automatisation sur les serveurs. 

### Clé publique 

L’authentification par clé publique est la méthode sécurisée la plus utilisée en entreprise pour établir une connexion entre un client et un serveur sans avoir à transmettre un mot de passe. Elle repose sur la cryptographie asymétrique, ce qui la rend beaucoup plus robuste que l’authentification par mot de passe.

L’authentification par clé publique utilise une paire de clés : une clé privée et une clé publique. La clé privée est conservée secrètement sur le poste client, tandis que la clé publique est copiée sur le serveur distant dans le fichier `~/.ssh/authorized_keys` de l’utilisateur cible. Lorsqu’une connexion SSH est initiée, le serveur utilise la clé publique pour générer un défi cryptographique que seul le détenteur de la clé privée peut résoudre. Si le client réussit à répondre correctement, l’authentification est validée. 

Lors d'une connexion SSH avec authentification par clé publique : 

1. Le client envoie une requête de connexion au serveur. 
1. Le serveur vérifie s’il possède une clé publique correspondant à l’utilisateur. 
1. Il envoie un défi (challenge) chiffré avec cette clé publique. 
1. Le client utilise sa clé privée pour déchiffrer le défi et le renvoie au serveur. 
1. Si le serveur valide la réponse, la connexion est établie sans mot de passe. 

Ce processus garantit que ni un mot de passe, ni la clé privée ne sont transmises sur le réseau, ce qui élimine le risque d’interception. 

L’authentification par clé publique présente plusieurs avantages : 

- **Sécurité accrue** : La clé privée reste sur le client et n’est jamais transmise. Elle peut aussi être protégée par une phrase secrète (passphrase). 
- **Automatisation** : Elle permet des connexions sans intervention humaine, idéales pour les scripts, les outils d’automatisation et/ou les tâches planifiées. 
- **Résistance aux attaques** : Elle est beaucoup moins vulnérable aux attaques par force brute ou par dictionnaire que l’authentification par mot de passe. 
- **Contrôle granulaire** : L’administrateur peut autoriser ou révoquer l’accès en ajoutant ou supprimant des clés dans les fichiers `authorized_keys`. 

Pour utiliser cette méthode, l’utilisateur doit générer une paire de clés avec une commande comme : 

```bash
ssh-keygen -t <rsa|ed25519> [-b <taille de la clé>] -C "<adresse_courriel>" 
ssh-keygen -t rsa -b 4096 -C "tux@maisonneuve.lan" 
```

Cela crée deux fichiers : 

- `id_rsa` : la clé privée (à garder secrète) 
- `id_rsa.pub` : la clé publique (à copier sur le serveur) 

La clé publique est ensuite ajoutée au fichier `~/.ssh/authorized_keys` sur le serveur, par exemple avec la commande suivante:  

```bash
ssh-copy-id [-i <clé publique>] <utilisateur>@<serveur> 
ssh-copy-id -i ~/.ssh/id_rsa.pub tux@serveur.maisonneuve.lan 
```

Pour renforcer la sécurité : 
- Protéger la clé privée avec une phrase secrète (passphrase) 
- Stocker la clé privée dans un répertoire sécurisé (`~/.ssh`) avec des permissions strictes. 
- Utiliser des clés de type ED25519 ou RSA 4096 bits pour une sécurité accrue. 
- Désactiver l’authentification par mot de passe sur le serveur (`PasswordAuthentication no`) une fois les clés en place. 

### Certificat SSH 

L’authentification par certificats SSH est une méthode avancée qui permet de gérer l’accès aux serveurs de manière plus centralisée et sécurisée que les clés publiques classiques. Elle repose sur l’utilisation de certificats numériques signés par une autorité de certification (CA), ce qui facilite la gestion des identités et des permissions dans des environnements complexes ou à grande échelle. 

Dans une authentification SSH par clé publique, chaque utilisateur génère une paire de clés, et la clé publique est copiée manuellement sur chaque serveur. Cela devient difficile à gérer lorsque le nombre d’utilisateurs ou de serveurs augmente. Les certificats SSH, en revanche, sont des clés publiques signées par une autorité de certification SSH. Le certificat contient des métadonnées comme l’identité de l’utilisateur, la durée de validité, et les permissions. Le serveur n’a pas besoin de connaître chaque clé publique individuellement, mais seulement de faire confiance à l’autorité de certification (Certificate Authority ou CA). 

Les principales étapes requises pour générer un certificat puis l’utiliser pour se connecter à un serveur sont: 

1. Un utilisateur génère une paire de clés SSH.
1. La clé publique est envoyée à une autorité de certification SSH. 
1. La CA signe cette clé et retourne un certificat SSH. 
1. L’utilisateur utilise ce certificat pour se connecter au serveur. 
1. Le serveur vérifie que le certificat est signé par une autorité de certification de confiance et que le certificat est encore valide (période de validité). 
1. Si tout est conforme, la connexion est autorisée. 

Ce mécanisme permet de révoquer ou limiter l’accès sans avoir à modifier les fichiers authorized_keys sur chaque serveur. Parmi les autres avantages, notons: 

- **Gestion centralisée** : Une seule autorité peut gérer l’accès à un grand nombre de serveurs. 
- **Expiration automatique** : Les certificats peuvent être valides pour une durée limitée (par exemple 24h), ce qui réduit les risques en cas de compromission. 
- **Révocation facile** : Il suffit de ne plus signer les certificats ou de mettre à jour la liste des CA autorisées. 
- **Flexibilité** : Les certificats peuvent contenir des options comme des restrictions de commande, des adresses IP autorisées, ou des rôles. 

OpenSSH prend en charge les certificats depuis la version 5.4. Voici les étapes principales pour utiliser l’authentification par certificats SSH : 

1. Création de l’autorité de certification (CA) :  

    ```bash
    ssh-keygen -f ca_key -C "CA SSH" 
    ```
    Cela génère une paire de clés pour signer les certificats. 

2. Signature d’une clé utilisateur :  

    ```bash
    ssh-keygen -s ca_key -I utilisateur1 -n tux -V +52w idrsa.pub
    ```
    Cela crée un certificat signé id_rsa-cert.pub. 

3. Configuration du serveur (le serveur doit connaître la clé publique de la CA) : 

    ```bash
    sudo nano /etc/ssh/sshd_config
    ```
    ```bash
    TrustedUserCAKeys /etc/ssh/ca_key.pub
    ```

4. Connexion avec le certificat (l’utilisateur utilise sa clé privée et le certificat) : 

    ```bash
    ssh -i id_rsa tux@serveur.maisonneuve.lan 
    ```
 

Les certificats SSH sont particulièrement utiles dans : 

- Les grandes infrastructures avec des centaines ou milliers de serveurs. 
- Les environnements cloud ou DevOps, où les accès doivent être dynamiques et sécurisés. 
- Les organisations qui souhaitent automatiser la gestion des accès sans compromettre la sécurité. 

### Autres méthodes d’authentification 

#### Kerberos 

L’authentification SSH avec Kerberos utilise le protocole GSSAPI (*Generic Security Services Application Program Interface*), qui permet à SSH d’utiliser les tickets Kerberos pour authentifier les utilisateurs sans mot de passe ni clé SSH. Lorsqu’un utilisateur est déjà authentifié dans un domaine Kerberos (par exemple via Active Directory ou un autre serveur Kerberos), le client SSH peut utiliser le ticket de session Kerberos pour prouver son identité au serveur SSH. Cela permet une authentification unique (*SSO*), où l’utilisateur n’a pas besoin de saisir ses identifiants à chaque connexion. 

Cette méthode est utile dans les environnements d’entreprise ou universitaires où Kerberos est déjà déployé pour gérer les identités. Elle renforce la sécurité en centralisant l’authentification et en évitant la gestion manuelle des clés SSH. Pour que cela fonctionne, le serveur SSH doit être configuré pour accepter l’authentification GSSAPI, et les horloges des machines doivent être synchronisées (Kerberos étant sensible à la dérive temporelle). Bien que puissante, cette méthode nécessite une infrastructure Kerberos bien configurée et une gestion rigoureuse des tickets et des politiques de sécurité. 

#### PAM 

L’authentification SSH via PAM (*Pluggable Authentication Modules*) permet à SSH de déléguer le processus d’authentification à un système modulaire géré par Linux. Plutôt que de se limiter aux méthodes internes comme les mots de passe ou les clés, SSH peut utiliser PAM pour intégrer des mécanismes d’authentification externes tels que les mots de passe stockés dans `/etc/shadow`, les modules biométriques, les jetons OTP (*One-Time Password*), ou encore des systèmes d’authentification centralisés comme LDAP. Cela offre une grande flexibilité et permet de personnaliser les politiques de sécurité selon les besoins de l’organisation. 

Lorsqu’un utilisateur tente de se connecter via SSH, si PAM est activé (`UsePAM yes` dans le fichier `sshd_config`), le serveur SSH passe la demande d’authentification au système PAM. Celui-ci applique les règles définies dans `/etc/pam.d/sshd`, qui peuvent inclure des vérifications supplémentaires comme la complexité du mot de passe, la présence d’un second facteur, ou autres restrictions. PAM permet également de gérer les sessions, les droits d’accès, et les audits, ce qui renforce la sécurité des connexions SSH dans les environnements Linux. 

 
## Authentification du serveur 

L’authentification du serveur SSH par le client est également important pour éviter les attaques de l’homme du milieu, aussi appelée attaque de l’intercepteur (*Man-in-the-Middle attack* en anglais). 

Lorsque le client SSH se connecte à un serveur, il doit s’assurer que ce serveur est bien celui auquel il veut accéder. Pour cela, SSH utilise un mécanisme basé sur des clés hôte (*host keys*). Chaque serveur SSH possède une paire de clés (privée et publique) qui l’identifie de manière unique. La clé privée reste sur le serveur, tandis que la clé publique est transmise au client. 

Lors de la première connexion, le client reçoit la clé publique du serveur et l’enregistre dans le fichier `~/.ssh/known_hosts`. À chaque connexion suivante, le client compare la clé reçue avec celle stockée. Si elles correspondent, la connexion continue normalement. Si elles diffèrent, SSH affiche un avertissement indiquant un risque d’attaque MITM.  

Les clés hôte peuvent être de différents types : 

- RSA (ssh-rsa) 
- ECDSA (ecdsa-sha2-nistp256) 
- ED25519 (ssh-ed25519)  

Ces clés sont utilisées pour signer les messages d’authentification envoyés par le serveur. Le client vérifie la signature avec la clé publique connue, garantissant que le serveur détient bien la clé privée associée 

Pour une sécurité accrue : 

- Utiliser des clés modernes comme ED25519 ou RSA 4096 bits. 
- Activer la directive HostKeyAlgorithms dans `sshd_config` pour limiter les algorithmes acceptés. 
- Surveiller les alertes SSH et ne jamais ignorer aveuglément un avertissement de clé modifiée. 