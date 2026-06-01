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