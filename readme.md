## Compte rendu TP

### Database

Pour lancer mon container de base de données j'ai créer un Dockerfile dans lequel se trouve un "COPY" pour dire à mon image de charger les scripts sql nécessaire.

Ensuite il y'a une variable ENV qui contient mes variables d'environnements pour mon image. Il ne s'agit pas de la façon la plus sécurisé. Je pourrai utiliser l'argument -e dans la commande docker run mais il faudrait fournir à chaque commande les valeurs des variables d'environnement.

Autre façon, on peut mettre nos variables d'environnement dans un fichier .env et fournir ce .env à la commande docker run avec l'argument --env-file

### 2-1 What are testcontainers?

C'est une librairie Java qui permet de faire des tests avec JUnit

### 2-2 Document your Github Actions configurations

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

      - name: Build and test with Maven #finally build your app with the latest command
        run: mvn clean verify --file ./Java/part2/simple-api/simple-api/pom.xml
```

J'ai uniquement mis la branche master puisque je suis seul à travailler sur mon projet donc je n'ai pas crée d'autres branches.

Ensuite pour la version de Java, j'ai choisi la même que notre projet Spring (11)

Et pour terminer pour le run, j'ai remis la commande fournie (mvn clean verify) et je lui fourni le path du pom.xml


### Secured variables, why ?

Il faut des variables sécurisées pour ne pas les mettre dans le .main.yml et qu'elle soit accessible par tout le monde 



### 2-3 Document your quality gate configuration

J'ai pris des valeurs plutôt normalisé pour ma Quality Gate puisque j'ai pris les valeurs de la la Quality Gate par défaut fournit par Sonar :
- moins de 80% de coverage
- moins de 3% de duplication
- et d'autres
  
Et ma Quality Gate s'applique uniquement sur le nouveau code

### 3-1 Document your inventory and base commands

Dans mon setup.yml, il y'a le nom de mon serveur, ainsi que le path pour aller à ma clé privée