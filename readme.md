## Compte rendu TP

### Database

Pour lancer mon container de base de données j'ai créer un Dockerfile dans lequel se trouve un "COPY" pour dire à mon image de charger les scripts sql nécessaire.

Ensuite il y'a une variable ENV qui contient mes variables d'environnements pour mon image. Il ne s'agit pas de la façon la plus sécurisé. Je pourrai utiliser l'argument -e dans la commande docker run mais il faudrait fournir à chaque commande les valeurs des variables d'environnement.

Autre façon, on peut mettre nos variables d'environnement dans un fichier .env et fournir ce .env à la commande docker run avec l'argument --env-file