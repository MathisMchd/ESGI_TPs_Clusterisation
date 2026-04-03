# Mathis Michenaud - Clusteurisation de Conteneur
[https://sysentive.link/tp-opti-docker-image](https://sysentive.link/tp-opti-docker-image)

Poids de l'image :
 - Classique : 1.63Gb
 - Multistatge : 296Mb


Ce qui influence le poids, c'est l'image de base utilisé : le poids entre node20 et node20-slim n'est pas le même.
https://hub.docker.com/_/node

Le fait de ne pas inclure les nodes_modules dans l'image permet également de ne pas allourdir l'image finale.
Le multistage contient uniquement les fichiers nécessaires pour l'exécution (pas de fichiers de build etc) et on ne prends pas les dépendances de dev.
