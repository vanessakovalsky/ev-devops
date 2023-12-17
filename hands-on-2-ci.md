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

