TP1

1) -e \
Passer des variables d'environnement avec l'option -e lors du lancement d'un conteneur évite d'écrire en dur les informations dans le docker file 
2) Volume \
Le volume permet la persistance des données, il permet de stocker les données en dehors du conteneur, ainsi on peut garder les données même si le conteneur est détruit 
3) Dockerfile \
  ``docker run -p 5432:5432 --name db --net=app-network -v /tp-1/data:/var/lib/postgresql/data   mathieu/db  -d mathieu/db ``