# SSH

## OpenSSL

OpenSSL est une boîte à outils de chiffrement comportant deux bibliothèques, libcrypto et libssl, fournissant respectivement une implémentation des algorithmes cryptographiques et du protocole de communication SSL/TLS, ainsi qu'une interface en ligne de commande, openssl.

## SSH Définition

Secure Shell (SSH) est à la fois un programme informatique et un protocole de communication sécurisé. Le protocole de connexion impose un échange de clés de chiffrement en début de connexion. Par la suite, tous les segments TCP sont authentifiés et chiffrés. Il devient donc impossible d'utiliser un sniffer pour voir ce que fait l'utilisateur.

Le programme `ssh` permet de se connecter à un ordinateur en utilisant le protocole `ssh`.

## Port SSH

Habituellement le protocole SSH utilise le port TCP 22. Il est particulièrement utilisé pour ouvrir un shell sur un ordinateur distant.

L’ordinateur sur lequel on souhaite se connecter doit être un serveur ssh, il doit avoir un port SSH ouvert, le service correspondant activé et la connexion doit être autorisée. Certains hébergeurs de sites Internet autorisent la connexion ssh, d'autres non.

Il est tout à fait possible d'utiliser ssh sous Windows.

## Installation

Si ssh n'est pas déjà installé :

```bash
sudo apt install openssh-server -y
sudo apt install openssh-client -y
```

Sous Windows : voir dans `Programmes et fonctionnalités`.

## Syntaxe

```bash
# Syntaxe
ssh [ -p <port> ] [ -v ]  <utilisateur>@<machine> 
# Exemple
ssh ubuntu@192.168.1.65
```

> Pour pouvoir utiliser un nom de machine au lieu d'une adresse IP vous pouvez utiliser le fichier `hosts`, sous Linux `/etc/hosts`, sous Widnows `C:\Windows\System32\drivers\etc\hosts`

- `<utilisateur>` : nom de l'utilisateur sur la machine. Éviter `root`. Il faut avoir le mot de passe ou avoir configuré une paire de clé publique/privée.
- `<machine>` : nom ou adresse IP de la machine sur laquelle on souhaite se connecter.
- `<port>` : si nécessaire, numéro du port sur lequel se connecter
- `-v` : mode verbeux, affiche des informations. On peut utiliser `-vv` et `-vvv` pour avoir plus de détails.

## Connexion sans mot de passe

### Création clé publique / clé privée

Pour se connecter sans mot de passe, il faut créer une paire de clé publique privée.

La clé privée n'est connue que du client et doit rester privée. Toute personne possédant cette clé privée pourrait avoir accès au serveur.

La clé publique est connue du client et du serveur. Il y a un lien mathématique entre les deux clés.

1. Le client connaît sa clé publique et sa clé privée
1. Le serveur autorise une liste de clés publiques. Par exemple, la clé publique du client est ajoutée dans le fichier `authorized_keys` du serveur. Le serveur autorise tous les clients dont la clé publique est enregistrée.
1. Le client effectue un calcul à partir de la clé publique et privée et envoie cela au serveur
1. Le serveur utilise la valeur envoyée par le client pour vérifier si une des clés publiques qui sont autorisées correspond à cette valeur et si oui il autorise la connexion

Les bases mathématiques des clés publiques et privées sont assez solides, il est théoriquement impossible de retrouver une clé privée dans un temps raisonnable avec la puissance actuelle des ordinateurs.

Pour créer la paire de clés sur le client :

```bash
ssh-keygen -t ed25519 [ -M <passphrase> ] [ -f <nom_fichier> ]
```

Pour le nom du fichier, une suggestion : `id_ed25519_<nom_machine>`

- `-t ed25519` : clé utilisant l'algorithme ed25519 (plus récent et efficace  que `rsa`)
- `-M <passphrase>` : facultatif, permet de mieux sécuriser la clé
- `-f <nom_fichier>` : nom du fichier, typiquement dans le dossier `~/.ssh` (Linux) ou `%USERPROFILE%\.ssh` (Windows cmd).

Cette commande créé deux fichiers, une clé publique et une clé privée qui sont liées mathématiquement. Le client va présenter sa clé privée pour se connecter. Le serveur a une liste de clés publiques autorisées à se connecter. Si le serveur reconnaît la valeur envoyée par le client avec une clé connue, il autorise la connexion.

La clé publique du client doit être déposée sur le serveur. On peut utlilser la commande `ssh-copy-id` ou le faire manuellement :

1. Afficher le contenu de la clé publique : `cat <nom_fichier>.pub` ou `type <nom_fichier>.pub`
2. Copier la clé publique (copier/coller du système d’exploitation)
3. Se connecter au serveur. Ouvrir le fichier `~/.ssh/authorized_keys` (le créer si nécessaire et en ajustant les permissions `touch ~/.ssh/authorized_keys && chmod 600 ~/.ssh/authorized_keys`).
4. Ajouter une novelle ligne dans ce fichier en collant la clé publique.

### Ajout dans le fichier config

Sur le client, ouvrir le fichier `%USERPROFILE%\config` (ou `~/.ssh/config`). Créer ce fichier si nécessaire et ajuster les droits pour que seul l'utilisateur actuel puisse manipuler ce fichier.

Ajouter les lignes suivantes :

```txt
Host <nom>              
  User <user>
  Hostname <machine>
  IdentityFile <nom_fichier>
  IdentitiesOnly yes
```

```txt
# <nom>         : nom utilisé pour la connexion ssh
# <user>        : nom de l'utilisateur
# <machine>     : nom ou adresse IP de la machine
# <nom_fichier> : chemin complet de la clé privée
```

Exemples :

```txt
Host vm
  User ubuntu
  Hostname vm.mshome.net
  IdentityFile c:\Users\Michel\.ssh\id_ed25519_vm
  IdentitiesOnly yes

Host github.com
  Hostname ssh.github.com
  Port 443
  User micheldiemer
  IdentityFile c:\Users\Michel\.ssh\id_ed25519_github
  IdentitiesOnly yes
```

### Tester

```bash
ssh <nom>
# si ça ne fonctionne pas
ssh -vvv <nom>
```

## ssh-agent

`ssh-agent` est le programme qui gère les autorisations de connexion ssh. Il est possible par exemple de définir une `passphrase` pour vos clés les plus sensibles et les faire mémoriser par `ssh-agent` pour ne pas avoir à réécrire trop souvents la `passphrase`.

## Tunnel SSH

SSH peut également être utilisé pour transférer des ports TCP d'une machine vers une autre, créant ainsi un tunnel. Cette méthode est couramment utilisée afin de sécuriser une connexion qui ne l'est pas.

## Sources

- [Wikipedia fr SSH](https://fr.wikipedia.org/wiki/Secure_Shell)
- [Wikipedia fr OpenSSL](https://fr.wikipedia.org/wiki/OpenSSL)
- [ssh keygen & copy-id](https://www.thegeekstuff.com/2008/11/3-steps-to-perform-ssh-login-without-password-using-ssh-keygen-ssh-copy-id/)
