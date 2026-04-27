## 2.2 Vérifier le contexte kubectl

1 : Quel est le nom du contexte courant (context) ?   :  minikube.

2 : Combien de nœuds ? On voit 1 nœud



## 3.3 Inspecter le pod

1: Quelle image exacte est utilisée ?  nginx:1.27.
2 : Quel évènement (Events) confirme le téléchargement et le démarrage ? L’événement qui confirme le téléchargement et le démarrage est Pulling puis Pulled et Started container.

## 4.2 Accéder au service depuis votre machine

1 : À quoi sert un Service dans Kubernetes ?
Le Service gère la redirection vers les pods automatiquement, il permet de rendre les pods accessibles via une Url

2 : Pourquoi le Service n’expose-t-il pas automatiquement une IP publique en local ?



## 5.2 Rollout / mise à jour d’image

1 : Que se passe-t-il au niveau des pods lors d’un changement d’image ?
Kubernetes crée un nouveau pod avec la nouvelle image. L'ancien pod est supprimé pour pouvoir ensuite faire le roulement

2 : Qu’est-ce qu’un “rollout” et pourquoi est-ce utile ?
Le “rollout” c'est le fait de déployer une nouvelle version sans interruption de service avec la possibilité de rollback s'il y a une erreur.




## Livrables 


Commandes de vérification :

    - kubectl get nodes
    - kubectl get pods -n tp-minikube
    - kubectl get svc -n tp-minikube



Le service est exposé et rendu accessible via  `minikube service web-nginx-svc -n tp-minikube --url.`


Difficultés :
 - Trouver comment rédiger le deployment.yml, avec la syntaxe (j'ai regardé la doc)
 - J'ai eu un problème d'indentation du yml du coup Kubernetess ne se lançais pas correctement (j'ai formatter le fichier :)


