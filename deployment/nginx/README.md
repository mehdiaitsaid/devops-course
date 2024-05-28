![alt text](image.png)

Nginx (prononcé "engine-x") est un serveur web open source très populaire, apprécié pour sa haute performance, sa stabilité et sa faible consommation de ressources. Développé par Igor Sysoev en 2004, Nginx a été initialement conçu pour résoudre le problème de gestion de plusieurs connexions simultanées. Aujourd'hui, il est largement utilisé non seulement comme serveur web, mais aussi comme proxy inverse, équilibreuse de charge et cache HTTP.

Nginx se distingue par sa capacité à gérer efficacement des milliers de connexions simultanées avec une utilisation minimale de la mémoire. Grâce à son architecture événementielle et non bloquante, il offre une performance supérieure, particulièrement pour les sites à fort trafic. De plus, Nginx est hautement configurable et extensible, permettant aux administrateurs de personnaliser son comportement pour répondre aux besoins spécifiques de leurs applications.

Cette section explorera les fonctionnalités clés de Nginx, y compris son installation, sa configuration de base et ses différents modules. Nous examinerons également comment utiliser Nginx comme serveur proxy et équilibreuse de charge pour améliorer la performance et la fiabilité des applications web. Nginx est un outil incontournable dans le monde du développement web moderne, et cette leçon vous fournira les connaissances nécessaires pour l'utiliser efficacement.


## 1 – Installation de Nginx
Étant donné que Nginx est disponible dans les dépôts par défaut d'Ubuntu, il est possible de l'installer à partir de ces dépôts en utilisant le système de paquets apt.

Comme il s'agit de notre première interaction avec le système de paquets apt dans cette session, nous allons mettre à jour notre index des paquets locaux pour avoir accès aux listes de paquets les plus récentes. Ensuite, nous pourrons installer Nginx :

```
sudo apt update
sudo apt install nginx
```

Appuyez sur Y lorsque vous êtes invité à confirmer l'installation. Si vous êtes invité à redémarrer des services, appuyez sur ENTER pour accepter les valeurs par défaut et continuer. apt installera Nginx et toutes les dépendances requises sur votre serveur.

## 2 – Ajustement du Pare-feu
Avant de tester Nginx, le logiciel de pare-feu doit être configuré pour permettre l'accès au service. Nginx s'enregistre comme un service avec ufw lors de l'installation, ce qui facilite l'autorisation de l'accès à Nginx.

Listez les configurations d'application que ufw sait gérer en tapant :

```
sudo ufw app list
```

Vous devriez obtenir une liste des profils d'application :

```
Applications disponibles :
  Nginx Full
  Nginx HTTP
  Nginx HTTPS
  OpenSSH
```

Comme indiqué par la sortie, il existe trois profils disponibles pour Nginx :

- **Nginx Full** : Ce profil ouvre à la fois le port 80 (trafic web normal non chiffré) et le port 443 (trafic chiffré TLS/SSL)
- **Nginx HTTP** : Ce profil ouvre uniquement le port 80 (trafic web normal non chiffré)
- **Nginx HTTPS** : Ce profil ouvre uniquement le port 443 (trafic chiffré TLS/SSL)

Il est recommandé d'activer le profil le plus restrictif qui permettra toujours le trafic que vous avez configuré. Pour l'instant, nous n'aurons besoin d'autoriser le trafic que sur le port 80.

Vous pouvez l'activer en tapant :

```
sudo ufw allow 'Nginx HTTP'
```

Vous pouvez vérifier le changement en tapant :

```
sudo ufw status
```

La sortie indiquera quel trafic HTTP est autorisé :

```
Output
Status: active

To                         Action      From
--                         ------      ----
OpenSSH                    ALLOW       Anywhere                  
Nginx HTTP                 ALLOW       Anywhere                  
OpenSSH (v6)               ALLOW       Anywhere (v6)             
Nginx HTTP (v6)            ALLOW       Anywhere (v6)
```

## 3 – Vérification de votre Serveur Web
À la fin du processus d'installation, Ubuntu 22.04 démarre Nginx. Le serveur web devrait déjà être en cours d'exécution.

Nous pouvons vérifier avec le système init systemd pour nous assurer que le service est en cours d'exécution en tapant :

```
systemctl status nginx
```

```
Output
● nginx.service - A high performance web server and a reverse proxy server
   Loaded: loaded (/lib/systemd/system/nginx.service; enabled; vendor preset: enabled)
   Active: active (running) since Fri 2022-03-01 16:08:19 UTC; 3 days ago
     Docs: man:nginx(8)
 Main PID: 2369 (nginx)
    Tasks: 2 (limit: 1153)
   Memory: 3.5M
   CGroup: /system.slice/nginx.service
           ├─2369 nginx: master process /usr/sbin/nginx -g daemon on; master_process on;
           └─2380 nginx: worker process
```

Comme confirmé par cette sortie, le service a démarré avec succès. Cependant, le meilleur moyen de tester cela est de demander réellement une page à Nginx.

Vous pouvez accéder à la page de démarrage par défaut de Nginx pour confirmer que le logiciel fonctionne correctement en naviguant à l'adresse IP de votre serveur. Si vous ne connaissez pas l'adresse IP de votre serveur, vous pouvez la trouver en utilisant l'outil icanhazip.com, qui vous donnera votre adresse IP publique telle que reçue d'un autre endroit sur Internet :

```
curl  your_domain
```

Lorsque vous avez l'adresse IP de votre serveur, entrez-la dans la barre d'adresse de votre navigateur :

```
http://your_server_ip
```

Vous devriez recevoir la page de démarrage par défaut de Nginx :


Si vous êtes sur cette page, votre serveur fonctionne correctement et est prêt à être géré.

Étape 4 – Gestion du Processus Nginx
Maintenant que vous avez votre serveur web en cours d'exécution, passons en revue quelques commandes de gestion de base.

Pour arrêter votre serveur web, tapez :

```
sudo systemctl stop nginx
```

Pour démarrer le serveur web lorsqu'il est arrêté, tapez :

```
sudo systemctl start nginx
```

Pour arrêter puis redémarrer le service, tapez :

```
sudo systemctl restart nginx
```

Si vous ne faites que des modifications de configuration, Nginx peut souvent recharger sans interrompre les connexions. Pour ce faire, tapez :

```
sudo systemctl reload nginx
```

Par défaut, Nginx est configuré pour démarrer automatiquement au démarrage du serveur. Si ce n'est pas ce que vous souhaitez, vous pouvez désactiver ce comportement en tapant :

```
sudo systemctl disable nginx
```

Pour réactiver le service afin qu'il démarre au démarrage, vous pouvez taper :

```
sudo systemctl enable nginx
```

Vous avez maintenant appris les commandes de gestion de base et devriez être prêt à configurer le site pour héberger plus d'un domaine.






