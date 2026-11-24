# Annexe 2: Génération d’un certificat SSH 

## Partie 1: Génération de l’autorité de certification 

Si une autorité de certification a déjà été fournie, cette étape n’est pas nécessaire. Aller directement à la partie 2.  

Les commandes suivantes assument que l’autorité de certification est générée directement sur le serveur SSH. Si elle est générée sur un autre serveur, les étapes seront les mêmes, mais il faudra copier le certificat sur le serveur. 

1. Générer la paire de clés pour l’autorité de certification (CA) 

    ```bash
    ssh-keygen -t <rsa | ed25519> -b <taille de clé> -f <fichier> -C "<identifiant>" 
    ssh-keygen -t rsa -b 4096 -f ~/.ssh/ssh_ca -C "SSH CA" 
    ```
 

1. Vérifier les fichiers créés 

    - `~/.ssh/ssh_ca`: clé privée de l’autorité de certification 
    - `~/.ssh/ssh_ca.pub`: clé publique de l’autorité de certification 

1. Installer la clé publique de l’autorité de certification sur le serveur SSH 

    ```bash
    sudo mkdir -p /etc/ssh 
    sudo cp ~/.ssh/ssh_ca.pub /etc/ssh/ssh_ca.pub 
    ```
 
1. Activer l’authentification par certificats sur le serveur SSH 

    ```bash
    sudo nano /etc/ssh/sshd_config 
    ```
    ```bash
    TrustedUserCAKeys /etc/ssh/ssh_ca.pub 
    PubkeyAuthentication yes 
    AuthorizedPrincipalsFile /etc/ssh/authorized_principals/%u 
    ```

1. Ajouter les utilisateurs autorisés à se connecter au serveur 

    ```bash
    sudo mkdir -p /etc/ssh/authorized_principals 
    echo "johndoe" | sudo tee /etc/ssh/authorized_principals/johndoe 
    sudo chown root:root /etc/ssh/authorized_principals/johndoe 
    sudo chmod 644 /etc/ssh/authorized_principals/johndoe 
    ```
 
    {: .highlight}
    > remplacer “johndoe” par l’utilisateur désiré 

1. Redémarrer le service SSH 

    ```bash
    sudo systemctl restart sshd 
    ```

## Partie 2: Génération du certificat de l’utilisateur 

1. Sur le client, générer une paire de clés SSH pour l’utilisateur.  

    Cette étape est identique à la création des clés pour une authentification par clé publique. 

    ```bash
    ssh-keygen -t <rsa | ed25519> -b <taille de clé> -C "<identifiant>" 
    ssh-keygen -t ed25519 -C "johndoe@cmaisonneuve.qc.ca" 
    ```
1. Signer la clé publique de l’utilisateur avec la clé privée de l’autorité de certification. 

    Cette étape doit s’effectuer sur le serveur où l’autorité de certification a été créée. 

    ```bash
    ssh-keygen -s <clé privée du CA> -I <identifiant du certificat> -n <utilisateur autorisé> -V <période de validité> <clé à signer> 
    ssh-keygen -s ~/.ssh/ssh_ca -I johndoe-cert -n johndoe -V +52w ~/.ssh/id_ed25519.pub 
    ```

    Cette commande générera un certificat appelé `id_ed25519-cert.pub` 

1. Copier le fichier `id_ed25519-cert.pub` sur le client, et vérifier que les fichiers sont au bon endroit sur le client. 

    - `~/.ssh/id_ed25519` (clé privée) 
    - `~/.ssh/id_ed25519-cert.pub` (certificat signé) 

 

1. Pour une sécurité accrue, vérifier que les permissions strictes sont appliquées 

    ```bash
    chmod 600 ~/.ssh/id_ed25519 
    chmod 644 ~/.ssh/id_ed25519-cert.pub
    ```

1. Se connecter au serveur SSH 

    ```bash
    ssh -i <clé privée> -o CertificateFile=<certificat> <utilisateur>@<hôte ou IP du serveur> 
    ssh -i ~/.ssh/id_ed25519 -o CertificateFile=~/.ssh/id_ed25519-cert.pub johndoe@serveur_distant 
    ```
 
Si la connexion s’établit avec succès, l’authentification par certificat SSH a été configurée correctement. 