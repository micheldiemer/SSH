# SSH

## Activer l'authentification par mot de passe

- `sudo vi /etc/ssh/sshd_config`
- `PasswordAuthentication yes`
- `sudo service ssh restart` | `sudo systemctl restart ssh.service`

## Autoriser root

- `sudo vi /etc/ssh/sshd_config`
- `PermitRootLogin yes`
- `sudo service ssh restart`
- `sudo passwd root` | `sudo systemctl restart ssh.service`

## Se connecter avec mot de passe

- `ssh -o PreferredAuthentications=password -o PubkeyAuthentication=no`
