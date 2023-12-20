# Securisation des pipelines : analyse statique de code et des dépendances

## Mise en place d'un pipeline pour une application Python

* Forker le dépôt présent ici sur votre compte github : https://github.com/vanessakovalsky/python-api-handle-it
* Dans Jenkins créer un job avec le pipeline se trouvant dans le dossier continuous-integration
* Lancer le build, quelles sont les étapes du build ?

## Sécurité du code et des dépendances

* Avant le build du paquet, nous allons rajouter les étapes de sécurité suivante :
* * Lancement d'un scanner de sécurity avec l'outil Python Bandit
  * Lancement d'une analyse des dépendances avec Python Safety
  * Lancement d'une analyse statique avec python-taint
* Ajouter les étapes suivantes avant le build à votre Jenkinsfile
```
stage ("Python Bandit Security Scan"){
			steps{
				sh "docker run --rm --volume \$(pwd) secfigo/bandit:latest"
			}
		}
		stage ("Dependency Check with Python Safety"){
			steps{
				sh "docker run --rm --volume \$(pwd) pyupio/safety:latest safety check"
				sh "docker run --rm --volume \$(pwd) pyupio/safety:latest safety check --json > report.json"
			}
		}
		stage ("Static Analysis with python-taint"){
			steps{
				sh "docker run --rm --volume \$(pwd) vickyrajagopal/python-taint-docker pyt ."
			}
		}
```
* Push votre fichier et relancer un build.
* L'application est t'elle sécurisée ?

## Pour aller plus loin - Analyse d'image de conteneur

* Vous pouvez aussi ajouter une analyse sur l'image de conteneur avec Anchor : https://www.jenkins.io/blog/2018/06/20/anchore-image-scanning/
* Ou bien avec sysdig : https://sysdig.com/blog/docker-scanning-jenkins/
