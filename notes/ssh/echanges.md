# Sécurisation des échanges 

La sécurisation des échanges de messages avec SSH repose sur une phase critique appelée négociation cryptographique, qui se déroule juste après l’échange des versions du protocole entre le client et le serveur. Cette phase permet aux deux parties de s’accorder sur les algorithmes à utiliser pour sécuriser la communication. Elle commence par l’échange de listes d’algorithmes supportés par chacune des deux parties pour le chiffrement symétrique, l’intégrité des données (HMAC), l’échange de clés et l’authentification. Le protocole SSH choisit les algorithmes les plus sécurisés communs aux deux parties. À ce stade, les messages sont encore en clair, ce qui permet d’observer la phase de négociation à l’aide d’outils comme Wireshark. 

Une fois les algorithmes choisis, SSH procède à l’échange de clés via des mécanismes pré-établis. Il s’agit souvent de dérivés de la méthode de Diffie-Hellman (DH), comme Diffie-Hellman sur courbe elliptique (*Elliptic Curve Diffie-Hellman* ou *ECDH*). Ces algorithmes permettent aux deux parties de générer une clé de session partagée sans jamais la transmettre directement sur le réseau. Cette clé est ensuite utilisée pour chiffrer tous les échanges à l’aide d’un algorithme symétrique comme AES (en mode CTR ou GCM) ou ChaCha20-Poly1305, selon ce qui a été négocié. En parallèle, un algorithme de hachage comme HMAC-SHA2 ou Poly1305 est utilisé pour garantir l’intégrité des données. Dès que cette phase est terminée, le canal SSH devient entièrement chiffré, et toutes les données échangées sont protégées contre l’interception et la falsification. 

## Chiffrement 

Lors de la phase de négociation cryptographique, les algorithmes suivants peuvent être considérés par le client et le serveur. Certains algorithmes ont été ajoutés à la liste pour des raisons historiques, mais ne sont plus utilisés en pratique, car on leur préfère des algorithmes plus sécuritaires ou plus performants, ou les deux. 

### AES (Advanced Encryption Standard) 
Modes utilisés dans SSH : aes128-ctr, aes192-ctr, aes256-ctr et aes128-gcm, aes256-gcm. 

**Caractéristiques**
- Algorithme très robuste, standardisé par NIST. 
- Les modes CTR (Counter) offrent un chiffrement par flux rapide et sécurisé. 
- Les modes GCM (Galois/Counter Mode) combinent chiffrement et authentification, réduisant le risque d’attaques par modification. 

**Avantages** : Très sécurisé, largement supporté, accélération matérielle disponible sur la plupart des CPU. 
**Inconvénients** : Peut être plus lent sur des systèmes sans support matériel. 

### ChaCha20-Poly1305 

**Caractéristiques** 

- Algorithme moderne conçu pour la performance et la sécurité. 
- Combine ChaCha20 (chiffrement par flux) et Poly1305 (authentification).
- Résistant aux attaques par minutage (timing), idéal pour les systèmes sans AES-NI. 

**Avantages** : Rapide sur CPU sans accélération AES, sécurité élevée. 
**Inconvénients** : Moins répandu que AES, mais recommandé par OpenSSH pour certaines plateformes. 

### Autres algorithmes 
Les algorithmes suivants étaient utilisés dans le passé, mais ont pour la plupart été désactivés par défaut dans les versions récentes de SSH. 

#### 3DES (Triple DES) 

**Caractéristiques** 
Ancien algorithme basé sur DES, appliqué trois fois pour renforcer la sécurité. 

**Mode utilisé** : 3des-cbc. 
**Avantages** : Historiquement fiable. 
**Inconvénients** : Obsolète, lent, vulnérable à certaines attaques (mode CBC), désactivé par défaut dans les versions récentes d’OpenSSH. 

#### Blowfish et CAST128 

**Caractéristiques** 
Algorithmes plus anciens, utilisés dans les premières versions de SSH. 

**Modes utilisés** : blowfish-cbc, cast128-cbc. 
**Avantages** : Simples et rapides à l’époque. 
**Inconvénients** : Désuets, moins sécurisés que AES ou ChaCha20, mode CBC vulnérable. 

#### RC4 

**Caractéristiques** 
Algorithme par flux très rapide. 

**Inconvénients** : Considéré comme non sécurisé aujourd’hui, retiré des configurations modernes. 
 

En résumé, pour chiffrer les communications, les versions récentes d’OpenSSH privilégient : 

- ChaCha20-Poly1305, ou 
- AES-GCM pour les environnements avec accélération matérielle.  

Les anciens algorithmes (3DES, Blowfish, RC4) sont dépréciés et ne devraient plus être utilisés. 

## Intégrité et authentification 

Après la phase de chiffrement, SSH doit garantir que les données transmises n’ont pas été modifiées en cours de route. Pour cela, chaque paquet SSH inclut un code d’authentification de message (Message Authentication Code ou MAC) calculé avec une clé secrète et une fonction de hachage. Ce MAC assure : 

**Intégrité** : détecter toute modification des données. 
**Authentification** : confirmer que le message provient bien de la partie légitime (car seul le client et le serveur connaissent la clé). 

Les algorithmes MAC généralement utilisés dans SSH sont : 

### HMAC-SHA2 
Basé sur SHA-256 ou SHA-512 pour assurer l’intégrité via un condensé (hash) 

**Variantes** : hmac-sha2-256, hmac-sha2-512. 
**Avantages** : Très sécurisé, recommandé. 
**Statut** : Standard actuel. 

### UMAC-64 et UMAC-128  
Algorithmes MAC basés sur des fonctions universelles de hachage. Plus rapides que HMAC sur certaines plateformes. 
**Statut** : Moins répandus. 

### Poly1305  
Utilisé en combinaison avec ChaCha20 (chacha20-poly1305). Fournit chiffrement et authentification en une seule opération. 

**Avantages** : Très performant et sécurisé. 
**Statut** : Alternative populaire à HMAC. 

En résumé, SSH utilise principalement l’un des 3 algorithmes d’authentification de messages (MAC) suivants: 

- **HMAC** : basé sur une clé symétrique et une fonction de hachage. 
- **Poly1305** : authentification intégrée au chiffrement (AEAD). 
- **UMAC** : optimisé pour la performance, mais moins courant.  