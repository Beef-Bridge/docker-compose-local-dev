# docker-compose-local-dev

Voici ma configuration pour un environnement de développement local basé sur Docker.  
Cette configuration est avant tout faite pour mes projets Symfony mais je me suis dit que ça pourrait servir à d'autres.

> #### Remarque
> Cette recette s'inspire de l'[article](http://www.christophe-meneses.fr/article/developper-une-application-symfony-avec-docker-version-php-fpm) de Christophe Meneses

### Configuration #
Personnellement j'utilise cette recette sur mon Mac, sur lequel j'ai Docker d'installé.  
De ce fait je partirai du principe que Docker est installé sur votre poste.

## Nos conteneurs
* **Apache** : un serveur web,
* **PHP-FPM** : une instance PHP-FPM qui sera utilisée par Apache,
* **MySQL** : une base de données,
* **Symfony** : notre code applicatif, qui sera lu par PHP-FPM et Apache
* **Les données de la base de données** : il va nous permettre de stocker les données de notre base de données
* **PhpMyAdmin** : PhpMyAdmin que l'on ne présente plus

La description de nos conteneurs est maintenant faite, voici donc ce à quoi va ressembler notre fichier docker-compose.yml :
```sh
version: '2'
services:
    cont_application:
        container_name: cont_application
        image: data_application
        volumes:
            - /chemin/local/vers/le/repertoire/Symfony:/var/www/html
    cont_data:
        container_name: cont_data
        image: data_mysql
        volumes:
            - /chemin/local/vers/le/repertoire/Data:/var/lib/mysql
    cont_database:
        container_name: cont_database
        image: mysql:5.7
        ports:
            - 60001:3306
        environment:
            MYSQL_ROOT_PASSWORD: root
            MYSQL_DATABASE: nom_de_la_base_de_donnees_que_vous_voulez
            MYSQL_USER: nom_utilisateur_mysql_que_vous_voulez
            MYSQL_PASSWORD: mot_de_passe_mysql_que_vous_voulez
        volumes_from:
            - cont_data
    cont_phpmyadmin:
        container_name: cont_phpmyadmin
        image: phpmyadmin:4.6
        ports:
            - 60002:80
        links:
            - cont_database:db
        environment:
            - PMA_USER=root
            - PMA_PASSWORD=root
    cont_php:
        container_name: test_php
        image: php-fpm:7.0
        volumes_from:
            - cont_application
    cont_web:
        container_name: cont_web
        image: apache:2.4
        ports:
            - 60000:80
        environment:
            FPM_HOST: test_php:9000
            XDEBUG_CONFIG: remote_host=172.22.0.1
        volumes_from:
            - cont_application
        links:
            - cont_database
            - cont_php
```

Maintenant que nous avons notre fichier docker-compose.yml, comprenant les diverses liaisons entre les containers et images, il nous faut créer ces images.  

## Nos images
