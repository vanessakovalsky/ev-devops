# Déploiement d’une application contenerisé en continu dans Kubernetes avec ArgoCD


## Configurer son application 

* Accéder à l'url d'argo cd, vous arrivez sur une mire d'authentification comme celle-ci :

![](http://blog.filador.fr/content/images/2020/12/argocd-authentication-2.png)


* Une fois connecté, la page d'accueil s'ouvre.
* Nous allons maintenant péparer notre application
* Pour cela, vous devez au préalable initialiser le dépôt de code Git qui contiendra l'ensemble des fichiers YAML ou templates de chart Helm à déployer. De plus, Argo CD peut directement pointer vers un dépôt Helm.

* ![](http://blog.filador.fr/content/images/2020/12/argocd-repository.png)

* A travers ce volet il est possible de réaliser cette action, celui-ci est accessible via Manage you repositories, projects, settings > Repositories > Connect repo using SSH / HTTPS.
* Utiliser le même dépôt que pour le CI, mais cette fois ci nous allons utiliser la branche kubetest1

## Déployer son application

* Une fois le dépôt de code configuré, on peut créer une application en suivant le menu Manage your applications, and diagnose health problems > New app.
* Un volet s'ouvre avec plusieurs catégories :
*   * General : Ici on va configurer les paramètres standards de l'application. La Sync policy permet à l'utilisateur de définir s'il veut que son application se synchronise automatiquement ou si c'est à lui d'appuyer sur un bouton pour qu'elle se mette à jour sur le cluster cible ;
    * Source : On configure le dépôt de code préalablement initialisé avec la Revision qui permet de sélectionner une branche, un tag ou un commit spécifique ;
    * Destination : C'est à travers cette section que l'on vient spécifier le cluster sur lequel on va déployer l'application. Par défaut, le cluster où est installé Argo CD est paramétré.

* Dans le cas d'un chart Helm par exemple, une section supplémentaire s'affiche :

![](http://blog.filador.fr/content/images/2020/12/argocd-application-helm.png)

* Elle permet de sélectionner le fichier values.yaml que l'on souhaite utiliser (dans le cas de plusieurs fichiers par type d'environnement par exemple), et de surcharger les valeurs de celui-ci dans la partie VALUES.
* Une fois le tout configuré, il reste à appuyer sur CREATE.
* Votre application apparaît sous le forme d'une tuile de couleur orange si l'application n'est pas synchronisée ou verte si elle l'est.
* On peut visualiser l'ensemble des objets Kubernetes si on clique sur la tuile de l'application que l'on souhaite déployer.
* Voici un exemple d'application avec l'ensemble des objets Kubernetes créé :

![](http://blog.filador.fr/content/images/2020/12/argocd-application-deploy.png)

* Cela permet de voir les dépendances entre les objets, par exemple : Deployment > Replica Set > Pod mais aussi la santé des objets, leur révision, le temps depuis leur création.
* En cliquant sur un objet, on peut voir l'ensemble des événements de celui-ci et sa configuration YAML.
