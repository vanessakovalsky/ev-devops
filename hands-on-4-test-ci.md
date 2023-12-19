# exécution de test unitaire et d'analyse qualité sur les chaines de CI

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
* Pour afficher des rapports nous allons avoir besoin de différents plugins de Jenkins, qui permettent à partir des logs générés au format XML de créer des rapports graphiques :
* * Warnings Next Generation : récupère de nombreux logs et affiche au format graphique (CPD, MD, CS)
* * xUnit : récupère les logs des tests et affiche un rapport
* Pour installer les plugins, rendez vous dans Manage Jenkins > Plugin Manager 
* Chercher et cocher ces 2 plugins 
* Cliquer sur Install without restart
* Il reste maintenant à ajouter les étapes de builds our la publication des rapports
* Retourner dans la configuration du Job
* Dans l'onglet Post Build, ajouter une action
* * Record Compiler warning and static analysis results
* * * Static Analysis
* * * * Tools : PHP Code sniffer
* * * * Report file pattern :build/reports/checkstyle.xml
* * Publish xUnit test result report
* * * Report type  : PHP Unit
* * * Patter : build/reports/unitreport.xml

--> Votre chaine est maintenant configuré avec l'analyse qualité et la publication des rapports associés côté Jenkins, félicitations :)
