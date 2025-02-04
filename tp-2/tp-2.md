# TP2 

### 1 TestContainers
Testcontainers est un concept qui consiste à utiliser des conteneurs Docker comme environnements de test temporaires pour les tests d’intégration. Il permet d’exécuter des bases de données, files de messages ou autres services nécessaires aux tests de manière isolée, reproductible et automatisée. Les conteneurs sont créés au début des tests et détruits après, garantissant un environnement propre et identique à chaque exécution.

### 2 GitHub Actions
```yaml
# Nom du workflow GitHub Actions

name: CI devops 2025

on:
# Déclenchement du workflow sur un push ou une pull request sur les branches main et develop
push:
branches:
- main
- develop
pull_request:

jobs:
# Définition du job de test du backend
test-backend:
# Spécifie l’environnement d’exécution
runs-on: ubuntu-22.04

    steps:
      # Étape 1 : Récupération du code source du dépôt
      - name: Checkout du code source
        uses: actions/checkout@v4

      # Étape 2 : Installation de JDK pour exécuter le projet Java
      - name: Set up JDK 23  # ⚠️ Le commentaire indique JDK 21 mais l'action installe JDK 23
        uses: actions/setup-java@v4
        with:
          distribution: 'corretto'  # Utilisation de la distribution Amazon Corretto
          java-version: '23'  # Version de Java installée

      # Étape 3 : Compilation et exécution des tests avec Maven
      - name: Build and test with Maven
        run: mvn clean verify  # Nettoie, compile et exécute les tests du projet
        working-directory: tp-1/java/simple-api-student  # Définit le dossier où exécuter la commande
```
### 3 Publication image docker hub 

Nous poussons des images Docker pour les partager, les déployer dans des environnements de production, ou les utiliser dans des pipelines CI/CD. Cela permet de garantir que les applications sont exécutées de manière cohérente sur différentes machines et infrastructures.

### 4 Sonar 

Etapes pour mettre en place Sonar :

- Création du compte sur SonarCloud
- Integartion de l'organization avec le repository GitHub
- Mise en place du fichier build.yaml
```yaml
name: SonarQube
on:
  push:
    branches:
      - main
  pull_request:
    types: [opened, synchronize, reopened]
jobs:
  build:
    name: Build and analyze
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Shallow clones should be disabled for a better relevancy of analysis
      - name: Set up JDK 23
        uses: actions/setup-java@v4
        with:
          java-version: 23
          distribution: 'corretto' # Alternative distribution options are available.
      - name: Cache SonarQube packages
        uses: actions/cache@v4
        with:
          path: ~/.sonar/cache
          key: ${{ runner.os }}-sonar
          restore-keys: ${{ runner.os }}-sonar
      - name: Cache Maven packages
        uses: actions/cache@v4
        with:
          path: ~/.m2
          key: ${{ runner.os }}-m2-${{ hashFiles('**/pom.xml') }}
          restore-keys: ${{ runner.os }}-m2
      - name: Build and analyze
        env:
          SONAR_TOKEN: ${{ secrets.SONAR_TOKEN }}
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=Mathieu-Cieslar_dev-ops-cpe -Dsonar.organization=mathieu-cieslar -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }}  --file ./tp-1/java/simple-api-student/pom.xml
```