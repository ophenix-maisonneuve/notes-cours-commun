# Annexe 1: Génération d’une paire de clés SSH 

1. Sur le client, générer une paire de clés SSH 

    ```bash
    ssh-keygen -t <rsa | ed25519> -b <taille de clé> -C "<identifiant>" 
    ssh-keygen -t ed25519 -C "johndoe@cmaisonneuve.qc.ca"

    Enter file in which to save the key (/home/johndoe/.ssh/id_ed25519):
    ```

    Appuyer sur Entrée pour accepter le nom par défaut 

    ```bash
    Enter passphrase (empty for no passphrase): 
    ```
    Fournir une phrase secrète, ou simplement appuyer sur Entrée pour ne pas utiliser de phrase secrète 
 

1. Vérifier les fichiers créés 

    - `id_ed25519` : clé privée (ne jamais partager !) 
    - `id_ed25519.pub` : clé publique (à copier sur le serveur) 

 

1. Copier la clé publique sur le serveur SSH 

    ```bash
    ssh-copy-id -i <clé publique> <utilisateur>@<hôte ou ip du serveur> 
    ssh-copy-id -i ~/.ssh/id_ed25519.pub johndoe@serveur_ssh 
    ```
 
1. Se connecter au serveur SSH 

    ```bash
    ssh <utilisateur>@<hôte ou ip du serveur> 
    ssh johndoe@serveur_ssh 
    ```
 

Si la connexion s’établit avec succès, l’authentification par clé publique a été configurée correctement. 