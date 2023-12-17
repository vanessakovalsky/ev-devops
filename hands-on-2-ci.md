# mise en place d’une CI avec analyse de code, build d’une application, depot dans un registre d’artefact

## Définition du build

## Pré-requis
Il est possible de faire ses exercices soit dans des conteneurs (voir docker ci-dessous) soit dans des VM vagrant (voir Vagrant ci-dessous) soit d'installer les outils en local (voir les sites officiels pour les procédures d'installation)

### Docker
* Afin d'éviter les installations trop nombreuses, nous allons utiliser docker et docker compose
* Installer docker :
https://docs.docker.com/get-docker/
* Installer docker-compose :
https://docs.docker.com/compose/install/ 
* Récupérer le fichier DockerFile : 
```
git clone https://github.com/vanessakovalsky/continuous-integration-training.git
```
* Récupérer le projet : 
```
cd exercice-2-ci-gradle-php-laravel
git clone https://github.com/vanessakovalsky/laravel-kingoludo.git
```   
* Build et Lancement du conteneur docker:
```
docker build --tag my-gradle .
```

### Vagrant

* Récupérer le fichier VagrantFile et install.sh qui se trouve dans le dossier vagrant/gradle
* Installer Vagrant : https://www.vagrantup.com/ 
* Dans un terminal se positionner dans le dossier contenant les deux fichiers récupérer et lancer la commande : ```vagrant up```

## Exemple de build avec un projet PHP
* Si ce n'est pas fait récupérer le code source de votre projet, ici :
```
git clone https://github.com/vanessakovalsky/laravel-kingoludo.git project
```
* Dans votre IDE, créer à la racine du projet un fichier build.gradle contenant :
```

buildscript {
  repositories {
    gradlePluginPortal()
    maven {
      url "https://plugins.gradle.org/m2/"
    }
  }
  dependencies {
    classpath "gradle.plugin.org.swissphpfriends:php-build-plugin:0.1-SNAPSHOT"
  }
}

apply plugin: "org.hasnat.php-build-plugin"
apply plugin: "distribution"

def majorVersion = System.getenv("MAJOR_VERSION") ?: "1"
def minorVersion = System.getenv("MINOR_VERSION") ?: "0"
version = majorVersion + "." + minorVersion 

task purge(type:Delete) {
  //println 'Cleaning up old files'
  delete 'vendor', 'logs', 'build'
}

task composer(type:Exec, dependsOn: purge) {
  //println 'Setting up dependencies'
  executable 'sh'
  args '-c', 'php -r "readfile(\'https://getcomposer.org/installer\');" | php'
  standardOutput = new ByteArrayOutputStream()
  ext.output = { return standardOutput.toString() }
}
task vendor(type:Exec, dependsOn: composer) {
  //println 'Installing dependencies'
  executable 'sh'
  args '-c', 'php composer.phar update'
  standardOutput = new ByteArrayOutputStream()
  ext.output = { return standardOutput.toString() }
}


def tarfile = "application-" + version
task packageDistribution(type: Zip, dependsOn: vendor) {
    from ('app') { into 'app' }
    from ('bootstrap') { into 'bootstrap' }
    from ('config') { into 'config' }
    from ('database') { into 'database' }
    from ('nbproject') { into 'nbproject' }
    from ('public') { into 'public' }
    from ('resources') { into 'resources' }
    from ('storage') {
        into 'storage'
        dirMode 0775
      }
    from ('vendor') { into 'vendor' }
    from { 'server.php' }
    archiveFileName = tarfile + ".zip"
    destinationDirectory = file("build")
}
```
* Si on détaille un peu ce fichier :
* * apply plugin permet d'aller chercher le plugin de gradle qui permet de packager
* * def permet de définir des variables à utiliser (ici les numéros de version)
* * task purge : supprimer les anciens fichiers pour éviter les conflit
* * task composer : installe la dernière version de composer 
* * task vendor : utilise composer pour aller chercher les dépendances de notre projet
* * distribution permet de choisir les fichier à packager, le nom du package, c'est ce pagckage qui est appeler avec la commande gradle applicationDistTar
* * task tarball permet de choisir le format et la destination de sortie
* Pour éxecuter le build, il faut relancer le build avec la commande docker build utiliser auparavant
* Il est nécessaire de lancer les taches une par une, ce qui est un peu fastidieux
* Pour éviter ça on rajoute des dépendances entre les taches :
```
task purge (type:Delete) {
    [...]
}
task composer(type:Exec, dependsOn: purge) {
    [...]
}
task vendor(type:Exec, dependsOn: composer) {
    [...]
}
[...]
task packageDistribution(type: Zip, dependsOn: vendor) {
    [...]
}
```
*Lancer seulement la dernière tache dans votre environnement :
```
gradle packageDistribution --no-demon --info
```
* Pour découvrir l'ensemble des options, aller voir la documentation de gradle :
https://docs.gradle.org/current/userguide/userguide.html 

## Créer un build pour votre propre projet
* Définisser l'ensemble des actions nécessaires pour builder votre projet
* créer un build.gradle qui décrit ses taches en utilisant les plugins de gradle si nécessaire : https://plugins.gradle.org/ 

-> Pour aller plus loin d'autres exercices sont disponible ici : 
https://github.com/praqma-training/gradle-katas

## Dépôt Nexus

## Mise en place de nexus
Lire le paragraphe qui concerne votre environnement (Docker ou Vagrant)

### Docker
* Récupérer le fichier docker-compose.yml dans exercice-3-ci-nexus et placer le dans votre dossier de travail
* Lancer le docker compose (depuis le dossier de travail)
```
docker-compose up -d
```
* Vérifier que les deux conteneurs sont lancés :
```
docker-compose ps 
```
* Nexus prend quelque minutes à démarrer complètement, pour vérifier utiliser :
```
docker logs -f nexus
```
* Cette commande doit afficher :
```
Started Sonatype Nexus OSS 3.23.0-03
```
* Une fois les conteneurs lancés, nexus est accessible à l'url :
http://localhost:8099
* Récupérer le mot de passe (dans un terminal) :
```
docker exec -t -i nexus sh
cat nexus-data/admin.password
```
* Noter le mot de passe afficher
* Il est indispensable de récupérer l'IP du container Nexus, pour cela on utilise docker network inspect qui permet d'avoir des informations sur le réseau :
```
docker network inspect 
```

### Vagrant
* Récupérer le Vagrantfile et le fichier install.sh dans le dossier vagrant/nexus de ce dépôt
* Ouvrir un terminal et se positionner dans le dossier contenant les deux fichiers que l'on vient de récupérer puis lancer un ```vagrant up```
* Choisir votre interface réseau
* Patientez le temps que la VM démarre
* Une fois la VM démarré en dernier, vous devriez avoir le mot de passe admin affiché, si ce n'est pas le cas : 
* COnnexter vous sur la VM (vagrant ssh)
* Aller afficher le mot de passe qui se trouve dans : /opt/nexus/sonatype-work/nexus3/admin.password
* Noter le mot de passe
* Aller sur http://192.168.0.44 

## Configuration (pour tous les environnements)
* Se connecter (login : admin, pass : (récupérer à l'étape précédente))
* Suivre les 4 étapes de configuration
* Quels sont les différents repositories existants ?
* Créer un repository de type Hosted Repository, qui nous servira à déposer nos builds

## Utilisation de nexus sur un projet PHP
* Nous allons utiliser notre repository pour publier les archives de notre application

* Dans les lignes affichés, on cherche Nexus et la ligne contenant IP/V4
* Créer un fichier gradle.properties à la racine du projet, il contient les informations de connexion (adapter l'IP à votre environnement)
```
# Maven repository for publishing artifacts
nexusRepo=172.22.0.3:8081/repository/maven-releases/
nexusUsername=admin
nexusPassword=VotrePassword
```
* Dans le build.gradle ajouter la publication dans nexus :
```
apply plugin: "maven-publish"
[...]
group = 'laravel-kingoludo'

publishing {
    repositories {
        maven {
            url nexusRepo
            allowInsecureProtocol = true
            credentials {
                username = nexusUsername
                password = nexusPassword
            }
        }
    }

    publications {
        maven(MavenPublication) {
            artifact packageDistribution
           
        }
    }
}
```
* Quelques explications sur la tache ci-dessus :
* * Pour publier il est obligatoire de donner un nom à un group de projet (le nom de votre choix ici laravel-kingoludo)
* * La partie suivante définie les options pour la publication 
* * publication définit le type de publication et la source (le fichier à déployer)
* * repositories définit la destination avec les informations de connexions et l'url (qui sont récupérés dans le gradle.properties à adapter avec vos informations)
* Lancer la publication dans le conteneur ou la vm
```
gradle generatePomFileForMavenPublication
gradle publish
```
* Vérifier dans nexus que votre archive zip a bien été publiée :)

## Utilisation de nexus sur votre propre projet
* Créer le repository (hosted) pour votre propre projet dans nexus
* Paramètrer le repository 
* Publier votre artifact dans nexus depuis gradle

## Pour aller plus loin (si vous avez le temps ou à faire plus tard)
* Mettre en place un repository proxy sur nexus qui va récupérer vos dépendances
* Utiliser ce repository proxy pour récupérer vos dépendances dans votre script gradle

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
