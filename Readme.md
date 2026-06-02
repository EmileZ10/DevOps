Q1-1

On évite de mettre les mots de passe dans le Dockerfile car ce fichier est souvent versionné sur Git. N'importe qui ayant accès au repo pourrait voir les credentials. Avec -e on les passe uniquement au moment du docker run, sans les écrire nulle part dans le code.

Q1-2

PostgreSQL stocke ses données dans le conteneur. Si on le supprime, tout est perdu. En attachant un volume, les données sont sauvegardées sur la machine hôte et survivent à la destruction du conteneur.

Q1-3

Dockerfile :
dockerfileFROM postgres:17.2-alpine
COPY sql/ /docker-entrypoint-initdb.d/
Commandes :
bashdocker build -t my-database ./database
docker run -d --name my-postgres --net app-network \
  -e POSTGRES_DB=db -e POSTGRES_USER=usr -e POSTGRES_PASSWORD=pwd \
  -v pgdata:/var/lib/postgresql/data my-database

Q1-4 

Sans multistage, l'image finale contiendrait le JDK et Maven qui sont lourds et inutiles en production. Le premier stage compile le code avec le JDK, le second ne copie que le jar et tourne avec un simple JRE, beaucoup plus léger.

Q1-5

Le reverse proxy (Apache httpd) sert d'intermédiaire entre le client et le backend. Avantages : il masque l'architecture interne (le client ne connaît pas le port 8080), il peut gérer le SSL, le load balancing, et servir un frontend statique. Le client n'accède qu'au port 80, tout le reste est caché.

Q1-6

Docker-compose permet d'orchestrer plusieurs containers en une seule commande. Sans lui, il faut démarrer, arrêter et configurer chaque container manuellement en faisant attention à l'ordre et aux réseaux. Avec docker-compose, tout est décrit dans un seul fichier et docker compose up fait tout automatiquement.

Q1-7

docker compose up -d : démarre tous les services en arrière-plan
docker compose down : arrête et supprime les containers
docker compose build : rebuild les images
docker compose logs : affiche les logs de tous les services
docker compose ps : liste les services et leur état

Q1-8

Voir commentaires

Q1-9

Les images Docker ont été taguées puis publiées sur Docker Hub avec les commandes suivantes :

docker login

docker tag tp-docker-backend emilez10/tp-docker-backend:1.0
docker tag tp-docker-database emilez10/tp-docker-database:1.0
docker tag tp-docker-httpd emilez10/tp-docker-httpd:1.0

docker push emilez10/tp-docker-backend:1.0
docker push emilez10/tp-docker-database:1.0
docker push emilez10/tp-docker-httpd:1.0

Images publiées sur Docker Hub :

emilez10/tp-docker-backend:1.0
emilez10/tp-docker-database:1.0
emilez10/tp-docker-httpd:1.0

Q1-10

Les images Docker sont stockées dans un dépôt en ligne afin de :

Partager facilement les images avec les autres membres de l'équipe.
Déployer l'application sur d'autres machines sans avoir à reconstruire les images.
Centraliser les versions grâce au système de tags (1.0, 2.0, latest, etc.).
Faciliter l'intégration continue et le déploiement continu (CI/CD), les serveurs pouvant télécharger directement les images depuis le registre.
Sauvegarder les images dans un emplacement accessible même si la machine locale est indisponible.
Garantir la reproductibilité des déploiements en utilisant exactement la même image sur tous les environnements (développement, test, production).

Docker Hub est un registre public très utilisé, mais les entreprises utilisent souvent des registres privés pour mieux contrôler l'accès à leurs images et protéger leur code.