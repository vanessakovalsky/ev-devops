# Créer des ressources sur AWS avec Terraform

## Pré-requis
* Afin de pouvoir utiliser un fournisseur, vous devez avoir un compte chez celui et les informations de connexion à ce compte
* De plus le compte que vous utilisez doit avoir les droits de créer la ressource que Terraform va crée

## Configuration du provider

* Commençons par configurer le provider (fournisseur) que vous souhaitez utiliser. Créer un fichier nommé main.tf. Ensuite, mettre le code suivant (en adaptant avec vos informations d'accès):
```
provider "aws" {
    region = "us-east-2"
    access_key = "votre-clé-dacces"
    secret_key = "votre-clé-secrète"
}
```
* Nous indiquons à Terrform différente information :
    - le provider avec le label "aws'
    - la region dans laquelle nos ressources seront créé
    - nos informations d'accès (pour l'instant ce n'est pas très sécurisé, nous sécuriserons cela plus tard lors d'un autre exercice)


## Ajout de la ressource

* Chaque fournisseur propose différents types de ressources. 
* Commençons par une ressource simple, une machine virtuelle appelé instance EC2 sur AWS
* Ajouter le code suivant à votre fichier main.tf :
```
resource "aws_instance" "my_ec2_instance" {
    ami = "ami-07c1207a9d40bc3bd"
    instance_type = "t2.micro"
}
```
* La syntaxe générale d'une ressource Terraform est la suivante :
```
resource "<FOURNISSEUR>_<TYPE>" "<NOM>" {
    [CONFIG …]
}
```
    * FOURNISSEUR : c'est le nom d'un fournisseur (ici le provider "aws").
    * TYPE : c'est le type de ressources à créer dans ce fournisseur (ici c'est une instance ec2)
    * NOM : c'est un identifiant que vous pouvez utiliser dans le code Terraform pour faire référence à cette ressource (ici "my_ec2_instance")
    * CONFIG : se compose de un ou plusieurs arguments spécifiques à cette ressource, dans notre cas :
        * ami : c'est l'acronyme d'"Amazon Machine Image" (AMI) , c'est donc l'image qui sera exécutée sur notre instance EC2.
        * instance_type : Type d'instance EC2 à exécuter.


## Lancement du déploiement

* Pour lancer notre déploiement, nous ouvrons un terminal et nous mettons dans le dossier contenant notre fichier main.tf
* Puis exécuter la commande :
```
terraform init
```
* Vous obtenez un résultat ressemblant à :
```
Initializing the backend...

Initializing provider plugins...
- Checking for available provider plugins...
- Downloading plugin for provider "aws" (hashicorp/aws) 2.57.0...
...
...
Terraform has been successfully initialized!
```
* Un nouveau fichier caché .terraform est alors apparu dans votre dossier
* Demander maintenant à terraform de créer le plan d'éxecution, qui détermine les actions nécessaires pour atteindre l'état souhaité dans votre fichier main.tf . 
* Pour cela exécuter la commande
```
terraform plan
```
* Qui donne un résultat similaire à :
```
Refreshing Terraform state in-memory prior to plan...
...
Resource actions are indicated with the following symbols:
  + create

Terraform will perform the following actions:

  # aws_instance.my_ec2_instance will be created
  + resource "aws_instance" "my_ec2_instance" {
      + ami                          = "ami-07c1207a9d40bc3bd"
      +  ...
      +  ...
    }

Plan: 1 to add, 0 to change, 0 to destroy.
"terraform apply" is subsequently run
```
* Vous voyez alors différentes informations : 
    * des ressources ajoutées avec le symbole + 
    * des ressources supprimées avec le symbole - 
    * des ressources modifiées avec le symbole ~
* Ici nous avons seulement un ajout
* Enfin, demander à terraform d'éxecuter le plan en utilisant la commande :
```
terraform apply
```
* Qui donne un résultat similaire à (pensez à taper yes pour approuver l'application) :
```
Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_instance.my_ec2_instance: Creating...
aws_instance.my_ec2_instance: Still creating... [10s elapsed]
aws_instance.my_ec2_instance: Still creating... [20s elapsed]
aws_instance.my_ec2_instance: Still creating... [30s elapsed]
aws_instance.my_ec2_instance: Creation complete after 39s [id=i-0f6d6ee734f745e22]
```

## Vérification du déploiement 

* Vous pouvez ouvrir la console web AWS ou utiliser AWS cli pour vérifier que votre instance est bien crée


## Modification du déploiement

* Nous allons maintenant modifier le déploiement et rajouter quelques informations à notre instance :
```
provider "aws" {
    region = "us-east-2"
    access_key = "votre-clé-dacces"
    secret_key = "votre-clé-secrète"
}

resource "aws_instance" "my_ec2_instance" {
    ami = "ami-07c1207a9d40bc3bd"
    instance_type = "t2.micro"

	user_data = <<-EOF
		#!/bin/bash
        sudo apt-get update
		sudo apt-get install -y apache2
		sudo systemctl start apache2
		sudo systemctl enable apache2
		sudo echo "<h1>Hello students</h1>" > /var/www/html/index.html
	EOF
    
    tags = {
        Name = "terraform test"
    }
}
```
* Ici nous avons rajouter deux éléments à notre instance :
    * L'installation d'un serveur web apache et son démarrage
    * Un tag pour retrouver plus facilement notre instance

* Notre instance sera donc prête à nous afficher une page web.
* Pour pouvoir accéder à celle-ci, nous devons ajouter un groupe de sécurité et ouvrir le port adéquat
* Ajouter a votre fichier main.tf au dessus de votre ressource instance la ressource ci-dessous
```
resource "aws_security_group" "instance_sg" {
    name = "terraform-test-sg"

    egress {
        from_port       = 0
        to_port         = 0
        protocol        = "-1"
        cidr_blocks     = ["0.0.0.0/0"]
    }

    ingress {
        from_port   = 80
        to_port     = 80
        protocol    = "tcp"
        cidr_blocks = ["0.0.0.0/0"]
    }
}

```
* Il nous reste à dire à AWS d'utiliser ce groupe de sécurité pour notre instance, en ajoutant sa référence.
* Les ressources créés renvoés des attributs qui nous permettent de les utliser en référence.
* La syntaxe est alors : 
```
<FOURNISSEUR>_<TYPE>.<NOM>.<ATTRIBUT>
```
    * FOURNISSEUR : le nom du fournisseur (ici "aws").
    * TYPE : le type de la ressource (ici "security_group").
    * NOM : le nom de cette ressource (ici "instance_sg").
    * ATTRIBUT : l'un des arguments de cette ressource (par exemple "name") ou l'un des attributs exportés (ici c'est l'attribut "id" qui est concerné).

* Il faudra donc modifier votre instance pour lui donner la référence vers le groupe de sécurité créé :
```
resource "aws_instance" "my_ec2_instance" {
    ami = "ami-07c1207a9d40bc3bd"
    instance_type = "t2.micro"
    vpc_security_group_ids = [aws_security_group.instance_sg.id]

	user_data = <<-EOF
		#!/bin/bash
        sudo apt-get update
		sudo apt-get install -y apache2
		sudo systemctl start apache2
		sudo systemctl enable apache2
		echo "<h1>Hello devopssec</h1>" > /var/www/html/index.html
	EOF
    
    tags = {
        Name = "terraform test"
    }
}

```
* En utilisant une référence via un attribut, vous créer une dépendance implicite entre les ressource. 
* Terraform utilise ses dépendances pour construire un graphique de dépendance et déterminer l'ordre dans lequel il doit créer les ressources. 
* Nous pouvons maintenant déployer notre groupes de sécurité et recréer notre instance :
```
terraform init && terraform apply
```
* Qui donnera un résultat similaire à :
```
Terraform will perform the following actions:
  # aws_security_group.instance_sg will be created
  + resource "aws_security_group" "instance_sg" {
      ...
      + ingress                = [
          + {
              + cidr_blocks      = [
                  + "0.0.0.0/0",
                ]
              + from_port        = 80
              + protocol         = "tcp"
              ...
              + to_port          = 80
            },
        ]
    }
...
aws_instance.my_ec2_instance: Modifications complete after 9s [id=i-0b25fb9f0b17211e7]
```
* Vérifier maintenant dans la console web d'AWS que votre groupe de sécurité est bien crée avec les bonne règles
* Puis retrouver votre instance, et ouvrez son adresse IP, une page avec le message "Hello students" devrait s'afficher

## Suppression de la ressource
* Les exercices étant terminer, nous allon supprimer les ressources avec la commande 
```
terraform destroy
```
* Qui donnera un résultat similaire à :
```
Terraform will perform the following actions:

  # aws_instance.my_ec2_instance will be destroyed
  - resource "aws_instance" "my_ec2_instance" {
        ...
    }

  # aws_security_group.instance_sg will be destroyed
  - resource "aws_security_group" "instance_sg" {
        ...
    }

  Enter a value: yes

aws_instance.my_ec2_instance: Destroying... [id=i-0b25fb9f0b17211e7]
aws_instance.my_ec2_instance: Still destroying... [id=i-0b25fb9f0b17211e7, 10s elapsed]
aws_instance.my_ec2_instance: Still destroying... [id=i-0b25fb9f0b17211e7, 20s elapsed]
aws_instance.my_ec2_instance: Destruction complete after 22s
aws_security_group.instance_sg: Destroying... [id=sg-07e6921e0c5dc4ca4]
aws_security_group.instance_sg: Destruction complete after 2s

Destroy complete! Resources: 2 destroyed.
```

## Pour aller plus loin :

* [Gérer des variables d'entrée / sortie](https://github.com/vanessakovalsky/terraform-training/blob/main/aws/exercice-2-variables.md)
* [Récupérer l'identifiant de l'image avec une data source](https://github.com/vanessakovalsky/terraform-training/blob/main/aws/exercice-3-datasource.md)
* [Rendre notre code réutilisable avec des modules](https://github.com/vanessakovalsky/terraform-training/blob/main/aws/exercice-4-module.md)
