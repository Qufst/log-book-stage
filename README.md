# Log book stage Computo

# Missions:

Mission n°1: [Lien_dépôt1](https://github.com/Qufst/test-imbrication-de-workflows)
L'objectif de ce dépôt est de trouver une méthode pour centraliser dans un dépôt le lancement de plusieurs autres dépôts et de récupérer des informations sur ceux-ci.

# Activité quotidienne

## Mission n°1

Idée: Introduire dans le workflows du dépôt principal une méthode qui permet de faire fonctionner les workflows des dépôts cibles et de récupérer les résultats nécessaires.

Problème 1: Comment agir sur un dépôt dont on a les droits depuis un autre dépôt qui nous appartient également?
solution que je n'ai pas réussi à faire aboutir: utilisation d'un TOKEN pour lancer directement le workflows du dépôt cible
solution utilisée: utilisation d'un TOKEN pour faire un commit vide sur l'autre répertoire. Le workflows du répertoire cible possède une configuration permettant de se lancer à chaque commit.
Le TOKEN dure seulement 30j, voir si j'en crée un plus long ?

On a donc pour l'instant un dépôt qui installe les dépendances pour quarto et Computorg, et qui arrive à éxécuter un autre répertoire nous appartenant. On veut désormais récupérer les render du dépôt cible.

