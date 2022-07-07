# SonarQube

Le SonarQube est une plateforme open source développée par SonarSource pour l'inspection continue de la qualité du code afin d'effectuer des revues automatiques avec analyse statique du code pour détecter bogues.

## Installation dans Openshift 

<!--Commencez par la création du projet dans l'interface d'utilisateur dans le Portail Openshift. --> 

Faites login sur le Portail Openshift. 

D'abord, copiez la commande de login remote. Cliquez sur votre nom d'usager, suivi par `Copy login command`. La plateforme vous demandera votre identifiant du site, et ensuite montre le lien `Display Token`. Cliquez sur le lien et copiez la valeur dans la boîte identifiée `Log in with this token`. 

Allez sur la ligne de commandes de votre environnement de développement, et collez le token. Vous êtes connecté à la plateforme via le client Openshift `oc`. 


```
oc new-project sonarqube-torjc01 --display-name="SonarQube torjc01" --description="Le SonarQube est une plateforme open source développée par SonarSource pour l'inspection continue de la qualité du code afin d'effectuer des revues automatiques avec analyse statique du code pour détecter bogues."
```

### Création d'une pvc (Persistent volume claim)

Pour créer la PVC, on recommande de le faire via l'interface d'usager du Portail Openshift. 

Allez dans `Administrator > Storage > Persistent Volume Claim` et cliquez sur le bouton `Create PersistentVolumeClaim`. 

Informez les valeurs suivantes: 

```
Storage class: ocs-storagecluster-gp2

Persistent Volume Claim name: <le nom choisi pour votre pvc> (par example, sonarqube-sauvegarde-claim)

Access mode: Shared Access (RWX)

Size: 1 Gb (ou toute autre valeur qui soit pertinente)

Volume Mode: Filesystem
```

À partir de la ligne de commande dans la racine du projet, exécutez les commandes qui suivent: 

```sh

# Permettre l'installation de DockerFile en mode root:
oc adm policy add-scc-to-user anyuid -z default

# Déployer les pods dans Openshift
$ oc process -f ./openshift/templates/sonarqube-postgresql.yaml | oc apply -f -
$ oc process -f ./openshift/templates/backup.yaml | oc apply -f - 

```

### Création du pod attaché au persistentVolumeClaim: 

```
Commande: oc process -f openshift/templates/backup.yaml | oc apply -f -
Ex: oc process -f openshift/templates/backup.yaml | oc apply -f -
```

### Création de la cronjob

Pour la création de la cronjob, il faut passer les paramètres de la BD, bien que le pvc créé pour le storage des fichiers pris en backup. 

La commande exige les paramètres obligatoires ci-dessous: 

```
DATABASE_HOST - le nom du container qui contient la BD
DATABASE_PORT
DATABASE_NAME
DATABASE_USER
DATABASE_PASSWORD
DATABASE_BACKUP_VOLUME - le nom du persistent volume claim qui contiendra les fichiers générés. 
```

Par la suite, soumettre la commande suivante pour faire la création de la cronjob dans le projet:  

``` 
Commande: 
oc process -f openshift/templates/cronjob.yaml DATABASE_USER=<user> DATABASE_PASSWORD=<pass> DATABASE_HOST=<nom containeur BD> DATABASE_NAME=<nom de la BD> DATABASE_PORT=<port> DATABASE_BACKUP_VOLUME_CLAIM=<id du PVC> | oc create -f -

Exemple:  
oc process -f openshift/templates/cronjob.yaml DATABASE_USER=sonar DATABASE_PASSWORD=motdepassebd DATABASE_HOST=postgresql-sonarqube DATABASE_NAME=sonar DATABASE_PORT=5432 DATABASE_BACKUP_VOLUME_CLAIM=sonarqube-sauvegarde-claim | oc create -f -
```
