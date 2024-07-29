# Log book stage Computo

# Missions:

Mission n°-1: [lien_du_dépôt-1]()
Mission n°0: [Lien_dépôt0](https://github.com/Qufst/Maj_yml)
L'objectif de ce dépôt est d'analyser les dépendances nécessaires au lancement des fichiers '.qmd', '.ipynb' et '.py' et de mettre à jour le fichier 'environment.yml' pour pouvoir previsualiser le '.qmd'. 
Mission n°1: [Lien_dépôt1](https://github.com/Qufst/test-imbrication-de-workflows)
L'objectif de ce dépôt est de trouver une méthode pour centraliser dans un dépôt le lancement de plusieurs autres dépôts et de récupérer des informations sur ceux-ci.
Articulation des workflows: - [template_python](https://github.com/Qufst/template-computo-python)
                            - [dépôt_workflows](https://github.com/Qufst/Workflows_computorg)
                            - [version_finale_computo](https://github.com/computorg/workflows/tree/main/.github/workflows)

Création d'image apptainer: [creation](https://github.com/Qufst/create_apptainer.sif)
Run les images apptainer: [run](https://github.com/Qufst/run_sif_and_deploy)


# Interêt aux workflows

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

## Mission n°2: utiliser la mission 1 et appliquer ce processus au template_python:

Problème 1: utiliser l'environnement créé dans un workflows dans un second workflows
J'ai mon workflows principal qui lance un second workflows. Celui-là crée l'environnement selon les dépendances pythons, puis lance le workflows publish qui installe les dépendances quarto computo et publie l'article. Cependant je n'arrive pas à réutiliser l'environnement du workflows 2 dans le workflows publish. Donc j'ai essayé de téléchargé l'environnement à un endroit spécifique du dépôt avec la commande 'python -m venv ./abc', et de l'activer dans le dernier workflows avec la commande 'source ./abc/bin/activate', mais github me répond qu'il ne trouve pas l'environnement, malgré que l'éxécution du second workflows se passe bien et que l'environnement soit créé. 

Autre idée: Chaque commit déclenche main_workflows -> python_workflows: installe les dépendances, crée un artifact et lance un commit vide s'appelant "py_commit" ce qui déclenche main_workflows -> quartopublish_workflows: active l'environnement grâce à l'artifact, installe quarto et dépendances, lance la publicaiton.


      - name: Archive the environment
        run: |
          zip -r abc.zip ./abc
      - name: Publish artifacts
        uses: actions/upload-artifact@v4
        with:
          name: my-artifact1
          path: ./abc.zip



      - name: List artifacts
        uses: actions/github-script@v4
        with:
          script: |
            const response = await github.actions.listArtifactsForRepo({
              owner: context.repo.owner,
              repo: context.repo.repo
            });
            response.data.artifacts.forEach(artifact => {
              console.log(`Artifact: ${artifact.name}`);
            });
      - name: Download artifacts
        uses: actions/download-artifact@v4
        with:
          name: 'my-artifact1'
          github-token: ${{ secrets.GITHUB_TOKEN }}

      - name: Unzip the artifact
        run: |
          unzip abc.zip
          
      - name: Activate virtual environment
        run: |
          source ./abc/bin/activate


      - name: Check commit message
        run: |
          if [[ "$(git log --format=%B -n 1 HEAD)" != "py_commit" ]]; then
            echo "Proceeding with python_workflows..."
            # Trigger python_workflows.yml
            gh workflow run python_workflows.yml -f config-path='./config/python_config.yml'
          fi
          if [[ "$(git log --format=%B -n 1 HEAD)" == "py_commit" ]]; then
            echo "Commit message is 'py_commit'. Triggering quartopublish_workflows..."
            # Trigger quartopublish_workflows.yml
            gh workflow run quartopublish_workflows.yml -f config-path='./config/quarto_config.yml'
          fi


      - name: clone
        env:
          GITHUB_TOKEN: ${{ secrets.REPO_DISPATCH_TOKEN }}
          GITHUB_REPOSITORY_URL: https://x-access-token:${{ secrets.REPO_DISPATCH_TOKEN }}@github.com/Qufst/template-computo-python.git
        run: |
          git clone https://github.com/Qufst/template-computo-python.git
          cd template-computo-python
      - name: Trigger external workflow with empty commit
        env:
          GITHUB_TOKEN: ${{ secrets.REPO_DISPATCH_TOKEN }}
        run: |
            git config --local user.email "quentin.festor@etu.umontpellier.com"
            git config --local user.name "Qufst"
            git checkout main
            git commit --allow-empty -m "py_commit"
            git remote set-url origin https://x-access-token:${{ secrets.REPO_DISPATCH_TOKEN }}@github.com/Qufst/template-computo-python.git
            git push origin main

## Articuler les workflows pour répartir les charges de travail
Pour séparer les charges de travail, et gérer l'environnement séparemment de la publication, il faut utiliser les artifacts et les caches github. Avec les artifacts on transmet les petits fichiers comme les clés sha qui vont permettre à rétablir les fichiers conséquents comme les environnement avec les caches.
template python:
- https://github.com/Qufst/template-computo-python
dépôt avec les workflows appelés à distance
- https://github.com/Qufst/Workflows_computorg

# Intêret aux images apptainer/singularity

## Objectif
L'objectif principal est de créer des images apptainer pérennes des environnements utilisés pour les recherches en statistiques afin d'aider à la reproductibilité, objectif premier du journal computo. 
Un autre objectif serait d'utiliser les images apptainer pour accélérer la création d'environnements afin d'accélérer la publication d'articles utilisant des environnements conséquents.

## Problème
Le premier problème est que l'environnement présent dans l'image ne semble pas pouvoir être extrader. Il faut éxécuter les actions dans l'image. 
Un des moyens de contourner ceci est de créer un sript qui va lancer les commandes: 
```
#!/bin/bash
apptainer exec image.sif /opt/conda/envs/myenv/bin/python "$@"
```
Il suffit alors de donner les droits d'écriture à ce script et de le mettre dans le PATH. Enfin on peut utiliser le python hors de l'image.

Le problème principal rencontré est l'utilisation de quarto. En effet malgré le script python, quarto n'arrive pas à l'utiliser correctement pour ses publications. 
La seule solution que j'ai trouvé pour l'instant est de mettre quarto dans l'image apptainer lors de la création.

## Exemple

Exemple de script de création d'image python:

```
Bootstrap: docker
From: mambaorg/micromamba:1.4.6-jammy

%help
    Container with micromamba on Linux.

%files
    environment.yml /environment.yml

%post
    eval "$(micromamba shell hook -s posix)"
    micromamba activate base

    micromamba create -y -f /environment.yml -n myenv


    echo 'eval "$(micromamba shell hook -s posix)"' >> /etc/profile
    echo 'micromamba activate myenv' >> /etc/profile

    apt-get update && apt-get install -y wget tar gdb file git
    
    QUARTO_VERSION=1.5.54
    mkdir -p /opt/quarto
    wget -qO- https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz | tar -xz -C /opt/quarto --strip-components=1

    /opt/quarto/bin/quarto add --no-prompt computorg/computo-quarto-extension
    # Add Quarto to PATH
    echo 'export PATH=/opt/quarto/bin:$PATH' >> /etc/profile
%environment
    eval "$(micromamba shell hook -s posix)"
    micromamba activate myenv
    export PATH=/opt/quarto/bin:$PATH
%runscript
    python
    exec "$@"
```

Exemple de script de création d'image R:
```
Bootstrap: docker
From: rocker/r-ver:4.4.0

%files
    renv.lock /home/renv.lock

%post
    # Mettre à jour et installer les dépendances système
    apt-get update && apt-get install -y \
        software-properties-common \
        dirmngr \
        gnupg \
        apt-transport-https \
        ca-certificates \
        curl \
        make \
        git \
        gcc \
        g++ \
        libcurl4-openssl-dev \
        libssl-dev \
        libxml2-dev \
        libfontconfig1-dev \
        libfreetype6-dev \
        libtiff5-dev \
        libpng-dev \
        zlib1g-dev \
        python3-pip \
        wget

    # Créer le répertoire de cache renv avec les bonnes permissions
    mkdir -p /renv/cache
    chmod -R 777 /renv/cache

    # Installer renv
    R -e "install.packages('renv')"

    # Copier le fichier renv.lock et restaurer les dépendances R avec renv
    cp /home/renv.lock /root/renv.lock
    R -e "renv::restore(lockfile = '/root/renv.lock')"

    # Installer Jupyter
    pip3 install notebook

    # Installer Quarto
    QUARTO_VERSION=1.5.54
    mkdir -p /opt/quarto
    wget -qO- https://github.com/quarto-dev/quarto-cli/releases/download/v${QUARTO_VERSION}/quarto-${QUARTO_VERSION}-linux-amd64.tar.gz | tar -xz -C /opt/quarto --strip-components=1

    # Ajouter l'extension Quarto
    /opt/quarto/bin/quarto add --no-prompt computorg/computo-quarto-extension

    # Ajouter Quarto au PATH
    echo 'export PATH=/opt/quarto/bin:$PATH' >> /etc/profile

    # Nettoyer les fichiers temporaires
    apt-get clean
    rm -rf /var/lib/apt/lists/*

%environment
    # Définir les variables d'environnement
    export RENV_PATHS_CACHE=/renv/cache
    export PATH=/opt/quarto/bin:$PATH

%runscript
    exec "$@"
```





