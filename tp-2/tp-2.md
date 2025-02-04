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
