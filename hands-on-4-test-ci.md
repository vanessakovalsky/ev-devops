# Mise en place d’environnement de test, et de leur execution sur les chaines de CI

## Ajout de test et lancement sur le projet en PHP
* Des tests existent dans l'application
* POur les lancers nous utilisons PHPUnit
* Pour cela on ajoute une tache dans le build.gradle qui va se charger de lancer les tests
```
task test(type:Exec, dependsOn: installDeps) {
  //println 'Executing tests'
  executable 'sh'
  args '-c', "php \
    './project/vendor/phpunit/phpunit/phpunit' \
    --configuration='./project/phpunit.xml' \
    --log-junit='./logs/unitreport.xml'\
    ./project/tests"
}
```
* Pour lancer les tests, on se connecte au conteneur 
```
docker run -it my-gradle bash
```

* Puis on lance les tests avec la tache gradle 
```
gradle test
```
* Vous obtenez alors le résultat de l'éxecution des tests : 
```
RRORS!
Tests: 20, Assertions: 3, Errors: 18.

FAILURE: Build failed with an exception.

* What went wrong:
Execution failed for task ':test'.
> Process 'command 'sh'' finished with non-zero exit value 2

* Try:
> Run with --stacktrace option to get the stack trace.
> Run with --info or --debug option to get more log output.
> Run with --scan to get full insights.

* Get more help at https://help.gradle.org

BUILD FAILED in 40s
4 actionable tasks: 3 executed, 1 up-to-date
```
* Certains tests ont échoués car il s'agit de test d'integration, qui nécessite une connexion à la base de donnée, notre application dans le conteneur n'est pas correctement configuré pour utiliser la base de données

* On peut alors : 
  * Soit corrigé l'environnement pour que les tests puissent tourner (et corriger le code si nécessaire), cela est généralement fait en collaboration avec les développeurs car nécessite des compétences dans le langage de programmation de l'application
  * Soit dire à gradle d'ignorer le résultat du test et de continuer le build 

* Pour la deuxième solution, sur une tâche de type exec on ajoute la propriété (la propriété est différente selon le type de tâche): 
```
ignoreExitValue true
```

* Nous pouvons alors remettre la dépendance à notre tache de tests et lancer la tâche de package et d'envoi sur nexus :

```
gradle up
```

## Automatisation avec Jenkins

Cet exercice a pour objectif :
* De créer un job sur jenkins qui va permettre:
* * de récupérer le code sur le dépôt Git
* * de lancer l'installation des dépendances et les tests
* * de construire un paquet et de le déployer sur Nexus

## Pré-requis 
L'accès à un serveur Jenkins est nécessaire. Vous pouvez l'installer en local soit avec Docker, soit avec Vagrant 

### Docker
* il est également possible de lancer un jenkins en local à l'aide du docker compose du dossier exercice-5-ci-jenkins avec la commande :
```
docker-compose up -d
```
* Un serveur jenkins et un serveur nexus sont alors lancés sur deux conteneurs
* Il est nécessaire de configurer jenkins
* Pour cela aller à l'adresse : http//localhost:8098 

### Vagrant
* Récupérer le Vagrantfile et le install.sh dans vagrant/jenkins sur ce dépôt
* Configurer l'adresse ip du vagrant file pour qu'elle soit utilisable sur votre réseau
* Lancer le vagrant depuis le dossier où se trouve les fichiers avec ```vagrant up```
* Accéder depuis votre hôte au jenkins sur localhot:8014 ou bien depuis l'adresse IP publique configurée dans le vagrant file sur le port 8080

## Configuration du Jenkins
* Récupérer le mot de passe initial de l'admin à l'aide du chemin fourni
* Configurer les plugins (vous pouvez laisser les plugins recommandés)
* Configurer votre compte admin
* Vous êtes prêt à utiliser votre jenkins :) 



## Configurer le premier job 
* Une fois connecter Jenkins vous propose de créer un job (si des jobs existent, utiliser le menu Add new item) 
* Cliquer sur freestyle project, puis sur suivant
* Dans la rubrique Source code management, choisir Git et rentrer l'URL du projet :
https://github.com/vanessakovalsky/laravel-kingoludo.git
* Dans credentials, laissez None (pas besoin d'identifiant pour cloner le projet)
* Cliquer sur Save 
* Vous arrivez sur la page du Job
* Vous pouvez executer le job en cliquant sur Build now
* Le job se lance, cliquez dessus pour avoir des informations
* En allant dans Console Output, que voyez vous ? 
```
Started by user Vanessa
Running as SYSTEM
Building in workspace /var/lib/jenkins/workspace/Test-PHP
No credentials specified
Cloning the remote Git repository
Cloning repository https://github.com/vanessakovalsky/laravel-kingoludo.git
 > git init /var/lib/jenkins/workspace/Test-PHP # timeout=10
Fetching upstream changes from https://github.com/vanessakovalsky/laravel-kingoludo.git
 > git --version # timeout=10
 > git fetch --tags --progress -- https://github.com/vanessakovalsky/laravel-kingoludo.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/vanessakovalsky/laravel-kingoludo.git # timeout=10
 > git config --add remote.origin.fetch +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git config remote.origin.url https://github.com/vanessakovalsky/laravel-kingoludo.git # timeout=10
Fetching upstream changes from https://github.com/vanessakovalsky/laravel-kingoludo.git
 > git fetch --tags --progress -- https://github.com/vanessakovalsky/laravel-kingoludo.git +refs/heads/*:refs/remotes/origin/* # timeout=10
 > git rev-parse refs/remotes/origin/master^{commit} # timeout=10
 > git rev-parse refs/remotes/origin/origin/master^{commit} # timeout=10
Checking out Revision 43b04e5d53edac5d010ae78353207bce9ed145a9 (refs/remotes/origin/master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f 43b04e5d53edac5d010ae78353207bce9ed145a9 # timeout=10
Commit message: "Add vue component"
First time build. Skipping changelog.
Finished: SUCCESS
```
* Le job est maintenant prêt, félicitations, votre premier job Jenkins est configuré et opérationnel

## Ajouter le build de gradle à notre job
* Retourner dans le build
* Cliquer à gauche dans le menu sur Configure
* Dans la section Build, choisir Add step
* Choisir Invoke Gradle script
* Dans Tasks entrer la tâche gradle à executer : installDeps
* Sauvegarder et lancer un build
* En consultant la sortie Console Output, vous verrez que l'installation des dépendances a bien été effectuée
* Vous pouvez maintenant ajouter les différentes étapes du build : test et publish 
* Votre job est maintenant constitué de 3 étapes :
* * l'installation des dépendances 
* * le lancement des tests
* * la construction d'une archive zip et son déploiement dans Nexus OSS

## Un job pour votre projet
* Créer maintenant un job jenkins pour votre projet
* Ajouter la récupération du code source
* Ajouter les différentes étapes de build depuis le script gradle créé précédement (il est nécessaire que les outils que vous utilisiez soit installer sur le serveur jenkins) + pensez à modifier les paratmètres de votre gradle.properties, notamment l'adresse du depôt Nexus

## Configurer Github pour déclencher un build à chaque commit
* Vous avez besoin d'un token d'API de Jenkins pour configurer le webhook dans github
* Sur jenkins, cliquer sur votre nom en haut à droite
* Puis sur Configure dans le menu
* Dans la section API Token, cliquer sur Add new token, et copier le token généré
* Aller sur votre projet Github
* Dans les paramètres, choisir Webhook
* Cliquer sur Add webhook :
* * Dans payload URL, entrer l'adresse de Jenkins sous la forme http://[login]:[APITOKEN]@[URLduJenkins.com:8080]/github-webhook
* Enregistrer
* Côté Jenkins, aller dans le job
* Dans la rubrique Build Triggers, cochez la case : GitHub hook trigger for GITScm polling 

-> Votre job est configuré pour être lancé à chaque push sur la branche master dans votre dépôt Github

## Ajouter du contrôle qualité à votre code

## Ajouter des contrôle qualité à notre code
* Pour commencer il faut séléctionner les outils que nous allons utiliser :
* * phploc: fournit des informations sur le nombre de ligne de code et la complexité de celui ci
* * phpcpd : fournit des informations sur les copier coller
* * phpmd : détection de potentiel problème dans le code (bugs, complexité, paramètres et méthodes inutilisés)
* * phpdox : permet de générer de la documentation à partir des commentaires du code
* * phpcs : permet d'analyser le code et de détecter les écarts avec les normes de codage
* Nous allons donc ajouter l'appel à ces différents outils dans notre build.gradle :
```
task phploc(type: Exec, description: 'Measure project size using PHPLOC') {
	executable 'sh'
	args '-c',"/home/ubuntu/tools/phploc --count-tests \
    --log-csv='./build/reports/phploc.csv' \
    --log-xml='./build/reports/phploc.xml' \
     ./app"
}

task phpmd (type : Exec, description : "Perform project mess detection using PHPMD, and Creating a log file for the Continuous Integration Server"){
	executable 'sh'
	args '-c',"/home/ubuntu/tools/phpmd ./app \
    xml './phpmd.xml' \
    --reportfile './build/reports/phpmd.xml'\
    --ignore-violations-on-exit"
}

task phpcpd (type : Exec, description : "Find duplicate code using PHPCPD and log result in XML format."){
	executable 'sh'
	args '-c',"/home/ubuntu/tools/phpcpd \
    --log-pmd './build/reports/pmd-cpd.xml' \
    ./app"
}

task phpcs (type : Exec, description : "Find Coding Standard Violations using PHP_CodeSniffer"){
	executable 'sh'
	args '-c',"/home/ubuntu/tools/phpcs \
    --report=checkstyle \
    --report-file='./build/reports/checkstyle.xml' \
    --standard=PSR2 \
    --extensions=php \
    --ignore=autoload.php \
    --runtime-set ignore_errors_on_exit 1 \
    --runtime-set ignore_warnings_on_exit 1 \
    ./app/"
}

task phpdox (type : Exec, description : "Generating Docs enriched with pmd, checkstyle, phpcs,phpunit,phploc"){
	executable 'sh'
	args '-c',"/home/ubuntu/tools/phpdox \
    -f ./phpdox.xml"
}
```
* NB : le fichier buid.gradle a été ajouté au dépôt source laravel-kingdomino puisqu'il est lié à ce projet
* NB2 : pour executer ce script, il est nécessaire que les outils utilisés soient installés sur la machine (pour l'éxecuter en local, il faudra donc modifier l'environnement d'execution de gradle pour installer ces outils, ce n'est pas fait dans cet exemple)

## Intégrer l'analyse qualité à notre job Jenkins
* Dans le build de notre projet, ajouter les étapes correspondantes au script gradle :
* * phpcs
* * phpmd
* * phpdox
* * phpcpd
* * phploc 
* Lancer un build, puis allez voir dans le workspace du produit :
--> Les résultats de ses différentes étapes sont maintenant disponibles sous forme de fichier XML ou dans la sortie Console Output
* NB : pour pouvoir faire tourner ces taches, les outils ont été installé sur le serveur qui fait tourner Jenkins et sur les différents agents qui executent les jobs

## Afficher les résultats sous formes de rapports
* Pour afficher des rapports nous allons avoir besoin de différents plugins de Jenkins, qui permettent à partir des logs générés au format XML de créer des rapports graphiques :
* * Warnings Next Generation : récupère de nombreux logs et affiche au format graphique (CPD, MD, CS)
* * xUnit : récupère les logs des tests et affiche un rapport
* * Plot : affiche des graphiques à partirs de données au format CSV (PHPLOC)
* * HTML Publisher : pour publier la documentation généré par PHPDox
* Pour installer les plugins, rendez vous dans Manage Jenkins > Plugin Manager 
* Chercher et cocher ces 3 plugins 
* Cliquer sur Install without restart
* Il reste maintenant à ajouter les étapes de buildsp our la publication des rapports
* Retourner dans la configuration du Job
* Dans l'onglet Post Build, ajouter une action
* *  Plot Build data :
* * * Data serie file : chemin vers le fichier CSV, ici : build/reports/phploc.csv
* * * Choisir Load Data from CSV File
* * Record Compiler warning and static analysis results
* * * Static Analysis
* * * * Tools : PHP Code sniffer
* * * * Report file pattern :build/reports/checkstyle.xml
* * * Cliquer sur add tool
* * * * Tool : CPD
* * * * Report file pattern : build/reports/pmd-cpd.xml
* * * Cliquer sur add tool
* * * * Tool : PMD
* * * * Report file pattern : build/reports/phpmd.xml
* * Publish xUnit test result report
* * * Report type  : PHP Unit
* * * Patter : build/reports/unitreport.xml
* * Publish HTML Report
* * * HTML directory to archive	: reports/api

--> Votre chaine est maintenant configuré avec l'analyse qualité et la publication des rapports associés côté Jenkins, félicitations :)
