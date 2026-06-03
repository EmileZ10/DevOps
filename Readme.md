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

Q2-1

Les Testcontainers sont des bibliothèques Java qui permettent de lancer des conteneurs Docker automatiquement pendant les tests.
Dans notre projet, ils servent à démarrer une base de données PostgreSQL temporaire afin de tester l’application dans des conditions proches de la production.
Cela permet d’avoir des tests d’intégration réalistes sans avoir besoin de configurer manuellement une base de données.

Q2-2

Les variables sécurisées (secrets) sont utilisées pour stocker des informations sensibles comme les identifiants Docker Hub ou les tokens d’accès.
On les utilise pour éviter de les écrire directement dans le code source ou dans le fichier YAML, car ce fichier est souvent versionné sur GitHub et pourrait être accessible publiquement.
Cela permet de sécuriser les credentials tout en les utilisant dans le pipeline CI/CD.

Q2-3

Le needs: test-backend permet de créer une dépendance entre les jobs GitHub Actions.
Cela garantit que le job de build et de push Docker ne s’exécute que si le job de test backend a réussi.
Sans cela, les images Docker pourraient être construites et publiées même si le code contient des erreurs ou si les tests échouent.

Q2-4

Les images Docker sont poussées vers Docker Hub pour pouvoir être stockées et utilisées à distance.
Cela permet de :
Déployer l’application sur n’importe quelle machine sans reconstruire les images
Partager facilement les images avec une équipe
Avoir un registre centralisé des versions de l’application
Automatiser les déploiements dans une logique CI/CD
Garantir que tous les environnements utilisent exactement la même image


Q2-5 (CI vs CD)

La CI (Continuous Integration) consiste à vérifier automatiquement que le code fonctionne correctement.
Dans notre pipeline, cela correspond à la compilation du projet et à l’exécution des tests Maven (mvn clean verify).
La CD (Continuous Delivery) consiste à préparer et livrer l’application.
Dans notre cas, cela correspond à la création des images Docker et leur publication sur Docker Hub.
La différence principale est que la CI vérifie le code tandis que la CD permet de le livrer sous forme utilisable.

Q2-6 (fonctionnement GitHub Secrets / tokens)

Les tokens Docker Hub sont stockés dans GitHub sous forme de secrets sécurisés.
Ils ne sont jamais écrits dans le code source.
Pendant l’exécution du pipeline, GitHub remplace automatiquement :

${{ secrets.DOCKERHUB_USERNAME }}
${{ secrets.DOCKERHUB_TOKEN }}

par les vraies valeurs stockées de manière sécurisée.
Le token permet ensuite de se connecter à Docker Hub via la commande docker login sans exposer les identifiants dans le repository.

Q2-7 (fonctionnement global CD)

La partie CD du pipeline fonctionne en plusieurs étapes :

GitHub Actions récupère le code source
Il se connecte à Docker Hub grâce aux secrets
Il construit les images Docker à partir des Dockerfiles
Il pousse ces images sur Docker Hub si le push est effectué sur la branche main
Cela permet d’automatiser complètement la création et la publication des images Docker après validation du code.

Q2-8 (Pourquoi séparer CI et CD)

Séparer CI et CD permet de mieux organiser le pipeline.
La CI est rapide et sert uniquement à tester le code.
La CD est plus lourde car elle implique la construction et la publication des images Docker.
Cela permet aussi d’éviter de publier une image si les tests ne sont pas passés, ce qui garantit la qualité du code livré.

Q2-9 (Résumé pipeline global)

Le pipeline CI/CD fonctionne de la manière suivante :
Un push déclenche GitHub Actions
La CI compile le projet et lance les tests
Si les tests réussissent, la CD démarre
Les images Docker sont construites
Les images sont envoyées sur Docker Hub
Cela permet d’avoir un processus automatisé de validation et de livraison du code.

Q3-1 

Inventaire setup.yml : définit le groupe prod avec le host emile.zanna.takima.school, l'utilisateur admin et le chemin vers la clé SSH privée.
Commandes utilisées :
ansible all -i inventories/setup.yml -m ping — vérifie la connectivité
ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*" — récupère les facts sur l'OS (Debian 12 bookworm)
ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become, supprime Apache2

Q3-2

Le playbook installe Docker en suivant la procédure officielle Debian : installation des dépendances, ajout de la clé GPG et du dépôt Docker, installation de docker-ce, puis création d'un environnement Python virtuel avec le SDK Docker. La dernière tâche vérifie que le service Docker est bien démarré.

Q3-3

database : image PostgreSQL, variables d'env POSTGRES_DB/USER/PASSWORD, connecté au réseau my-network
backend : image Spring Boot, variables d'env POSTGRES_* pour se connecter à la DB, connecté au réseau my-network
my-httpd : image Apache httpd, port 80:80 exposé, variables BACKEND_HOST=backend et BACKEND_PORT=8080 pour rediriger vers l'API

Q3-4

Ce n'est pas totalement sûr car n'importe quelle image taguée latest sera déployée en production, même si elle contient un bug ou une faille. Pour sécuriser on peut utiliser des tags de version (v1.0.1) au lieu de latest, et ne déployer que les tags validés. On peut aussi ajouter des tests supplémentaires avant le déploiement, ou une étape de validation manuelle.