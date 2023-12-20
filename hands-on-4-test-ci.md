# Exécution de test unitaire et d'analyse qualité sur les chaines de CI

## Lancement des tests unitaires
* Des tests existent dans l'application
* Pour les lancers nous utilisons junit via maven
* Pour lancer les tests manuellement, vous pouvez éxécuter : 
```
mvn test
```
* Vous obtenez alors le résultat de l'éxecution des tests 

## Automatisation avec Jenkins

* Revenir au fichier jenkinsfile
* Ajouter l'étape suivante entre le bulid et le dépôt de l'artefact dans nexus :

```
        stage('Test'){
            steps {
                sh 'mvn test'
            }

        }
```
* Push votre jenkinsfile
* Lancer un build et observer le résultat

## Ajouter du contrôle qualité à votre pipeline

* Nous allons ajouter une étape d'analyse qualité à notre code avec l'outil checkstyle*
* Ajouter l'étape suivante entre les build et les tests unitaire :

```
        stage('Checkstyle Analysis'){
            steps {
                sh 'mvn checkstyle:checkstyle'
            }
        }
```
* Push votre jenkinsfile
* Lancer un build, puis allez voir dans le workspace du produit :
--> Les résultats de ses différentes étapes sont maintenant disponibles sous forme de fichier XML ou dans la sortie Console Output


## Afficher les résultats sous formes de rapports

* Ajouter dans l'étape de test le post traitement suivant
```
post {
    always {
      junit(testResults: 'target/surefire-reports/*.xml', allowEmptyResults : true)
    }
  }
```
* Push votre jeninsfile et lancer un build
--> Votre chaine est maintenant configuré avec l'analyse qualité et la publication des rapports associés côté Jenkins, félicitations :)


## Pour aller plus loin :

* Déployer un environnement avec terraform depuis jenkins (pour déployer un environnement de test par exemple) : https://spacelift.io/blog/terraform-jenkins
* Ajouter des tests de charge avec JMeter sur Jenkins : https://www.jenkins.io/doc/book/using/using-jmeter-with-jenkins/ 
