TP1

### 1) -e 
Passer des variables d'environnement avec l'option -e lors du lancement d'un conteneur évite d'écrire en dur les informations dans le docker file 
### 2) Volume 
Le volume permet la persistance des données, il permet de stocker les données en dehors du conteneur, ainsi on peut garder les données même si le conteneur est détruit 
### 3) Dockerfile 
  ``docker run -p 5432:5432 --name db --net=app-network -v /tp-1/data:/var/lib/postgresql/data   mathieu/db  -d mathieu/db ``
### 4) Multi-stage
Le multi stage permet de séparer les différentes étapes du préparation du service dans l'exemple utilise dans /java/simpleapi il permet de séparer la compilation et l'exécution du code, et à la fin le conteneur contiendra uniquement les outils nécessaires pour l'éxecution du code (ici on embarquant pas le compilateur)\
Explication docker file : 

 Étape 1 : Stage de construction (Build Stage)
- **FROM maven:3.9.9-amazoncorretto-21 AS myapp-build**  
  Utilise l'image **Maven 3.9.9** avec **Amazon Corretto 21 (JDK 21)** pour compiler l'application Java.

- **ENV MYAPP_HOME=/opt/myapp**  
  Définit une variable d'environnement `MYAPP_HOME=/opt/myapp` pour une gestion uniforme du chemin.

- **WORKDIR $MYAPP_HOME**  
  Définit le répertoire de travail à `/opt/myapp`.

- **COPY pom.xml .**  
  Copie le fichier **pom.xml** (le fichier de configuration de Maven) dans le conteneur.

- **COPY src ./src**  
  Copie le code source (**src/**) dans le conteneur.

- **RUN mvn package -DskipTests**  
  Exécute la commande `mvn package -DskipTests`, qui :
  - Compile le code Java.
  - Crée un fichier `.jar` pour l'application.
  - **Ignore les tests** pour accélérer le processus de construction.

 Étape 2 : Stage d'exécution (Run Stage)
- **FROM amazoncorretto:21**  
  Utilise **Amazon Corretto 21** (JDK 21) comme environnement d'exécution, mais sans Maven.

- **ENV MYAPP_HOME=/opt/myapp**  
  Définit à nouveau le répertoire de travail `/opt/myapp`.

- **WORKDIR $MYAPP_HOME**  
  Définit le répertoire de travail à `/opt/myapp`.

- **COPY --from=myapp-build $MYAPP_HOME/target/*.jar $MYAPP_HOME/myapp.jar**  
  Copie le fichier `.jar` généré par le **stage de construction (`myapp-build`)** dans l'image finale.

- **ENTRYPOINT ["java", "-jar", "myapp.jar"]**  
  Définit la commande par défaut pour exécuter l'application Java avec `java -jar myapp.jar`.

### 5) Reverse proxy
Un reverse proxy est un serveur intermédiaire entre les clients et un ou plusieurs serveurs backend. Il offre plusieurs avantages :
-  Sécurité : Protège les serveurs backend en cachant leur IP, en filtrant le trafic et en ajoutant un pare-feu.
-  Équilibrage de charge : Répartit les requêtes entre plusieurs serveurs pour éviter la surcharge et améliorer la disponibilité.
-  Performance : Met en cache les réponses, compresse les données et optimise la bande passante.
-  Simplification : Centralise la gestion des accès et permet d’héberger plusieurs applications sur un même domaine.
-  Support SSL/TLS : Gère les certificats pour sécuriser les communications sans impacter les serveurs backend.

