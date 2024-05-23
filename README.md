# Log book stage Computo

# Missions:

Mission n°-1: [lien_du_dépôt-1]()
Mission n°0: [Lien_dépôt0](https://github.com/Qufst/Maj_yml)
L'objectif de ce dépôt est d'analyser les dépendances nécessaires au lancement des fichiers '.qmd', '.ipynb' et '.py' et de mettre à jour le fichier 'environment.yml' pour pouvoir previsualiser le '.qmd'. 
Mission n°1: [Lien_dépôt1](https://github.com/Qufst/test-imbrication-de-workflows)
L'objectif de ce dépôt est de trouver une méthode pour centraliser dans un dépôt le lancement de plusieurs autres dépôts et de récupérer des informations sur ceux-ci.

# Activité quotidienne

## Mission n°0

Idée: utiliser le workflows pour transformer tous les fichiers '.qmd' et '.ipynb' en fichiers '.py', lire les dépendances sur les fichiers '.py', et mettre à jour l'environnement virtuel avant de previsualiser le '.qmd'

Problème1: Le code semble mettre à jour les dépendances virtuelles, cependant elles sont au nombre de 1000 ce qui est très étonnant. Ceci est probablement la cause du lancement interminable de l'environnement derrière. 


## Mission n°1

Idée: Introduire dans le workflows du dépôt principal une méthode qui permet de faire fonctionner les workflows des dépôts cibles et de récupérer les résultats nécessaires.

Problème 1: Comment agir sur un dépôt dont on a les droits depuis un autre dépôt qui nous appartient également?
solution que je n'ai pas réussi à faire aboutir: utilisation d'un TOKEN pour lancer directement le workflows du dépôt cible
solution utilisée: utilisation d'un TOKEN pour faire un commit vide sur l'autre répertoire. Le workflows du répertoire cible possède une configuration permettant de se lancer à chaque commit.
Le TOKEN dure seulement 30j, voir si j'en crée un plus long ?

On a donc pour l'instant un dépôt qui installe les dépendances pour quarto et Computorg, et qui arrive à éxécuter un autre répertoire nous appartenant. On veut désormais récupérer les render du dépôt cible.

Problème 2: Comment avec un workflows faire fonctionner d'autres workflows:
solution pour un seul dépôt: Dans le même dépôt j'ai plusieurs workflows. Un workflows principal qui lance plusieurs workflows situés dans le même dépôt qui s'éxécutent donc en même temps indépendamment les uns des autres. [workflows_principal](https://github.com/Qufst/test-imbrication-de-workflows/.github/workflows/main_workflows.yml)

Problème 3: Comment faire fonctionner des workflows situés sur des dépôts publics m'appartenant:

