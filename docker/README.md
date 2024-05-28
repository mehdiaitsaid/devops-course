![alt text](image-1.png)

Docker est une plateforme logicielle qui permet de créer, déployer et gérer des applications dans des conteneurs. Les conteneurs sont des unités légères, portables et autonomes qui incluent tout le nécessaire pour exécuter une application : le code, les bibliothèques, les dépendances et les fichiers de configuration. Grâce à Docker, les développeurs peuvent standardiser leur environnement de développement, ce qui facilite grandement le déploiement et la mise à l'échelle des applications sur différents systèmes.

Lancé en 2013 par Docker Inc., Docker a rapidement gagné en popularité grâce à sa capacité à simplifier le processus de développement et de déploiement des logiciels. Avant Docker, les développeurs devaient souvent gérer des environnements complexes et variés, ce qui pouvait entraîner des problèmes de compatibilité et de configuration. Docker résout ces problèmes en fournissant un environnement cohérent et reproductible, ce qui améliore la collaboration entre les équipes de développement et d'exploitation.

Les principaux avantages de Docker incluent :

1. **Isolation des applications** : Chaque conteneur fonctionne de manière isolée, ce qui permet de s'assurer que les applications ne se chevauchent pas et n'interfèrent pas les unes avec les autres.
2. **Portabilité** : Les conteneurs Docker peuvent être exécutés de manière identique sur n'importe quel système prenant en charge Docker, qu'il s'agisse d'un ordinateur portable de développement, d'un serveur de production ou d'un cloud.
3. **Efficacité des ressources** : Les conteneurs partagent le même noyau du système d'exploitation, ce qui permet une utilisation plus efficace des ressources par rapport aux machines virtuelles traditionnelles.
4. **Déploiement et mise à l'échelle rapides** : Les conteneurs peuvent être créés, démarrés, arrêtés et répliqués rapidement, ce qui facilite le déploiement et la mise à l'échelle des applications.

## 1 — Installation de Docker

Le paquet d'installation de Docker disponible dans le dépôt officiel d'Ubuntu peut ne pas être la dernière version. Pour nous assurer d'obtenir la version la plus récente, nous allons installer Docker à partir du dépôt officiel de Docker. Pour ce faire, nous allons ajouter une nouvelle source de paquets, ajouter la clé GPG de Docker pour garantir la validité des téléchargements, puis installer le paquet.

Tout d'abord, mettez à jour votre liste de paquets existants :

```
sudo apt update
```

Ensuite, installez quelques paquets prérequis qui permettent à apt d'utiliser des paquets via HTTPS :

```
sudo apt install apt-transport-https ca-certificates curl software-properties-common
```

Ajoutez ensuite la clé GPG pour le dépôt officiel de Docker à votre système :

```
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
```

Ajoutez le dépôt Docker aux sources APT :

```
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

Mettez à jour à nouveau votre liste de paquets existants pour que l'ajout soit reconnu :

```
sudo apt update
```

Assurez-vous que vous allez installer à partir du dépôt Docker au lieu du dépôt par défaut d'Ubuntu :

```
apt-cache policy docker-ce
```

Vous verrez une sortie similaire à celle-ci, bien que le numéro de version de Docker puisse être différent :

```
Output
docker-ce:
  Installed: (none)
  Candidate: 5:20.10.14~3-0~ubuntu-jammy
  Version table:
     5:20.10.14~3-0~ubuntu-jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
     5:20.10.13~3-0~ubuntu-jammy 500
        500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages

```

Remarquez que docker-ce n'est pas installé, mais le candidat pour l'installation provient du dépôt Docker pour Ubuntu 22.04 (jammy).

Enfin, installez Docker :

```
sudo apt install docker-ce
```

Docker devrait maintenant être installé, le démon démarré, et le processus activé pour démarrer au boot. Vérifiez que Docker fonctionne :

```
sudo systemctl status docker
```

La sortie devrait être similaire à ce qui suit, montrant que le service est actif et en cours d'exécution :

```
Output
● docker.service - Docker Application Container Engine
     Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
     Active: active (running) since Fri 2022-04-01 21:30:25 UTC; 22s ago
TriggeredBy: ● docker.socket
       Docs: https://docs.docker.com
   Main PID: 7854 (dockerd)
      Tasks: 7
     Memory: 38.3M
        CPU: 340ms
     CGroup: /system.slice/docker.service
             └─7854 /usr/bin/dockerd -H fd:// --containerd=/run/containerd/containerd.sock

```

L'installation de Docker vous donne non seulement le service Docker (démon) mais aussi l'utilitaire en ligne de commande docker, ou le client Docker. Nous explorerons comment utiliser la commande docker plus tard dans ce tutoriel.

## 2 — Exécution de la Commande Docker Sans Sudo (Optionnel)
Par défaut, la commande docker ne peut être exécutée que par l'utilisateur root ou par un utilisateur du groupe docker, qui est automatiquement créé lors de l'installation de Docker. Si vous essayez d'exécuter la commande docker sans la préfixer par sudo ou sans être dans le groupe docker, vous obtiendrez une sortie comme celle-ci :

```
Output
docker: Cannot connect to the Docker daemon. Is the docker daemon running on this host?.
See 'docker run --help'.
```

Si vous souhaitez éviter de taper sudo chaque fois que vous exécutez la commande docker, ajoutez votre nom d'utilisateur au groupe docker :

```
sudo usermod -aG docker ${USER}
```

Pour appliquer la nouvelle appartenance au groupe, déconnectez-vous du serveur et reconnectez-vous, ou tapez la commande suivante :

```
su - ${USER}
```
Vous serez invité à entrer le mot de passe de votre utilisateur pour continuer.

Confirmez que votre utilisateur est maintenant ajouté au groupe docker en tapant :

```
groups
```

Si vous avez besoin d'ajouter un utilisateur au groupe docker qui n'est pas celui avec lequel vous êtes connecté, spécifiez ce nom d'utilisateur explicitement en utilisant :

```
sudo usermod -aG docker username
```

## 3 — Installation de Docker Compose

Pour vous assurer d'obtenir la version stable la plus récente de Docker Compose, vous téléchargerez ce logiciel à partir de son dépôt officiel sur Github.

Tout d'abord, confirmez la dernière version disponible sur leur page de versions. Au moment de la rédaction de ce document, la version stable la plus récente est la 2.3.3.

Utilisez la commande suivante pour télécharger :

```
mkdir -p ~/.docker/cli-plugins/
```
```
curl -SL https://github.com/docker/compose/releases/download/v2.3.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
```

Ensuite, définissez les permissions correctes pour que la commande docker compose soit exécutable :

```
chmod +x ~/.docker/cli-plugins/docker-compose
```

Pour vérifier que l'installation a réussi, vous pouvez exécuter :

```
docker compose version
```