# Compte rendu TP Clément Bourdier 4IRC

## TP1

### Database

> Why should we run the container with a flag -e to give the environment variables ?

C'est mieux car si on change les variables d'environnement, il n'y a pas besoin de re build l'image.

> Why do we need a volume to be attached to our postgres container ?

L'utilisation d'un volume permet de garder les données quand on redémarre un conteneur.

> DockerFile
```DockerFile
FROM postgres:11.6-alpine
COPY ["./01-CreateScheme.sql", "./02-InsertData.sql", "/docker-entrypoint-initdb.d/"]

ENV POSTGRES_DB=db \ 
    POSTGRES_USER=usr \ 
    POSTGRES_PASSWORD=pwd
```

Pour lancer mon container de base de données j'ai créé un Dockerfile dans lequel se trouve un "COPY" pour dire à mon image de charger les scripts SQL nécessaire.

Ensuite il y a une variable ENV qui contient mes variables d'environnements pour mon image. Il ne s'agit pas de la façon la plus sécurisée. Je pourrai utiliser l'argument -e dans la commande docker run mais il faudrait fournir à chaque commande les valeurs des variables d'environnement.

Autre façon, on peut mettre nos variables d'environnement dans un fichier .env et fournir ce .env à la commande docker run avec l'argument --env-file.

### BACKEND API

> Why do we need a multistage build ? And explain each steps of this dockerfile

C'est une question d'optimisation. Le fait de faire en 2 étapes (build et run séparés) permet l'utilisation du JDK (Java Development Kit) pour le build, outil assez lourd et du JRE (Java Runtime Environment) bien plus léger pour le run.
Le build permet la compilation du projet Java (maven) et le run de lancer l'application.

### Http server

> Why do we need a reverse proxy ?

Le reverse proxy permet de faire un point d'entrée unique entre le client et le serveur, ce qui est plus sécurisé. Il gère donc les requêtes qu'il reçoit de la part du client puis il s'occupe de les rediriger par rapport à la demande du client (ou de les bloquer s'il le client n'a pas accès à la ressource demandée).

> httpd.conf

```
ServerName localhost
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass /api http://java2:8080/
    ProxyPassReverse /api http://java2:8080/
</VirtualHost>

```


> docker-compose.yml
```yml
version: '3.7' 
services:
  backend: 
    build: ./Java/part2/simple-api/simple-api/
    image: bourde/java2
    container_name: java2
    networks:
      - app-network
    depends_on:
      - database
  front:
    build:
      ./devops-front-main
    image: bourde/front:1.0
    container_name: front
    networks:
      - app-network
    depends_on:
      - backend
  database: 
    build: ./Database/
    image: bourde/database
    container_name: database
    networks:
      -  app-network
    volumes:
      - /Users/bourdier/Documents/S7/DEVOPS/TP/Database/data:/var/lib/postgresql/data
  httpd: 
    build: ./Httpd/
    image: bourde/httpd:1.0
    container_name: httpd
    ports:
      - 80:80
    networks:
      - app-network
    depends_on:
      - backend
networks: 
  app-network: 
    external: true

```

> Why is docker-compose so important ?

L'utilisation de docker-compose permet de lancer plusieurs conteneurs en une seule commande.

> Document docker-compose most important commands
> 
```
docker-compose -p 'TP1' up -d
```

**`-p`** : Pour donner un nom 

**`up`** : Pour lancer le service (les conteneurs dedans le docker-compose)

**`-d`** : Pour avoir le mode détaché (garder l'utilisation du terminal)


### Publish
```
    docker login -u "bourde" -p "mon_mot_de_passe" docker.io
    docker tag java2 bourde/java2:1.0
    docker push bourde/java2:1.0
```

> Why do we put our images into an online repository ?

C'est pour que l'on puisse pull ces images sur une machine distante (par exemple sur un serveur ou pour que quelqu'un d'autre utilise nos images).


## TP2

> Ok, what is it supposed to do ?

La commande **mvn clean verify** permet de build le projet et de lancer les tests mais également télécharger les dépendances si elles ne sont pas installées.

> Unit tests ? Component test ?

Unit test : C'est pour tester les fonctions que l'on a développé, voir leur réaction selon l'argument donné à la fonction (objet null ou objet de mauvais type). Ces tests sont faits par les développeurs (code).

Component test : C'est pour tester le fonctionnement d'un composant (bouton, champs texte etc).

> What are testcontainers?

C'est une librairie Java qui permet de lancer des tests unitaires (par exemple avec JUnit) et des bases de données le tout dans un conteneur Docker. 

### 2-2 Document your Github Actions configurations

```yml
name: CI devops 2022 CPE 
on:
  #to begin you want to launch this job in main and develop
  # La pipeline se lance si un push est effectué sur la branche master
  push:
    branches: 
      - master
  pull_request:
    branches:
      - master   

jobs: 
  test-backend:
    runs-on: ubuntu-18.04 
    steps:
      - uses: actions/checkout@v2.3.3   #checkout your github code using actions/checkout@v2.3.3
     
      - name: Set up JDK 11 #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '11'

      - name: Build and test with Maven #finally build your app with the latest command
        run: mvn clean verify --file ./Java/part2/simple-api/simple-api/pom.xml
```

J'ai uniquement mis la branche master puisque je suis seul à travailler sur mon projet donc je n'ai pas créé d'autres branches. Pour être plus propre, j'aurai dû créer une branche **dévelop** dans laquelle je faisais mes développements et dès que j'avais validé un checkpoint, j'aurais pu push dans **master**.

Ensuite pour la version de Java, j'ai choisi la même que notre projet Spring (11).

Et pour terminer pour le run, j'ai remis la commande fournie (mvn clean verify) et je lui fournis le path du pom.xml avec l'argument --file.

```yml
name: CI devops 2022 CPE 
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - master
  pull_request:
    branches:
      - master   
jobs: 
  test-backend:
    runs-on: ubuntu-18.04 
    steps:
      - uses: actions/checkout@v2.3.3   #checkout your github code using actions/checkout@v2.3.3
     
      - name: Set up JDK 11 #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '11'

      - name: Build and test with Maven and Sonar #finally build your app with the latest command
        # run: mvn clean verify --file ./Java/part2/simple-api/simple-api/pom.xml
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
          SONAR_TOKEN: ${{secrets.SONAR_TOKEN}}
        run: mvn clean verify --file ./Java/part2/simple-api/simple-api/pom.xml
  
  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
  # run only when code is compiling and tests are passing 
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
      - name: Connexion à Dockerhub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_PWD }}
      - name: Checkout code
        uses: actions/checkout@v2
        # BACKEND
      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Java/part2/simple-api/simple-api/
          # Note: tags has to be all lower-case 
          tags: ${{secrets.DOCKERHUB_USERNAME}}/java2:2.0
          push: ${{ github.ref == 'refs/heads/master' }}
        # FRONT
      - name: Build image and push front
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./devops-front-main
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/front:2.0
          push: ${{ github.ref == 'refs/heads/master' }} 

        # DATABASE
      - name: Build image and push database 
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Database
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/database:2.0
          push: ${{ github.ref == 'refs/heads/master' }}
        # HTTPD
      - name: Build image and push httpd 
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Httpd
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/httpd:2.0
          push: ${{ github.ref == 'refs/heads/master' }}

```
> Secured variables, why ?

Il faut des variables sécurisées pour ne pas les mettre dans le .main.yml et qu'elle soit accessible par tout le monde.

> Why did we put needs: build-and-test-backend on this job? Maybe try without this and you will see !

On doit avoir le backend de builder et tester avant de créer l'image et de la push sur le Docker Hub.

> For what purpose do we need to push docker images?

Pour versionné son code et que l'on garde des traces des anciennes versions. Mais aussi, comme dit auparavant, push les images sur le docker hub peut être utile pour d'autres développeurs, donc c'est nécessaire de les mettre à jour.

> .main.yml avec Sonar

```yml
name: CI devops 2022 CPE 
on:
  #to begin you want to launch this job in main and develop
  push:
    branches: 
      - master
  pull_request:
    branches:
      - master   
jobs: 
  test-backend:
    runs-on: ubuntu-18.04 
    steps:
      - uses: actions/checkout@v2.3.3   #checkout your github code using actions/checkout@v2.3.3
     
      - name: Set up JDK 11 #do the same with another action (actions/setup-java@v2) that enable to setup jdk 11
        uses: actions/setup-java@v2
        with:
          distribution: 'temurin'
          java-version: '11'

      - name: Build and test with Maven and Sonar #finally build your app with the latest command
        # run: mvn clean verify --file ./Java/part2/simple-api/simple-api/pom.xml
        env:
          GITHUB_TOKEN: ${{secrets.GITHUB_TOKEN}}
          SONAR_TOKEN: ${{secrets.SONAR_TOKEN}}
        run: mvn -B verify sonar:sonar -Dsonar.projectKey=ClementBourdier_CICDProject -Dsonar.organization=clementbourdier -Dsonar.host.url=https://sonarcloud.io -Dsonar.login=${{ secrets.SONAR_TOKEN }} --file ./Java/part2/simple-api/simple-api/pom.xml
  
  # define job to build and publish docker image
  build-and-push-docker-image:
    needs: test-backend
  # run only when code is compiling and tests are passing 
    runs-on: ubuntu-latest
    # steps to perform in job
    steps:
      - name: Connexion à Dockerhub
        run: docker login -u ${{ secrets.DOCKERHUB_USERNAME }} -p ${{ secrets.DOCKERHUB_PWD }}
      - name: Checkout code
        uses: actions/checkout@v2
        # BACKEND
      - name: Build image and push backend
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Java/part2/simple-api/simple-api/
          # Note: tags has to be all lower-case 
          tags: ${{secrets.DOCKERHUB_USERNAME}}/java2:2.0
          push: ${{ github.ref == 'refs/heads/master' }}
        # FRONT
      - name: Build image and push front
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./devops-front-main
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/front:2.0
          push: ${{ github.ref == 'refs/heads/master' }} 

        # DATABASE
      - name: Build image and push database 
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Database
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/database:2.0
          push: ${{ github.ref == 'refs/heads/master' }}
        # HTTPD
      - name: Build image and push httpd 
        uses: docker/build-push-action@v2
        with:
          # relative path to the place where source code with Dockerfile is located
          context: ./Httpd
          # Note: tags has to be all lower-case
          tags: ${{secrets.DOCKERHUB_USERNAME}}/httpd:2.0
          push: ${{ github.ref == 'refs/heads/master' }}

```

> Document your quality gate configuration

Pour ne pas avoir d'erreur il faut bien penser à mettre à OFF l'analyse automatique de SonarCloud.

Pour configurer ma Quality Gate j'ai utilisé les valeurs de la Quality gate par défaut, ce qui comprend :
- coverage < 80%
- duplicated lines > 3%
- maintanability rating is worse than A
- reliability rating is worse than A
- security sotspots reviewed < 100%
- security rating is worse than A
  
J'ai choisi que ma Quality Gate s'applique uniquement sur le nouveau code.

## TP3

### Intro

```yml
all: 
  vars:
    ansible_user: centos
    # chemin pour accèder à ma clé ssh
     ansible_ssh_private_key_file: /Users/bourdier/Documents/S7/DEVOPS/SSH/id_rsa
  children:
    # mon serveur
    prod:
      hosts: clement.bourdier.takima.cloud
```
> Document your inventory and base commands

Dans mon setup.yml, il y a le nom de mon serveur, ainsi que le chemin pour aller à ma clé privée

- ansible all -i inventories/setup.yml -m ping
- ansible all -i inventories/setup.yml -m setup -a "filter=ansible_distribution*"
- ansible all -i inventories/setup.yml -m yum -a "name=httpd state=absent" --become

### Playbooks
```yml
- hosts: all
  gather_facts: false
  become: yes
  # tous les rôles que je fais appel
  roles:
    - create_docker
    - create_network
    - launch_database
    - launch_app
    - launch_front
    - launch_proxy
```
> Document your playbook

Dans mon playbook, j'ai utilisé les rôles. Je fais donc appelle aux différents rôles.

Exemple du rôle de base de données, launch_database :

```yml
---
# tasks file for roles/launch_database
- name: Create database
  # mon cinteneur docker avec son image
  docker_container:
    name: database
    image: bourde/database:2.0
    # le réseau que j'ai créer
    networks:
      - name: network
    # mon volume
    volumes:
      - /Users/bourdier/Documents/S7/DEVOPS/TP/Database/data
```

### Deploy your app

> Document your docker_container tasks configuration.

Pour la base de données, voir l'exemple ci-dessus.

> create_docker : Installe tout le nécessaire pour pouvoir run docker correctement.

```yml
---
# tasks file for roles/create_docker
- name: Clean packages
  command:
    cmd: dnf clean -y packages

- name: Install device-mapper-persistent-data
  dnf:
    name: device-mapper-persistent-data
    state: latest

- name: Install lvm2
  dnf:
    name: lvm2
    state: latest

- name: add repo docker
  command:
    cmd: sudo dnf config-manager --add-repo=https://download.docker.com/linux/centos/docker-ce.repo

- name: Install Docker
  dnf:
    name: docker-ce
    state: present

- name: install python3
  dnf:
    name: python3

- name: Pip install
  pip:
    name: docker

- name: Make sure Docker is running
  service: name=docker state=started
  tags: docker
```

> create_network : création du réseau.

```yml
---
# tasks file for roles/create_network
- name: Create a network
  docker_network:
    name: network
```

> launcha_app : pour lancer le backend.

```yml
---
# tasks file for roles/launch_app
- name: Create App
  docker_container:
    name: java2
    image: bourde/java2:2.0
    networks:
      - name: network
```

> launch_proxy : pour lancer le proxy.

```yml
---
# tasks file for roles/launch_proxy
- name: Create proxy
  docker_container:
    name: httpd
    image: bourde/httpd:2.0
    networks:
      - name: network
    ports:
      - "80:80"
```

À cette partie du TP, nous n'avons pas encore fait le front mais dans la partie suivante j'ai dû créer un rôle donc je vous le mets aussi.

> launch_front : pour lancer le front.

```yml
---
# tasks file for roles/launch_front
- name: Launch front
  docker_container:
    name: front
    image: bourde/front:2.0
    networks:
      - name: network 
```

### Front

J'ai créé un conteneur docker à partir du DockerFile présent dans la partie front. Je l'ai tag puis push sur le docker hub.

Ensuite j'ai rajouté dans le docker-compose la partie pour le front :

```yml
front:
    build:
      ./devops-front-main
    image: bourde/front:1.0
    container_name: front
    networks:
      - app-network
    depends_on:
      - backend
```

J'ai ensuite modifié le httpd.conf :
```
ServerName localhost
<VirtualHost *:80>
    ProxyPreserveHost On
    ProxyPass /api http://java2:8080/
    ProxyPassReverse /api http://java2:8080/
    ProxyPass / http://front:80/
    ProxyPassReverse / http://front:80/
</VirtualHost>
```

Dans le fichier .env.production j'ai mis ça :
```
VUE_APP_API_URL=clement.bourdier.takima.cloud:80/api
```
si je veux que mon front accède à mon api sur le serveur ou
```
VUE_APP_API_URL=localhost/api
```
si je veux que mon front accède à mon api en local.

J'ai ensuite créer le rôle front, vu dans la partie précédente, pour le playbook.