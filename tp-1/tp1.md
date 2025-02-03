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

### 6) Docker-compose
Docker-compose est un outil qui permet de définir et gérer des applications multi-conteneurs avec Docker. Il utilise un fichier YAML pour configurer les services, les réseaux et les volumes nécessaires à l'application. Voici quelques avantages de docker-compose :
-  Simplification : Permet de définir et lancer plusieurs conteneurs en une seule commande.
- Configuration : Facilite la gestion des dépendances, des réseaux et des volumes.
- Portabilité : Permet de déployer l'application sur différents environnements sans modification.
- Gestion des versions : Permet de spécifier les versions des images et des services pour garantir la compatibilité.



### 7) Docker Hub :

**Commandes utilisées :**

1. **Se connecter à Docker Hub :**
   ```bash
   docker login
   ```
   Cette commande vous permet de vous connecter à votre compte Docker Hub en utilisant vos identifiants.

2. **Taguer l'image Docker :**
   ```bash
   docker tag my-database USERNAME/my-database:1.0
   ```
   Cette commande permet d'attribuer un tag à l'image `my-database`, en l'associant à votre nom d'utilisateur Docker Hub et à une version spécifique (`1.0`).

3. **Pousser l'image vers Docker Hub :**
   ```bash
   docker push USERNAME/my-database:1.0
   ```
   Cela pousse l'image taggée vers Docker Hub, la rendant disponible dans votre compte Docker Hub, où elle pourra être utilisée par d'autres personnes ou sur d'autres machines.
****

### 8) Docker Hub 2 :

Placer nos images Docker dans un dépôt en ligne comme Docker Hub présente plusieurs avantages :

Accessibilité : Cela permet de partager facilement des images avec d'autres développeurs ou machines, peu importe leur localisation.
Centralisation : En hébergeant les images dans un dépôt centralisé, on simplifie la gestion des versions et l'accès aux dernières versions de l'image.
Facilité de déploiement : Les images stockées sur un dépôt en ligne peuvent être utilisées sur différents environnements sans avoir besoin de recréer l'image à chaque fois.
Collaboration : Lorsque l'image est publique ou partagée avec des équipes, elle devient un moyen de collaboration efficace, avec la possibilité de récupérer des mises à jour ou des corrections.
Sauvegarde : Stocker des images sur Docker Hub ou un dépôt privé garantit leur sécurité et leur disponibilité en cas de perte locale.