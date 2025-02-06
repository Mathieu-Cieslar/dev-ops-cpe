# Ansible

### 1 Introduction à Ansible

liste des commandes : 

connexion ssh : `ssh -i id_rsa admin@mathieu.cieslar.takima.cloud `

création d'une page html `ansible all -m shell -a 'echo "<html><h1>Hello World</h1></html>" > /var/www/html/index.html' --private-key=id_rsa -u admin --become`

deploiement d'un serveur apache avec ansible :
`ansible all -m apt -a "name=apache2 state=present" --private-key=id_rsa -u admin --become`

supression du serveur apache : `ansible all -i inventories/setup.yml -m apt -a "name=apache2 state=absent" --become`

fichier de configuration :
```yaml 
all:  # 🔹 Niveau global : toutes les machines de l'inventaire
  vars:  # 🔹 Variables globales valables pour tous les hôtes
    ansible_user: admin  # ✅ Définit l'utilisateur utilisé pour la connexion SSH
    ansible_ssh_private_key_file: /Users/mcieslar/projects-cpe/id_rsa  # ✅ Clé SSH privée utilisée pour l'authentification

  children:  # 🔹 Groupes d'hôtes
    prod:  # ✅ Groupe "prod" contenant les serveurs de production
      hosts:  # 🔹 Liste des hôtes dans le groupe "prod"
        mathieu.cieslar.takima.cloud  # ✅ Serveur cible

```

### 2 playbook 
    
 ```yaml 
  # Playbook Ansible pour l'installation de Docker et de ses dépendances sur toutes les machines ciblées
  - hosts: all
    gather_facts: true  # Récupère les informations système des hôtes
    become: true  # Exécute les tâches avec les privilèges root

    tasks:
      # Installation des paquets nécessaires pour Docker
      - name: Installer les paquets requis
        apt:
          name:
            - apt-transport-https  # Permet le téléchargement sécurisé via HTTPS
            - ca-certificates  # Gère les certificats SSL
            - curl  # Outil de transfert de données
            - gnupg  # Gère les clés GPG
            - lsb-release  # Identifie la distribution Linux
            - python3-venv  # Nécessaire pour créer des environnements virtuels Python
          state: latest  # Installe la dernière version des paquets
          update_cache: yes  # Met à jour la liste des paquets avant installation

      # Ajout de la clé GPG officielle de Docker pour sécuriser le dépôt
      - name: Ajouter la clé GPG de Docker
        apt_key:
          url: https://download.docker.com/linux/debian/gpg
          state: present

      # Ajout du dépôt officiel Docker pour Debian
      - name: Ajouter le dépôt APT de Docker
        apt_repository:
          repo: "deb [arch=amd64] https://download.docker.com/linux/debian {{ ansible_facts['distribution_release'] }} stable"
          state: present  # Ajoute le dépôt s'il n'existe pas déjà
          update_cache: yes  # Met à jour la liste des paquets

      # Installation de Docker
      - name: Installer Docker
        apt:
          name: docker-ce  # Installe le moteur Docker
          state: present  # Assure qu'il est bien installé

      # Installation de Python3 et pip3
      - name: Installer Python3 et pip3
        apt:
          name:
            - python3
            - python3-pip
          state: present  # Vérifie que Python3 et pip3 sont bien installés

      # Création d'un environnement virtuel pour gérer les dépendances Python liées à Docker
      - name: Créer un environnement virtuel pour le SDK Docker
        command: python3 -m venv /opt/docker_venv
        args:
          creates: /opt/docker_venv  # Évite de recréer l'environnement si le dossier existe déjà

      # Installation du SDK Docker pour Python dans l'environnement virtuel
      - name: Installer le SDK Docker pour Python dans l'environnement virtuel
        command: /opt/docker_venv/bin/pip install docker

      # Vérification et démarrage du service Docker
      - name: S'assurer que Docker est en cours d'exécution
        service:
          name: docker
          state: started
        tags: docker  # Permet d'exécuter uniquement cette tâche avec `--tags docker`

  ```
### 3 Docker container tasks 

# Déploiement Docker avec Ansible

## 1. Création des réseaux Docker

Création de deux réseaux Docker distincts :

- **`backend-network`** : Réseau pour la communication entre le backend et la base de données.
- **`proxy-network`** : Réseau pour la communication entre le backend et le serveur proxy HTTP.

### Playbook pour la création des réseaux

```yaml
---
- name: Créer les réseaux Docker
  docker_network:
    name: "{{ item }}"
    driver: bridge
  loop:
    - backend-network
    - proxy-network
  vars:
    ansible_python_interpreter: /usr/bin/python3
```

### Explication

- **`docker_network`** : Ce module permet de créer un réseau Docker.
- **`loop`** : Boucle pour créer `backend-network` et `proxy-network`.

## 2. Gestion des variables d’environnement

### Objectif

Utiliser un fichier `.env.yml` pour gérer les variables d’environnement des conteneurs Docker (base de données, backend, etc.).

### Exemple de fichier `.env.yml`

```yaml
DB_URL: ""
DB_USR: ""
DB_PWD: ""
DB_NAME: ""
```

### Playbook pour charger les variables d’environnement

```yaml
---
- name: Charger les variables d’environnement
  ansible.builtin.include_vars:
    file: ../../.env.yml
```

### Explication

- **`include_vars`** : Ce module permet d’inclure un fichier de variables dans le playbook.
- **`file`** : Spécifie le chemin du fichier `.env.yml` contenant les variables d’environnement pour la base de données et l’application.


## 3. Déploiement des conteneurs Docker

### Objectif

Déployer les conteneurs pour le backend, la base de données et le proxy HTTP, en les attachant aux réseaux appropriés et en utilisant les variables d’environnement.

### Playbook pour le déploiement des conteneurs

Sur chaque deploiement on deploie les dernieres image créer auparavant  

#### Déploiement du conteneur de la base de données

```yaml
- name: Run Database container
  docker_container:
    name: tp-1-database-1
    image: mathieuc71/database-cpe:latest
    pull: yes
    env:
      POSTGRES_DB: "{{DB_NAME}}"
      POSTGRES_USER: "{{DB_USR}}"
      POSTGRES_PASSWORD: "{{DB_PWD}}"
    networks:
      - name: backend-network
    volumes:
      - postgres-data:/var/lib/postgresql/data
```

#### Déploiement du conteneur Backend

```yaml
- name: Run Application container
  docker_container:
    name: tp-1-backend-1
    image: mathieuc71/backend-cpe:latest
    pull: yes
    env:
      DB_URL: "{{ DB_URL }}"
      DB_USR: "{{ DB_USR }}"
      DB_PWD: "{{ DB_PWD }}"
    networks:
      - name: backend-network
      - name: proxy-network
```

#### Déploiement du conteneur Proxy HTTP

```yaml

- name: Run Application container
  docker_container:
    name: tp-1-httpd-1
    image: mathieuc71/httpd-cpe:latest
    pull: yes
    ports:
      - "80:80"
      - "8080:8080"
    networks:
      - name: proxy-network
    restart_policy: always
    state: started
```

- **`pull`** : On mets la valeur a yes pour être sur de pull la dernière image 


#### Instruction de deploiement 

Enfin on donne les instruction avec la liste des tâches a executer 

```yaml 
- hosts: all
  gather_facts: true
  become: true

  roles:
    #- install_docker
    - copy_env
    - create_network
    - launch_database
    - launch_app
    - launch_proxy
```
On vois dans ce fichier que chaque etape du deploiments correspond à un rôles 







