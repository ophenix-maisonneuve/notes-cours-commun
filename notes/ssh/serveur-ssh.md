# Serveur SSH 

## Configuration 

Afin de permettre l’accès à un serveur par SSH, il faut d’abord s’assurer que le serveur SSH est installé.  

Sous Debian: 

```bash
sudo apt install openssh-server 
```
 

Sous Alma Linux: 

```bash
sudo dnf install openssh-server 
```
 

Toutes les configurations du serveur SSH se retrouvent dans le fichier /etc/ssh/sshd_config. On remarque dans ce fichier la directive suivante: 

```bash
Include /etc/ssh/sshd_config.d/*.conf 
```
 
Il est donc possible de modifier les configurations directement dans le fichier `/etc/ssh/sshd_config`, mais pour obtenir une configuration plus modulaire et maintenable, il est aussi possible de créer un ou plusieurs fichiers `.conf` sous `/etc/ssh/sshd_config.d/`. 


### Directives essentielles 

#### Port et interface 

`Port 22` : Définit le port d’écoute (par défaut 22).

`ListenAddress 0.0.0.0` : Adresse IP sur laquelle le serveur écoute. 

#### Authentification 

`PasswordAuthentication yes|no` : Active/désactive l’authentification par mot de passe.

`PubkeyAuthentication yes|no` : Active/désactive l’authentification par clé publique.

`PermitRootLogin yes|no` : Permet/interdit la connexion directe en root (`no` recommandé). 

`AuthenticationMethods publickey,password` : Définit les méthodes autorisées (peut inclure 2FA). 

#### Sécurité et restrictions 

`UsePAM yes|no` : Active/désactive l’intégration avec PAM pour la gestion des politiques. 

`AllowUsers user1 user2` : Limite les utilisateurs autorisés. 

`AllowGroups sshusers` : Limite aux membres d’un groupe. 

`MaxAuthTries 3` : Nombre maximum de tentatives d’authentification. 

`LoginGraceTime 60` : Temps avant expiration de la tentative de connexion. 

#### Chiffrement et algorithmes 

`Ciphers aes256-gcm@openssh.com,chacha20-poly1305@openssh.com` : Liste des algorithmes de chiffrement autorisés. 

`MACs hmac-sha2-256,hmac-sha2-512` : Algorithmes d’intégrité. 

`KexAlgorithms curve25519-sha256,diffie-hellman-group-exchange-sha256` : Algorithmes d’échange de clés. 

#### Journalisation 

`LogLevel VERBOSE` : Niveau de détail des journaux (logs), utile pour les audits ou le diagnostic 